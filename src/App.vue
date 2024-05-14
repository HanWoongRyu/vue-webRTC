<script setup>
import { ref, onMounted } from 'vue';
import Peer from 'peerjs';
import VideoDisplay from './components/VideoDisplay.vue';
import CallControls from './components/CallControls.vue';

const peer = ref(null);
const localStream = ref(null);
const customPeerId = ref('');
const audioInputs = ref([]);
const videoInputs = ref([]);
const selectedAudioInput = ref('');
const selectedVideoInput = ref('');

// 미디어 디바이스 가져오기
onMounted(async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  audioInputs.value = devices.filter(device => device.kind === 'audioinput'); 
  videoInputs.value = devices.filter(device => device.kind === 'videoinput');
});

// peer 초기화
const initPeer = async () => {
  if (peer.value) {
    peer.value.destroy(); // Ensure old peer connections are closed
  }

  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { deviceId: selectedVideoInput.value ? { exact: selectedVideoInput.value } : undefined },
      audio: { deviceId: selectedAudioInput.value ? { exact: selectedAudioInput.value } : undefined }
    });
    localStream.value = stream;
    peer.value = new Peer(customPeerId.value);

    peer.value.on('call', call => {
      if (call.metadata.type !=="isol") {
        return
      }
      call.answer(localStream.value);
      call.on('stream', remoteStream => {
        const remoteVideo = document.getElementById('remote-video');
        remoteVideo.srcObject = remoteStream;
      });
    });
  } catch (error) {
    console.error('Failed to initialize peer or media devices', error);
  }
};

function toggleMute() {
  if (localStream.value) {
    localStream.value.getAudioTracks().forEach(track => track.enabled = !track.enabled);
  }
}

function callPeer(id) {
  const options = {metadata: {"type":"isol"}};
  const call = peer.value.call(id, localStream.value, options);
  call.on('stream', remoteStream => {
    const remoteVideo = document.getElementById('remote-video');
    remoteVideo.srcObject = remoteStream;
  });
}

function toggleCamera() {
  if (localStream.value) {
    localStream.value.getVideoTracks().forEach(track => track.enabled = !track.enabled);
  }
}
</script>
<template>
  <div id="app">
    <div v-if="peer && localStream">
      <select v-model="selectedAudioInput">
        <option v-for="device in audioInputs" :key="device.deviceId" :value="device.deviceId">{{ device.label }}</option>
      </select>
      <select v-model="selectedVideoInput">
        <option v-for="device in videoInputs" :key="device.deviceId" :value="device.deviceId">{{ device.label }}</option>
      </select>
      <video-display :peer="peer" :local-stream="localStream" />
      <call-controls :peer="peer" :local-stream="localStream"
                      @toggleMute="toggleMute"
                      @callPeer="callPeer"
                      @toggleCamera="toggleCamera" />
    </div>
    <div v-else>
      <input v-model="customPeerId" placeholder="영문 숫자 - 만 가능" />
      <button @click="initPeer">PeerID 설정</button>
    </div>
  </div>
</template>

