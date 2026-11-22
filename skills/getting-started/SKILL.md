---
name: getting-started
description: Vuer quick start and basic usage patterns for real-time 3D visualization in robotics and AI applications (plugin:vuer@vuer)
---

# Vuer Quick Start

Vuer: Python async backend (aiohttp) <--WebSocket--> Browser client (Three.js)

Install: `pip install vuer` (vuer 0.2 needs Python >= 3.10).

## Minimal Example

```python
from vuer import Vuer, VuerSession
from vuer.schemas import DefaultScene, Box, Sphere

app = Vuer()

@app.spawn(start=True)
async def main(session: VuerSession):
    session.set @ DefaultScene(
        Box(key="box", args=[0.2, 0.2, 0.2]),
        Sphere(key="sphere", args=[0.1], position=[0.5, 0, 0]),
    )
    await session.forever()
```

View at https://vuer.ai (connects to ws://localhost:8012), or at the bundled
client on http://localhost:8012.

To share with other people: `Vuer(auth=True)` and `app.share_url(app.issue_token("viewer", uses=5))`
(vuer >= 0.2). Pages on origins other than vuer.ai / localhost need `VUER_CORS`.

## Session APIs

| API | Purpose |
|-----|---------|
| `session.set @ Scene(...)` | Initialize root scene (once) |
| `session.upsert @ Element(...)` | Update or insert element |
| `session.update @ Element(...)` | Update existing only |
| `session.add @ Element(...)` | Add new element |
| `session.remove @ "key"` | Remove by key |

Upgrading from 0.1: read the migration guide (Python 3.10+, binary arrays, `SERVER_INFO`, `VUER_CORS` default, React 19 peers, `@dreamlake/uikit` workspace UI).
Full guides: the Dockit site in `docs-site/` of vuer-ai/vuer (wire protocol, auth, streaming, library mode, migration 0.1 -> 0.2).
