<template>
  <div class="my-calendar-comp bg-white shadow-lg rounded-lg max-w-xs p-4">
    <div class="flex justify-between items-center mb-3">
      <button
        type="button"
        class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-gray-100 transition-colors"
        @click="changeMonth(-1)"
      >
        <span class="text-gray-600 font-bold">&lt;</span>
      </button>

      <div class="flex items-center space-x-2">
        <div class="font-bold text-lg capitalize">
          {{ monthLabel }}
        </div>
        <button
          @click="toggleLang"
          class="text-xs uppercase font-medium bg-gray-200 hover:bg-gray-300 rounded px-2 py-0.5"
        >
          {{ currentLocale }}
        </button>
      </div>

      <button
        type="button"
        class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-gray-100 transition-colors"
        @click="changeMonth(1)"
      >
        <span class="text-gray-600 font-bold">&gt;</span>
      </button>
    </div>

    <div class="grid grid-cols-7 gap-1 text-center text-gray-500 text-sm mb-2">
      <div v-for="day in localeData.weekdays" :key="day" class="uppercase">
        {{ day }}
      </div>
    </div>

    <div class="grid grid-cols-7 gap-1">
      <div
        v-for="(day, index) in daysGrid"
        :key="index"
        class="w-10 h-10 flex items-center justify-center rounded-full cursor-pointer transition-colors"
        :class="getDayClasses(day)"
        @click="selectDay(day)"
      >
        {{ day.dayNum }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

// пропсы и емиты
const props = defineProps({
  // дата YYYY-MM-DD
  value: {
    type: String,
    default: null,
  },
  // стартовая локаль
  locale: {
    type: String,
    default: "ru",
  },
});

const emit = defineEmits(["date-selected"]);

// типа парсер строки в дату (базовый, без замудрения)
function parseDate(dateStr) {
  if (!dateStr) return new Date();
  const parts = dateStr.split("-").map(Number);
  return new Date(parts[0], parts[1] - 1, parts[2]);
}

// обратно в строчный формат
function fomatDate(date) {
  const y = date.getFullYear();
  const m = (date.getMonth() + 1).toString().padStart(2, "0");
  const d = date.getDate().toString().padStart(2, "0");
  return `${y}-${m}-${d}`;
}

// какой месяц показываем
const displayMonth = ref(parseDate(props.value));
// а это который выбрали
const selectedDay = ref(props.value);
const today = new Date(); //егодня

// сравниваем даты
function isSame(d1, d2) {
  if (!d1 || !d2) return false;
  const date1 = new Date(d1);
  const date2 = new Date(d2);
  return (
    date1.getDate() === date2.getDate() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getFullYear() === date2.getFullYear()
  );
}

// внутренняя локаль, чтобы ееменять
const currentLocale = ref(props.locale);

// локали можно вынести, но такой маленький проект этого не требует
const allLocales = {
  ru: {
    weekdays: ["пн", "вт", "ср", "чт", "пт", "сб", "вс"],
    monthFormat: { month: "long", year: "numeric" },
  },
  en: {
    weekdays: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    monthFormat: { month: "long", year: "numeric" },
  },
};

const localeData = computed(
  () => allLocales[currentLocale.value] || allLocales.ru
);

//надпись
const monthLabel = computed(() => {
  const label = displayMonth.value.toLocaleString(
    currentLocale.value,
    localeData.value.monthFormat
  );
  return label.charAt(0).toUpperCase() + label.slice(1);
});

//делаем сетку 42 дня
const daysGrid = computed(() => {
  const daysArr = [];
  const curr = new Date(displayMonth.value.getTime());

  curr.setDate(1); // сперва на 1 число
  const month = curr.getMonth();

  // 0=вс, 1=пн ... 6=сб
  let dayOfWeek = curr.getDay();
  // а нам надо 0=пн, 6=вс
  let startOffset = dayOfWeek === 0 ? 6 : dayOfWeek - 1;

  // отмотка на сколько дней назад
  curr.setDate(1 - startOffset);

  // цикл на 6 недель
  for (let i = 0; i < 42; i++) {
    const d = new Date(curr.getTime());
    const dateStr = fomatDate(d);

    daysArr.push({
      date: d,
      dateStr: dateStr,
      dayNum: d.getDate(),
      isCurrentMonth: d.getMonth() === month,
      isToday: isSame(d, today),
      isSelected: dateStr === selectedDay.value,
    });

    curr.setDate(curr.getDate() + 1);
  }

  return daysArr;
});

// стрелки < >
function changeMonth(dir) {
  const newDate = new Date(
    displayMonth.value.setMonth(displayMonth.value.getMonth() + dir)
  );
  displayMonth.value = newDate;
}

// нажатие по дню
function selectDay(day) {
  // если клик на серый день, меняем месяц
  if (!day.isCurrentMonth) {
    displayMonth.value = new Date(day.date);
  }

  selectedDay.value = day.dateStr;
  emit("date-selected", day.dateStr); // emit!!
}

// переключаем язык
function toggleLang() {
  currentLocale.value = currentLocale.value === "ru" ? "en" : "ru";
}

// стили для ячейки (сегодня/выбран)
function getDayClasses(day) {
  return {
    "text-gray-400": !day.isCurrentMonth,
    "text-black": day.isCurrentMonth,

    // выбранный
    "bg-blue-600 text-white hover:bg-blue-700": day.isSelected,

    // сегодня
    "ring-1 ring-blue-600": day.isToday && !day.isSelected,

    // ховер
    "hover:bg-gray-200": !day.isSelected,
  };
}

// если пропс value поменяли снаружи
watch(
  () => props.value,
  (newVal) => {
    selectedDay.value = newVal;
    displayMonth.value = parseDate(newVal);
  }
);

// если пропс locale поменяли снаружи
watch(
  () => props.locale,
  (newLang) => {
    currentLocale.value = newLang;
  }
);
</script>

<style scoped>
.my-calendar-comp {
  user-select: none;
}
</style>
