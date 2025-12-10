<script setup>
import { reactive, ref, onMounted, watch } from "vue";
import IncidentTable from "./components/IncidentTable.vue";
import { neighborhoodToString } from "./scripts/neighborhood.js";

import NewIncidentForm from "./components/NewIncidentForm.vue";
import IncidentTypeFilter from "./components/IncidentTypeFilter.vue";
import IncidentLimitFilter from "./components/IncidentLimitFilter.vue";
import NeighborhoodFilter from "./components/NeighborhoodFilter.vue";
import DateFilter from "./components/DateFilter.vue";

import incidentTypes from "./json/incident-types.json";
import neighborhoods from "./json/neighborhoods.json";

let crime_url = ref("http://localhost:8000"); // temp filled
let dialog_err = ref(false);

// init neighborhood number crime sums to 0
let sums = {};
for (let i = 1; i <= 17; i++) {
  sums[i] = 0;
}
let crimeSums = ref(sums);
let incidents = ref([]);

// filters
let selectedIncidentTypeFilters = ref([]);
let selectedLimit = ref(1000);
let selectedNeighborhoods = ref([]);
let selectedStartDate = ref("");
let selectedEndDate = ref("");

let map = reactive({
  leaflet: null,
  center: {
    lat: 44.955139,
    lng: -93.102222,
    address: "",
  },
  zoom: 12,
  bounds: {
    nw: { lat: 45.008206, lng: -93.217977 },
    se: { lat: 44.883658, lng: -92.993787 },
  },
  neighborhood_markers: [
    { location: [44.942068, -93.020521], marker: null, nn: 1 },
    { location: [44.977413, -93.025156], marker: null, nn: 2 },
    { location: [44.931244, -93.079578], marker: null, nn: 3 },
    { location: [44.956192, -93.060189], marker: null, nn: 4 },
    { location: [44.978883, -93.068163], marker: null, nn: 5 },
    { location: [44.975766, -93.113887], marker: null, nn: 6 },
    { location: [44.959639, -93.121271], marker: null, nn: 7 },
    { location: [44.9477, -93.128505], marker: null, nn: 8 },
    { location: [44.930276, -93.119911], marker: null, nn: 9 },
    { location: [44.982752, -93.14791], marker: null, nn: 10 },
    { location: [44.963631, -93.167548], marker: null, nn: 11 },
    { location: [44.973971, -93.197965], marker: null, nn: 12 },
    { location: [44.949043, -93.178261], marker: null, nn: 13 },
    { location: [44.934848, -93.176736], marker: null, nn: 14 },
    { location: [44.913106, -93.170779], marker: null, nn: 15 },
    { location: [44.937705, -93.136997], marker: null, nn: 16 },
    { location: [44.949203, -93.093739], marker: null, nn: 17 },
  ],
});
let locationInput = ref("St. Paul, MN");
let searchMarker = null;

// Vue callback for once <template> HTML has been added to web page
onMounted(() => {
  // Create Leaflet map (set bounds and valied zoom levels)
  map.leaflet = L.map("leafletmap").setView(
    [map.center.lat, map.center.lng],
    map.zoom,
  );
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution:
      '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    minZoom: 11,
    maxZoom: 18,
  }).addTo(map.leaflet);
  map.leaflet.setMaxBounds([
    [44.883658, -93.217977],
    [45.008206, -92.993787],
  ]);

  // Add center marker
  const centerIcon = L.divIcon({
    className: "center-marker",
    html: '<div style="background: red; width: 20px; height: 20px; border-radius: 50%; border: 3px solid white; box-shadow: 0 0 5px black;"></div>',
    iconSize: [20, 20],
    iconAnchor: [10, 10],
  });

  const centerMarker = L.marker(map.leaflet.getCenter(), {
    icon: centerIcon,
    interactive: false,
  }).addTo(map.leaflet);

  // Update location when map moves
  map.leaflet.on("moveend", async () => {
    const center = map.leaflet.getCenter();
    centerMarker.setLatLng(center);

    // Get address for center
    const response = await fetch(
      `https://nominatim.openstreetmap.org/reverse?format=json&lat=${center.lat}&lon=${center.lng}`,
    );
    const data = await response.json();
    locationInput.value = data.display_name;
  });

  // Get boundaries for St. Paul neighborhoods
  let district_boundary = new L.geoJson();
  district_boundary.addTo(map.leaflet);
  fetch("data/StPaulDistrictCouncil.geojson")
    .then((response) => {
      return response.json();
    })
    .then((result) => {
      result.features.forEach((value) => {
        district_boundary.addData(value);
      });
    })
    .catch((error) => {
      console.log("Error:", error);
    });
  initializeCrimes(); // temp
});

function removeFirstXs(address) {
  const parts = address.split(" ");
  parts[0] = parts[0].replaceAll("X", "0");
  return parts.join(" ");
}

// FUNCTIONS
// Function called once user has entered REST API URL
function initializeCrimes() {
  fetch(`${crime_url.value}/incidents`)
    .then((res) => res.json())
    .then((data) => {
      incidents.value = data;
      sumCrimeNums(incidents.value);
      drawNeighborhoods();
    });
}

// Sum crime amounts for each neighborhood
function sumCrimeNums(incidents) {
  for (let key in crimeSums.value) {
    sums[key] = 0;
  }

  incidents.forEach((inc) => {
    crimeSums.value[inc.neighborhood_number] += 1;
  });
}

// Function called when user presses 'OK' on dialog box
function closeDialog() {
  let dialog = document.getElementById("rest-dialog");
  let url_input = document.getElementById("dialog-url");
  if (crime_url.value !== "" && url_input.checkValidity()) {
    dialog_err.value = false;
    dialog.close();
    initializeCrimes();
  } else {
    dialog_err.value = true;
  }
}

// Draw neighborhood markers
function drawNeighborhoods() {
  map.neighborhood_markers.forEach((n) => {
    if (n.marker) {
      map.leaflet.removeLayer(n.marker);
    }

    L.marker([n.location[0], n.location[1]])
      .addTo(map.leaflet)
      .bindTooltip(
        `
          <p><b>${neighborhoodToString(n.nn)}</b></p>
          <p>${crimeSums.value[n.nn.toString()]} crimes</p>
        `,
        { permanent: true, className: "marker-label" },
      );
  });
}

function filterIncidents() {
  // build codes
  let allCodes = [];
  selectedIncidentTypeFilters.value.forEach((incidentType) => {
    const lowerBound = incidentType.code_range[0];
    const upperBound = incidentType.code_range[1];
    for (let i = lowerBound; i <= upperBound; i++) {
      allCodes.push(i);
    }
  });

  // build neighborhood numbers
  let allNeighborhoodsNumbers = [];
  selectedNeighborhoods.value.forEach((neighborhood) => {
    allNeighborhoodsNumbers.push(neighborhood.id);
  });

  const url = new URL("incidents", crime_url.value);
  url.searchParams.set("limit", selectedLimit.value);
  if (allCodes.length != 0) {
    const allCodesString = allCodes.join(",");
    url.searchParams.set("code", allCodesString);
  }
  if (allNeighborhoodsNumbers.length != 0) {
    const allNNString = allNeighborhoodsNumbers.join(",");
    url.searchParams.set("neighborhood", allNNString);
  }
  if (selectedStartDate.value != 0) {
    url.searchParams.set("start_date", selectedStartDate.value);
  }
  if (selectedEndDate.value != 0) {
    url.searchParams.set("end_date", selectedEndDate.value);
  }

  fetch(url)
    .then((res) => res.json())
    .then((data) => {
      incidents.value = data;
      sumCrimeNums(data);
      drawNeighborhoods();
    });
}
// Go to location
async function goToLocation() {
  const address = locationInput.value + ", St. Paul, MN";
  const response = await fetch(
    `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(address)}`,
  );
  const data = await response.json();

  if (data.length > 0) {
    const lat = parseFloat(data[0].lat);
    const lng = parseFloat(data[0].lon);
    map.leaflet.setView([lat, lng], 15);
  }
}
</script>

<template>
  <div className="main-container">
    <div class="grid-container">
      <div class="grid-x grid-padding-x">
        <div class="cell small-9">
          <input
            type="text"
            v-model="locationInput"
            @keyup.enter="goToLocation"
            placeholder="Enter address"
          />
        </div>
        <div class="cell small-3">
          <button class="button" @click="goToLocation">Go</button>
        </div>
      </div>
    </div>
    <div class="grid-container">
      <div class="grid-x grid-padding-x">
        <div id="leafletmap" class="cell auto"></div>
      </div>
    </div>
    <div id="form">
      <NewIncidentForm />
    </div>
    <div id="filter">
      <h1>Filters:</h1>
      <IncidentTypeFilter
        :incidentTypes="incidentTypes"
        v-model:selectedFilters="selectedIncidentTypeFilters"
      />
      <IncidentLimitFilter v-model="selectedLimit" />
      <NeighborhoodFilter
        :neighborhoods="neighborhoods"
        v-model:selectedNeighborhoods="selectedNeighborhoods"
      />
      <DateFilter
        v-model:start="selectedStartDate"
        v-model:end="selectedEndDate"
      />
      <button class="button" @click="filterIncidents">Filter</button>
    </div>
    <div id="table">
      <IncidentTable :incidents="incidents" />
    </div>
  </div>
</template>

<style>
.main-container {
  margin: 10vh;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

#rest-dialog {
  width: 20rem;
  margin-top: 1rem;
  z-index: 1000;
}

.grid-container,
.grid-x,
.cell {
  width: 100%;
}

#leafletmap {
  height: 500px;
}

.dialog-header {
  font-size: 1.2rem;
  font-weight: bold;
}

.dialog-label {
  font-size: 1rem;
}

.dialog-input {
  font-size: 1rem;
  width: 100%;
}

.dialog-error {
  font-size: 1rem;
  color: #d32323;
}

.marker-label {
}

.marker-label p {
  margin: 0px;
}
</style>
