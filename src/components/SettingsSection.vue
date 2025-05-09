<!-- SettingsSection.vue -->
<template>
  <div class="settings-container">
    <h1 class="settings-title">Unleash the Options</h1>
    
    <!-- Dog images positioned around the page -->
    <div class="decorative-dogs" :class="sliderState">
      <!-- Minimal image (always visible in minimal state) -->
      <img src="@/assets/images/settings/setting-1.png" alt="Dog" class="dog-image minimal-dog" />
      
      <!-- Medium images (only visible in medium and maximum states) -->
      <img src="@/assets/images/settings/setting-2.png" alt="Dog" class="dog-image medium-dog medium-plus" />
      <img src="@/assets/images/settings/setting-3.png" alt="Dog" class="dog-image medium-dog medium-plus" />
      <img src="@/assets/images/settings/setting-4.png" alt="Dog" class="dog-image medium-dog medium-plus" />
      <img src="@/assets/images/settings/setting-5.png" alt="Dog" class="dog-image medium-dog medium-plus" />
      
      <!-- Maximum images (only visible in maximum state) -->
      <img src="@/assets/images/settings/setting-6.png" alt="Dog" class="dog-image max-dog max-only" />
      <img src="@/assets/images/settings/setting-7.png" alt="Dog" class="dog-image max-dog max-only" />
      <img src="@/assets/images/settings/setting-8.png" alt="Dog" class="dog-image max-dog max-only" />
      <img src="@/assets/images/settings/setting-9.png" alt="Dog" class="dog-image max-dog max-only" />
    </div>
    
    <div class="settings-panel">
      <div class="settings-panel-header">Who Let the Dogs In?</div>
      <p class="settings-top-description">You did. Use the slider below to decide how many dogs you want to see in this project.</p>
      
      <div class="slider-container">
        <div class="slider-label" :class="{ 'active': sliderState === 'minimal' }">I LIKE CATS</div>
        <input 
          type="range" 
          min="0" 
          max="2" 
          step="1" 
          v-model="dogCountIndex" 
          class="slider"
          @input="handleSliderChange"
          :style="sliderTrackStyle"
        />
        <div class="slider-label" :class="{ 'active': sliderState === 'maximum' }">DOGPILE ME</div>
      </div>
    </div>
    
    <div class="settings-panel">
      <div class="settings-panel-header">Fetching some guidance?</div>
      <p class="settings-description">Would you like a guide dog to accompany you through the project?</p>
      
      <div class="option-buttons">
        <label class="option-button">
          <input type="radio" v-model="needsGuide" :value="true" name="guide" @change="emitGuideSelection">
          <span class="button-box">
            <span v-if="needsGuide === true" class="x-mark">x</span>
          </span>
          <span class="button-label">YES</span>
        </label>
        
        <label class="option-button">
          <input type="radio" v-model="needsGuide" :value="false" name="guide" @change="emitGuideSelection">
          <span class="button-box">
            <span v-if="needsGuide === false" class="x-mark">x</span>
          </span>
          <span class="button-label">NO</span>
        </label>
      </div>
    </div>
    
    <!-- Replaced continue button with scroll indicator -->
    <div class="scroll-indicator" @click="onContinueClick">
      <span>Click to Continue</span>
      <div class="arrow-down"></div>
    </div>
  </div>
</template>

<script>
import { gsap } from 'gsap';

export default {
  name: 'SettingsSection',
  data() {
    return {
      dogCountIndex: 1, // Default to middle position (0=minimal, 1=medium, 2=maximum)
      needsGuide: null
    }
  },
  computed: {
    // Map the index to a state name
    sliderState() {
      const states = ['minimal', 'medium', 'maximum'];
      return states[this.dogCountIndex];
    },
    
    // Get actual number of dogs based on slider state
    actualDogCount() {
      switch(this.sliderState) {
        case 'minimal':
          return 2;
        case 'medium':
          return 10;
        case 'maximum':
          return 20;
        default:
          return 5;
      }
    },
    
    // Get active track color based on state
    sliderTrackStyle() {
      // Calculate percentage filled based on dogCountIndex
      const percentage = (this.dogCountIndex / 2) * 100;
      return {
        background: `linear-gradient(90deg, red ${percentage}%, #ddd ${percentage}%)`
      };
    }
  },
  mounted() {
    // Add the GSAP animation for the scroll indicator
    gsap.to(".scroll-indicator", {
      y: 10,
      repeat: -1,
      yoyo: true,
      duration: 1.5,
      delay: 5,
    });
    
    // Initialize slider track color
    this.updateSliderTrack();
  },
  methods: {
    emitGuideSelection() {
      console.log('Emitting guide selection:', this.needsGuide); // For debugging
      // Only emit if a choice has been made
      if (this.needsGuide !== null) {
        this.$emit('guide-selected', this.needsGuide);
      }
    },
    onContinueClick() {
      this.emitGuideSelection();
      this.$emit('continue-clicked');
    },
    updateSliderTrack() {
      // Emit both the state name and dog count
      this.$emit('dog-display-mode-changed', {
        mode: this.sliderState,
        count: this.actualDogCount
      });
    },
    handleSliderChange() {
      this.updateSliderTrack();
      
      // Animate slider thumb
      gsap.to('.slider::-webkit-slider-thumb', {
        scale: 1.2,
        duration: 0.2,
        yoyo: true,
        repeat: 1
      });
      
      // Animate image transitions based on slider state
      if (this.sliderState === 'minimal') {
        // Force hide all dogs first
        gsap.set(['.medium-dog', '.max-dog'], {
          opacity: 0,
          scale: 0,
          display: 'none'
        });
        
        // Show minimal dogs with animation
        gsap.to('.minimal-dog', {
          opacity: 1,
          scale: 1,
          display: 'block',
          duration: 0.6,
          ease: "back.out(1.7)"
        });
      } 
      else if (this.sliderState === 'medium') {
        // Force hide minimal and max dogs
        gsap.set(['.minimal-dog', '.max-dog'], {
          opacity: 0,
          scale: 0,
          display: 'none'
        });
        
        // Make medium dogs visible first
        gsap.set('.medium-dog', {
          display: 'block'
        });
        
        // Then animate them
        gsap.to('.medium-dog', {
          opacity: 1,
          scale: 1,
          duration: 0.6,
          stagger: 0.1,
          ease: "back.out(1.7)"
        });
      } 
      else if (this.sliderState === 'maximum') {
        // Force hide minimal dogs
        gsap.set('.minimal-dog', {
          opacity: 0,
          scale: 0,
          display: 'none'
        });
        
        // Make medium and max dogs visible first
        gsap.set(['.medium-dog', '.max-dog'], {
          display: 'block'
        });
        
        // Create an animation timeline
        const tl = gsap.timeline();
        
        // Animate setting-8.png immediately (max-dog:nth-of-type(8))
        tl.to('.max-dog:nth-of-type(8)', {
          opacity: 1,
          scale: 1,
          duration: 0.6,
          ease: "elastic.out(1, 0.3)"
        }, 0);
        
        // Animate all other medium and max dogs with stagger
        tl.to(['.medium-dog', '.max-dog:not(:nth-of-type(8))'], {
          opacity: 1,
          scale: 1,
          duration: 0.6,
          stagger: 0.1,
          ease: "elastic.out(1, 0.3)"
        }, 0);
      }
    }
  },
  watch: {
    dogCountIndex() {
      this.updateSliderTrack();
    }
  }
}
</script>

<style scoped>
.settings-container {
  width: 100%;
  min-height: 100vh;
  background: #f2efea;
  padding: 40px 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: "ivypresto-headline", serif;
  font-weight: 400;
  font-style: normal;
  position: relative; /* Added to position the scroll indicator */
}

.settings-title {
  color: red;
  font-family: "ivypresto-headline", serif;
  margin-top: 70px;
  font-weight: 400;
  font-style: normal;
  font-size: 64px;
  margin-bottom: 60px;
  text-align: center;
  perspective: 500px;
}

/* Individual character styling for animation */
.settings-title .char {
  display: inline-block;
  white-space: pre;
}

.settings-top-description {
  font-size: 18px;
  font-family: "neue-haas-grotesk-display", sans-serif;
  font-weight: 400;
  font-style: normal;
  text-align: center;
  margin-bottom: 30px;
  color: #333;
  width: 100%;
  position: relative;
}

.settings-panel {
  width: 100%;
  max-width: 800px;
  margin-bottom: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 2; /* Ensure panels stay above the dogs */
}

.settings-panel-header {
  background-color: #81C9E3;
  padding: 10px 20px;
  font-size: 24px;
  margin-bottom: 20px;
  text-align: center;
}

.settings-description {
  font-size: 18px;
  font-family: "neue-haas-grotesk-display", sans-serif;
  font-weight: 400;
  font-style: normal;
  text-align: center;
  margin-bottom: 30px;
  color: #333;
  width: 100%;
  position: relative;
}

.slider-container {
  display: flex;
  align-items: center;
  width: 100%;
  justify-content: space-between;
  margin-top: 10px;
  position: relative;
}

.slider-label {
  font-weight: bold;
  font-size: 18px;
  width: 115px;
  text-align: center;
  font-family: "roca", sans-serif;
  font-weight: 400;
  font-style: normal;
}

.slider {
  flex: 1;
  height: 8px;
  appearance: none;
  background: #ddd;
  outline: none;
  margin: 0 20px;
  border-radius: 4px;
}

.slider::-webkit-slider-thumb {
  appearance: none;
  width: 25px;
  height: 25px;
  background: red;
  border-radius: 50%;
  cursor: pointer;
}

.option-buttons {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-start;
  gap: 32px; /* space between YES and NO */
}

.option-button input[type="radio"] {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
  pointer-events: none;
}

.button-box {
  width: 50px;
  height: 50px;
  border: 2px solid #333;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
  border-radius: 10px;
}

.x-mark {
  color: red;
  font-size: 24px;
  font-weight: bold;
}

.button-label {
  font-size: 20px;
  font-weight: bold;
  font-family: "roca", sans-serif;
  font-style: normal;
  text-align: center;
  margin: 0;
  display: block;
  /* Remove width for better centering */
}

/* Add the scroll indicator styles */
.scroll-indicator {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  color: red;
  font-family: "ivypresto-headline", serif;
  opacity: 0;
  animation: fadeIn 1s ease-in-out 5s forwards;
}

.scroll-indicator span {
  margin-bottom: 10px;
}

.arrow-down {
  width: 20px;
  height: 20px;
  border-left: 2px solid red;
  border-bottom: 2px solid red;
  transform: rotate(-45deg);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Add slider images preview styles */
.slider-images-preview {
  position: relative;
  width: 100%;
  height: 200px; /* Adjust based on your image size needs */
  margin: 20px 0;
  display: flex;
  justify-content: center;
  align-items: center;
}

.dog-image {
  position: absolute;
  width: 80px;
  height: 80px;

  transition: all 0.5s ease-out;
}

/* Hide medium and max images initially */
.medium-plus {
  opacity: 0;
  transform: scale(0);
}

.max-only {
  opacity: 0;
  transform: scale(0);
}

/* Also add this to make sure we start in a clean state */
.decorative-dogs {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

.decorative-dogs.minimal .medium-dog,
.decorative-dogs.minimal .max-dog {
  opacity: 0;
  transform: scale(0);
}

/* Minimal state styling (1 image) */
.slider-images-preview.minimal .minimal-dog {
  width: 100px;
  height: 100px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* Medium state styling (4 images) */
.slider-images-preview.medium .minimal-dog {
  display: none;
}

.slider-images-preview.medium .medium-plus {
  display: block;
}

.slider-images-preview.medium .medium-dog:nth-child(2) {
  top: 20%;
  left: 25%;
}

.slider-images-preview.medium .medium-dog:nth-child(3) {
  top: 20%;
  left: 75%;
}

.slider-images-preview.medium .medium-dog:nth-child(4) {
  top: 70%;
  left: 30%;
}

.slider-images-preview.medium .medium-dog:nth-child(5) {
  top: 70%;
  left: 70%;
}

/* Maximum state styling (8 images) */
.slider-images-preview.maximum .minimal-dog {
  display: none;
}

.slider-images-preview.maximum .medium-plus {
  display: block;
}

.slider-images-preview.maximum .max-only {
  display: block;
}

.slider-images-preview.maximum .medium-dog:nth-child(2) {
  top: 10%;
  left: 20%;
}

.slider-images-preview.maximum .medium-dog:nth-child(3) {
  top: 10%;
  left: 80%;
}

.slider-images-preview.maximum .medium-dog:nth-child(4) {
  top: 50%;
  left: 15%;
}

.slider-images-preview.maximum .medium-dog:nth-child(5) {
  top: 50%;
  left: 85%; 
}

.slider-images-preview.maximum .max-dog:nth-child(6) {
  top: 30%;
  left: 50%;
}

.slider-images-preview.maximum .max-dog:nth-child(7) {
  top: 70%;
  left: 40%;
}

.slider-images-preview.maximum .max-dog:nth-child(8) {
  top: 70%;
  left: 60%;
}

.slider-images-preview.maximum .max-dog:nth-child(9) {
  top: 30%;
  left: 35%;
}

/* Decorative dogs styles */
.decorative-dogs {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

/* Base styling for all dog images */
.dog-image {
  position: absolute;

  transition: all 0.8s ease;
  opacity: 0;
  transform: scale(0);

}

/* Minimal state images */
.decorative-dogs.minimal .minimal-dog {
  width: 270px;
  height: 270px;
  transform: rotate(-11.521deg)!important;
  opacity: 1;
  transform: scale(1);
  left: 10%;
  top: 25%;
}

/* Medium state styling */
.decorative-dogs.medium .minimal-dog {
  opacity: 0;
  transform: scale(0);
}

.decorative-dogs.medium .medium-dog {
  opacity: 1;
  transform: scale(1);
}

.decorative-dogs.medium .medium-dog:nth-of-type(2) {
  width: 400px;
  height: 400px;
  left: -2%;
  bottom: 0%;
}

.decorative-dogs.medium .medium-dog:nth-of-type(3) {
  width: 370px;
  height: 370px;
  right: -2%;
  bottom: 0%;
}

.decorative-dogs.medium .medium-dog:nth-of-type(4) {
  width: 350px;
  height: 350px;
  right: -3%;
  top: 15%;
}

.decorative-dogs.medium .medium-dog:nth-of-type(5) {
  width: 300px;
  height: 300px;
  transform: rotate(-14.891deg)!important;
  left: -3%;
  top: 15%;
}

/* Maximum state styling */
.decorative-dogs.maximum .minimal-dog {
  opacity: 0;
  transform: scale(0);
}

.decorative-dogs.maximum .medium-dog,
.decorative-dogs.maximum .max-dog {
  opacity: 1;
  transform: scale(1);
}

.decorative-dogs.maximum .medium-dog:nth-of-type(2) {
  width: 400px;
  height: 400px;
  left: -2%;
  bottom: 0%;
}

.decorative-dogs.maximum .medium-dog:nth-of-type(3) {
  width: 370px;
  height: 370px;
  right: -2%;
  bottom: 0%;
}

.decorative-dogs.maximum .medium-dog:nth-of-type(4) {
  width: 350px;
  height: 350px;
  right: -3%;
  top: 15%;
}

.decorative-dogs.maximum .medium-dog:nth-of-type(5) {
  width: 300px;
  height: 300px;
  transform: rotate(-14.891deg)!important;
  left: -3%;
  top: 15%;
 
}

.decorative-dogs.maximum .max-dog:nth-of-type(6) {
  width: 300px;
  height: 300px;
  transform: rotate(14.759deg)!important;
 right: -3%;
  top: -5%;
}

.decorative-dogs.maximum .max-dog:nth-of-type(7) {
  width: 250px;
  height: 250px;
  transform: rotate(15deg)!important;
  right: -3%;
  bottom: 32%;
  top: auto;
  left: auto;
}

.decorative-dogs.maximum .max-dog:nth-of-type(8) {
  width: 370px;
  height: 370px;
  left: -5%;
  top: 35%;
}

.decorative-dogs.maximum .max-dog:nth-of-type(9) {
  width: 270px;
  height: 270px;
  transform: rotate(14.633deg)!important;
  left: 0%;
  top: -5%;
}



/* Set z-index for specified images */
.decorative-dogs .medium-dog:nth-of-type(2) {
  z-index: 40; /* setting-2.png - frontmost */
}

.decorative-dogs .max-dog:nth-of-type(8) {
  z-index: 30; /* setting-8.png - second */
}

.decorative-dogs .medium-dog:nth-of-type(5) {
  z-index: 20; /* setting-5.png - third */
}

.decorative-dogs .max-dog:nth-of-type(7) {
  z-index: 16; /* setting-7.png - add this to place it above setting-4.png */
}

.decorative-dogs .medium-dog:nth-of-type(4) {
  z-index: 15; /* setting-4.png */
}

.decorative-dogs .max-dog:nth-of-type(6) {
  z-index: 4; /* setting-6.png - give it a lower z-index than default */
}

.decorative-dogs .max-dog:nth-of-type(9) {
  z-index: 10; /* setting-9.png - backmost */
}

/* Ensure other images don't interfere with this stacking order */
.decorative-dogs .dog-image:not(.medium-dog:nth-of-type(2)):not(.max-dog:nth-of-type(8)):not(.medium-dog:nth-of-type(5)):not(.medium-dog:nth-of-type(4)):not(.max-dog:nth-of-type(6)):not(.max-dog:nth-of-type(9)):not(.max-dog:nth-of-type(7)) {
  z-index: 5;
}

</style>