<template>
  <div>
    <div v-if="isLoading" class="article-preview">Loading articles...</div>
    <div v-else>
      <div
        class="article-preview"
        v-for="(article, index) in articles"
        :key="article.title + index"
      >
        <ArticleMeta :article="article" />
        <ArticlePreview :article="article" :link="articleLink(article)" />
      </div>
      <VPagination :pages="pages" :currentPage.sync="currentPage" />
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import VPagination from "./VPagination";
import TagList from "./TagList";
import ArticlePreview from "./ArticlePreview";
import ArticleMeta from "./ArticleMeta";

import {
  FETCH_ARTICLES,
  FETCH_PROFILE_FOLLOW,
  FETCH_PROFILE_UNFOLLOW
} from "@/store/actions.type";

export default {
  name: "RwvArticleList",
  components: {
    VPagination,
    TagList,
    ArticlePreview,
    ArticleMeta
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
    ...mapGetters([
      "articlesCount",
      "isLoading",
      "articles",
      "currentUser",
      "profile",
      "isAuthenticated"
    ])
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
    // toggleFavorite() {
    //   if (!this.isAuthenticated) {
    //     this.$router.push({ name: "login" });
    //     return;
    //   }
    //   const action = this.article.favorited ? FAVORITE_REMOVE : FAVORITE_ADD;
    //   this.$store.dispatch(action, this.article.slug);
    // },
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
    }
    // favoriteArticleLabel(article) {
    //   return article.favorited ? "Unfavorite Article" : "Favorite Article";
    // },
    // favoriteCounter(article) {
    //   return `(${article.favoritesCount})`;
    // }
  }
};
</script>
