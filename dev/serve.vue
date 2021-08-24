<script lang="ts">
import { defineComponent } from 'vue';
import VueStepperTouch from '@/VueStepperTouch.vue';

export default defineComponent({
  name: 'ServeDev',
  components: {
    VueStepperTouch
  }
});
</script>

<template>
  <div id="app">
    <VueStepperTouch
        class="m-1 px-3 border border-collapse border-black-900 text-center"
        axis="x"
        background="#6d72fe"
        :opacity=25
        color="#ffffff"
        fontFamily="Poppins"
        :min=0
        :max=100
        :initialValue=0
        :size=4
    />
  </div>
</template>

<style>
  body {
    background-color: #414379;
  }
  #app {
    position: absolute;
    left: 50%;
    top: 50%;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }
</style>
