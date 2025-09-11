<template>
  <div class="lobby">
    <h2>Lobby - Sala {{ salaId }}</h2>
    <ul>
      <li v-for="j in jugadores" :key="j.id">{{ j.nombre }} <span v-if="j.id === host">(Host)</span></li>
    </ul>
    <div v-if="esHost">
      <label>Máx jugadores:
        <input type="number" v-model="maxJugadores" min="4" max="10" />
      </label>
      <label>Impostores:
        <select v-model="impostores">
          <option :value="1">1</option>
          <option :value="2">2</option>
        </select>
      </label>
      <button @click="configurarSala">Guardar configuración</button>
      <button @click="comenzar" :disabled="!configGuardada">Comenzar partida</button>
    </div>
    <div v-else>
      <p>Esperando que el host comience la partida...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import socket from '../socket';

const props = defineProps(['salaId', 'jugadores', 'host', 'miId']);
const emit = defineEmits(['partidaIniciada']);
const maxJugadores = ref(10);
const impostores = ref(1);
const configGuardada = ref(false);
const esHost = computed(() => props.miId === props.host);

function configurarSala() {
  socket.emit('configurarSala', {
    salaId: props.salaId,
    maxJugadores: maxJugadores.value,
    impostores: impostores.value
  });
  configGuardada.value = true;
}
function comenzar() {
  socket.emit('comenzarPartida', props.salaId);
}
socket.on('partidaIniciada', () => {
  emit('partidaIniciada');
});
</script>

<style scoped>
</style>
