<template>
  <div class="control-box">
    <span class="box-label">配料特調 (複選)</span>
    <div class="chip-grid">
      <button
        v-for="topping in options"
        :key="topping"
        type="button"
        class="chip-btn topping-chip"
        :class="{ active: modelValue.includes(topping) }"
        @click="toggleTopping(topping)"
      >
        {{ topping }}
      </button>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  modelValue: {
    type: Array,
    required: true,
    default: () => []
  },
  options: {
    type: Array,
    default: () => ['珍珠', '茶凍', '奶蓋', '粉條']
  }
})

const emit = defineEmits(['update:modelValue'])

const toggleTopping = (topping) => {
  const currentList = [...props.modelValue]
  const targetIndex = currentList.indexOf(topping)

  if (targetIndex > -1) {
    currentList.splice(targetIndex, 1)
  } else {
    currentList.push(topping)
  }

  emit('update:modelValue', currentList)
}
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

.topping-chip.active {
  background: #FFD4DB;
  border-color: #4A3E3D;
  box-shadow: 2px 2px 0px #4A3E3D;
  color: #4A3E3D;
}
</style>