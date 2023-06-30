<template>
  <div>
    <h2>Remaining time...</h2>
    <p class="timer">{{ countdownRef.hours }}:{{ countdownRef.minutes }}:{{ countdownRef.seconds }}</p>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';

const targetHour = 14;
const targetMinute = 30;

const countdown = {
  hours: 0,
  minutes: 0,
  seconds: 0
};

const countdownRef = ref(countdown);

const startCountdown = () => {
  const alarmSound = new Audio('@/assets/alarm.mp3');

  const checkAlarm = () => {
    const now = new Date();
    const targetTime = new Date();
    targetTime.setHours(targetHour, targetMinute, 0, 0);

    // Verificar si la hora objetivo ya ha pasado
    if (targetTime < now) {
      targetTime.setDate(targetTime.getDate() + 1); // Aumentar la fecha en 1 día
    }

    const diff = targetTime - now;

    countdownRef.value.seconds = String(Math.floor(diff / 1000) % 60).padStart(2, '0');
    countdownRef.value.minutes = String(Math.floor(diff / 1000 / 60) % 60).padStart(2, '0');
    countdownRef.value.hours = String(Math.floor(diff / 1000 / 60 / 60) % 24).padStart(2, '0');

    // Update document title with the timer text
    document.title = `${countdownRef.value.hours}:${countdownRef.value.minutes}`;

    // Check if the countdown reaches zero
    if (diff <= 0) {
      // Play the alarm sound
      alarmSound.play();
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
</script>

<style scoped>
.timer {
  font-size: 12rem;
  font-weight: 800;
}

div {
  background-color: rgb(23, 23, 23);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
  font-family: sans-serif;
  background-image: linear-gradient(to top, rgba(0, 0, 0, 1), rgba(0, 0, 0, 0)), url(./assets/bg.jpg);
  background-size: cover;
  color: white;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
}
</style>
