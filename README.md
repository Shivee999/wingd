# WING'D — one-page site + menu + ordering

Live: https://shivee999.github.io/wingd/

Single page, single link. Home / Menu / Find Us are tabs inside `index.html`.

- `index.html` — the whole site
- `menu.json` — **the only file you need to edit.** Items, prices, sauces, dips, fees, hours, store WhatsApp number.
- `style.css` — brand system taken from the printed WING'D menu
- `assets/rooster.png` — brand illustration extracted from the printed menu
- `porter-n8n-workflow.json` + `PORTER-SETUP.md` — Porter (porter.in) 2-wheeler delivery integration, shipped **off**

## Menu source
All items and prices are transcribed from `WINGD_Menu_Print.pdf` (Aug 2026):
Wings 4/6/8 PC ₹349|479|649 · Boneless 8/10/12 PC ₹349|479|649 · Flavour Fix Platter ₹949 ·
Peri-Peri Tenders ₹379|599 · Classic Tenders ₹399|599 · Fries ₹349–479 · Sides ₹349–479 ·
Dips ₹50 · Beverages ₹199.

## Before going live — NOT from the printed menu, confirm each
1. `store.whatsapp` — currently a placeholder. Set the WING'D store number (country code, digits only, no `+`).
2. `deliveryFee` (49), `freeDeliveryAbove` (999), `packagingFee` (20), `minOrder` (199), `deliveryRadiusKm` (6), `prepTimeMins`, `hours`, `address`, `phone`.
3. `gstPercent` (5) — the correct restaurant GST rate depends on the establishment's category. Confirm with an accountant.

## Porter delivery
Wired and tested, currently `porter.enabled: false`. See **PORTER-SETUP.md** to switch it on.

Edit `menu.json` straight in GitHub's web editor; the site picks it up on the next page load.
