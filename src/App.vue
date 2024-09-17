<template>
  <div class="container">
    <div class="sidebar">
      <h2 class="sidebar-title">Filtros</h2>
      <div class="filter-section">
        <h3 class="filter-title">Estado</h3>
        <ul class="filter-list">
          <li v-for="estado in ['todos', 'confirmado', 'pendiente']" :key="estado">
            <label>
              <input type="radio" v-model="filtroEstado" :value="estado">
              {{ estado.charAt(0).toUpperCase() + estado.slice(1) }}
            </label>
          </li>
        </ul>
      </div>
      <div class="filter-section">
        <h3 class="filter-title">Género</h3>
        <ul class="filter-list">
          <li v-for="genero in generos" :key="genero">
            <label>
              <input type="radio" v-model="filtroGenero" :value="genero">
              {{ genero.charAt(0).toUpperCase() + genero.slice(1) }}
            </label>
          </li>
        </ul>
      </div>
    </div>
    <div class="main-content">
      <h1 class="title">Organizador de eventos</h1>

      <!-- Formulario para agregar invitados -->
      <form @submit.prevent="agregarInvitado" class="form">
        <input v-model="nuevoInvitado.nombre" placeholder="Nombre" required class="input-text">
        <select v-model="nuevoInvitado.genero" class="input-select">
          <template v-for="(genero, index) in generos">
            <option   v-if="index > 0" :key="index" :value="genero">{{ genero }}</option>
          </template>
        </select>
        <button type="submit" class="btn-primary">Agregar Invitado</button>
      </form>

      <!-- Lista de invitados -->
      <ul class="list">
        <li v-for="invitado in invitadosFiltrados" :key="invitado.id" 
            :class="{ 'confirmado': invitado.confirmado, 'list-item': true }">
          {{ invitado.nombre }} ({{ invitado.genero }})
          <button @click="toggleConfirmacion(invitado)" class="btn-secondary">
            {{ invitado.confirmado ? 'Desconfirmar' : 'Confirmar' }}
          </button>
        </li>
      </ul>

      <!-- Contador de confirmados -->
      <p class="info">Invitados confirmados: {{ contadorConfirmados }}</p>

      <!-- Tareas pendientes -->
      <button @click="mostrarTareas = !mostrarTareas" class="btn-toggle"> 
        {{ mostrarTareas ? 'Ocultar Tareas' : 'Mostrar Tareas' }}
      </button>
      <ul v-show="mostrarTareas" class="list">
        <li v-for="tarea in tareasPendientes" :key="tarea" class="list-item">{{ tarea }}</li>
      </ul>

      <h2 class="subtitle">Vota por la temática del evento</h2>
      <ul class="list">
        <li v-for="(votos, tema) in temasParaEvento" :key="tema" class="list-item">
          {{ tema }}: {{ votos }} 
          <button @click="votarTema(tema)" class="btn-secondary">Votar</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'


const generos = ['todos', 'hombre', 'mujer','otro']
// Estado
const invitados = ref([])
const nuevoInvitado = ref({ nombre: '', genero: generos[1]})
const filtroEstado = ref('todos')
const filtroGenero = ref('todos')
const mostrarTareas = ref(false)
const tareasPendientes = ref([
  'Comprar bebidas',
  'Contratar DJ',
  'Decorar el lugar'
])
const temasParaEvento = ref({
  'Años 80': 0,
  'Superhéroes': 0,
  'Gotico': 0,
  'Futurista': 0
})


// Métodos
const agregarInvitado = () => {
  invitados.value.push({
    id: Date.now(),
    nombre: nuevoInvitado.value.nombre,
    genero: nuevoInvitado.value.genero,
    confirmado: false
  })
  nuevoInvitado.value = { nombre: '', genero: generos[1] }
}

const toggleConfirmacion = (invitado) => {
  invitado.confirmado = !invitado.confirmado
}

const votarTema = (tema) => {
  temasParaEvento.value[tema]++
}

// Computed properties
const invitadosFiltrados = computed(() => {
  return invitados.value.filter(invitado => {
    const cumpleEstado = filtroEstado.value === 'todos' || 
      (filtroEstado.value === 'confirmado' && invitado.confirmado) ||
      (filtroEstado.value === 'pendiente' && !invitado.confirmado)
    const cumpleGenero = filtroGenero.value === 'todos' || invitado.genero === filtroGenero.value
    return cumpleEstado && cumpleGenero
  })
})

const contadorConfirmados = computed(() => {
  return invitados.value.filter(invitado => invitado.confirmado).length
})
</script>

<style scoped>
.container {
  display: flex;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  background-color: #f6f8fa;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
}

.sidebar {
  width: 250px;
  padding-right: 20px;
  border-right: 1px solid #e1e4e8;
}

.main-content {
  flex-grow: 1;
  padding-left: 20px;
}

.sidebar-title {
  font-size: 18px;
  font-weight: 600;
  color: #24292f;
  margin-bottom: 15px;
}

.filter-section {
  margin-bottom: 20px;
}

.filter-title {
  font-size: 16px;
  font-weight: 500;
  color: #586069;
  margin-bottom: 10px;
}

.filter-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.filter-list li {
  margin-bottom: 8px;
}

.filter-list label {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.filter-list input[type="radio"] {
  margin-right: 8px;
}

.title, .subtitle {
  font-size: 24px;
  font-weight: 600;
  color: #24292f;
  margin-bottom: 20px;
}

.form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.input-text, .input-select {
  padding: 8px 12px;
  border: 1px solid #d1d5da;
  border-radius: 6px;
  background-color: #fff;
  font-size: 14px;
}

.input-text:focus, .input-select:focus {
  outline: none;
  border-color: #0366d6;
}

.list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.list-item {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border: 1px solid #e1e4e8;
  background-color: #fff;
  border-radius: 6px;
  margin-bottom: 10px;
}

.confirmado {
  color: green;
  font-weight: bold;
}

.btn-primary, .btn-secondary, .btn-toggle {
  padding: 8px 12px;
  border: 1px solid #d1d5da;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.2s;
}

.btn-primary {
  background-color: #28a745;
  color: white;
}

.btn-primary:hover {
  background-color: #218838;
}

.btn-secondary {
  background-color: #f6f8fa;
  color: #0366d6;
}

.btn-secondary:hover {
  background-color: #e1e4e8;
}

.btn-toggle {
  margin-top: 10px;
  background-color: #f6f8fa;
  color: #24292f;
}

.info {
  font-size: 16px;
  font-weight: 500;
  color: #586069;
  margin-top: 20px;
}
</style>