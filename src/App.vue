<template>
  <div id="app">
    <!-- Vue Router will put the content that matches the current URL. -->
    <router-view />
  </div>
</template>
<script>
export default {
  metaInfo: {
    title: "",
    titleTemplate: "Thank You%s",
    htmlAttrs: {
      lang: "en",
      amp: true
    }
  }
};
</script>
