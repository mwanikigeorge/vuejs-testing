<template>
  <router-link :to="link" class="preview-link">
    <h1 v-text="article.title" />
    <p v-text="article.description" />
    <span>Read more...</span>
    <!-- <TagList :tags="article.tagList" /> -->
  </router-link>
</template>
<script>
export default {
  props: {
    article: {
      type: Object,
      required: true
    },
    link: {
      type: Object,
      required: true
    }
  },
  computed: {
    articleLink() {
      return {
        name: "article",
        params: {
          slug: this.article.slug
        }
      };
    }
  }
};
</script>
