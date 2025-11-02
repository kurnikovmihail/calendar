<template>
  <div
    class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4"
  >
    <div class="w-full max-w-md">
      <h1 class="text-2xl font-bold text-center mb-4">
        Календарь Курникова Михаила
      </h1>

      <div class="mb-6 flex flex-col items-center">
        <my-calendar
          :value="initialDate"
          locale="en" 
          @date-selected="handleDateSelect"
        />
        <p class="mt-2 text-sm text-gray-600">
          Выбрано: {{ selectedDate }}
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
// ипорт компонента
import MyCalendar from './components/MyCalendar.vue';

function getTodayDateString() {
  const d = new Date();
  const year = d.getFullYear();
  // месяц +1 и падстарт, чтобы было 09, а не 9
  const month = (d.getMonth() + 1).toString().padStart(2, '0');
  const day = d.getDate().toString().padStart(2, '0');
  
  return `${year}-${month}-${day}`;
}

// начальная дата. можно и null
const initialDate = ref(getTodayDateString());
// тут храним то что выбрали
const selectedDate = ref(initialDate.value);

const handleDateSelect = (date) => {
  selectedDate.value = date;
};

</script>