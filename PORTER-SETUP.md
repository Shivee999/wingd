# Porter delivery — setup

The WING'D site is wired for **Porter** (porter.in) 2-wheeler delivery. The code is
live and tested; it is **switched off** until you complete the steps below. While it is
off the site behaves exactly as before: flat delivery fee, WhatsApp checkout.

---

## Why there is an n8n step in the middle

This site is static (GitHub Pages). There is no server. **A Porter API key placed in
`menu.json` or in the page's JavaScript is readable by anyone who opens View Source**,
and could be used to book deliveries billed to the WING'D account.

So the browser never talks to Porter. It talks to two n8n webhooks, and n8n holds the key:

```
customer's browser  ──>  n8n webhook  ──>  Porter API
   (no secrets)          (holds key)      (Get Quote / Create Order)
```

---

## Steps

### 1. Get Porter API credentials
Fill the form at **https://porter.in/api-integrations**. Porter's own FAQ on that page
says credentials are issued through that form, and that a Porter Enterprise account
activates in about one business day. They will send API documentation with the keys.

Confirmed constraints from Porter's FAQ (checked August 2026):
- **2-wheeler only** via API. Trucks and tempos cannot be booked through it.
- **One pickup and one drop** per order — no multi-drop routes.
- Addresses must be sent as **latitude/longitude**, not text. This is why the site now
  asks the customer for a map pin.
- **Mumbai is a served city.**
- Fares are **dynamic** — the Get Quote API returns the live fare and also tells you
  whether a pickup or drop point is serviceable at all.
- Track Order API is **rate limited to one call per minute per order**.
- Webhooks fire on **Accepted, Live, Ended, Reopened, Cancelled**.
- The Create Order response includes a **tracking link**, also SMSed to both numbers.
- Optional **Proof of Delivery**: a delivery code is SMSed to the receiver and must be
  given to the rider to close the trip. Enable it in the Porter admin dashboard.

### 2. Import the n8n workflow
Import `porter-n8n-workflow.json` into n8n (Workflows → Import from File).

Then:
- Open **"Porter config — FILL THESE IN"** and replace `PORTER_BASE`, `PATH_QUOTE`,
  `PATH_ORDER` and `PATH_TRACK` with the real values from Porter's documentation.
- Create a **Header Auth** credential holding the Porter API key and attach it to both
  **"Porter — Get Quote"** and **"Porter — Create Order"**.
- Check the request bodies in those two nodes against Porter's documentation and fix
  the field names.
- Check the two **Normalise** Code nodes — they map Porter's response onto the shape
  the site expects (`{serviceable, fare, eta}` and `{order_id, tracking_url}`).
- Activate the workflow and copy the two **Production** webhook URLs.

> **The endpoint paths and request bodies in that workflow are placeholders.** Porter's
> documentation page blocks automated access, so those exact values could not be
> verified — they are structured guesses marked with `REPLACE`. Everything listed in
> step 1 above *was* verified from Porter's public FAQ. Do not activate without checking
> the paths against the docs Porter sends you.

### 3. Point the site at your webhooks
In `menu.json`:

```json
"porter": {
  "enabled": true,
  "quoteUrl": "https://bettercallwrld.app.n8n.cloud/webhook/wingd/porter-quote",
  "orderUrl": "https://bettercallwrld.app.n8n.cloud/webhook/wingd/porter-order",
  "pickup": { "lat": 19.0596, "lng": 72.8295, "contact": "+91…" }
}
```

**Set the real pickup coordinates.** The values shipped are approximate Bandra West
coordinates. Porter sends the rider to these numbers, not to the address text — a wrong
pin means a rider at the wrong door on every single order.

### 4. Give Porter the status webhook
`https://bettercallwrld.app.n8n.cloud/webhook/wingd/porter-status` — this receives the
Accepted / Live / Ended / Reopened / Cancelled events.

---

## What the customer sees once it is on

1. Picks **Delivery**, then sets a drop pin — "Use my location", or pastes a Google Maps
   link (the site pulls coordinates out of the URL).
2. The live Porter fare replaces the flat ₹49 and the total recalculates.
3. If the drop is outside Porter's area, checkout is blocked with an explanation rather
   than taking an order that cannot be delivered.
4. On checkout the Porter job is created, and the **order ID plus tracking link** are
   added to the WhatsApp message sent to the store.

## Failure behaviour (tested)

| Situation | What happens |
|---|---|
| `enabled: false` | No Porter UI at all. Flat fee, WhatsApp checkout. Unchanged. |
| n8n or Porter unreachable at quote | Falls back to the flat ₹49, tells the customer, order still goes through |
| n8n or Porter unreachable at booking | Order still reaches the store on WhatsApp, flagged **"PORTER BOOKING FAILED — arrange the rider manually"**, with the drop coordinates attached |
| Drop outside Porter's range | Checkout blocked, customer told to move the pin or switch to Takeaway |
| Customer refuses location permission | "Paste map link" fallback |

An order is **never** silently lost because Porter is down.

## Billing note

Above the free-delivery threshold (`freeDeliveryAbove`, currently ₹999) the customer is
charged nothing for delivery but Porter still bills the store. The cart shows this as
"Porter fare covered by the store" so the number is never hidden. Set
`porter.storeAbsorbsFareAboveFreeThreshold` to `false` to always pass the fare on.
