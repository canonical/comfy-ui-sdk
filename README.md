# ComfyUI SDK for Workshop

This SDK provides ComfyUI as a persistent browser-accessible service for
node-based diffusion workflows in Workshop. ComfyUI runs with GPU acceleration
on NVIDIA, AMD, and Intel GPUs with a CPU fallback when no supported GPU is
available. The Python virtual environment, downloaded models, and generated
outputs are persisted on the host across workshop updates. Optionally, connect
the `venv` plug to the `uv` SDK for a uv-managed environment.

---

## Reference workshop

A minimal workshop:

```yaml
# workshop.yaml
name: comfy-app
base: ubuntu@24.04
sdks:
  - name: system
    plugs:
      comfy-ui:
        interface: tunnel
        endpoint: 127.0.0.1:8188
  - name: comfy
    channel: latest/stable

actions:
  verify: |
    curl -fsS http://127.0.0.1:8188/system_stats
```

This exposes ComfyUI through the browser tunnel, so the UI is accessible at
`http://localhost:8188` on the host.

---

## Using the SDK

### Prerequisites, project layout

1. No prerequisite SDKs are required for the default setup. The `uv` SDK is
   optional; see [Using a uv-managed venv](#using-a-uv-managed-venv) below.
2. No specific project layout is needed. ComfyUI reads and writes its own
   directories under `/home/workshop/comfy/` inside the workshop.
3. The first launch installs PyTorch and ComfyUI dependencies into the virtual
   environment via the `setup-project` hook, so it takes longer than subsequent
   launches. The installed packages are persisted via the `venv` mount plug and
   reused across workshop updates.

### Access ComfyUI

After `workshop launch`, open `http://localhost:8188` in a browser.

- The default ComfyUI workflow is available in the UI. Custom workflows can be
  loaded by drag-and-drop; see the
  [ComfyUI repository](https://github.com/comfyanonymous/ComfyUI) for workflow
  examples and upstream documentation.
- Generated images are written to `/home/workshop/comfy/output/`, which
  persists across workshop updates via the `output` mount plug.

### Verify from the command line

To confirm the ComfyUI service is running, check the user service from a
workshop shell:

```bash
workshop shell
systemctl --user status comfy
journalctl --user -u comfy
```

### Using a uv-managed venv

By default, ComfyUI runs inside a Python virtual environment managed by the
SDK itself. If you prefer to manage packages with `uv`, you can connect the
`comfy:venv` plug to the `uv` SDK's `venv` slot.

When connected, the uv SDK's virtual environment is mounted at the comfy SDK's
venv path. PyTorch and ComfyUI dependencies are still installed into the venv
on first launch. This lets you manage packages with `uv pip` instead of plain
`pip`.

```yaml
# workshop.yaml
name: comfy-app
base: ubuntu@24.04
sdks:
  - name: system
    plugs:
      comfy-ui:
        interface: tunnel
        endpoint: 127.0.0.1:8188
  - name: uv
    channel: latest/stable
  - name: comfy
    channel: latest/stable

connections:
  - plug: comfy:venv
    slot: uv:venv
```

---

## Plugs (resources this SDK consumes)

### `gpu`

- Interface: `gpu`
- Purpose: Grants access to host GPU hardware for accelerated inference.

### `venv`

- Interface: `mount`
- Workshop target: `$SDK/venv`
- Purpose: Persists the Python virtual environment (PyTorch and ComfyUI
  dependencies) across workshop updates.

### `models`

- Interface: `mount`
- Workshop target: `/home/workshop/comfy/models`
- Purpose: Persists downloaded model checkpoints, VAEs, LoRAs, and other model
  files across workshop updates.

### `output`

- Interface: `mount`
- Workshop target: `/home/workshop/comfy/output`
- Purpose: Persists generated images and other output files across workshop
  updates.

## Slots (resources this SDK provides)

### `comfy-ui`

- Interface: `tunnel`
- Endpoint: `127.0.0.1:8188`
- Purpose: Exposes the ComfyUI HTTP server to the host for browser access.
  Connect a matching plug on the `system` SDK to make ComfyUI accessible at
  the plug address on the host.

---

## Documentation and guidance

- [ComfyUI repository and documentation](https://github.com/comfyanonymous/ComfyUI)
- [ComfyUI examples](https://comfyanonymous.github.io/ComfyUI_examples/)
- [Workshop documentation](https://canonical-workshop.readthedocs-hosted.com/latest/)

---

## Community and support

- ComfyUI community: [ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI)
- Workshop forum:
  [Workshop Discourse](https://discourse.canonical.com/c/engineering/workshops/34)
- Please review our
  [Code of Conduct](https://ubuntu.com/community/ethos/code-of-conduct) before
  participating.

---

## Contributions

All contributions, including code, documentation updates, and issue reports,
are welcome!

- See `CONTRIBUTING.md` for guidelines.
- Open issues or pull requests on the official repository.

---

## License and copyright

Copyright 2025-2026 Canonical Ltd.

ComfyUI is licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.html).
