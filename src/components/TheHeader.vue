<template>
  <nav class="navbar navbar-light">
    <div class="container">
      <router-link
        class="navbar-brand"
        :to="{ name: 'home', params: { tag: 'home-tag' } }"
      >
        conduit
      </router-link>
      <ul v-if="!isAuthenticated" class="nav navbar-nav pull-xs-right">
        <li class="nav-item">
          <router-link
            class="nav-link"
            active-class="active"
            exact
            :to="{ name: 'home', params: { tag: 'home-tag' } }"
          >
            Home
          </router-link>
        </li>
        <li class="nav-item">
          <router-link
            class="nav-link"
            active-class="active"
            exact
            :to="{ name: 'login' }"
          >
            Sign in
          </router-link>
        </li>
        <li class="nav-item">
          <router-link
            class="nav-link"
            active-class="active"
            exact
            :to="{ name: 'register' }"
          >
            Sign up
          </router-link>
        </li>
      </ul>
      <ul v-else class="nav navbar-nav pull-xs-right">
        <li class="nav-item">
          <router-link
            class="nav-link"
            active-class="active"
            exact
            :to="{ name: 'home' }"
          >
            Home
          </router-link>
        </li>
        <li class="nav-item">
          <router-link
            class="nav-link"
            active-class="active"
            :to="{ name: 'article-edit' }"
          >
            <i class="ion-compose"></i>&nbsp;New Article
          </router-link>
        </li>
        <li class="nav-item">
          <router-link
            class="nav-link"
            active-class="active"
            exact
            :to="{ name: 'settings' }"
          >
            <i class="ion-gear-a"></i>&nbsp;Settings
          </router-link>
        </li>
        <li class="nav-item" v-if="currentUser.username">
          <router-link
            class="nav-link"
            active-class="active"
            exact
            :to="{
              name: 'profile',
              params: { username: currentUser.username }
            }"
          >
            {{ currentUser.username }}
          </router-link>
        </li>
        <li class="nav-item" v-if="currentUser.username">
          <button @click="logout" class="btn btn-sm btn-outline-warning">
            logout
          </button>
        </li>
      </ul>
    </div>
  </nav>
</template>

<script>
import { mapGetters } from "vuex";
import { LOGOUT, FETCH_ARTICLES } from "@/store/actions.type";
export default {
  name: "RwvHeader",
  computed: {
    ...mapGetters(["currentUser", "isAuthenticated"]),
    listConfig() {
      const type = "all";
      const filters = {
        offset: 0,
        limit: 10
      };
      return {
        type,
        filters
      };
    }
  },
  methods: {
    logout() {
      this.$store.dispatch(LOGOUT).then(() => {
        this.$store.dispatch(FETCH_ARTICLES, this.listConfig);
      });
    }
  }
};
</script>
<style>
.navbar-nav {
  display: flex;
  align-items: center;
  flex-direction: row;
}
</style>
