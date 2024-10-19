<template>
    <div>
        <!-- Date Range Input Fields -->
         <h1>Line Chart for Daily Apple Stocks</h1>
        <div class="date-filters">
            <label for="start-date">Start Date:</label>
            <input type="date" v-model="startDate" id="start-date" />

            <label for="end-date">End Date:</label>
            <input type="date" v-model="endDate" id="end-date" />

            <button @click="filterData">Filter</button>
        </div>

        <!-- Chart Container -->
        <div ref="chart" style="width: 100%; height: 400px;"></div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';
import stockData from '../assets/stockData.json';

const chart = ref(null);

// Reactive state for date range filters
const startDate = ref(null);
const endDate = ref(null);
const filteredData = ref([]); // Store filtered stock data

// Fetch and format the data only once
const allStockData = ref([]); // Store all stock data for filtering

const fetchStockData = () => {
    const timeSeries = stockData['Time Series (Daily)'];
    return Object.keys(timeSeries).map(date => ({
        date: new Date(date),
        close: parseFloat(timeSeries[date]['4. close']),
    }));
};

// Filter data based on selected date range
const filterData = () => {
    if (startDate.value && endDate.value) {
        const start = new Date(startDate.value);
        const end = new Date(endDate.value);
        filteredData.value = allStockData.value.filter(d => d.date >= start && d.date <= end);
    } else {
        filteredData.value = allStockData.value; // Use all data if no date range is selected
    }
    renderChart(); // Rerender the chart with the filtered data
};

// Render the D3 chart
const renderChart = () => {
    const data = filteredData.value.length ? filteredData.value : allStockData.value;
    const margin = { top: 20, right: 30, bottom: 30, left: 40 };
    const width = 810 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    d3.select(chart.value).selectAll('*').remove();

    const svg = d3.select(chart.value)
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

    const x = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
        .range([0, width])
        .clamp(true);

    const y = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.close) * 1.1])
        .range([height, 0])
        .clamp(true);

    svg.append('clipPath')
        .attr('id', 'clip')
        .append('rect')
        .attr('width', width)
        .attr('height', height);

    const chartArea = svg.append('g')
        .attr('clip-path', 'url(#clip)');

    const xAxis = svg.append('g')
        .attr('class', 'x-axis')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x));

    const yAxis = svg.append('g')
        .attr('class', 'y-axis')
        .call(d3.axisLeft(y));

    const lineGenerator = d3.line()
        .x(d => x(d.date))
        .y(d => y(d.close));

    const linePath = chartArea.append('path')
        .datum(data)
        .attr('fill', 'none')
        .attr('stroke', 'steelblue')
        .attr('stroke-width', 1.5)
        .attr('d', lineGenerator);

    const tooltip = d3.select(chart.value)
        .append('div')
        .style('position', 'absolute')
        .style('visibility', 'hidden')
        .style('background', 'lightsteelblue')
        .style('padding', '5px')
        .style('border-radius', '3px');

    chartArea.selectAll("circle")
        .data(data)
        .enter()
        .append("circle")
        .attr("r", 2)
        .attr("cx", d => x(d.date))
        .attr("cy", d => y(d.close))
        .attr("fill", "blue")
        .on("mouseover", (event, d) => {
            tooltip.html(`Date: ${d.date.toLocaleDateString()}<br>Value: ${d.close}`)
                .style("visibility", "visible");
        })
        .on("mousemove", (event) => {
            tooltip.style("top", (event.pageY - 10) + "px")
                .style("left", (event.pageX + 10) + "px");
        })
        .on("mouseout", () => tooltip.style("visibility", "hidden"));

        const zoomBehavior = d3.zoom()
    .scaleExtent([1, 10])
    .translateExtent([[0, 0], [width, height]])
    .extent([[0, 0], [width, height]])
    .on("zoom", (event) => {
        const new_xScale = event.transform.rescaleX(x);
        const new_yScale = event.transform.rescaleY(y);

        // Update axes
        xAxis.call(d3.axisBottom(new_xScale));
        yAxis.call(d3.axisLeft(new_yScale));

        // Update line and circles with both x and y scaling
        linePath.attr("d", lineGenerator.x(d => new_xScale(d.date)).y(d => new_yScale(d.close)));
        chartArea.selectAll("circle")
            .attr("cx", d => new_xScale(d.date))
            .attr("cy", d => new_yScale(d.close));
    });


    // Apply zoom behavior
    d3.select(chart.value).select('svg').call(zoomBehavior);
};

// Watch the date range input and re-render chart on change
onMounted(() => {
    allStockData.value = fetchStockData();
    filteredData.value = allStockData.value;
    renderChart();
});
</script>

<style scoped>
svg {
    font-family: Arial, sans-serif;
    background: #f9f9f9;
    border: 1px solid #ccc;
}
.date-filters {
    margin-bottom: 20px;
}
.date-filters input {
    margin-right: 10px;
}
.tooltip {
    position: absolute;
    visibility: hidden;
    background-color: lightgrey;
    padding: 5px;
    border-radius: 5px;
}
</style>
