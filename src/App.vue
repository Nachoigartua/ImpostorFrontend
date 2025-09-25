<template>
  <div>
    <Inicio v-if="fase === 'inicio'" @salaCreada="onSalaCreada" @salaUnida="onSalaUnida" />
    <Lobby v-if="fase === 'lobby'" :salaId="salaId" :jugadores="jugadores" :host="host" :miId="miId" @partidaIniciada="onPartidaIniciada" />
    <Juego v-if="fase === 'juego'" :salaId="salaId" :jugadores="jugadores" :host="host" :miId="miId" />
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
socket.on('partidaIniciada', ({ jugadores: js }) => {
  if (js) {
    jugadores.value = js;
  }
  fase.value = 'juego';
});
socket.on('faseVotacion', (js) => {
  jugadoresVivos.value = js;
  fase.value = 'votacion';
});
socket.on('jugadoresOrdenados', ({ jugadores: js }) => {
  if (js) {
    jugadores.value = js;
  }
});

socket.on('finPartida', () => {
  fase.value = 'inicio';
  salaId.value = '';
  jugadores.value = [];
  host.value = '';
});
</script>
<style>
body {
  font-family: 'Montserrat', 'Segoe UI', Arial, sans-serif;
  background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
  min-height: 100vh;
  margin: 0;
  color: #fff;
}

.inicio, .lobby, .juego, .votacion {
  background: rgba(255,255,255,0.08);
  border-radius: 24px;
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  border: 1px solid rgba(255,255,255,0.18);
  padding: 2em 1.5em 1.5em 1.5em;
  margin: 2em auto;
  max-width: 420px;
  transition: box-shadow 0.3s;
}

h2, h3 {
  font-weight: 700;
  letter-spacing: 1px;
  text-shadow: 0 2px 8px #0006;
  margin-top: 0.5em;
  margin-bottom: 1em;
}

input, select {
  display: block;
  width: 100%;
  padding: 0.8em 1em;
  margin: 0.7em 0 0.7em 0;
  box-sizing: border-box;
  border-radius: 12px;
  border: none;
  background: rgba(255,255,255,0.15);
  color: #fff;
  font-size: 1.1em;
  box-shadow: 0 2px 8px #0002;
  outline: none;
  transition: background 0.2s, box-shadow 0.2s;
}
input:focus, select:focus {
  background: rgba(255,255,255,0.25);
  box-shadow: 0 4px 16px #0003;
}
input::placeholder {
  color: #e0e0e0;
  opacity: 1;
  font-weight: 500;
  letter-spacing: 0.5px;
}

button {
  display: inline-block;
  padding: 0.8em 2em;
  margin: 1em 0.5em 0 0;
  border-radius: 16px;
  border: none;
  background: linear-gradient(90deg, #ff512f 0%, #dd2476 100%);
  color: #fff;
  font-size: 1.1em;
  font-weight: 600;
  box-shadow: 0 4px 16px #dd247666;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s, background 0.3s;
  position: relative;
  overflow: hidden;
}
button:active {
  transform: scale(0.97);
}
button:disabled {
  background: #999;
  cursor: not-allowed;
  box-shadow: none;
}
button::after {
  content: '';
  position: absolute;
  left: 50%;
  top: 50%;
  width: 0;
  height: 0;
  background: rgba(255,255,255,0.3);
  border-radius: 100%;
  transform: translate(-50%, -50%);
  transition: width 0.4s cubic-bezier(.4,0,.2,1), height 0.4s cubic-bezier(.4,0,.2,1);
  z-index: 0;
}
button:active::after {
  width: 200%;
  height: 200%;
}

.error {
  color: #ff512f;
  background: rgba(255,81,47,0.08);
  border-radius: 12px;
  padding: 0.7em 1em;
  margin-top: 1em;
  font-weight: 600;
  box-shadow: 0 2px 8px #ff512f33;
}

ul {
  list-style: none;
  padding: 0;
}
li {
  margin: 0.7em 0;
  font-size: 1.1em;
  background: rgba(255,255,255,0.07);
  border-radius: 8px;
  padding: 0.5em 1em;
  box-shadow: 0 1px 4px #0002;
  display: flex;
  align-items: center;
}
label {
  font-weight: 500;
  cursor: pointer;
}

@media (max-width: 600px) {
  .inicio, .lobby, .juego, .votacion {
    padding: 1.2em 0.5em;
    max-width: 98vw;
  }
}

</style>
