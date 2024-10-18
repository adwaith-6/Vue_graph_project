<template>
    <div ref="chart"></div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  import * as d3 from 'd3';
  import stockData from '../assets/stockData.json'
  
  // Function to fetch and format the data
const fetchStockData = () => {


  // Extract the time series data and map it to an array
  const timeSeries = stockData['Time Series (Daily)'];
  const formattedStockData = Object.keys(timeSeries).map(date => ({
    date: new Date(date),
    close: parseFloat(timeSeries[date]['4. close']),
  }));
  console.log('data',formattedStockData)
  
  return formattedStockData;
};
  

  const chart = ref(null);
  const renderChart = async () => {
  const data = fetchStockData();
  console.log('Fetched Data:', data); // Check if data is fetched correctly


  const margin = { top: 20, right: 30, bottom: 30, left: 40 };
  const width = 810 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;

  const svg = d3.select(chart.value)
    .append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`);

  // Use the correct data structure for x and y domains
  const x = d3.scaleTime()
    .domain(d3.extent(data, d => d.date))
    .range([0, width]);

  const y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.close)]) // Access `close` property
    .range([height, 0]);
  

    svg.append('g')
      .attr('transform', `translate(0,${height})`)
      .call(d3.axisBottom(x));

    svg.append('g')
      .call(d3.axisLeft(y));

    const line = d3.line()
      .x(d => x(d.date))
      .y(d => y(d.close)); // Access close instead of value

    svg.append('path')
      .datum(data)
      .attr('fill', 'none')
      .attr('stroke', 'steelblue')
      .attr('stroke-width', 1.5)
      .attr('d', line);

    // Tooltip
    const tooltip = d3.select(chart.value)
      .append('div')
      .style('position', 'absolute')
      .style('visibility', 'hidden')
      .style('background', 'lightsteelblue')
      .style('padding', '5px')
      .style('border-radius', '3px');

    svg.selectAll("dot")
      .data(data)
      .enter()
      .append("circle")
      .attr("r", 2)
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.close)) // Access close instead of value
      .attr("fill", "blue")
      .on("mouseover", (event, d) => {
        tooltip.html(`Date: ${d.date}<br>Value: ${d.close}`) // Access close instead of value
          .style("visibility", "visible");
      })
      .on("mousemove", (event) => {
        tooltip.style("top", (event.pageY - 10) + "px")
          .style("left", (event.pageX + 10) + "px");
      })
      .on("mouseout", () => tooltip.style("visibility", "hidden"));
};

  
  onMounted(() => {
  renderChart();
});
  </script>
  
  <style scoped>
  svg {
    font-family: Arial, sans-serif;
    background: #f9f9f9;
    border: 1px solid #ccc;
  }
  </style>
  