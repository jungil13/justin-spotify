<template>
  <section class="app-main">
    <div class="min-h-screen flex bg-violet-200 text-black">
      <!-- Main Content -->
      <div class="flex-grow p-4 w-3/4">
        <header class="bg-slate-800 py-4 rounded-xl border border-gray-700 mb-4">
          <div class="container mx-auto px-4 flex flex-col lg:flex-row justify-between items-center">
            <div class="flex items-center">
              <img src="/logo.png" alt="Logo" class="w-10 h-10 mr-2">
              <h1 class="text-xl font-extrabold cursor-pointer text-green-400">SPOTICHAT</h1>
            </div>

            <div class="flex items-center">
              <p class="hidden lg:block mr-6 cursor-pointer text-white">Welcome, <span class="text-green-300 font-semibold">{{
                  username }}</span></p>
              <button @click="logout" class="bg-green-700 hover:bg-blue-600 rounded-full px-4 py-2 font-semibold text-white">Logout</button>
            </div>
          </div>
        </header>

        <!-- Search Input -->
        <div class="flex justify-center">
          <div class="flex w-30">
            <input @keyup.enter="search" v-model="searchQuery" type="text" placeholder="Search music . . ."
              class="flex-1 bg-gray-100 hover:bg-gray-700 border border-blue-800 text-black rounded-l-full px-4 py-2 font-semibold text-md">
            <button @click="search"
              class="bg-green-800 hover:bg-blue-800 border border-white text-white rounded-r-full px-4 py-2 font-semibold text-md">
              <i class="fas fa-search mr-1 text-lg"></i>
              <span class="hidden sm:inline-block">Search</span>
            </button>
          </div>
        </div>

        <!-- Top Songs Section -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-2 gap-6 mt-8">
          <div v-for="track in tracks" :key="track.id"
            class="bg-slate-900 rounded-lg overflow-hidden shadow-lg hover:shadow-xl transition duration-300 border border-gray-500">
            <iframe :src="generateEmbedUrl(track.id)" class="w-full" height="250" frameborder="" allowfullscreen=""
              allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
            <div class="p-3">
              <div class="font-bold text-lg text-white">{{ track.name }}</div>
              <p class="text-gray-400 mb-1">{{ track.artists[0].name }}</p>
              <button @click="shareMessage(track.id)" class="flex items-center">
                <i class="fas fa-share mr-1 text-green-400 text-lg"></i> <span
                  class="ml-2 font-bold font-sans text-white">Share</span>
              </button>
            </div>
          </div>
           <iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/4Rlbzn3nj9XJMZTijQUXMs?utm_source=generator" width="100%" height="352" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
          <iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/5pTvmkVXAxFbjAzoY2e7vj?utm_source=generator" width="100%" height="352" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
          <iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/7iBF9PuSNGYUxePK4w7b4l?utm_source=generator" width="100%" height="352" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
          <iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/1iIJtD9hkzw4ZHfR7ND9yb?utm_source=generator" width="100%" height="352" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
          <!-- Render your top songs here -->
        </div>
      </div>
      <!-- Right Drawer -->
          <div class="w-1/3 mr-12">
            <div class="rounded-lg fixed p-2 w-1/3 h-4/3 bg-zinc-900 border-l border-gray-700">
              <ul>
                <!-- List of top songs -->
                <li v-for="(song, index) in topSongs" :key="index" class="mb-4">
                  <div
                    :class="song.bgColor + ' rounded-lg overflow-hidden shadow-lg hover:shadow-xl transition duration-300'">
                    <MessageView />
                    </div>
                </li>
              </ul>
            </div>
          </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import Swal from 'sweetalert2';
import { useRouter } from 'vue-router';
import MessageView from './MessageView.vue';
import ArtistView from './ArtistView.vue';
import ArtistaView from './ArtistaView.vue';

const searchQuery = ref('');
const tracks = ref([]);
const isLoggedIn = ref(false);
const message = ref('');
const username = ref('');
const router = useRouter();

const topSongs = [
  { url: "https://open.spotify.com/embed/track/29WxJqIfDRMo9isV07kbJP", bgColor: "bg-violet-900" },
  { url: "https://open.spotify.com/embed/track/1iIJtD9hkzw4ZHfR7ND9yb", bgColor: "bg-pink-800" },
  { url: "https://open.spotify.com/embed/track/2v6jmF6VQWS96x6tSg05IC", bgColor: "bg-gray-400" },
  { url: "https://open.spotify.com/embed/track/5sbooPcNgIE22DwO0VNGUJ", bgColor: "bg-violet-400" }
];

const toggleDrawer = () => {
  isDrawerOpen.value = !isDrawerOpen.value;
};

const search = async () => {
  try {
    const response = await axios.get('http://localhost:3000/search', {
      params: {
        q: searchQuery.value
      }
    });
    tracks.value = response.data.tracks.items;
  } catch (error) {
    console.error('Error searching tracks:', error);
  }
};

const generateEmbedUrl = (trackId) => {
  return `https://open.spotify.com/embed/track/${trackId}`;
};

const shareMessage = (trackId) => {
  const embedUrl = generateEmbedUrl(trackId);
  const message = `Check out this track: ${embedUrl}`;

  const input = document.createElement('input');
  input.style.position = 'fixed';
  input.style.opacity = 0;
  input.value = message;
  document.body.appendChild(input);
  input.select();
  document.execCommand('copy');
  document.body.removeChild(input);

  alert('Link copied to clipboard!');
};

const logout = () => {
  console.log("Logged out!");
  localStorage.removeItem('token');
  router.push('/');
  Swal.fire({
    icon: 'success',
    title: 'Logged out successfully',
    position: 'top',
    showConfirmButton: false,
    timer: 1500,
    customClass: {
      popup: 'my-swal-popup',
      title: 'my-swal-title'
    }
  });
};

onMounted(async () => {
  try {
    const response = await axios.get('http://localhost:3000/dashboard', {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`
      }
    });
    message.value = response.data.message;
    isLoggedIn.value = true;

    const token = localStorage.getItem('token');
    const decodedToken = JSON.parse(atob(token.split('.')[1]));
    if (decodedToken) {
      username.value = decodedToken.username;
    }
  } catch (error) {
    console.error('Error fetching dashboard message:', error);
  }
});
</script>



<style>
.my-swal-popup {
  width: 200px;
  /* Adjust width as needed */
  background-color: #4CAF50;
  /* Custom background color */
  color: white;
  /* Custom text color */
}

.my-swal-title {
  font-size: 16px;
  /* Adjust font size as needed */
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter,
.slide-leave-to {
  transform: translateX(100%);
}
</style>
