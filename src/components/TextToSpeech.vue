<script lang="ts" setup>
import {Ref, ref} from "vue";

const props = defineProps<{
  message: string;
}>();

const isSpeaking: Ref<boolean> = ref(false);

const tts = () => {
  const {message} = props;
  const msg = new SpeechSynthesisUtterance(message);
  msg.rate = 0.8;
  msg.pitch = 0.2;
  msg.onstart = () => isSpeaking.value = true;
  msg.onend = () => isSpeaking.value = false;
  window.speechSynthesis.speak(msg);
};

</script>
<template>
  <v-btn :disabled="isSpeaking" append-icon="mdi-microphone" color="blue" @click="tts">Text-To-Speech</v-btn>
</template>
