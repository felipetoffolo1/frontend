<template>
  <audio
    ref="audioRef"
    controls
    :src="audio"
    loop
    class="audio-control"
  ></audio>
</template>

<script setup lang="ts">
import audio from "@/assets/5_mins_of_silence.mp3";
import { useMediaBrowserMetaData } from "@/helpers/useMediaBrowserMetaData";
import api from "@/plugins/api";
import { PlayerState } from "@/plugins/api/interfaces";
import { store } from "@/plugins/store";
import { onMounted, ref, watch } from "vue";

const audioRef = ref<HTMLAudioElement>();
const interactedRef = ref(false);

function apiCommandWithCurrentPlayer<T extends (id: string) => any>(
  command: T,
) {
  const activePlayer = store.activePlayer;
  if (!activePlayer) return;
  return command(activePlayer?.player_id);
}

function updateMediaState(state?: PlayerState) {
  if (!state || !audioRef.value || !interactedRef.value) return;
  let mediaState: MediaSessionPlaybackState;

  switch (state) {
    case PlayerState.PLAYING:
      audioRef.value.play();
      mediaState = state;
      break;
    case PlayerState.PAUSED:
      audioRef.value.pause();
      mediaState = state;
      break;
    default:
      audioRef.value.pause();
      mediaState = "none";
  }

  navigator.mediaSession.playbackState = mediaState;
}

watch(
  () => store.activePlayer?.state,
  (stateUpdate) => updateMediaState(stateUpdate),
);

useMediaBrowserMetaData();

// MediaSession setup
onMounted(() => {
  window.addEventListener("click", () => {
    //There is a safety rule in which you need to interact with the page for the audio to play
    interactedRef.value = true;
    updateMediaState(store.activePlayer?.state);
  });

  navigator.mediaSession.setActionHandler("play", () => {
    apiCommandWithCurrentPlayer(api.playerCommandPlay.bind(api));
  });
  navigator.mediaSession.setActionHandler("pause", () => {
    apiCommandWithCurrentPlayer(api.playerCommandPause.bind(api));
  });
  navigator.mediaSession.setActionHandler("nexttrack", () => {
    apiCommandWithCurrentPlayer(api.playerCommandNext.bind(api));
  });
  navigator.mediaSession.setActionHandler("previoustrack", () => {
    apiCommandWithCurrentPlayer(api.playerCommandPrevious.bind(api));
  });
});
</script>
<style lang="css">
.audio-control {
  width: 0;
  height: 0;
}
</style>
