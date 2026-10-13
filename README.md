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
| 🛰️ **Location** | HTML5 Geolocation, labelled by **nearest MRT station** (141 stations) with the distance shown. Or search/pick from **142 places** manually, with nickname search ("Bukit Timah" → Beauty World). |
| 🎡 **Wheel** | Canvas wheel of 16 hawker classics, `requestAnimationFrame` spin with quartic easing (~4.5–5.3s) and haptic ticks as segments pass the pointer. Labels auto-scale to the segment count. |
| 📍 **Nearby** | 220 curated stalls across the island, ranked by haversine distance from your **exact** coordinates. |
| 🎉 **Result** | Confetti burst, winning dish, 3 nearest stalls with distance + walk/drive estimate, and a Google Maps directions link. |
| 📣 **Share** | Web Share API where supported, clipboard fallback. |

On the wheel: Chicken Rice, Laksa, Nasi Lemak, Char Kway Teow, Roti Prata,
Mala Xiang Guo, Hokkien Mee, Kaya Toast, Bak Chor Mee, Satay, Wanton Mee,
Bak Kut Teh, Fish Soup, Carrot Cake, Yong Tau Foo, Chendol.
To add in more as my craving grows or if i am sick of eating the same food.

### Adding a dish

Append to `FOODS`, then add stalls for it in `STALLS` with the matching `f:` id:

```js
{ id:'duck-rice', name:'Duck Rice', emoji:'🦆', color:'#8A0014',
  tag:'Braised or roasted, sauce until drown.' },
```

The wheel re-fits itself — `LABEL_SCALE` shrinks the labels as segments get
narrower, and the shrink-to-fit loop keeps text clear of the hub. Give each new
dish **at least 3 stalls**, ideally ~10 spread across the island, so the
nearest-3 lookup works wherever you happen to be standing.

## How "where am I" works

Geolocation gives exact coordinates, which is what stall distances are measured
from — the label never affects the ranking. But raw coordinates are useless to
read, so the app names the **nearest MRT station** and shows how far it is:

```
Outram Park MRT · 460 m away
```

Earlier versions snapped to the nearest of 28 planning-area centroids, which
told you "Chinatown" while you were standing in Outram. Stations are ~5× more
granular, need no API key or network call, and match how people here actually
describe where they are. Beyond 1.6 km from any station (Changi Village, Lim Chu
Kang) it falls back to the planning area.

Station coordinates are approximate to the station entrance, good to ~100 m.
If you want true street-level accuracy, swap `describeLocation()` for
[OneMap's reverse geocode](https://www.onemap.gov.sg/apidocs/) — it returns road
name, building and postal code, at the cost of an API token and a network round
trip.

The manual picker (`PLACES`) is **derived**, not hand-listed: every MRT station,
plus any planning area more than 1.2 km from a station (currently just Changi
Village). Add a station to `MRT` and it appears in the picker automatically.
`PLACE_ALIASES` adds searchable nicknames where the neighbourhood and the
station go by different names:

```js
'Beauty World':   ['Bukit Timah', 'Upper Bukit Timah'],
'Tanjong Katong': ['Katong', 'Joo Chiat'],
'Raffles Place':  ['CBD', 'Downtown Core'],
```

## Why directions search by name, not coordinates

`directionsUrl()` sends the venue **name** as a Maps search query rather than a
`lat,lng` pair. This is deliberate and a little counter-intuitive:

> A bare coordinate carries no name, so Google reverse-geocodes it to whichever
> building is nearest. `1.28022,103.84477` is ~50 m from Maxwell Food Centre —
> and resolved to *"MND Building, 5 Maxwell Rd"* across the road. The pin was
> fine; the label and the walking route were the neighbour's.

Searching `"Jin Hua Fish Head Bee Hoon, Maxwell Food Centre, Singapore"` lets
Maps resolve its own correctly-pinned place entry. Unit numbers (`#01-77`) are
stripped first because they derail the search.

The trade-off: a wrongly-spelled stall name sends you to the wrong place, where
a coordinate would at least have been geographically close. Per-stall precision
would need a Google **Place ID** for each entry, which means the Places API.

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
