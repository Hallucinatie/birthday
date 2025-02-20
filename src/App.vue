<template>
  <div class="container">
    <!-- 欢迎界面 -->
    <div v-if="currentStep === 'welcome'" class="step-container">
      <h1>欢迎使用生日聚会计划便签</h1>
      <el-button type="primary" @click="nextStep">按任意键继续</el-button>
    </div>

    <!-- 日期输入界面 -->
    <div v-if="currentStep === 'dateInput'" class="step-container">
      <h2>请输入日期信息</h2>
      <div class="form-item">
        <span>出生日期：</span>
        <el-date-picker v-model="birthDate" type="date" placeholder="选择出生日期" />
      </div>
      <div class="form-item">
        <span>今天日期：</span>
        <el-date-picker v-model="today" type="date" placeholder="选择今天日期" />
      </div>
      <div v-if="daysUntilNextBirthday !== null" class="result-text">
        距离下次生日还有 {{ daysUntilNextBirthday }} 天
      </div>
      <el-button type="primary" @click="nextStep" :disabled="!birthDate || !today">
        继续
      </el-button>
    </div>

    <!-- 计划制定界面 -->
    <div v-if="currentStep === 'planDate'" class="step-container">
      <h2>生日聚会计划制定</h2>
      <div class="info-text">
        <p>下次生日日期：{{ nextBirthdayDate }}</p>
        <p>距离今天：{{ daysUntilNextBirthday }} 天</p>
      </div>
      <div class="form-item">
        <span>提前天数：</span>
        <el-input-number v-model="daysInAdvance" :min="1" :max="365" />
      </div>
      <div v-if="planDate" class="result-text">
        <p>计划日期：{{ planDate }}</p>
        <p v-if="isWorkday">由于是工作日，已调整为最近的周六</p>
      </div>
      <el-button type="primary" @click="nextStep" :disabled="!daysInAdvance">
        确认
      </el-button>
    </div>

    <!-- 结果显示界面 -->
    <div v-if="currentStep === 'result'" class="step-container">
      <h2>计划结果</h2>
      <div class="result-info">
        <p>下次生日日期：{{ nextBirthdayDate }}</p>
        <p>距离今天天数：{{ daysUntilNextBirthday }} 天</p>
        <p>提前准备天数：{{ daysInAdvance }} 天</p>
        <p>计划制定日期：{{ planDate }}</p>
      </div>
      <div class="button-group">
        <el-button type="primary" @click="resetPlanDate">重新制定</el-button>
        <el-button @click="nextStep">确认完成</el-button>
      </div>
    </div>

    <!-- 结束界面 -->
    <div v-if="currentStep === 'end'" class="step-container">
      <h2>计划已完成</h2>
      <p>您的生日计划将在 {{ planDate }} 制定</p>
      <el-button type="primary" @click="resetAll">重新开始</el-button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import dayjs from 'dayjs'

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
  currentStep.value = 'planDate'
}

const resetAll = () => {
  birthDate.value = null
  today.value = dayjs().format('YYYY-MM-DD')
  daysInAdvance.value = 7
  currentStep.value = 'welcome'
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.step-container {
  text-align: center;
  padding: 20px;
}

.form-item {
  margin: 20px 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.result-text {
  margin: 20px 0;
  color: #409EFF;
}

.result-info {
  text-align: left;
  margin: 20px auto;
  max-width: 400px;
  padding: 20px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
}

.button-group {
  margin-top: 20px;
  display: flex;
  gap: 10px;
  justify-content: center;
}

h1, h2 {
  color: #303133;
  margin-bottom: 20px;
}

.info-text {
  margin: 20px 0;
  color: #606266;
}
</style>
