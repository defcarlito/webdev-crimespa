<script setup>
import { neighborhoodToString } from "../scripts/neighborhood.js";

const props = defineProps({
  incidents: {
    type: Array,
    required: true,
  },
  crimeUrl: {
    type: String,
  },
});

const emit = defineEmits(["delete-row", "add-location"]);

function formatDateString(dateStr) {
  return dateStr;
}

function formatTimeString(timeStr) {
  return timeStr;
}

const violentCrimeCodes = [
  [100, 120],
  [210, 220],
  [300, 374],
  [400, 453],
  [810, 863],
  [2619, 2619],
];

const propertyCrimeCodes = [
  [500, 566],
  [600, 693],
  [700, 732],
  [900, 982],
  [1400, 1436],
];

function getRowClass(code) {
  for (const [low, high] of violentCrimeCodes) {
    if (code >= low && code <= high) return "violent-crimes";
  }

  for (const [low, high] of propertyCrimeCodes) {
    if (code >= low && code <= high) return "property-crimes";
  }

  return "other-crimes";
}

function deleteRow(caseNumber) {
  if (!confirm(`Delete case ${caseNumber}?`)) {
    return;
  }
  console.log(`Deleted case: ${caseNumber}`);
  fetch(`${props.crimeUrl}/remove-incident?case_number=${caseNumber}`, {
    method: "DELETE",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ case_number: caseNumber }),
  })
    .then((res) => res.text())
    .then(() => {
      emit("delete-row", caseNumber);
    });
}

function removeFirstXs(address) {
  const parts = address.split(" ");
  parts[0] = parts[0].replaceAll("X", "0");
  return parts.join(" ");
}

function findCrime(inc) {
  console.log(inc);
  const address = removeFirstXs(inc.block);
  console.log(address);
  fetch(`https://nominatim.openstreetmap.org/search?q=${address}&format=jsonv2`)
    .then((res) => res.json())
    .then((data) => {
      console.log(data);
      const topSearch = data[0];
      if (!topSearch) {
        console.log("Address not found");
        return;
      }
      const latLon = [topSearch.lat, topSearch.lon];
      emit("add-location", latLon, inc);
    });
}
</script>

<template>
  <div>
    <h5>Legend:</h5>
    <div style="width: fit-content">
      <ul>
        <li class="violent-crimes">Violent crimes</li>
        <li class="property-crimes">Property crimes</li>
        <li class="other-crimes">Other crimes</li>
      </ul>
    </div>
  </div>
  <table>
    <thead>
      <tr>
        <th>Case #</th>
        <th>Type</th>
        <th>Neighborhood</th>
        <th>Date</th>
        <th>Time</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="inc in incidents" :class="getRowClass(inc.code)">
        <td>{{ inc.case_number }}</td>
        <td>{{ inc.incident }}</td>
        <td>{{ neighborhoodToString(inc.neighborhood_number) }}</td>
        <td>{{ formatDateString(inc.date) }}</td>
        <td>{{ formatTimeString(inc.time) }}</td>
        <td>
          <button class="button alert" @click="deleteRow(inc.case_number)">
            Delete
          </button>
          <button class="button secondary" @click="findCrime(inc)">
            Find Crime
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style>
.violent-crimes {
  background-color: rgba(255, 0, 0, 0.3) !important;
}

.property-crimes {
  background-color: rgba(0, 0, 255, 0.3) !important;
}

.other-crimes {
  background-color: rgba(0, 128, 0, 0.3) !important;
}
</style>
