<template>
  <div>
    <div v-if="isLoading" class="article-preview">Loading articles...</div>
    <div v-else>
      <div
        class="article-preview"
        v-for="(article, index) in articles"
        :key="article.title + index"
      >
        <div class="article-meta">
          <router-link
            :to="{ name: 'profile', params: { username: article.author.username } }"
          >
            <img :src="article.author.image" />
          </router-link>
          <div class="info">
            <router-link
              :to="{ name: 'profile', params: { username: article.author.username } }"
              class="author"
            >
              {{ article.author.username }}
            </router-link>
            <span class="date">{{ article.createdAt | date }}</span>
          </div>
          <span v-if="isCurrentUser(article)">
            <router-link class="btn btn-sm btn-outline-secondary" :to="editArticleLink">
              <i class="ion-edit"></i> <span>&nbsp;Edit Article</span>
            </router-link>
            <span>&nbsp;&nbsp;</span>
            <button class="btn btn-outline-danger btn-sm" @click="deleteArticle">
              <i class="ion-trash-a"></i> <span>&nbsp;Delete Article</span>
            </button>
          </span>
          <!-- Used in ArticleView when not author -->
          <span v-else>
            <button class="btn btn-sm btn-outline-secondary" @click="toggleFollow">
              <i class="ion-plus-round"></i> <span>&nbsp;</span>
              <span v-text="followUserLabel(article)" />
            </button>
            <span>&nbsp;&nbsp;</span>
            <button
              class="btn btn-sm"
              @click="toggleFavorite"
              :class="toggleFavoriteButtonClasses(article)"
            >
              <i class="ion-heart"></i> <span>&nbsp;</span>
              <span v-text="favoriteArticleLabel(article)" /> <span>&nbsp;</span>
              <span class="counter" v-text="favoriteCounter(article)" />
            </button>
          </span>
          <button
            v-else
            class="btn btn-sm pull-xs-right"
            @click="toggleFavorite"
            :class="{
              'btn-primary': article.favorited,
              'btn-outline-primary': !article.favorited
            }"
          >
            <i class="ion-heart"></i>
            <span class="counter"> {{ article.favoritesCount }} </span>
          </button>
        </div>

        <router-link :to="articleLink(article)" class="preview-link">
          <h1 v-text="article.title" />
          <p v-text="article.description" />
          <span>Read more...</span>
          <TagList :tags="article.tagList" />
        </router-link>
      </div>
      <VPagination :pages="pages" :currentPage.sync="currentPage" />
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import VPagination from "./VPagination";
import TagList from "./TagList";

import {
  FAVORITE_ADD,
  FAVORITE_REMOVE,
  FETCH_ARTICLES,
  ARTICLE_DELETE,
  FETCH_PROFILE_FOLLOW,
  FETCH_PROFILE_UNFOLLOW,
} from "@/store/actions.type";

export default {
  name: "RwvArticleList",
  components: {
    VPagination,
    TagList
  },
  props: {
    type: {
      type: String,
      required: false,
      default: "all"
    },
    author: {
      type: String,
      required: false
    },
    tag: {
      type: String,
      required: false
    },
    favorited: {
      type: String,
      required: false
    },
    itemsPerPage: {
      type: Number,
      required: false,
      default: 10
    },
    actions: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  data() {
    return {
      currentPage: 1
    };
  },
  computed: {
    listConfig() {
      const { type } = this;
      const filters = {
        offset: (this.currentPage - 1) * this.itemsPerPage,
        limit: this.itemsPerPage
      };
      if (this.author) {
        filters.author = this.author;
      }
      if (this.tag) {
        filters.tag = this.tag;
      }
      if (this.favorited) {
        filters.favorited = this.favorited;
      }
      return {
        type,
        filters
      };
    },
    pages() {
      if (this.isLoading || this.articlesCount <= this.itemsPerPage) {
        return [];
      }
      return [
        ...Array(Math.ceil(this.articlesCount / this.itemsPerPage)).keys()
      ].map(e => e + 1);
    },
    editArticleLink() {
      return { name: "article-edit", params: { slug: this.article.slug } };
    },
    ...mapGetters(["articlesCount", "isLoading", "articles", "currentUser", "profile", "isAuthenticated"])
  },

  watch: {
    currentPage(newValue) {
      this.listConfig.filters.offset = (newValue - 1) * this.itemsPerPage;
      this.fetchArticles();
    },
    type() {
      this.resetPagination();
      this.fetchArticles();
    },
    author() {
      this.resetPagination();
      this.fetchArticles();
    },
    tag() {
      this.resetPagination();
      this.fetchArticles();
    },
    favorited() {
      this.resetPagination();
      this.fetchArticles();
    }
  },
  mounted() {
    this.fetchArticles();
  },

  methods: {
    fetchArticles() {
      this.$store.dispatch(FETCH_ARTICLES, this.listConfig);
    },
    resetPagination() {
      this.listConfig.offset = 0;
      this.currentPage = 1;
    },
    isCurrentUser(article) {
      if (this.currentUser.username && article.author.username) {
        return this.currentUser.username === article.author.username;
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
    articleLink(article) {
      return {
        name: "article",
        params: {
          slug: article.slug
        }
      };
    },
    followUserLabel(article) {
      return `${this.profile.following ? "Unfollow" : "Follow"} ${
        article.author.username
      }`;
    },
    favoriteArticleLabel(article) {
      return article.favorited ? "Unfavorite Article" : "Favorite Article";
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
    },
  }
};
</script>
