<template>
  <div class="form-wrapper">
    <div class="form-container">
      <h2>Add New Incident</h2>

      <div v-if="errorMessage" class="error">{{ errorMessage }}</div>

      <form @submit.prevent="submitIncident">
        <div class="form-row">
          <div class="form-field">
            <label>Case Number:</label>
            <input v-model="case_number" />
          </div>

          <div class="form-field">
            <label>Date (YYYY-MM-DD):</label>
            <input v-model="date" />
          </div>
        </div>

        <div class="form-row">
          <div class="form-field">
            <label>Time (HH:MM:SS):</label>
            <input v-model="time" />
          </div>

          <div class="form-field">
            <label>Code:</label>
            <input v-model="code" />
          </div>
        </div>

        <div class="form-row full">
          <div class="form-field full">
            <label>Incident Description:</label>
            <input v-model="incident" />
          </div>
        </div>

        <div class="form-row">
          <div class="form-field">
            <label>Police Grid:</label>
            <input v-model="police_grid" />
          </div>

          <div class="form-field">
            <label>Neighborhood #:</label>
            <input v-model="neighborhood_number" />
          </div>
        </div>

        <div class="form-row full">
          <div class="form-field full">
            <label>Block:</label>
            <input v-model="block" />
          </div>
        </div>

        <button type="submit" class="submit-btn">Submit Incident</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
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
