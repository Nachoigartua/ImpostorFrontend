<template>
  <div class="inicio">
    <h2>Impostor Futbol</h2>
    <input v-model="nombre" placeholder="Nombre de jugador" />
    <input v-model="salaId" placeholder="ID de sala" />
    <button @click="unirseSala">Unirse a sala</button>
    <button @click="crearSala">Crear sala</button>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import socket from '../socket';

const nombre = ref('');
const salaId = ref('');
const error = ref('');

const emit = defineEmits(['salaCreada', 'salaUnida']);

socket.on('salaCreada', ({ salaId: id }) => {
  emit('salaCreada', { nombre: nombre.value, salaId: id });
});
socket.on('errorSala', (msg) => {
  error.value = msg;
});
socket.on('actualizarLobby', (jugadores) => {
  emit('salaUnida', { nombre: nombre.value, salaId: salaId.value, jugadores });
});

function crearSala() {
  if (!nombre.value) return error.value = 'Ingresa tu nombre';
  socket.emit('crearSala', { nombre: nombre.value, maxJugadores: 10, impostores: 1 });
}
function unirseSala() {
  if (!nombre.value || !salaId.value) return error.value = 'Completa los campos';
  socket.emit('unirseSala', { nombre: nombre.value, salaId: salaId.value });
}
</script>

<style scoped>
</style>
