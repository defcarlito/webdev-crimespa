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
        <li>Vue</li>
        <li>Node.js & Express.js</li>
        <li>Nominatim API</li>
        <li>Leaflet API</li>
      </ul>
    </div>
    <div>
      <h1 class="text-center">Video Overview</h1>
      <!-- placeholder video for now -->
      <iframe
        class="grid-x align-center"
        src="https://www.youtube.com/watch?v=PZ_ebxkNZmo"
      ></iframe>
    </div>
    <div>
      <h1 class="text-center">Interesting Findings</h1>
      <ul>
        <li>Thing 1</li>
        <li>Thing 2</li>
        <li>Thing 3</li>
        <li>Thing 4</li>
        <li>Thing 5</li>
        <li>Thing 6</li>
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
