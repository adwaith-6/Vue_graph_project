<template>
  <h1>Bar Chart for Population of the world</h1>
  <div class="bar-container">
    <div ref="chart" class="bar-chart"></div>
    <div class="dropdown-container">
      <div class="dropdown">
        <label>Select Countries:</label>
        <button class="dropdown-btn" @click="toggleDropdown">{{ dropdownLabel }}</button>
        <div v-if="isDropdownOpen" class="dropdown-content">
          <label class="dropdown-label" v-for="country in orderedCountries" :key="country">
            <input
              type="checkbox"
              :value="country"
              v-model="selectedCountries"
              @change="updateChart"
            />
            {{ country }}
          </label>
        </div>
      </div>
    </div>
  </div>
</template>


  
  
  <script setup>
import { ref, onMounted, computed } from 'vue';
import * as d3 from 'd3';
import population from '../assets/population.json';

// Initialize countries
const allCountries = ref(population.map(country => country.name.common));

// Selected countries (initially showing top 10)
const selectedCountries = ref([
  'China', 'India', 'United States', 'Indonesia', 'Pakistan', 
  'Brazil', 'Nigeria', 'Bangladesh', 'Russia', 'Mexico'
]);

// Dropdown open/close state
const isDropdownOpen = ref(false);
const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

// Reordered countries for dropdown (selected first, others in alphabetical order)
const orderedCountries = computed(() => {
  const selected = allCountries.value.filter(country => selectedCountries.value.includes(country));
  const unselected = allCountries.value.filter(country => !selectedCountries.value.includes(country))
                                       .sort(); // Sort unselected alphabetically
  return [...selected, ...unselected];
});

// Fetch population data from JSON
const fetchPopulationData = async () => {
  try {
    const data = population;
    return data.map(country => ({
      name: country.name.common,
      population: country.population,
    }));
  } catch (error) {
    console.error('Error fetching population data:', error);
    return [];
  }
};

// Function to update chart based on selected countries
const updateChart = async () => {
  const data = await fetchPopulationData();
  const filteredData = data.filter(country => selectedCountries.value.includes(country.name));

  // Clear previous chart before rendering new data
  d3.select(chart.value).selectAll("*").remove();

  renderChart(filteredData);
};

// Function to render chart
const renderChart = (data) => {
  const margin = { top: 20, right: 30, bottom: 60, left: 50 };
  const width = 900 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;

  const svg = d3.select(chart.value)
    .append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left}, ${margin.top})`);
  
  const tooltip = d3.select("body").append("div")
    .attr("class", "tooltip")
    .style("opacity", 0);  

  // X axis
  const x = d3.scaleBand()
    .domain(data.map(d => d.name))
    .range([0, width])
    .padding(0.2);

  svg.append('g')
    .attr('transform', `translate(0,${height})`)
    .call(d3.axisBottom(x))
    .selectAll("text")
    .attr("transform", "rotate(-40)")
    .style("text-anchor", "end");

  // Y axis
  const y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.population)])
    .range([height, 0]);

  const yAxis = d3.axisLeft(y)
    .ticks(10) 
    .tickFormat(d => {
      if (d >= 1e9) return `${(d / 1e9).toFixed(1)}B`;
      else if (d >= 1e6) return `${(d / 1e6).toFixed(1)}M`;
      else if (d >= 1e3) return `${(d / 1e3).toFixed(1)}K`;
      return d;
    });

  svg.append('g')
    .call(yAxis);
  
  // Bars
  svg.selectAll('.bar')
  .data(data)
  .enter()
  .append('rect')
  .attr('class', 'bar')
  .attr('x', d => x(d.name))
  .attr('y', d => {
    // If the population is very small, set y position at the bottom (height of the chart)
    return y(d.population) < height - 5 ? y(d.population) : height - 5;
  })
  .attr('width', x.bandwidth())
  .attr('height', d => Math.max(5, height - y(d.population)))  // Minimum height for visibility
  .attr('fill', '#091057')
  .on('mouseover', function(event, d) {
    tooltip.transition()
      .duration(200)
      .style("opacity", .9);
    tooltip.html(`Country: ${d.name}<br/>Population: ${d.population}`)
      .style("left", (event.pageX + 5) + "px")
      .style("top", (event.pageY - 28) + "px");
  })
  .on('mouseout', function(d) {
    tooltip.transition()
      .duration(500)
      .style("opacity", 0);
  });


};

// Label for the dropdown button
const dropdownLabel = computed(() => {
  return selectedCountries.value.length ? `${selectedCountries.value.length} Selected` : "Select Countries";
});

const chart = ref(null);

// Initial render
onMounted(async () => {
  updateChart();  // Render the initial chart with default selected countries
});
</script>