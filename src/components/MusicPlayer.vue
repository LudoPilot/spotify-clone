<script setup>
import { ref, watch, onMounted } from 'vue'
import MusicPlayerVolume from '../components/MusicPlayerVolume.vue'
import Heart from 'vue-material-design-icons/Heart.vue';
import PictureInPictureBottomRight from 'vue-material-design-icons/PictureInPictureBottomRight.vue';
import Play from 'vue-material-design-icons/Play.vue';
import Pause from 'vue-material-design-icons/Pause.vue';
import SkipBackward from 'vue-material-design-icons/SkipBackward.vue';
import SkipForward from 'vue-material-design-icons/SkipForward.vue';

import { useSongStore } from '../stores/song'
import { storeToRefs } from 'pinia';
const useSong = useSongStore()
const { isPlaying, audio, currentTrack, currentArtist } = storeToRefs(useSong)

let isHover = ref(false)
let isTrackTimeCurrent = ref(null)
let isTrackTimeTotal = ref(null)
let seeker = ref(null)
let seekerContainer = ref(null)
let range = ref(0)

onMounted(() => {

  if (audio.value) {
    setTimeout(() => {
      timeupdate()
      loadmetadata()
    }, 300)
  }

  if (currentTrack.value) {
    seeker.value.addEventListener("change", function () {
      const time = audio.value.duration * (seeker.value.value / 100);
      audio.value.currentTime = time;
    });

    seeker.value.addEventListener("mousedown", function () {
      audio.value.pause();
      isPlaying.value = false
    });

    seeker.value.addEventListener("mouseup", function () {
      audio.value.play();
      isPlaying.value = true
    });

    seekerContainer.value.addEventListener("click", function (e) {
      const clickPosition = (e.pageX - seekerContainer.value.offsetLeft) / seekerContainer.value.offsetWidth;
      const time = audio.value.duration * clickPosition;
      audio.value.currentTime = time;
      seeker.value.value = (100 / audio.value.duration) * audio.value.currentTime;
    });
  }
})

const timeupdate = () => {
  audio.value.addEventListener("timeupdate", function () {
    var minutes = Math.floor(audio.value.currentTime / 60);
    var seconds = Math.floor(audio.value.currentTime - minutes * 60);
    isTrackTimeCurrent.value = minutes+':'+seconds.toString().padStart(2, '0')
    const value = (100 / audio.value.duration) * audio.value.currentTime;
    range.value = value
    seeker.value.value = value;
  });
}

const loadmetadata = () => {
  audio.value.addEventListener('loadedmetadata', function() {
    const duration = audio.value.duration;
    const minutes = Math.floor(duration / 60);
    const seconds = Math.floor(duration % 60);
    isTrackTimeTotal.value = minutes+':'+seconds.toString().padStart(2, '0')
  });
}

watch(() => audio.value, () => {
  timeupdate()
  loadmetadata()
})

watch(() => isTrackTimeCurrent.value, (time) => {
  if (time && time === isTrackTimeTotal.value) {
    useSong.nextSong(currentTrack.value)
  }
})
</script>

<template>

</template>

<style scoped>

</style>