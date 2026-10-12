<script setup>
import { ref, onMounted } from 'vue'

// --- ESTADO DEL JUEGO ---
const operacionActual = ref('multiplicar') // 'multiplicar' | 'dividir'
const aciertos = ref(0)
const totalIntentos = ref(0)
const bloqueado = ref(false)

const pregunta = ref('')
const respuestaCorrecta = ref(null)
const opciones = ref([])

const mensajeModal = ref('')
const mostrarModal = ref(false)
const esCorrecto = ref(true)

// --- HELPER DE ALEATORIEDAD ---
const aleatorio = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min
const mezclar = (arr) => [...arr].sort(() => Math.random() - 0.5)

// --- GENERACIÓN DE PREGUNTAS ---
const generarPregunta = () => {
  bloqueado.value = false
  mostrarModal.value = false

  let a, b, correcta, simbolo

  if (operacionActual.value === 'multiplicar') {
    a = aleatorio(1, 10)
    b = aleatorio(1, 10)
    correcta = a * b
    simbolo = '×'
  } else {
    // Divisiones exactas sencillas
    b = aleatorio(2, 10) // Divisor
    const cociente = aleatorio(1, 10)
    a = b * cociente // Dividendo
    correcta = cociente
    simbolo = '÷'
  }

  pregunta.value = `${a} ${simbolo} ${b}`
  respuestaCorrecta.value = correcta

  // Generar 2 distractores únicos
  const falsas = new Set()
  while (falsas.size < 2) {
    const delta = aleatorio(1, 4) * (Math.random() < 0.5 ? 1 : -1)
    const err = correcta + delta
    if (err > 0 && err !== correcta) falsas.add(err)
  }

  opciones.value = mezclar([correcta, ...Array.from(falsas)])
}

// --- EVALUACIÓN DE RESPUESTA ---
const seleccionar = (opcion) => {
  if (bloqueado.value) return
  bloqueado.value = true
  totalIntentos.value++

  if (opcion === respuestaCorrecta.value) {
    aciertos.value++
    esCorrecto.value = true
    mensajeModal.value = '¡Excelente trabajo! ✨'
    mostrarModal.value = true

    setTimeout(() => {
      generarPregunta()
    }, 1200)
  } else {
    esCorrecto.value = false
    mensajeModal.value = '¡Casi! Inténtalo de nuevo 💪'
    mostrarModal.value = true
  }
}

const cambiarOperacion = (tipo) => {
  if (operacionActual.value === tipo) return
  operacionActual.value = tipo
  generarPregunta()
}

const reintentar = () => {
  mostrarModal.value = false
  bloqueado.value = false
}

onMounted(() => {
  generarPregunta()
})
</script>

<template>
  <div class="max-w-sm mx-auto p-4 flex flex-col items-center gap-4">
    <!-- Selector de Operación -->
    <div class="grid grid-cols-2 gap-2 w-full bg-slate-200 p-1 rounded-2xl">
      <button
        @click="cambiarOperacion('multiplicar')"
        class="py-2.5 rounded-xl font-bold text-sm transition-all cursor-pointer"
        :class="operacionActual === 'multiplicar' ? 'bg-white text-indigo-600 shadow-sm' : 'text-slate-600'"
      >
        ✖️ Multiplicar
      </button>
      <button
        @click="cambiarOperacion('dividir')"
        class="py-2.5 rounded-xl font-bold text-sm transition-all cursor-pointer"
        :class="operacionActual === 'dividir' ? 'bg-white text-indigo-600 shadow-sm' : 'text-slate-600'"
      >
        ➗ Dividir
      </button>
    </div>

    <!-- Marcador -->
    <div class="w-full bg-white p-4 rounded-2xl border border-slate-200 flex justify-around text-center shadow-sm">
      <div>
        <span class="block text-xs font-semibold text-slate-400">Aciertos</span>
        <span class="text-2xl font-black text-emerald-600">{{ aciertos }}</span>
      </div>
      <div class="w-px bg-slate-200"></div>
      <div>
        <span class="block text-xs font-semibold text-slate-400">Intentos</span>
        <span class="text-2xl font-black text-slate-700">{{ totalIntentos }}</span>
      </div>
    </div>

    <!-- Tarjeta Principal -->
    <main class="w-full bg-white p-6 rounded-3xl border border-slate-200 shadow-sm text-center">
      <span class="text-xs font-bold text-slate-400 uppercase tracking-widest">Resuelve</span>
      <h1 class="text-5xl font-black text-slate-800 my-6">{{ pregunta }} = ?</h1>

      <!-- Opciones -->
      <div class="grid grid-cols-3 gap-3">
        <button
          v-for="(opcion, index) in opciones"
          :key="index"
          :disabled="bloqueado"
          @click="seleccionar(opcion)"
          class="py-4 text-2xl font-black bg-slate-50 border-2 border-slate-200 hover:border-indigo-400 hover:bg-indigo-50 rounded-2xl transition-all cursor-pointer active:scale-95 disabled:opacity-50"
        >
          {{ opcion }}
        </button>
      </div>
    </main>

    <!-- Modal Feedback -->
    <div
      v-if="mostrarModal"
      class="fixed inset-0 bg-slate-900/40 backdrop-blur-xs flex items-center justify-center p-4 z-50"
    >
      <div 
        class="p-6 rounded-3xl text-center text-white max-w-xs w-full shadow-xl border-4"
        :class="esCorrecto ? 'bg-emerald-500 border-emerald-300' : 'bg-rose-500 border-rose-300'"
      >
        <div class="text-5xl mb-2">{{ esCorrecto ? '✨' : '💪' }}</div>
        <h2 class="text-2xl font-black mb-2">{{ esCorrecto ? '¡Muy bien!' : '¡Casi!' }}</h2>
        <p class="text-sm opacity-90 mb-4">{{ mensajeModal }}</p>
        <button
          v-if="!esCorrecto"
          @click="reintentar"
          class="w-full py-2 bg-white text-rose-600 font-bold rounded-xl shadow cursor-pointer"
        >
          Reintentar
        </button>
      </div>
    </div>
  </div>
</template>