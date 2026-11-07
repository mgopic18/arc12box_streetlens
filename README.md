# StreetLens — point, tap, evaluate

**A mobile app for buyer's agents to identify and evaluate any property from the street, in about 30 seconds.**

The agent points the phone at a building and taps where it meets the ground. The app works out the exact address from the camera's direction and tilt, then shows what that property is worth, what comparable homes with the same number of bedrooms sold for, what is for sale and for rent around it, the indicative yield, and how the suburb performs.


---

## What we built

| Part | What it is |
|---|---|
| **Mobile app** (`mobileapp/`) | Expo / React Native, Android build plus over-the-air updates. Screens: camera scan, Records (All / Shortlisted), Insights, Analysis, Compare, Market and Suburb |
| **API** (`api/`) | Python FastAPI on our own server. It connects to the Microburbs API, runs address lookups against our own national address database, and stores scans, photos, notes and the shortlist |
| **Address database** | Australia's full G-NAF address file (about 15 million addresses) in our own PostGIS database |

What the agent can do:

1. **Scan.** Point at a building and tap its base. A pin drops, a photo is taken, and the likely addresses are listed best match first. One tap on the right address saves the scan. If the address isn't listed, type it: Google suggests addresses, and the pick is matched back to G-NAF.
2. **Insights.** This is the decision screen. Pick 1, 2, 3 or 4+ bedrooms to see:
   - the median comparable price, with how many sales it is based on and over what period
   - recent sales and current listings, those in the same building first, each with distance
   - indicative weekly rent and gross yield
   - quick notes, a shortlist star, and hand-off to a research agent (a placeholder for now)
3. **Go deeper.**
   - **Analysis:** automated valuation and range, council land value by year, sale and rent history.
   - **Compare:** the CMA comparable set and the closest match.
   - **Market:** sold, for sale and for rent, in the building and nearby.
   - **Suburb:** houses and units side by side (price, rent, yield, 12-month and 5-year growth, days on market), price history, livability, crime, flood and bushfire risk.
4. **Keep track.** Every scan is saved with its photo. Notes are kept per property, and ★ adds a property to the Shortlisted tab.

## Who it helps

**Buyer's agents**, the people who walk streets and open homes on a client's behalf and have to answer "is this one worth pursuing, and at what price?" on the spot.

- **No address hunting.** Many buildings show no number, unit blocks have several street fronts, and GPS alone is off by 5 to 15 m in a city, which can be three buildings. The camera tap finds *that* building.
- **The comparison they actually make.** A buyer considering a 2-bed apartment needs 2-bed apartment sales, not the suburb median. Insights filters by bedrooms and property type, and in a unit block it shows the building's own sales first.
- **One screen to brief the client.** Price, comparables, listings, rent, yield and the suburb's story are together on one screen. Notes and the shortlist travel with the property.

Secondary users: investors comparing apartment layouts, and home buyers walking a suburb.

## The data

| Source | Used for | Cost |
|---|---|---|
| **G-NAF** (Geocoded National Address File, data.gov.au), self-hosted in PostGIS | Every Australian address with coordinates: line-of-sight lookup, typed-address matching | Free, ours |
| **Microburbs API** (hackathon endpoint) | Address-to-ID lookup; property basics, valuation, land values, sale and rent history, comparables, recent nearby sales; suburb listings for sale and rent; suburb market, growth, livability, crime and risk | Paid per call |
| **Google Places API (New)** | Suggestions when an address is typed | Per session |
| **Phone sensors** | GPS position, compass bearing (gravity plus magnetometer), camera tilt | Free |
| **Our own records** (Neon Postgres, Cloudflare R2) | Scans, photos, notes, shortlist | Ours |

## The clever step

**Turning a tap on the camera into an exact address, using geometry rather than search.**

1. The phone's gravity sensor and magnetometer give the camera's true bearing and tilt.
2. The agent taps where the building meets the ground. That pixel becomes a ray: its **bearing** comes from the camera's field of view, and its downward **angle** together with the phone's height (about 1.5 m) gives the **distance** to the building's front by simple trigonometry.
3. That line of sight ("from here, 290°, about 18 m") is sent to **our own G-NAF database**, which returns every address within a 15 m corridor along the line. It picks the one at the measured distance, sets aside the address the agent is standing on, and collapses all the units of a tower into one building candidate.
4. Only then, once the address is confirmed, do we spend money: the address goes to Microburbs, and its property ID drives every screen.

So finding the address costs nothing and never leaves our own infrastructure.

## How to run it

The app runs on an Android phone and talks to our API, which is already live at `https://microapi.codedream.com.au`, so there is nothing to set up.

1. **Install** the StreetLens (MicroArc) app on the Android phone, and allow camera and location access when asked.
2. **Scan a building.** Tap **Scan**, hold the phone upright, and tap where the building meets the ground, below the eye-level line. A pin drops and the likely addresses are listed, best match first.
3. **Pick the address.** Tap the right one and it is looked up and saved. If it isn't listed, type it and choose from the suggestions.
4. **Evaluate it.** In **Records**, open **Insights** on the property's card and switch between 1, 2, 3 and 4+ bedrooms to compare prices, sales, listings, rent and yield.
5. **Go deeper or keep it.** Use **Analysis**, **Compare**, **Market** and **Suburb** for more detail, add notes, and tap ★ to keep the property on the Shortlisted tab.
