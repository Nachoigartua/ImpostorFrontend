<template>
  <div class="juego">
    <h2>Partida en curso</h2>
    <div v-if="rol === 'IMPOSTOR'">
      <h3>¡Sos impostor!</h3>
    </div>
    <div v-else>
      <h3>Tu jugador de fútbol: {{ rol }}</h3>
    </div>
    
    <div class="jugadores-lista">
      <h4>Orden:</h4>
      <div class="jugadores-header">
        <div class="jugadores-title">
          <span v-if="jugadores.length > 0">{{ jugadores.length }} Este es el orden actual:</span>
          <button v-if="esHost" @click="sortearJugadores" class="sort-button">
            Sortear orden
          </button>
        </div>
      </div>
      <ul class="jugadores-ul">
        <li v-for="jugador in jugadores" :key="jugador.id" class="jugador-item">
          {{ jugador.nombre }}
        </li>
      </ul>
    </div>

    <div v-if="expulsado" class="estado">Expulsado: {{ expulsado }}</div>
    <div v-if="nadieExpulsado" class="estado">Nadie fue expulsado</div>
    <div v-if="fin" class="estado">{{ fin }}</div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import socket from '../socket';

const props = defineProps(['salaId']);
const rol = ref(null);
const expulsado = ref('');
const nadieExpulsado = ref(false);
const fin = ref('');
const jugadores = ref([]);
const miId = ref(socket.id);
const host = ref('');

const esHost = computed(() => miId.value === host.value);

function sortearJugadores() {
  socket.emit('sortearJugadores', props.salaId);
}

// Escuchamos los eventos del socket
socket.on('rolAsignado', ({ rol: r }) => {
  rol.value = r;
});

socket.on('partidaIniciada', ({ jugadores: js }) => {
  if (js) {
    jugadores.value = js;
  }
});

socket.on('actualizarLobby', ({ jugadores: js, host: h }) => {
  if (js) jugadores.value = js;
  if (h) host.value = h;
});

socket.on('jugadoresOrdenados', ({ jugadores: js }) => {
  if (js) {
    jugadores.value = js;
  }
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

// Solicitamos el estado inicial
socket.emit('obtenerEstadoSala', props.salaId);
</script>

<style scoped>
.juego {
  padding: 1em;
}

.jugadores-lista {
  margin-top: 2em;
  background: rgba(255,255,255,0.08);
  border-radius: 16px;
  padding: 1.5em;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.jugadores-lista h4 {
  margin: 0 0 1em 0;
  color: #fff;
  font-size: 1.2em;
  font-weight: 600;
}

.jugadores-header {
  margin-bottom: 1em;
}

.jugadores-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1em;
  font-size: 1em;
  color: rgba(255, 255, 255, 0.9);
}

.sort-button {
  padding: 0.5em 1.2em;
  margin: 0;
  font-size: 0.9em;
  background: linear-gradient(90deg, #4b6cb7 0%, #182848 100%);
  border: none;
  border-radius: 8px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.sort-button:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.jugadores-ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.jugador-item {
  background: rgba(255,255,255,0.05);
  padding: 1em 1.2em;
  margin: 0.5em 0;
  border-radius: 8px;
  transition: all 0.2s ease;
  font-size: 1em;
  color: rgba(255, 255, 255, 0.9);
  display: flex;
  align-items: center;
}

.jugador-item:hover {
  background: rgba(255,255,255,0.1);
  transform: translateX(2px);
}

.estado {
  margin-top: 1em;
  padding: 1em;
  background: rgba(255,255,255,0.05);
  border-radius: 8px;
  text-align: center;
}
</style>