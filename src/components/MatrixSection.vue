<template>
  <div class="matrix-container">
    <div class="fixed-container" ref="fixedContainer">
      <h1 class="header">Some Cool Dog Jobs</h1>
      
      <!-- Scroll indicator -->
      <div class="scroll-indicator" ref="scrollIndicator">
        <span>scroll to explore</span>
        <div class="arrow-down"></div>
      </div>
      
      <!-- Category Labels -->
      <div class="category-label" id="sniffers" ref="sniffersLabel">sniffers</div>
      <div class="category-label" id="guides" ref="guidesLabel">guides</div>
      <div class="category-label" id="medical" ref="medicalLabel">medical</div>
      <div class="category-label" id="misc" ref="miscLabel">blue collar & etc.</div>
      
      <!-- Axes -->
      <div class="axis horizontal-axis" ref="horizontalAxis"></div>
      <div class="axis vertical-axis" ref="verticalAxis"></div>
      
      <div class="hover-instruction" ref="hoverInstruction">hover to explore popular barking orders</div>
      
      <div class="highlighted-job" ref="highlightedJob">
        <div class="job-content-wrapper">
          <div class="job-title">{{ selectedJob.name }}</div>
          <div class="job-description">{{ selectedJob.description || 'No description available' }}</div>
        </div>
      </div>
      
      <!-- Replace circles with images -->
      <div 
        v-for="(circle, index) in visibleCircles" 
        :key="`circle-${circle.id}`"
        :id="`circle-${circle.id}`"
        class="job-icon" 
        :data-category="circle.category"
        :style="getCircleStyles(circle)"
        @mouseenter="showJobDetails(circle, $event)" 
        @mouseleave="hideJobDetails"
      >
        <img :src="getCategoryImage(circle.category)" :alt="circle.category">
        <div class="job-label" v-if="showLabels">{{ circle.job.name }}</div>
      </div>
      
      <!-- Toggle for labels -->
      <div class="label-toggle" ref="labelToggle">
        <label>
          <input type="checkbox" v-model="showLabels">
          Show all job names
        </label>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, onBeforeUnmount, ref, reactive, computed } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

// Import category images
import sniffersImage from '@/assets/images/sniffers.png';
import guidesImage from '@/assets/images/guides.png';
import medicalImage from '@/assets/images/medical.png';
import miscImage from '@/assets/images/misc.png';

gsap.registerPlugin(ScrollTrigger);

export default {
  name: 'MatrixSection',
  setup() {
    // Add new ref to track if hover is enabled
    const hoverEnabled = ref(false);
    
    // Existing refs
    const fixedContainer = ref(null);
    const horizontalAxis = ref(null);
    const verticalAxis = ref(null);
    const highlightedJob = ref(null);
    const sniffersLabel = ref(null);
    const guidesLabel = ref(null);
    const medicalLabel = ref(null);
    const miscLabel = ref(null);
    const hoverInstruction = ref(null);
    const scrollIndicator = ref(null);
    const jobCount = ref(null);
    const labelToggle = ref(null);
    
    const circles = ref([]);
    const scrollProgress = ref(0);
    const scrollTrigger = ref(null);
    const showLabels = ref(false);
    
    // Instead, just show all circles:
    const visibleCircles = computed(() => circles.value);
    
    // Store selected job info
    const selectedJob = ref({ name: '', description: '' });
    
    // Define dog jobs by category - using expanded dataset
    const dogJobs = {
      sniffers: [
        { name: "lantern fly hunting", description: "Detects invasive lanternflies to support ecological control efforts." },
        { name: "drug detection", description: "Finds illegal drugs at borders, airports, and crime scenes." },
        { name: "search and rescue", description: "Locates missing persons in disaster zones or wilderness areas." },
        { name: "truffle hunting", description: "Sniffs out valuable truffles underground for culinary use." },
        { name: "bomb detection", description: "Identifies explosives in high-risk environments." },
        { name: "cadaver detection", description: "Locates human remains during investigations or disasters." },
        { name: "termite detection", description: "Finds termites and infestations hidden within structures." },
        { name: "wildlife tracking", description: "Tracks endangered or invasive wildlife via scat and scent." },
        { name: "currency detection", description: "Detects large amounts of hidden or smuggled cash." },
        { name: "electronics detection", description: "Sniffs out hidden electronics like phones or SD cards." },
        { name: "scat detection", description: "Finds animal droppings for wildlife research and conservation." },
        { name: "food allergen detection", description: "Detects allergens like peanuts in food for safety." },
        { name: "trailing", description: "Follows a specific scent trail to locate individuals." },
        { name: "firearms detection", description: "Sniffs out guns and gunpowder at crime scenes or checkpoints." },
        { name: "biodiversity monitoring", description: "Locates rare species to support conservation surveys." },
        { name: "water quality detection", description: "Detects pollution or toxins in bodies of water." },
        { name: "gas leak detection", description: "Identifies hazardous gas leaks for utility companies." },
        { name: "contraband detection", description: "Finds smuggled items at airports and borders." },
        { name: "mold detection", description: "Sniffs out mold hidden in buildings or homes." },
        { name: "cheese quality control", description: "Used to ensure cheese products meet scent standards." },
        { name: "disease detection", description: "Identifies illnesses like COVID or cancer through scent." },
        { name: "bee disease detection", description: "Detects infections in bee colonies." },
        { name: "arson detection", description: "Finds accelerants at suspected arson crime scenes." },
        { name: "pregnancy detection", description: "Reportedly detects hormonal changes related to pregnancy." },
        { name: "whale detection", description: "Sniffs whale feces on boats for marine biology research." },
        { name: "crop protection", description: "Detects early pest or disease threats in agriculture." },
        { name: "invasive species control", description: "Sniffs out invasive pests like brown tree snakes." },
        { name: "habitat restoration", description: "Helps locate species for environmental restoration planning." },
        { name: "wine inspectors", description: "Sniff out cork taint (TCA) in wine production." }

      ],
      guides: [
        { name: "guide dogs", description: "Help visually impaired individuals navigate the world safely." },
        { name: "therapy dogs", description: "Provide emotional support in hospitals or disaster areas." },
        { name: "service dogs", description: "Assist with daily tasks for people with physical disabilities." },
        { name: "emotional support", description: "Offer comfort for individuals with anxiety or mental health needs." },
        { name: "mobility assistance", description: "Support those with mobility impairments or balance issues." },
        { name: "hearing dogs", description: "Alert deaf or hard-of-hearing people to important sounds." },
        { name: "autism support", description: "Help individuals with autism by offering comfort and routine." },
        { name: "PTSD support", description: "Assist veterans and others with post-traumatic stress disorder." },
        { name: "dual-role service dogs", description: "Trained to assist with multiple disabilities or impairments." },
        { name: "alzheimer's assistance", description: "Help individuals with memory loss navigate safely." }
      ],
      medical: [
        { name: "seizure alert", description: "Detects and warns of impending epileptic seizures." },
        { name: "diabetes alert", description: "Alerts handlers to blood sugar level changes." },
        { name: "cancer detection", description: "Sniffs out cancers through breath or stool samples." },
        { name: "blood pressure alert", description: "Notifies handlers of dangerous blood pressure changes." },
        { name: "migraine alert", description: "Senses and alerts to the onset of migraines." },
        { name: "allergy detection", description: "Detects allergens like peanuts or gluten in environments." },
        { name: "hospital therapy", description: "Provides psychological comfort to patients during hospital stays." },
        { name: "rehabilitation support", description: "Aids in recovery for those healing from physical or emotional trauma." },
        { name: "cardiac alert", description: "Detects signs of heart rate irregularities or cardiac events." },
        { name: "neurological alert", description: "Senses subtle changes before seizures or narcoleptic episodes." }
      ],
      misc: [
        { name: "herding", description: "Manages livestock such as sheep, cattle, or reindeer." },
        { name: "police K9", description: "Assists officers with detection, pursuit, and protection tasks." },
        { name: "movie dogs", description: "Trained for acting roles in TV and film productions." },
        { name: "sled dogs", description: "Pull sleds in snowy environments for transport or sport." },
        { name: "military working dogs", description: "Serve in combat zones for tracking, detection, or patrol." },
        { name: "farm dogs", description: "Assist with various farm duties including guarding and herding." },
        { name: "dock diving", description: "Compete in long-distance water jumping sports." },
        { name: "agility competition", description: "Run timed obstacle courses in competitive events." },
        { name: "carting/draft dogs", description: "Pull carts or equipment historically or in competition." },
        { name: "hunting dogs", description: "Assist hunters by pointing, flushing, or retrieving game." },
        { name: "guard dogs", description: "Protect homes, livestock, or properties from intruders." },
        { name: "firehouse dogs", description: "Traditionally lived with firefighters and helped clear paths." },
        { name: "mascots", description: "Serve as symbolic representatives or morale boosters in teams or stations." },
        { name: "penguin protectors", description: "Guard endangered penguins from predators." },
        { name: "goose patrol", description: "Deter nuisance geese from golf courses or airports." },
        { name: "reindeer herders", description: "Used by Arctic communities to herd reindeer." },
        { name: "sports retrieval dogs", description: "Retrieve balls or equipment in sports like golf or tennis." }
      ]
    };
    
    // Generate balanced circle data with the full dataset
    function generateCircles() {
      const generatedCircles = [];
      const categories = Object.keys(dogJobs);
      
      // Create random but visually interesting initial positions
      // Use a dispersed grid pattern to avoid initial clustering
      const columns = 12;
      const rows = 12;
      const cellWidth = 100 / columns;
      const cellHeight = 100 / rows;
      
      // Prepare a grid for position tracking
      const usedPositions = Array(rows * columns).fill(false);
      
      // Maximum attempts to find an unused position
      const maxAttempts = 100;
      
      categories.forEach(category => {
        const jobs = dogJobs[category];
        
        jobs.forEach((job, jobIndex) => {
          // Give each job a unique ID
          const id = `${category}-${jobIndex}`;
          
          // Find an unused grid position
          let position;
          let attempts = 0;
          
          do {
            // Get a random grid cell
            position = Math.floor(Math.random() * (rows * columns));
            attempts++;
            
            // If we've tried too many times, just use any position
            if (attempts > maxAttempts) break;
          } while (usedPositions[position]);
          
          // Mark this position as used
          usedPositions[position] = true;
          
          // Convert grid position to x,y percentages
          const col = position % columns;
          const row = Math.floor(position / columns);
          
          const x = col * cellWidth + (Math.random() * cellWidth * 0.5);
          const y = row * cellHeight + (Math.random() * cellHeight * 0.5);
          
          generatedCircles.push({
            id: id,
            size: 55, // Default size for consistency
            category: category,
            job: job,
            initialX: x,
            initialY: y,
            finalX: 0, // Will be calculated later
            finalY: 0, // Will be calculated later
            hover: false
          });
        });
      });
      
      return generatedCircles;
    }
    
    // Calculate styles based on current scroll progress
    const getCircleStyles = (circle) => {
      const progress = scrollProgress.value;
      
      // Calculate current position based on progress
      const currentX = circle.initialX + (circle.finalX - circle.initialX) * progress;
      const currentY = circle.initialY + (circle.finalY - circle.initialY) * progress;
      
      // Calculate z-index to ensure hovered circles appear on top
      const zIndex = circle.hover ? 100 : 10;
      
      // Dynamic sizing based on hover state for better tap/click targets
      const size = circle.hover ? '70px' : '55px';
      
      return {
        width: size,
        height: size,
        left: `${currentX}%`,
        top: `${currentY}%`,
        transform: circle.hover ? 'translate3d(0,0,0) scale(1.2)' : 'translate3d(0,0,0) scale(1)', 
        zIndex: zIndex,
        willChange: 'transform, left, top', // Hint to browser about properties that will change
        transition: circle.hover ? 'transform 0.3s cubic-bezier(0.22, 1, 0.36, 1)' : 'none' // Only apply transition on hover
      };
    };
    
    // Set up the quadrant system with more spread for larger dataset
    function setupQuadrants() {
      if (!fixedContainer.value) return;
      
      // Map categories to quadrants with explicit coordinates (percentages)
      const quadrants = {
        'sniffers': { x: 25, y: 25 },
        'guides': { x: 75, y: 25 },
        'medical': { x: 25, y: 75 },
        'misc': { x: 75, y: 75 }
      };
      
      // Calculate optimal distribution using a spiral pattern algorithm
      // This provides more space between points and minimizes overlapping
      circles.value.forEach((circle, index) => {
        const quadrant = quadrants[circle.category];
        const categoryJobs = dogJobs[circle.category];
        
        // Get index of the current job in its category
        const jobIndex = categoryJobs.findIndex(job => job.name === circle.job.name);
        
        // Calculate spiral position
        // This creates a fibonacci-like spiral pattern which naturally spaces elements
        const itemCount = categoryJobs.length;
        const angleStep = 2.4; // Controls how quickly the spiral expands
        const radiusStep = 0.2; // Controls distance between adjacent points
        
        // Calculate angle and radius based on item index
        const angle = jobIndex * angleStep;
        const radius = Math.sqrt(jobIndex) * radiusStep;
        
        // Convert to X,Y coordinates (percentage-based)
        // Using polar to cartesian conversion (r,θ) → (r·cos θ, r·sin θ)
        const spreadX = radius * Math.cos(angle) * 15; // 15% quadrant spread
        const spreadY = radius * Math.sin(angle) * 15; // 15% quadrant spread
        
        // Calculate final position, centering in the quadrant
        // The quadrant center is at (quadrant.x, quadrant.y)
        circle.finalX = quadrant.x + spreadX;
        circle.finalY = quadrant.y + spreadY;
        
        // Apply boundary constraints to keep all points within their quadrants
        // These limits ensure points don't cross the axes or go out of bounds
        const axisLimit = 5; // Minimum distance from axes (percentage)
        const edgeLimit = 40; // Maximum distance from quadrant center (percentage)
        
        // X-axis constraint (horizontal)
        if (circle.category === 'sniffers' || circle.category === 'medical') {
          // Left quadrants: constrain between 0% and 45%
          circle.finalX = Math.max(axisLimit, Math.min(circle.finalX, 45));
        } else {
          // Right quadrants: constrain between 55% and 100%
          circle.finalX = Math.max(55, Math.min(circle.finalX, 100 - axisLimit));
        }
        
        // Y-axis constraint (vertical)
        if (circle.category === 'sniffers' || circle.category === 'guides') {
          // Top quadrants: constrain between 0% and 45%
          circle.finalY = Math.max(axisLimit, Math.min(circle.finalY, 45));
        } else {
          // Bottom quadrants: constrain between 55% and 100%
          circle.finalY = Math.max(55, Math.min(circle.finalY, 100 - axisLimit));
        }
      });
      
      // Update category label positions
      if (sniffersLabel.value) sniffersLabel.value.style.left = '25%';
      if (sniffersLabel.value) sniffersLabel.value.style.top = '25%';
      if (guidesLabel.value) guidesLabel.value.style.left = '75%';
      if (guidesLabel.value) guidesLabel.value.style.top = '25%';
      if (medicalLabel.value) medicalLabel.value.style.left = '25%';
      if (medicalLabel.value) medicalLabel.value.style.top = '75%';
      if (miscLabel.value) miscLabel.value.style.left = '75%';
      if (miscLabel.value) miscLabel.value.style.top = '75%';
      
      // Set axis positions
      if (horizontalAxis.value) horizontalAxis.value.style.top = '50%';
      if (verticalAxis.value) verticalAxis.value.style.left = '50%';
    }

    // Modify the showJobDetails function to check if hover is enabled
    function showJobDetails(circle, event) {
      // Don't show details if hover isn't enabled yet
      if (!highlightedJob.value || !hoverEnabled.value) return;
      
      // Reset hover state for all circles first
      circles.value.forEach(c => c.hover = false);
      
      // Update circle hover state
      circle.hover = true;
      
      // Update selected job data
      selectedJob.value = circle.job;
      
      // Get parent container's reference for positioning
      const containerRect = fixedContainer.value.getBoundingClientRect();
      
      // Position the popup next to the circle
      // Use the actual element that triggered the event (the image inside the div)
      const targetElement = event.target.closest('.job-icon');
      const rect = targetElement.getBoundingClientRect();
      const jobElement = highlightedJob.value;
      
      // Calculate the best position for the popup
      // Keep it within the viewport bounds
      const viewportWidth = window.innerWidth;
      const viewportHeight = window.innerHeight;
      
      // Calculate center point of the circle for better popup alignment
      const circleCenterX = rect.left + (rect.width / 2);
      const circleCenterY = rect.top + (rect.height / 2);
      
      // Calculate quadrant to determine best popup position
      // This ensures the popup opens away from the center axes
      let leftPos, topPos;
      
      // Determine which quadrant the circle is in
      const isLeftSide = circleCenterX < containerRect.width / 2;
      const isTopSide = circleCenterY < containerRect.height / 2;
      
      if (isLeftSide) {
        // Place popup to the right of the circle
        leftPos = rect.right + 15;
      } else {
        // Place popup to the left of the circle
        leftPos = rect.left - jobElement.offsetWidth - 15;
      }
      
      if (isTopSide) {
        // Align with top if in top half
        topPos = rect.top;
      } else {
        // Align with bottom if in bottom half
        topPos = rect.top - jobElement.offsetHeight + rect.height;
      }
      
      // Final boundary checks
      if (leftPos < 10) leftPos = 10;
      if (leftPos + jobElement.offsetWidth > viewportWidth - 10) {
        leftPos = viewportWidth - jobElement.offsetWidth - 10;
      }
      
      if (topPos < 10) topPos = 10;
      if (topPos + jobElement.offsetHeight > viewportHeight - 10) {
        topPos = viewportHeight - jobElement.offsetHeight - 10;
      }
      
      // Apply smooth animation
      gsap.to(jobElement, {
        left: leftPos,
        top: topPos,
        opacity: 1,
        duration: 0.2,
        ease: "power2.out"
      });
    }

    // Also update the hideJobDetails function
    function hideJobDetails() {
      if (!highlightedJob.value || !hoverEnabled.value) return;
      
      // Reset hover state for all circles
      circles.value.forEach(c => c.hover = false);
      
      // Hide popup with animation
      gsap.to(highlightedJob.value, {
        opacity: 0,
        duration: 0.2,
        ease: "power2.in"
      });
    }

    function setupScrollAnimation() {
      // Create scroll trigger animation
      scrollTrigger.value = ScrollTrigger.create({
        trigger: '.matrix-container',
        start: 'top top',
        end: 'bottom+=1000 top',
        scrub: 0.5, // Add small amount of smoothing to reduce jerky movement
        pin: true,
        onUpdate: (self) => {
          // Update scroll progress value
          scrollProgress.value = self.progress;
          
          // First phase: Fade out header text and scroll indicator (0-0.4 progress)
          const headerPhase = Math.min(scrollProgress.value * 2.5, 1);
          const headerOpacity = 1 - headerPhase;
          
          const header = document.querySelector('.matrix-container .header');
          const scrollIndicatorElement = document.querySelector('.matrix-container .scroll-indicator');
          
          if (header && scrollIndicatorElement) {
            header.style.opacity = headerOpacity;
            scrollIndicatorElement.style.opacity = headerOpacity;
          }
          
          // Second phase: Show matrix and move circles (0.4-0.8 progress)
          const matrixPhase = Math.max(scrollProgress.value * 2.5 - 1, 0);
          const matrixOpacity = matrixPhase > 1 ? 1 : matrixPhase;
          
          if (horizontalAxis.value && verticalAxis.value) {
            horizontalAxis.value.style.opacity = matrixOpacity;
            verticalAxis.value.style.opacity = matrixOpacity;
          
            document.querySelectorAll('.category-label').forEach(label => {
              label.style.opacity = matrixOpacity;
            });
          }
          
          // Third phase: Show UI controls (0.8-1.0 progress)
          const controlsPhase = Math.max((scrollProgress.value - 0.7) * 3.3, 0);
          const controlsOpacity = controlsPhase > 1 ? 1 : controlsPhase;
          
          if (hoverInstruction.value) {
            hoverInstruction.value.style.opacity = controlsOpacity;
            if (controlsPhase > 0 && controlsPhase < 1) {
              hoverInstruction.value.style.transform = `translateY(${20 - (controlsPhase * 20)}px)`;
            } else if (controlsPhase >= 1) {
              hoverInstruction.value.style.transform = 'translateY(0)';
            }
          }
          
          // Show filter UI and job count
          if (jobCount.value) {
            jobCount.value.style.opacity = controlsOpacity;
          }
          
          if (labelToggle.value) {
            labelToggle.value.style.opacity = controlsOpacity;
          }
          
          // Enable/disable hover functionality based on the controlsPhase
          // Only enable hover when the instructions start to appear
          hoverEnabled.value = controlsOpacity > 0.2; // Add a small threshold
        }
      });
    }

    onMounted(() => {
      console.log('MatrixSection mounted');
      
      // Initialize with circle data
      circles.value = generateCircles();
      console.log(`Generated ${circles.value.length} circles`);
      
      // Setup quadrants once the component is mounted
      setupQuadrants();
      
      // Setup scroll animation
      setupScrollAnimation();
      
      // Add the GSAP animation for the scroll indicator
      gsap.to(".scroll-indicator", {
        y: 10,
        repeat: -1,
        yoyo: true,
        duration: 1.5
      });
      
      // Handle window resize
      window.addEventListener('resize', setupQuadrants);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('resize', setupQuadrants);
      if (scrollTrigger.value) {
        scrollTrigger.value.kill();
      }
    });

    // Function to get the correct image for each category
    function getCategoryImage(category) {
      const imageMap = {
        'sniffers': sniffersImage,
        'guides': guidesImage,
        'medical': medicalImage,
        'misc': miscImage
      };
      
      return imageMap[category];
    }

    return {
      circles,
      visibleCircles,
      fixedContainer,
      horizontalAxis,
      verticalAxis,
      highlightedJob,
      sniffersLabel,
      guidesLabel,
      medicalLabel,
      miscLabel,
      hoverInstruction,
      scrollIndicator,
      jobCount,
      labelToggle,
      getCircleStyles,
      showJobDetails,
      hideJobDetails,
      getCategoryImage,
      showLabels,
      selectedJob,
      hoverEnabled // Export this if you want to use it in the template
    };
  }
};
</script>

<style scoped>
.matrix-container {
  height: 200vh;
  width: 100%;
  position: relative;
}

.fixed-container {
  position: sticky;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.header {
  position: absolute;
  top: 10%;
  left: 0;
  width: 100%;
  text-align: center;
  font-size: 3rem;
  transition: opacity 0.5s;
  font-family: "ivypresto-headline", serif;
  color: #FF0000;
  font-weight: normal;
}

.scroll-indicator {
  position: absolute;
  top: 20%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  color: black;
  font-family: "ivypresto-headline", serif;
  transition: opacity 0.5s;
}

.scroll-indicator span {
  margin-bottom: 10px;
  font-size: 1rem;
}

.arrow-down {
  width: 12px;
  height: 12px;
  border-left: 2px solid black;
  border-bottom: 2px solid black;
  transform: rotate(-45deg);
}

.category-label {
  position: absolute;
  font-size: 3rem;
  opacity: 0;
  transition: opacity 0.5s;
  color: #FF0000;
  font-family: "ivypresto-headline", serif;
  font-style: italic;
  font-weight: normal;
  transform: translate(-50%, -50%);
}

.highlighted-job {
  position: fixed;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  padding: 20px;
  width: 240px;
  opacity: 0;
  pointer-events: none;
  z-index: 110;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  border: 1px solid rgba(240, 240, 240, 0.7);
  box-sizing: border-box;
}

.job-content-wrapper {
  width: 100%;
  box-sizing: border-box;
}

.job-title {
  color: #F00;
  font-family: "IvyPresto Headline", "ivypresto-headline", serif;
  font-size: 20px;
  font-weight: 400;
  margin: 0 0 12px 0;
  text-align: left;
  width: 100%;
  box-sizing: border-box;
}

.job-description {
  font-family: neue-haas-grotesk-display, sans-serif;
  font-weight: 500;
  font-style: normal;
  color: #000;
  font-size: 14px;
  font-weight: 500;
  line-height: 1.3;
  text-align: left;
  width: 100%;
  box-sizing: border-box;
  word-break: normal;
  word-wrap: break-word;
}

.axis {
  position: absolute;
  background-color: #000;
  opacity: 0;
  transition: opacity 0.5s;
}

.horizontal-axis {
  height: 1px;
  width: 80%;
  left: 10%;
  transform: translateY(-50%);
}

.vertical-axis {
  width: 1px;
  height: 70%;
  top: 20%;
  transform: translateX(-50%);
}

.job-icon {
  position: absolute;
  transition: all 0.5s cubic-bezier(0.22, 1, 0.36, 1);
  cursor: pointer;
  transform-origin: center center;
  /* Add a minimum touch/click target size */
  min-width: 40px;
  min-height: 40px;
  /* Add interactive shadow */
  filter: drop-shadow(0 0 0 rgba(0,0,0,0));
}

.job-icon img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  transition: all 0.3s ease;
  /* Make the image bigger relative to its container */
  padding: 5px;
}

.job-icon:hover {
  transform: scale(1.2);
  filter: drop-shadow(0 3px 6px rgba(0,0,0,0.2));
  z-index: 100;
}

/* Style for the job label that appears with the checkbox */
.job-label {
  position: absolute;
  top: 105%;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  font-size: 11px;
  background-color: rgba(255, 255, 255, 0.9);
  color: #333;
  padding: 3px 8px;
  border-radius: 12px;
  margin-top: 5px;
  pointer-events: none;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  font-weight: 600;
  /* Ensure labels are always above dots */
  z-index: 101;
  transition: opacity 0.2s ease;
}

/* Add this style for the new hover instruction */
.hover-instruction {
  position: absolute;
  top: 10%;
  left: 0;
  width: 100%;
  text-align: center;
  color: black;
  font-size: 18px;
  opacity: 0;
  transition: opacity 0.5s, transform 0.5s;
  font-family: "ivypresto-headline", serif;
  pointer-events: none;
}

/* Job count display */
.job-count {
  position: absolute;
  bottom: 20px;
  left: 20px;
  font-size: 14px;
  opacity: 0;
  transition: opacity 0.5s;
}

.job-count span {
  font-weight: bold;
}

/* Label toggle styles */
.label-toggle {
  font-family: neue-haas-grotesk-display, sans-serif;
font-weight: 500;
font-style: normal;
  position: absolute;
  bottom: 20px;
  right: 20px;
  opacity: 0;
  transition: opacity 0.5s;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 14px;
}

.label-toggle label {
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
}

.label-toggle input {
  margin: 0;
}
</style>