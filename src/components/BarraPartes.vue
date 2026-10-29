<script setup>
// Rectángulo dividido en columnas que se pintan al tocarlas
defineProps({
  pintadas: { type: Array, required: true }, // un booleano por columna
  interactiva: { type: Boolean, default: true },
  compacta: { type: Boolean, default: false }
})
defineEmits(['alternar'])
</script>

<template>
  <div
    class="flex w-full rounded-2xl border-4 border-indigo-300 overflow-hidden bg-white"
    :class="compacta ? 'h-12' : 'h-24'"
  >
    <button
      v-for="(pintada, i) in pintadas"
      :key="i"
      type="button"
      :disabled="!interactiva"
      :aria-label="`Parte ${i + 1}`"
      :aria-pressed="pintada"
      @click="$emit('alternar', i)"
      class="flex-1 border-indigo-200 transition-colors"
      :class="[
        pintada ? 'bg-indigo-500' : 'bg-white',
        i < pintadas.length - 1 ? 'border-r-2' : '',
        interactiva ? 'cursor-pointer' : ''
      ]"
    />
  </div>
</template>
