<template>
  <div class="font-sans">
    <div class="sm:p-0 pt-2 float-right cursor-pointer">
      <span
        class="text-sm sm:text-base pt-3 font-fancy cursor-pointer border-opacity-50 border-dashed border-b-2 border-gray-900"
        @click="$store.commit('togglePanel', true)"
      >
        View Testimonials
      </span>
    </div>
  </div>
</template>
<script>
export default {
  name: "ViewTestimonialsButton"
};
</script>
