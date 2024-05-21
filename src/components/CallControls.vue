<script setup>
import { ref, defineEmits, computed, defineProps } from 'vue';

const emits = defineEmits(['toggleMute', 'callPeer', 'answerCall', 'rejectCall', 'cancelCall', 'toggleCamera', 'endCall']);
const peerId = ref('');

const props = defineProps({
  incomingCall: Boolean,
  isWaiting: Boolean,
  isAudioEnabled: Boolean,
  isVideoEnabled: Boolean
});

const buttonLabel = computed(() => {
  if (props.incomingCall) {
    return 'Answer';
  } else if (props.isWaiting) {
    return 'Wait';
  } else {
    return 'Call';
  }
});

function call() {
  emits('callPeer', peerId.value);
}

function answer() {
  emits('answerCall');
}

function reject() {
  emits('rejectCall');
}

function cancel() {
  emits('cancelCall');
}

function toggleMute() {
  emits('toggleMute');
}

function toggleCamera() {
  emits('toggleCamera');
}

function endCall() {
  emits('endCall');
}
</script>

<template>
  <div>
    <input v-model="peerId" placeholder="Enter peer ID" v-if="!props.incomingCall && !props.isWaiting" />
    <button @click="props.incomingCall ? answer() : call()">{{ buttonLabel }}</button>
    <button @click="toggleMute">{{ props.isAudioEnabled ? 'Mute' : 'Unmute' }}</button>
    <button @click="toggleCamera">{{ props.isVideoEnabled ? 'Disable Camera' : 'Enable Camera' }}</button>
    <button @click="props.incomingCall ? reject() : endCall()">{{ props.incomingCall ? 'Reject' : 'End Call' }}</button>
    <button v-if="props.isWaiting" @click="cancel">Cancel Call</button>
  </div>
</template>
