<template>
    <div class="wildlife-detection-section">
      <div class="section-content">
        <h1 class="main-title">Wildlife Detection Effectiveness</h1>
        
        <p class="description">
          Dogs' remarkable noses go beyond medicine—they're also used in wildlife detection. This study highlights Daisy, a detection dog from Zoos Victoria, and her unique ability to sniff out rare fungi—something no existing technology can achieve.
        </p>
  
        <div class="chart-container" ref="chartContainer">
          <svg ref="svgChart"></svg>
        </div>
  
        <div class="legend">
          <div class="legend-item">
            <div class="legend-icon dog-icon">
               <img src="@/assets/images/daisy.png" alt="Daisy the detection dog icon" />
            </div>
            <span>Daisy the Detection dog</span>
          </div>
          <div class="legend-item">
            <div class="legend-icon human-icon"></div>
            <span>Human Surveyor</span>
          </div>
        </div>
  
        <div class="source">
          Source (adapted): Amor et al., iScience. 2024. Fig 2. <a href="https://doi.org/10.1016/j.isci.2024.109729" target="_blank" rel="noopener noreferrer">https://doi.org/10.1016/j.isci.2024.109729</a>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { ref, onMounted, onBeforeUnmount } from 'vue';
  import * as d3 from 'd3';
  import { gsap } from 'gsap';
  import { ScrollTrigger } from 'gsap/ScrollTrigger';
  import daisyImage from '@/assets/images/daisy.png'; // Import the image
  
  // Ensure ScrollTrigger is registered (usually done globally, but good practice)
  gsap.registerPlugin(ScrollTrigger);
  
  export default {
    name: 'WildlifeDetectionSection',
    setup() {
      const chartContainer = ref(null);
      const svgChart = ref(null);
      let resizeObserver = null;
      let scrollTriggerInstance = null;
  
      const drawChart = () => {
        if (!chartContainer.value || !svgChart.value) return;
  
        // --- Data ---
        const dimensions = ["Proportion of Fungus Found", "Time to First Detection", "Probability of False Negative"];
        const dogData = { "Proportion of Fungus Found": 0.67, "Time to First Detection": 7, "Probability of False Negative": 0.13 }; // 67%, 7min, 13%
        const humanData = { "Proportion of Fungus Found": 0.42, "Time to First Detection": 14, "Probability of False Negative": 0.37 }; // 42%, 14min, 37%
  
        // --- Setup SVG ---
        const containerWidth = chartContainer.value.clientWidth;
        const margin = { top: 60, right: 50, bottom: 80, left: 50 }; // Increased top/bottom margin
        const width = containerWidth - margin.left - margin.right;
        const height = 500 - margin.top - margin.bottom; // Fixed height
  
        const svg = d3.select(svgChart.value)
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .html(""); // Clear previous chart
  
        const g = svg.append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);
  
        // --- Scales ---
        const yScales = {
          "Proportion of Fungus Found": d3.scaleLinear().domain([0, 1]).range([height, 0]), // 0% at bottom, 100% at top
          "Time to First Detection": d3.scaleLinear().domain([20, 0]).range([height, 0]), // Inverted: 0min at top, 20min at bottom
          "Probability of False Negative": d3.scaleLinear().domain([0.5, 0]).range([height, 0]) // Inverted: 0% at top, 50% at bottom
        };
        const x = d3.scalePoint().range([0, width]).padding(0.5).domain(dimensions);
  
        // --- Draw Axes ---
        const axes = g.selectAll(".axis")
          .data(dimensions)
          .enter().append("g")
          .attr("class", "axis")
          .attr("transform", d => `translate(${x(d)},0)`);
  
        axes.append("line")
            .attr("y1", 0)
            .attr("y2", height)
            .attr("stroke", "#ccc");
  
        axes.append("text")
            .attr("class", "axis-label")
            .style("text-anchor", "middle")
            .attr("y", height + margin.bottom / 2) // Position below axis
            .text(d => d)
            .style("fill", "#333")
            .style("font-size", "12px")
            .style("font-family", '"ivypresto-headline", serif');
  
        // Add "better" labels at the top
        axes.append("text")
            .attr("class", "better-label")
            .style("text-anchor", "middle")
            .attr("y", -margin.top / 2 + 10) // Position above axis line
            .text("better")
            .style("fill", "#555")
            .style("font-size", "12px")
            .style("font-style", "italic")
            .style("font-family", '"ivypresto-headline", serif');
  
        // Add specific scale labels (0%, 50%, 100%, etc.)
        axes.each(function(d) {
            const axis = d3.select(this);
            const scale = yScales[d];
            let ticks;
            let format = d3.format(".0%"); // Default format
  
            if (d === "Proportion of Fungus Found") ticks = [0, 0.5, 1];
            else if (d === "Time to First Detection") {
                ticks = [0, 10, 20];
                format = val => `${val}min`;
            }
            else if (d === "Probability of False Negative") ticks = [0, 0.25, 0.5];
  
            axis.selectAll(".tick-label")
                .data(ticks)
                .enter().append("text")
                .attr("class", "tick-label")
                .attr("y", tick => scale(tick))
                .attr("x", -10) // Position to the left of the axis
                .attr("dy", "0.32em")
                .style("text-anchor", "end")
                .style("font-size", "10px")
                .style("fill", "#666")
                .text(format);
        });
  
        // --- Path function ---
        function path(d) {
          return d3.line()
            .x(p => x(p)) // X-coordinate based on dimension
            .y(p => yScales[p](d[p])) // Y-coordinate based on data value
            .curve(d3.curveLinear)(dimensions); // Use d3.curveLinear for straight lines
        }
  
        // --- Draw Lines ---
        const dogLine = g.append("path")
          .datum(dogData)
          .attr("class", "line dog-line")
          .attr("d", path)
          .style("fill", "none") // Ensure no fill for the path
          .style("stroke", "#000") // Black line for dog
          .style("stroke-width", "3px") // Adjust thickness
          .style("stroke-linecap", "round") // Round line ends
          .style("stroke-linejoin", "round") // Round line joins
          .style("stroke-dasharray", function() { return this.getTotalLength(); })
          .style("stroke-dashoffset", function() { return this.getTotalLength(); });
  
        const humanLine = g.append("path")
          .datum(humanData)
          .attr("class", "line human-line")
          .attr("d", path)
          .style("fill", "none") // Ensure no fill for the path
          .style("stroke", "#aaa") // Gray line for human
          .style("stroke-width", "3px") // Adjust thickness
          .style("stroke-linecap", "round") // Round line ends
          .style("stroke-linejoin", "round") // Round line joins
          .style("stroke-dasharray", function() { return this.getTotalLength(); })
          .style("stroke-dashoffset", function() { return this.getTotalLength(); });
  
        // Create a tooltip element
        const tooltip = d3.select(chartContainer.value)
          .append("div")
          .attr("class", "tooltip")
          .style("position", "absolute")
          .style("background", "#fff")
          .style("border", "1px solid #ccc")
          .style("border-radius", "4px")
          .style("padding", "8px")
          .style("font-size", "12px")
          .style("font-family", '"ivypresto-headline", serif')
          .style("box-shadow", "0 2px 4px rgba(0, 0, 0, 0.1)")
          .style("pointer-events", "none")
          .style("opacity", 0);
  
        // Function to show tooltip with formatted values
        const showTooltip = (event, d, data, label) => {
          // Format the value based on the dimension
          let formattedValue;
          if (d === "Proportion of Fungus Found" || d === "Probability of False Negative") {
            formattedValue = `${(data[d] * 100).toFixed(0)}%`; // Convert to percentage
          } else if (d === "Time to First Detection") {
            formattedValue = `${data[d]} min`; // Add "min" for time
          }
  
          tooltip
            .html(`<strong>${label}</strong>: ${formattedValue}`)
            .style("left", `${event.pageX + 10}px`)
            .style("top", `${event.pageY - 20}px`)
            .style("opacity", 1);
        };
  
        // Function to hide tooltip
        const hideTooltip = () => {
          tooltip.style("opacity", 0);
        };
  
        // --- Draw Data Points (initially hidden) ---
        const dogPoints = g.selectAll(".dog-point")
          .data(dimensions)
          .enter().append("image")
          .attr("class", "dog-point data-point")
          .attr("xlink:href", daisyImage) // Use the imported image
          .attr("x", d => x(d) - 40) 
          .attr("y", d => yScales[d](dogData[d]) - 40)
          .attr("width", 90)
          .attr("height", 90)
          .style("opacity", 0)
          .on("mouseover", (event, d) => {
            // Show tooltip
            showTooltip(event, d, dogData, "Daisy");
            
            // Grow the image
            d3.select(event.currentTarget)
              .transition()
              .duration(300)
              .attr("width", 110)
              .attr("height", 110)
              .attr("x", d => x(d) - 50) // Adjust position to keep centered
              .attr("y", d => yScales[d](dogData[d]) - 50);
              
            // Also animate the blue outline if it exists
            g.selectAll(".dog-outline")
              .filter((dp, i) => dp === d)
              .transition()
              .duration(300)
              .attr("r", 32); // Increase radius of the outline
          })
          .on("mousemove", (event) => {
            tooltip.style("left", `${event.pageX + 10}px`).style("top", `${event.pageY - 20}px`);
          })
          .on("mouseout", (event, d) => {
            // Hide tooltip
            hideTooltip();
            
            // Return image to original size
            d3.select(event.currentTarget)
              .transition()
              .duration(300)
              .attr("width", 90)
              .attr("height", 90)
              .attr("x", d => x(d) - 40) 
              .attr("y", d => yScales[d](dogData[d]) - 40);
              
            // Restore outline
            g.selectAll(".dog-outline")
              .filter((dp, i) => dp === d)
              .transition()
              .duration(300)
              .attr("r", 27);
          });
  
        // Add blue outline using a clipPath and rect/path
        const clipId = "dog-clip-";
        svg.append("defs").selectAll("clipPath")
          .data(dimensions)
          .enter().append("clipPath")
          .attr("id", (d, i) => clipId + i)
          .append("circle") // Simple circle clip, adjust as needed
          .attr("cx", d => x(d) + margin.left)
          .attr("cy", d => yScales[d](dogData[d]) + margin.top)
          .attr("r", 27); // Slightly larger than half the image size
  
        g.selectAll(".dog-outline")
          .data(dimensions)
          .enter().append("circle") // Draw the blue circle outline
          .attr("class", "dog-outline data-point")
          .attr("cx", d => x(d))
          .attr("cy", d => yScales[d](dogData[d]))
          .attr("r", 27) // Match clip path radius
          .style("fill", "none")
          .style("stroke-width", 3)
          .style("opacity", 0);
  
        const humanPoints = g.selectAll(".human-point")
          .data(dimensions)
          .enter().append("circle")
          .attr("class", "human-point data-point")
          .attr("cx", d => x(d))
          .attr("cy", d => yScales[d](humanData[d]))
          .attr("r", 6) // Red circle size
          .style("opacity", 0)
          .style("fill", "red") // Add this line to set the fill color
          .on("mouseover", (event, d) => showTooltip(event, d, humanData, "Human"))
          .on("mousemove", (event) => {
            tooltip.style("left", `${event.pageX + 10}px`).style("top", `${event.pageY - 20}px`);
          })
          .on("mouseout", hideTooltip);
  
        // --- Animation ---
        scrollTriggerInstance = ScrollTrigger.create({
          trigger: svgChart.value,
          start: "top 70%", // Start animation when 70% of the chart is visible
          once: true, // Animate only once
          onEnter: () => {
            gsap.to(".dog-line", { strokeDashoffset: 0, duration: 2, ease: "power1.inOut" });
            gsap.to(".human-line", { strokeDashoffset: 0, duration: 2, ease: "power1.inOut", delay: 0.5 });
            gsap.to(".data-point", { opacity: 1, duration: 0.5, stagger: 0.2, delay: 0.5 });
          }
        });
      };
  
      onMounted(() => {
        drawChart(); // Initial draw
  
        // Redraw chart on resize
        resizeObserver = new ResizeObserver(drawChart);
        if (chartContainer.value) {
          resizeObserver.observe(chartContainer.value);
        }
      });
  
      onBeforeUnmount(() => {
        // Cleanup
        if (resizeObserver && chartContainer.value) {
          resizeObserver.unobserve(chartContainer.value);
        }
        if (scrollTriggerInstance) {
          scrollTriggerInstance.kill();
        }
      });
  
      return {
        chartContainer,
        svgChart,
      };
    },
  };
  </script>
  
  <style scoped>
  .wildlife-detection-section {
    width: 100%;
    min-height: 140vh;
    background-color: #F2EFEB;
    padding: 100px 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    scroll-snap-align: start;
    scroll-snap-stop: always;
    font-family: "ivypresto-headline", serif;
  }
  
  .section-content {
    width: 90%;
    max-width: 900px; /* Adjusted max-width for chart */
    margin: 0 auto;
    text-align: center; /* Center title and subtitle */
  }
  
  .main-title {
    font-weight: 400;
    font-style: normal;
    font-size: 50px; /* Slightly smaller */
    color: red;
    margin-bottom: 15px;
  }
  
  .description {
    font-family: "neue-haas-grotesk-display", sans-serif;
    font-weight: 150;
    font-style: normal;
    color: #A2A2A2;
    font-size: 16px;
    max-width: 650px;
    margin-top: 50px;
    position: relative;
    z-index: 2;
    display: block !important;
    visibility: visible !important;
    opacity: 1 !important;
  }
  
  .chart-container {
    width: 100%;
    margin-bottom: 40px;
  }
  
  svg {
    display: block; /* Prevent extra space below SVG */
    margin: 0 auto; /* Center SVG if container is wider */
  }
  
  /* D3 Chart Styles */
  .line {
    fill: none;
    stroke-width: 2px;
  }
  
  .axis line {
    stroke: #ccc;
  }
  
  .axis-label {
    font-family: "ivypresto-headline", serif;
    font-size: 12px;
    fill: #333;
  }
  
  .tick-label {
     font-family: "ivypresto-headline", serif;
     font-size: 10px;
     fill: #666;
  }
  
  .better-label {
     font-family: "ivypresto-headline", serif;
     font-size: 12px;
     font-style: italic;
     fill: #555;
  }
  
  /* Legend Styles */
  .legend {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 30px;
    margin-bottom: 20px;
  }
  
  .legend-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
  }
  
  .legend-icon {
    width: 40px; /* Adjust size */
    height: 40px; /* Adjust size */
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .dog-icon img {
    width: 100px; /* Adjusted size */
    height: 100px; /* Adjusted size */
    margin-right: 50px; /* Increased space between image and text */
    object-fit: contain;
    border-radius: 50%;
    padding: 2px; /* Space between image and border */
  }
  
  .human-icon {
    width: 19px; /* Match point size */
    height: 19px; /* Match point size */
    background-color: red;
    border-radius: 50%;
  }
  
  /* Source Styles */
  .source {
    font-size: 11px;
    color: #999;
    margin-top: 30px;
    text-align: center;
  }
  
  .source a {
    color: #777;
    text-decoration: none;
  }
  
  .source a:hover {
    text-decoration: underline;
  }
  
  /* Tooltip styles - moved outside media query */
  .tooltip {
    position: absolute;
    background: #fff;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 8px;
    font-size: 12px;
    font-family: "ivypresto-headline", serif;
    font-weight: 400;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    pointer-events: none;
    opacity: 0;
    transition: opacity 0.2s ease-in-out;
    z-index: 1000; /* Ensure tooltip is on top */
  }
  
  @media (max-width: 768px) {
    .main-title {
      font-size: 36px;
    }
    .description {
      font-size: 18px; /* Smaller font on mobile */
      max-width: 90%;
    }
    .axis-label {
      font-size: 10px;
    }
    .legend {
      flex-direction: column;
      gap: 15px;
    }
  }
  </style>