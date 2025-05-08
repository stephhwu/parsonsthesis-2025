<template>
  <div class="app-container">
    <DogsWithJobs @guide-selected="handleGuideSelection" />
    <DogGuide v-if="showDogGuide" />
  </div>
</template>

<script>
import DogsWithJobs from './components/DogsWithJobs.vue'
import DogGuide from './components/DogGuide.vue'

export default {
  name: 'App',
  components: {
    DogsWithJobs,
    DogGuide
  },
  data() {
    return {
      showDogGuide: false
    }
  },
  methods: {
    handleGuideSelection(needsGuide) {
      console.log('Guide dog selected:', needsGuide);
      this.showDogGuide = needsGuide;
    }
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
}

.app-container {
  width: 100%;
  display: flex;
  flex-direction: column;
}
</style>
