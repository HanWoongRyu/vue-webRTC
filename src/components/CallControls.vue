<script setup>
import { ref, defineEmits } from 'vue';

const emits = defineEmits(['toggleMute', 'callPeer', 'toggleCamera']);
const peerId = ref('');

function call() {
  emits('callPeer', peerId.value);
}

function toggleMute() {
  emits('toggleMute');
}

function toggleCamera() {
  emits('toggleCamera');
}
</script>

<template>
  <div>
    <input v-model="peerId" placeholder="Enter peer ID" />
    <button @click="call">Call</button>
    <button @click="toggleMute">Toggle Mute</button>
    <button @click="toggleCamera">Toggle Camera</button>
  </div>
</template>
