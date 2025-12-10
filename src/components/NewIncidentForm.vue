<template>
  <div>
    <button class="button" @click="showForm = !showForm">
      {{ showForm ? "Hide Form" : "Add New Incident" }}
    </button>

    <transition name="slide">
      <div v-if="showForm" class="callout" style="margin-top: 1rem">
        <h4>Add New Incident</h4>

        <div v-if="errorMessage" class="callout alert">
          {{ errorMessage }}
        </div>

        <form @submit.prevent="submitIncident">
          <div class="grid-x grid-margin-x">
            <div class="cell small-12 medium-6">
              <label>
                Case Number:
                <input v-model="case_number" />
              </label>
            </div>

            <div class="cell small-12 medium-6">
              <label>
                Date (YYYY-MM-DD):
                <input v-model="date" />
              </label>
            </div>
          </div>

          <div class="grid-x grid-margin-x">
            <div class="cell small-12 medium-6">
              <label>
                Time (HH:MM:SS):
                <input v-model="time" />
              </label>
            </div>

            <div class="cell small-12 medium-6">
              <label>
                Code:
                <input v-model="code" />
              </label>
            </div>
          </div>

          <div class="grid-x grid-margin-x">
            <div class="cell small-12">
              <label>
                Incident Description:
                <input v-model="incident" />
              </label>
            </div>
          </div>

          <div class="grid-x grid-margin-x">
            <div class="cell small-12 medium-6">
              <label>
                Police Grid:
                <input v-model="police_grid" />
              </label>
            </div>

            <div class="cell small-12 medium-6">
              <label>
                Neighborhood #:
                <input v-model="neighborhood_number" />
              </label>
            </div>
          </div>

          <div class="grid-x grid-margin-x">
            <div class="cell small-12">
              <label>
                Block:
                <input v-model="block" />
              </label>
            </div>
          </div>

          <button type="submit" class="button primary expanded">
            Submit Incident
          </button>
        </form>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showForm: false,
      case_number: "",
      date: "",
      time: "",
      code: "",
      incident: "",
      police_grid: "",
      neighborhood_number: "",
      block: "",
      errorMessage: "",
    };
  },

  methods: {
    async submitIncident() {
      if (
        !this.case_number ||
        !this.date ||
        !this.time ||
        !this.code ||
        !this.incident ||
        !this.police_grid ||
        !this.neighborhood_number ||
        !this.block
      ) {
        this.errorMessage = "Please fill out ALL fields.";
        return;
      }

      this.errorMessage = "";

      const newIncident = {
        case_number: this.case_number,
        date: this.date,
        time: this.time,
        code: this.code,
        incident: this.incident,
        police_grid: this.police_grid,
        neighborhood_number: this.neighborhood_number,
        block: this.block,
      };

      try {
        const res = await fetch("http://localhost:8000/new-incident", {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(newIncident),
        });

        if (!res.ok) {
          this.errorMessage = "Server error. Could not submit.";
          return;
        }

        alert("Incident added successfully!");

        this.case_number = "";
        this.date = "";
        this.time = "";
        this.code = "";
        this.incident = "";
        this.police_grid = "";
        this.neighborhood_number = "";
        this.block = "";
      } catch (err) {
        this.errorMessage = "Network error.";
      }
    },
  },
};
</script>

<style scoped>
.form-wrapper {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.form-container {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 25px;
  width: 550px;
  background: white;
}

h2 {
  text-align: center;
  margin-bottom: 15px;
}

.error {
  color: red;
  margin-bottom: 10px;
  text-align: center;
}

.form-row {
  display: flex;
  gap: 15px;
  margin-bottom: 12px;
}

.form-row.full {
  flex-direction: column;
}

.form-field {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.form-field.full input {
  width: 100%;
}

label {
  margin-bottom: 4px;
  font-size: 14px;
}

input {
  padding: 7px;
  border: 1px solid #aaa;
  border-radius: 4px;
  width: 100%;
}

.submit-btn {
  margin-top: 15px;
  width: 100%;
  padding: 10px;
  font-size: 16px;
  background: #4677f5;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.submit-btn:hover {
  background: #325fd1;
}
</style>
