<script setup>
import { defineProps } from 'vue';

const props = defineProps({
  localStream: Object,
  remoteStream: Object
});
</script>

<template>
  <div class="video-container">
    <video id="local-video" autoplay :srcObject="props.localStream" :muted="true"></video>
    <video id="remote-video" autoplay :srcObject="props.remoteStream"></video>
  </div>
</template>

<style scoped>
.video-container video {
  width: 300px;
  height: 300px;
  border: 1px solid black;
}
</style>
