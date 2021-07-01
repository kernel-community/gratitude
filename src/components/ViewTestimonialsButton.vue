<template>
  <div class="font-sans">
    <div class="sm:p-0 pt-2 float-right ">
      <button
        class="text-sm sm:text-base bg-gray-50 font-fancy border-opacity-50 border-dashed border-b-2 border-gray-900"
        @click="$store.commit('togglePanel', true)"
      >
        View Testimonials
      </button>
    </div>
  </div>
</template>
<script>
export default {
  name: "ViewTestimonialsButton"
};
</script>
