<script setup lang="ts">
import { ref, computed, onMounted } from "vue";

// Tipe data untuk anime
interface Anime {
  mal_id: number;
  title: string;
  images: {
    jpg: {
      image_url: string;
    };
  };
  total_episodes: number;
  watched_episodes: number;
}

// Tipe data untuk hasil pencarian anime
interface AnimeSearchResult {
  mal_id: number;
  title: string;
  images: {
    jpg: {
      image_url: string;
    };
  };
  synopsis?: string;
  episodes: number;
}

const query = ref<string>("");
const my_anime = ref<Anime[]>([]);
const search_result = ref<AnimeSearchResult[]>([]);

// Mengurutkan anime berdasarkan judul
const my_anime_asc = computed(() => {
  return my_anime.value.slice().sort((a, b) => {
    return a.title.localeCompare(b.title);
  });
});

interface JikanResponse {
  data: AnimeSearchResult[];
}

const searchAnime = () => {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
  fetch(url)
    .then((res) => res.json())
    .then((res: JikanResponse) => {
      search_result.value = res.data;
    });
};

// Fungsi untuk menangani input kosong
const handleInput = (e: Event) => {
  const target = e.target as HTMLInputElement;
  if (!target.value) {
    search_result.value = [];
  }
};

// Fungsi untuk menambah anime ke dalam daftar "My Anime"
const addAnime = (anime: AnimeSearchResult) => {
  search_result.value = [];
  query.value = "";

  my_anime.value.push({
    mal_id: anime.mal_id,
    title: anime.title,
    images: anime.images,
    total_episodes: anime.episodes,
    watched_episodes: 0,
  });

  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

// Fungsi untuk menambah jumlah episode yang ditonton
const increaseWatch = (anime: Anime) => {
  anime.watched_episodes++;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

// Fungsi untuk mengurangi jumlah episode yang ditonton
const decreaseWatch = (anime: Anime) => {
  if (anime.watched_episodes > 0) {
    anime.watched_episodes--;
    localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
  }
};

// Mengambil data "My Anime" dari localStorage ketika komponen dimount
onMounted(() => {
  try {
    const savedAnime = localStorage.getItem("my_anime");
    if (savedAnime) {
      my_anime.value = JSON.parse(savedAnime);
    }
  } catch (error) {
    console.error("Error loading from localStorage", error);
  }
});
</script>

<template>
  <main>
    <h1>My Anime Tracker</h1>

    <form @submit.prevent="searchAnime">
      <input
        type="text"
        placeholder="Search for an anime..."
        v-model="query"
        @input="handleInput" />
      <button class="button" type="submit">Search</button>
    </form>
    <div class="results" v-if="search_result.length > 0">
      <div
        class="result"
        v-for="anime in search_result"
        :key="anime.mal_id">
        <img :src="anime.images.jpg.image_url" />
        <div class="details">
          <h3>
            {{ anime.title }}
          </h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...
          </p>
          <span class="flex-1"></span>
          <button @click="addAnime(anime)" class="button">
            Add to my anime
          </button>
        </div>
      </div>
    </div>
    <div class="myanime" v-if="my_anime.length > 0">
      <h2>My Anime</h2>
      <div
        v-for="anime in my_anime_asc"
        :key="anime.mal_id"
        class="anime">
        <img :src="anime.images.jpg.image_url" />
        <h3>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes">
          {{ anime.watched_episodes }} / {{ anime.total_episodes }}
        </span>
        <button
          v-if="anime.total_episodes !== anime.watched_episodes"
          @click="increaseWatch(anime)"
          class="button">
          +
        </button>
        <button
          v-if="anime.watched_episodes > 0"
          @click="decreaseWatch(anime)"
          class="button">
          -
        </button>
      </div>
    </div>
  </main>
</template>

<style>
/* Styles are the same as in your original code */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Fira sans", sans-serif;
}

body {
  background-color: #eee;
}

main {
  margin: 0 auto;
  max-width: 768px;
  padding: 1.5rem;
}

h1 {
  text-align: center;
  margin-bottom: 1.5rem;
}

form {
  display: flex;
  max-width: 480px;
  margin: 0 auto 1.5rem;
}

form input {
  appearance: none;
  outline: none;
  border: none;
  background-color: white;

  display: block;
  color: #888;
  font-size: 1.125rem;
  padding: 0.5rem 1rem;
  width: 100%;
}

.button {
  appearance: none;
  outline: none;
  border: none;
  cursor: pointer;
  background: none;

  display: block;
  padding: 0.5rem 1rem;
  background-image: linear-gradient(
    to right,
    deeppink 50%,
    darkviolet 50%
  );
  background-size: 200%;
  color: #fff;
  font-size: 1.125rem;
  font-weight: 700;
  text-transform: uppercase;
  transition: 0.4s;
}

.button:hover {
  background-position: right;
}

.results {
  background-color: #fff;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-height: 480px;
  overflow-y: scroll;
  margin-bottom: 1.5rem;
}

.result {
  display: flex;
  margin: 1rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
  transition: 0.4s;
}

.result img {
  width: 100px;
  border-radius: 1rem;
  margin-right: 1rem;
}

.details {
  flex: 1 1 0%;
  display: flex;
  align-items: flex-start;
  flex-direction: column;
}

.flex-1 {
  flex: 1 1 0%;
}

.details h3 {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
}

.details p {
  font-size: 0.875rem;
  line-height: 1.4;
  margin-bottom: 0.5rem;
}

.details .button {
  margin-left: auto;
}

.myanime h2 {
  color: #888;
  font-weight: 400;
  margin-bottom: 1.5rem;
}

.myanime .anime {
  display: flex;
  align-items: center;
  margin-bottom: 1.5rem;
  background-color: #fff;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}

.anime img {
  width: 72px;
  height: 72px;
  object-fit: cover;
  border-radius: 1rem;
  margin-right: 1rem;
}

.anime h3 {
  color: #888;
  font-size: 1.125rem;
}

.anime .episodes {
  margin-right: 1rem;
  color: #888;
}

.anime .button:first-of-type {
  margin-right: 1rem;
}

.anime .button:last-of-type {
  margin-right: 0;
}
</style>
