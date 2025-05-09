<template>
  <div class="breed-strength-section" ref="breedSection">
    <div class="filters" ref="filtersElement">
      <button 
        v-for="(data, breedId) in breedData" 
        :key="breedId"
        class="filter-button" 
        :class="{ active: currentBreed === breedId }"
        :data-breed="breedId"
        @click="updateBreed(breedId)"
      >
        {{ data.name }}
      </button>
    </div>
    
    <div class="breed-container">
      <div class="breed-info">
        <div class="dog-image" ref="dogImageElement">
          <img :src="getImageUrl(breedData[currentBreed].imageSrc)" :alt="breedData[currentBreed].name">
        </div>
        <div class="popular-breed-role" ref="breedRoleLabelElement">Popular breed role:</div>
        <div class="breed-role" ref="breedNameElement">{{ breedData[currentBreed].optimal }}</div>
        <div class="breed-description" ref="breedDescriptionElement">{{ breedData[currentBreed].description }}</div>
      </div>
      
      <div class="chart-container" ref="chartContainerElement">
        <div id="radar-chart" ref="radarChart"></div>
        <div class="chart-source">Source: <a href="https://www.akc.org/dog-breeds/" target="_blank" rel="noopener noreferrer">American Kennel Club</a></div>      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import * as d3 from 'd3';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

// Register ScrollTrigger plugin
gsap.registerPlugin(ScrollTrigger);

// Import images directly
import labImage1 from '@/assets/images/lab/lab-1.png';
import labImage2 from '@/assets/images/lab/lab-2.png';
import labImage3 from '@/assets/images/lab/lab-3.png';

import gshepImage1 from '@/assets/images/gshep/gshep-1.png';
import gshepImage2 from '@/assets/images/gshep/gshep-2.png';
import gshepImage3 from '@/assets/images/gshep/gshep-3.png';

import bcollieImage1 from '@/assets/images/bcollie/bcollie-1.png';
import bcollieImage2 from '@/assets/images/bcollie/bcollie-2.png';
import bcollieImage3 from '@/assets/images/bcollie/bcollie-3.png';

import beagleImage1 from '@/assets/images/beagle/beagle-1.png';
import beagleImage2 from '@/assets/images/beagle/beagle-2.png';
import beagleImage3 from '@/assets/images/beagle/beagle-3.png';
import { descending } from 'd3';

export default {
  name: 'BreedStrengthSection',
  props: {
    dogDisplayMode: {
      type: String,
      default: 'minimal',
      validator: (value) => ['minimal', 'medium', 'maximum'].includes(value)
    }
  },
  setup(props) {
    const isAnimating = ref(false);
    const breedSection = ref(null);
    const dogImageElement = ref(null);
    const breedRoleLabelElement = ref(null);
    const breedNameElement = ref(null);
    const breedDescriptionElement = ref(null);
    const radarChart = ref(null);
    const chartContainerElement = ref(null);
    const filtersElement = ref(null);
    const currentBreed = ref('labrador-retriever');
    let svgElement = null;
    let animationsInitialized = false;

    // Function to get image URL - update this function
    function getImageUrl(imageSrcObject) {
      // The current display mode from props
      const mode = props.dogDisplayMode;
      
      // Get the path for the current mode from the image source object
      const path = typeof imageSrcObject === 'object' ? imageSrcObject[mode] : imageSrcObject;
      
      // Map paths to imported images
      const images = {
        '@/assets/images/lab/lab-1.png': labImage1,
        '@/assets/images/lab/lab-2.png': labImage2,
        '@/assets/images/lab/lab-3.png': labImage3,
        '@/assets/images/gshep/gshep-1.png': gshepImage1,
        '@/assets/images/gshep/gshep-2.png': gshepImage2,
        '@/assets/images/gshep/gshep-3.png': gshepImage3,
        '@/assets/images/bcollie/bcollie-1.png': bcollieImage1,
        '@/assets/images/bcollie/bcollie-2.png': bcollieImage2,
        '@/assets/images/bcollie/bcollie-3.png': bcollieImage3,
        '@/assets/images/beagle/beagle-1.png': beagleImage1,
        '@/assets/images/beagle/beagle-2.png': beagleImage2,
        '@/assets/images/beagle/beagle-3.png': beagleImage3
      };
      
      // Debug which image is being requested
      console.log("Requested image path:", path);
      console.log("Display mode:", mode);
      
      return images[path] || '';
    }

    // Breed data for the chart
    const breedData = {
      'labrador-retriever': {
        name: 'Labrador Retriever',
        optimal: 'Service/Guide',
        imageSrc: {
          minimal: '@/assets/images/lab/lab-1.png',
          medium: '@/assets/images/lab/lab-2.png',
          maximum: '@/assets/images/lab/lab-3.png'
        },
        description: 'Highly sociable, trainable, and adaptable, labs are common for service work, especially for people with disabilities, thanks to its friendly, reliable nature and ability to adjust to their human’s daily routines',
        traits: [
          {axis: "Socialbility", value: 5, description: "Everyone is my best friend"},
          {axis: "Protective Nature", value: 3, description: "Cautiously aware"},
          {axis: "Trainability", value: 5, description: "Eager to please"},
          {axis: "Intelligence", value: 5, description: "Sharp as a Tack"},
          {axis: "Playfulness", value: 5, description: "Non-stop"},
          {axis: "Adaptability", value: 5, description: "Highly Adaptable"}
        ]
      },
      'german-shepherd': {
        name: 'German Shepherd',
        optimal: 'Police/Military',
        imageSrc: {
          minimal: '@/assets/images/gshep/gshep-1.png',
          medium: '@/assets/images/gshep/gshep-2.png',
          maximum: '@/assets/images/gshep/gshep-3.png'
        },
        description: 'German Shepherds are courageous, confident, and smart, excelling in police and military work.',
        traits: [
        {axis: "Socialbility", value: 3, description: "Friendly if Familiar"},
          {axis: "Protective Nature", value: 5, description: "Watchdog"},
          {axis: "Trainability", value: 5, description: "Eager to Please"},
          {axis: "Intelligence", value: 5, description: "Sharp as a Tack"},
          {axis: "Playfulness", value: 4, description: "Energetic Entertainer"},
          {axis: "Adaptability", value: 5, description: "Highly Adaptable"}
        ]
      },
      'border-collie': {
        name: 'Border Collie',
        optimal: 'Herding',
        imageSrc: {
          minimal: '@/assets/images/bcollie/bcollie-1.png',
          medium: '@/assets/images/bcollie/bcollie-2.png',
          maximum: '@/assets/images/bcollie/bcollie-3.png'
        },
        description: 'Border Collies are energetic, intelligent, and highly trainable, making them the top choice for herding.',
        traits: [
        {axis: "Socialbility", value: 4, description: "Social Butterfly"},
          {axis: "Protective Nature", value: 3, description: "Observant"},
          {axis: "Trainability", value: 5, description: "Eager to Please"},
          {axis: "Intelligence", value: 5, description: "Sharp as a Tack"},
          {axis: "Playfulness", value: 5, description: "Non-stop"},
          {axis: "Adaptability", value: 5, description: "Highly Adaptable"}
        ]
      },
     
      'beagle': {
        name: 'Beagle',
        optimal: 'Scent Detection',
        imageSrc: {
          minimal: '@/assets/images/beagle/beagle-1.png',
          medium: '@/assets/images/beagle/beagle-2.png',
          maximum: '@/assets/images/beagle/beagle-3.png'
        },
        description: 'Beagles are curious, friendly, and have an excellent sense of smell, making them perfect for scent detection.',
        traits: [
          {axis: "Socialbility", value: 3, description: "Friendly if Familiar"},
          {axis: "Protective Nature", value: 2, description: "Friendly, but Cautious"},
          {axis: "Trainability", value: 3, description: "Willing but Not Always Quick"},
          {axis: "Intelligence", value: 1, description: "Blissfully Unbothered"},
          {axis: "Playfulness", value: 4, description: "Energetic Entertainer"},
          {axis: "Adaptability", value: 4, description: "Quick Adjuster"}
        ]
      }
    };

    // Updated radar chart configuration
    const radarChartOptions = {
      w: 500 - 100,  // width minus margins
      h: 500 - 120,  // height minus margins (increased from 500)
      margin: {top: 100, right: 100, bottom: 100, left: 100}, // Increased margins
      maxValue: 5,  // Changed from 10 to 5
      levels: 5,
      roundStrokes: true,
      color: d3.scaleOrdinal().range(["#737373"]),
      transitionDuration: 800  // Animation duration in milliseconds
    };
    

    // Function to animate the optimal role text
    function animateOptimalRole(newOptimalRole, timeline) {
      const roleElement = breedNameElement.value;
      if (!roleElement) return;
      
      // Instead of creating a new timeline, add to the provided one
      timeline.to(roleElement, {
        opacity: 0,
        y: -50,
        duration: 0.3,
        ease: "power2.in",
        onComplete: () => {
          roleElement.textContent = newOptimalRole;
        }
      });
      
      timeline.fromTo(roleElement, 
        { opacity: 0, y: 50 },
        { opacity: 1, y: 0, duration: 0.5, ease: "back.out(1.7)" }
      );
    }

    // Function to animate the breed description text
    function animateBreedDescription(newDescription) {
      const descElement = breedDescriptionElement.value;
      if (!descElement) return;
      const tl = gsap.timeline();
      tl.to(descElement, {
        opacity: 0,
        y: -30,
        duration: 0.2,
        ease: "power2.in",
        onComplete: () => {
          descElement.textContent = newDescription;
        }
      });
      tl.fromTo(descElement, 
        { opacity: 0, y: 30 },
        { opacity: 1, y: 0, duration: 0.4, ease: "back.out(1.7)" }
      );
    }

    // Function to animate the dog image with a reveal effect
    function animateDogImage(newImageSrc) {
      const dogElement = dogImageElement.value;
      if (!dogElement) return;
      
      // Create a GSAP timeline for the animation sequence
      const tl = gsap.timeline();
      
      // First, animate the current image out with a scale effect
      tl.to(dogElement, {
        opacity: 0,
        scale: 0.8,
        duration: 0.4,
        ease: "power2.in",
        onComplete: () => {
          // Update the image source
          const imageElement = dogElement.querySelector('img');
          if (imageElement) {
            imageElement.src = getImageUrl(newImageSrc);
          }
        }
      });
      
      // Then, animate the new image in with a reveal effect
      tl.fromTo(dogElement, 
        { opacity: 0, scale: 0.8 },
        { 
          opacity: 1, 
          scale: 1, 
          duration: 0.6, 
          ease: "back.out(1.7)",
          clearProps: "all" // Ensures no transform residue stays
        }
      );
    }

    // Initialize the radar chart with improved positioning
    function initRadarChart() {
      if (!radarChart.value) return;
      
      const margin = radarChartOptions.margin,
            width = radarChartOptions.w + margin.left + margin.right,
            height = radarChartOptions.h + margin.top + margin.bottom;
      
      const svg = d3.select(radarChart.value).append("svg")
          .attr("width", width + 50)  // Add extra width
          .attr("height", height + 50)  // Add extra height
          .attr("class", "radar");
      
      svgElement = svg;
      
      // Append a g element
      const g = svg.append("g")
          .attr("transform", `translate(${(width/2)},${(height/2)})`);
      
      // Glow filter for the lines
      const filter = g.append('defs').append('filter').attr('id','glow'),
        feGaussianBlur = filter.append('feGaussianBlur').attr('stdDeviation','2.5').attr('result','coloredBlur'),
        feMerge = filter.append('feMerge'),
        feMergeNode_1 = feMerge.append('feMergeNode').attr('in','coloredBlur'),
        feMergeNode_2 = feMerge.append('feMergeNode').attr('in','SourceGraphic');
      
      // Draw the background circles
      const axisGrid = g.append("g").attr("class", "axisWrapper");
      
      const allAxis = breedData[currentBreed.value].traits.map(i => i.axis),
            total = allAxis.length,
            radius = Math.min(radarChartOptions.w/2, radarChartOptions.h/2),
            angleSlice = Math.PI * 2 / total;
      
      // Scale for the radius
      const rScale = d3.scaleLinear()
        .range([0, radius])
        .domain([0, radarChartOptions.maxValue]);
      
      // Draw background circles
      axisGrid.selectAll(".levels")
        .data(d3.range(1, (radarChartOptions.levels+1)).reverse())
        .enter()
        .append("circle")
        .attr("class", "gridCircle")
        .attr("r", d => radius / radarChartOptions.levels * d)
        .style("fill", "#CDCDCD")
        .style("stroke", "#CDCDCD")
        .style("fill-opacity", 0.1);
      
      // Text indicating at what % each level is
      axisGrid.selectAll(".axisLabel")
        .data(d3.range(1, (radarChartOptions.levels+1)).reverse())
        .enter().append("text")
        .attr("class", "axisLabel")
        .attr("x", 4)
        .attr("y", d => -d * radius / radarChartOptions.levels)
        .attr("dy", "0.4em")
        .style("font-size", "10px")
        .attr("fill", "#737373")
        .text(d => radarChartOptions.maxValue * d / radarChartOptions.levels);
      
      // Create the straight lines radiating outward from the center
      const axis = axisGrid.selectAll(".axis")
        .data(allAxis)
        .enter()
        .append("g")
        .attr("class", "axis");
      
      // Append the lines
      axis.append("line")
        .attr("x1", 0)
        .attr("y1", 0)
        .attr("x2", (d, i) => rScale(radarChartOptions.maxValue * 1.0) * Math.cos(angleSlice * i - Math.PI / 2))
        .attr("y2", (d, i) => rScale(radarChartOptions.maxValue * 1.) * Math.sin(angleSlice * i - Math.PI / 2))
        .attr("class", "line")
        .style("stroke", "#CDCDCD")
        .style("stroke-width", "1px");
      
        axis.append("text")
        .attr("class", "legend")
        .style("font-size", "14px")
        .style("font-family", "'roca', sans-serif") // Add Neue Haas font
        .attr("text-anchor", function(d, i) {
          // Custom text anchor based on position
          const angle = angleSlice * i - Math.PI / 2;
          // Bottom label needs "middle", left needs "end", right needs "start"
          if (Math.abs(Math.sin(angle)) > 0.9) { // Bottom or top
            return "middle";
          } else if (Math.cos(angle) < 0) { // Left side
            return "end";
          } else { // Right side
            return "start";
          }
        })
        .attr("dy", function(d, i) {
          // Adjust vertical position for bottom label
          const angle = angleSlice * i - Math.PI / 2;
          return Math.sin(angle) > 0.8 ? "1em" : "0.35em"; // Give more space at bottom
        })
        .attr("x", function(d, i) {
          const angle = angleSlice * i - Math.PI / 2;
          // Create specific spacing based on the axis position
          let distanceFactor;
          
          if (i === 0) { // Socialbility (top)
            distanceFactor = 1.20;
          } else if (i === 1) { // Protective Nature (top-right)
            distanceFactor = 1.15;
          } else if (i === 2) { // Trainability (right)
            distanceFactor = 1.15;
          } else if (i === 3) { // Intelligence (bottom)
            distanceFactor = 1.05;
          } else if (i === 4) { // Playfulness (bottom-left)
            distanceFactor = 1.15;
          } else if (i === 5) { // Adaptability (left)
            distanceFactor = 1.15;
          }
          
          return rScale(radarChartOptions.maxValue * distanceFactor) * Math.cos(angle);
        })
        .attr("y", function(d, i) {
          const angle = angleSlice * i - Math.PI / 2;
          // Create specific spacing based on the axis position
          let distanceFactor;
          let yOffset = -10; // Move all labels up by 10px
          
          if (i === 0) { // Socialbility (top)
            distanceFactor = 1.20;
            yOffset = -10; // Adjusted to move the label down closer to the chart
          } else if (i === 1) { // Protective Nature (top-right)
            distanceFactor = 1.15;
          } else if (i === 2) { // Trainability (right)
            distanceFactor = 1.0;
          } else if (i === 3) { // Intelligence (bottom)
            distanceFactor = 1.00;
            yOffset = 0; // Don't shift the bottom label up
          } else if (i === 4) { // Playfulness (bottom-left)
            distanceFactor = 1.05;
          } else if (i === 5) { // Adaptability (left)
            distanceFactor = 1.15;
          }
          
          return rScale(radarChartOptions.maxValue * distanceFactor) * Math.sin(angle) + yOffset;
        })
        .text(d => d)
        .call(wrap, 60);
      
      // Create a wrapper for the blobs
      const blobWrapper = g.append("g")
          .attr("class", "radarWrapper");
      
      // The radial line function
      const radarLine = d3.lineRadial()
        .curve(d3.curveLinearClosed)
        .radius(d => rScale(d.value))
        .angle((d, i) => i * angleSlice);
      
      // Append the backgrounds
      blobWrapper.append("path")
        .attr("class", "radarArea")
        .attr("d", radarLine(breedData[currentBreed.value].traits))
        .style("fill", "rgba(139, 206, 229)")
        .style("fill-opacity", 0.5)
        .on('mouseover', function() {
          d3.select(this).transition().duration(200).style("fill-opacity", 0.8);
        })
        .on('mouseout', function() {
          d3.select(this).transition().duration(200).style("fill-opacity", 0.6);
        });
      
      // Create the outlines
      blobWrapper.append("path")
        .attr("class", "radarStroke")
        .attr("d", radarLine(breedData[currentBreed.value].traits))
        .style("stroke-width", "2px")
        .style("stroke", "rgba(139, 206, 229, 0.2)")
        .style("fill", "none")
        .style("filter", "url(#glow)");
      
      // Append the circles
      blobWrapper.selectAll(".radarCircle")
        .data(breedData[currentBreed.value].traits)
        .enter()
        .append("circle")
        .attr("class", "radarCircle")
        .attr("r", 4)
        .attr("cx", (d, i) => rScale(d.value) * Math.cos(angleSlice * i - Math.PI / 2))
        .attr("cy", (d, i) => rScale(d.value) * Math.sin(angleSlice * i - Math.PI / 2))
        .style("fill", "rgba(139, 206, 229)")
        .style("fill-opacity", 0.8);
      
      // Set up the tooltip
      const tooltip = g.append("g")
        .attr("class", "tooltip-group")
        .style("opacity", 0);

      // Add background rect
      tooltip.append("rect")
        .attr("class", "tooltip-bg")
        .attr("rx", 16) // more rounded corners to match screenshot
        .attr("ry", 16)
        .style("fill", "white")
       .style("opacity", 0.7)
        .style("stroke", "#eaeaea")
        .style("stroke-width", "1px");

      // Add description text (now displayed on top in red Ivy Presto)
      tooltip.append("text")
        .attr("class", "tooltip-desc")
        .attr("x", 0)
        .attr("y", 25)
        .attr("text-anchor", "middle") // center align
        .style("font-family", "'ivypresto-headline', serif")
        .style("font-style", "italic")
        .style("font-size", "20px")
        .style("fill", "#FF0000");

      // Add value text (now displayed below in Neue Haas)
      tooltip.append("text")
        .attr("class", "tooltip-title")
        .attr("x", 0)
        .attr("y", 55)
        .attr("text-anchor", "middle") // center align
        .style("font-size", "16px")
        .style("font-family", "'neue-haas-grotesk-display', sans-serif")
        .style("color", "#000000");
      
      // Append invisible circles for tooltip
      blobWrapper.selectAll(".radarInvisibleCircle")
        .data(breedData[currentBreed.value].traits)
        .enter().append("circle")
        .attr("class", "radarInvisibleCircle")
        .attr("r", 6)
        .attr("cx", (d, i) => rScale(d.value) * Math.cos(angleSlice * i - Math.PI / 2))
        .attr("cy", (d, i) => rScale(d.value) * Math.sin(angleSlice * i - Math.PI / 2))
        .style("fill", "none")
        .style("pointer-events", "all")
        .on("mouseover", function(event, d) {
          const newX = parseFloat(d3.select(this).attr('cx'));
          const newY = parseFloat(d3.select(this).attr('cy')) - 90;
          
          // Set description and value text
          const descText = d.description || ""; 
          const valueText = `${d.axis}: ${d.value}`;
          
          // Update tooltip text content
          tooltip.select(".tooltip-desc").text(descText);
          tooltip.select(".tooltip-title").text(valueText);
          
          // Calculate width based on text length
          const descWidth = tooltip.select(".tooltip-desc").node().getComputedTextLength();
          const titleWidth = tooltip.select(".tooltip-title").node().getComputedTextLength();
          const boxWidth = Math.max(descWidth, titleWidth) + 40;
          
          // Update background rect
          tooltip.select(".tooltip-bg")
            .attr("width", boxWidth)
            .attr("height", 70)
            .attr("x", -boxWidth/2); // Center the background
          
          // Position tooltip
          tooltip.attr("transform", `translate(${newX},${newY})`)
            .transition().duration(150)
            .style('opacity', 1);
        })
        .on("mouseout", function() {
          tooltip.transition().duration(200)
            .style("opacity", 0);
        });
    }

    // Function to update the radar chart with animation
    function updateRadarChart(breedId) {
      const breedInfo = breedData[breedId];
      
      if (!svgElement) return;
      
      const g = svgElement.select("g");
      if (!g) return;
      
      const margin = radarChartOptions.margin,
            radius = Math.min(radarChartOptions.w/2, radarChartOptions.h/2),
            angleSlice = Math.PI * 2 / breedInfo.traits.length;
      
      // Scale for the radius
      const rScale = d3.scaleLinear()
        .range([0, radius])
        .domain([0, radarChartOptions.maxValue]);
      
      // The radial line function
      const radarLine = d3.lineRadial()
        .curve(d3.curveLinearClosed)
        .radius(d => rScale(d.value))
        .angle((d, i) => i * angleSlice);
      
      // Select and update the radarArea
      const radarArea = g.select(".radarArea");
      if (radarArea) {
        radarArea.transition()
          .duration(radarChartOptions.transitionDuration)
          .attr("d", radarLine(breedInfo.traits));
      }
      
      // Select and update the radarStroke
      const radarStroke = g.select(".radarStroke");
      if (radarStroke) {
        radarStroke.transition()
          .duration(radarChartOptions.transitionDuration)
          .attr("d", radarLine(breedInfo.traits));
      }
      
      // Select and update the radarCircles
      const radarCircles = g.selectAll(".radarCircle");
      if (radarCircles) {
        radarCircles.data(breedInfo.traits)
          .transition()
          .duration(radarChartOptions.transitionDuration)
          .attr("cx", (d, i) => rScale(d.value) * Math.cos(angleSlice * i - Math.PI / 2))
          .attr("cy", (d, i) => rScale(d.value) * Math.sin(angleSlice * i - Math.PI / 2));
      }
      
      // Select and update the invisible circles for tooltips
      const invisibleCircles = g.selectAll(".radarInvisibleCircle");
      if (invisibleCircles) {
        invisibleCircles.data(breedInfo.traits)
          .transition()
          .duration(radarChartOptions.transitionDuration)
          .attr("cx", (d, i) => rScale(d.value) * Math.cos(angleSlice * i - Math.PI / 2))
          .attr("cy", (d, i) => rScale(d.value) * Math.sin(angleSlice * i - Math.PI / 2));
      }
    }

    // Helper function to wrap text
    function wrap(text, width) {
      text.each(function() {
        var text = d3.select(this),
            words = text.text().split(/\s+/).reverse(),
            word,
            line = [],
            lineNumber = 0,
            lineHeight = 1.4,
            y = text.attr("y"),
            x = text.attr("x"),
            dy = parseFloat(text.attr("dy")),
            tspan = text.text(null).append("tspan").attr("x", x).attr("y", y).attr("dy", dy + "em");
        
        while (word = words.pop()) {
          line.push(word);
          tspan.text(line.join(" "));
          if (tspan.node().getComputedTextLength() > width) {
            line.pop();
            tspan.text(line.join(" "));
            line = [word];
            tspan = text.append("tspan").attr("x", x).attr("y", y).attr("dy", ++lineNumber * lineHeight + dy + "em").text(word);
          }
        }
      });
    }

    // Function to update the current breed
    function updateBreed(breedId) {
      // Don't update if it's already the current breed
      if (currentBreed.value === breedId) return;
      
      // Don't allow another update if we're currently animating
      if (isAnimating.value) return;
      
      try {
        // Set animating state
        isAnimating.value = true;
        
        // Update current breed
        currentBreed.value = breedId;
        
        // Create a timeline for all animations
        const masterTimeline = gsap.timeline({
          onComplete: () => {
            // Reset animation state when all animations are done
            isAnimating.value = false;
          }
        });
        
        // Add all animations to the timeline
        animateOptimalRole(breedData[breedId].optimal, masterTimeline);
        animateBreedDescription(breedData[breedId].description, masterTimeline);
        animateDogImage(breedData[breedId].imageSrc, masterTimeline);
        
        // Update radar chart with the new breed data
        updateRadarChart(breedId);
      } catch (error) {
        console.error("Error updating breed:", error);
        isAnimating.value = false; // Reset animation state if there's an error
      }
    }

    // Add a function to set up scroll-triggered animations
    function setupScrollAnimations() {
      if (animationsInitialized) return; // Prevent re-initializing animations
      
      // Set initial states
      gsap.set(filtersElement.value, { opacity: 0, y: -30 });
      gsap.set(dogImageElement.value, { opacity: 0, scale: 0.8 });
      gsap.set(breedRoleLabelElement.value, { opacity: 0, x: -30 });
      gsap.set(breedNameElement.value, { opacity: 0, x: -50 });
      gsap.set(breedDescriptionElement.value, { opacity: 0, y: 30 });
      gsap.set(chartContainerElement.value, { opacity: 0, scale: 0.9 });
      
      // Create the timeline
      const tl = gsap.timeline({
        scrollTrigger: {
          trigger: breedSection.value,
          start: "top 80%",
          end: "top 20%",
          toggleActions: "play none none none"
        }
      });
      
      // Add animations to timeline
      tl.to(filtersElement.value, { opacity: 1, y: 0, duration: 0.6, ease: "power2.out" })
        .to(dogImageElement.value, { opacity: 1, scale: 1, duration: 0.8, ease: "back.out(1.2)" }, "-=0.3")
        .to(breedRoleLabelElement.value, { opacity: 1, x: 0, duration: 0.5 }, "-=0.5")
        .to(breedNameElement.value, { opacity: 1, x: 0, duration: 0.6, ease: "power2.out" }, "-=0.4")
        .to(breedDescriptionElement.value, { opacity: 1, y: 0, duration: 0.6 }, "-=0.4")
        .to(chartContainerElement.value, { opacity: 1, scale: 1, duration: 0.8, ease: "back.out(1)" }, "-=0.6");
      
      animationsInitialized = true;
    }

    // Watch for changes in the dogDisplayMode prop
    watch(() => props.dogDisplayMode, (newMode) => {
      // Force update the image when the mode changes
      const imageElement = dogImageElement.value?.querySelector('img');
      if (imageElement) {
        // Create a small animation to transition the image change
        gsap.to(dogImageElement.value, {
          opacity: 0,
          duration: 0.3,
          onComplete: () => {
            imageElement.src = getImageUrl(breedData[currentBreed.value].imageSrc);
            gsap.to(dogImageElement.value, {
              opacity: 1,
              duration: 0.3
            });
          }
        });
      }
    }, { immediate: true });

    onMounted(() => {
      // Initialize the chart with the first breed
      initRadarChart();
      
      // Set up scroll animations after a slight delay to ensure DOM is ready
      setTimeout(() => {
        setupScrollAnimations();
      }, 100);
    });

    onBeforeUnmount(() => {
      // Clean up D3 elements to prevent memory leaks
      if (svgElement) {
        svgElement.remove();
        svgElement = null;
      }
      
      // Kill any active ScrollTrigger instances
      ScrollTrigger.getAll().forEach(trigger => trigger.kill());
    });

    return {
      isAnimating,
      breedSection,
      filtersElement,
      radarChart,
      dogImageElement,
      breedRoleLabelElement,
      breedNameElement,
      breedDescriptionElement,
      chartContainerElement,
      currentBreed,
      updateBreed,
      breedData,
      getImageUrl
    };
  }
};
</script>

<style scoped>
.breed-strength-section {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  height: 100vh; /* Ensure it takes full viewport height */
  display: flex;
  flex-direction: column;
  position: relative; /* Add this to create positioning context */
  isolation: isolate; /* Add this for stacking context isolation */
  scroll-snap-align: start; /* For section scrolling */
  scroll-snap-stop: always;
  margin-bottom: 5vh;
}

.filters {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 40px;
  margin-top: 40px;
}

.filter-button {
  font-family: "Neue Haas Grotesk Display Pro";
  font-size: 20px;
  font-style: normal;
  font-weight: 400;
  background-color: #F2EFEB;
  border: 1.5px solid #000;  border-radius: 20px;
  padding: 8px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}


.filter-button:hover {
  background-color: rgba(139, 206, 229, 0.30); /* background on hover */
}

.filter-button.active {
  background-color: #81C9E3;
}

/* Update breed-container to ensure it stays within its section */
.breed-container {
  display: flex;
  justify-content: space-between;
  margin-top: 30px;
  min-height: 0;
  position: relative; /* Add this */
  z-index: 1; /* Ensure proper stacking */
}

/* Update breed-info to ensure proper positioning */
.breed-info {
  flex: 1;
  padding-right: 60px;
  padding-top: 40px;
  display: flex;
  flex-direction: column;
  position: relative; /* Add this */
}

.popular-breed-role {
  color: #000;
font-family: Roca;
font-size: 24px;
font-style: normal;
font-weight: 400;
  text-align: left; /* Change from center to left */
}

.breed-role {
  margin-bottom: 5px;
  color: #F00;
font-family: "IvyPresto Headline";
font-size: 70px;
font-style: italic;
font-weight: 400;
  text-align: left; /* Change from center to left */
  position: relative;
  overflow: hidden;
}

/* Hide the old optimal-role class */
.optimal-role {
  display: none;
}

.dog-image {
  width: 600px;
  height: 400px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
}

.chart-container {
  flex: 1;
  position: relative;
}

.radar-tooltip {
  position: absolute;
  background-color:white;
  opacity: 0.5;
  border-radius: 5px;
  padding: 5px 10px;
  font-size: 14px;
  pointer-events: none;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
  .breed-container {
    flex-direction: column;
  }
  
  .dog-image {
    width: 100%;
    height: 300px;
  }
}
.breed-description {
  color: #000;
  font-family: "Neue Haas Grotesk Display Pro";
  font-size: 16px;
  font-style: normal;
  font-weight: 400;
  line-height: normal;

}
.chart-source {
  position: absolute;
  right: 0;
  bottom: 0;
  color: #A2A2A2;
  font-family: "Neue Haas Grotesk Display Pro";
  font-size: 12px;
  font-style: normal;
  font-weight: 400;
  line-height: normal;
  padding: 10px 16px 8px 16px;
  background: transparent;
  pointer-events: none;
  z-index: 2;
}

.chart-source a {
  color: inherit;
  text-decoration: none;
  pointer-events: auto;
  transition: text-decoration 0.2s;
  color: #666666;
  text-decoration: none;
  border-bottom: 1px dotted #999999;
  transition: color 0.2s ease;
}

.chart-source a:hover {
  text-decoration: underline;
}

/* Add these styles instead */
.tooltip-group {
  pointer-events: none;
  filter: drop-shadow(0px 2px 4px rgba(0,0,0,0.15));
}

/* Add this to improve animation smoothness */
.breed-info > div, .chart-container, .filters {
  will-change: transform, opacity;
}
</style>