<template>
  <div class="tech-vs-dog-section">
    <div class="section-content">
      <h1 class="main-title">Sniffers in Medicine</h1>
      
      <div class="comparison-block">
        <h2 class="comparison-description">Detecting cancer early is costly and difficult with CT scans, but some dogs can sniff it out non-invasively—without radiation and often more accurately. The chart below compares how well dogs and CT machines avoid false positives (specificity)</h2>
        
        <div class="progress-container-wrapper">
          <div class="progress-row">
            <div class="label">Working Dog</div>
            <div class="progress-container">
              <div class="progress-bar dog"></div>
            </div>
            <div class="percentage">93%-99%*</div>
          </div>
          
          <div class="progress-row">
            <div class="label">Low-dose CT</div>
            <div class="progress-container">
              <div class="progress-bar tech"></div>
            </div>
            <div class="percentage">92.6%</div>
          </div>
        </div>
      </div>
      
      <div class="comparison-block cost-block">
        
        <div class="cost-comparison">
          <div class="comparison-column">
            <div class="image-container">
              <img src="@/assets/images/dog-lung.png" alt="Working dog with dollar sign" />
            </div>
            
            <div class="entity-label">Working Dog</div>
            
            <div class="price">~$79 per day</div>
            
            <div class="description">Medical Scent Detection</div>
          </div>

          <div class="vs">          <h2 class="comparison-title">Cost Comparison<sup>†</sup></h2>
            vs</div>
          
          <div class="comparison-column">
            <div class="image-container">
              <img src="@/assets/images/ct-lung.png" alt="X-ray/CT scan image" />
            </div>
            
            <div class="entity-label">Low-dose CT</div>
            
            <div class="price">$3,074 per person</div>
            
            <div class="description">Equipment &amp; Operation</div>
          </div>
        </div>
        
        <div class="footnotes">
        <p class="footnote footnote-dagger">†This price estimate comes from prior use of medical scent canines used in research for detecting coronavirus disease 2019 (COVID-19).</p>
        <p class="footnote footnote-asterisk">*The paper compares findings from four studies on dogs' ability to detect lung cancer (McCulloch et al. 2006, Ehmann et al. 2011, Junqueira et al. 2019, Fjellanger et al. 2022)</p>
      </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

// Register ScrollTrigger plugin
gsap.registerPlugin(ScrollTrigger);

export default {
  name: 'TechVsDogSection',
  setup() {
    onMounted(() => {
      // Animate progress bars when section comes into view
      gsap.fromTo('.progress-bar.dog', 
        { width: '0%' },
        { 
          width: '99%', 
          duration: 1.5, 
          ease: 'power2.out',
          scrollTrigger: {
            trigger: '.progress-row',
            start: 'top 70%',
            toggleActions: 'play none none none'
          }
        }
      );
      
      gsap.fromTo('.progress-bar.tech', 
        { width: '0%' },
        { 
          width: '92.6%', 
          duration: 1.5, 
          ease: 'power2.out',
          scrollTrigger: {
            trigger: '.progress-row',
            start: 'top 70%',
            toggleActions: 'play none none none'
          }
        }
      );
      
      // Animate cost comparison
      gsap.fromTo('.comparison-column', 
        { opacity: 0, y: 20 },
        { 
          opacity: 1, 
          y: 0,
          stagger: 0.3,
          duration: 0.8, 
          ease: 'back.out(1.2)',
          scrollTrigger: {
            trigger: '.cost-comparison',
            start: 'top 70%',
            toggleActions: 'play none none none'
          }
        }
      );
      
      // Animate VS
      gsap.fromTo('.vs', 
        { opacity: 0, scale: 0.6 },
        { 
          opacity: 1, 
          scale: 1,
          duration: 0.8, 
          delay: 0.6,
          ease: 'elastic.out(1.2)',
          scrollTrigger: {
            trigger: '.cost-comparison',
            start: 'top 70%',
            toggleActions: 'play none none none'
          }
        }
      );
      
      gsap.fromTo('.footnote', 
  { opacity: 0, y: 10 },
  { 
    opacity: 1, 
    y: 0,
    stagger: 0.01, // Reduced from 0.2
    duration: 0.2,  // Reduced from 0.8
    delay: 0.3,     // Reduced from 1.0
    ease: 'power2.out',
    scrollTrigger: {
      trigger: '.footnotes',
      start: 'top 90%',
      toggleActions: 'play none none none'
    }
  }
);
    });

    return {};
  }
};
</script>

<style scoped>
.tech-vs-dog-section {
  width: 100%;
  min-height: 100vh;
  background-color: #F2EFEB;
  padding: 80px 0;
  display: flex;
  justify-content: center;
  align-items: center;
  scroll-snap-align: start;
  scroll-snap-stop: always;
  position: relative;
}

.section-content {
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
}

.main-title {
  font-family: "ivypresto-headline", serif;
  font-weight: 400;
  font-style: normal;
  font-size: 50px;
  color: #FF0000; /* Red color from mockup */
  text-align: center;
}

.comparison-block {
  margin-bottom: 60px;
}

.comparison-description {
  font-family: "neue-haas-grotesk-display", sans-serif;
  font-weight: 150;
  font-style: normal;
  color: #A2A2A2;
  font-size: 16px;
  max-width: 650px;
  margin: 20px auto; /* Change top/bottom margins and add auto for left/right */
  position: relative;
  z-index: 2;
  text-align: center;
}

.comparison-title {
  font-family: "ivypresto-headline", serif;
  font-size: 24px;
  font-weight: 400;
  margin-bottom: 40px;
  text-align: center;
  color: #333333;
}

.progress-container-wrapper {
  max-width: 1200px;
  margin: 0 auto;
}

.progress-row {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.label {
  width: 180px;
  font-family: "ivypresto-headline", serif;
  color: #999999; /* Gray from mockup */
  font-size: 24px;
  text-align: right;
  padding-right: 20px;
}

.progress-container {
  flex-grow: 1;
  height: 20px;
  background-color: #E5E5E5;
  border-radius: 10px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  border-radius: 10px;
}

.progress-bar.dog, .progress-bar.tech {
  background-color: #81C9E3; /* Light blue from mockup */
}

.percentage {
  width: 120px;
  font-family: neue-haas-grotesk-display, sans-serif;
  font-weight: 600;
  font-style: normal;  font-size: 20px;
  text-align: left;
  padding-left: 20px;
  color: #333333;
}

/* Cost comparison styles */
.cost-block {
  margin-top: 30px;
}

.cost-comparison {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 40px;
  flex-wrap: wrap;
}

.comparison-column {
  width: 40%;
  max-width: 380px;
  text-align: center;
}

.image-container {
  width: 100%;
  height: 320px;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.image-container img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.entity-label {
  font-family: "ivypresto-headline", serif;
  color: #999999; /* Gray from mockup */
  font-size: 28px;
  margin-bottom: 10px;
}

.price {
  font-family: "ivypresto-headline", serif;
  font-size: 36px;
  font-weight: 600;
  margin-bottom: 10px;
  color: #333333;
}

.description {
  font-family: neue-haas-grotesk-display, sans-serif;
font-weight: 400;
font-style: normal;  font-size: 18px;
  color: #333333;
}

.vs {
  font-family: "ivypresto-headline", serif;
  font-size: 64px;
  font-style: italic;
  color: #FF0000; /* Red from mockup */
  text-align: center;
  align-self: center;
}

.footnotes {
  max-width: 900px;
  margin: 60px auto 0;
  padding: 0 20px 40px;
  display: flex;
  flex-direction: column;
  gap: 5px; /* Ensures proper spacing between footnotes */
  position: relative; /* Changed from absolute */
}

.footnote {
  font-family: "neue-haas-grotesk-display", sans-serif;
  font-style: normal;
  font-weight: 300;
  font-size: 12px;
  color: #999999;
  line-height: 1.4;
  margin: 0; /* Reset margin */
  position: relative; /* Add this to ensure proper stacking */
  width: 100%; /* Ensure full width */
}
@media (max-width: 768px) {
  .main-title {
    font-size: 40px;
  }
  .lung-title {
    font-size: 24px;
  }
  .comparison-title {
    font-size: 30px;
  }
  
  .label {
    width: 120px;
    font-size: 18px;
  }
  
  .percentage {
    width: 90px;
    font-size: 16px;
  }
  
  .cost-comparison {
    flex-direction: column;
  }
  
  .comparison-column {
    width: 80%;
    margin-bottom: 30px;
  }
  
  .vs {
    margin: 20px 0;
  }
  
  .entity-label {
    font-size: 24px;
  }
  
  .price {
    font-size: 28px;
  }
}
</style>