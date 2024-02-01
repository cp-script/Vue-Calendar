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
        <span class="date">{{ day.date }}</span>
        <div v-for="event in day.events" :key="event.id" class="event" :class="`${event.status}`">{{ event.title
        }}</div>
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
const events = [
  { id: 1, title: 'Event 1', start: '2024-02-10', end: '2024-02-12' },
  { id: 2, title: 'Event 2', start: '2024-02-13', end: '2024-02-15' },
  { id: 3, title: 'Event 3', start: '2024-02-11', end: '2024-02-18' },
  { id: 4, title: 'Event 4', start: '2024-02-05', end: '2024-02-05' },
  { id: 6, title: 'Event 5', start: '2024-02-06', end: '2024-02-07' },
  { id: 8, title: 'Event 6', start: '2024-02-06', end: '2024-02-07' },
  { id: 9, title: 'Event 7', start: '2024-02-06', end: '2024-02-07' },
  { id: 10, title: 'Event 8', start: '2024-02-12', end: '2024-02-13' },
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

const getEventsForDate = (date, listIds) => {
  let newListIds = new Array(listIds.length).fill(-1);
  const list = [];
  const formattedDate = new Date(date).toLocaleDateString();
  const current = new Date(formattedDate).getTime();
  events.map(event => {
    const start = new Date(new Date(event.start).toLocaleDateString()).getTime();
    const end = new Date(new Date(event.end).toLocaleDateString()).getTime();
    if (start === current) {
      if (end === current || date.getDay() === 6) {
        list.push({ ...event, status: "one" });
        newListIds[event.id - 1] = list.length - 1;
      } else {
        list.push({ ...event, status: "start" });
        newListIds[event.id - 1] = list.length - 1;
      }
    } else if (end === current) {
      if (date.getDay() === 0) {
        list.push({ ...event, status: "one" });
        newListIds[event.id - 1] = list.length - 1;
      } else {
        list.push({ ...event, title: "" });
        newListIds[event.id - 1] = list.length - 1;
      }
    } else if (start < current && current < end) {
      if (date.getDay() === 0) {
        list.push({ ...event, status: "start" });
        newListIds[event.id - 1] = list.length - 1;
      } else if (listIds[event.id - 1] === list.length) {
        if (date.getDay() === 6) {
          list.push({ ...event, title: "" });
          newListIds[event.id - 1] = list.length - 1;
        } else {
          list.push({ ...event, title: "" });
          newListIds[event.id - 1] = list.length - 1;
        }
      } else {
        if (date.getDay() === 6) {
          list.push({ ...event, status: "one" });
          newListIds[event.id - 1] = list.length - 1;
        } else {
          list.push({ ...event, status: "start" });
          newListIds[event.id - 1] = list.length - 1;
        }
      }
    }
  });
  if (Math.max(...newListIds) >= 0) {
  }
  return { list, listIds: newListIds };
}

const daysInMonth = computed(() => {
  let currentDate = new Date(month.value.year, month.value.month, 1);
  const maxId = events.reduce((max, event) => event.id > max ? event.id : max, 0);
  let listIds = new Array(maxId).fill(-1);

  const firstWeek = new Date(currentDate.getFullYear(), currentDate.getMonth(), 1).getDay();
  const days = new Array(firstWeek).fill().map(() => ({ date: null, event: null, blank: true }));

  const lastDay = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0);
  const lastWeek = 7 - (lastDay.getDay() + 1) % 7;
  const lastDays = new Array(lastWeek).fill().map(() => ({ date: null, event: null, blank: true }));

  for (let i = 1; i <= lastDay.getDate(); i++) {
    const date = new Date(currentDate.getFullYear(), currentDate.getMonth(), i);
    const dayEvents = getEventsForDate(date, listIds);
    listIds = dayEvents.listIds;
    days.push({ date: i, events: dayEvents.list, blank: false });
  }
  return days.concat(lastDays);
});
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
  /* Increase the padding to make day areas larger */
  border: 0.5px solid #eeeeee;
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
  width: calc(100% + 10px);
  margin-left: -10px;
  height: 24px;
  background-color: green;
  border-radius: 0px;
  margin-left: -15px;
  border-radius: 0px 5px 5px 0px;
}

.event.one {
  text-align: left;
  width: calc(100% - 5px);
  margin-left: 0px;
  padding-left: 10px;
  color: white;
  border-radius: 5px;
}

.event.start {
  text-align: left;
  margin-left: 0px;
  padding-left: 10px;
  color: white;
  border-radius: 5px 0px 0px 5px;
}
</style>