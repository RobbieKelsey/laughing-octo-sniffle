# DragonPass — API Integration Playground

A three-column sandbox that lets a developer feel what integrating the DragonPass
v2 API does to the end-user experience. Toggle features on the left, watch the
mocked app rebuild in the middle, and read the matching API request/response
pairs on the right — every endpoint modelled from the published v2 specs.

```
┌──────────────┬──────────────────────┬───────────────────────┐
│ 1 · Features │ 2 · End-user app     │ 3 · API calls         │
│ recipes,     │ a DragonPass-styled  │ request + response,   │
│ presets,     │ phone that rebuilds  │ cURL / JS / Py /      │
│ modules,     │ as you flip features │ Java / C#,            │
│ capabilities │ + admin console      │ Postman export        │
└──────────────┴──────────────────────┴───────────────────────┘
```

The centre column is deliberately split in two. The **phone** is what a
DragonPass client's own app looks like — it contains only what an end user would
ever see. The **admin console** beneath it holds everything an integrator calls
from their *backend*: user and membership administration, pricing lookups,
cancellations, push-event replay and sandbox simulations. Keeping those apart is
what lets the phone stay a believable product demo rather than a button board.

It runs in two transports, switched from the top bar:

- **Mock** (default) — canned, portal-accurate responses. No network, no
  credentials, nothing leaves the machine. Responses mirror the live envelope
  (`{ "code": 0, "data": { … } }`) and the real endpoints.
- **Live** — your own sandbox credentials against the real DragonPass API.
  Every call in the console is a genuine request: real status code, real
  latency, real response body (including real error envelopes).

**Sandbox only.** There is deliberately no production switch. This is a learning
and integration tool, and a mistyped click against production would create real
orders and consume real entitlements.

---

## Live mode

Open **Live** in the top bar and a *Connection* panel appears in the left rail.

| Field | What to put in it |
|---|---|
| Base URL | `https://developer-sandbox.dragonpass.com/api` (prefilled) |
| X-Program-ID | The program id DragonPass assigned you |
| Auth | Either **Client creds** (`clientId` + `secret` → `POST /v2/auth/token`) or **Access token** (paste a JWT you signed yourself — see the portal's HS256/RS256 notes) |

Then press **Connect**. The Connection panel confirms the token, and every
subsequent tap in the phone issues a real request.

**No server or proxy is required.** The sandbox's gateway is CORS-permissive —
it returns `Access-Control-Allow-Origin: *` and `Access-Control-Allow-Headers: *`
on both the preflight and the response — so the browser calls it directly. Live
mode therefore works from any served origin: localhost, GitHub Pages, or any
static host.

The wildcard is valid for these requests because the token travels in an
`Authorization` header rather than a cookie; `*` is only refused for requests
that set `credentials: 'include'`, which this never does.

Credentials live in the tab's memory for as long as it is open. Nothing is
written to disk, `localStorage` or `sessionStorage`, and the client secret is
replaced with `<your-client-secret>` in the generated snippets so they are safe
to copy and paste.

---

## Run it

### Easiest — just open the file
Double-click **`index.html`**. It is fully self-contained (no build step, no
server, no dependencies) and runs in any modern browser — this is all you need
for a mock-mode demo, and it works offline. For **live** mode, serve it over
localhost (below) or a static host.

### Hosting it (GitHub Pages or any static host)

`index.html` is the entire application — every asset, including the icon font
and the DragonPass logo, is inlined. Deployment is "commit the file":

```bash
git add index.html && git commit -m "Deploy playground" && git push
# then: repo Settings → Pages → deploy from branch
```

It works unchanged at a project path such as `you.github.io/dp-playground/` —
nothing in the page resolves a relative URL. No `.nojekyll` is needed (there are
no underscore-prefixed files). Pages serves over HTTPS, so calling the HTTPS
sandbox raises no mixed-content problem, and **live mode works on the deployed
page with no extra infrastructure**.

Two things worth knowing before you publish one publicly:

- Anyone who opens it can use live mode **with their own credentials**. Nothing
  is baked into the page, credentials never leave the tab except to DragonPass,
  and there is no shared state — but it is a public tool, not a private one.
- It is sandbox-only by construction. There is no production switch to fat-finger.

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

## Assets & attribution

The playground ships as a single self-contained file, so every asset is inlined.
`assets/build_assets.py` is the build-time step that produces that inline block —
it is **not** needed to run the playground.

```bash
pip install fonttools brotli
python assets/build_assets.py
```

It writes two marked blocks into `index.html` (`ASSETS:BEGIN/END` in `<head>`,
`SPRITE:BEGIN/END` after `<body>`):

- **DragonPass module icons** from `../Icons/*.svg` — the genuine DP set, matching
  the app one-for-one. Inlined as `<symbol>`s with `fill="currentColor"`.
- **The official DragonPass logo** — mark and wordmark, pulled from the
  [media centre](https://www.dragonpass.com/media-centre#dragonpass-logos) zip
  that the portal's own UI Design Guidelines point integrators at. Also used as
  the favicon. © DragonPass.
- **UI icons: [Flaticon UIcons](https://www.flaticon.com/uicons)** (solid,
  rounded), used under the free licence, which **requires attribution** — hence
  the credit in the left rail and here. The full webfont is 188 KB, so the script
  subsets it to the ~45 glyphs actually used: **3.9 KB**.

If you add an icon, add its name to `GLYPHS` in the build script and re-run it.

---

## How to extend it

The whole app is config-driven. Open `index.html` and look for the numbered
config blocks near the top of the `<script>`:

| To add… | Edit | Notes |
|---|---|---|
| A product module (e.g. Spa) | `MODULES` | give it an `icon` (a sprite symbol id), blurb, price, `walkin`/`prebook` flags and a couple of `sample` resources — a tile and list screen appear automatically. Set `lifestyle:true` if it is not hub-bound, `options:true` if orders are placed at the option level |
| A capability toggle | `CAPABILITIES` | add the entry; wire any screen logic where `cap('yourKey')` is checked |
| A one-click scenario | `PRESETS` | list the `modules` and `caps` it switches on |
| A guided journey | `RECIPES` | list steps with the endpoint each `match`es; they tick automatically |
| A test fixture | `PERSONAS` | PII plus `adults`/`children`; `PERSON` resolves to the selected one |
| An API endpoint | `API.*` | return `{ method, path, body, response }` and call `logCall()` from a handler in `handleAct()` |

> The Arazzo workflow tracker that used to live above the phone has been removed
> from the UI. Its definitions are preserved in **`ARAZZO.md`** if you want to
> bring it back — but keep it out of the phone column.

No framework, no bundler — vanilla JS so it stays portable and easy to hand to
another engineer.

---

## What's modelled today

- **Issuing models:** E-pass, **Cross-Module E-pass**, and Membership & Entitlements
- **Modules:** Lounge (1), Fast Track (2), **Set Meal (3)**, Dining Coupon (4), Fitness (7), eSIM (8), **Local Offer (11)** — integer codes match the portal

### Per-module capabilities

The app only ever offers a journey the API actually supports. Two independent
sources in the portal agree on this matrix — which modules have a
"Create …Prebooking Order" endpoint, and which `/v2/simulations/{module}/…`
variants exist — and `MODULES` encodes it as `walkin` / `prebook` flags:

| Module | Code | Walk-in | Pre-booking |
|---|---|---|---|
| Lounge | 1 | ✅ | ✅ |
| Fast Track | 2 | ❌ | ✅ |
| Set Meal | 3 | ✅ | ❌ |
| Dining Coupon | 4 | ✅ | ❌ |
| Fitness | 7 | ❌ | ✅ (option-level) |
| eSIM | 8 | ❌ | ✅ (option-level) |
| Local Offer | 11 | ❌ | ✅ (option-level) |

So Fast Track never shows "Get Pass", Set Meal and Dining never show
"Book a slot", and each says why rather than silently hiding the control.
Resource cards carry a green **Pre-booking available** pill driven by the same
flag.

### Pass display

"Get Pass" issues the pass (creating the user implicitly) and then
shows it, exactly as a real app would — the redemption fires from the pass
screen, not from the venue listing. Both variants follow the portal's
[UI Design Guidelines](https://apifox.dragonpass.com/apidoc/docs-site/6000000/338654m0):

| Walk-in shows | Pre-booking shows |
|---|---|
| QR code | Voucher no. + QR |
| Membership / E-pass ID | DragonPass logo |
| DragonPass logo | User's name |
| User's name | Pre-booking date & time |
| Expiration date | Resource name & location |

The QR is **real and scannable** — a dependency-free encoder (byte mode, ECC M,
versions 1–6 with Reed–Solomon) is included, and the test suite decodes its
output back with `jsqr` to prove it. Per the guidelines it encodes *only* the
16-digit id, the voucher, or the dynamic string: "Including any additional text
may cause verification to fail." When Dynamic QR is switched off, the membership
card falls back to a static QR of the membership ID.
- **Capabilities:** search & discovery, prebooking, walk-in redemption, dynamic QR, user management, **cancel & amend**, **pricing queries**, push events (webhooks + recovery), sandbox simulation
- **Auth:** JWT bearer token via `POST /v2/auth/token`, auto-fetched on first call
- **Guided recipes:** issue & redeem a pass · pre-book a lounge · cross-module pass · membership & entitlements · buy an eSIM · claim a local offer. Picking one switches on what it needs and ticks each step off as you reach its endpoint.
- **Test personas:** Jordan Lee (solo, UK), Priya Raman (cardholder + guest, SG), Tomás Duarte (family of four, BR) — they drive the PII and passenger counts in every request body.
- **Shareable permalink:** the scenario is encoded in the URL fragment. Credentials never are; `serialiseState()` reads a fixed whitelist and there is a test asserting no credential substring can reach the URL.
- **Export:** copy any request or response, snippets in cURL / JS / Python / Java / C#, and a one-click Postman v2.1 collection of the session.

### Local Offer

Local Offers (module 11) are merchant discounts rather than airport services, so
they behave differently and the playground models that faithfully:

- They are **lifestyle resources** — `GET /v2/resources` returns them under
  `lifestyleResources`, not `transportHubResources`, and they are keyed by city
  rather than transport hub.
- Orders are placed at the **option** level under a merchant resource, so they
  use `POST /v2/resources/options/prebookings/availability` — *not* the
  resource-level availability endpoint that Lounge and Fast Track use.
- `GET /v2/resources/localOffers/{resourceId}/options` returns each offer with a
  `marketingTag` (1 Buy X get Y · 2 Discount · 3 Value off · 4 Set meal) and a
  `validity` block that is one of three shapes: a fixed date range, a relative
  duration (`value` + `DAY|MONTH|YEAR`), or dynamic.
- The order response carries `localOfferOptions{optionId, optionName,
  redemptionUrl, voucherValidUntil}`, and **each order covers one person only**.
- The client must render the voucher from `vouchersList` — the playground has a
  dedicated voucher screen for this, because presenting the E-pass ID instead
  will be refused at the merchant.

### Schema fidelity

Every call in the console carries a badge:

Every endpoint the playground models is now **✓ schema** — pulled from the
portal's own OpenAPI specs (`apifox.dragonpass.com/.../llms.txt` indexes them
all). The badge is still rendered per call so the distinction stays meaningful
if a future addition is only representative.

| Area | Endpoints |
|---|---|
| Auth | `POST /v2/auth/token` |
| Users | `POST /v2/users`, `/v2/users/search`, `/v2/users/update`, `/v2/users/delete`, `/v2/users/ePasses`, `/v2/users/memberships` |
| Search & hubs | `POST /v2/search`, `GET /v2/transportHubs`, `GET /v2/transportHubs/{id}`, `GET /v2/search/modules` |
| Resources | `GET /v2/resources`, `GET /v2/resources/{resourceId}`, `POST /v2/resources/prebookings/availability`, `POST /v2/resources/options/prebookings/availability` |
| Module options | `GET /v2/resources/esims/{id}/options`, `GET /v2/resources/fitness/{id}/schedule`, `GET /v2/resources/fitness/{id}/options/{optionId}`, `GET /v2/resources/localOffers/{id}/options`, `GET /v2/resources/localOffers/{id}/options/{optionId}` |
| Pricing | `POST /v2/resources/prices`, `POST /v2/resources/options/prices` |
| E-pass | `POST /v2/orders/{module}/ePasses`, `/v2/orders/multiModule/ePasses`, `/v2/orders/{module}/ePasses/prebooking`, `/v2/orders/{module}/ePasses/bundle`, `POST /v2/orders/ePasses/search`, `POST /v2/orders/ePasses` |
| Membership | `POST /v2/memberships`, `/v2/memberships/search`, `/v2/memberships/update`, `/v2/memberships/qrCodes`, `/v2/entitlements/update`, `/v2/entitlements/search`, `/v2/orders/{module}/memberships/prebooking`, `/v2/orders/{module}/memberships/prebooking/preview`, `POST /v2/orders/memberships` |
| eSIM | `/v2/orders/esims/{ePasses\|memberships}/prebooking`, `/v2/orders/esims/{ePasses\|memberships}/topup`, `/v2/orders/esims/topup/availability`, `/v2/orders/esims/packages/query`, `/v2/orders/esims/details` |
| Fitness | `/v2/orders/fitness/{ePasses\|memberships}/prebooking`, `POST /v2/orders/fitness/checkin` |
| Local Offer | `/v2/orders/localOffers/{ePasses\|memberships}/prebooking` |
| Usage & lifecycle | `POST /v2/orders/ePasses/details`, `/v2/orders/memberships/details`, **`PATCH /v2/orders`** (cancel an order or an E-pass) |
| Sandbox simulation | `/v2/simulations/{lounges\|fastTrack\|setMeals\|coupons}/redemptions`, `.../prebookings/redemptions`, `.../cancellation` |
| Push | `walkin.redemption`, `prebooking.statusChanged`, `resource.metadata.updated`, `GET /v2/event` (recovery) |

Corrections made against earlier versions of this playground:

- The sandbox host is **`https://developer-sandbox.dragonpass.com/api`** (the
  server URL in every portal spec), not `sandbox-api.dragonpass.com`. The
  production host is not published, so it is left blank for you to supply.
- `GET /v2/resources` splits results into `transportHubResources` and
  `lifestyleResources`; Fitness, eSIM and Local Offer belong to the latter.
- Option-based modules use `/v2/resources/options/prebookings/availability`,
  not the resource-level availability endpoint.
- The membership prebooking **preview** returns `usedEntitlementDetails` and
  `usedEntitlementBreakdown[]` — there is no `payable` block.
- Sandbox simulation paths are per module (`setMeals`, `coupons`, `fastTrack`),
  and Fast Track has prebooking simulation only, no walk-in.
- Cancellation is a single `PATCH /v2/orders` with `{status: 3, orderId}` for
  both orders and E-passes.

Webhook payloads are exact: the **walk-in redemption** event (`eventType:"walkin.redemption"`) carries `orderId, programId, dpId, module, status (2 Created / 3 Cancelled), orderCreatedDate, usageDate, orderCancelledDate, transportHubId, transportHubName, resourceId, resourceName, passengers{cardholder,guests}`; the **prebooking status change** event (`eventType:"prebooking.statusChanged"`) carries `orderId, programId, dpId, module, previousStatus, currentStatus, statusChangedDate, remarks`. Both expect the client to ACK with HTTP 200 and `{"status":"success"}`.

Module-specific detail from the portal: **eSIM** options return `optionList[]{optionId,optionName,duration,dataMode,totalVolume,unit}`, and an eSIM order returns the full `esim{iccid,status,...}` + `esimOptions{volume{total,remaining,dailyLimit},coverage{regions,rawText},providers[]}` block with `esimOrderType` (1 = new, 2 = top-up). **Fitness** schedule returns `schedules[].optionList[]{optionId,optionName,optionType(1 Day Pass/2 Class),startTime,endTime,price{...}}`; a fitness order returns `fitnessOptions{...}`. **Entitlements** are granted one `POST /v2/entitlements/update` per module returning `entitlements.moduleExclusiveEntitlements[].details{...}`. The **E-pass order list** returns `orders[]{orderId,status,usedUsages,module,category,orderCreatedDate,resourceId,resourceName,image,extra{optionId,optionName,transportHubId,transportHubName,city,countryOrRegion}}`.

Earlier corrections still apply: availability is `POST /v2/resources/prebookings/availability` returning `available`+`price`; membership prebooking/preview carry `toUseEntitlementCount`/`toUseEntitlementDetail`; usage detail is a POST by `orderId`; fitness check-in returns `vouchersList[]`.

Real details baked in: integer `module` codes, `status`/`category`/`ePassStatus` enums, the `X-Program-ID` and `X-Request-ID` (UUID idempotency) headers on every call, and the distinct identity fields — `ePassId` (16-digit, walk-in QR), `voucher` (prebooking redemption, `MS…`), `membershipId` (16-digit), and `dpId` (unified id for simulations).

### Cross-Module note

Per the DragonPass spec, a Cross-Module E-pass (`module: 0`, issued at `/v2/orders/multiModule/ePasses`) supports **only Lounge (1) + Set Meal (3) + Coupon (4)** — Fast Track, Fitness and eSIM are **not** cross-module eligible. The playground reflects this: the "Cross-Module pass" preset enables Lounge + Dining and greys out the ineligible modules. One pass is redeemed at either venue, and the resulting usage order re-attributes to the actual module used (1 or 4), exactly as the API behaves.
