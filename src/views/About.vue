<script setup>
import TeamMembers from "../json/team-members.json";
</script>

<template>
  <div class="main-container">
    <div>
      <h1 class="text-center">Team Members</h1>
      <div class="grid-x align-center grid-margin-x">
        <div
          v-for="member in TeamMembers"
          class="cell medium-shrink small-12 text-center"
        >
          <img :src="member.imgUrl" alt="team member photo" class="portrait" />
          <h3>{{ member.name }}</h3>
          <p>{{ member.description }}</p>
        </div>
      </div>
    </div>
    <div>
      <h1 class="text-center">Tools Used</h1>
      <ul>
        <li>Vue.js for the frontend UI.</li>
        <li>
          Node.js & Express.js for the backend (exposing API endpoints for
          interacting with the database).
        </li>
        <li>Nominatim API for getting locations of incidents.</li>
        <li>Leaflet API for map related features.</li>
      </ul>
    </div>
    <div>
      <h1 class="text-center">Video Overview</h1>
      <video
        class="grid-x align-center"
        src="/video/demonstration.mp4"
        controls
      ></video>
    </div>
    <div>
      <h1 class="text-center">Interesting Findings</h1>
      <ul>
        <li>
          A majority of the violent crimes occurred late at night/super early in
          the morning (10pm-4am).
        </li>
        <li>
          There was more crime during the warmer months (March - August) opposed
          to the colder months (September - February). It peaked in August and
          dipped in February.
        </li>
        <li>
          Property related crimes make up a majority of incidents in each of the
          neighborhoods.
        </li>
        <li>
          Most common incidents were Proactive Police Visits, Theft, Narcotics,
          Burglary, and Vandalism - in that order.
        </li>
        <li>
          Saint Anthony Park and Summit Hill have the lowest total incidents
          while Downtown Saint Paul has the highest.
        </li>
        <li>
          The number of incidents rose between 2014 to 2021 and has been on a
          decline since.
        </li>
      </ul>
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

.center-list {
  list-style-position: inside;
  padding-left: 0;
  margin: 0 auto;
  text-align: center;
}

.portrait {
  width: 256px;
  height: 256px;
}
</style>
