<template>
  <div class="article-meta">
    <router-link
      :to="{
        name: 'profile',
        params: { username: author.username }
      }"
    >
      <img :src="author.image" />
    </router-link>
    <div class="info">
      <router-link
        :to="{
          name: 'profile',
          params: { username: author.username }
        }"
        class="author"
      >
        {{ author.username }}
      </router-link>
      <span class="date">{{ article.createdAt | date }}</span>
    </div>
    <span v-if="isCurrentUser(article)" class="pull-xs-right">
      <router-link
        class="btn btn-sm btn-outline-secondary"
        :to="editArticleLink"
      >
        <i class="ion-edit"></i> <span>&nbsp;Edit Article</span>
      </router-link>
      <span>&nbsp;&nbsp;</span>
      <button class="btn btn-outline-danger btn-sm" @click="deleteArticle">
        <i class="ion-trash-a"></i> <span>&nbsp;Delete Article</span>
      </button>
    </span>
    <!-- Used in ArticleView when not author -->
    <span v-else class="pull-xs-right">
      <!-- <button
              class="btn btn-sm btn-outline-secondary"
              @click="toggleFollow"
            >
              <i class="ion-plus-round"></i> <span>&nbsp;</span>
              <span v-text="followUserLabel(article)" />
            </button> -->
      <!-- <span>&nbsp;&nbsp;</span> -->
      <button
        class="btn btn-sm pull-xs-right"
        @click="toggleFavorite"
        :class="toggleFavoriteButtonClasses(article)"
      >
        <i class="ion-heart"></i> <span>&nbsp;</span>
        <!-- <span v-text="favoriteArticleLabel(article)" /> <span>&nbsp;</span> -->
        <span class="counter" v-text="favoriteCounter(article)"></span>
      </button>
    </span>
    <!-- <button
      class="btn btn-sm pull-xs-right"
      @click="toggleFavorite"
      :class="{
        'btn-primary': article.favorited,
        'btn-outline-primary': !article.favorited
      }"
    >
      <i class="ion-heart"></i>
      <span class="counter"> {{ article.favoritesCount }} </span>
    </button> -->
  </div>
</template>
<script>
import { mapGetters } from "vuex";
import {
  FAVORITE_ADD,
  FAVORITE_REMOVE,
  ARTICLE_DELETE,
  FETCH_PROFILE_FOLLOW,
  FETCH_PROFILE_UNFOLLOW
} from "@/store/actions.type";
export default {
  props: {
    article: {
      type: Object,
      required: true,
      default: null
    },
    author: {
      type: Object,
      required: true,
      default: null
    }
  },
  computed: {
    ...mapGetters(["currentUser", "isAuthenticated"])
  },

  methods: {
    isCurrentUser(article) {
      if (this.currentUser.username && article.username) {
        return this.currentUser.username === article.username;
      }
      return false;
    },
    toggleFavorite() {
      if (!this.isAuthenticated) {
        this.$router.push({ name: "login" });
        return;
      }
      const action = this.article.favorited ? FAVORITE_REMOVE : FAVORITE_ADD;
      this.$store.dispatch(action, this.article.slug);
    },
    toggleFollow() {
      if (!this.isAuthenticated) {
        this.$router.push({ name: "login" });
        return;
      }
      const action = this.article.following
        ? FETCH_PROFILE_UNFOLLOW
        : FETCH_PROFILE_FOLLOW;
      this.$store.dispatch(action, {
        username: this.profile.username
      });
    },
    toggleFavoriteButtonClasses(article) {
      return {
        "btn-primary": article.favorited,
        "btn-outline-primary": !article.favorited
      };
    },
    favoriteCounter(article) {
      return `(${article.favoritesCount})`;
    },
    async deleteArticle() {
      try {
        await this.$store.dispatch(ARTICLE_DELETE, this.article.slug);
        this.$router.push("/");
      } catch (err) {
        console.error(err);
      }
    }
  }
};
</script>
