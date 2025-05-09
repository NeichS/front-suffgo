<template>
  <div class="flex flex-col items-center justify-center text-center h-screen w-screen">
    <div v-if="socketStore.timer !== null">
      <div>
        <h2>Tiempo restante</h2>
      </div>
      <div>
        <h1 v-if="settingsRoom && settingsRoom.proposal_timer" class="font-bold">
          {{ socketStore.countdown }}
        </h1>
      </div>
    </div>
    <div v-else class="flex flex-col items-center justify-center text-center">
      <h2>Esperando resultados</h2>
      <div class="loader my-2" />
    </div>
    <VoteState />
    
    <TabChat />
  </div>
</template>

<script setup>
import { useWebSocketStore } from '../../stores/socketStore';
import { onMounted, provide, onUnmounted, watch, ref } from 'vue';
import TabChat from '../components/TabChat.vue';
import VoteState from '../components/VoteState.vue';
import { storeToRefs } from 'pinia'
import { useRouter } from 'vue-router'
import { Icon } from "@iconify/vue";

const router = useRouter();
const socketStore = useWebSocketStore();
const {
  userList
} = storeToRefs(socketStore)

const room = ref('');
const settingsRoom = ref(null);
const user = ref('');

onMounted(() => {
  const loggedUser = localStorage.getItem('user');
    if (loggedUser) {
        user.value = JSON.parse(loggedUser);
    }
  const storedRoom = localStorage.getItem('currentRoom');
  if (storedRoom) {
    try {
      room.value = JSON.parse(storedRoom);
    } catch (e) {
      error.value = 'Error al leer los datos de la sala.';
      console.error(e);
    }
  } else {
    error.value = 'No se encontraron datos de la sala en el almacenamiento local.';
  }

  const storedSettingsRoom = localStorage.getItem('settingsRoom');
  if (storedSettingsRoom) {
    try {
      settingsRoom.value = JSON.parse(storedSettingsRoom);
    } catch (e) {
      error.value = 'Error al leer los datos de la configuracion de la sala.';
      console.error(e);
    }
  } else {
    error.value = 'No se encontraron datos de la configuracion de la sala en el almacenamiento local.';
  }
});

onUnmounted(() => {
  socketStore.stopCountdown();
});


//El admin voto y los resultados estan listos
watch(
  () => socketStore.resultsAvailable,
  (available) => {
    if (available && room.value?.privileges) {
      router.push('/confirmResults');
    }
  },
  { immediate: true }
);

//Votante normal y resultados listos
watch(
  () => socketStore.resultsReady,
  (available) => {
    if (available && !room.value?.privileges) {
      router.push('/results');
    }
  },
  { immediate: true }
);

provide('user', user)
</script>

<style>
.loader {
  width: 60px;
  aspect-ratio: 4;
  --_g: no-repeat radial-gradient(circle closest-side, #6B48FF 90%, #0000);
  background:
    var(--_g) 0% 50%,
    var(--_g) 50% 50%,
    var(--_g) 100% 50%;
  background-size: calc(100%/3) 100%;
  animation: l7 1s infinite linear;
}

@keyframes l7 {
  33% {
    background-size: calc(100%/3) 0%, calc(100%/3) 100%, calc(100%/3) 100%
  }

  50% {
    background-size: calc(100%/3) 100%, calc(100%/3) 0%, calc(100%/3) 100%
  }

  66% {
    background-size: calc(100%/3) 100%, calc(100%/3) 100%, calc(100%/3) 0%
  }
}
</style>