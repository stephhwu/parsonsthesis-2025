<template>
  <div class="sniffer-section">
    <div class="scrollama-container">
      <div class="step" data-step="1" ref="step">
        <h2 class="step-heading" ref="heading">
          <span class="line-wrapper"><span class="line">WHY SO MANY</span></span>
          <span class="line-wrapper"><span class="line">SNIFFERS?</span></span>
          <span class="line-wrapper"><span class="line">DOGS</span></span>
          <span class="line-wrapper dog-line">
            <span class="line">HAVE</span>
          </span>
          <!-- Place image outside the line-wrapper -->
          <img src="/src/assets/images/dog-nose.png" alt="Dog nose" class="dog-nose-image">
          <span class="line-wrapper"><span class="line highlight">40x</span></span>
          <span class="line-wrapper"><span class="line">THE OLFACTORY</span></span>
          <span class="line-wrapper"><span class="line">RECEPTORS OF</span></span>
          <span class="line-wrapper"><span class="line">HUMANS</span></span>
        </h2>
        <div class="dots-container" ref="combinedDots"></div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import scrollama from 'scrollama';
import { gsap } from 'gsap';
import * as d3 from 'd3';

export default {
  name: 'SnifferSection',
  setup() {
    const step = ref(null);
    const heading = ref(null);
    const combinedDots = ref(null);
    
    let scroller = null;
    
    // Adjust these values to match the screenshot
    const humanDotCount = 6; 
    const dogDotCount = 250;  // 5x more than humans
    
    const animateLineReveal = (headingRef) => {
      if (!headingRef) return;
      
      gsap.fromTo(
        headingRef.querySelectorAll('.line'),
        { 
          y: "100%",
          opacity: 0 
        },
        { 
          y: "0%",
          opacity: 1,
          stagger: 0.15,
          duration: 0.7,
          ease: "power2.out"
        }
      );
      
      const highlightEl = headingRef.querySelector('.highlight');
      if (highlightEl) {
        gsap.fromTo(
          highlightEl,
          { 
            scale: 0.5, 
            opacity: 0 
          },
          { 
            scale: 1,
            opacity: 1,
            delay: 0.5,
            duration: 0.8,
            ease: "elastic.out(1, 0.5)"
          }
        );
      }
    };
    
    const createCombinedVisualization = () => {
      if (!combinedDots.value) return;
      
      const container = combinedDots.value;
      const width = container.offsetWidth;
      const height = container.offsetHeight;
      
      // Clear container
      container.innerHTML = '';
      
      // Create SVG element to fill the container
      const svg = d3.select(container)
        .append('svg')
        .attr('width', width)
        .attr('height', height);
      
      // Set a consistent dot size for both types
      const dotRadius = 22; // Fixed size for all dots
      
      // Generate human dots data - place them in the bottom right corner
      const humanData = Array.from({ length: humanDotCount }, () => ({
        x: width * 0.75 + Math.random() * width * 0.2,
        y: height * 0.2 + Math.random() * height * 0.2,
        r: dotRadius, // Using fixed size
        type: 'human'
      }));
      
      // Generate dog dots data - initially offscreen to right
      const dogData = Array.from({ length: dogDotCount }, () => ({
        x: width + Math.random() * 100,
        y: Math.random() * height,
        r: dotRadius, // Using same fixed size
        type: 'dog'
      }));
      
      // Combine data
      const allData = [...humanData, ...dogData];
      
      // Create circles with higher opacity and blend mode
      const circles = svg.selectAll('circle')
        .data(allData)
        .enter()
        .append('circle')
        .attr('class', d => d.type === 'human' ? 'human-circle' : 'dog-circle')
        .attr('cx', d => d.x)
        .attr('cy', d => d.y)
        .attr('r', dotRadius) // Using direct value instead of d.r for consistency
        .attr('fill', d => d.type === 'human' ? '#FF0000' : '#81C9E3') // Exact colors from screenshot
        .attr('opacity', 0) // Start both invisible
        .attr('fill-opacity', 0.99) // Higher opacity to match screenshot
        .style('mix-blend-mode', 'multiply'); // Add blend mode for overlapping effect
      
      // Update the force simulation to use the fixed radius
      const simulation = d3.forceSimulation(allData)
        .force('charge', d3.forceManyBody().strength(-10))
        .force('collide', d3.forceCollide().radius(dotRadius + 2).iterations(4))
        .force('x', d3.forceX(d => {
          // Position dogs on right side, humans on bottom right
          return d.type === 'human' ? width * 0.75 : width * 0.55;
        }).strength(0.05))
        .force('y', d3.forceY(d => {
          // Position human dots lower
          return d.type === 'human' ? height * 0.75 : height * 0.5;
        }).strength(0.05))
        .on('tick', () => {
          circles
            .attr('cx', d => d.x = Math.max(dotRadius, Math.min(width - dotRadius, d.x)))
            .attr('cy', d => d.y = Math.max(dotRadius, Math.min(height - dotRadius, d.y)));
        });
      
      // Rest of the function remains the same
      
      // Store simulation and references
      container._simulation = simulation;
      container._humanCircles = svg.selectAll('.human-circle');
      container._dogCircles = svg.selectAll('.dog-circle');
      
      // Initially pause simulation
      simulation.stop();
      
      return { svg, circles, simulation };
    };
    
    const animateVisualization = () => {
      if (!combinedDots.value || !combinedDots.value._simulation) return;
      
      const container = combinedDots.value;
      const simulation = container._simulation;
      
      // Step 1: Fade in human circles first
      gsap.to('.human-circle', {
        opacity: 0.9,
        stagger: 0.01,
        duration: 0.5,
        onComplete: () => {
          // Step 2: Start simulation after human circles appear
          simulation.alpha(1).restart();
          
          // Step 3: After a delay, bring in dog circles
          setTimeout(() => {
            gsap.to('.dog-circle', {
              opacity: 0.9,
              stagger: 0.002,
              duration: 0.8,
              onStart: () => {
                // Apply stronger force when dog dots appear
                simulation.force('collide', d3.forceCollide().radius(d => d.r + 2).iterations(4));
                
                // Push from right to left
                simulation.force('push', d3.forceX(d => {
                  return d.type === 'dog' ? 0.5 * container.offsetWidth : 0.7 * container.offsetWidth;
                }).strength(0.2));
              }
            });
          }, 1000);
          
          // After animation completes, reduce the force
          setTimeout(() => {
            simulation.force('push', null);
          }, 3000);
        }
      });
    };
    
    const initScrollama = () => {
      scroller = scrollama();
      
      scroller
        .setup({
          step: '.step',
          offset: 0.5,
          debug: false
        })
        .onStepEnter(response => {
          response.element.classList.add('active');
          animateLineReveal(heading.value);
          
          // Create and animate visualization only when scrolled into view
          setTimeout(() => {
            if (!combinedDots.value._initialized) {
              createCombinedVisualization();
              combinedDots.value._initialized = true;
              
              setTimeout(() => {
                animateVisualization();
              }, 800);
            }
          }, 500);
        })
        .onStepExit(response => {
          response.element.classList.remove('active');
        });
    };
    
    onMounted(() => {
      // Set initial state for animation
      if (heading.value) {
        gsap.set(heading.value.querySelectorAll('.line'), { 
          y: "100%",
          opacity: 0
        });
        
        if (heading.value.querySelector('.highlight')) {
          gsap.set(heading.value.querySelector('.highlight'), {
            scale: 0.5,
            opacity: 0
          });
        }
      }
      
      // Initialize scrollama
      initScrollama();
      
      // REMOVED: Automatic animation start without scrolling
      // Now animations will only trigger when scrolling to section
      
      // Handle window resize
      const handleResize = () => {
        scroller.resize();
        
        // Reset visualization on resize
        if (combinedDots.value && combinedDots.value._initialized) {
          combinedDots.value._initialized = false;
          if (combinedDots.value._simulation) {
            combinedDots.value._simulation.stop();
            combinedDots.value._simulation = null;
          }
          combinedDots.value.innerHTML = '';
          
          // Recreate after resize
          createCombinedVisualization();
          combinedDots.value._initialized = true;
          animateVisualization();
        }
      };
      
      window.addEventListener('resize', handleResize);
      
      // Cleanup
      onBeforeUnmount(() => {
        window.removeEventListener('resize', handleResize);
        if (scroller) scroller.destroy();
        
        if (combinedDots.value && combinedDots.value._simulation) {
          combinedDots.value._simulation.stop();
        }
      });
    });
    
    return {
      step,
      heading,
      combinedDots
    };
  }
};
</script>

<style scoped>
/* Add this new class definition */
.ivy-italic {
  font-family: ivypresto-headline, serif;
  font-style: italic;
  font-weight: 600;
}

.sniffer-section {
  position: relative;
  width: 100%;
  min-height: 100vh;
  background-color: #F2EFEB;
}

.scrollama-container {
  position: relative;
  width: 100%;
  padding: 0;
}

.step {
  position: relative;
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  opacity: 1;
  transition: opacity 0.5s;
  overflow: hidden;
}

.step-heading {
  font-family: "neue-haas-grotesk-display", sans-serif;
  font-weight: 700;
  font-style: normal;
  color: red;
  font-size: 48px;
  text-align: left;
  line-height: 1.1;
  z-index: 10;
  position: absolute;
  top: 35%;
  left: 10%;
  transform: translateY(-50%);
  width: 30%;
  max-width: 400px;
  margin: 0;
  text-transform: uppercase;
}

/* Text animation styles */
.line-wrapper {
  display: block;
  overflow: hidden;
  line-height: 1.1;
  position: relative;
}

.line {
  display: block;
  transform-origin: center bottom;
}

.dots-container {
  position: absolute;
  padding-left: 300px;
  top: 0;
  right: 3;
  width: 70%;
  height: 70vh;
  background-color: #F2EFEB;
}

.highlight {
  font-family: ivypresto-headline, serif;
  font-weight: 600;
  font-style: italic;
  color: #2FA6D2;
  font-size: 170px;
  line-height: 0.9;
  display: inline-block;
  transform-origin: center;
  position: relative;
  text-transform: none;
}

.dog-line {
  position: relative; /* Ensure this line can be positioned relatively */
}

.dog-nose-image {
  position: absolute;
  height: 147px;
  width: 211px;
  left: 140px;  /* Position based on the heading, not its parent */
  top: 70px;    /* Align with "HAVE" text */
  z-index: 11;
  pointer-events: none;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .step-heading {
    font-size: 36px;
    position: relative;
    top: 20%;
    left: 5%;
    transform: none;
    width: 90%;
    margin-bottom: 30px;
  }
  
  .highlight {
    font-size: 100px;
  }
  
  .dog-nose-image {
    width: 150px;
    right: -160px; /* Adjust for smaller screens */
    top: -15px;
  }
}
</style>