<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import BarraPartes from './BarraPartes.vue'

const CLAVE_MARCADOR = 'mathkids-marcador'

const operaciones = [
  { id: 'multiplicar', etiqueta: '✖️ Multiplicar' },
  { id: 'dividir', etiqueta: '➗ Dividir' },
  { id: 'fracciones', etiqueta: '🍕 Fracciones' },
  { id: 'decimales', etiqueta: '🔢 Decimales' }
]

// --- ESTADO DEL JUEGO ---
const operacionActual = ref('multiplicar') // 'multiplicar' | 'dividir' | 'fracciones' | 'decimales'
const aciertos = ref(0)
const totalIntentos = ref(0)
const bloqueado = ref(false)

const pregunta = ref('')
const respuestaCorrecta = ref(null)
const opciones = ref([])

// Fracciones y decimales: pintar `objetivo` columnas de un total de `partes`
const partes = ref(0)
const objetivo = ref(0)
const pintadas = ref([])

const mensajeModal = ref('')
const mostrarModal = ref(false)
const esCorrecto = ref(true)
const solucion = ref('') // Respuesta correcta que se muestra tras un error
const mostrarBienvenida = ref(false)

const esModoBarra = computed(() => ['fracciones', 'decimales'].includes(operacionActual.value))
const barraSolucion = computed(() => pintadas.value.map((_, i) => i < objetivo.value))

// --- MARCADOR GUARDADO ---
const leerMarcador = () => {
  try {
    return JSON.parse(localStorage.getItem(CLAVE_MARCADOR))
  } catch {
    return null
  }
}

watch([aciertos, totalIntentos], ([a, t]) => {
  try {
    localStorage.setItem(CLAVE_MARCADOR, JSON.stringify({ aciertos: a, totalIntentos: t }))
  } catch {
    // Sin almacenamiento disponible: el marcador solo dura esta sesión
  }
})

const reiniciarMarcador = () => {
  aciertos.value = 0
  totalIntentos.value = 0
  mostrarBienvenida.value = false
}

const confirmarReinicio = () => {
  if (confirm('¿Empezar de cero? Se borrarán los aciertos e intentos.')) reiniciarMarcador()
}

// --- HELPER DE ALEATORIEDAD ---
const aleatorio = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min
const mezclar = (arr) => [...arr].sort(() => Math.random() - 0.5)

// --- GENERACIÓN DE PREGUNTAS ---
const generarOperacion = () => {
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

const generarBarra = () => {
  if (operacionActual.value === 'fracciones') {
    partes.value = aleatorio(2, 10)
    objetivo.value = aleatorio(1, partes.value - 1)
    pregunta.value = `${objetivo.value}/${partes.value}`
  } else {
    // Décimas: la barra siempre tiene 10 partes
    partes.value = 10
    objetivo.value = aleatorio(1, 9)
    pregunta.value = `0.${objetivo.value}`
  }
  pintadas.value = Array(partes.value).fill(false)
}

const generarPregunta = () => {
  bloqueado.value = false
  mostrarModal.value = false
  solucion.value = ''

  if (esModoBarra.value) generarBarra()
  else generarOperacion()
}

// --- EVALUACIÓN DE RESPUESTA ---
// Si se pasa `textoSolucion`, al fallar se muestra la respuesta correcta y se pasa a la siguiente
const registrarResultado = (correcto, textoSolucion = '') => {
  bloqueado.value = true
  totalIntentos.value++
  esCorrecto.value = correcto
  mostrarModal.value = true

  if (correcto) {
    aciertos.value++
    mensajeModal.value = '¡Excelente trabajo! ✨'

    setTimeout(() => {
      generarPregunta()
    }, 1200)
  } else if (textoSolucion) {
    solucion.value = textoSolucion
    mensajeModal.value = 'La respuesta correcta es:'
  } else {
    mensajeModal.value = '¡Casi! Inténtalo de nuevo 💪'
  }
}

const seleccionar = (opcion) => {
  if (bloqueado.value) return
  const textoSolucion =
    operacionActual.value === 'multiplicar' ? `${pregunta.value} = ${respuestaCorrecta.value}` : ''
  registrarResultado(opcion === respuestaCorrecta.value, textoSolucion)
}

const alternarParte = (i) => {
  if (bloqueado.value) return
  pintadas.value[i] = !pintadas.value[i]
}

const comprobar = () => {
  if (bloqueado.value) return
  const cantidad = pintadas.value.filter(Boolean).length
  const equivalencia =
    operacionActual.value === 'decimales' ? `${pregunta.value} = ${objetivo.value}/10` : pregunta.value
  registrarResultado(
    cantidad === objetivo.value,
    `${equivalencia} → ${objetivo.value} de ${partes.value} partes`
  )
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
  const guardado = leerMarcador()
  if (guardado?.totalIntentos > 0) {
    aciertos.value = guardado.aciertos
    totalIntentos.value = guardado.totalIntentos
    mostrarBienvenida.value = true
  }
  generarPregunta()
})
</script>

<template>
  <div class="max-w-sm mx-auto p-4 flex flex-col items-center gap-4">
    <!-- Selector de Operación -->
    <div class="grid grid-cols-2 gap-2 w-full bg-slate-200 p-1 rounded-2xl">
      <button
        v-for="op in operaciones"
        :key="op.id"
        @click="cambiarOperacion(op.id)"
        class="py-2.5 rounded-xl font-bold text-sm transition-all cursor-pointer"
        :class="operacionActual === op.id ? 'bg-white text-indigo-600 shadow-sm' : 'text-slate-600'"
      >
        {{ op.etiqueta }}
      </button>
    </div>

    <!-- Marcador -->
    <div class="w-full bg-white p-4 rounded-2xl border border-slate-200 flex justify-around items-center text-center shadow-sm">
      <div>
        <span class="block text-xs font-semibold text-slate-400">Aciertos</span>
        <span class="text-2xl font-black text-emerald-600">{{ aciertos }}</span>
      </div>
      <div class="w-px self-stretch bg-slate-200"></div>
      <div>
        <span class="block text-xs font-semibold text-slate-400">Intentos</span>
        <span class="text-2xl font-black text-slate-700">{{ totalIntentos }}</span>
      </div>
      <div class="w-px self-stretch bg-slate-200"></div>
      <button
        @click="confirmarReinicio"
        class="text-xs font-bold text-slate-400 hover:text-rose-500 cursor-pointer"
        aria-label="Reiniciar marcador"
      >
        <span class="block text-2xl">↺</span>
        Reiniciar
      </button>
    </div>

    <!-- Tarjeta Principal -->
    <main class="w-full bg-white p-6 rounded-3xl border border-slate-200 shadow-sm text-center">
      <template v-if="esModoBarra">
        <span class="text-xs font-bold text-slate-400 uppercase tracking-widest">Pinta</span>
        <div class="text-5xl font-black text-slate-800 my-6">
          <span v-if="operacionActual === 'decimales'">{{ pregunta }}</span>
          <span v-else class="inline-flex flex-col items-center leading-none">
            <span>{{ objetivo }}</span>
            <span class="w-14 h-1.5 bg-slate-800 rounded-full my-2"></span>
            <span>{{ partes }}</span>
          </span>
        </div>

        <BarraPartes :pintadas="pintadas" :interactiva="!bloqueado" @alternar="alternarParte" />

        <button
          @click="comprobar"
          :disabled="bloqueado || !pintadas.some(Boolean)"
          class="w-full mt-6 py-4 text-xl font-black text-white bg-indigo-600 rounded-2xl transition-all cursor-pointer active:scale-95 disabled:opacity-50"
        >
          Comprobar
        </button>
      </template>

      <template v-else>
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
      </template>
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

        <template v-if="solucion">
          <p class="text-2xl font-black mb-4">{{ solucion }}</p>
          <div v-if="esModoBarra" class="mb-4">
            <BarraPartes :pintadas="barraSolucion" :interactiva="false" compacta />
          </div>
          <button
            @click="generarPregunta"
            class="w-full py-2 bg-white text-rose-600 font-bold rounded-xl shadow cursor-pointer"
          >
            Siguiente
          </button>
        </template>
        <button
          v-else-if="!esCorrecto"
          @click="reintentar"
          class="w-full py-2 bg-white text-rose-600 font-bold rounded-xl shadow cursor-pointer"
        >
          Reintentar
        </button>
      </div>
    </div>

    <!-- Bienvenida: continuar o empezar de cero -->
    <div
      v-if="mostrarBienvenida"
      class="fixed inset-0 bg-slate-900/40 backdrop-blur-xs flex items-center justify-center p-4 z-50"
    >
      <div class="p-6 rounded-3xl text-center bg-white max-w-xs w-full shadow-xl border-4 border-indigo-200">
        <div class="text-5xl mb-2">👋</div>
        <h2 class="text-2xl font-black text-slate-800 mb-2">¡Hola de nuevo!</h2>
        <p class="text-sm text-slate-500 mb-4">
          Llevas <strong class="text-emerald-600">{{ aciertos }}</strong> aciertos de
          <strong class="text-slate-700">{{ totalIntentos }}</strong> intentos.
        </p>
        <button
          @click="mostrarBienvenida = false"
          class="w-full py-3 mb-2 bg-indigo-600 text-white font-bold rounded-xl shadow cursor-pointer"
        >
          Continuar 🚀
        </button>
        <button
          @click="reiniciarMarcador"
          class="w-full py-3 bg-slate-100 text-slate-600 font-bold rounded-xl cursor-pointer"
        >
          Empezar de cero
        </button>
      </div>
    </div>
  </div>
</template>
