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
            <div class="welcome-buttons">
              <el-button type="primary" @click="nextStep" class="welcome-button" :class="{'button-hover': buttonHover}" 
                @mouseenter="buttonHover = true" 
                @mouseleave="buttonHover = false">
                <el-icon class="button-icon"><ArrowRight /></el-icon>
                开始规划
              </el-button>
              <el-button v-if="savedPlans.length > 0" @click="viewSavedPlans" class="welcome-button view-saved-button" 
                :class="{'button-hover': savedButtonHover}"
                @mouseenter="savedButtonHover = true"
                @mouseleave="savedButtonHover = false">
                <el-icon class="button-icon"><Calendar /></el-icon>
                查看已保存计划
                <span class="plan-count">({{ savedPlans.length }})</span>
              </el-button>
            </div>
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
            <div v-if="daysUntilNextBirthday !== null" class="result-card">
              <el-icon><Bell /></el-icon>
              距离下次生日还有 <span class="highlight">{{ daysUntilNextBirthday }}</span> 天
              <!-- mqh -->
              <div v-if=" isBirthDateBeforeToday" class="info-item adjust-notice">
                    <el-icon><Warning /></el-icon>
                    <span>出生晚于今天！系统将仅默认您选择的出生月日</span>
              </div>
              <div v-if="isBirthDateLeapYear" class="info-item adjust-notice">
                    <el-icon><Warning /></el-icon>
                    <span>出生日期是2月29日，系统将默认为您在下一个2月28日过生日</span>
              </div>
              <!-- end -->
            </div>
          </Transition>
          <el-button type="primary" @click="nextStep" :disabled="!birthDate || !today">
            <el-icon class="button-icon"><ArrowRight /></el-icon>
            继续
          </el-button>
        </div>
<!-- mqh修改的 -->
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
                <div class="info-item">
                  <el-icon><Calendar /></el-icon>
                  <span>计划日期：</span>
                  <span class="highlight">{{ planDate }}</span>
                </div>
                <div v-if="isWorkday" class="info-item adjust-notice">
                  <el-icon><Warning /></el-icon>
                  <span>由于是工作日，已调整为最近的周六</span>
                </div>
                <!-- 计划日期在生日之后的提示 -->
                <div v-if="isPlanDateAfterBirthday" class="info-item adjust-notice">
                  <el-icon><Warning /></el-icon>
                  <span>计划日期已经超过生日日期！</span>
                </div>
                <!-- 所有可能日期都已过期的提示 -->
                <div v-if="allPossibleDatesExpired" class="info-item adjust-notice">
                  <el-icon><Warning /></el-icon>
                  <span>所有可能的计划日期都已过期，建议立即开始计划！下次记得早点准备哦～</span>
                </div>
              </div>
            </Transition>
          </div>
          <el-button type="primary" @click="nextStep" :disabled="!daysInAdvance">
            <el-icon class="button-icon"><Check /></el-icon>
            确认
          </el-button>
        </div>
<!-- end -->
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
            
            <!-- 根据情况显示不同的计划日期信息 -->
            <div class="info-item">
              <el-icon><Calendar /></el-icon>
              <span>计划制定日期：</span>
              <template v-if="allPossibleDatesExpired">
                <span class="highlight warning">建议立即开始计划</span>
              </template>
              <span v-else class="highlight">{{ planDate }}</span>
            </div>
          </div>

          <!-- 根据情况显示不同的按钮组 -->
          <div class="button-group">
            <template v-if="allPossibleDatesExpired">
              <el-button type="danger" @click="startImmediatePlan">
                <el-icon class="button-icon"><Timer /></el-icon>
                立即开始计划
              </el-button>
              <el-button @click="resetPlanDate">
                <el-icon class="button-icon"><RefreshLeft /></el-icon>
                重新选择日期
              </el-button>
            </template>
            <template v-else>
              <el-button @click="resetPlanDate">
                <el-icon class="button-icon"><RefreshLeft /></el-icon>
                重新制定
              </el-button>
              <el-button type="primary" @click="savePlan">
                <el-icon class="button-icon"><Check /></el-icon>
                保存计划
              </el-button>
              <el-button type="success" @click="nextStep">
                <el-icon class="button-icon"><ArrowRight /></el-icon>
                完成
              </el-button>
            </template>
          </div>
        </div>

        <!-- 结束界面 -->
        <div v-else-if="currentStep === 'end'" class="step-container glass-card">
          <h2>计划已完成</h2>
          <div class="end-message">
            <el-icon class="success-icon"><SuccessFilled /></el-icon>
            <p>您的生日计划将在 <span class="highlight">{{ planDate }}</span> 制定</p>
          </div>
          
          <!-- 已保存的计划列表 -->
          <div class="saved-plans" v-if="savedPlans.length > 0">
            <h3>已保存的计划</h3>
            <div class="plans-list">
              <div v-for="plan in savedPlans" :key="plan.id" class="plan-item glass-card">
                <div class="plan-info">
                  <div class="plan-date">
                    <el-icon><Calendar /></el-icon>
                    <span>生日：{{ dayjs(plan.nextBirthday).format('YYYY年MM月DD日') }}</span>
                  </div>
                  <div class="plan-date">
                    <el-icon><AlarmClock /></el-icon>
                    <span>计划日期：{{ dayjs(plan.planDate).format('YYYY年MM月DD日') }}</span>
                  </div>
                  <div class="plan-created">
                    <el-icon><Timer /></el-icon>
                    <span>创建时间：{{ plan.createdAt }}</span>
                  </div>
                </div>
                <div class="plan-actions">
                  <el-button type="primary" size="small" @click="loadPlan(plan)">
                    <el-icon><RefreshRight /></el-icon>
                    加载
                  </el-button>
                  <el-button type="danger" size="small" @click="deletePlan(plan.id)">
                    <el-icon><Delete /></el-icon>
                    删除
                  </el-button>
                </div>
              </div>
            </div>
          </div>
          
          <el-button type="primary" @click="resetAll" class="new-plan-button">
            <el-icon class="button-icon"><RefreshRight /></el-icon>
            制定新计划
          </el-button>
        </div>

        <!-- 查看已保存计划界面 -->
        <div v-else-if="currentStep === 'savedPlans'" class="step-container glass-card">
          <h2>已保存的计划</h2>
          <div class="saved-plans-content">
            <div class="saved-plans-list">
              <div v-for="plan in savedPlans" :key="plan.id" class="saved-plan-item glass-card">
                <div class="plan-info">
                  <div class="plan-date">
                    <el-icon><Calendar /></el-icon>
                    <span>生日：{{ dayjs(plan.nextBirthday).format('YYYY年MM月DD日') }}</span>
                  </div>
                  <div class="plan-date">
                    <el-icon><AlarmClock /></el-icon>
                    <span>计划日期：{{ dayjs(plan.planDate).format('YYYY年MM月DD日') }}</span>
                  </div>
                  <div class="plan-created">
                    <el-icon><Timer /></el-icon>
                    <span>创建时间：{{ plan.createdAt }}</span>
                  </div>
                </div>
                <div class="plan-actions">
                  <el-button type="primary" size="small" @click="loadPlan(plan)">
                    <el-icon><RefreshRight /></el-icon>
                    加载
                  </el-button>
                  <el-button type="danger" size="small" @click="deletePlan(plan.id)">
                    <el-icon><Delete /></el-icon>
                    删除
                  </el-button>
                </div>
              </div>
            </div>
            <el-button type="primary" @click="backToWelcome" class="back-to-welcome-button">
              <el-icon class="button-icon"><ArrowLeft /></el-icon>
              返回欢迎页面
            </el-button>
          </div>
        </div>
      </Transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
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
  SuccessFilled,
  Delete,
  ArrowLeft
} from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 步骤控制
const currentStep = ref('welcome')
const steps = ['welcome', 'dateInput', 'planDate', 'result', 'end', 'savedPlans']

// 表单数据
const birthDate = ref(null)
const today = ref(dayjs().format('YYYY-MM-DD'))
const daysInAdvance = ref(7)

// 保存的计划列表
const savedPlans = ref([])

// 从本地存储加载数据
const loadSavedPlans = () => {
  const plans = localStorage.getItem('birthdayPlans')
  if (plans) {
    savedPlans.value = JSON.parse(plans)
  }
}

// 初始化时加载数据
loadSavedPlans()

// 保存当前计划
const savePlan = () => {
  const newPlan = {
    id: Date.now(), // 使用时间戳作为唯一ID
    birthDate: birthDate.value,
    nextBirthday: nextBirthdayDate.value,
    planDate: planDate.value,
    daysInAdvance: daysInAdvance.value,
    createdAt: dayjs().format('YYYY-MM-DD HH:mm:ss')
  }
  
  savedPlans.value.push(newPlan)
  localStorage.setItem('birthdayPlans', JSON.stringify(savedPlans.value))
  
  ElMessage({
    message: '计划已保存',
    type: 'success'
  })
}

// 删除保存的计划
const deletePlan = (planId) => {
  savedPlans.value = savedPlans.value.filter(plan => plan.id !== planId)
  localStorage.setItem('birthdayPlans', JSON.stringify(savedPlans.value))
  
  ElMessage({
    message: '计划已删除',
    type: 'success'
  })
}

// 加载已保存的计划
const loadPlan = (plan) => {
  birthDate.value = plan.birthDate
  daysInAdvance.value = plan.daysInAdvance
  currentStep.value = 'result'
}

// 监听计划列表变化
watch(savedPlans, (newPlans) => {
  localStorage.setItem('birthdayPlans', JSON.stringify(newPlans))
}, { deep: true })

// 计算下次生日日期和天数
const nextBirthdayDate = computed(() => {
  if (!birthDate.value || !today.value) return null
  const birth = dayjs(birthDate.value)
  const todayDate = dayjs(today.value)
  let nextBirthday = dayjs(todayDate)
  .set('month', birth.month())
  .set('date',  isLeapBirthday ? 28 : birth.date()) // 2月29日调整为2月28日
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

// 修改isWorkday的计算逻辑
const originalPlanDate = computed(() => {
  if (!nextBirthdayDate.value || !daysInAdvance.value) return null
  return dayjs(nextBirthdayDate.value).subtract(daysInAdvance.value, 'day')
})

const isWorkday = computed(() => {
  if (!originalPlanDate.value) return false
  const dayOfWeek = originalPlanDate.value.day()
  return dayOfWeek > 0 && dayOfWeek < 6
})
// 检查是否是2月29日
const isLeapBirthday = birth.month() === 1 && birth.date() === 29 
// 检查计划日期是否在生日之后
const isPlanDateAfterBirthday = computed(() => {
  if (!planDate.value || !nextBirthdayDate.value) return false
  return dayjs(planDate.value).isAfter(dayjs(nextBirthdayDate.value))
})
// 检查出生日期是否在当前日期之前
const isBirthDateBeforeToday = computed(() => {
  if (!birthDate.value) return false
  return dayjs(birthDate.value).isAfter(dayjs(today.value))
})
// 检查出生日期是否是2月29日
const isBirthDateLeapYear = computed(() => {
  if (!birthDate.value) return false
  return dayjs(birthDate.value).isLeapYear()
})
// 检查所有可能的计划日期是否都已过期
const allPossibleDatesExpired = computed(() => {
  if (!originalPlanDate.value || !nextBirthdayDate.value) return false
  const originalDate = originalPlanDate.value
  const dayOfWeek = originalDate.day()
  
  // 如果是工作日，检查前后的周六
  if (dayOfWeek > 0 && dayOfWeek < 6) {
    let previousSaturday = originalDate.subtract(dayOfWeek + 1, 'day')
    let nextSaturday = originalDate.add(6 - dayOfWeek, 'day')
    // 检查这两个周六是否都在生日之后或者都已经过期
    return (previousSaturday.isAfter(dayjs(nextBirthdayDate.value)) || 
            previousSaturday.isBefore(dayjs(today.value))) &&
           (nextSaturday.isAfter(dayjs(nextBirthdayDate.value)) || 
            nextSaturday.isBefore(dayjs(today.value)))
  }
  return false
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
const savedButtonHover = ref(false)

// 查看已保存计划
const viewSavedPlans = () => {
  currentStep.value = 'savedPlans'
}

// 返回欢迎页面
const backToWelcome = () => {
  currentStep.value = 'welcome'
}

// 添加立即开始计划的方法
const startImmediatePlan = () => {
  // 将计划日期设置为今天
  const newPlan = {
    id: Date.now(),
    birthDate: birthDate.value,
    nextBirthday: nextBirthdayDate.value,
    planDate: dayjs().format('YYYY-MM-DD'), // 使用今天作为计划日期
    daysInAdvance: daysUntilNextBirthday.value, // 剩余天数作为准备时间
    createdAt: dayjs().format('YYYY-MM-DD HH:mm:ss'),
    isImmediate: true // 标记为立即计划
  }
  
  savedPlans.value.push(newPlan)
  localStorage.setItem('birthdayPlans', JSON.stringify(savedPlans.value))
  
  ElMessage({
    message: '已创建立即计划，请尽快开始准备！',
    type: 'success'
  })
  
  nextStep()
}
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
  padding: 40px 20px;
  margin: 40px 20px;
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

.welcome-buttons {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
  margin-top: 40px;
}

.welcome-button {
  font-size: 1.2em;
  padding: 15px 40px !important;
  min-width: 200px;
}

.view-saved-button {
  background: var(--accent-color) !important;
  color: white !important;
  border: none !important;
}

.view-saved-button:hover {
  background: var(--primary-color) !important;
}

.plan-count {
  margin-left: 8px;
  font-size: 0.9em;
  opacity: 0.9;
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
  margin: 20px 0;
}

.form-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.date-picker {
  flex: 1;
}

.number-input {
  width: 120px;
}

.info-card, .result-card {
  background: rgba(255, 255, 255, 0.5);
  border-radius: 15px;
  margin: 0 0 10px 0;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 20px 0;
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

/* 已保存的计划列表样式 */
.saved-plans {
  margin: 30px 0;
  width: 100%;
}

.saved-plans h3 {
  margin-bottom: 20px;
  color: var(--text-color);
  font-size: 1.5em;
}

.plans-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
  max-height: 400px;
  overflow-y: auto;
  padding: 10px;
}

.plan-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 12px;
  transition: all 0.3s ease;
}

.plan-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
}

.plan-info {
  flex: 1;
}

.plan-date, .plan-created {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 5px 0;
  color: var(--text-color);
}

.plan-actions {
  display: flex;
  gap: 10px;
}

.new-plan-button {
  margin-top: 20px;
}

/* 滚动条样式 */
.plans-list::-webkit-scrollbar {
  width: 6px;
}

.plans-list::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 3px;
}

.plans-list::-webkit-scrollbar-thumb {
  background: var(--primary-color);
  border-radius: 3px;
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

  .plan-item {
    flex-direction: column;
    align-items: stretch;
  }
  
  .plan-actions {
    margin-top: 15px;
    justify-content: flex-end;
  }
  
  .plans-list {
    max-height: 300px;
  }

  .welcome-buttons {
    gap: 10px;
    margin-top: 30px;
  }

  .welcome-button {
    font-size: 1.1em;
    padding: 12px 30px !important;
    min-width: 180px;
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

.saved-plans-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.saved-plans-list {
  margin-bottom: 20px;
  width: 100%;
}

.saved-plan-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 12px;
  transition: all 0.3s ease;
  margin-top: 20px;
}

.saved-plan-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
}

.back-to-welcome-button {
  margin-top: 20px;
}

.result-text {
  display: flex;
  align-items: center;
  color: var(--text-color);
  margin-bottom: 15px;
  background: rgba(255, 255, 255, 0.5);
}

.expired-notice {
  margin-top: 10px;
  padding: 8px 12px;
  background: rgba(245, 108, 108, 0.1);
  border-radius: 8px;
  color: #f56c6c;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9em;
}

.warning {
  color: #f56c6c !important;
  font-weight: bold;
}

.button-group {
  margin-top: 30px;
  display: flex;
  gap: 15px;
  justify-content: center;
}

</style>
