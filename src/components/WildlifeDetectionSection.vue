<template>
  <div class="wildlife-detection-section">
    <!-- Decorative dogs container with tooltips -->
    <div class="decorative-dogs" :class="dogDisplayMode">
      <!-- Kipp with tooltip -->
      <div class="dog-wrapper">
        <img src="@/assets/images/kipp.png" alt="Kipp" class="dog-image kipp medium-dog" />
        <div class="dog-tooltip">
          <div class="dog-name">Detection Dog Kipp</div>
          <div class="dog-info">
            <div><strong>Breed:</strong> Kelpie-cross</div>
            <div><strong>Target species:</strong> Baw Baw frog, Platypus, Tasmanian devil (oestrus research), and Victorian Grassland Earless Dragon</div>
            <div><strong>Special qualities:</strong> Kip is our most experienced detection dog. He is highly motivated and doesn’t like sitting still for long unless it is a cuddle on the couch at the end of the day. Kip loves toys. His favourite is a squeaky tennis ball.</div>
          </div>
        </div>
      </div>

      <!-- Moss with tooltip -->
      <div class="dog-wrapper">
        <img src="@/assets/images/moss.png" alt="Moss" class="dog-image moss medium-dog" />
        <div class="dog-tooltip">
          <div class="dog-name">Detection Dog Moss</div>
          <div class="dog-info">
            <div><strong>Breed:</strong> Labrador Retriever</div>
            <div><strong>Target species:</strong> Broad-toothed rat, Platypus and Tasmanian Devil (oestrus research)</div>
            <div><strong>Special qualities:</strong> Moss is our team's naughtiest-good boy. He is always getting into mischief. He is confident in all environments and loves exploring the world with his nose. Moss loves food (bananas are his favourite) and belly rubs from his people.</div>
          </div>
        </div>
      </div>

      <!-- Sugar with tooltip -->
      <div class="dog-wrapper">
        <img src="@/assets/images/sugar.png" alt="Sugar" class="dog-image sugar medium-dog" />
        <div class="dog-tooltip">
          <div class="dog-name">Detection Dog Sugar</div>
          <div class="dog-info">
            <div><strong>Breed:</strong> German Shorthaired Pointer cross Springer Spaniel</div>
            <div><strong>Target species:</strong> Victorian Grassland Earless Dragon</div>
            <div><strong>Special qualities:</strong> Whilst Sugar is as sweet as her name suggests, she is fearless in moving through all environments, especially when following her nose. Sugar loves food, soft tug toys and cuddles from her people.</div>
          </div>
        </div>
      </div>

      <!-- Finn with tooltip -->
      <div class="dog-wrapper">
        <img src="@/assets/images/finn.png" alt="Finn" class="dog-image finn medium-dog" />
        <div class="dog-tooltip">
          <div class="dog-name">Detection Dog Finn</div>
          <div class="dog-info">
            <div><strong>Breed:</strong> Border Collie</div>
            <div><strong>Target species:</strong> All frogs! Finn is a frog generalist, and he works on projects for the Baw Baw Frog and other fighting extinction frog species.</div>
            <div><strong>Special qualities:</strong> Finn is energetic and loves challenging environments. He's particularly skilled at working in mountainous regions and has exceptional agility.</div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="section-content">
      <h1 class="main-title">Wildlife Detection Effectiveness</h1>
      
      <p class="description">
        Dogs' remarkable noses go beyond medicine—they're also used in wildlife detection. This study highlights Daisy, a detection dog from Zoos Victoria's dog squad, and her unique ability to sniff out rare fungi—something no existing technology can achieve.
      </p>

      <div class="chart-container" ref="chartContainer">
        <svg ref="svgChart"></svg>
      </div>

      <div class="legend">
        <div class="legend-item">
          <div class="legend-icon dog-icon daisy-wrapper">
            <img src="@/assets/images/daisy.png" alt="Daisy the detection dog icon" class="daisy-image" />
            <div class="dog-tooltip daisy-tooltip">
              <div class="dog-name">Detection Dog Daisy</div>
              <div class="dog-info">
                <div><strong>Breed:</strong> Lagotto Romagnolo</div>
                <div><strong>Target species:</strong> Victorian Grassland Earless Dragon and Tasmanian devil (oestrus research)</div>
                <div><strong>Special qualities:</strong> Don’t let Daisy’s size fool you, she is a whole lot of dog in a tiny body! Daisy loves food and cuddles above all else and moves delicately through her search environment, which is critical in sensitive habitats.</div>
              </div>
            </div>
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
        <br>Dog squad info: <a href="Fighting Extinction Dog Squad" target="_blank" rel="noopener noreferrer">https://www.zoo.org.au/fighting-extinction/fighting-extinction-dog-squad/</a>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import * as d3 from 'd3';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import daisyImage from '@/assets/images/daisy.png';
import kippImage from '@/assets/images/kipp.png';
import mossImage from '@/assets/images/moss.png';
import sugarImage from '@/assets/images/sugar.png';
import finnImage from '@/assets/images/finn.png';

// Ensure ScrollTrigger is registered
gsap.registerPlugin(ScrollTrigger);

export default {
  name: 'WildlifeDetectionSection',
  props: {
    dogDisplayMode: {
      type: String,
      default: 'minimal',
      validator: (value) => ['minimal', 'medium', 'maximum'].includes(value)
    }
  },
  setup(props) {
    const chartContainer = ref(null);
    const svgChart = ref(null);
    let resizeObserver = null;
    let scrollTriggerInstance = null;

    const drawChart = () => {
      if (!chartContainer.value || !svgChart.value) return;

      // Remove any existing tooltips before creating a new one
      d3.select(chartContainer.value).selectAll(".tooltip").remove();

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
        .style("stroke-dasharray", function() { return this.getTotalLength(); })
        .style("stroke-dashoffset", function() { return this.getTotalLength(); });

      // Create a tooltip element attached to the body instead of the chart container
      const tooltip = d3.select("body")
        .append("div")
        .attr("class", "wildlife-tooltip") // Use a more specific class name
        .style("position", "absolute")
        .style("background", "#fff")
        .style("border", "1px solid #ccc")
        .style("border-radius", "4px")
        .style("padding", "8px")
        .style("font-size", "12px")
        .style("font-family", '"ivypresto-headline", serif')
        .style("box-shadow", "0 2px 4px rgba(0, 0, 0, 0.1)")
        .style("pointer-events", "none")
        .style("opacity", 0)
        .style("z-index", "1000"); // Ensure tooltip is on top

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
          // Get direct references to the SVG elements
          const dogLine = svgChart.value.querySelector(".dog-line");
          const humanLine = svgChart.value.querySelector(".human-line");
          const dataPoints = Array.from(svgChart.value.querySelectorAll(".data-point"));
          
          // Use GSAP with direct element references
          gsap.to(dogLine, { strokeDashoffset: 0, duration: 2, ease: "power1.inOut" });
          gsap.to(humanLine, { strokeDashoffset: 0, duration: 2, ease: "power1.inOut", delay: 0.5 });
          gsap.to(dataPoints, { opacity: 1, duration: 0.5, stagger: 0.2, delay: 0.5 });
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
      
      // Initialize dog tooltips
      setupDogTooltips();
    });

    const setupDogTooltips = () => {
      // Use GSAP to handle dog tooltip animations
      gsap.utils.toArray('.dog-wrapper, .daisy-wrapper').forEach(wrapper => {
        const tooltip = wrapper.querySelector('.dog-tooltip');
        const dog = wrapper.querySelector('.dog-image') || wrapper.querySelector('.daisy-image');
        
        // Hide tooltips initially
        gsap.set(tooltip, { opacity: 0, scale: 0.8, transformOrigin: 'center top' });
        
        // Mouse enter animation
        dog.addEventListener('mouseenter', () => {
          gsap.to(tooltip, {
            duration: 0.3,
            opacity: 1,
            scale: 1,
            ease: 'back.out(1.7)'
          });
        });
        
        // Mouse leave animation
        dog.addEventListener('mouseleave', () => {
          gsap.to(tooltip, {
            duration: 0.2,
            opacity: 0,
            scale: 0.8,
            ease: 'power3.out'
          });
        });
      });
    };

    onBeforeUnmount(() => {
      // Cleanup
      if (resizeObserver && chartContainer.value) {
        resizeObserver.unobserve(chartContainer.value);
      }
      if (scrollTriggerInstance) {
        scrollTriggerInstance.kill();
      }
      
      // Remove any tooltips when component unmounts
      d3.selectAll(".wildlife-tooltip").remove();
    });

    // Watch for changes to dogDisplayMode
    watch(() => props.dogDisplayMode, (newMode) => {
      // Animate dogs based on mode change
      if (newMode === 'minimal') {
        // Hide all dogs
        gsap.to('.dog-image', {
          opacity: 0,
          scale: 0,
          duration: 0.5,
          stagger: 0.1,
          ease: 'power3.out'
        });
      } else {
        // Show dogs with staggered animation
        gsap.to('.medium-dog', {
          opacity: 1,
          scale: 1,
          rotation: (index) => [10, -8, 12, -15][index % 4], // Random rotation for each dog
          duration: 0.8,
          stagger: 0.15,
          ease: 'back.out(1.7)'
        });
      }
    }, { immediate: true });

    return {
      chartContainer,
      svgChart
    };
  }
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
  position: relative;
}

.section-content {
  width: 90%;
  max-width: 900px;
  margin: 0 auto;
  text-align: center;
}

.main-title {
  font-weight: 400;
  font-style: normal;
  font-size: 50px;
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
  display: block;
  margin: 0 auto;
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
  width: 40px;
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.dog-icon img {
  width: 100px;
  height: 100px;
  margin-right: 50px;
  object-fit: contain;
  border-radius: 50%;
  padding: 2px;
}

.human-icon {
  width: 19px;
  height: 19px;
  background-color: red;
  border-radius: 50%;
}

/* Source Styles */
.source {
  font-size: 11px;
  color: #A2A2A2;
  font-family: "Neue Haas Grotesk Display Pro";
  font-size: 12px;  margin-top: 30px;
  text-align: center;
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

/* Tooltip styles */
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
  z-index: 1000;
}

/* Decorative Dogs Styles */
.decorative-dogs {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  overflow: hidden;
  pointer-events: none; /* Add this line */
}

/* Dog wrapper to contain both image and tooltip */
.dog-wrapper {
  position: absolute;
  pointer-events: auto; /* Keep this to ensure dog interaction works */
}

.dog-image {
  position: relative;
  transition: all 0.8s cubic-bezier(0.22, 1, 0.36, 1);
  opacity: 0;
  transform: scale(0);
  cursor: pointer; /* Show pointer cursor on hover */
}

/* Dog tooltips */
.dog-tooltip {
  position: absolute;
  background-color: rgba(255, 255, 255, 0.7); /* White with 50% opacity */
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
  width: 280px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 10;
  pointer-events: none;
  font-family: "neue-haas-grotesk-display", sans-serif;
}

/* Position tooltips relative to their dogs */
.dog-wrapper:has(.kipp) .dog-tooltip {
  right: 70%;
  top: 40px;
}

.dog-wrapper:has(.moss) .dog-tooltip {
  left: 70%;
  top: 40px;
}

.dog-wrapper:has(.sugar) .dog-tooltip {
  left: 70%;
  bottom: 40px;
}

.dog-wrapper:has(.finn) .dog-tooltip {
  right: 70%;
  bottom: 40px;
}

.dog-name {
  color: #e63946;
  font-size: 22px;
  margin-top: 0;
  margin-bottom: 12px;
  color: #F00;
  font-family: "IvyPresto Headline";
  font-style: italic;
  font-weight: 400;
  line-height: normal;
}

.dog-info div {
  margin: 8px 0;
  color: #333;
  font-size: 14px;
  line-height: 1.4;
}

.dog-info strong {
  font-weight: 600;
}

/* For minimal state - hide all decorative dogs */
.decorative-dogs.minimal .dog-image {
  opacity: 0;
  transform: scale(0);
}

/* Medium and maximum states - show decorative dogs */
.decorative-dogs.medium .medium-dog,
.decorative-dogs.maximum .medium-dog {
  opacity: 1;
  transform: scale(1) rotate(0deg);
}

/* Position the dogs and their wrappers */
.dog-wrapper:has(.kipp) {
  bottom: 25%;
  right: -3%;
}

.dog-image.kipp {
  width: 300px;
  height: 300px;
  transform: rotate(8deg)!important;
}

.dog-wrapper:has(.moss) {
  bottom: 25%;
  left: -3%;
}

.dog-image.moss {
  width: 300px;
  height: 300px;
  transform: rotate(-15deg)!important;
}

.dog-wrapper:has(.sugar) {
  top: 25%;
  left: -3%;
}

.dog-image.sugar {
  width: 320px;
  height: 320px;
  transform: rotate(15.796deg)!important;
}

.dog-wrapper:has(.finn) {
  top: 25%;
  right: -3%;
}

.dog-image.finn {
  width: 320px;
  height: 320px;
  transform: rotate(-20.365deg)!important;
}

/* Daisy tooltip in legend */
.daisy-wrapper {
  position: relative;
  pointer-events: auto;
}

.daisy-image {
  cursor: pointer;
}

.daisy-tooltip {
  position: absolute;
  right: 130%; /* Position to the right of the image */
  top: -5%;
  width: 400px;
  z-index: 100;
}

/* For mobile */
@media (max-width: 768px) {
  .daisy-tooltip {
    left: auto;
    right: 110%;
    top: -60px;
    width: 200px;
  }
}

/* Adjust for responsive layouts */
@media (max-width: 768px) {
  .dog-image {
    width: 200px !important;
    height: 200px !important;
  }
  
  .dog-wrapper:has(.kipp) {
    right: -10%;
  }
  
  .dog-wrapper:has(.sugar) {
    left: -10%;
  }
  
  .dog-tooltip {
    width: 220px;
    padding: 10px;
  }
  
  .dog-name {
    font-size: 18px;
  }
  
  .dog-info p {
    font-size: 12px;
  }
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