

<template>
  <div>
    <select v-model="selectedFilter" @change="filterData">
      <option value="all">All</option>
      <option value="lastMonth">Last Month</option>
      <option value="lastWeek">Last Week</option>
    </select>
    <LineChart :data="filteredData" />
    <BarChart />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import LineChart from './components/LineChart.vue';
import BarChart from './components/BarChart.vue';

const selectedFilter = ref('all');
const data = ref([
  // Mock data
]);

const filteredData = ref([...data.value]);

const filterData = () => {
  // Add logic to filter data based on selectedFilter
};
</script>

<style scoped lang="scss">
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
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
