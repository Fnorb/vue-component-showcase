<template>
  <div class="component-showcase">

    <!-- active component -->
    <component :is="components[activeComponent].component" />

    <!-- overlay-->
    <div class="comp-selection">

      <!-- selection buttons -->
      <div class="buttons">
        <button v-for="(comp, key) in components" :key="key" @click="activeComponent = key" :class="{ active: activeComponent === key }">
          {{ comp.name }}
        </button>
      </div>

      <!-- short description of the component -->
      <div class="description">
        <p>{{ components[activeComponent].description }}</p>
      </div>
    </div>

  </div>
</template>

<script setup>
import 'reset-CSS'
import { ref, shallowRef } from 'vue'
import AnimatedSquaresBackground from './components/AnimatedSquaresBackground.vue'
import ParticlesBackgroundPlaceholder from './components/ParticlesBackgroundPlaceholder.vue'
import TextAnimatorPlaceholder from './components/TextAnimatorPlaceholder.vue'

// Component list with description
const components = shallowRef({
  AnimatedSquares: {
    component: AnimatedSquaresBackground,
    name: 'Animated Squares',
    description: 'A dynamic background with animated squares that change over time.',
  },
  ParticlesBackground: {
    component: ParticlesBackgroundPlaceholder,
    name: 'Particles Background',
    description: 'Placeholder',
  },
  TextAnimator: {
    component: TextAnimatorPlaceholder,
    name: 'Text Animator',
    description: 'Placeholder',
  },
});

const activeComponent = ref('AnimatedSquares');
</script>

<style scoped lang="stylus">

@font-face
  font-family 'Inter'
  src url('/Inter-VariableFont_opsz,wght.ttf') format('truetype')
  font-weight 100 900 
  font-display swap

.component-showcase
  font-family: 'Inter', sans-serif
  position absolute
  top 0
  left 0
  width 100%
  height 100%
  color white

.comp-selection
  position absolute
  max-width 80%
  display flex
  gap 10px
  top 10px
  left 50%
  transform translateX(-50%)
  flex-direction column

  .buttons
    display flex
    gap 10px
    white-space nowrap

.description
  margin-top 20px
  font-size 1rem
  text-align center
  position relative
  display block
  font-size 1.1em
  filter drop-shadow(0px 4px 3px rgba(0, 0, 0, 0.5))

.buttons button
  padding 0.8em 1.5em
  background-color #42b883
  border-radius 20px
  font-size 1.1em
  color rgb(255, 255, 255)
  font-weight 500
  border none
  cursor pointer
  box-shadow 0 4px 6px rgba(0, 0, 0, 0.1)
  transition all 0.3s ease

  &.active
    background-color #358e6e
    box-shadow 0 8px 12px rgba(0, 0, 0, 0.2)

  &:hover
    background-color #358e6e
    box-shadow 0 8px 12px rgba(0, 0, 0, 0.2)

</style>
