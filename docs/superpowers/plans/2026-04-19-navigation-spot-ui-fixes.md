# Navigation, Spot Grid, and UI Fixes Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix three UX bugs: navigation showing multiple/dotted route lines, parking spot grids not conforming to lot shapes, and bottom UI being pushed down when buttons are clicked.

**Architecture:** The app is a single-file PWA (`index.html`) using Leaflet.js for maps. We'll modify the existing route drawing, spot grid generation, and layout CSS to fix the issues while maintaining the self-contained architecture.

**Tech Stack:** Vanilla JS, Leaflet.js, CSS (single-file app)

---

## File Structure

- **Modify:** `index.html` — single self-contained file with HTML/CSS/JS
  - Lines 195, 328 — `routeLine` variable tracking
  - Lines 331-420 — `drawRoute()` function
  - Lines 260-273 — `navigateToLot()` function
  - Lines 422-471 — `drawSpotGrid()` function
  - Lines 90-146 — CSS for `.fbtn`, `.nav`, `.app` layout
  - Lines 315-323 — `genSpots()` function

---

### Task 1: Fix navigation route line accumulation bug

**Files:**
- Modify: `index.html:328-329`

**Problem:** Multiple route lines accumulate. The `clearActive()` function removes `activeLayers` and sets `routeLine = null`, but race conditions with `skipRouteRedraw` and async calls mean old routes persist.

- [ ] **Step 1: Verify the bug** Open the app in browser, select a parking lot, observe multiple lines appear. Check console for repeated `drawRoute` calls.

- [ ] **Step 2: Fix clearActive to always remove routeLine** Modify the `clearActive()` function to ensure routeLine is always removed before new routes are drawn:
```javascript
function clearActive(){
  // Always remove route line first
  if(routeLine){
    try{map.removeLayer(routeLine)}catch(e){}
    routeLine = null;
  }
  activeLayers.forEach(l=>{try{map.removeLayer(l)}catch(e){}});
  activeLayers = [];
}
```

- [ ] **Step 3: Fix drawRoute to not skip cleanup** In `drawRoute()` (line 348-349), the generation check returns early without cleaning up. Change to always clean up old routes:
```javascript
async function drawRoute(lot) {
  // Clear any existing route lines and arrow markers BEFORE making request
  clearActive();
  // Increment generation to invalidate any pending route requests
  const myGeneration = ++routeGeneration;

  const url = `https://router.project-osrm.org/route/v1/driving/` +
    `${USER.lng},${USER.lat};${lot.lng},${lot.lat}` +
    `?overview=full&geometries=geojson&steps=true`;

  let routeSuccessful = false;
  try {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 20000);
    const resp = await fetch(url, { signal: controller.signal });
    clearTimeout(timeoutId);

    // Check generation - if outdated, just return (clearActive already ran)
    if (myGeneration !== routeGeneration) return;

    const data = await resp.json();
    if (data.code !== 'Ok' || !data.routes.length) throw new Error('no route');
    routeSuccessful = true;

    const coords = data.routes[0].geometry.coordinates.map(c => [c[1], c[0]]);

    // Only draw if we're still the current generation
    if (myGeneration !== routeGeneration) return;

    routeLine = L.polyline(coords, {
      color:'#FDBF57', weight:5, opacity:0.95, lineCap:'round', lineJoin:'round'
    }).addTo(map);

    // Get bounds of route for map fitting
    const routeBounds = L.latLngBounds(coords);

    // Add "FASTEST ROUTE" label at the start of the path
    if (coords.length > 0) {
      const startCoord = coords[0];
      const fastestLabel = L.marker(startCoord, {
        icon: L.divIcon({
          className: '',
          html: `<div style="background:#7A003C;color:#FDBF57;font-size:9px;font-weight:800;padding:3px 8px;border-radius:4px;white-space:nowrap;font-family:'Barlow Condensed',sans-serif;box-shadow:0 2px 4px rgba(0,0,0,0.3);transform:translate(-50%,-180%)">★ FASTEST ROUTE</div>`,
          iconSize: [100, 20],
          iconAnchor: [50, 20]
        }),
        interactive: false,
        zIndexOffset: 300
      }).addTo(map);
      activeLayers.push(fastestLabel);
    }

    // Add turn-arrow markers every ~8 coords
    const step = Math.max(1, Math.floor(coords.length / 8));
    for (let i = step; i < coords.length - 1; i += step) {
      const a = coords[i-1], b = coords[i];
      const angle = Math.atan2(b[1]-a[1], b[0]-a[0]) * 180 / Math.PI;
      const arrow = L.marker(b, {
        icon: L.divIcon({className:'',
          html:`<div style="width:0;height:0;border-left:5px solid transparent;border-right:5px solid transparent;border-bottom:9px solid #FDBF57;transform:rotate(${angle-90}deg);transform-origin:center;opacity:0.9"></div>`,
          iconSize:[10,9], iconAnchor:[5,4]}),
        interactive: false, zIndexOffset: 200
      }).addTo(map);
      activeLayers.push(arrow);
    }

    // Update direction steps from OSRM
    const steps = data.routes[0].legs[0].steps;
    if (steps && steps.length) {
      const filtered = steps
        .filter(s => s.maneuver.type !== 'depart' || steps.indexOf(s) === 0)
        .slice(0, 6);
      document.getElementById('dirsteps').innerHTML = filtered.map((s, i) => {
        const dist = s.distance < 50 ? `${Math.round(s.distance)}m` : `${Math.round(s.distance/10)*10}m`;
        const name = s.name || s.ref || '';
        const type = s.maneuver.type;
        const mod = s.maneuver.modifier || '';
        let instr = '';
        if (type === 'depart') instr = `Head ${mod} ${name ? 'on '+name : ''}`;
        else if (type === 'arrive') instr = `Arrive at ${lot.name}`;
        else if (type === 'turn') instr = `Turn ${mod}${name ? ' onto '+name : ''}`;
        else if (type === 'continue')instr = `Continue${name ? ' on '+name : ''}`;
        else if (type === 'roundabout') instr = `Take the roundabout`;
        else instr = `${type.charAt(0).toUpperCase()+type.slice(1)}${name?' onto '+name:''}`;
        return `<div class="ds"><div class="dsn">${i+1}</div><div>${instr.trim()} <span style="color:var(--gold);font-size:10px">(${dist})</span></div></div>`;
      }).join('');
    }

    // If navigating, fit bounds to show entire route
    if (navigating && routeBounds) {
      map.fitBounds(routeBounds.pad(0.1), { duration: 0.5 });
    }

  } catch(e) {
    // Fallback: straight dashed line - only if no successful route
    if (!routeSuccessful) {
      const mid = [(USER.lat+lot.lat)/2+(lot.lat>USER.lat?.0005:-.0005),(USER.lng+lot.lng)/2-.0003];
      console.warn("Routing unavailable:", e.message);
      routeLine = L.polyline([[USER.lat,USER.lng],mid,[lot.lat,lot.lng]],
        {color:'#FDBF57',weight:3.5,opacity:.82,dashArray:'9,6',lineCap:'round'}).addTo(map);

      // Simple fallback directions
      document.getElementById('dirsteps').innerHTML = `
        <div class="ds"><div class="dsn">1</div><div>Head toward ${lot.name}</div></div>
        <div class="ds"><div class="dsn">2</div><div>Arrive at destination</div></div>
      `;
    }
  }
}
```

- [ ] **Step 4: Test the fix** Open the app, click on different lots rapidly, verify only one route line appears at a time. No dotted lines unless OSRM fails.

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "fix: prevent multiple route lines from accumulating"
```

---

### Task 2: Fix navigation "follow user" mode (like Google Maps "Go Now")

**Files:**
- Modify: `index.html:260-273`

**Problem:** When clicking Navigate, the map should follow the user's location like GPS navigation apps. Currently it just zooms once.

- [ ] **Step 1: Update navigateToLot to track user continuously** Replace the existing `navigateToLot` function with one that enables continuous tracking:
```javascript
async function navigateToLot(id) {
  navigating = true;
  document.getElementById('ovl').classList.remove('on');

  // Make sure the lot is selected (draws spots + route)
  await selectLot(id);

  const lot = allLots.find(l => l.id === id);
  if (!lot) return;

  // Show navigation banner
  showNavBanner(lot);

  // Start continuous navigation mode - map will follow user location updates
  // The updateUserMarker function handles panning when navigating=true
  toast('Navigation started — map follows your location');

  // Initial center on user
  if (userMarker) {
    map.panTo([USER.lat, USER.lng], {animate:true, duration:0.5});
  }
}
```

- [ ] **Step 2: Test navigation mode** Open app, click a lot, click Navigate button, verify map follows user location with smooth panning every 3 seconds (as controlled by `updateUserMarker`).

- [ ] **Step 3: Commit**
```bash
git add index.html
git commit -m "fix: make navigation follow user location continuously"
```

---

### Task 3: Fix parking spot grid to conform to lot bounds

**Files:**
- Modify: `index.html:422-471`

**Problem:** The spot grid is a fixed matrix that doesn't follow the actual polygon shape of the parking lot.

- [ ] **Step 1: Replace drawSpotGrid with lot-aware generation** Replace the existing `drawSpotGrid` function with one that spots only within the actual lot polygon:
```javascript
function drawSpotGrid(lot){
  const b = lot.bounds;
  const PAD = 0.08; // Smaller padding to use more of the lot
  const latMin = b.getSouth() + (b.getNorth() - b.getSouth()) * PAD;
  const latMax = b.getNorth() - (b.getNorth() - b.getSouth()) * PAD;
  const lngMin = b.getWest() + (b.getEast() - b.getWest()) * PAD;
  const lngMax = b.getEast() - (b.getEast() - b.getWest()) * PAD;

  const W = lngMax - lngMin;
  const H = latMax - latMin;

  // Calculate grid dimensions based on lot size
  const mPerLat = 111000, mPerLng = 77000;
  const lotWidthM = W * mPerLng;
  const lotHeightM = H * mPerLat;

  // Spot size: ~2.5m wide, ~5m deep
  const spotWidthM = 2.7;
  const spotDepthM = 5.2;

  // Number of spots that can fit
  const cols = Math.max(2, Math.floor(lotWidthM / spotWidthM));
  const rows = Math.max(2, Math.floor(lotHeightM / spotDepthM));

  // Calculate actual spacing
  const spotW = W / cols * 0.85;
  const spotH = H / rows * 0.80;
  const gapX = W / cols * 0.15;
  const gapY = H / rows * 0.20;

  const spots = lot.spotStates;
  let bestIdx = spots.findIndex(s => s === 'available');
  let spotCount = 0;

  // Generate spots only within the lot polygon
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      // Calculate spot center
      const cLng = lngMin + c * (spotW + gapX) + spotW / 2;
      const cLat = latMin + r * (spotH + gapY) + spotH / 2;

      // Check if spot center is inside the lot polygon
      const isInside = isPointInPolygon([cLat, cLng], lot.osmPoly);
      if (!isInside) continue;

      const idx = Math.min(spotCount++, spots.length - 1);
      const state = spots[idx] || 'taken';
      const isBest = idx === bestIdx;

      // Create rotated spot polygon (slight rotation for realism)
      const corners = [
        [cLat - spotH/2, cLng - spotW/2],
        [cLat - spotH/2, cLng + spotW/2],
        [cLat + spotH/2, cLng + spotW/2],
        [cLat + spotH/2, cLng - spotW/2]
      ];

      const fill = isBest ? '#FDBF57' : state === 'available' ? '#007B4B' : '#A6192E';
      const p = L.polygon(corners, {
        color: isBest ? '#c8921a' : state === 'available' ? '#004d30' : '#5a1020',
        weight: isBest ? 2 : 0.5,
        fillColor: fill,
        fillOpacity: state === 'taken' ? 0.65 : 0.88,
        interactive: state === 'available'
      }).addTo(map);

      if (state === 'available') {
        p.on('click', () => toast(isBest ? '★ Best spot reserved!' : 'Spot reserved!'));
      }
      activeLayers.push(p);

      if (isBest) {
        const m = L.marker([cLat, cLng], {icon: L.divIcon({className:'',
          html: `<div style="background:#FDBF57;color:#7A003C;font-size:8px;font-weight:800;padding:2px 6px;border-radius:3px;white-space:nowrap;font-family:'Barlow Condensed',sans-serif;transform:translate(-50%,-130%)">★ BEST</div>`,
          iconSize: [0, 0], iconAnchor: [0, 0]})}).addTo(map);
        activeLayers.push(m);
      }
    }
  }

  // Draw lot boundary outline
  const ol = L.polygon(lot.osmPoly, {
    color: '#FDBF57',
    weight: 2,
    fillColor: 'transparent',
    dashArray: '5,4',
    interactive: false
  }).addTo(map);
  activeLayers.push(ol);

  // Entrance marker at south edge
  const entM = L.marker([(b.getSouth() + latMin * 2) / 3, (lngMin + lngMax) / 2], {
    icon: L.divIcon({
      className: '',
      html: `<div style="background:rgba(24,9,15,0.9);border:1px solid #FDBF57;color:#FDBF57;font-size:8px;font-weight:700;padding:2px 7px;border-radius:4px;white-space:nowrap;font-family:Barlow,sans-serif;transform:translateX(-50%)">ENTRANCE ↑</div>`,
      iconSize: [0, 0],
      iconAnchor: [0, 0]
    })
  }).addTo(map);
  activeLayers.push(entM);
}

// Helper: check if point is inside polygon (ray casting algorithm)
function isPointInPolygon(point, polygon) {
  const [lat, lng] = point;
  let inside = false;
  for (let i = 0, j = polygon.length - 1; i < polygon.length; j = i++) {
    const [piLat, piLng] = polygon[i];
    const [pjLat, pjLng] = polygon[j];
    if (((piLng > lng) !== (pjLng > lng)) &&
        (lat < (pjLat - piLat) * (lng - piLng) / (pjLng - piLng) + piLat)) {
      inside = !inside;
    }
  }
  return inside;
}
```

- [ ] **Step 2: Test spot grid** Open app, click a lot with an irregular shape, verify spots only appear inside the actual lot polygon, not in a rectangular grid pattern.

- [ ] **Step 3: Commit**
```bash
git add index.html
git commit -m "fix: conform spot grid to actual lot polygon shapes"
```

---

### Task 4: Fix bottom UI being pushed down

**Files:**
- Modify: `index.html:45-46, 90-92` (CSS)

**Problem:** When buttons are clicked, the bottom navigation gets pushed down and becomes inaccessible. This is likely due to the app container allowing overflow or scroll behavior.

- [ ] **Step 1: Add overflow containment to app** Update the `.app` CSS to prevent content from pushing the nav:
```css
.app {
  max-width: 430px;
  margin: 0 auto;
  background: var(--bg);
  min-height: 100vh;
  height: 100vh; /* Fixed height */
  display: flex;
  flex-direction: column;
  overflow: hidden; /* Prevent pushing */
}
```

- [ ] **Step 2: Make scrollable areas contained** Update the `.llist` to scroll within bounds:
```css
.llist {
  overflow-y: auto;
  flex: 1; /* Take available space instead of fixed height */
  padding: 0 10px 4px;
  min-height: 0; /* Allow shrinking in flex container */
}
```

- [ ] **Step 3: Ensure nav stays fixed** Update `.nav` to have flex-shrink: 0:
```css
.nav {
  background: var(--mard);
  border-top: 2px solid var(--gold);
  padding: 9px 0 13px;
  display: flex;
  justify-content: space-around;
  flex-shrink: 0; /* Prevent shrinking */
}
```

- [ ] **Step 4: Fix fbtn to not expand** Update `.fbtn` to prevent vertical expansion:
```css
.fbtn {
  margin: 10px 12px;
  background: var(--gold);
  border: none;
  color: var(--mar);
  border-radius: 26px;
  padding: 14px;
  font-size: 16px;
  font-weight: 700;
  font-family: 'Barlow Condensed', sans-serif;
  letter-spacing: 2px;
  text-transform: uppercase;
  cursor: pointer;
  transition: all .15s;
  width: calc(100% - 24px);
  flex-shrink: 0; /* Prevent shrinking/expansion */
}
```

- [ ] **Step 5: Test UI stability** Open app in browser, click "Find Me A Spot" button repeatedly, verify bottom navigation bar stays visible and accessible.

- [ ] **Step 6: Commit**
```bash
git add index.html
git commit -m "fix: prevent bottom nav from being pushed down"
```

---

## Self-Review

**Spec coverage:**
- ✅ Task 1: Fix multiple route lines — clears old routes before drawing new ones
- ✅ Task 2: Fix navigation "follow me" — simplified to enable continuous tracking
- ✅ Task 3: Fix spot grid to lot shape — uses polygon intersection check
- ✅ Task 4: Fix bottom UI push — flexbox containment with overflow:hidden

**Placeholder scan:** No TBD/TODO placeholders found.

**Type consistency:** Function names match: `clearActive`, `drawRoute`, `navigateToLot`, `drawSpotGrid`, `isPointInPolygon`.

---

## Verification Steps

After all tasks complete, verify:

1. **Route lines:** Click rapidly between different lots — only one route line should ever be visible
2. **Navigation mode:** Click Navigate on a lot — map should follow your location
3. **Spot grid:** Look at irregularly shaped lots — spots should only appear inside the lot polygon
4. **UI:** Click "Find Me A Spot" repeatedly — navigation bar should stay visible at bottom

Run local server and test:
```bash
python3 -m http.server 8000
# Open http://localhost:8000 in browser
```
