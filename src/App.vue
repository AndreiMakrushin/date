<script setup>
import { ref, watch } from 'vue'
const language = ref('ru')
const currentDate = ref(new Date())
const currentMonth = ref(formatMonth(currentDate.value, language.value))
const monthDays = ref(generateMonthDays(currentDate.value, language.value))

function formatMonth(date, language) {
  const months = {
    en: [
      'January',
      'February',
      'March',
      'April',
      'May',
      'June',
      'July',
      'August',
      'September',
      'October',
      'November',
      'December'
    ],
    ru: [
      'Январь',
      'Февраль',
      'Март',
      'Апрель',
      'Май',
      'Июнь',
      'Июль',
      'Август',
      'Сентябрь',
      'Октябрь',
      'Ноябрь',
      'Декабрь'
    ]
  }
  return `${months[language][date.getMonth()]} ${date.getFullYear()}`
}
const weekdays = {
  en: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
  ru: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
}
function getPrevMonthDays(date) {
  const prevMonth = new Date(date.getFullYear(), date.getMonth() - 1, 1)
  const lastDayOfPrevMonth = new Date(date.getFullYear(), date.getMonth(), 0)
  const daysToAdd = lastDayOfPrevMonth.getDay()

  const prevMonthDays = []
  const day = new Date(prevMonth)
  for (let i = daysToAdd - 1; i >= 0; i--) {
    day.setDate(day.getDate() - i)
    prevMonthDays.push({
      date: day,
      dayNumber: day.getDate(),
      isCurrentMonth: false,
      isPrevMonth: true,
      isNextMonth: false
    })
  }

  return prevMonthDays
}
function getNextMonthDays(date) {
  const nextMonth = new Date(date.getFullYear(), date.getMonth() + 1, 1)
  const daysToAdd = 7 - nextMonth.getDay()

  const nextMonthDays = []
  for (let i = 1; i <= daysToAdd; i++) {
    nextMonthDays.push({
      date: new Date(nextMonth),
      dayNumber: i,
      isCurrentMonth: false,
      isPrevMonth: false,
      isNextMonth: true
    })
  }
  return nextMonthDays
}
function generateMonthDays(date) {
  const firstDayOfMonth = new Date(date.getFullYear(), date.getMonth(), 1)
  const lastDayOfMonth = new Date(date.getFullYear(), date.getMonth() + 1, 0)
  const prevMonthDays = getPrevMonthDays(firstDayOfMonth)
  const nextMonthDays = getNextMonthDays(lastDayOfMonth)

  const monthDays = []
  let day = new Date(firstDayOfMonth)
  while (day <= lastDayOfMonth) {
    monthDays.push({
      date: new Date(day),
      dayNumber: day.getDate(),
      isCurrentMonth: true,
      isPrevMonth: false,
      isNextMonth: false
    })
    day.setDate(day.getDate() + 1)
  }

  return [...prevMonthDays, ...monthDays, ...nextMonthDays]
}
const formatDate = (date) => {
  const year = date.getFullYear()
  const month = (date.getMonth() + 1).toString().padStart(2, '0')
  const day = date.getDate().toString().padStart(2, '0')
  return `${year}-${month}-${day}`
}
const prevMonth = () => {
  currentDate.value = new Date(currentDate.value.getFullYear(), currentDate.value.getMonth() - 1, 1)
  currentMonth.value = formatMonth(currentDate.value, language.value)
  monthDays.value = generateMonthDays(currentDate.value)
}

const nextMonth = () => {
  currentDate.value = new Date(currentDate.value.getFullYear(), currentDate.value.getMonth() + 1, 1)
  currentMonth.value = formatMonth(currentDate.value, language.value)
  monthDays.value = generateMonthDays(currentDate.value)
}

const selectedDate = ref(formatDate(new Date()))

watch(selectedDate, (newVal) => {
  currentDate.value = new Date(newVal)
  currentMonth.value = formatMonth(currentDate.value, language.value)
  monthDays.value = generateMonthDays(currentDate.value)
})

const onDateChange = (event) => {
  selectedDate.value = event.target.value
}

const selectDate = (date) => {
  selectedDate.value = formatDate(date)
}
</script>

<template>
  <div class="calendar">
    <div class="calendar-header">
      <input type="date" v-model="selectedDate" @change="onDateChange" />
      <button @click="prevMonth">&lt;</button>
      <span>{{ currentMonth }}</span>
      <button @click="nextMonth">&gt;</button>
    </div>
    <div class="calendar-body">
      <div class="calendar-weekdays">
        <span v-for="day in weekdays[language]" :key="day">{{ day }}</span>
      </div>
      <div class="calendar-days">
        <span
          v-for="(day, index) in monthDays"
          :key="`day-${index}-${day.date.toISOString()}`"
          :class="{
            'current-day':
              day.date.getMonth() === new Date().getMonth() &&
              day.date.getDate() === new Date().getDate(),
            'selected-day': selectedDate === formatDate(day.date),
            'prev-month': day.isPrevMonth,
            'next-month': day.isNextMonth
          }"
          @click="selectDate(day.date)"
          >{{ day.dayNumber }}</span
        >
      </div>
    </div>
  </div>
  <div class="language-switcher">
    <button @click="(language = 'en'), (currentMonth = formatMonth(currentDate, language))">
      English
    </button>
    <button @click="(language = 'ru'), (currentMonth = formatMonth(currentDate, language))">
      Русский
    </button>
  </div>
</template>

<style scoped>
.language-switcher {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}

.language-switcher button {
  margin: 0 5px;
}
.calendar {
  width: 500px;
  font-family: Arial, sans-serif;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.calendar-weekdays {
  width: 100%;
  display: flex;
  justify-content: space-between;
}
.selected-day {
  background: rgba(0, 255, 85, 0.267);
}
.calendar-weekdays span {
  width: 50px;
  text-align: center;
}

.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
}

.calendar-days span {
  display: block;
  text-align: center;
  padding: 5px;
}

.current-day {
  background-color: lightblue;
}

.prev-month,
.next-month {
  color: white;
}
</style>
