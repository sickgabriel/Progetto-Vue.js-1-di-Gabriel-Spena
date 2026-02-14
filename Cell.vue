<script setup>
import { X, Circle } from 'lucide-vue-next'

const props = defineProps({
  value: { type: String, default: null },
  index: { type: Number, required: true },
  disabled: { type: Boolean, default: false },
  isWinning: { type: Boolean, default: false },
})

const emit = defineEmits(['click'])

const handleClick = () => {
  if (!props.disabled) emit('click', { index: props.index })
}
</script>

<template>
  <button
    :aria-label="value ? `Cell ${index + 1}: ${value}` : `Cell ${index + 1} empty`"
    :disabled="disabled"
    @click="handleClick"
    class="aspect-square w-full flex items-center justify-center border-[3px] border-[#1a4332] transition-colors duration-200"
    :class="[
      !value && !disabled ? 'hover:bg-emerald-50' : '',
      disabled ? 'cursor-not-allowed' : 'cursor-pointer',
      isWinning ? 'glow' : ''
    ]"
  >
    <transition name="symbol" mode="out-in">
      <X v-if="value === 'X'" :size="72" class="text-emerald-600" />
      <Circle v-else-if="value === 'O'" :size="72" class="text-yellow-500" />
    </transition>
  </button>
  </template>

<style scoped>
@keyframes glowPulse {
  0%   { box-shadow: 0 0 0 0 rgba(26, 67, 50, 0.0), inset 0 0 0 0 rgba(26, 67, 50, 0.0); }
  50%  { box-shadow: 0 0 24px 6px rgba(16, 185, 129, 0.35), inset 0 0 12px 2px rgba(245, 158, 11, 0.25); }
  100% { box-shadow: 0 0 0 0 rgba(26, 67, 50, 0.0), inset 0 0 0 0 rgba(26, 67, 50, 0.0); }
}
.glow {
  animation: glowPulse 1.3s ease-in-out infinite;
}
</style>
