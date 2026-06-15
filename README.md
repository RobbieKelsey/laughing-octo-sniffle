# DragonPass — API Integration Playground

A three-column sandbox that lets a developer feel what integrating the DragonPass
v2 API does to the end-user experience. Toggle features on the left, watch the
mocked app rebuild in the middle, and read the matching API request/response
pairs on the right — all driven by the real Arazzo workflow set.

```
┌──────────────┬──────────────────────┬───────────────────────┐
│ 1 · Features │ 2 · End-user app     │ 3 · API calls         │
│ presets,     │ live phone preview   │ request + response,   │
│ modules,     │ that rebuilds as you │ cURL / JS / Java,     │
│ capabilities │ flip features        │ Arazzo flow trace     │
└──────────────┴──────────────────────┴───────────────────────┘
```

Everything is **mocked** — no credentials, no network, nothing leaves the machine.
Responses mirror the live envelope (`{ "code": 0, "data": { … } }`) and the real
endpoints (auth → search → resource → issue/order → redeem → usage).

---

## Run it

### Easiest — just open the file
Double-click **`index.html`**. It is fully self-contained (no build step, no
server, works offline) and runs in any modern browser. For a demo this is all
you need.

### As a localhost server (the "exe-style" experience)
Needs Python 3 (already on macOS/Linux; on Windows install from python.org and
tick *Add Python to PATH*).

- **macOS / Linux:** double-click `run.command` (first time you may need
  `chmod +x run.command`), or run `python3 serve.py`
- **Windows:** double-click `run.bat`, or run `python serve.py`

It picks a free port, serves the playground, and opens your browser at
`http://localhost:<port>`. Stop with Ctrl+C.

### A genuine standalone `.exe` (no Python on the demo machine)
A Windows `.exe` must be built **on Windows** (PyInstaller can't cross-compile
from Linux/Mac). One command, on the target OS:

```bash
pip install pyinstaller
pyinstaller --onefile --name DragonPassPlayground --add-data "index.html;." serve.py
```

(on macOS/Linux the separator is a colon: `--add-data "index.html:."`)

The result lands in `dist/` — a single double-clickable binary that serves the
bundled page on localhost. `serve.py` already handles the PyInstaller bundle
path, so no code changes are needed.

---

## How to extend it

The whole app is config-driven. Open `index.html` and look for the numbered
config blocks near the top of the `<script>`:

| To add… | Edit | Notes |
|---|---|---|
| A product module (e.g. Spa) | `MODULES` | give it an emoji, blurb, price, and a couple of `sample` resources — a tile and list screen appear automatically |
| A capability toggle | `CAPABILITIES` | add the entry; wire any screen logic where `cap('yourKey')` is checked |
| A one-click scenario | `PRESETS` | list the `modules` and `caps` it switches on |
| An Arazzo flow | `FLOWS` + `STEP_LABEL` | list the ordered step ids; the flow bar renders them |
| An API endpoint | `API.*` | return `{ method, path, body, response, flow, step }` and call `logCall()` from a button handler in `handleAct()` |

No framework, no bundler — vanilla JS so it stays portable and easy to hand to
another engineer.

---

## What's modelled today

- **Issuing models:** E-pass, **Cross-Module E-pass**, and Membership & Entitlements
- **Modules:** Lounge (1), Fast Track (2), Dining/Coupon (4), Fitness (7), eSIM (8) — integer codes match the portal
- **Capabilities:** search & discovery, prebooking, walk-in redemption, dynamic QR, user management, push events (webhooks), sandbox simulation
- **Auth:** JWT bearer token via `POST /v2/auth/token`, auto-fetched on first call, shown in the top bar
- **Flows:** authenticate, discover, issue & redeem E-pass, **cross-module pass**, E-pass prebooking, register membership & redeem, membership prebooking, eSIM purchase & top-up, fitness booking & check-in, sandbox redemption simulation

### Schema fidelity

Every call in the console carries a badge:

- **✓ schema** — request/response field names verified against the developer portal. These are exact: `POST /v2/auth/token`, `POST /v2/users`, `GET /v2/search/modules`, `POST /v2/orders/{module}/ePasses`, `POST /v2/orders/multiModule/ePasses`, `POST /v2/orders/{module}/ePasses/prebooking`, `POST /v2/memberships`, `POST /v2/memberships/qrCodes`, `POST /v2/simulations/lounges/redemptions`.
- **≈ shape** — representative shape (resource list/details, availability, membership prebooking, preview, eSIM order/top-up, fitness check-in, usage details). Consistent with the verified fields and the documented patterns, but the individual schema wasn't pulled — confirm against the portal before relying on exact names.

Real details baked in: integer `module` codes, `status`/`category`/`ePassStatus` enums, the `X-Program-ID` and `X-Request-ID` (UUID idempotency) headers on every call, and the distinct identity fields — `ePassId` (16-digit, walk-in QR), `voucher` (prebooking redemption, `MS…`), `membershipId` (16-digit), and `dpId` (unified id for simulations).

### Cross-Module note

Per the DragonPass spec, a Cross-Module E-pass (`module: 0`, issued at `/v2/orders/multiModule/ePasses`) supports **only Lounge (1) + Set Meal (3) + Coupon (4)** — Fast Track, Fitness and eSIM are **not** cross-module eligible. The playground reflects this: the "Cross-Module pass" preset enables Lounge + Dining and greys out the ineligible modules. One pass is redeemed at either venue, and the resulting usage order re-attributes to the actual module used (1 or 4), exactly as the API behaves.
