<template>
  <div class="timeline-container">
    <div class="title-container">
      <h1 class="timeline-title">Dog Domestication</h1>
      <p class="timeline-subtitle">
        <span>Canis lupus: grey wolf</span> 
        <span class="arrow">&#10230;</span> 
        <span>Canis familiaris: dog</span>
      </p>
    </div>
    
    <!-- Timeline track with drag instructions -->
    <div class="timeline-track">
      <div class="drag-instructions">Drag the marker to navigate</div>
    </div>
    
    <!-- Draggable marker with handle -->
    <div class="marker-wrapper" ref="markerWrapper">
      <div class="marker">
        <div class="grab-handle">
          <img src="@/assets/images/dog-paw.png" alt="Dog Paw" class="dog-paw-icon" />
        </div>
      </div>
      <div class="timeline-year">
        {{ currentYear }}
        <div class="drag-arrow" v-if="showArrow">→</div>
      </div>
    </div>

    <div class="slider">
      <div class="slider-wrapper" ref="sliderWrapper">
        <div class="slide" v-for="(item, i) in timelineItems" :key="i">
          <img :src="item.image" :alt="item.title" />
          <h3 class="slide-heading">{{ item.heading }}</h3>
          <div class="slide-caption">{{ item.caption }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref, onBeforeUnmount, computed } from 'vue';
import { gsap } from 'gsap';
import { Draggable } from 'gsap/Draggable';

// Import all timeline images
import timeline1 from '@/assets/images/timeline/timeline-1.jpg';
import timeline2 from '@/assets/images/timeline/timeline-2.jpg';
import timeline3 from '@/assets/images/timeline/timeline-3.jpg';
import timeline4 from '@/assets/images/timeline/timeline-4.jpg';
import timeline5 from '@/assets/images/timeline/timeline-5.jpg';
import timeline6 from '@/assets/images/timeline/timeline-6.jpg';
import timeline7 from '@/assets/images/timeline/timeline-7.jpg';

// Register the Draggable plugin
gsap.registerPlugin(Draggable);

export default {
  name: 'TimelineSection',
  props: {
    isUnlocked: {
      type: Boolean,
      default: false
    }
  },
  setup(props, { emit }) {
    const sliderWrapper = ref(null);
    const markerWrapper = ref(null);
    let target = 0;
    let current = 0;
    let ease = 0.075;
    let maxScroll = 0;
    let animationFrame = null;
    let resizeListener = null;
    let draggableInstance = null;
    const currentIndex = ref(0);
    
    const timelineItems = [
      {
        year: '30,000–15,000 BCE',
        image: timeline1,
        heading: 'Prehistoric Mutualism',
        caption: 'Wolves and humans share overlapping hunting grounds, gradually adjusting behaviors in response to one another. This isn’t top-down domestication — it’s co-evolution. Wolves benefit from food scraps; humans gain early warning systems.',
      },
      {
        year: '15,000–8,000 BCE',
        image: timeline2,
        heading: 'Emergent Companionship',
        caption: 'In ancient cultures across the world — from Siberia to the Americas — dogs begin appearing in graves, often with signs of care and ritual. This marks a shift from utility to emotional and symbolic partnership.',
      },
      {
        year: '8,000–3,000 BCE',
        image: timeline3,
        heading: 'Agricultural Integration',
        caption: 'As humans settle into agrarian life, dogs adapt too. Roles like herding, guarding, and hunting emerge in societies across Europe, the Middle East, and Central Asia. These aren’t just tasks — they’re shared infrastructures of survival.',
      },
      {
        year: '3,000 BCE–500 CE',
        image: timeline4,
        heading: 'Symbolic & Spiritual Dogs',
        caption: 'Dogs appear in myths, art, and religious rituals across cultures: Anubis in Egypt, Xolotl in Mesoamerica, the hounds of Hecate in Greece, and hunting dogs in Roman society. They transcend utility, becoming symbolic figures in life, death, and spiritual journeys. ',
      },
      {
        year: '500–1500 CE',
        image: timeline5,
        heading: 'Medieval Multiplicity',
        caption: 'Dogs play diverse roles worldwide — from hunting in Europe to ratting in Asia and working in Africa. No longer just laborers, they are now companions: named, loved, and mourned.',
      },
      {
        year: '1500–1900 CE',
        image: timeline6,
        heading: 'Breeding & Control',
        caption: 'During the colonial period and Enlightenment, dog breeding becomes formalized in Europe, and breeds like the Greyhound, Mastiff, and Terriers gain prominence. Dogs are bred for specific roles or statuses.',
      },
      {
        year: '1900–Present',
        image: timeline7,
        heading: 'Companion Species Now',
        caption: 'Dogs are family, helpers, and healers. From guiding the blind to aiding police and conservationists, they shape societies worldwide. Emotionally attuned, studies have shown that our bonds releases oxytocin in both species.',
      }
    ];

    // Computed property for current year based on index
    const currentYear = computed(() => {
      return timelineItems[currentIndex.value]?.year || '';
    });
    
    // Track if we're at the beginning of the timeline
    const isAtStart = computed(() => {
      return currentIndex.value === 0;
    });

    // Track if we're at the end of the timeline
    const isAtEnd = computed(() => {
      return currentIndex.value === timelineItems.length - 1;
    });

    function lerp(start, end, factor) {
      return start + (end - start) * factor;
    }

    // Update the timeline based on marker position
// Update the timeline based on marker position
function updateTimelineFromMarkerPosition(markerX) {
  // Calculate which timeline item we're closest to
  const trackWidth = window.innerWidth - 200; // Account for marker width and padding
  const progress = (markerX - 70) / trackWidth;
  
  // Calculate the index based on the total number of items (7)
  const index = Math.min(
    timelineItems.length - 1,
    Math.max(0, Math.round(progress * (timelineItems.length + 2)))
  );
  
  currentIndex.value = index;
  
  // Calculate target position for the slider
  const slideWidth = 450; // Width of slide + gap
  target = index * slideWidth;
}

    // Move the timeline by a specific number of items
    function moveTimeline(direction) {
  const newIndex = Math.min(
    timelineItems.length - 1,
    Math.max(0, currentIndex.value + direction)
  );
  
  if (newIndex !== currentIndex.value) {
    currentIndex.value = newIndex;
    
    // Calculate new marker position - adjust for 7 items
    const trackWidth = window.innerWidth - 200;
    // Use timelineItems.length - 1 to properly distribute positions
    const markerPosition = 70 + (trackWidth * newIndex / (timelineItems.length - 1));
    
    // Animate the marker
    gsap.to(markerWrapper.value, {
      x: markerPosition,
      duration: 0.5,
      ease: "power2.out"
    });
    
    // Calculate target position for the slider
    const slideWidth = 600; // Width of slide + gap
    target = newIndex * slideWidth;
  }
}

    function update() {
      // Smooth the animation
      current = lerp(current, target, ease);
      
      // Update slider position
      if (sliderWrapper.value) {
        gsap.set(sliderWrapper.value, {
          x: -current
        });
      }
      
      animationFrame = requestAnimationFrame(update);
    }

    // Add a new ref to track arrow visibility
    const showArrow = ref(true);

    // Set up draggable marker
    function setupDraggable() {
  if (!markerWrapper.value) return;
  
  // Calculate bounds for the marker
  const trackWidth = window.innerWidth - 200;
  
  // Create the draggable instance
  draggableInstance = Draggable.create(markerWrapper.value, {
    type: "x",
    bounds: { minX: 70, maxX: trackWidth + 70 },
    edgeResistance: 0.9,
    onDrag: function() {
      updateTimelineFromMarkerPosition(this.x);
    },
    onDragEnd: function() {
      // Snap to the nearest timeline item
      const trackWidth = window.innerWidth - 400;
      // Use division by (timelineItems.length - 1) for even distribution
      const markerPosition = 70 + (trackWidth * currentIndex.value / (timelineItems.length - 1));
      
      gsap.to(markerWrapper.value, {
        x: markerPosition,
        duration: 0.3,
        ease: "power2.out"
      });
      
      // Check if we're at the last slide (1900-Present)
      if (currentIndex.value === timelineItems.length - 1) {
        // Hide the arrow when we reach the end
        showArrow.value = false;
      }
    }
  })[0];
  
  // Position marker at first item
  gsap.set(markerWrapper.value, { x: 70 });
}

    onMounted(() => {
      // Start animation loop
      update();
      
      // Set up the draggable marker
      setupDraggable();
      
      // Add resize listener
      resizeListener = () => {
        // Recalculate dimensions
        maxScroll = sliderWrapper.value.offsetWidth - window.innerWidth;
        
        // Update draggable bounds if instance exists
        if (draggableInstance) {
          const trackWidth = window.innerWidth - 200;
          draggableInstance.applyBounds({ minX: 70, maxX: trackWidth + 70 });
          
          // Reposition marker based on current index
          const markerPosition = 70 + (trackWidth * currentIndex.value / (timelineItems.length - 1));
          gsap.set(markerWrapper.value, { x: markerPosition });
        }
      };
      
      window.addEventListener("resize", resizeListener);
      
      // Add title animation
      const timelineTitle = document.querySelector('.timeline-title');
      if (timelineTitle) {
        const titleText = timelineTitle.textContent;
        let titleHTML = "";
        
        for (let i = 0; i < titleText.length; i++) {
          titleHTML += `<span class="char">${titleText[i]}</span>`;
        }
        
        timelineTitle.innerHTML = titleHTML;
        
        gsap.from(".timeline-title .char", {
          opacity: 0,
          y: 100,
          rotateX: -90,
          stagger: 0.02,
          ease: "back.out(1.7)",
          duration: 1
        });
      }
      
      // Show drag instructions briefly
      gsap.to(".drag-instructions", {
        opacity: 1,
        duration: 1,
        delay: 0.5,
        onComplete: () => {
          gsap.to(".drag-instructions", {
            opacity: 0,
            duration: 1,
            delay: 3
          });
        }
      });
    });
    
    onBeforeUnmount(() => {
      window.removeEventListener("resize", resizeListener);
      cancelAnimationFrame(animationFrame);
      
      // Kill draggable instance
      if (draggableInstance) {
        draggableInstance.kill();
        draggableInstance = null;
      }
    });

    return {
      sliderWrapper,
      markerWrapper,
      timelineItems,
      currentYear,
      isAtStart,
      isAtEnd,
      moveTimeline,
      showArrow,
      currentIndex
    };
  }
};
</script>

<style scoped>
.timeline-container {
  width: 100%;
  height: 100vh;
  position: relative;
  overflow: hidden;
}

.title-container {
  position: absolute;
  top: 40px;
  right: 60px;
  text-align: right;
  z-index: 20;
}

.timeline-title {
  color: red;
  font-family: "ivypresto-headline", serif;
  font-weight: 400;
  font-style: normal;
  font-size: 55px;
  margin-bottom: 10px;
}

.timeline-subtitle {
  font-family: "ivypresto-headline", serif;
  font-weight: 100;
  font-style: italic;
  font-size: 18px;
  color: red;
  margin-top: 15px;
  text-align: right;
}

.timeline-subtitle .arrow {
  margin: 0 10px;
  display: inline-block;
}

/* Timeline track for visual guidance */
.timeline-track {
  position: absolute;
  top: 210px;
  left: 120;
  width: 100%;
  height: 2px;
  z-index: 5;
}

/* Drag instructions tooltip */
.drag-instructions {
  position: absolute;
  top: -40px;
  left: 13%;
  transform: translateX(-50%);
  background-color: rgba(255,0,0,0.8);
  color: white;
  padding: 5px 12px;
  border-radius: 15px;
  font-size: 14px;
  opacity: 0;
  white-space: nowrap;
}

/* Individual character styling for animation */
.timeline-title .char {
  display: inline-block;
  white-space: pre;
}

.slider {
  width: 100%;
  height: 70vh;
  overflow: hidden;
  margin-top: 180px; /* Push content down to make room for the timeline */
}

.slider-wrapper {
  width: max-content;
  padding: 0 150px;
  height: 100%;
  display: flex;
  align-items: center;
  gap: 100px;
}

.slide {
  width: 500px;
  height: 500px !important;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.slide img {
  width: 100%;
  height: 85%;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.slide-heading {
  color: #000;
  margin-top: 15px;
text-align: center;
font-family: "IvyPresto Headline";
font-size: 22px;
font-style: italic;
font-weight: 400;
line-height: normal;
}

.slide-caption {
  margin-top: 20px;
  color: #000;
text-align: center;
font-family: "Neue Haas Grotesk Display Pro";
font-size: 16px;
font-style: normal;
font-weight: 400;
line-height: normal;
  max-width: 85%;
  text-align: center;
}

.marker-wrapper {
  position: absolute;
  top: 0;
  left: 0;
  width: max-content;
  height: 100vh;
  z-index: 10;
  cursor: grab;
}

.marker-wrapper:active {
  cursor: grabbing;
}

.marker {
  position: relative;
  width: 1px;
  height: 100%;
  background: red;
}

.marker:after {
  position: absolute;
  content: "";
  display: block;
  top: 200px; /* Moved higher to avoid covering images */
  left: -20px;
  width: 40px;
  height: 40px;
  background-color: #F2EFEB;
  border: 2px solid red;
  border-radius: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: grab;
}

.grab-handle {
  position: absolute;
  top: 202px;
  left: -18px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 12;
  cursor: grab;
}

.grab-handle:active {
  cursor: grabbing;
}

.dog-paw-icon {
  width: 30px; /* Adjust size as needed */
  height: 30px;
  object-fit: contain;
  cursor: grab;
}

.grab-handle:active .dog-paw-icon {
  cursor: grabbing;
}

.timeline-year {
  position: absolute;
  top: 200px; /* Moved below the circle */
  left: 0;
  width: 100px;
  font-family: "IvyPresto Headline";
font-size: 24px;
font-style: italic;
font-weight: 600;
line-height: normal;
  color: red;
  text-align: center;
  transform: translateX(-50%);
  margin-left: 130px;
  white-space: nowrap; /* Prevents text wrapping */
  overflow: visible; /* Allows text to extend beyond the container */
}

.drag-arrow {
  display: inline-block;
  color: red;
  font-size: 28px;
  margin-left: 10px;
  animation: bounceArrow 1.5s ease-in-out infinite;
  animation-delay: 0.5s;
  position: relative;
  top: 2px;
}

@keyframes bounceArrow {
  0%, 100% {
    transform: translateX(0);
  }
  50% {
    transform: translateX(10px);
  }
}

.nav-btn {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: rgba(255, 0, 0, 0.8);
  color: white;
  border: none;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: background-color 0.2s;
}

.nav-btn:hover {
  background-color: rgba(255, 0, 0, 1);
}

.nav-btn:disabled {
  background-color: rgba(200, 200, 200, 0.5);
  cursor: not-allowed;
}

/* Arrow responsiveness */
@media (max-width: 768px) {
  .drag-arrow {
    font-size: 24px;
  }
  
  .timeline-year {
    font-size: 20px;
  }
}
</style>