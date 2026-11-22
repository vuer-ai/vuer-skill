---
name: server
description: Vuer server configuration, decorators, event handlers, SSL/TLS for VR (plugin:vuer@vuer)
---

# Vuer Server

## Configuration

```python
app = Vuer(
    port=8012,              # WebSocket port
    workspace=".",          # Files served at /workspace/
    free_port=False,        # Auto-kill process on port
    # vuer >= 0.2 (needs Python >= 3.10):
    auth=False,             # True: every WebSocket needs a token
    array_protocol="auto",  # auto | binary | list (list = 0.1 wire format)
    uplink_policy="fifo",   # fifo | latest (latest frame wins for slow clients)
)
```

Environment variables: `VUER_PORT`, `VUER_DOMAIN`, `VUER_CORS`, `VUER_WORKSPACE`,
`VUER_AUTH`, `VUER_AUTH_SECRET`, `VUER_ARRAY_PROTOCOL`, `VUER_UPLINK_POLICY`,
`VUER_DOWNLINK_POLICY`

## Origins (VUER_CORS)

Since 0.2 `VUER_CORS` is an allow-list of fnmatch origin patterns. It is used
for HTTP CORS and for the WebSocket `Origin` check (HTTP 403). Default:
`https://vuer.ai,https://*.vuer.ai,http(s)://localhost:*,http(s)://127.0.0.1:*,https://dash.ml`.
Same-origin pages and non-browser clients are always allowed. Add your page's
origin, or set `VUER_CORS="*"` for the 0.1 behavior.

## Access tokens and roles (0.2)

```python
app = Vuer(auth=True)                              # tokens checked before the upgrade (401)
token = app.issue_token("viewer", ttl=600, uses=1) # one-time; uses=5 or uses=None (reusable)
print(app.share_url(token))                        # <client url>#token=...

@app.spawn(role="controller")                      # session.role: "viewer" | "controller"
async def operator(session): ...
```

Shell: `vuer token --role viewer --uses 5 --url http://localhost:8012`,
`vuer token --reusable --ttl 3600`. Viewers see the scene; the app gets only
their `INIT` and RPC responses. `/relay` needs a controller token.

## Binary arrays (0.2)

numpy arrays with more than 16 elements go to protocol-2 clients as
MessagePack ExtType 0x56 (typed arrays in the browser). Small arrays and old
clients get lists. Each session gets a `SERVER_INFO` event first
(`protocol`, `arrayProtocol`, `role`).

Full guides: the Dockit site in `docs-site/` of vuer-ai/vuer (wire protocol, auth, streaming, library mode, migration 0.1 -> 0.2).

## Decorators

```python
# Main handler (start=True auto-starts)
@app.spawn(start=True)
async def main(session: VuerSession):
    session.set @ DefaultScene()
    await session.forever()

# Generator for streaming
@app.bind(start=True)
async def handler(session: VuerSession):
    while True:
        yield Frame(Update(...), frame_rate=60)
```

## Event Handlers

```python
@app.add_handler("CAMERA_MOVE")
async def on_camera(event: ClientEvent, session: VuerSession):
    print(event.value)
```

Events: `CAMERA_MOVE`, `HAND_MOVE`, `CLICK`, `UPLOAD`, `OBJECT_MOVE`

## SSL for VR

VR requires wss://. Use ngrok:

```bash
ngrok http 8012
# Visit: https://vuer.ai?ws=wss://xxxx.ngrok.io
```

Or configure SSL:
```python
app = Vuer(cert="/path/to/cert.pem", key="/path/to/key.pem")
```
