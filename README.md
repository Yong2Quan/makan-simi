# 🎯 Makan Roulette

> *What to makan today?* — Cannot decide? Let the wheel decide

A mobile-first, single-page food picker for Singapore. Spin a wheel of hawker classics,
and it tells you the three nearest places to actually eat it.

**[index.html](index.html) is the whole app.** No build step, no dependencies to install,
no server. Open the file in a browser and it works.

---

## Features

| | |
|---|---|
| 🛰️ **Location** | HTML5 Geolocation, with a 28-area manual picker as fallback. Non-SG coordinates are detected and fall back to Orchard. |
| 🎡 **Wheel** | Canvas wheel of 8 hawker classics, `requestAnimationFrame` spin with quartic easing (~4.5–5.3s) and haptic ticks as segments pass the pointer. |
| 📍 **Nearby** | 116 curated stalls across the island, ranked by haversine distance from you. |
| 🎉 **Result** | Confetti burst, winning dish, 3 nearest stalls with distance + walk/drive estimate, and a Google Maps directions link. |
| 📣 **Share** | Web Share API where supported, clipboard fallback. |

On the wheel: Chicken Rice, Laksa, Nasi Lemak, Char Kway Teow, Roti Prata,
Mala Xiang Guo, Hokkien Mee, Kaya Toast. To add in more as my craving grows or if i am sick of eating the same food.

## Running it

```sh
# Just open it
start index.html          # Windows
open index.html           # macOS
```

Geolocation is the one catch: browsers only grant it on `https://` or `localhost`,
**not** on `file://`. Opening the file directly still works — it just falls back to the
area picker. To test real GPS, serve it locally:

```sh
npx serve .               # then visit http://localhost:3000
# or
python -m http.server 8000
```

To try it on your phone, deploy anywhere that serves static files (GitHub Pages,
Netlify, Cloudflare Pages) — it's one file.

## Stack

Vanilla JavaScript, Canvas 2D, and [Tailwind CSS](https://tailwindcss.com) via the Play
CDN. Fonts are Baloo 2 + Plus Jakarta Sans from Google Fonts. The two CDNs are the only
network calls; everything else is local.

> **Note:** The Tailwind Play CDN compiles styles in the browser, which is great for a
> single file but not ideal for production traffic. If this ever grows up, swap it for a
> real Tailwind build.

## Mobile notes

Built phone-first and tested at 390×844. Safe-area insets are respected (iPhone notch /
home indicator), the canvas renders at `devicePixelRatio` for crisp text, taps give
haptic feedback via `navigator.vibrate`, and `prefers-reduced-motion` disables the
confetti and idle animations.

## Swapping the mock data for a real API

The stall list is a plain array near the top of the `<script>` block:

```js
const STALLS = [
  { f:'chicken-rice', n:'Tian Tian Hainanese Chicken Rice',
    c:'Maxwell Food Centre, #01-10', a:'Chinatown', lat:1.28022, lng:103.84477 },
  // f = food id · n = stall name · c = centre/address · a = planning area
];
```

To go live, replace the body of `nearestStalls(foodId, origin, k)` with a call to
[Google Places Nearby Search](https://developers.google.com/maps/documentation/places/web-service/search-nearby)
or [OneMap](https://www.onemap.gov.sg/apidocs/) (free, and SG-specific), keyed on the
winning food's `name`. Return objects with the same `{ n, c, a, lat, lng, dist }` shape
and the rest of the UI needs no changes. Note that a browser-side key would be publicly
visible — proxy it through a small serverless function if you ship it.

## Caveats

Stall listings are a hand-curated snapshot, not a live directory. Coordinates are
accurate to the hawker centre, not the individual stall unit, and opening hours are *not*
confirm plus chop — better check before you make the trip.
