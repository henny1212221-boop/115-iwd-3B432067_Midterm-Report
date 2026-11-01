<template>
  <div class="control-box">
    <span class="box-label">茶底選擇</span>
    <div class="chip-grid">
      <button
        v-for="tea in options"
        :key="tea"
        type="button"
        class="chip-btn tea-chip"
        :class="{ active: modelValue === tea }"
        @click="emit('update:modelValue', tea)"
      >
        {{ tea }}
      </button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  modelValue: {
    type: String,
    required: true
  },
  options: {
    type: Array,
    default: () => ['青茶', '紅茶', '烏龍茶', '春茶']
  }
})

const emit = defineEmits(['update:modelValue'])
</script>

<style scoped>
.control-box {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.box-label {
  font-size: 13px;
  font-weight: 800;
  color: #8C756B;
  white-space: nowrap;
  letter-spacing: 0.5px;
}

.chip-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
}

.chip-btn {
  height: 42px;
  width: 100%;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #FFFFFF;
  border: 2px solid #D6CCC7;
  border-radius: 12px;
  padding: 0 4px;
  font-size: 13px;
  font-weight: 700;
  color: #7A6966;
  cursor: pointer;
  user-select: none;
  font-family: inherit;
  transition: transform 0.15s cubic-bezier(0.34, 1.56, 0.64, 1), box-shadow 0.15s ease, border-color 0.15s ease;
}

.chip-btn:hover {
  transform: translateY(-2px);
  border-color: #9C8987;
}

.chip-btn:active {
  transform: scale(0.92);
}

.tea-chip.active {
  background: #FFE8A3;
  border-color: #4A3E3D;
  box-shadow: 2px 2px 0px #4A3E3D;
  color: #4A3E3D;
}
</style>