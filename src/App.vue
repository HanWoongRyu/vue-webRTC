<script setup>
import { ref, onMounted } from 'vue';
import Peer from 'peerjs';
import VideoDisplay from './components/VideoDisplay.vue';
import CallControls from './components/CallControls.vue';

const peer = ref(null);
const myID = ref(null);
const localStream = ref(null);
const customPeerId = ref('');
const audioInputs = ref([]);
const videoInputs = ref([]);
const selectedAudioInput = ref('');
const selectedVideoInput = ref('');
const currentCall = ref(null); // 현재 통화 객체
const incomingCall = ref(false); // 수신된 통화 여부
const isWaiting = ref(false); // 대기 상태 여부
const isAudioEnabled = ref(true); // 오디오 활성화 상태
const isVideoEnabled = ref(true); // 비디오 활성화 상태

// 미디어 장치 목록 가져오기
onMounted(async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  audioInputs.value = devices.filter(device => device.kind === 'audioinput'); 
  videoInputs.value = devices.filter(device => device.kind === 'videoinput');
});

// peer 초기화
const initPeer = async () => {
  if (peer.value) {
    peer.value.destroy(); // 기존 peer 객체 파괴
    peer.value = null;  // peer를 null로 설정
  }

  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { deviceId: selectedVideoInput.value ? { exact: selectedVideoInput.value } : undefined },
      audio: { deviceId: selectedAudioInput.value ? { exact: selectedAudioInput.value } : undefined }
    });
    localStream.value = stream;
    myID.value = customPeerId.value;
    peer.value = new Peer(customPeerId.value);

    peer.value.on('open', id => { // peer가 연결되었을 때 이벤트
      console.log(`Peer 연결 성공, ID: ${id}`);
    });

    peer.value.on('error', error => { // peer 연결 오류 이벤트
      console.error("peer 연결 오류", error);
      peer.value = null; 
    });

    peer.value.on('call', call => {
      if (call.metadata.type !== "isollab") {
        return;
      }

      if (currentCall.value) {
        console.log('이미 통화 중입니다. 새로운 통화 요청을 거절합니다.');
        call.close();
        const conn = peer.value.connect(call.peer);
        conn.on('open', () => {
          conn.send('busy');
          conn.close();
        });
        return;
      }
      console.log("수신된 통화", call.metadata.id);
      currentCall.value = call;
      incomingCall.value = true; // 수신된 통화 상태 변경

      call.on('close', () => {
        console.log('통화가 종료되었습니다.');
        currentCall.value = null; // 현재 통화 해제
        isWaiting.value = false; // 대기 상태 해제
    });
    });

    peer.value.on('connection', conn => {
      conn.on('data', data => {
        if (data === 'busy') {
          console.log('상대방이 통화 중입니다.');
        }
        else if(data === 'reject') {
          console.log('상대방이 전화를 거절했습니다.')
          endCall()
        }
        else if(data === 'cancel') {
          console.log("전화를 취소했습니다.")
          endCall()
        }
      });
    });

  } catch (error) {
    console.error('미디어 스트림 가져오기 오류', error);
  }
};

function toggleMute() {
  isAudioEnabled.value = !isAudioEnabled.value;
  if (localStream.value) {
    localStream.value.getAudioTracks().forEach(track => track.enabled = isAudioEnabled.value);
  }
}

function toggleCamera() {
  isVideoEnabled.value = !isVideoEnabled.value;
  if (localStream.value) {
    localStream.value.getVideoTracks().forEach(track => track.enabled = isVideoEnabled.value);
  }
}

function callPeer(id) {
  if (currentCall.value) {
    console.log('이미 통화 중입니다.');
    return;
  }
  const options = { metadata: { "type": "isollab", "id": myID.value } };
  const call = peer.value.call(id, localStream.value, options);
  currentCall.value = call;
  isWaiting.value = true; // 대기 상태로 변경

  call.on('stream', remoteStream => {
    const remoteVideo = document.getElementById('remote-video');
    remoteVideo.srcObject = remoteStream;
    isWaiting.value = false; // 대기 상태 해제
  });

  call.on('close', () => {
    console.log('통화가 종료되었습니다.');
    currentCall.value = null; // 현재 통화 해제
    isWaiting.value = false; // 대기 상태 해제
  });
}

function answerCall() {
  if (currentCall.value) {
    currentCall.value.answer(localStream.value);
    incomingCall.value = false; // 통화 응답 상태로 변경

    currentCall.value.on('stream', remoteStream => {
      const remoteVideo = document.getElementById('remote-video');
      remoteVideo.srcObject = remoteStream;
    });

    currentCall.value.on('close', () => {
      console.log('통화가 종료되었습니다.');
      currentCall.value = null; // 현재 통화 해제
    });
  }
}

function rejectCall() {
  if (currentCall.value){
    console.log("통화 거절 누름!")
    const conn = peer.value.connect(currentCall.value.peer);
        conn.on('open', () => {
          conn.send('reject');
          conn.close();
        });
    endCall()
  }
}

function cancelCall() {
  if (currentCall.value){
    console.log("통화 취소 누름!")
    const conn = peer.value.connect(currentCall.value.peer);
        conn.on('open', () => {
          conn.send('cancel');
          conn.close();
        });
    endCall()
  }
}

function endCall() {
  if (currentCall.value) {
    currentCall.value.close();
    console.log('통화가 종료되었습니다.');
    currentCall.value = null; // 현재 통화 해제
    incomingCall.value = false; // 통화 종료 상태로 변경
    isWaiting.value = false; // 대기 상태 해제
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
                      :incoming-call="incomingCall"
                      :is-waiting="isWaiting"
                      :is-audio-enabled="isAudioEnabled"
                      :is-video-enabled="isVideoEnabled"
                      @toggleMute="toggleMute"
                      @callPeer="callPeer"
                      @answerCall="answerCall"
                      @toggleCamera="toggleCamera"
                      @endCall="endCall" 
                      @rejectCall="rejectCall"
                      @cancelCall="cancelCall"
                      />
    </div>
    <div v-else>
      <input v-model="customPeerId" placeholder="Enter custom Peer ID" />
      <button @click="initPeer">Initialize Peer</button>
    </div>
  </div>
</template>
