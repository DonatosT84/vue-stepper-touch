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
        axis="y"
        background="#6d72fe"
        :opacity=80
        color="#fff"
        fontFamily="Poppins"
        :min=0
        :max=100
        :initialValue="value"
        :size=4
    />
  </div>
</template>
