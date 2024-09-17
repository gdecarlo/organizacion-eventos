<template>
    <div class="min-h-screen bg-gray-100 py-12 px-4 sm:px-6 lg:px-8">
      <div class="max-w-7xl mx-auto">
        <h1 class="text-3xl font-bold text-gray-900 mb-8">Organizador de eventos</h1>
        
        <div class="flex flex-col md:flex-row gap-8">
          <!-- Columna de filtros -->
          <div class="w-full md:w-1/4">
            <div class="bg-white shadow-md rounded-lg overflow-hidden">
              <div class="p-4 bg-gray-50 border-b">
                <h2 class="text-lg font-semibold text-gray-700">Filtros</h2>
              </div>
              <div class="p-4">
                <h3 class="font-medium text-gray-700 mb-2">Estado</h3>
                <div class="space-y-2">
                  <label class="flex items-center">
                    <input type="radio" v-model="filter.status" value="all" class="form-radio text-indigo-600">
                    <span class="ml-2">Todos</span>
                  </label>
                  <label class="flex items-center">
                    <input type="radio" v-model="filter.status" value="confirmed" class="form-radio text-indigo-600">
                    <span class="ml-2">Confirmados</span>
                  </label>
                  <label class="flex items-center">
                    <input type="radio" v-model="filter.status" value="pending" class="form-radio text-indigo-600">
                    <span class="ml-2">Pendientes</span>
                  </label>
                </div>
                
                <h3 class="font-medium text-gray-700 mt-6 mb-2">Género</h3>
                <div class="space-y-2">
                  <label class="flex items-center">
                    <input type="checkbox" v-model="filter.gender" value="all" class="form-checkbox text-indigo-600">
                    <span class="ml-2">Todos</span>
                  </label>
                  <label class="flex items-center">
                    <input type="checkbox" v-model="filter.gender" value="male" class="form-checkbox text-indigo-600">
                    <span class="ml-2">Hombre</span>
                  </label>
                  <label class="flex items-center">
                    <input type="checkbox" v-model="filter.gender" value="female" class="form-checkbox text-indigo-600">
                    <span class="ml-2">Mujer</span>
                  </label>
                  <label class="flex items-center">
                    <input type="checkbox" v-model="filter.gender" value="other" class="form-checkbox text-indigo-600">
                    <span class="ml-2">Otro</span>
                  </label>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Contenido principal -->
          <div class="w-full md:w-3/4">
            <div class="bg-white shadow-md rounded-lg overflow-hidden">
              <!-- Formulario para añadir invitado -->
              <div class="p-6 border-b">
                <h2 class="text-xl font-semibold mb-4">Añadir nuevo invitado</h2>
                <form @submit.prevent="addGuest" class="flex flex-col sm:flex-row space-y-4 sm:space-y-0 sm:space-x-4">
                  <input v-model="newGuest.name" type="text" placeholder="Nombre" class="flex-grow px-3 py-2 border rounded" required>
                  <select v-model="newGuest.gender" class="px-3 py-2 border rounded">
                    <option value="male">Hombre</option>
                    <option value="female">Mujer</option>
                    <option value="other">Otro</option>
                  </select>
                  <button type="submit" class="px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700">Añadir</button>
                </form>
              </div>
              
              <!-- Lista de invitados -->
              <div class="p-6">
                <h2 class="text-xl font-semibold mb-4">Lista de invitados</h2>
                <ul class="space-y-2">
                  <li v-for="guest in filteredGuests" :key="guest.id" 
                      :class="{'bg-green-100': guest.confirmed, 'bg-yellow-100': !guest.confirmed}"
                      class="p-3 rounded flex justify-between items-center">
                    <span>{{ guest.name }} ({{ guest.gender }})</span>
                    <button @click="toggleConfirmation(guest)" 
                            :class="{'bg-green-500': guest.confirmed, 'bg-yellow-500': !guest.confirmed}"
                            class="px-3 py-1 text-white rounded">
                      {{ guest.confirmed ? 'Confirmado' : 'Pendiente' }}
                    </button>
                  </li>
                </ul>
                <p class="mt-4 font-semibold">
                  Invitados confirmados: <span v-text="confirmedCount"></span>
                </p>
              </div>
              
              <!-- Tareas pendientes -->
              <div class="p-6 border-t">
                <button @click="showTasks = !showTasks" class="mb-4 px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700">
                  {{ showTasks ? 'Ocultar' : 'Mostrar' }} tareas pendientes
                </button>
                <ul v-show="showTasks" class="list-disc pl-5">
                  <li>Organizar el catering</li>
                  <li>Reservar el lugar</li>
                  <li>Enviar invitaciones</li>
                  <li>Planificar actividades</li>
                </ul>
              </div>
              
              <!-- Votación de temática -->
              <div class="p-6 border-t">
              <h2 class="text-xl font-semibold mb-4">Vota por la temática del evento</h2>
              <div class="space-y-4">
                <div v-for="theme in themes" :key="theme" class="flex items-center justify-between">
                  <span class="text-lg">{{ theme }}</span>
                  <div class="flex items-center">
                    <button @click="voteTheme(theme)" 
                            class="focus:outline-none transform transition-transform duration-200 hover:scale-110"
                            :class="{ 'text-red-500': themeVotes[theme] > 0, 'text-gray-400': !themeVotes[theme] }"
                            :aria-label="themeVotes[theme] > 0 ? 'Quitar voto' : 'Votar'">
                      <Heart :fill="themeVotes[theme] > 0 ? 'currentColor' : 'none'" size="24" />
                    </button>
                    <span class="ml-2 text-gray-600">{{ themeVotes[theme] || 0 }}</span>
                  </div>
                </div>
              </div>
            </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  
  <script setup>
  import { ref, computed } from 'vue'
  import { Heart } from 'lucide-vue-next'
  const guests = ref([])
  const newGuest = ref({ name: '', gender: 'male' })
  const filter = ref({ status: 'all', gender: ['all'] })
  const showTasks = ref(false)
  const themes = ['Fiesta en la playa', 'Baile de máscaras', 'Aventura futurista']

  const themeVotes = ref({})
  
  const addGuest = () => {
    guests.value.push({
      id: Date.now(),
      name: newGuest.value.name,
      gender: newGuest.value.gender,
      confirmed: false
    })
    newGuest.value = { name: '', gender: 'male' }
  }
  
  const toggleConfirmation = (guest) => {
    guest.confirmed = !guest.confirmed
  }
  
  const filteredGuests = computed(() => {
    return guests.value.filter(guest => {
      const statusMatch = filter.value.status === 'all' || 
        (filter.value.status === 'confirmed') === guest.confirmed
      const genderMatch = filter.value.gender.includes('all') || 
        filter.value.gender.includes(guest.gender)
      return statusMatch && genderMatch
    })
  })
  
  const confirmedCount = computed(() => {
    return guests.value.filter(guest => guest.confirmed).length
  })
  
  const voteTheme = (theme) => {
    if (!themeVotes.value[theme]) {
      themeVotes.value[theme] = 0
    }
    themeVotes.value[theme]++
  }
  </script>