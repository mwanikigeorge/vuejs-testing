<template>
  <router-link
    :to="homeRoute"
    :class="className"
    v-text="tagname"
  ></router-link>
</template>

<script>
export default {
  name: "RwvTag",
  props: {
    tagname: {
      type: String,
      required: false
    },
    className: {
      type: String,
      default: "tag-pill tag-default"
    }
  },
  computed: {
    homeRoute: (tagname = this.tagname) => ({
      name: "home-tag",
      params: { tag: tagname }
    })
  }
};
</script>
