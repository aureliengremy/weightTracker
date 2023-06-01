<script setup>
import { ref, shallowRef, computed, watch, nextTick, onUpdated } from 'vue';
import Chart from 'chart.js/auto';

const weights = ref([])

const weightChartEl = ref(null)

const weightChart = shallowRef(null)

const weightInput = ref('')

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime()
  })
}

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

watch(weights, newWeights => {
  const thisWeight = [...newWeights]

  nextTick(() => {
    console.log(weightChartEl);

    if (weightChart.value) {
      weightChart.value.data.labels = thisWeight
        .sort((a, b) => a.date - b.date)
        .map(weight => new Date(weight.date).toLocaleDateString())
        .slice(-7)
      weightChart.value.data.datasets[0].data = thisWeight
        .sort((a, b) => a.date - b.date)
        .map(weight => weight.weight)
        .slice(-7)

      return weightChart.value.update()


    }

    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: thisWeight
          .sort((a, b) => a.date - b.date)
          .map(weight => new Date(weight.date).toLocaleDateString()),
        datasets: [{
          label: 'Weight',
          data: thisWeight
            .sort((a, b) => a.date - b.date)
            .map(weight => weight.weight),
          backgroundColor: 'rgba(224, 122, 95, 0.2)',
          borderColor: 'rgba(255, 255, 255)',
          borderWidth: 1,
          fill: true
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
      }
    })
  })

}, { deep: true })

// onUpdated(() => {
//   console.log(currentWeight.value)
//   console.log(weights.value)
//   console.log(weightChartEl.value)
// })

</script>

<template>
  <div class="container">
    <h1>Weight Tracker</h1>
    <div class="current">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 340 340">
        <circle cx="170" cy="170" r="160" stroke="#E2007C" />
        <circle cx="170" cy="170" r="135" stroke="#404041" />
      </svg>
      <div class="current-weight">
        <h2>{{ currentWeight.weight }} kg</h2>
        <p class="small-text">current weight</p>
      </div>
    </div>
    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput">
      <!-- <input type="range" v-model="duration" min="1" max="30000"> -->
      <input type="submit" value="Add weight">
    </form>
  
    <div class="weight-chart" v-if="weights && weights.length > 0">
      <h3>Last 7 days</h3>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
    </div>
    <div v-if="weights && weights.length > 0" class="weight-history">
      <h3>Weight History</h3>
      <div class="weight-list" v-for="weight in weights">
        <div class="item-weight">{{ weight.weight }}kg</div>
        <div class="item-date">{{ new Date(weight.date).toLocaleDateString() }}</div>
      </div>
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'montserrat', sans-serif;
}

body {

  background-image: linear-gradient(to right top, #3d405b, #7c6276, #ae8c8e, #d4bdaf, #f4f1de);
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  /* height: 100vh; */
  /* width: 100vw; */
  padding: 2rem;
  background-color: #ffffff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

h1 {
  font-size: 2rem;
  margin-bottom: 2rem;
}

.current {
  position: relative;
  max-width: 15rem;
  width: 100%;
  height: auto;
  stroke-linecap: round;
}

circle {
  fill: none;
  stroke-width: 3.5;
  animation-name: circleAnim;
  animation-duration: 12s;
  animation-iteration-count: infinite;
  animation-timing-function: ease-in-out;
  transform-origin: 170px 170px;
  will-change: transform;
}

circle:nth-child(1) {
  stroke-dasharray: 750px;
  animation-delay: 1.2s;
}

circle:nth-child(2) {
  stroke-dasharray: 500px;
}

@keyframes circleAnim {
  50% {
    transform: rotate(360deg);
  }
}

.current-weight {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}
.small-text {
  padding-top: 5px;
  font-size: 0.8rem;
  opacity: 0.6;
}

h3 {
  font-size: 1.2rem;
  margin-bottom: 0.6rem;
}

.weight-chart {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  margin: 2rem 0;
}

.weight-history {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.weight-list {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
  width: 100%;
  margin-top: 0.4rem;
}
.item-weight {
  font-size: 1rem;
  font-weight: bold;
}


@media only screen and (min-width: 600px) {
  /* For tablets: */
}

@media only screen and (min-width: 768px) {
  /* For desktop: */
}</style>
