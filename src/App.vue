<template>
  <div class="template">
    <div class="card p-4" v-if="showSelectTime">
      <h4 class="text-center mb-3">Select time</h4>
      <div class="d-flex justify-content-between align-items-center gap-2 mb-3">
        <p class="mb-0">From now:</p>
        <div class="d-flex gap-1">
          <button class="btn btn-outline-secondary" @click="addMinutes(30)">
            +30 m
          </button>
          <button class="btn btn-outline-secondary" @click="addMinutes(60)">
            +60 m
          </button>
          <button class="btn btn-outline-secondary" @click="addMinutes(90)">
            +90 m
          </button>
        </div>
      </div>
      <div class="time d-flex justify-content-between mb-3">
        <input
          type="number"
          min="0"
          max="23"
          v-model="targetHour"
          class="no-spinners"
          maxlength="2"
        />
        <span>:</span>
        <input
          type="number"
          min="0"
          max="59"
          v-model="targetMinute"
          class="no-spinners"
          maxlength="2"
        />
      </div>
      <button class="btn btn-outline-primary" @click="handleInput">
        Start
      </button>
    </div>
    <div class="timer-div" v-else>
      <h2>Remaining time...</h2>
      <p class="timer">
        {{ countdownRef.hours }}:{{ countdownRef.minutes }}:{{
          countdownRef.seconds
        }}
      </p>
      <h4 class="text-end">
        Target Time: {{ targetHour }}:{{ targetMinute }} hs
      </h4>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";

const defaultTargetTime = new Date();
defaultTargetTime.setMinutes(defaultTargetTime.getMinutes() + 90);

const targetHour = ref(
  localStorage.getItem("targetHour") || defaultTargetTime.getHours()
);
const targetMinute = ref(
  localStorage.getItem("targetMinute") || defaultTargetTime.getMinutes()
);

const showSelectTime = ref(
  localStorage.getItem("showSelectTime") === "true" || true
);

const countdown = {
  hours: 0,
  minutes: 0,
  seconds: 0,
};

const countdownRef = ref(countdown);

const startCountdown = () => {
  const checkAlarm = () => {
    const now = new Date();
    const targetTime = new Date();
    targetTime.setHours(targetHour.value, targetMinute.value, 0, 0);

    // Verificar si la hora objetivo ya ha pasado
    if (targetTime < now) {
      targetTime.setDate(targetTime.getDate() + 1); // Aumentar la fecha en 1 día
    }

    const diff = targetTime - now;

    countdownRef.value.seconds = String(Math.floor(diff / 1000) % 60).padStart(
      2,
      "0"
    );
    countdownRef.value.minutes = String(
      Math.floor(diff / 1000 / 60) % 60
    ).padStart(2, "0");
    countdownRef.value.hours = String(
      Math.floor(diff / 1000 / 60 / 60) % 24
    ).padStart(2, "0");

    // Update document title with the timer text
    document.title = `${countdownRef.value.hours}:${countdownRef.value.minutes}:${countdownRef.value.seconds} - Remaining time`;

    // Check if the countdown reaches zero
    if (diff <= 0) {
      clearInterval(interval);
    }
  };

  // Run the initial check immediately
  checkAlarm();

  // Run the check every second
  const interval = setInterval(checkAlarm, 1000);
};

onMounted(() => {
  startCountdown();
});

const handleInput = () => {
  showSelectTime.value = !showSelectTime.value;
  localStorage.setItem("showSelectTime", showSelectTime.value);
};

// Observa los cambios en targetHour, targetMinute y showSelectTime y almacena los valores en localStorage
watch([targetHour, targetMinute, showSelectTime], () => {
  localStorage.setItem("targetHour", targetHour.value);
  localStorage.setItem("targetMinute", targetMinute.value);
  localStorage.setItem("showSelectTime", showSelectTime.value);
});

// Verificar el valor inicial de showSelectTime al cargar la página
if (showSelectTime.value === false) {
  showSelectTime.value = true;
}

const addMinutes = (minutes) => {
  const now = new Date();
  now.setMinutes(now.getMinutes() + minutes);
  targetHour.value = now.getHours();
  targetMinute.value = now.getMinutes();
};
</script>

<style scoped>
.timer {
  font-size: 12rem;
  font-weight: 800;
}

.template {
  background-color: rgb(23, 23, 23);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
  font-family: sans-serif;
  background-image: linear-gradient(to top, rgba(0, 0, 0, 1), rgba(0, 0, 0, 0)),
    url(./assets/bg.jpg);
  background-size: cover;
  color: white;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
}

.card {
  background-color: rgba(0, 0, 0, 0.6);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
  color: white;
}

input {
  width: 160px;
  background-color: rgba(0, 0, 0, 0.6);
  border: none;
  color: white;
  text-align: center;
  padding: 8px;
  font-weight: 800;
  border-radius: 10px;
}

input:focus {
  outline: 1px solid rgb(31, 128, 255);
}

.no-spinners::-webkit-inner-spin-button,
.no-spinners::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* .no-spinners {
  -moz-appearance: textfield;
} */
</style>
