<template>
  <div>
    <Inicio v-if="fase === 'inicio'" @salaCreada="onSalaCreada" @salaUnida="onSalaUnida" />
    <Lobby v-if="fase === 'lobby'" :salaId="salaId" :jugadores="jugadores" :host="host" :miId="miId" @partidaIniciada="onPartidaIniciada" />
    <Juego v-if="fase === 'juego'" :salaId="salaId" />
    <Votacion v-if="fase === 'votacion'" :salaId="salaId" :jugadores="jugadoresVivos" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Inicio from './components/Inicio.vue';
import Lobby from './components/Lobby.vue';
import Juego from './components/Juego.vue';
import Votacion from './components/Votacion.vue';
import socket from './socket';

const fase = ref('inicio');
const salaId = ref('');
const jugadores = ref([]);
const host = ref('');
const miId = ref('');
const jugadoresVivos = ref([]);

socket.on('connect', () => {
  miId.value = socket.id;
});
function onSalaCreada({ nombre, salaId: id }) {
  salaId.value = id;
  fase.value = 'lobby';
}
function onSalaUnida({ nombre, salaId: id, jugadores: js }) {
  salaId.value = id;
  jugadores.value = js;
  host.value = js[0]?.id;
  fase.value = 'lobby';
}
socket.on('actualizarLobby', ({ jugadores: js, host: h }) => {
  jugadores.value = js;
  host.value = h;
});
socket.on('partidaIniciada', () => {
  fase.value = 'juego';
});
socket.on('faseVotacion', (js) => {
  jugadoresVivos.value = js;
  fase.value = 'votacion';
});
socket.on('finPartida', () => {
  fase.value = 'inicio';
  salaId.value = '';
  jugadores.value = [];
  host.value = '';
});
</script>
