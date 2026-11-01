<template>
  <div class="pos-wrapper">
    <!-- 主機台容器 -->
    <main class="pos-cabinet">
      
      <!-- 頂部標題與狀態列 -->
      <header class="pos-header">
        <div class="brand">
          <div>
            <h1 class="brand-title">森森手搖日常 POS</h1>
            <p class="brand-subtitle">Fresh Tea & Sweet Hours</p>
          </div>
        </div>

        <div class="status-group">
          <!-- 計時器 -->
          <div class="pill-badge time-badge" :class="{ 'time-warning': timeLeft <= 10 }">
            <span class="badge-label">剩餘時間</span>
            <span class="badge-val">{{ timeLeft }}s</span>
          </div>

          <!-- 得分看板 -->
          <div class="pill-badge score-badge">
            <span class="badge-label">營業額</span>
            <span class="badge-val">{{ score }} 分</span>
          </div>

          <!-- 重新開始 / 重置 -->
          <button class="btn btn-reset" @click="startGame">
            {{ isPlaying ? '重新開局' : '開始營業' }}
          </button>
        </div>
      </header>

      <!-- 遊戲說明看板 -->
      <section class="tutorial-board">
        <div class="tutorial-title">
          <span>吧台作業流程</span>
        </div>
        <div class="tutorial-steps">
          <div class="step-item">
            <span class="step-num">1</span>
            <span>確認候單區<strong>最左側急件</strong></span>
          </div>
          <span class="step-arrow">➔</span>
          <div class="step-item">
            <span class="step-num">2</span>
            <span>下方依序選擇<strong>茶底、甜度、冰量、配料</strong></span>
          </div>
          <span class="step-arrow">➔</span>
          <div class="step-item">
            <span class="step-num">3</span>
            <span>點擊<strong>「確認出單」</strong>完成核對</span>
          </div>
        </div>
      </section>

      <!-- 中間候單區 -->
      <section class="queue-section">
        <div class="section-header">
          <span class="section-title">
            <span class="dot"></span>
            顧客點單清單
          </span>
          <span class="queue-count">候單客數：{{ orders.length }}/3</span>
        </div>

        <div class="cards-grid">
          <div 
            v-for="(order, index) in orders" 
            :key="order.id"
            :class="['order-card', { 'is-target': index === 0 }]"
          >
            <div v-if="index === 0" class="target-tag">優先出餐</div>
            
            <div class="card-head">
              <span class="ticket-no">#00{{ order.id }}</span>
              <span class="priority-label">{{ index === 0 ? '目前急件' : '排隊中' }}</span>
            </div>

            <div class="card-body">
              <div class="drink-name">{{ order.tea }}</div>
              <div class="tags-group">
                <span class="tag tag-sugar">{{ order.sugar }}</span>
                <span class="tag tag-ice">{{ order.ice }}</span>
                <span 
                  v-for="t in order.toppings" 
                  :key="t" 
                  class="tag tag-topping"
                >
                  + {{ t }}
                </span>
                <span v-if="order.toppings.length === 0" class="tag tag-empty">純茶微甜</span>
              </div>
            </div>

            <div class="card-foot">
              <span>外帶客訂</span>
              <span>單價 $45</span>
            </div>
          </div>
        </div>
      </section>

      <!-- 下方吧台操作區 -->
      <section class="bar-section">
        <div class="bar-header">
          <h2 class="bar-title">調製吧台操作區</h2>
          <transition name="pop">
            <span v-if="feedbackMsg" :class="['feedback-toast', feedbackType]">
              {{ feedbackMsg }}
            </span>
          </transition>
        </div>

        <div class="controls-grid">
          <!-- 1. 茶底子元件 -->
          <TeaSelector 
            v-model="currentDrink.tea" 
            :options="TEA_OPTIONS" 
          />

          <!-- 2. 甜度子元件 -->
          <SugarSelector 
            v-model="currentDrink.sugar" 
            :options="SUGAR_OPTIONS" 
          />

          <!-- 3. 冰量子元件 -->
          <IceSelector 
            v-model="currentDrink.ice" 
            :options="ICE_OPTIONS" 
          />

          <!-- 4. 配料特調子元件 -->
          <ToppingSelector 
            v-model="currentDrink.toppings" 
            :options="TOPPING_OPTIONS" 
          />
        </div>

        <!-- 結算與出單操作 -->
        <div class="bar-footer">
          <div class="current-mix">
            <span class="mix-label">當前調配內容：</span>
            <span class="mix-details">
              {{ currentDrink.tea }} / {{ currentDrink.sugar }} / {{ currentDrink.ice }} / 
              {{ currentDrink.toppings.length ? currentDrink.toppings.join('、') : '不加料' }}
            </span>
          </div>

          <button 
            class="btn btn-submit"
            :disabled="!isPlaying"
            @click="submitOrder"
          >
            確認出單
          </button>
        </div>
      </section>

    </main>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import TeaSelector from './components/TeaSelector.vue'
import SugarSelector from './components/SugarSelector.vue'
import IceSelector from './components/IceSelector.vue'
import ToppingSelector from './components/ToppingSelector.vue'

const TEA_OPTIONS = ['青茶', '紅茶', '烏龍茶', '春茶']
const SUGAR_OPTIONS = ['無糖', '微糖', '半糖', '全糖']
const ICE_OPTIONS = ['正常冰', '少冰', '微冰', '去冰']
const TOPPING_OPTIONS = ['珍珠', '茶凍', '奶蓋', '粉條']

// 狀態管理
const score = ref(0)
const timeLeft = ref(60)
const isPlaying = ref(false)
const feedbackMsg = ref('')
const feedbackType = ref('success')

const currentDrink = reactive({
  tea: '青茶',
  sugar: '微糖',
  ice: '少冰',
  toppings: []
})

const orders = ref([])
let orderSequence = 1
let timerInterval = null

const generateRandomOrder = () => {
  const randomTea = TEA_OPTIONS[Math.floor(Math.random() * TEA_OPTIONS.length)]
  const randomSugar = SUGAR_OPTIONS[Math.floor(Math.random() * SUGAR_OPTIONS.length)]
  const randomIce = ICE_OPTIONS[Math.floor(Math.random() * ICE_OPTIONS.length)]
  const shuffled = [...TOPPING_OPTIONS].sort(() => 0.5 - Math.random())
  const count = Math.floor(Math.random() * 3)
  const selectedToppings = shuffled.slice(0, count)

  return {
    id: orderSequence++,
    tea: randomTea,
    sugar: randomSugar,
    ice: randomIce,
    toppings: selectedToppings
  }
}

const initOrders = () => {
  orders.value = [
    generateRandomOrder(),
    generateRandomOrder(),
    generateRandomOrder()
  ]
}

const resetDrinkForm = () => {
  currentDrink.tea = '青茶'
  currentDrink.sugar = '微糖'
  currentDrink.ice = '少冰'
  currentDrink.toppings = []
}

const startGame = () => {
  score.value = 0
  timeLeft.value = 60
  isPlaying.value = true
  feedbackMsg.value = ''
  initOrders()
  resetDrinkForm()

  if (timerInterval) clearInterval(timerInterval)

  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
    } else {
      clearInterval(timerInterval)
      isPlaying.value = false
      feedbackMsg.value = `時間到！本日總營業額：${score.value} 分`
      feedbackType.value = 'success'
    }
  }, 1000)
}

const submitOrder = () => {
  if (!isPlaying.value || orders.value.length === 0) return

  const target = orders.value[0]
  const isTeaMatch = currentDrink.tea === target.tea
  const isSugarMatch = currentDrink.sugar === target.sugar
  const isIceMatch = currentDrink.ice === target.ice
  const isToppingsMatch = 
    currentDrink.toppings.length === target.toppings.length &&
    currentDrink.toppings.every(t => target.toppings.includes(t))

  if (isTeaMatch && isSugarMatch && isIceMatch && isToppingsMatch) {
    score.value += 10
    feedbackMsg.value = '調配完全正確！+10 分'
    feedbackType.value = 'success'
    orders.value.shift()
    orders.value.push(generateRandomOrder())
    resetDrinkForm()
  } else {
    score.value = Math.max(0, score.value - 5)
    feedbackMsg.value = '配方調錯了！顧客退單 -5 分'
    feedbackType.value = 'error'
  }
}

onMounted(() => {
  startGame()
})

onUnmounted(() => {
  if (timerInterval) clearInterval(timerInterval)
})
</script>

<style scoped>
.pos-wrapper {
  min-height: 100vh;
  background-color: #FFF8F0;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 24px 16px;
  box-sizing: border-box;
  font-family: 'PingFang TC', 'Microsoft JhengHei', -apple-system, sans-serif;
  color: #4A3E3D;
}

.pos-cabinet {
  width: 100%;
  max-width: 960px;
  background: #FFFFFF;
  border: 3.5px solid #4A3E3D;
  border-radius: 32px;
  box-shadow: 0px 8px 0px #4A3E3D;
  padding: 28px;
  display: flex;
  flex-direction: column;
  gap: 22px;
  box-sizing: border-box;
}

/* 1. Header 區域 */
.pos-header {
  background: #FFF2DF;
  border: 2.5px solid #4A3E3D;
  border-radius: 20px;
  padding: 14px 20px;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 14px;
}

.brand {
  display: flex;
  align-items: center;
}
.brand-title {
  margin: 0;
  font-size: 20px;
  font-weight: 800;
  color: #4A3E3D;
  letter-spacing: 0.5px;
}
.brand-subtitle {
  margin: 2px 0 0 0;
  font-size: 11px;
  color: #9C8987;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.status-group {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.pill-badge {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #FFFFFF;
  border: 2px solid #4A3E3D;
  border-radius: 12px;
  padding: 6px 14px;
  box-shadow: 2px 2px 0px #4A3E3D;
}

.badge-label {
  font-size: 12px;
  font-weight: 700;
  color: #8C7A78;
}
.badge-val {
  font-size: 17px;
  font-weight: 900;
  color: #4A3E3D;
}
.time-warning {
  background: #FFD2D2;
}
.time-warning .badge-val {
  color: #E24A4A;
  animation: pulse 0.8s infinite alternate;
}

.score-badge {
  background: #E8F8F5;
}
.score-badge .badge-val {
  color: #2D9F83;
}

/* 2. 說明看板 */
.tutorial-board {
  background: #FDFBF7;
  border: 2px dashed #CBB8B0;
  border-radius: 16px;
  padding: 12px 18px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.tutorial-title {
  font-size: 12px;
  font-weight: 800;
  color: #8C756B;
}

.tutorial-steps {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  font-size: 12px;
  font-weight: 600;
  color: #635350;
  flex-wrap: wrap;
}

.step-item {
  display: flex;
  align-items: center;
  gap: 6px;
}

.step-num {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  background: #FFE3B3;
  border: 1.5px solid #4A3E3D;
  border-radius: 50%;
  font-size: 11px;
  font-weight: 800;
  color: #4A3E3D;
}

.step-arrow {
  color: #CBB8B0;
  font-weight: bold;
}

/* 3. 候單區 */
.queue-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 4px;
}

.section-title {
  font-size: 13px;
  font-weight: 800;
  color: #6F5E5C;
  display: flex;
  align-items: center;
  gap: 6px;
}
.dot {
  width: 8px;
  height: 8px;
  background: #FFB347;
  border: 1.5px solid #4A3E3D;
  border-radius: 50%;
}
.queue-count {
  font-size: 11px;
  font-weight: 700;
  color: #9C8987;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 14px;
}

@media (max-width: 640px) {
  .cards-grid {
    grid-template-columns: 1fr;
  }
}

/* 訂單卡片 */
.order-card {
  position: relative;
  background: #FAF7F2;
  border: 2.5px solid #C4B5AF;
  border-radius: 18px;
  padding: 14px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 10px;
  opacity: 0.75;
  transition: all 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.order-card.is-target {
  opacity: 1;
  background: #FFFFFF;
  border-color: #4A3E3D;
  box-shadow: 0px 6px 0px #FFE3B3, 0px 6px 0px 2.5px #4A3E3D;
  transform: translateY(-2px);
}

.target-tag {
  position: absolute;
  top: -11px;
  right: 12px;
  background: #FF7B90;
  color: #FFFFFF;
  font-size: 10px;
  font-weight: 900;
  border: 1.5px solid #4A3E3D;
  padding: 2px 8px;
  border-radius: 999px;
  letter-spacing: 0.5px;
}

.card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1.5px dashed #E5DCD6;
  padding-bottom: 6px;
}

.ticket-no {
  font-size: 11px;
  font-weight: 800;
  color: #B5A49F;
  font-family: monospace;
}
.priority-label {
  font-size: 11px;
  font-weight: 700;
  color: #8C7A78;
}

.drink-name {
  font-size: 16px;
  font-weight: 800;
  color: #4A3E3D;
  margin-bottom: 8px;
}

.tags-group {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.tag {
  font-size: 11px;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 8px;
  border: 1.5px solid #4A3E3D;
  line-height: 1.2;
}
.tag-sugar {
  background: #EAF4FC;
  color: #4B7EAB;
}
.tag-ice {
  background: #E1F8FA;
  color: #1F8A99;
}
.tag-topping {
  background: #FFF1E6;
  color: #B86B35;
}
.tag-empty {
  background: #F3EFEA;
  color: #9C8987;
  border-color: #D6CCC7;
}

.card-foot {
  border-top: 1.5px solid #F1ECE6;
  padding-top: 6px;
  display: flex;
  justify-content: space-between;
  font-size: 11px;
  font-weight: 600;
  color: #9C8987;
}

/* 4. 吧台操作區 (對齊整頓) */
.bar-section {
  background: #FFF9F3;
  border: 2.5px solid #4A3E3D;
  border-radius: 22px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.bar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #EFE4DA;
  padding-bottom: 10px;
  min-height: 32px;
}

.bar-title {
  margin: 0;
  font-size: 15px;
  font-weight: 800;
  color: #4A3E3D;
}

.feedback-toast {
  font-size: 12px;
  font-weight: 800;
  padding: 4px 12px;
  border-radius: 999px;
  border: 2px solid #4A3E3D;
}
.feedback-toast.success {
  background: #D7F9E8;
  color: #1A7E56;
}
.feedback-toast.error {
  background: #FFDADB;
  color: #BD2B38;
}

/* 四區等寬網格 */
.controls-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

@media (max-width: 860px) {
  .controls-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 16px;
  }
}

@media (max-width: 480px) {
  .controls-grid {
    grid-template-columns: 1fr;
  }
}

/* 底部出單欄 */
.bar-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 14px;
  border-top: 2px solid #EFE4DA;
  padding-top: 14px;
}

.current-mix {
  font-size: 13px;
  color: #6F5E5C;
  display: flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
}

.mix-label {
  font-weight: 800;
}

.mix-details {
  background: #FFFFFF;
  border: 1.5px solid #4A3E3D;
  border-radius: 10px;
  padding: 4px 10px;
  font-weight: 700;
  color: #4A3E3D;
}

.btn {
  font-family: inherit;
  font-weight: 800;
  border: 2.5px solid #4A3E3D;
  cursor: pointer;
  user-select: none;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.12s cubic-bezier(0.34, 1.56, 0.64, 1), box-shadow 0.12s ease;
}

.btn:active:not(:disabled) {
  transform: scale(0.93) translateY(3px);
  box-shadow: 0px 0px 0px #4A3E3D !important;
}

.btn-reset {
  background: #4A3E3D;
  color: #FFFFFF;
  border-radius: 12px;
  padding: 8px 14px;
  font-size: 13px;
  box-shadow: 2px 2px 0px #8C7A78;
}
.btn-reset:hover {
  background: #5E504F;
}

.btn-submit {
  background: #FFC043;
  color: #4A3E3D;
  font-size: 15px;
  padding: 10px 28px;
  border-radius: 14px;
  box-shadow: 3px 3px 0px #4A3E3D;
}
.btn-submit:hover:not(:disabled) {
  background: #FFCB65;
  transform: translateY(-2px);
  box-shadow: 3px 5px 0px #4A3E3D;
}
.btn-submit:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(0.6);
}

@keyframes pulse {
  0% { opacity: 0.6; }
  100% { opacity: 1; }
}

.pop-enter-active {
  animation: popIn 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.pop-leave-active {
  animation: popIn 0.2s reverse ease;
}
@keyframes popIn {
  0% { transform: scale(0.7); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}
</style>