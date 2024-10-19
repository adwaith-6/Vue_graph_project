<template>
  <div>
    <NavBar />
    <LineChart />
    <BarChart />
  </div>
</template>

<script setup>
import LineChart from './components/LineChart.vue';
import BarChart from './components/BarChart.vue';
import NavBar from './components/NavBar.vue';

</script>

<style scoped lang="scss">
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 5rem;
}

select {
  margin-bottom: 20px;
}

@media screen and (max-width: 768px) {
  #app {
    flex-direction: column;
  }
}
</style>
