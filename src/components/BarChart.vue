<template>
  <h1>Bar Chart for Population of the world</h1>
  <div class="bar-container">
    <!-- Bar Chart on the left -->
    <div ref="chart" class="bar-chart"></div>
    
    <!-- Dropdown on the right -->
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

// Chart reference
const chart = ref(null);

// Initial render
onMounted(async () => {
  updateChart();  // Render the initial chart with default selected countries
});
</script>

  

  
  <!-- <style>
  .tooltip {
    position: absolute;
    text-align: center;
    padding: 8px;
    font: 12px sans-serif;
    background: lightsteelblue;
    border: 0px;
    border-radius: 8px;
    pointer-events: none;
  }
  .dropdown {
  position: relative;
  display: inline-block;
}
.dropdown-btn {
  padding: 10px;
  border: 1px solid #ccc;
  background-color: rgb(74, 30, 134);
  cursor: pointer;
}
.dropdown-content {
  display: block;
  position: absolute;
  background-color: rgb(34, 54, 209);
  box-shadow: 0px 8px 16px rgba(0, 0, 0, 0.2);
  border: 1px solid #ccc;
  z-index: 1;
  min-width: 200px; /* Set a minimum width for the dropdown */
  max-height: 300px; /* Limit the height */
  overflow-y: auto; /* Add scroll if content exceeds max height */
  padding: 0; /* Remove padding */
}

.dropdown-content label {
  display: flex; /* Use flexbox for alignment */
  align-items: center; /* Center the checkbox and text vertically */
  padding: 5px 10px; /* Reduce padding to align text and checkbox */
  cursor: pointer;
  box-sizing: border-box; /* Include padding in width */
  white-space: nowrap; /* Prevent text from wrapping */
  overflow: hidden; /* Hide overflow text */
  text-overflow: ellipsis; /* Show ellipsis for overflowed text */
}

.dropdown-content input[type="checkbox"] {
  margin-right: 10px; /* Space between checkbox and label */
}
  </style>
   -->