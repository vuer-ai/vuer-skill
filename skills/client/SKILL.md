---
name: client
description: VuerClient for connecting to Vuer servers and sending custom events (plugin:vuer@vuer)
---

# VuerClient

## Basic Usage

```python
import asyncio
from vuer import VuerClient
from vuer.events import ClientEvent

class MyEvent(ClientEvent):
    etype = "MY_EVENT"

async def main():
    # token= (or VUER_TOKEN) for servers with auth=True (vuer >= 0.2)
    async with VuerClient(URI="ws://localhost:8012") as client:
        client.send @ MyEvent(value={"data": 123})  # Fire-and-forget
        await client.send(MyEvent(value="x"))       # Awaitable

asyncio.run(main())
```

## Custom Events

```python
class SetPositionEvent(ClientEvent):
    etype = "SET_POSITION"
    value = [0, 0, 0]  # Optional default
```

## Receiving

Since vuer 0.2 the first event is `SERVER_INFO` (`protocol`, `arrayProtocol`,
`role`); skip it in custom readers. numpy arrays arrive as ndarrays (binary
ExtType 0x56); the client announces protocol 2 and sends binary arrays only to
servers that announced protocol 2.

```python
event = await client.recv(timeout=5.0)

async for event in client:
    print(event.etype, event.value)
```

## Server Handler

```python
@app.add_handler("MY_EVENT")
async def on_event(event, session: VuerSession):
    session.upsert @ Box(key="box", position=event.value)
```

## Browser client as a library (@vuer-ai/vuer 0.1)

No server needed: `<VuerScene scene={...} onEvent={...} ref={handle} />` with
`handle.set/add/update/upsert/remove`, and `<VuerLocalProvider>` for vuer
components (e.g. MuJoCo) inside your own r3f `<Canvas>`. Peers: React 19,
@react-three/fiber 9, three >= 0.178. The `./workspace` entry uses
`@dreamlake/uikit` ^0.3.0 (load `@dreamlake/uikit/styles.css`) and follows the
host theme (`data-theme`, `?theme=`, or a `{type: 'theme', theme, source:
'vuer-host'}` postMessage); there is no theme switch.

Full guides: the Dockit site in `docs-site/` of vuer-ai/vuer (wire protocol, auth, streaming, library mode, migration 0.1 -> 0.2).
