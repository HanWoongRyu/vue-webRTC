<script setup>
import { ref, onMounted } from 'vue';
import Peer from 'peerjs';
import VideoDisplay from './components/VideoDisplay.vue';
import CallControls from './components/CallControls.vue';

const peer = ref(null);
const myID = ref(null)
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
    peer.value.destroy(); // 시작할때 피어 파괴
    peer.value = null;  // peer를 null로 설정

  }

  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { deviceId: selectedVideoInput.value ? { exact: selectedVideoInput.value } : undefined },
      audio: { deviceId: selectedAudioInput.value ? { exact: selectedAudioInput.value } : undefined }
    });
    localStream.value = stream;
    myID.value= customPeerId.value
    peer.value = new Peer(customPeerId.value);

    peer.value.on('open', id => { //peer를 성공적으로 생성하면 받을 수 있음
      console.log(`Peer 연결 성공, ID: ${id}`);
    });

    peer.value.on('error', error => { //peer를 성공적으로 생성하면 받을 수 있음
      console.error("peer 설정 error", error);
      peer.value = null; 
    });



    peer.value.on('call', call => {
      if (call.metadata.type !=="WebRTC 테슷흐") {
        return
      }
      console.log("전화가 왔습니다!", call.metadata.id)

      call.answer(localStream.value);
      call.on('stream', remoteStream => {
        const remoteVideo = document.getElementById('remote-video');
        remoteVideo.srcObject = remoteStream;
      });
    });
  } catch (error) {
    console.error('피어 혹은 미디어 찾기 실패', error);
  }
};

function toggleMute() {
  if (localStream.value) {
    localStream.value.getAudioTracks().forEach(track => track.enabled = !track.enabled);
  }
}

function callPeer(id) {
  const options = {metadata: {"type":"WebRTC 테슷흐", "id" : myID.value}};
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

