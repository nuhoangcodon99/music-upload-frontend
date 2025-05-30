<template>
  <div class="min-h-screen bg-gray-50 text-black font-sans">
    <!-- Header -->
    <header class="fixed top-0 left-0 w-full bg-white shadow-sm p-3 flex items-center justify-between z-10">
      <div class="flex items-center">
        <!-- <img src="https://via.placeholder.com/40x40?text=SC" alt="Logo" class="w-8 h-8 mr-2"> -->
        <h1 class="text-lg font-bold text-black">NamP Music</h1>
      </div>
      <div class="flex-1 mx-2 sm:mx-4">
        <input 
          type="text" 
          placeholder="Search for tracks, artists..." 
          class="w-full max-w-md p-2 rounded-full bg-gray-100 border-none focus:outline-none focus:ring-2 focus:ring-orange-500 text-sm"
        >
      </div>
      <div class="flex items-center space-x-2 sm:space-x-3">
        <button class="hidden sm:block px-3 py-1 bg-orange-500 text-white rounded-full text-sm font-semibold hover:bg-orange-600 transition">Upload</button>
        <!-- <button class="text-black hover:text-orange-500">
          <svg class="w-5 h-5 sm:w-6 sm:h-6" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 2a10 10 0 100 20 10 10 0 000-20zm0 18a8 8 0 110-16 8 8 0 010 16zm-1-5h2v2h-2zm0-8h2v6h-2z"/>
          </svg>
        </button> -->
        <button class="md:hidden text-black hover:text-orange-500 focus:outline-none" @click="toggleMenu">
          <svg class="w-5 h-5 sm:w-6 sm:h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16m-7 6h7"></path>
          </svg>
        </button>
      </div>
      <!-- Menu mobile -->
      <div v-if="isMenuOpen" class="md:hidden absolute top-14 right-3 bg-white p-3 rounded-lg shadow-lg z-10">
        <a href="#" class="block text-black hover:text-orange-500 py-1 text-sm">Search</a>
        <a href="#" class="block text-black hover:text-orange-500 py-1 text-sm">Upload</a>
        <a href="#" class="block text-black hover:text-orange-500 py-1 text-sm">Profile</a>
      </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto pt-16 p-3 sm:p-4">
      <UploadForm @file-uploaded="refreshMusicList" />
      <MusicList ref="musicList" />
    </main>

    <!-- Footer -->
    <footer class="bg-gray-100 p-3 text-center text-gray-600">
      <p class="text-xs sm:text-sm">© 2025 Ngocanh Music App. All rights reserved.</p>
    </footer>
  </div>
</template>

<script>
import UploadForm from './components/UploadForm.vue'
import MusicList from './components/MusicList.vue'

export default {
  components: {
    UploadForm,
    MusicList
  },
  data() {
    return {
      isMenuOpen: false
    }
  },
  methods: {
    refreshMusicList() {
      this.$refs.musicList.fetchMusicList()
    },
    toggleMenu() {
      this.isMenuOpen = !this.isMenuOpen
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin-top: 60px;
}
</style>