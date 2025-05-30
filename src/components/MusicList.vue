<template>
  <div class="bg-white rounded-lg shadow-sm p-3 sm:p-4">
    <h2 class="text-lg sm:text-xl font-semibold text-black mb-2 sm:mb-3">Your Music</h2>
    <div v-if="musicList.length === 0" class="text-gray-500 text-center text-sm sm:text-base">No music uploaded yet.</div>
    <div v-else class="space-y-3 sm:space-y-4">
      <div v-for="music in musicList" :key="music" class="p-2 sm:p-3 bg-white border-b border-gray-200 hover:bg-gray-50 transition-all duration-300">
        <div class="flex items-start">
          <!-- Nút Play/Pause -->
          <button 
            @click="togglePlay(music)" 
            class="w-10 h-10 flex items-center justify-center bg-[#f50] text-white rounded-full mr-2 hover:bg-[#ff7733] transition"
          >
            <svg v-if="!playingTrack || playingTrack !== music" class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M8 5v14l11-7z"/>
            </svg>
            <svg v-else class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M6 4h4v16H6zm8 0h4v16h-4z"/>
            </svg>
          </button>
          <!-- Nút Repeat -->
          <button 
            @click="toggleRepeat(music)" 
            :class="repeatTracks[music] ? 'bg-[#f50] hover:bg-[#ff7733]' : 'bg-gray-300 hover:bg-gray-400'" 
            class="w-10 h-10 flex items-center justify-center rounded-full mr-3 transition-colors duration-300"
          >
            <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M7 7h10v3l4-4-4-4v3H5v6h2V7zm10 10H7v-3l-4 4 4 4v-3h12v-6h-2v4z"/>
            </svg>
          </button>
          <div class="flex-1">
            <span class="text-black font-medium text-base truncate">{{ music }}</span>
            <div v-if="playingTrack === music" :id="'waveform-' + music.replace(/[^a-zA-Z0-9]/g, '')" class="mt-2"></div>
            <div 
              v-if="playingTrack === music"
              class="relative h-1 w-full bg-gray-200 rounded cursor-pointer mt-2"
              @click="seekTrack($event, music)"
            >
              <div 
                class="absolute h-full bg-[#f50] rounded" 
                :style="{ width: (progressMap[music] || 0) + '%' }"
              ></div>
            </div>
            <div v-if="playingTrack === music" class="flex justify-between text-xs text-gray-600 mt-1">
              <span>{{ formatTime(currentTimes[music] || 0) }}</span>
              <span>{{ formatTime(durations[music] || 0) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { reactive } from 'vue'

export default {
  data() {
    return {
      musicList: [],
      playingTrack: null,
      progressMap: reactive({}),
      currentTimes: reactive({}),
      durations: reactive({}),
      savedTimes: reactive({}),
      repeatTracks: reactive({}), // Lưu trạng thái Repeat cho từng bài
      audio: null,
      waveforms: {},
    }
  },
  methods: {
    async fetchMusicList() {
      try {
        const response = await axios.get('https://music-upload-backend.onrender.com/music')
        this.musicList = response.data.files
        this.musicList.forEach(music => {
          if (!(music in this.progressMap)) {
            this.progressMap[music] = 0
            this.currentTimes[music] = 0
            this.durations[music] = 0
            this.savedTimes[music] = 0
            this.repeatTracks[music] = false // Khởi tạo trạng thái Repeat
          }
        })
      } catch (error) {
        console.error('Error fetching music list:', error)
      }
    },
    getMusicUrl(filename) {
      return `https://music-upload-backend.onrender.com/uploads/${filename}`
    },
    formatTime(seconds) {
      const minutes = Math.floor(seconds / 60)
      const secs = Math.floor(seconds % 60)
      return `${minutes}:${secs < 10 ? '0' : ''}${secs}`
    },
    toggleRepeat(music) {
      this.repeatTracks[music] = !this.repeatTracks[music]
    },
    togglePlay(music) {
      if (this.playingTrack === music) {
        // Dừng bài nhạc
        this.savedTimes[music] = this.audio.currentTime
        this.audio.pause()
        this.waveforms[music]?.pause()
        this.playingTrack = null
      } else {
        // Nếu đang phát bài khác, dừng và reset
        if (this.audio && this.playingTrack !== music) {
          this.audio.pause()
          this.waveforms[this.playingTrack]?.destroy()
          this.progressMap[this.playingTrack] = 0
          this.currentTimes[this.playingTrack] = 0
          this.savedTimes[this.playingTrack] = 0
          this.repeatTracks[this.playingTrack] = false // Reset Repeat khi chuyển bài
        }

        // Nếu là bài cùng bài, khôi phục từ savedTimes
        if (this.audio && this.playingTrack === music) {
          this.audio.currentTime = this.savedTimes[music] || 0
          this.audio.play()
        } else {
          // Tạo mới nếu là bài khác
          this.audio = new Audio(this.getMusicUrl(music))
          if (this.savedTimes[music]) {
            this.audio.currentTime = this.savedTimes[music]
          }
          this.audio.play()
        }

        this.playingTrack = music

        this.audio.addEventListener('loadedmetadata', () => {
          this.durations[music] = this.audio.duration
          // Khôi phục tiến trình nếu có savedTime
          if (this.savedTimes[music] && this.audio.currentTime !== this.savedTimes[music]) {
            this.audio.currentTime = this.savedTimes[music]
          }
        })

        this.audio.addEventListener('timeupdate', () => {
          this.progressMap[music] = (this.audio.currentTime / this.audio.duration) * 100 || 0
          this.currentTimes[music] = this.audio.currentTime || 0
          if (this.waveforms[music]) {
            this.waveforms[music].seekTo(this.audio.currentTime / this.audio.duration || 0)
          }
        })

        this.audio.addEventListener('ended', () => {
          if (this.repeatTracks[music]) {
            // Nếu Repeat bật, phát lại từ đầu
            this.audio.currentTime = 0
            this.audio.play()
            this.progressMap[music] = 0
            this.currentTimes[music] = 0
            if (this.waveforms[music]) {
              this.waveforms[music].seekTo(0)
            }
          } else {
            // Nếu không Repeat, dừng như bình thường
            this.playingTrack = null
            this.progressMap[music] = 0
            this.currentTimes[music] = 0
            this.savedTimes[music] = 0
            this.repeatTracks[music] = false
            this.waveforms[music]?.destroy()
          }
        })

        this.$nextTick(() => {
          const waveformId = `waveform-${music.replace(/[^a-zA-Z0-9]/g, '')}`
          this.waveforms[music] = window.WaveSurfer.create({
            container: `#${waveformId}`,
            waveColor: '#d1d5db',
            progressColor: '#f50',
            height: 80,
            barWidth: 2,
            barGap: 1,
            cursorWidth: 1,
            cursorColor: '#000',
            responsive: true,
            interact: true,
          })
          this.waveforms[music].load(this.getMusicUrl(music))

          this.waveforms[music].on('seek', (progress) => {
            if (this.playingTrack === music) {
              const seekTime = progress * this.audio.duration
              this.audio.currentTime = seekTime
              this.progressMap[music] = progress * 100
              this.currentTimes[music] = seekTime
            }
          })
        })
      }
    },
    seekTrack(event, music) {
      if (this.playingTrack !== music) return
      const rect = event.target.getBoundingClientRect()
      const clickX = event.clientX - rect.left
      const width = rect.width
      const seekPercentage = (clickX / width)
      this.audio.currentTime = seekPercentage * this.audio.duration
      this.progressMap[music] = seekPercentage * 100
      this.currentTimes[music] = this.audio.currentTime
      if (this.waveforms[music]) {
        this.waveforms[music].seekTo(seekPercentage)
      }
    },
  },
  mounted() {
    this.fetchMusicList()
  },
  beforeDestroy() {
    if (this.audio) {
      this.audio.pause()
    }
    Object.values(this.waveforms).forEach(waveform => waveform.destroy())
  }
}
</script>

<style scoped>
.flex-1 {
  flex: 1 1 0%;
  overflow-x: hidden;
}
</style>