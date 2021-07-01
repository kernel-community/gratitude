<template>
  <section
    class="animate-pulse text-sm bg-gradient-to-r from-blue-50 via-blue-100 to-blue-50 py-5 px-5 shadow-md rounded-xl m-5 ring-4 ring-blue-100 font-lato font-light"
  >
    <p>
      {{ text }}
    </p>
    <p v-if="transactionHash" class="mt-2">
      <span class="font-mono">
        <a :href="linkToTxHash" target="_new" >
          {{ transactionHash.substring(0, 10) + "..." }}
          <font-awesome-icon :icon="['fas', 'external-link-alt']" />
        </a>
      </span>
    </p>
  </section>
</template>
<script>
export default {
  name: "PulsingInfoText",
  props: ["text", "transactionHash", "linkToTxHash"]
};
</script>
