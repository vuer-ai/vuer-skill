---
name: events
description: Vuer event system - session APIs, custom events, handlers, RPC, streaming (plugin:vuer@vuer)
---

# Vuer Events

## Session APIs

```python
session.set @ DefaultScene(Box(key="box"))           # Init once
session.upsert @ Box(key="box", position=[1,0,0])    # Update or insert
session.upsert @ [Box(key="b1"), Sphere(key="s1")]   # Multiple
session.upsert(Box(key="child"), to="parent-key")    # To parent
session.update @ Box(key="box", color="blue")        # Update only
session.add @ Box(key="new")                         # Add new
session.remove @ "key"                               # Remove
session.remove @ ["k1", "k2"]                        # Multiple
```

## Custom Events

```python
from vuer.events import ClientEvent

class SetPositionEvent(ClientEvent):
    etype = "SET_POSITION"

# From VuerClient
client.send @ SetPositionEvent(value=[1, 2, 3])
```

## Handlers

```python
@app.add_handler("CAMERA_MOVE")
async def on_camera(event, session):
    print(event.value)  # position, rotation, matrix

cleanup = app.add_handler("EVENT", fn)
cleanup()  # Remove later
```

Events: `CAMERA_MOVE`, `HAND_MOVE`, `OBJECT_MOVE`, `CLICK`, `UPLOAD`

With `Vuer(auth=True)` (vuer >= 0.2), handlers do not get events from `viewer`
sessions except `INIT` and RPC responses; check `session.role`. Arrays that a
browser sends as typed arrays arrive as numpy ndarrays.

## RPC

```python
render = await session.grab_render(quality=0.9)
image = render.value  # PIL.Image

mesh_data = await session.get_webxr_mesh(key="webxr-mesh")
```

## Streaming

```python
@app.bind(start=True)
async def streaming(session: VuerSession):
    session.set @ DefaultScene()
    t = 0
    while True:
        t += 0.016
        yield Frame(Update(Box(key="box", position=[np.sin(t), 0, 0])), frame_rate=60)
```

### Latest frame wins (vuer >= 0.2)

For slow clients, keep only the newest unsent frame per frame key:

```python
app = Vuer(uplink_policy="latest")   # or VUER_UPLINK_POLICY=latest
session.send(Upsert(PointCloud(key="lidar", vertices=v)), frame_key="lidar")
```

Default frame keys: one for `Set`; event type + parent + element keys for
`Update`/`Upsert`/`Add`. `Remove`, RPCs and custom events are never dropped.
Send the full element state in each frame. `downlink_policy="latest"` does the
same for client streams (`CAMERA_MOVE`, `HAND_MOVE`).

Large numpy arrays (> 16 elements) go on the wire as binary typed arrays;
`Vuer(array_protocol="list")` restores the 0.1 list format.
