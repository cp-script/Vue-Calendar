<template>
  <div class="calendar">
    <div class="header">
      <button @click="goToPreviousMonth">&lt;</button>
      <div class="datepicker">
        <VueDatePicker v-model="month" month-picker />
      </div>
      <button @click="goToNextMonth">&gt;</button>
    </div>
    <div class="days-of-week">
      <div v-for="day in daysOfWeek" :key="day" class="day-of-week">{{ day }}</div>
    </div>
    <div class="days">
      <div v-for="day in daysInMonth" :key="day.date" class="day" :class="{ 'blank': day.blank }">
        <div class="day-panel"
          :class="{ 'active': day.date === active, 'semi-active': Math.abs(day.date - active) === 1 }">
          <button @click="showMore(0)">&times;</button>
          <div class="week">{{ daysOfWeek[new Date(month.year, month.month, day.date).getDay()] }}</div>
          <div class="date">{{ day.date }}</div>
          <div v-if="day.events" v-for="event in day.events.slice(0, 5)" :key="event.id" class="event"
            :class="`${event.status}`">{{ event.title }}</div>
          <div v-if="day.date === active" v-for="event in day.events.slice(5)" :key="event.id" class="event"
            :class="`${event.status}`">{{ event.title }}</div>
          <div v-if="day.events && day.events.length > 5" class="show-more">
            <span @click="showMore(day.date)">
              {{ day.events.length - 5 }} more...
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue';

const month = ref({
  month: new Date().getMonth(),
  year: new Date().getFullYear()
});
const daysOfWeek = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
const active = ref(0);
const events = [
  { id: 1, title: 'Event 1', start: '2024-02-10', end: '2024-02-12' },
  { id: 2, title: 'Event 2', start: '2024-02-13', end: '2024-02-15' },
  { id: 3, title: 'Event 3', start: '2024-02-11', end: '2024-02-18' },
  { id: 4, title: 'Event 4', start: '2024-02-05', end: '2024-02-05' },
  { id: 5, title: 'Event 5', start: '2024-02-06', end: '2024-02-07' },
  { id: 6, title: 'Event 6', start: '2024-02-06', end: '2024-02-07' },
  { id: 7, title: 'Event 7', start: '2024-02-06', end: '2024-02-07' },
  { id: 8, title: 'Event 8', start: '2024-02-12', end: '2024-02-13' },
  { id: 9, title: 'Event 9', start: '2024-02-12', end: '2024-02-13' },
  { id: 10, title: 'Event 10', start: '2024-02-27', end: '2024-03-01' },
  { id: 11, title: 'Event 11', start: '2024-02-27', end: '2024-03-03' },
  { id: 12, title: 'Event 12', start: '2024-02-27', end: '2024-03-03' },
  { id: 20, title: 'Event 20', start: '2024-03-12', end: '2024-03-13' },
  { id: 21, title: 'Event 21', start: '2024-02-06', end: '2024-02-07' },
  { id: 22, title: 'Event 22', start: '2024-02-05', end: '2024-02-06' },
  { id: 23, title: 'Event 23', start: '2024-02-06', end: '2024-02-07' },
  { id: 24, title: 'Event 24', start: '2024-02-06', end: '2024-02-07' },
  // Add more events as needed
];

const goToPreviousMonth = () => {
  let currentDate = new Date(month.value.year, month.value.month, 1);
  currentDate.setMonth(currentDate.getMonth() - 1);

  month.value = {
    month: currentDate.getMonth(),
    year: currentDate.getFullYear()
  };
}

const goToNextMonth = () => {
  let currentDate = new Date(month.value.year, month.value.month, 1);
  currentDate.setMonth(currentDate.getMonth() + 1);

  month.value = {
    month: currentDate.getMonth(),
    year: currentDate.getFullYear()
  };
}

const getEventsForDate = (date) => {
  let list = [];
  const currentDate = new Date(new Date(date).toLocaleDateString());
  const current = currentDate.getTime();

  const prev = current - 24 * 60 * 60 * 1000;
  const next = current + 24 * 60 * 60 * 1000;
  const last = new Date(new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0).toLocaleDateString()).getTime();

  events.map(event => {
    const start = new Date(new Date(event.start).toLocaleDateString()).getTime();
    const end = new Date(new Date(event.end).toLocaleDateString()).getTime();

    let status = [];

    if (start <= current && current <= end) {
      const idx = list.length;
      const prevList = events.filter(item => new Date(new Date(item.start).toLocaleDateString()).getTime() <= prev && prev <= new Date(new Date(item.end).toLocaleDateString()).getTime());
      const nextList = events.filter(item => new Date(new Date(item.start).toLocaleDateString()).getTime() <= next && next <= new Date(new Date(item.end).toLocaleDateString()).getTime());

      if (start === current || date.getDay() === 0 || date.getDate() === 1 || !prevList[idx] || prevList[idx] && prevList[idx].id !== event.id) status.push("start");
      if (end === current || date.getDay() === 6 || current === last || !nextList[idx] || nextList[idx] && nextList[idx].id !== event.id) status.push("end");

      list.push({ ...event, status: status.join(" ") });
    }
  });
  return list;
};

const daysInMonth = computed(() => {
  let currentDate = new Date(month.value.year, month.value.month, 1);
  const firstWeek = new Date(currentDate.getFullYear(), currentDate.getMonth(), 1).getDay();
  const days = new Array(firstWeek).fill().map(() => ({ date: null, event: null, blank: true }));

  const lastDay = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0);
  const lastWeek = 7 - (lastDay.getDay() + 1) % 7;
  const lastDays = new Array(lastWeek).fill().map(() => ({ date: null, event: null, blank: true }));

  for (let i = 1; i <= lastDay.getDate(); i++) {
    const date = new Date(currentDate.getFullYear(), currentDate.getMonth(), i);
    const dayEvents = getEventsForDate(date);
    days.push({ date: i, events: dayEvents, blank: false });
  }
  return days.concat(lastDays);
});

const showMore = (day) => {
  active.value = day;
}
</script>

<style scoped>
.calendar {
  font-family: Arial, sans-serif;
}

.header {
  display: flex;
  align-items: center;
  padding: 10px;
  background-color: #f2f2f2;
}

.header button {
  background-color: transparent;
  border: 1px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  font-size: 20px;
  color: #666;
}

.datepicker {
  display: flex;
}

.days-of-week {
  display: flex;
  background-color: #f2f2f2;
}

.day-of-week {
  flex: 1;
  text-align: center;
  padding: 10px;
  font-weight: bold;
  color: #666;
}

.days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}

.day {
  position: relative;
  text-align: center;
  width: 150px;
  height: 150px;
  background-color: #fff;
}

.day:before {
  content: '';
  display: block;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  border: 0.5px solid #eee;
  z-index: 0;
}

.day::-webkit-scrollbar {
  display: none;
}

.day.blank {
  background-color: #d7d7d7;
}

.day:last-child {
  border-right: 1px solid #eeeeee;
}

.today {
  background-color: #f9f9f9;
}

.date {
  font-size: 14px;
  color: #666;
}

.event {
  margin-top: 2px;
  /* width: calc(100% + 10px); */
  height: 18px;
  color: transparent;
  background-color: green;
  /* margin-left: -15px; */
  font-size: 12px;
  position: relative;
  z-index: 1;
  /* left: 2px; */
}

.event.end {
  margin-right: 4px;
  border-top-right-radius: 5px;
  border-bottom-right-radius: 5px;
}

.event.start {
  left: 2px;
  text-align: left;
  padding-left: 5px;
  color: white;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
}

.show-more {
  color: black;
  text-align: left;
  font-size: 12px;
  padding: 5px;
}

.show-more span {
  cursor: pointer;
}

.day-panel {
  position: relative;
  z-index: 2;
  background-color: white;
}

.day-panel.active {
  position: relative;
  z-index: 3;
  height: fit-content;
  min-height: 120%;
  padding-bottom: 10px;
  border: 1px solid #eee;
  border-radius: 5px;
}

.day-panel.active .show-more {
  display: none;
}

.day-panel.active .event {
  color: white;
  margin-right: 4px;
  border-radius: 5px;
  left: 2px;
  text-align: left;
  padding-left: 5px;
}

.day-panel button {
  display: none;
  font-size: large;
}

.day-panel.active button {
  position: absolute;
  display: block;
  border: none;
  background-color: transparent;
  cursor: pointer;
  right: 5px;
  top: 5px;
}

.day-panel.semi-active .event {
  color: white;
  margin-right: 4px;
  border-radius: 5px;
  left: 2px;
  text-align: left;
  padding-left: 5px;
}

.week {
  display: none;
}

.day-panel.active .week {
  display: block;
  color: #666;
  font-size: 16px;
  font-weight: bold;
}
</style>