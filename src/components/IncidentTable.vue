<script setup>
import { neighborhoodToString } from "../scripts/neighborhood.js";

const props = defineProps({
  incidents: {
    type: Array,
    required: true,
  },
});

function formatDateString(dateStr) {
  return dateStr;
}

function formatTimeString(timeStr) {
  return timeStr;
}
</script>

<template>
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
      <tr v-for="inc in incidents">
        <td>{{ inc.case_number }}</td>
        <td>{{ inc.incident }}</td>
        <td>{{ neighborhoodToString(inc.neighborhood_number) }}</td>
        <td>{{ formatDateString(inc.date) }}</td>
        <td>{{ formatTimeString(inc.time) }}</td>
      </tr>
    </tbody>
  </table>
</template>
