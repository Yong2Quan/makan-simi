<div align="center">

# 🎯 Makan Roulette

### *What to makan today?* — Cannot decide? Let the wheel decide

A mobile-first food picker for Singapore. Spin a wheel of hawker classics,
and it tells you the three nearest places to actually eat it.

<br />

### 👉 [**DECIDE NOW!**](https://yong2quan.github.io/makan-simi/) 👈

[![Open the app](https://img.shields.io/badge/Spin%20the%20wheel-yong2quan.github.io%2Fmakan--simi-B3001B?style=for-the-badge&labelColor=F4601F&logo=googlechrome&logoColor=white)](https://yong2quan.github.io/makan-simi/)

**No install, no sign-up.** Opens straight in your phone browser —
add it to your home screen and it behaves like an app.

</div>

---

## Features

| | |
|---|---|
| 🛰️ **Location** | HTML5 Geolocation, labelled by **nearest MRT station** (141 stations) with the distance shown. Or search/pick from **142 places** manually, with nickname search ("Bukit Timah" → Beauty World). |
| 🎡 **Wheel** | Canvas wheel of 16 hawker classics, `requestAnimationFrame` spin with quartic easing (~4.5–5.3s) and haptic ticks as segments pass the pointer. Labels auto-scale to the segment count. |
| 📍 **Nearby** | ~1,200 options across 80 hawker centres, ranked by haversine distance from your **exact** coordinates. Famous stalls are named; the rest point at the centre. |
| 🎉 **Result** | Confetti burst, winning dish, nearest stalls with distance + walk/drive estimate, and a Google Maps directions link. Starts at 3, expands 3 at a time or jumps to all ~13. |
| 📣 **Share** | Shareable result links — the recipient opens the same dish and places. Copy link, 8 platform targets, plus the OS share sheet. |

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

## How distances are calculated — and why the data shape matters

`haversine()` gives great-circle distance between two lat/lng points: the
straight line over the earth's surface. It is **not** walking or driving
distance, so real travel is always somewhat longer — rivers, expressways and
the lack of a bridge don't exist to it. For hawker-hunting at 0.5–5 km that
approximation is fine, and the Google Maps link gives the true route.

The maths was never the weak part. The **data** was. An earlier version hand-listed
~13 stalls per dish, clustered wherever the famous stalls happen to be, which
produced absurd results:

```
CHENDOL from Outram Park (before)        CHENDOL from Outram Park (after)
 1.  0.5 km  Old Amoy Chendol             1.  0.5 km  Old Amoy Chendol
 4.  0.9 km  Tiong Bahru Chendol          4.  0.7 km  People's Park Food Centre
 5.  2.7 km  Ah Chew Desserts             5.  0.7 km  Tanjong Pagar Plaza Market
 6.  6.0 km  Powerful Chendol  <- cliff   6.  0.8 km  Amoy Street Food Centre
```

There are obviously chendol stalls between Chinatown and Geylang — the list just
didn't contain them. Bukit Panjang had **zero** stalls within 2 km of anything.

The fix was structural rather than typing hundreds more rows. `CENTRES` lists
~80 hawker centres island-wide; `EVERYDAY` lists the dishes any decent-sized
centre reliably has (chicken rice, laksa, CKT, chendol…), and `AT_BIG_ONES`
covers satay/prata/bak kut teh, which really only show up at the larger ones.
`STALLS` is generated from that and layered under the curated `FAMOUS` list:

```js
STALLS = FAMOUS + (every CENTRE × its likely dishes, minus duplicates)
```

Coverage within 2 km, summed over all 16 dishes:

| From | Before | After |
|---|---:|---:|
| Outram Park | 30 | 179 |
| Queenstown | 8 | 100 |
| Ang Mo Kio | 9 | 99 |
| Bukit Panjang | **0** | 39 |

### Honesty about the generated entries

Generated rows carry `generic: true` and are **never given an invented stall
name**. They display the centre ("Amoy Street Food Centre") with the subtitle
"Chendol stalls · hawker centre", and their Maps link routes to the centre. The
claim being made is only "this centre has chendol stalls", which is a fair bet
for any sizeable hawker centre — not "this specific stall exists".

Remaining honest gap: Tuas and the far west industrial estates genuinely have
almost no hawker centres, so results there really are 10 km+ away.

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

## Sharing

**Jio Friends** builds a link that encodes the result, so whoever opens it lands
on the same dish and the same ranked places rather than a blank wheel:

```
https://yong2quan.github.io/makan-simi/#f=hokkien-mee&lat=1.30480&lng=103.83200&at=Orchard
```

The state rides in the URL **hash**, so it needs no server — GitHub Pages serves
one static file and the app rehydrates from the fragment on load. A shared link
also suppresses the GPS prompt, so the recipient sees the sender's context first.

### What can and can't be in the share sheet

Buttons exist only for platforms publishing a real web-share endpoint —
WhatsApp, Telegram, Facebook, X, LINE, Reddit, SMS, email.

**Instagram, TikTok and WeChat are deliberately absent.** None of them offer a
public share URL for arbitrary links: Instagram and TikTok only support sharing
from inside their own apps, and WeChat requires its JS-SDK running inside
WeChat's in-app browser. A button for any of them could only open the app's
homepage, which is worse than no button.

**Frequent contacts cannot be listed by a web page at all.** The faces at the
top of your OS share dialog come from the operating system, which knows your
address book and recent chats. A browser page cannot read either — there is no
API for it, by design. The only route to those contacts is handing off via
`navigator.share()`, which is what **More apps & contacts** does. That's also
why the dialog you saw lists your desktop's apps: that dialog *is* Windows',
not the app's.

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

## Running it locally

Just want to use the app? **[Decide Now!](https://yong2quan.github.io/makan-simi/)** —
no need to clone anything. The rest of this is for hacking on it.

**[`index.html`](index.html) is the whole app** — no build step, no dependencies,
no server. Clone and open it:

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

The live copy at **[yong2quan.github.io/makan-simi](https://yong2quan.github.io/makan-simi/)**
is served over HTTPS, so GPS works there without any of this. Every push to `main`
redeploys it within a minute.

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
