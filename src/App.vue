<template>
  <div class="app-container">
    <div class="background-decoration"></div>
    <div class="container">
      <Transition name="fade" mode="out-in">
        <!-- 欢迎界面 -->
        <div v-if="currentStep === 'welcome'" class="step-container glass-card welcome-container">
          <div class="welcome-content">
            <h1 class="welcome-title">欢迎使用生日聚会计划便签</h1>
            <p class="welcome-text">让我们一起规划一个完美的生日聚会吧！</p>
            <el-button type="primary" @click="nextStep" class="welcome-button" :class="{'button-hover': buttonHover}" 
              @mouseenter="buttonHover = true" 
              @mouseleave="buttonHover = false">
              <el-icon class="button-icon"><ArrowRight /></el-icon>
              开始规划
            </el-button>
          </div>
          <div class="welcome-decoration"></div>
        </div>

        <!-- 日期输入界面 -->
        <div v-else-if="currentStep === 'dateInput'" class="step-container glass-card centered-content">
          <h2>请输入日期信息</h2>
          <div class="form-group">
            <div class="form-item">
              <el-icon><Calendar /></el-icon>
              <span>出生日期：</span>
              <el-date-picker 
                v-model="birthDate" 
                type="date" 
                placeholder="选择出生日期"
                :clearable="false"
                class="date-picker" 
              />
            </div>
            <div class="form-item">
              <el-icon><Timer /></el-icon>
              <span>今天日期：</span>
              <el-date-picker 
                v-model="today" 
                type="date" 
                placeholder="选择今天日期"
                :clearable="false"
                class="date-picker" 
              />
            </div>
          </div>
          <Transition name="slide-up">
            <div v-if="daysUntilNextBirthday !== null" class="result-text">
              <el-icon><Bell /></el-icon>
              距离下次生日还有 <span class="highlight">{{ daysUntilNextBirthday }}</span> 天
            </div>
          </Transition>
          <el-button type="primary" @click="nextStep" :disabled="!birthDate || !today">
            <el-icon class="button-icon"><ArrowRight /></el-icon>
            继续
          </el-button>
        </div>

        <!-- 计划制定界面 -->
        <div v-else-if="currentStep === 'planDate'" class="step-container glass-card centered-content">
          <h2>生日聚会计划制定</h2>
          <div class="info-card">
            <div class="info-item">
              <el-icon><Calendar /></el-icon>
              <span>下次生日日期：</span>
              <span class="highlight">{{ nextBirthdayDate }}</span>
            </div>
            <div class="info-item">
              <el-icon><Timer /></el-icon>
              <span>距离今天：</span>
              <span class="highlight">{{ daysUntilNextBirthday }} 天</span>
            </div>
          </div>
          <div class="form-item">
            <el-icon><AlarmClock /></el-icon>
            <span>提前天数：</span>
            <el-input-number 
              v-model="daysInAdvance" 
              :min="1" 
              :max="daysUntilNextBirthday" 
              class="number-input"
            />
          </div>
          <Transition name="slide-up">
            <div v-if="planDate" class="result-card">
              <p class="plan-date">
                <el-icon><Calendar /></el-icon>
                计划日期：<span class="highlight">{{ planDate }}</span>
              </p>
              <p v-if="isWorkday" class="adjust-notice">
                <el-icon><Warning /></el-icon>
                由于是工作日，已调整为最近的周六
              </p>
            </div>
          </Transition>
          <el-button type="primary" @click="nextStep" :disabled="!daysInAdvance">
            <el-icon class="button-icon"><Check /></el-icon>
            确认
          </el-button>
        </div>

        <!-- 结果显示界面 -->
        <div v-else-if="currentStep === 'result'" class="step-container glass-card centered-content">
          <h2>计划结果</h2>
          <div class="result-info">
            <div class="info-item">
              <el-icon><Calendar /></el-icon>
              <span>下次生日日期：</span>
              <span class="highlight">{{ nextBirthdayDate }}</span>
            </div>
            <div class="info-item">
              <el-icon><Timer /></el-icon>
              <span>距离今天天数：</span>
              <span class="highlight">{{ daysUntilNextBirthday }} 天</span>
            </div>
            <div class="info-item">
              <el-icon><AlarmClock /></el-icon>
              <span>提前准备天数：</span>
              <span class="highlight">{{ daysInAdvance }} 天</span>
            </div>
            <div class="info-item">
              <el-icon><Calendar /></el-icon>
              <span>计划制定日期：</span>
              <span class="highlight">{{ planDate }}</span>
            </div>
          </div>
          <div class="button-group">
            <el-button @click="resetPlanDate">
              <el-icon class="button-icon"><RefreshLeft /></el-icon>
              重新制定
            </el-button>
            <el-button type="primary" @click="nextStep">
              <el-icon class="button-icon"><Check /></el-icon>
              确认完成
            </el-button>
          </div>
        </div>

        <!-- 结束界面 -->
        <div v-else-if="currentStep === 'end'" class="step-container glass-card">
          <h2>计划已完成</h2>
          <div class="end-message">
            <el-icon class="success-icon"><SuccessFilled /></el-icon>
            <p>您的生日计划将在 <span class="highlight">{{ planDate }}</span> 制定</p>
          </div>
          <el-button type="primary" @click="resetAll">
            <el-icon class="button-icon"><RefreshRight /></el-icon>
            重新开始
          </el-button>
        </div>
      </Transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import dayjs from 'dayjs'
import {
  Calendar,
  Timer,
  Bell,
  AlarmClock,
  Warning,
  Check,
  RefreshLeft,
  RefreshRight,
  ArrowRight,
  SuccessFilled
} from '@element-plus/icons-vue'

// 步骤控制
const currentStep = ref('welcome')
const steps = ['welcome', 'dateInput', 'planDate', 'result', 'end']

// 表单数据
const birthDate = ref(null)
const today = ref(dayjs().format('YYYY-MM-DD'))
const daysInAdvance = ref(7)

// 计算下次生日日期和天数
const nextBirthdayDate = computed(() => {
  if (!birthDate.value || !today.value) return null
  const birth = dayjs(birthDate.value)
  const todayDate = dayjs(today.value)
  let nextBirthday = dayjs(todayDate).set('month', birth.month()).set('date', birth.date())
  if (nextBirthday.isBefore(todayDate)) {
    nextBirthday = nextBirthday.add(1, 'year')
  }
  return nextBirthday.format('YYYY-MM-DD')
})

const daysUntilNextBirthday = computed(() => {
  if (!nextBirthdayDate.value || !today.value) return null
  return dayjs(nextBirthdayDate.value).diff(dayjs(today.value), 'day')
})

// 计算计划日期
const planDate = computed(() => {
  if (!nextBirthdayDate.value || !daysInAdvance.value) return null
  let date = dayjs(nextBirthdayDate.value).subtract(daysInAdvance.value, 'day')
  const dayOfWeek = date.day()
  
  // 如果是工作日（周一到周五），调整到最近的周六
  if (dayOfWeek > 0 && dayOfWeek < 6) {
    if (dayOfWeek < 3) { // 周一、周二往前调
      date = date.subtract(dayOfWeek + 1, 'day')
    } else { // 周三、周四、周五往后调
      date = date.add(6 - dayOfWeek, 'day')
    }
  }
  return date.format('YYYY-MM-DD')
})

const isWorkday = computed(() => {
  if (!planDate.value) return false
  const dayOfWeek = dayjs(planDate.value).day()
  return dayOfWeek > 0 && dayOfWeek < 6
})

// 方法
const nextStep = () => {
  const currentIndex = steps.indexOf(currentStep.value)
  if (currentIndex < steps.length - 1) {
    currentStep.value = steps[currentIndex + 1]
  }
}

const resetPlanDate = () => {
  // 如果默认的提前天数大于可用天数，则设置为可用天数
  if (daysInAdvance.value > daysUntilNextBirthday.value) {
    daysInAdvance.value = daysUntilNextBirthday.value
  }
  currentStep.value = 'planDate'
}

const resetAll = () => {
  birthDate.value = null
  today.value = dayjs().format('YYYY-MM-DD')
  daysInAdvance.value = 7
  currentStep.value = 'welcome'
}

const buttonHover = ref(false)
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.background-decoration {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 10% 20%, var(--accent-color) 0%, transparent 20%),
    radial-gradient(circle at 90% 80%, var(--primary-color) 0%, transparent 20%);
  opacity: 0.1;
  z-index: -1;
}

.container {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 40px 20px;
}

.step-container {
  padding: 40px;
  margin: 20px;
  transition: all 0.3s ease;
}

/* 欢迎界面，此部分是mqh修改的 */
.welcome-container {
  position: relative;
  overflow: hidden;
  padding: 60px 40px;
  text-align: center;
  background: linear-gradient(135deg, rgba(255,255,255,0.9), rgba(255,255,255,0.7));
}

.welcome-content {
  position: relative;
  z-index: 2;
}

.welcome-title {
  font-size: 2.5em;
  background: linear-gradient(45deg, var(--primary-color), var(--accent-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 30px;
  animation: fadeInDown 0.8s ease-out;
}


.welcome-text {
  font-size: 1.3em;
  color: var(--text-color);
  margin: 30px 0;
  opacity: 0;
  animation: fadeIn 0.8s ease-out 0.3s forwards;
}


.welcome-button {
  margin-top: 40px;
  font-size: 1.2em;
  padding: 15px 40px !important;
  background: linear-gradient(45deg, var(--primary-color), var(--accent-color));
  border: none;
  opacity: 0;
  animation: fadeInUp 0.8s ease-out 0.6s forwards;
  transition: all 0.3s ease;
}

.button-hover {
  transform: translateY(-3px);
  box-shadow: 0 7px 20px rgba(37, 99, 235, 0.3);
}

.welcome-decoration {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 90% 10%, var(--accent-color) 0%, transparent 20%),
    radial-gradient(circle at 10% 90%, var(--primary-color) 0%, transparent 20%);
  opacity: 0.1;
  z-index: 1;
  animation: pulse 8s ease-in-out infinite;
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    opacity: 0.1;
  }
  50% {
    transform: scale(1.05);
    opacity: 0.15;
  }
}

/* end */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin: 30px 0;
}

.form-item {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 10px 0;
}

.date-picker {
  flex: 1;
}

.number-input {
  width: 120px;
}

.info-card, .result-card {
  background: rgba(255, 255, 255, 0.5);
  border-radius: 16px;
  padding: 20px;
  margin: 20px 0;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 10px 0;
}

.highlight {
  color: var(--primary-color);
  font-weight: 600;
}

.adjust-notice {
  color: #f59e0b;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 10px;
}

.button-icon {
  margin-right: 8px;
}

.success-icon {
  font-size: 48px;
  color: #10b981;
  margin-bottom: 20px;
}

.end-message {
  text-align: center;
  margin: 30px 0;
}

h1 {
  font-size: 2.5em;
  background: linear-gradient(to right, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 20px;
}

h2 {
  font-size: 2em;
  color: var(--text-color);
  margin-bottom: 30px;
}

@media (max-width: 768px) {
  .container {
    padding: 20px 10px;
  }

  .step-container {
    padding: 20px;
    margin: 10px;
  }

  h1 {
    font-size: 2em;
  }

  h2 {
    font-size: 1.5em;
  }

  .info-card, .result-card {
    padding: 15px;
  }

  /* mqh修改的 */
  .welcome-container {
    padding: 40px 20px;
  }
  
  .welcome-title {
    font-size: 2em;
  }
  
  .welcome-text {
    font-size: 1.1em;
  }
  
  .welcome-button {
    padding: 12px 30px !important;
  }
}

/* end */

.centered-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

</style>
