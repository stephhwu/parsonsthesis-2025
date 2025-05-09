<template>
  <div>
    <!-- Landing Page -->
    <div class="landing-container">
      <nav class="menu"></nav>
      <div class="images">
        <div class="left img"></div>
        <div class="right img"></div>
      </div>
      <div class="hero-image">
        <div class="wrapper-img">
          <div class="box"></div>
          <div>
            <img class="image" src="@/assets/images/dog-1.png" />
          </div>
        </div>
      </div>
      <div class="header">
        <p class="title" ref="titleElement">dogs with jobs</p>
      </div>
      <div class="hero-container"></div>
      
      <!-- Scroll indicator -->
      <div class="scroll-indicator" @click="scrollToSettings">
        <span>Scroll down</span>
        <div class="arrow-down"></div>
      </div>
    </div>
    
    <!-- Settings Section -->
    <SettingsSection 
      ref="settingsSection" 
      @guide-selected="handleGuideSelection"
      @continue-clicked="unlockTimeline"
      @dog-display-mode-changed="updateDogDisplayMode"
    />

    <!-- Timeline Section - only render when unlocked -->
    <TimelineSection 
      v-if="timelineUnlocked"
      ref="timelineSection"
      :isUnlocked="timelineUnlocked"
      @scroll-top="handleScrollToTop" 
    />
    
    <!-- DNA Fact Section - only render when timeline is unlocked -->
    <DNAFactSection v-if="timelineUnlocked" ref="dnaFactSection" />

    <!-- Breed Strength Section - only render when timeline is unlocked -->
    <BreedStrengthSection 
  v-if="timelineUnlocked" 
  ref="breedStrengthSection" 
  :dogDisplayMode="dogDisplayMode" 
/>    
    <!-- Matrix Section - only render when timeline is unlocked -->
    <MatrixSection v-if="timelineUnlocked" ref="matrixSection" />
    
    <!-- Sniffer Section - only render when timeline is unlocked -->
    <SnifferSection v-if="timelineUnlocked" ref="snifferSection" />
    
    <!-- Tech vs Dog Section - only render when timeline is unlocked -->
    <TechVsDogSection v-if="timelineUnlocked" ref="techVsDogSection" />
    
    <!-- NEW: Wildlife Detection Section - only render when timeline is unlocked -->
    <WildlifeDetectionSection v-if="timelineUnlocked" ref="wildlifeDetectionSection":dogDisplayMode="dogDisplayMode" />

    <WelfarePolicySection v-if="timelineUnlocked" ref="welfarePolicySection" />
    
    <!-- Outro Section - only render when timeline is unlocked -->
    <OutroSection v-if="timelineUnlocked" ref="outroSection" :dogDisplayMode="dogDisplayMode" />
    
    <!-- Scroll blocker to prevent scrolling past settings until unlocked -->
    <div 
      v-if="!timelineUnlocked" 
      class="scroll-blocker"
    ></div>
  </div>
</template>

<script>
import { onMounted, ref, onBeforeUnmount } from 'vue';
import anime from 'animejs/lib/anime.es.js';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { Observer } from 'gsap/Observer';
import '@/assets/styles/dogs-with-jobs.css';
import SettingsSection from './SettingsSection.vue';
import TimelineSection from './TimelineSection.vue';
import DNAFactSection from './DNAFactSection.vue';
import BreedStrengthSection from './BreedStrengthSection.vue';
import MatrixSection from './MatrixSection.vue';
import SnifferSection from './SnifferSection.vue';
import TechVsDogSection from './TechVsDogSection.vue';
import WildlifeDetectionSection from './WildlifeDetectionSection.vue'; // Import the new component
import WelfarePolicySection from './WelfarePolicySection.vue'; // Import the new component
import OutroSection from './OutroSection.vue';

// Register GSAP plugins
gsap.registerPlugin(ScrollTrigger, Observer);

export default {
name: 'DogsWithJobs',
components: {
  SettingsSection,
  TimelineSection,
  DNAFactSection,
  BreedStrengthSection,
  MatrixSection,
  SnifferSection,
  TechVsDogSection,
  WildlifeDetectionSection, // Register the new component
  WelfarePolicySection, // Register the new component
  OutroSection
},
setup(props, { emit }) {
  const titleElement = ref(null);
  const settingsSection = ref(null);
  const timelineSection = ref(null);
  const dnaFactSection = ref(null);
  const breedStrengthSection = ref(null);
  const matrixSection = ref(null);
  const snifferSection = ref(null);
  const techVsDogSection = ref(null);
  const wildlifeDetectionSection = ref(null); // Add reference to new section
  const welfarePolicySection = ref(null); // Add reference to new section
  const outroSection = ref(null);
  const timelineUnlocked = ref(false);
  let scrollListenerAdded = false;
  const searchQuery = ref('');
  const dogDisplayMode = ref('medium'); // Default to medium

  // Function to scroll to settings
  const scrollToSettings = () => {
    const settingsElement = settingsSection.value?.$el;
    if (settingsElement) {
      settingsElement.scrollIntoView({ behavior: 'smooth' });
    }
  };

  // Function to handle scroll prevention - ensure it only applies BEFORE timeline is unlocked
  const handleScroll = (e) => {
    // For upward scrolling - improve smoothness
    if (e.deltaY < 0) {
      // Only apply special handling when needed
      if (Math.abs(e.deltaY) > 50) {
        // Allow default behavior for smoother native scrolling
        return;
      }
    }

    // Only prevent downward scrolling past settings when timeline is locked
    if (!timelineUnlocked.value) {
      const settingsEl = settingsSection.value?.$el;
      if (settingsEl) {
        const settingsRect = settingsEl.getBoundingClientRect();
        const settingsBottom = settingsRect.bottom;

        // If we're trying to scroll past the settings section going downward
        if (settingsBottom < window.innerHeight && e.deltaY > 0) {
          e.preventDefault();
          // Position settings section at bottom of viewport
          window.scrollTo({
            top: window.pageYOffset + settingsRect.top - 
                (window.innerHeight - settingsRect.height),
            behavior: 'smooth' // Changed from 'auto' to 'smooth'
          });
        }
      }
    }
    // No scroll prevention after timeline is unlocked
  };

  // Function to unlock and scroll to timeline
  const unlockTimeline = () => {
    timelineUnlocked.value = true;

    // Remove scroll prevention when timeline is unlocked
    if (scrollListenerAdded) {
      window.removeEventListener('wheel', handleScroll, { passive: false });
      scrollListenerAdded = false;
    }

    // Add a small delay before scrolling
    setTimeout(() => {
      const timelineElement = timelineSection.value?.$el;
      if (timelineElement) {
        timelineElement.scrollIntoView({ behavior: 'smooth' });
      }
    }, 100);
  };

  // Function to handle guide selection
  const handleGuideSelection = (needsGuide) => {
    emit('guide-selected', needsGuide);
  };

  // Function to handle scroll to top
  const handleScrollToTop = () => {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  const handleSearch = () => {
    if (searchQuery.value.trim() === '') return;
    console.log('Searching for:', searchQuery.value);
    // Implement actual search logic here
  };

  // Add function to update dog display mode when slider changes
  const updateDogDisplayMode = (payload) => {
    dogDisplayMode.value = payload.mode;
    console.log('Dog display mode updated:', dogDisplayMode.value);
  };

  onMounted(() => {
    // Prevent unwanted scrolling past settings section until timeline is unlocked
    window.addEventListener('wheel', handleScroll, { passive: false });
    scrollListenerAdded = true;

    // Process the title text for animation
    const textWrapper = titleElement.value;
    const originalText = textWrapper.textContent.trim();
    const words = originalText.split(" ");
    let result = "";

    words.forEach((word, wIndex) => {
      result += `<span class="word">`;
      for (let i = 0; i < word.length; i++) {
        result += `<span class="letter${word === "with" ? " italic" : ""}">${word[i]}</span>`;
      }
      result += `</span>`;
      if (wIndex < words.length - 1) result += " ";
    });

    textWrapper.innerHTML = result;

    // Animation timeline for the letters
    anime.timeline()
      .add({
        targets: ".title .letter",
        translateY: [150, 0],
        translateZ: 0,
        opacity: [0, 1],
        easing: "easeOutExpo",
        duration: 2000,
        delay: (el, i) => 4800 + 70 * i,
      });

    // GSAP animations
    gsap.to(".box", {
      y: "-100%",
      ease: "expo.inOut",
      duration: 2.4,
      delay: 1,
    });

    gsap.from("img", {
      scale: 2,
      ease: "expo.inOut",
      duration: 4,
      delay: 0,
    });

    gsap.to(".wrapper-img", {
      width: 600,
      height: 650,
      ease: "expo.inOut",
      duration: 2.4,
      delay: 3.6,
    });

    gsap.from(".img", {
      opacity: 0,
      ease: "expo.inOut",
      duration: 0.4,
      delay: 3.4,
    });

    gsap.to(".left", {
      x: -350,
      rotation: -35,
      ease: "expo.inOut",
      duration: 2,
      delay: 3.8,
    });

    gsap.to(".right", {
      x: 400,
      rotation: 25,
      ease: "expo.inOut",
      duration: 2,
      delay: 3.8,
    });

    gsap.from(".menu > div, .hero-container > div", {
      opacity: 0,
      y: 30,
      ease: "expo.inOut",
      duration: 2,
      delay: 4.2,
      stagger: 0.1,
    });
    
    // Add animation for scroll indicator
    gsap.to(".scroll-indicator", {
      y: 10,
      repeat: -1,
      yoyo: true,
      duration: 1.5,
      delay: 5,
    });
    
    // Set up Observer for detecting scrolling
    Observer.create({
      type: "wheel,touch,pointer",
      onDown: () => {
        if (window.scrollY < 10) {
          scrollToSettings();
        }
      },
      // Optional - prevents default behavior (prevents scroll)
      // preventDefault: true
    });
    
    // ScrollTrigger for settings title animation
    const settingsTitle = document.querySelector('.settings-title');
    if (settingsTitle) {
      // Split the title text into individual characters for animation
      const titleText = settingsTitle.textContent;
      let titleHTML = "";
      
      for (let i = 0; i < titleText.length; i++) {
        titleHTML += `<span class="char">${titleText[i]}</span>`;
      }
      
      settingsTitle.innerHTML = titleHTML;
      
      // Animate each character with ScrollTrigger
      gsap.from(".settings-title .char", {
        scrollTrigger: {
          trigger: ".settings-container",
          start: "top 80%",
          end: "top 30%",
          scrub: 1,
          // markers: true, // for debugging
        },
        opacity: 0,
        y: 100,
        rotateX: -90,
        stagger: 0.02,
        ease: "back.out(1.7)",
      });
      
      // Animate the settings panels
      gsap.from(".settings-panel", {
        scrollTrigger: {
          trigger: ".settings-container",
          start: "top 60%",
          end: "top 10%",
          scrub: 1,
          // markers: true, // for debugging
        },
        opacity: 0,
        y: 50,
        stagger: 0.3,
        ease: "power2.out",
      });
    }
  });

  onBeforeUnmount(() => {
    if (scrollListenerAdded) {
      window.removeEventListener('wheel', handleScroll, { passive: false });
    }
  });

  return {
    titleElement,
    settingsSection,
    timelineSection,
    dnaFactSection,
    breedStrengthSection,
    matrixSection,
    snifferSection,
    techVsDogSection,
    wildlifeDetectionSection, // Add to returned refs
    welfarePolicySection, // Add to returned refs
    outroSection,
    timelineUnlocked,
    searchQuery,
    handleSearch,
    dogDisplayMode,
    updateDogDisplayMode,
    scrollToSettings,
    unlockTimeline,
    handleGuideSelection,
    handleScrollToTop
  };
}
};
</script>

<style scoped>
/* Additional styles for scroll indicator */
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

@keyframes fadeIn {
from { opacity: 0; }
to { opacity: 1; }
}

.scroll-indicator span {
margin-bottom: 10px;
}

.arrow-down {
width: 20px;
height: 20px;
border-right: 3px solid red;
border-bottom: 3px solid red;
transform: rotate(45deg);
}

.scroll-blocker {
position: fixed;
top: 0;
left: 0;
width: 100%;
height: 100%;
background: transparent;
pointer-events: none;
}

/* Add search bar styles */
.search-container {
  position: absolute;
  top: 30px;
  right: 40px;
  display: flex;
  align-items: center;
  z-index: 100;
}

.search-input {
  width: 250px;
  height: 40px;
  border: 2px solid #ff0000;
  border-radius: 20px 0 0 20px;
  padding: 0 15px;
  font-family: "ivypresto-headline", serif;
  font-size: 16px;
  background-color: rgba(242, 239, 234, 0.85);
  backdrop-filter: blur(5px);
  color: #333;
  outline: none;
  transition: all 0.3s ease;
}

.search-input:focus {
  background-color: rgba(255, 255, 255, 0.95);
  box-shadow: 0 0 8px rgba(255, 0, 0, 0.3);
  width: 280px;
}

.search-button {
  height: 44px;
  width: 44px;
  background-color: #ff0000;
  border: none;
  border-radius: 0 20px 20px 0;
  color: white;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: background-color 0.3s ease;
}

.search-button:hover {
  background-color: #cc0000;
}

.search-icon {
  font-size: 18px;
}
</style>