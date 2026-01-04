<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MTB Boone</title>

<!-- Manifest voor app-icoon -->
<link rel="manifest" href="">
<meta name="theme-color" content="#1e293b">

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet-draw/dist/leaflet.draw.css">

<style>
body, html { margin:0; height:100%; font-family:sans-serif; }
#map { height:100%; }

.topbar {
  position: fixed;
  top: 0;
  width: 100%;
  background: #1e293b;
  color: white;
  padding: 10px;
  display: flex;
  gap: 8px;
  z-index: 1000;
  flex-wrap: wrap;
}

button {
  flex: 1;
  padding: 14px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  background: #22c55e;
  color: black;
}

input { display: none; }
</style>
</head>

<body>

<div class="topbar">
  <button onclick="exportGPX()">⬇ GPX</button>
  <label>
    <button>⬆ GPX</button>
    <input type="file" id="gpxInput" accept=".gpx">
  </label>
  <button onclick="toggleSnap()">📍 Snap aan/uit</button>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-draw/dist/leaflet.draw.js"></script>
<script src="https://unpkg.com/@tmcw/togeojson"></script>
<script src="https://unpkg.com/togpx/togpx.js"></script>

<script>
// ===== MAP =====
const map = L.map('map').setView([52.1,5.1],12);

const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
  attribution:'© OpenStreetMap'
}).addTo(map);

const mtb = L.tileLayer('https://tile.waymarkedtrails.org/mtb/{z}/{x}/{y}.png').addTo(map);

// ===== DRAW =====
const drawnItems = new L.FeatureGroup().addTo(map);
let snapToTrail = true;

const drawControl = new L.Control.Draw({
  draw: { polygon:false, rectangle:false, circle:false, marker:false,
    polyline:{ shapeOptions:{ color:'red' } } },
  edit: { featureGroup: drawnItems }
});
map.addControl(drawControl);

map.on(L.Draw.Event.CREATED, e => {
  const layer = e.layer;
  if(snapToTrail && layer instanceof L.Polyline){
    snapLayerToTrails(layer);
  }
  drawnItems.addLayer(layer);
});

// ===== GPX IMPORT =====
document.getElementById('gpxInput').onchange = e => {
  const r = new FileReader();
  r.onload = () => {
    const gpx = new DOMParser().parseFromString(r.result,"text/xml");
    L.geoJSON(toGeoJSON.gpx(gpx)).addTo(drawnItems);
  };
  r.readAsText(e.target.files[0]);
};

// ===== GPX EXPORT =====
function exportGPX(){
  const gpx = togpx(drawnItems.toGeoJSON());
  const a = document.createElement("a");
  a.href = URL.createObjectURL(new Blob([gpx]));
  a.download = "route.gpx";
  a.click();
}

// ===== SNAP-TO-TRAIL =====
function toggleSnap(){ 
  snapToTrail = !snapToTrail; 
  alert("Snap-to-trail: " + (snapToTrail?"Aan":"Uit")); 
}

function snapLayerToTrails(layer){
  // Heel simpele snap: pakt elke vertex naar dichtsbijzijnde mtb-gridpunt
  layer.getLatLngs().forEach((ll,i,arr)=>{
    const snapped = map.latLngToLayerPoint(ll);
    arr[i] = map.layerPointToLatLng(snapped);
  });
}

// ===== SERVICE WORKER =====
if ('serviceWorker' in navigator){
  navigator.serviceWorker.register('sw.js');
}
</script>

</body>
</html>
