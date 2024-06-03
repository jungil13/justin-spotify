<template>
  <div class="min-h-screen flex flex-col bg-slate-800 text-white">
    <div
      class="flex flex-col flex-grow bg-violet-200 shadow-xl rounded-lg overflow-hidden border border-gray-400">
      <header class="bg-slate-900 py-4 border border-stone-500">
        <div class="flex ml-6">
          <img src="/logo.png" alt="Logo" class="w-10 h-10">
          <RouterLink class="text-2xl font-extrabold cursor-pointer font-sans text-white ml-6" to="/dashboard"><span class="text-green-300">SPOTICHAT</span></RouterLink>
          <div class="flex items-center">
          </div>
        </div>
      </header>
      <div class="flex flex-col flex-grow h-0 p-4 overflow-auto">
        <div v-for="msg in messages" :key="msg.id" class="flex w-full mt-2">
          <div v-if="msg.username === username" class="flex items-end ml-auto">
            <div class="bg-green-800 text-white p-3 rounded-l-lg rounded-br-lg">
              <!-- Display the message text -->
              <p class="text-sm text-white">{{ msg.text }}</p>
              <!-- If there's an embed URL, display the iframe -->
              <iframe v-if="msg.embedUrl" :src="msg.embedUrl" class="w-full" height="250" frameborder="0"
                allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                loading="lazy"></iframe>
              <span class="text-xs text-gray-300 leading-none">2 min ago</span>
            </div>
            <span class="text-sm text-gray-700 leading-none ml-3 font-bold mb-6">Me</span>
          </div>
          <div v-else class="flex items-end">
            <div class="bg-gray-800 p-3 rounded-r-lg rounded-bl-lg">
              <!-- Display the message text -->
              <p class="text-sm text-white">{{ msg.text }}</p>
              <!-- If there's an embed URL, display the iframe -->
              <iframe v-if="msg.embedUrl" :src="msg.embedUrl" class="w-full" height="250" frameborder="0"
                allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                loading="lazy"></iframe>
              <span class="text-xs text-gray-500 leading-none">2 min ago</span>
            </div>
            <span v-if="msg.username !== username" class="text-sm text-black leading-none ml-3 font-bold mb-6">{{
              msg.username }}</span>
          </div>
        </div>
      </div>
      <div class="bg-slate-900 p-4 flex items-center space-x-2 border border-stone-500">
        <input v-model="newMessage" placeholder="Type a message" @keyup.enter="sendMessage"
          class="flex-grow h-10 rounded px-3 text-sm rounded-lg text-black" />
        <button @click="sendMessage"
          class="bg-green-700 hover:bg-sky-900 focus:outline-none text-white py-2 px-4 rounded-xl border-2 border-blue-300 font-semibold">Send
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { io } from 'socket.io-client';
import { ref, onMounted } from 'vue';

export default {
  data() {
    return {
      socket: null,
      messages: [],
      newMessage: '',
      loggedInUsername: ''
    };
  },
  setup() {
    const username = ref('');
    const isLoggedIn = ref(false);

    const fetchDashboard = async () => {
      try {
        const response = await axios.get('http://localhost:3000/dashboard', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        });
        isLoggedIn.value = true;
        // Extract username from JWT token payload
        const token = localStorage.getItem('token');
        if (token) {
          try {
            const decodedToken = JSON.parse(atob(token.split('.')[1]));
            username.value = decodedToken.username;
          } catch (e) {
            // Handle token decode error
          }
        }
      } catch (error) {
        // Error fetching dashboard message
      }
    };

    onMounted(fetchDashboard);

    return { username, isLoggedIn };
  },
  mounted() {
    this.socket = io('http://localhost:3000');

    // Unbind any existing event listeners before binding a new one
    this.socket.off('chat message');

    this.socket.on('chat message', (msg) => {
      this.messages.push(msg);
      this.saveMessage(msg);
    });

    this.fetchMessages();
  },

  methods: {
    // Client-side code
    sendMessage() {
  if (this.newMessage.trim()) {
    // Check if the message is a Spotify link
    const spotifyRegex = /https:\/\/open\.spotify\.com\/embed\/track\/(\w+)/;
    const matches = this.newMessage.match(spotifyRegex);

    let messageData = {
      text: this.newMessage,
      username: this.username
    };

    // If it's a Spotify link and the regex match is successful, generate the embed URL
    if (matches && matches[1]) {
      const trackId = matches[1];
      messageData.track_id = trackId; // Assign the track_id
      messageData.embedUrl = `https://open.spotify.com/embed/track/${trackId}`;
    }

    // Send the message with or without the embed URL
    this.socket.emit('chat message', messageData);
    this.newMessage = '';
  }
},

    async fetchMessages() {
      try {
        const response = await axios.get('http://localhost:3000/api/messages');
        this.messages = response.data;
      } catch (error) {
        // Handle error
      }
    },
    saveMessage(msg) {
      axios.post('http://localhost:3000/api/messages', msg)
        .catch(error => {
          // Handle error
        });
    }
  },
  beforeDestroy() {
    this.socket.disconnect();
  }
};
</script>