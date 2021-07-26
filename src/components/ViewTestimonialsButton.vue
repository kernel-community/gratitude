<template>
  <div
    class="font-sans cursor-pointer p-2"
    @click="$store.commit('togglePanel', true)"
  >
    <button class="sm:p-0 pt-2 float-right">
      <button
        class="text-sm sm:text-base bg-gray-50 font-fancy border-opacity-50 border-dashed border-b-2 border-gray-900"
      >
        View Testimonials
      </button>
    </button>
  </div>
</template>
<script>
export default {
  name: "ViewTestimonialsButton"
};
</script>
