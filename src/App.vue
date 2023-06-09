<!-- Author: Sebastian Aguirre Duque
E-mail: sadw621@gmail.com -->


<template>
  <div class='app'>

    <!-- Content -->
    <article class="content">
      <h1 class="content__title">Search GitHub Users</h1>

      <!-- Search -->
      <form class="search" @submit.prevent="doSearch">
        <input type="text" class="search__input" required placeholder="Search GitHub Users" v-model="user">
        <input type="submit" class="search__submit" value="Search">
      </form>

      <!-- Result -->
      <div class="result" v-if="result">
        <a v-if="isFavorite" href="#" class="result__toggle-favorite" @click="removeFavorite">Remove from favorites</a>
        <a v-else href="#" class="result__toggle-favorite" @click="addFavorite">Add to favorites</a>
        <h2 class="result_name">{{ userInfo.name }}</h2>
        <img v-bind:src="userInfo.avatar_url" v-bind:alt="userInfo.name" class="result__avatar">
        <p class="result__bio">
          {{ userInfo.bio }}
          <br>
          <a :href="userInfo.blog" target="_blank" class="result__blog">{{ userInfo.blog }}</a>
        </p>
      </div>

      <!-- Error -->
      <div class="result__error" v-if="error">Something went wrong: {{ error }}</div>
    </article>

    <!-- Favorites -->
    <div class="favorites">
      <!-- <div class="favorite" v-for="([, favorite], index) in favorites" :key="index"> -->
      <!-- ? Aquí desestructura el Map para obtener solo sus values -->
      <div class="favorite" v-for="(favorite, index) in allFavorites" :key="index">
        <a href="#" @click="() => showFavorite(favorite)">
          <img :src="favorite.avatar_url" :alt="favorite.name" class="favorite__avatar">
        </a>
      </div>
    </div>

  </div>

  <!-- <RouterView /> -->
</template>

<script>
// import { RouterView } from 'vue-router';
import axios from 'axios';

import url from './assets/GitHubAPI';

export default {
  name: 'GitHubSearch',
  data() {
    return {
      url: url,
      user: '',
      userInfo: {},
      result: null,
      error: null,
      favorites: new Map(),
    }
  },
  methods: {
    async doSearch() {
      this.result = this.error = null;
      try {
        const response = await axios.get(`${this.url}` + `${this.user}`);
        this.userInfo = response.data;
        this.result = true;
        return () => {
          console.log(response);
        }
      } catch (err) {
        return this.error = err;
      } finally {
        return this.user = '';
      }
    },
    addFavorite() {
      this.favorites.set(this.userInfo.id, this.userInfo);
      this.updateStorage();
    },
    removeFavorite() {
      this.favorites.delete(this.userInfo.id);
    },
    showFavorite(favorite) {
      this.userInfo = favorite;
    }
  },
  computed: {
    isFavorite() {
      return this.favorites.has(this.userInfo.id);
    },
    allFavorites() {
      return Array.from(this.favorites.values());
    },
    updateStorage() {
      window.localStorage.setItem('favorites', JSON.stringify(this.allFavorites));
    }
  },
  mounted() {

  },
  created() {
    const getFavs = JSON.parse(window.localStorage.getItem('favorites'));
    if (getFavs?.length > 0) {
      const favorites = new Map(getFavs.map(favorite => [favorite.id, favorite]));
      this.favorites = favorites;
    }
  }
}
</script>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  width: 80%;
  align-items: center;
  justify-content: center;
  margin: auto;
  gap: 2.5rem;
}

/* Content */
.content {
  background-color: #bc4ed8;
  width: 70%;
  padding: 1rem;
  border-radius: 1rem;
  margin-top: 2rem;
}

.content__title {
  margin: 1rem;
  text-align: center;
  color: #ffffff;
}

/* Search */
.search {
  display: flex;
  margin-bottom: 2.5rem;
}

.search__input {
  flex: 1;
  padding: 1rem;
  font-size: 1rem;
}

/* Result */
.result {
  position: relative;
  background-color: #6b188a;
  border-radius: 0.3rem;
  box-shadow: 2px 2px 3px gray;
  color: white;
  padding: 2.5rem;
  display: grid;
  gap: 1rem;
  grid-template-areas:
    "name name"
    "avatar bio";
}

.result__toggle-favorite {
  position: absolute;
  top: 0.3rem;
  right: 0.3rem;
  border: none;
  color: white;
  text-decoration: none;
  padding: 0.4rem;
}

.result__name {
  grid-area: name;
  margin: 0.4rem 0;
}

.result__avatar {
  grid-area: avatar;
  max-width: 6rem;
  height: auto;
  border-radius: 1rem;
}

.result__bio {
  grid-area: bio;
  margin: 0;
}

.result__blog {
  grid-area: blog;
  color: #d8bc4e;
}

.result__error {
  padding: 0.3rem;
  background-color: tomato;
  color: white;
  text-align: center;
  border: 1px solid red;
}

/* Favorites */
.favorites {
  width: 50%;
  display: flex;
  flex-direction: row;
  gap: 0.5em;
  flex-wrap: wrap;
}

.favorite {
  transition: transform 0.3s ease-out;
}

.favorite__avatar {
  height: 5rem;
  margin: 0.3rem;
  border: 1px solid #ffffff;
  border-radius: 10px;
}

.favorite--selected {
  transform: scale(1.3);
}

/* Transitions */
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.list-move,
/* apply transition to moving elements */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}
</style>
