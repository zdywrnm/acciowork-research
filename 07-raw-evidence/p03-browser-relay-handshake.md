# manifest.json
```json
{
  "manifest_version": 3,
  "name": "Accio Browser Relay",
  "version": "0.1.3",
  "description": "Attach Accio to your existing Chrome tab via a local CDP relay server.",
  "icons": {
    "16": "icons/icon16.png",
    "32": "icons/icon32.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },
  "permissions": ["debugger", "tabs", "tabGroups", "windows", "activeTab", "scripting", "storage", "alarms", "notifications"],
  "host_permissions": ["<all_urls>"],
  "background": { "service_worker": "background.js", "type": "module" },
  "action": {
    "default_title": "Accio Browser Relay",
    "default_popup": "pages/popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "32": "icons/icon32.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "options_ui": { "page": "pages/options.html", "open_in_tab": true }
}
```

# constants.js (ports)
```js
/**
 * Shared constants for the Accio Browser Relay extension.
 */

export const DEFAULT_CONTROL_PORT = 9234
export const RELAY_PORT_OFFSET = 2

const MAX_CONTROL_PORT = 65535 - RELAY_PORT_OFFSET

export function clampPort(value) {
  const n = Number.parseInt(String(value || ''), 10)
  if (!Number.isFinite(n) || n <= 0 || n > MAX_CONTROL_PORT) return DEFAULT_CONTROL_PORT
  return n
}

export function computeRelayPort(controlPort) {
  return clampPort(controlPort) + RELAY_PORT_OFFSET
}

export const RelayState = Object.freeze({
  DISABLED: 'disabled',
  DISCONNECTED: 'disconnected',
  CONNECTED: 'connected',
})

export const TabType = Object.freeze({
  USER: 'user',
  AGENT: 'agent',
  RETAINED: 'retained',
})

export const STATE_UI = {
  [RelayState.DISABLED]:     { dotColor: null,      title: 'Accio Browser Relay — disabled' },
  [RelayState.DISCONNECTED]: { dotColor: '#F59E0B', title: 'Accio Browser Relay — connecting…' },
  [RelayState.CONNECTED]:    { dotColor: '#22C55E', title: 'Accio Browser Relay — connected' },
}

export const STATE_TEXT = {
  [RelayState.DISABLED]:     { label: 'Not Enabled', detail: 'Click the toggle or toolbar icon to enable.' },
  [RelayState.DISCONNECTED]: { label: 'Connecting…', detail: 'Relay is enabled. Trying to connect…' },
  [RelayState.CONNECTED]:    { label: 'Connected',   detail: 'Relay is active — agent can control your browser.' },
}

// ── Settings keys ──

export const SETTINGS_KEYS = Object.freeze({
  CLOSE_GROUP_ON_DISABLE: 'closeGroupOnDisable',
})

export async function getSetting(key, defaultValue = false) {
  const stored = await chrome.storage.local.get([key])
  return stored[key] ?? defaultValue
}

export async function setSetting(key, value) {
  await chrome.storage.local.set({ [key]: value })
}
```

# connection.js (preflight + websocket + hello)
```js
function connectToPort(port, signal) {
  const httpBase = `http://127.0.0.1:${port}`
  const wsUrl = `ws://127.0.0.1:${port}/extension`

  return (async () => {
    const preflightCtrl = new AbortController()
    const preflightTimer = setTimeout(() => preflightCtrl.abort(), PREFLIGHT_TIMEOUT)
    if (signal) signal.addEventListener('abort', () => preflightCtrl.abort(), { once: true })
    try {
      await fetch(`${httpBase}/`, { method: 'HEAD', signal: preflightCtrl.signal })
        .catch((err) => {
          if (signal.aborted) throw new Error('Connection cancelled')
          throw new Error(`Relay server not reachable at ${httpBase} (${String(err)})`)
        })
    } finally {
      clearTimeout(preflightTimer)
    }

    if (signal.aborted) throw new Error('Connection cancelled')

    const ws = new WebSocket(wsUrl)

    try {
      await new Promise((resolve, reject) => {
        const t = setTimeout(() => reject(new Error('WebSocket connect timeout')), WS_CONNECT_TIMEOUT)
        const cleanup = () => { clearTimeout(t); signal.removeEventListener('abort', onAbort) }
        const onAbort = () => { cleanup(); reject(new Error('Connection cancelled')) }
        signal.addEventListener('abort', onAbort, { once: true })
        ws.onopen = () => { cleanup(); resolve() }
        ws.onerror = () => { cleanup(); reject(new Error('WebSocket connect failed')) }
        ws.onclose = (ev) => { cleanup(); reject(new Error(`WebSocket closed (${ev.code} ${ev.reason || 'no reason'})`)) }
      })
    } catch (err) {
      try { ws.close() } catch { /* already closing */ }
      throw err
    }

    if (signal.aborted) {
      try { ws.close() } catch { /* noop */ }
      throw new Error('Connection cancelled')
    }

    return ws
  })()
}

/**
 * Race multiple candidate ports in parallel via Promise.any.
 * First port to complete preflight + WS open wins; losers are aborted and closed.
 * Falls back to single-port connect when only one candidate.
 *
 * @param {number[]} ports
 * @param {AbortSignal} signal
 * @returns {Promise<WebSocket>}
 */
async function raceConnectPorts(ports, signal) {
  if (ports.length === 1) return connectToPort(ports[0], signal)

  const perPort = ports.map((port) => {
    const ctrl = new AbortController()
    signal.addEventListener('abort', () => ctrl.abort(), { once: true })
    return { port, ctrl, promise: connectToPort(port, ctrl.signal) }
  })

  const winner = await Promise.any(perPort.map(async (entry) => {
    const ws = await entry.promise
    return { ws, port: entry.port }
  }))

  for (const entry of perPort) {
    if (entry.port !== winner.port) {
      entry.ctrl.abort()
      entry.promise.then((ws) => { try { ws.close() } catch {} }).catch(() => {})
    }
  }

  log.info('connected to relay port', winner.port)
  return winner.ws
}

async function ensureConnection() {
  if (relayWs && relayWs.readyState === WebSocket.OPEN) return
  if (relayConnectPromise) return await relayConnectPromise

  const currentAbortCtrl = new AbortController()
  connectAbortCtrl = currentAbortCtrl
  const { signal } = currentAbortCtrl

  relayConnectPromise = (async () => {
    const defaultPort = await getRelayPort()
    const multiProbe = await shouldProbeMultiplePorts()
    const ports = multiProbe && ALT_RELAY_PORT !== defaultPort
      ? [ALT_RELAY_PORT, defaultPort]
      : [defaultPort]

    const ws = await raceConnectPorts(ports, signal)

    relayWs = ws
    ws.onmessage = (event) => void onRelayMessage(String(event.data || ''))
    ws.onclose = () => { if (relayWs === ws) onRelayClosed('closed') }
    ws.onerror = () => { if (relayWs === ws) onRelayClosed('error') }

    // Handshake: send Extension.hello so the server knows our protocol version.
    //
    // protocolVersion: 桌面端与扩展之间的通信协议兼容性标识（整数），
    //   与 manifest.json 中的 version（展示版本号）是两个独立概念。
    //   仅在通信协议发生不兼容变更时才递增。
    //
    // extensionVersion: 从 manifest.json 读取的展示版本号，
    //   仅用于 mismatch 时桌面端展示给用户，不参与兼容性判断。
    //
    // encryptedSessionKey: AES-256 session key encrypted with the embedded RSA public key.
    //   If present, the server will decrypt and enable transport encryption.
    //   If absent (old extension), communica
```

# connection.js (helloAck + encryption)
```js
d：
  //   - protocolVersion 不匹配 → reloadTargetKey = 'proto:<n>'（随后 ws 会被关闭）
  //   - extensionVersion 不一致但协议兼容 → reloadTargetKey = 'ext:<version>'（连接保持可用）
  // 桌面端主动要求扩展 reload（独立于 helloAck 的显式指令）
  if (msg?.method === 'Extension.reload') {
    log.info('Extension.reload: received reload command from desktop')
    void maybeSelfReload(msg.params?.reloadTargetKey || 'ext:forced')
    return
  }

  if (msg?.method === 'Extension.helloAck') {
    const p = msg.params || {}
    if (p.status === 'ok') {
      if (p.encrypted === true) {
        activateEncryption()
        void chrome.storage.local.remove('_relayError')
        log.info('Extension.helloAck: handshake OK — transport encryption activated (AES-256-GCM)')
      } else {
        log.warn('Extension.helloAck: server does not support encryption — disconnecting')
        void chrome.storage.local.set({
          _relayError: 'encryption_unsupported',
        })
        disconnect()
        return
      }
    } else {
      log.warn('Extension.helloAck: server rejected —', p.status)
    }
    if (p.action === 'reload' && typeof p.reloadTargetKey === 'string' && p.reloadTargetKey.length > 0) {
      void maybeSelfReload(p.reloadTargetKey)
    }
    return
  }

  const MESSAGE_EXPIRE_MS = 130_000
  if (typeof msg?.ts === 'number') {
    const age = Date.now() - msg.ts
    if (age > MESSAGE_EXPIRE_MS) {
      log.warn('dropping expired message:', msg.method || msg.id, 'age:', age, 'ms')
      if (typeof msg.id === 'number') {
        trySendToRelay({ id: msg.id, error: `Message expired (age ${age}ms > ${MESSAGE_EXPIRE_MS}ms)` })
      }
      return
    }
  }

  if (typeof msg?.id === 'number' && msg.method === 'forwardCDPCommand') {
    const cdpMethod = msg.params?.method || ''
    pushLog('↓', msg.method, cdpMethod)
    try {
      const result = await callbacks.onMessage(msg)
      trySendToRelay({ id: msg.id, result })
    } catch (err) {
      trySendToRelay({ id: msg.id, error: err instanceof Error ? err.message : String(err) })
    }
  }
}

// ── Public lifecycle methods ──

/**
 * Attempt to connect. On failure, automatically schedules reconnect
 * if relay is still enabled — callers do NOT need to call scheduleReconnect.
 */
export async function connectAndAttach() {
  const t0 = performance.now()
  log.debug('connectAndAttach: begin, connected:', isRelayConnected(), 'reconnecting:', _reconnectPending)
  if (isRelayConnected()) return true
  if (_reconnectPending) { log.debug('connectAndAttach: reconnect already scheduled, skip'); return false }
  if (!(await isRelayEnabled())) { log.debug('con
```