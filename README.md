# WING'D — website + menu + ordering

Live site: https://shivee999.github.io/wingd/

- `index.html` — main site (hero wing build animation, heat scale, menu preview, find us)
- `menu.html` — full menu, item customiser, cart, delivery/takeaway/dine-in, WhatsApp checkout
- `menu.json` — **the only file you need to edit.** Items, prices, flavours, dips, fees, hours, store WhatsApp number.
- `style.css` — shared design system

## Before going live
1. Replace every price in `menu.json` with the real till price.
2. Replace `store.whatsapp` with the WING'D store number (country code, digits only, no `+`).
3. Confirm `deliveryFee`, `freeDeliveryAbove`, `packagingFee`, `gstPercent`, `minOrder`, `hours`, `address`, `phone`.
4. Optional: add `swiggy` / `zomato` links.

Edit `menu.json` straight in GitHub's web editor — the site picks it up on the next page load.
