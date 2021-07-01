<template>
  <section>
    <p class="pl-5 sm:pl-0 font-heading font-bold text-4xl sm:text-5xl">
      {{ name }}
    </p>
    <p v-if="award" class="pl-5 sm:pl-0 font-award text-4xl sm:mt-5 mt-0">
      {{ award }}
    </p>
    <div class="pl-5 sm:pl-0 font-fancy text-gray-400 text-lg sm:p-0 pt-3">
      fancy a
      <span class="tweet-link">
        <a :href="tweetText" target="_new" class="text-blue-500"
          >tweet <font-awesome-icon :icon="['fab', 'twitter']"
        /></a>
        ?
      </span>
    </div>
  </section>
</template>
<script>
export default {
  props: {
    tweetText: {
      type: String,
      required: true
    },
    name: {
      type: String,
      required: true
    },
    award: {
      type: String,
      default: ""
    }
  }
};
</script>
