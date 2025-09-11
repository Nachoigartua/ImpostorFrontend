<template>
  <div class="juego">
    <h2>Partida en curso</h2>
    <div v-if="rol === 'impostor'">
      <h3>¡Sos impostor!</h3>
    </div>
    <div v-else>
      <h3>Tu jugador de fútbol: {{ rol }}</h3>
    </div>
  <!-- Botón de iniciar votación eliminado -->
    <div v-if="expulsado">Expulsado: {{ expulsado }}</div>
    <div v-if="nadieExpulsado">Nadie fue expulsado</div>
    <div v-if="fin">{{ fin }}</div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import socket from '../socket';

const props = defineProps(['salaId']);
const rol = ref(null);
const expulsado = ref('');
const nadieExpulsado = ref(false);
const fin = ref('');

// Función de iniciar votación eliminada
socket.on('rolAsignado', ({ rol: r }) => {
  rol.value = r;
});
socket.on('partidaIniciada', () => {
  rol.value = null;
});
socket.on('jugadorExpulsado', (id) => {
  expulsado.value = id;
  nadieExpulsado.value = false;
});
socket.on('nadieExpulsado', () => {
  expulsado.value = '';
  nadieExpulsado.value = true;
});
socket.on('finPartida', (msg) => {
  fin.value = msg;
});
</script>

<style scoped>
</style>
