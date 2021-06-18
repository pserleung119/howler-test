<template>
  <div>
    <div v-if="audioLoaded">
      <button @click="unload()">Unload</button>
    </div>
    <div v-else>
      <button @click="loadPerFile()">Load Per File</button>
      <button @click="loadAtOnce()">Load At Once</button>
    </div>

    <div v-if="audioLoaded">
      <div>
        <button v-if="isPlaying" @click="stop()">Stop</button>
        <button v-else @click="play()">Play</button>
      </div>
      <div>
        <button @click="tuneVolume(0.1)">Volume+</button>
        <button @click="tuneVolume(-0.1)">Volume-</button>
      </div>

    </div>

  </div>
</template>

<script>
import data from './json/audio_files.json'
import {Howl, Howler} from 'howler';

export default {
  name: 'App',
  data() {
    return {
      audio: [],
      audioLoaded: false,
      isPlaying: false
    }
  },
  methods: {
    async loadPerFile() {
      try {
        const startMemory = window.performance.memory.usedJSHeapSize
        const startTime = Date.now()
        for (let i = 0; i < data.length; i++) {
          const audio = await this.readFile(data[i])
          this.audio.push(audio)
        }
        this.audioLoaded = true
        console.log(`Finished in ${Date.now() - startTime}ms, ${window.performance.memory.usedJSHeapSize - startMemory} used`)
      } catch(e) {
        console.log(e)
      }
    },

    async loadAtOnce() {
      try {
        const startMemory = window.performance.memory.usedJSHeapSize
        const startTime = Date.now()
        this.audio = await Promise.all(data.map((fileName) => {return this.readFile(fileName)}))
        console.log(`Finished in ${Date.now() - startTime}ms, ${window.performance.memory.usedJSHeapSize - startMemory} used`)
        this.audioLoaded = true
      } catch(e) {
        console.log(e)
      }
    },

    async readFile(fileName) {
      return new Promise((res, rej) => {
        const node = new Howl({
          src: `${process.env.VUE_APP_AUDIO_FILE_DOMAIN}${fileName}`,
          loop: true,
          onload: () => {
            console.log('loaded')
            res(node)
          },
          onloaderror: (e) => {
            rej(e)
          }
        })
      })
    },

    play() {
      this.audio.forEach((node) => {
        node.play()
      })
      this.isPlaying = true
    },

    stop() {
      this.audio.forEach((node) => {
        node.stop()
      })
      // Or use Howler.stop() to stop all
      // Howler.stop()
      this.isPlaying = false
    },

    tuneVolume(value) {
      this.audio.forEach((node) => {
        node.volume(node.volume() + value)
      })
      // Or use Howler.stop() to tune all
      // Howler.volume(Howler.volume() + value)
    },

    unload() {
      Howler.unload()
      this.audio = []
      this.audioLoaded = false
    }

  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

button {
  margin: 1rem;
}
</style>
