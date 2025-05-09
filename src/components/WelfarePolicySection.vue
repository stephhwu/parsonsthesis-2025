<template>
  <div class="welfare-policy-section" ref="welfareSection">
    <div class="main-content-container">
      <div class="chart-container">
        <div class="chart-controls">
          <div class="sort-control">
            <label for="sort-select">Sort by compliance rate:</label>
            <select id="sort-select" v-model="sortOption" @change="refreshChart">
              <option value="asc">Lowest compliance first</option>
              <option value="desc">Highest compliance first</option>
            </select>
          </div>
        </div>
        
        <div ref="bubbleChart" class="heatmap-chart"></div>
        
        <div class="chart-legend">
          <div class="legend-title">Policy Compliance</div>
          <div class="legend-items">
            <div class="legend-item compliant">
              <div class="legend-square"></div>
              <div class="legend-label">Compliant</div>
            </div>
            <div class="legend-item non-compliant">
              <div class="legend-square"></div>
              <div class="legend-label">Non-Compliant</div>
            </div>
          </div>
        </div>
        <div class="source">
  <a href="https://www.gao.gov/assets/gao-23-104489.pdf" target="_blank" rel="noopener noreferrer">
    Source: GAO analysis of agency data | GAO-23-104489
  </a>
</div>
      </div>

      <div class="statistics-container">
<div class="title-container">
  <h2 class="main-title">Working<br>Like a <span class="highlighted-text">Dog:</span></h2>
</div>
<div class="welfare-stats-description">
  As of 2022, over <span style="font-family: 'ivypresto-headline', serif; font-style: italic; color: black; font-weight: normal; font-size: 20px;">5,600</span> dogs were working for the U.S. federal government. Yet the Government Accountability Office report found major welfare gaps.
</div>
        <div class="welfare-stats-description2">
          In response, Congress passed the Working Dog Health and Welfare Act, requiring agencies to provide better veterinary care, rest, and oversight.
        </div>
        <img class="dog-image" src="@/assets/images/working.png" alt="Working dog with laptop" />
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as d3 from 'd3';
import gaoData from '/Users/stephaniewu/Downloads/thesis-2025/gao.json';

export default {
  name: 'WelfarePolicySection',
  setup() {
    const bubbleChart = ref(null);
    const welfareSection = ref(null);
    const sortOption = ref('asc'); // Default to lowest compliance first
    let tooltip = null;
    let observer = null;
    let chartInitialized = false;
    
    onMounted(() => {
      // Create intersection observer to detect when section scrolls into view
      observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !chartInitialized) {
            // Section is now visible, trigger chart creation with animation
            chartInitialized = true;
            createHeatmap();
          }
        });
      }, { threshold: 0.2 }); // Trigger when 20% of element is visible
      
      // Start observing the section
      if (bubbleChart.value) {
        observer.observe(bubbleChart.value.parentElement.parentElement);
      }
    });
    
    onBeforeUnmount(() => {
      if (tooltip) {
        tooltip.remove();
      }
      if (observer) {
        observer.disconnect();
      }
    });
    
    function refreshChart() {
      if (tooltip) {
        tooltip.transition().duration(0).style("opacity", 0); // Immediately hide tooltip
        // tooltip.remove();  // Don't remove completely, just hide it
        // tooltip = null;    // Don't set to null - keep the reference
      }
      
      // Clear any active row highlights
      if (bubbleChart.value) {
        const svg = d3.select(bubbleChart.value).select("svg");
        if (!svg.empty()) {
          svg.select("g").selectAll(".row-highlight")
            .attr("stroke", "none")
            .style("opacity", 0);
        }
      }
      
      createHeatmap();
    }
    
    function createHeatmap() {
  if (!bubbleChart.value) return;
  
  const { departments, issues, complianceMatrix, implementationPercentages, dogCounts } = processGaoDataForHeatmap(gaoData);
  let sortedIndices = [...Array(departments.length).keys()];
  
  if (sortOption.value === 'asc') {
    sortedIndices.sort((a, b) => implementationPercentages[a] - implementationPercentages[b]);
  } else if (sortOption.value === 'desc') {
    sortedIndices.sort((a, b) => implementationPercentages[b] - implementationPercentages[a]);
  }
  
  const sortedDepts = sortedIndices.map(i => departments[i]);
  const sortedMatrix = sortedIndices.map(i => complianceMatrix[i]);
  const sortedPercentages = sortedIndices.map(i => implementationPercentages[i]);
  const sortedDogCounts = sortedIndices.map(i => dogCounts[i]);
  
  const margin = { top: 100, right: 50, bottom: 110, left: 50 };
  const width = 1100 - margin.left - margin.right; 
  const height = Math.max(300, departments.length * 11) - margin.top - margin.bottom;
  
  // Create or select SVG
  let svg = d3.select(bubbleChart.value).select("svg");
  let g;
  
  if (svg.empty()) {
    // First time creating the chart
    d3.select(bubbleChart.value).selectAll("*").remove();
    svg = d3.select(bubbleChart.value)
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom);
    
    g = svg.append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);
  } else {
    // Reuse existing SVG
    g = svg.select("g");
  }
  
  const colorScale = d3.scaleOrdinal()
    .domain([true, false])
    .range(["#81C9E3", "#FF0000"]);
    
  // Create tooltip if it doesn't exist
  if (!tooltip) {
    tooltip = d3.select("body").append("div")
      .attr("class", "welfare-dog-tooltip")
      .style("opacity", 0)
      .style("position", "absolute")
      .style("background-color", "white")
      .style("border", "1px solid #ddd")
      .style("border-radius", "20px")
      .style("padding", "20px")
      .style("box-shadow", "0 4px 12px rgba(0,0,0,0.08)")
      .style("pointer-events", "none")
      .style("font-size", "14px")
      .style("min-width", "300px")
      .style("max-width", "300px")
      .style("z-index", "10000");
  }
  
  const xScale = d3.scaleBand()
    .domain(issues)
    .range([0, width])
    .padding(0.05);
    
  const yScale = d3.scaleBand()
    .domain(sortedDepts)
    .range([0, height])
    .padding(0.1);
  
  // Update axes
  let xAxisGroup = g.select(".x-axis");
  if (xAxisGroup.empty()) {
    xAxisGroup = g.append("g")
      .attr("class", "x-axis")
      .style("font-size", "10px")
      .attr("transform", `translate(0,0)`);
  }
  
  xAxisGroup.call(d3.axisTop(xScale))
    .selectAll("text")
    .attr("transform", "rotate(-25)")
    .style("text-anchor", "start")
    .style("font-family", "'neue-haas-grotesk-display', sans-serif") 
    .style("font-weight", "400") 
    .attr("dx", "0.8em")
    .attr("dy", "-0.5em");

  // Remove x-axis domain line
  xAxisGroup.select(".domain").remove();
  
  let yAxisGroup = g.select(".y-axis");
  if (yAxisGroup.empty()) {
    yAxisGroup = g.append("g")
      .attr("class", "y-axis")
      .style("font-size", "10px");
  }
  
  yAxisGroup.transition().duration(750)
    .call(d3.axisLeft(yScale).tickSize(0).tickFormat(""));

  // Remove y-axis domain line
  yAxisGroup.select(".domain").remove();
  
  // First remove any existing labels to prevent duplicates
  g.selectAll(".y-axis-label-main, .y-axis-label-sub").remove();

  // Then add the new labels
  g.append("text")
    .attr("class", "y-axis-label-main")
    .attr("transform", "rotate(-90)")
    .attr("y", -margin.left -5)  // Position to the left of the chart
    .attr("x", -height / 2)        // Center vertically
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .style("font-family", "Roca, sans-serif")
    .style("font-size", "16px")
    .style("font-weight", "700")
    .text("Department");

  // Add second line (hover instructions) with different styling
  g.append("text")
    .attr("class", "y-axis-label-sub")
    .attr("transform", "rotate(-90)")
    .attr("y", -margin.left -4)  // Same position as main label
    .attr("x", -height / 1.95 + 5)   // Offset slightly down
    .attr("dy", "3em")            // Move down to create second line
    .style("text-anchor", "middle")
    .style("font-family", "neue-haas-grotesk-display, sans-serif")
    .style("font-size", "12px")   // Smaller font
    .style("font-weight", "400")  // Normal weight
    .style("fill", "#888888")     // Grey color
    .text("(hover to explore details)");
  
  // Prepare data for cells
  let cellData = [];
  sortedDepts.forEach((dept, i) => {
    issues.forEach((issue, j) => {
      cellData.push({
        dept,
        issue,
        compliant: sortedMatrix[i][j],
        percentage: sortedPercentages[i],
        dogCount: sortedDogCounts[i],
        id: `${dept}-${issue}` // Unique identifier
      });
    });
  });
  
  // IMPORTANT: Remove old event listeners before binding new ones
  g.selectAll(".cell")
    .on("mouseover", null)
    .on("mouseout", null)
    .on("mousemove", null);
  
  // Create or update cells with animation
  const cells = g.selectAll(".cell")
    .data(cellData, d => d.id);
  
  // Remove cells that are no longer needed
  cells.exit()
    .transition()
    .duration(200)
    .style("opacity", 0)
    .remove();
  
  // Add new cells
  const enterCells = cells.enter()
    .append("rect")
    .attr("class", "cell")
    .attr("x", d => xScale(d.issue))
    .attr("y", height) // Start from bottom for new cells
    .attr("width", xScale.bandwidth())
    .attr("height", yScale.bandwidth())
    .attr("fill", d => colorScale(d.compliant))
    .attr("stroke", "white")
    .attr("stroke-width", 0.5)
    .style("opacity", 0);
  
  // Update or create row highlight rectangles
  g.selectAll(".row-highlight").remove(); // Remove old highlights completely
  
  const rowHighlights = g.selectAll(".row-highlight")
    .data(sortedDepts)
    .enter()
    .append("rect")
    .attr("class", "row-highlight")
    .attr("x", 0)
    .attr("y", d => yScale(d))
    .attr("width", width)
    .attr("height", yScale.bandwidth())
    .attr("fill", "none")
    .attr("stroke", "none")
    .attr("stroke-width", 0) // Reset stroke width
    .attr("pointer-events", "none") // Make sure it doesn't interfere with mouse events
    .style("opacity", 0);
  
  // Merge and transition all cells to their new positions
  const allCells = enterCells.merge(cells);
  
  // Apply transitions to all cells
  allCells.transition()
    .duration(750)
    .delay((d, i) => i * 3) // Staggered animation
    .attr("x", d => xScale(d.issue))
    .attr("y", d => yScale(d.dept))
    .style("opacity", 1)
    .on("end", function() {
      // Only attach event handlers after all transitions are complete
      // This prevents event handlers from firing during transition
      if (this === allCells.nodes()[allCells.size() - 1]) {
        attachEventHandlers();
      }
    });
  
  // Define the event handler function to keep it consistent
  function attachEventHandlers() {
    // Attach event listeners to cells AFTER transitions complete
    allCells
      .on("mouseover", function(event, d) {
        // Find and highlight the row background
        g.selectAll(".row-highlight")
          .filter(dept => dept === d.dept)
          .attr("stroke", "black")
          .attr("stroke-width", 2)
          .style("opacity", 1);

        // Find detection roles from the original data
        const departmentData = gaoData.find(item => item.department === d.dept);
        const detectionRoles = departmentData?.detection || [];
        
        tooltip.transition()
          .duration(100)
          .style("opacity", 0.9);
        
        // Show a single tooltip with department information
        tooltip.html(`
  <div style="font-family:'ivypresto-headline',serif; font-style:italic; color:#FF0000; font-size:20px; margin-bottom:10px;">
    ${d.dept}
  </div>
  <div style="font-family:'neue-haas-grotesk-display',sans-serif; font-size:14px; margin-top:5px;">
    <b>Compliance Rate:</b> ${d.percentage.toFixed(0)}%<br>
    <b>Working Dogs:</b> ${d.dogCount}<br>
    <b>Detection Roles:</b> ${detectionRoles.join(", ") || "None"}
  </div>
`)
          .style("left", (event.pageX + 15) + "px")
          .style("top", (event.pageY - 15) + "px");
      })
      .on("mousemove", function(event) {
        tooltip
          .style("left", (event.pageX + 15) + "px")
          .style("top", (event.pageY - 15) + "px");
      })
      .on("mouseout", function(event, d) {
        // Hide the row highlight
        g.selectAll(".row-highlight")
          .filter(dept => dept === d.dept)
          .attr("stroke", "none")
          .attr("stroke-width", 0)
          .style("opacity", 0);
        
        tooltip.transition()
          .duration(100)
          .style("opacity", 0);
      });
  }
}

    function processGaoDataForHeatmap(data) {
      const validData = data.filter(item => item.issues && item.department);
      const departments = validData.map(item => item.department);
      let allIssues = [];
      validData.forEach(item => {
        allIssues = [...allIssues, ...Object.keys(item.issues)];
      });
      const issues = Array.from(new Set(allIssues)).sort();
      const complianceMatrix = [];
      const implementationPercentages = [];
      const dogCounts = [];
      departments.forEach(dept => {
        const deptData = validData.find(item => item.department === dept);
        const row = [];
        issues.forEach(issue => {
          const isCompliant = deptData.issues[issue] === true;
          row.push(isCompliant);
        });
        complianceMatrix.push(row);
        const implementedCount = row.filter(v => v === true).length;
        const percentage = (implementedCount / issues.length) * 100;
        implementationPercentages.push(percentage);
        dogCounts.push(deptData.dogs || 0);
      });
      return {
        departments,
        issues,
        complianceMatrix,
        implementationPercentages,
        dogCounts
      };
    }
    return {
      bubbleChart,
      welfareSection,
      sortOption,
      refreshChart
    };
  }
}
</script>

<style scoped>
/* Update styles for heatmap */
.heatmap-chart {
  width: 100%;
  height: 100%;
  position: relative;
  overflow-y: auto;
}

/* Update legend for heatmap */
.legend-square {
  width: 15px;
  height: 15px;
}

.compliant .legend-square {
  background-color: #81C9E3;
}

.non-compliant .legend-square {
  background-color: red;
}

/* Retain your existing styles */
.welfare-policy-section {
  padding: 3rem 0;
  background-color: #F2EFEB;
  position: relative;
  width: 100%;
  overflow: hidden;
  isolation: isolate; /* Creates a new stacking context */
}

.chart-title {
  font-family: "ivypresto-headline", serif;
  font-weight: normal;
  text-align: center;
  margin-bottom: 2rem;
  font-size: 1.8rem;
}

.main-content-container {
  display: flex;
  flex-direction: row;
  max-width: 1400px;
  margin: 0 auto;
  gap: 2rem;
  position: relative;
}

.chart-container {
  flex: 2; /* Reduce from 3 to give more space to statistics */
  position: relative;
  height: 750px;
}

.bubble-chart {
  width: 100%;
  height: 100%;
  position: relative;
}

.chart-legend {
  position: absolute;
  bottom: 20px;
  left: 50px;
  z-index: 5;
}

.legend-title {
  color: #000;
font-family: Roca;
font-size: 14px;
font-style: normal;
font-weight: 700;
line-height: normal;
margin-top: 20px;
  margin-bottom: 10px;
}

.legend-items {
  color: #000;
font-family: Roca;
font-size: 12px;
font-style: normal;
font-weight: 400;
line-height: normal;
  display: flex;
  gap: 20px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.legend-bubble {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  border: 1px solid white;
}

.contractor .legend-bubble {
  background-color: #85c3d1;
}

.federal .legend-bubble {
  background-color: #f8d76a;
}

.legend-label {
  font-size: 0.8rem;
}


.title-container {
  width: 100%;
}

.main-title {
  font-family: "ivypresto-headline", serif;
  font-weight: normal;
  text-align: right;
  font-size: 4rem;
  color: red;
  line-height: 0.9;
  margin: 0 0 1rem 0;
}

.highlighted-text {
  font-style: italic;
}

.welfare-stats-description {
  font-family: "neue-haas-grotesk-display", sans-serif; /* Move from parent to here */
  font-size: 15px;
  font-style: normal;
  font-weight: 400;
  line-height: 1.4;
  margin-bottom: 0.5rem;
  text-align: right;
  color: #666666; /* Slightly darker gray to match mockup */
}

.welfare-stats-description2 {
  font-family: "neue-haas-grotesk-display", sans-serif; /* Move from parent to here */
  font-size: 15px;
  font-style: normal;
  font-weight: 400;
  line-height: 1.4;
  margin-bottom: 0.5rem;
  text-align: right;
  color: #666666; /* Slightly darker gray to match mockup */
}

.dog-image {
  max-width: 383px;
  height: auto;
  padding: 5px;
  align-self: flex-end; /* Right align the image */
}

.statistics-container {
  width: 450px; /* Increase width */
  min-width: 370px; /* Ensure it doesn't shrink */
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  text-align: right;
  gap: 1rem;
  margin-top: 100px;
  padding-right: 20px;
}

.stats-column {
  flex: 1;
  position: relative;
  text-align: right;
}

.stats-column h3 {
  font-family: "ivypresto-headline", serif;
  font-weight: normal;
  margin-bottom: 1rem;
  text-align: right;
}

.stats-column ol {
  padding-left: 1.5rem;
  margin: 0;
  text-align: right;
}

.stats-column li {
  margin-bottom: 0.5rem;
  text-align: right;
}

.source {
  position: absolute;
  bottom: 60px;
  right: 20px;
  color: #A2A2A2;
  font-family: "Neue Haas Grotesk Display Pro";
  font-size: 12px;
  font-style: normal;
  font-weight: 400;
  line-height: normal;
  padding: 2px 8px;
  border-radius: 6px;
  margin-right: 20px;
}

.source a {
  color: #666666;
  text-decoration: none;
  border-bottom: 1px dotted #999999;
  transition: color 0.2s ease;
}

.source a:hover {
  text-decoration: underline;
}

/* Media queries for responsiveness */
@media (max-width: 1200px) {
  .main-content-container {
    flex-direction: column;
    align-items: center; /* Center the components when stacked */
  }
  
  .chart-container {
    height: 550px;
  }
  
  .statistics-container {
    width: 90%;
    max-width: 400px;
    margin: 2rem auto;
    padding: 0 1rem;
    align-items: center; /* Center on mobile */
    text-align: center;
  }
  
  .main-title,
  .welfare-stats-description,
  .welfare-stats-description2 {
    text-align: center; /* Center text on mobile */
  }
  
  .dog-image {
    align-self: center; /* Center image on mobile */
    max-width: 300px;
  }
}

.chart-controls {
  padding: 10px 0;
}

.sort-control {
  display: flex;
  align-items: center;
  gap: 12px;
  font-family: "Neue Haas Grotesk Display Pro", sans-serif;
}

.sort-control label {
  color: #000;
font-family: Roca;
font-size: 14px;
font-style: normal;
font-weight: 700;
line-height: normal;
margin-left: 50px;
}

.sort-control select {
  appearance: none;
  background-color: #F2EFEB;
  border: 1px solid black;
  border-radius: 20px;
  padding: 8px 36px 8px 12px;
  font-family: "Neue Haas Grotesk Display Pro", sans-serif;
  font-size: 12px;
  color: black;
  cursor: pointer;
  background-image: url("data:image/svg+xml;charset=US-ASCII,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%22292.4%22%20height%3D%22292.4%22%3E%3Cpath%20fill%3D%22%23333333%22%20d%3D%22M287%2069.4a17.6%2017.6%200%200%200-13-5.4H18.4c-5%200-9.3%201.8-12.9%205.4A17.6%2017.6%200%200%200%200%2082.2c0%205%201.8%209.3%205.4%2012.9l128%20127.9c3.6%203.6%207.8%205.4%2012.8%205.4s9.2-1.8%2012.8-5.4L287%2095c3.5-3.5%205.4-7.8%205.4-12.8%200-5-1.9-9.2-5.5-12.8z%22%2F%3E%3C%2Fsvg%3E");
  background-repeat: no-repeat;
  background-position: right 12px top 50%;
  background-size: 10px auto;
  transition: all 0.2s ease;
}

/* Hover state */
.sort-control select:hover {
  border-color: #aaa;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

/* Focus state */
.sort-control select:focus {
  outline: none;
  border-color: #81C9E3;
  box-shadow: 0 0 0 3px rgba(129, 201, 227, 0.25);
}

/* Style the dropdown options (works in Firefox but not all browsers) */
.sort-control select option {
  font-family: "Neue Haas Grotesk Display Pro", sans-serif;
  padding: 8px;
}
</style>

<style>
/* Global styles that could be affecting other components - making them more specific */
.welfare-dog-tooltip {
  z-index: 10000;
  position: absolute;
  pointer-events: none;
}
</style>