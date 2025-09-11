<template>
  <div class="votacion">
    <h2>Votación</h2>
    <ul>
      <li v-for="j in jugadores" :key="j.id">
        <label>
          <input type="radio" :value="j.id" v-model="voto" /> {{ j.nombre }}
        </label>
      </li>
    </ul>
    <label>
      <input type="radio" value="skip" v-model="voto" /> Saltar
    </label>
    <button @click="enviarVoto">Votar</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import socket from '../socket';

const props = defineProps(['salaId', 'jugadores']);
const voto = ref('');

function enviarVoto() {
  if (!voto.value) return;
  socket.emit('votar', { salaId: props.salaId, voto: voto.value });
}
</script>

<style scoped>
</style>
