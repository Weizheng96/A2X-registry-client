# A2X Registry Client SDK

English | [中文](README_cn.md)

[![PyPI](https://img.shields.io/pypi/v/a2x-registry-client.svg)](https://pypi.org/project/a2x-registry-client/)
[![Python](https://img.shields.io/pypi/pyversions/a2x-registry-client.svg)](https://pypi.org/project/a2x-registry-client/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Python client for the **A2X Registry** FastAPI backend. Primary use case:
**Agent Team registration and discovery** — each team member is registered
as an A2A Agent Card (v0.0) into a shared dataset and managed (updated,
queried, deregistered) through this SDK.

## Install

```bash
pip install a2x-registry-client
```

Requires Python ≥ 3.10. Only runtime dependency is `httpx`.

## Quickstart (sync)

```python
from a2x_client import A2XRegistryClient

with A2XRegistryClient(base_url="http://127.0.0.1:8000") as client:
    client.create_dataset("research_team")   # formats defaults to {"a2a": "v0.0"}

    planner = client.register_agent("research_team", {
        "protocolVersion": "0.0",
        "name": "Task Planner",
        "description": "Decompose complex tasks into executable subtasks",
    })

    client.set_status("research_team", planner.service_id, "online")
    client.update_agent("research_team", planner.service_id,
                        {"description": "updated desc"})

    for brief in client.list_agents("research_team"):
        print(brief.id, "-", brief.name)

    client.deregister_agent("research_team", planner.service_id)
    client.delete_dataset("research_team")
```

## Quickstart (async)

```python
import asyncio
from a2x_client import AsyncA2XRegistryClient

async def main():
    async with AsyncA2XRegistryClient(base_url="http://127.0.0.1:8000") as client:
        await client.create_dataset("research_team")
        resp = await client.register_agent("research_team", {
            "protocolVersion": "0.0", "name": "a", "description": "b",
        })
        print(resp.service_id)

asyncio.run(main())
```

## Ownership model

The SDK tracks which services **this client** registered. Mutating calls
(`update_agent`, `set_status`, `deregister_agent`) require the
`service_id` to be in that tracker, otherwise `NotOwnedError` is raised
**before** any HTTP request is sent.

Ownership is persisted to `~/.a2x_client/owned.json` by default (atomic
writes after every change, segmented by `base_url`). Disable with
`ownership_file=False` or override the path with `ownership_file="..."`.

## Exceptions

All errors inherit from `A2XError`. HTTP errors (`A2XHTTPError`) carry
`status_code` and `payload`; the local ownership error (`NotOwnedError`)
does not.

- `NotFoundError` — 404
- `ValidationError` — 400 / 422
- `UserConfigServiceImmutableError` — 400 (subclass of `ValidationError`)
- `UnexpectedServiceTypeError` — `get_agent` received a non-JSON payload
- `ServerError` — 5xx
- `A2XConnectionError` — transport failure (timeout, connect refused, ...)

## Compatibility

| SDK version | Compatible A2X Registry backend |
|-------------|--------------------------------|
| 0.1.x       | v0.1.4 and later               |

## Related

- [A2X Registry](https://github.com/Weizheng96/A2Xregistry) — the backend this SDK talks to
