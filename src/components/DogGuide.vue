<template>
  <div class="dog-guide-container">
    <!-- Replace div leash with SVG element -->
    <svg id="leash-svg" width="100%" height="100%">
      <path id="leash-path" fill="none" stroke="#F9D74A" stroke-width="2"/>
    </svg>
    <img id="dog" src="@/assets/images/dog.gif" alt="Dog Guide" />
    <div id="cursor"></div>
  </div>
</template>

<script>
export default {
  name: 'DogGuide',
  mounted() {
    this.initDogSimulation();
  },
  beforeUnmount() {
    // Clean up event listeners when component is removed
    document.removeEventListener('mousemove', this.handleMouseMove);
    window.removeEventListener('resize', this.handleResize);
    cancelAnimationFrame(this.animationFrame);
  },
  data() {
    return {
      dog: null,
      leashPath: null,
      cursor: null,
      dogX: 0,
      dogY: 0,
      mouseX: 0,
      mouseY: 0,
      dogSpeed: 0,
      maxSpeed: 5,
      acceleration: 0.2,
      deceleration: 0.1,
      isMoving: false,
      lastMouseX: 0,
      lastMouseY: 0,
      dogDirection: 0,
      animationFrame: null,
      facingRight: false // Track the dog's facing direction
    };
  },
  methods: {
    initDogSimulation() {
      this.dog = document.getElementById('dog');
      this.leashPath = document.getElementById('leash-path');
      this.cursor = document.getElementById('cursor');
      
      // Dog's initial position
      this.dogX = window.innerWidth / 2;
      this.dogY = window.innerHeight * 0.7;
      
      // Mouse position
      this.mouseX = window.innerWidth / 2;
      this.mouseY = window.innerHeight / 2;
      this.lastMouseX = this.mouseX;
      this.lastMouseY = this.mouseY;
      
      // Update cursor position on mouse move - FIX FOR SCREEN RECORDING
      this.handleMouseMove = (e) => {
        // Fixed scaling for 1920x1080 recording
        this.mouseX = e.clientX;
        this.mouseY = e.clientY;
        
        // Check if mouse is moving
        if (Math.abs(this.mouseX - this.lastMouseX) > 1 || 
            Math.abs(this.mouseY - this.lastMouseY) > 1) {
          this.isMoving = true;
        } else {
          this.isMoving = false;
        }
        
        this.lastMouseX = this.mouseX;
        this.lastMouseY = this.mouseY;
      };
      
      document.addEventListener('mousemove', this.handleMouseMove);
      
      // Initial dog positioning
      this.dog.style.left = (this.dogX - 30) + 'px';
      this.dog.style.top = (this.dogY - 20) + 'px';
      
      // Start animation
      this.animate();
      
      // Add window resize handler to recalculate positions
      window.addEventListener('resize', this.handleResize);
    },
    
    handleResize() {
      // Recalculate dog position based on viewport size
      this.dogX = Math.min(this.dogX, window.innerWidth - 40);
      this.dogY = Math.min(this.dogY, window.innerHeight - 40);
    },
    
    animate() {
      // Update cursor position
      this.cursor.style.left = this.mouseX + 'px';
      this.cursor.style.top = this.mouseY + 'px';
      
      // Calculate distance between dog and mouse
      const dx = this.mouseX - this.dogX;
      const dy = this.mouseY - this.dogY;
      const distance = Math.sqrt(dx * dx + dy * dy);
      
      // Calculate angle for dog direction
      this.dogDirection = Math.atan2(dy, dx);
      
      // If mouse is moving, accelerate dog
      if (this.isMoving && distance > 50) {
        this.dogSpeed = Math.min(this.dogSpeed + this.acceleration, this.maxSpeed);
      } else {
        // Decelerate dog if mouse stops or is close enough
        this.dogSpeed = Math.max(this.dogSpeed - this.deceleration, 0);
      }
      
      // Move dog if it has speed
      if (this.dogSpeed > 0.1) {
        this.dogX += Math.cos(this.dogDirection) * this.dogSpeed;
        this.dogY += Math.sin(this.dogDirection) * this.dogSpeed;
        
        // Flip dog GIF based on direction
        // Since the original GIF faces left, we need to flip when going right
        if (dx > 0) {
          // Moving right, so flip the dog
          this.dog.style.transform = 'scaleX(-1)';
          this.facingRight = true;
        } else {
          // Moving left, keep original orientation
          this.dog.style.transform = 'scaleX(1)';
          this.facingRight = false;
        }
      }
      
      // Position dog
      this.dog.style.left = (this.dogX - 30) + 'px';
      this.dog.style.top = (this.dogY - 20) + 'px';
      
      // Draw leash as a curved path
      const leashLength = Math.sqrt(
        Math.pow(this.mouseX - this.dogX, 2) + Math.pow(this.mouseY - this.dogY, 2)
      );

      // Calculate the slack amount based on distance
      const slackAmount = Math.min(leashLength * 0.2, 50); // More slack for longer distances, max 50px

      // Calculate leash starting point based on dog's direction
      // Add an offset to the right of the dog (or left if flipped)
      const leashOffsetX = 15; // Horizontal offset from dog center
      const leashOffsetY = -5; // Vertical offset (slightly higher than center)

      let leashStartX, leashStartY;

      if (this.facingRight) {
        // When dog is facing right (flipped), attach to his right side (which is visually the left side of the original image)
        leashStartX = this.dogX - leashOffsetX;
        leashStartY = this.dogY + leashOffsetY;
      } else {
        // When dog is facing left (not flipped), attach to his right side
        leashStartX = this.dogX + leashOffsetX;
        leashStartY = this.dogY + leashOffsetY;
      }

      // Calculate the midpoint between dog and cursor
      const midX = (leashStartX + this.mouseX) / 2;
      const midY = (leashStartY + this.mouseY) / 2;

      // Add some gravity effect by moving the control point downward
      const controlX = midX;
      const controlY = midY + slackAmount;

      // Create the path data for a quadratic bezier curve
      // Starting from the adjusted leash position to the cursor position with a control point for the curve
      const pathData = `M ${leashStartX} ${leashStartY} Q ${controlX} ${controlY} ${this.mouseX} ${this.mouseY}`;
      this.leashPath.setAttribute('d', pathData);
      
      this.animationFrame = requestAnimationFrame(this.animate);
    }
  }
}
</script>

<style scoped>
.dog-guide-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
  pointer-events: none;
}

#dog {
  position: absolute;
  width: 80px; /* You may need to adjust this size based on your GIF */
  height: auto; /* Maintain aspect ratio */
  transition: transform 0.1s ease;
  pointer-events: none;
  transform-origin: center center;
}

#leash-svg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 999;
}

#leash-path {
  stroke: #A2A2A2;
  stroke-width: 2px;
  fill: none;
}

#cursor {
  position: absolute;
  width: 20px;
  height: 20px;
  background-color: #81C9E3;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  z-index: 100;
  pointer-events: none;
}
</style>