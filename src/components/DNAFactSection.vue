<template>
  <div class="dnafact-container">
    <div class="fact-text-container">
      <p class="fact-text">
        <span ref="textLine1">Although dog breeds vary greatly due to their historical breeding for specific tasks, they still share</span>
        <div class="inline-container">
          <span class="percentage-counter red-text" ref="percentageCounter">0%</span>
          <span ref="textLine2">of their DNA with<span class="red-text"> wolves</span></span>
        </div>
      </p>
    </div>
    
    <div class="boxes-container">
  <div class="dog-image-container">
    <img src="@/assets/images/komondor.png" alt="Komondor dog breed" class="dog-image">
  </div>
  
  <div class="dog-image-container">
    <img src="@/assets/images/doberman.png" alt="Doberman dog breed" class="dog-image">
  </div>
  
  <div class="dog-image-container">
    <img src="@/assets/images/chihuahua.png" alt="Chihuahua dog breed" class="dog-image">
  </div>
</div>
  </div>
</template>

<script>
import { onMounted, ref, onBeforeUnmount } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

export default {
  name: 'DNAFactSection',
  setup() {
    const percentageCounter = ref(null);
    const textLine1 = ref(null);
    const textLine2 = ref(null);
    
    onMounted(() => {
      // Make sure counter starts at 0% every time
      if (percentageCounter.value) {
        percentageCounter.value.innerHTML = "0%";
      }
      
      // Animate the text appearance with proper toggle actions
      gsap.from(textLine1.value, {
        scrollTrigger: {
          trigger: ".dnafact-container",
          start: "top 60%",
          toggleActions: "restart none none reverse" // Key change here
        },
        opacity: 0,
        y: 50,
        duration: 1,
        ease: "power2.out"
      });
      
      gsap.from(textLine2.value, {
        scrollTrigger: {
          trigger: ".dnafact-container",
          start: "top 60%",
          toggleActions: "restart none none reverse" // Key change here
        },
        opacity: 0,
        y: 50,
        duration: 1,
        delay: 0.2,
        ease: "power2.out"
      });
      
      // Counter animation with toggle actions
      let counterTween = null;
      
      // Create ScrollTrigger for counter animation
      ScrollTrigger.create({
        trigger: ".dnafact-container",
        start: "top 70%", // Start a bit earlier so animation is visible on scroll
        end: "bottom 20%",
        toggleActions: "restart none none reset", // Important - restart animation on each entry
        onEnter: () => {
          // Kill any existing animation
          if (counterTween) counterTween.kill();
          
          // Start new animation
          counterTween = gsap.to(percentageCounter.value, {
            innerHTML: "99.9",
            duration: 2, 
            snap: { innerHTML: 0.1 },
            onUpdate: function() {
              percentageCounter.value.innerHTML = parseFloat(percentageCounter.value.innerHTML).toFixed(1) + "%";
            }
          });
        },
        onLeave: () => {
          // Optional: reset when scrolling quickly past
          if (percentageCounter.value) {
            percentageCounter.value.innerHTML = "0%";
          }
        },
        onEnterBack: () => {
          // Re-animate when scrolling back up into view
          if (counterTween) counterTween.kill();
          
          counterTween = gsap.to(percentageCounter.value, {
            innerHTML: "99.9",
            duration: 2,
            snap: { innerHTML: 0.1 },
            onUpdate: function() {
              percentageCounter.value.innerHTML = parseFloat(percentageCounter.value.innerHTML).toFixed(1) + "%";
            }
          });
        },
        onLeaveBack: () => {
          // Reset when scrolling back up out of view
          if (percentageCounter.value) {
            percentageCounter.value.innerHTML = "0%";
          }
        }
      });
    });
    
    onBeforeUnmount(() => {
      // Kill all ScrollTriggers to prevent memory leaks
      ScrollTrigger.getAll().forEach(trigger => trigger.kill());
    });
    
    return {
      percentageCounter,
      textLine1,
      textLine2
    };
  }
};
</script>

<style scoped>
.dnafact-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 2rem;
  background-color: #F2EFEB;
  position: relative;
  overflow: hidden; /* Change this from visible to hidden */
  z-index: 10;
  margin-bottom: 15vh; /* Add this line or increase the value */
}

.dog-image-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.dog-image {
  width: 270px;
  object-fit: contain;
  transition: transform 0.3s ease;
  background-color: transparent;
}

.dog-image:hover {
  transform: scale(1.05);
}


/* For smaller screens */
@media (max-width: 768px) {
  .dog-image {
    width: 150px;
    height: 150px;
  }
}

.fact-text-container {
  width: 90%;
  max-width: 800px;
  margin: 0 auto;
  margin-top: 30vh; /* Position higher in the container */
  display: flex;
  justify-content: center;
  position: relative;
  z-index: 10; /* Ensure text is above boxes */
}

.fact-text {
  font-family: "ivypresto-headline", serif;
  font-weight: 400;
  font-style: normal;
  font-size: 50px;
  line-height: 1.2;
  text-align: center;
  max-width: 1000px;
  color: black
}

.inline-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.red-text {
  color: red;
  font-weight: 400;
}

.percentage-counter {
  display: inline-block;
  font-size: 64px;
  line-height: 1;
  margin-right: 10px;
}

.boxes-container {
  display: flex;
  gap: 10px;
  margin-top: 20vh; /* Push boxes down below text */
  margin-bottom: 5vh;
  align-self: center;
  justify-content: center;
  position: relative;
  z-index: 1;
}

.box-placeholder {
  width: 200px;
  height: 200px;
  background-color: #ddd;
}

/* For screens wider than 1200px */
@media (min-width: 1200px) {
  .fact-text {
    font-size: 55px;
  }
}

/* For smaller screens */
@media (max-width: 768px) {
  .fact-text {
    font-size: 30px;
  }
  
  .boxes-container {
    flex-direction: column;
  }
}
</style>