<template>
  <q-page class="flex flex-center">
    <div class="calendar-wrapper">
      <div class="future-weather-calendar">
        <div class="calendar-header">
          <div class="year">
            <div class="year-part q-pl-xs">{{ calendarYearPart1 }}</div>
            <div class="year-part q-pl-xs">{{ calendarYearPart2 }}</div>
          </div>
          <div class="month q-pl-md">AI Calendar</div>
        </div>
        <div class="main-content">
          <div class="calendar-section">
            <div class="calendar-grid">
              <div
                class="day-header"
                v-for="day in daysOfWeek"
                :key="day"
                :class="{ weekend: isWeekend(day) }"
              >
                {{ day }}
              </div>
              <div
                v-for="(day, index) in calendarGrid"
                :key="index"
                :class="{
                  'no-data': !day.weatherIcon,
                  weekend: isWeekendIndex(index),
                }"
                class="day-cell"
              >
                <div class="day-content">
                  <div
                    class="day-number"
                    :class="{ 'text-red': isWeekendIndex(index) }"
                  >
                    <div>
                      <div
                        v-if="day.showMonth"
                        :class="{
                          'text-red': isWeekendIndex(index),
                          'month-label': true,
                        }"
                      >
                        {{ day.month }}月
                      </div>
                      {{ day.date }}
                    </div>
                  </div>
                  <div class="day-icon">
                    <q-img
                      v-if="day.weatherIcon"
                      :src="day.weatherIcon"
                      class="weather-icon"
                    >
                      <q-tooltip
                        transition-show="scale"
                        style="font-size: 16px"
                      >
                        {{ day.description }}
                      </q-tooltip>
                    </q-img>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="statistics-section">
            <span class="statistics-title">各項指標統計</span>
            <div
              class="stat-item"
              v-for="(stat, index) in filteredStatistics"
              :key="index"
            >
              <q-img :src="stat.icon" class="stat-icon">
                <q-tooltip transition-show="scale" style="font-size: 16px">{{
                  stat.description
                }}</q-tooltip>
              </q-img>
              <span>{{ stat.count }}天</span>
            </div>
          </div>
        </div>
        <span class="note-text">
          *註:
          本項「AI智繪月曆」為使用AI全球模式進行天氣類型辯識之實驗性產品，使用本產品前，務必了解數值模式的預報能力與極限。正確之天氣資訊，請參考中央氣象署發布之官方預報。
        </span>
      </div>
    </div>
  </q-page>
</template>

<script>
import { ref, computed, onMounted } from "vue";
import Papa from "papaparse";
import { weatherIcons, descriptions } from "assets/calendar/weatherIcons.js";

export default {
  name: "FutureWeatherCalendar",
  setup() {
    const daysOfWeek = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"];
    const calendarDays = ref([]);
    const calendarGrid = ref([]);
    const calendarYear = ref(0);
    const calendarMonth = ref(0);

    const calendarYearPart1 = computed(() =>
      calendarYear.value.toString().slice(0, 2)
    );
    const calendarYearPart2 = computed(() =>
      calendarYear.value.toString().slice(2)
    );

    const statistics = ref([]);

    const loadCSV = () => {
      Papa.parse("/data/calendar/caloutput.csv", {
        download: true,
        header: true,
        complete: (results) => {
          const data = results.data;
          if (data.length > 0) {
            const currentMonth = parseInt(data[0].Month);
            const nextMonth = currentMonth === 12 ? 1 : currentMonth + 1;
            const currentMonthData = data.filter(
              (item) => parseInt(item.Month) === currentMonth
            );
            const nextMonthData = data.filter(
              (item) => parseInt(item.Month) === nextMonth
            );

            calendarYear.value = parseInt(data[0].Year);
            calendarMonth.value = currentMonth;
            calendarDays.value = currentMonthData
              .concat(nextMonthData)
              .map((item, index) => ({
                date: item.Day.padStart(2, "0"),
                month: parseInt(item.Month),
                weatherIcon: weatherIcons[item.Type],
                description: descriptions[item.Type],
                type: parseInt(item.Type),
                index,
              }));
            generateCalendarGrid();
            generateStatistics();
          }
        },
        error: (error) => {
          console.error("Error loading CSV: ", error);
        },
      });
    };

    const generateCalendarGrid = () => {
      if (calendarDays.value.length === 0) return;

      const firstDay = calendarDays.value[0];
      const firstDate = new Date(
        calendarYear.value,
        firstDay.month - 1,
        firstDay.date
      );
      const firstDayIndex = firstDate.getDay();

      calendarGrid.value = Array(42).fill({
        date: "",
        weatherIcon: null,
        description: "",
        index: null,
      });

      let dayIndex = 0;
      let currentMonth = calendarMonth.value;

      for (let i = firstDayIndex; dayIndex < calendarDays.value.length; i++) {
        const day = calendarDays.value[dayIndex];
        if (dayIndex === 0 || day.month !== currentMonth) {
          currentMonth = day.month;
          calendarGrid.value[i] = {
            ...day,
            showMonth: true,
          };
        } else {
          calendarGrid.value[i] = day;
        }
        dayIndex++;
      }

      calendarGrid.value = calendarGrid.value.filter((_, index) => {
        const weekIndex = Math.floor(index / 7);
        return calendarGrid.value
          .slice(weekIndex * 7, (weekIndex + 1) * 7)
          .some((day) => day.date !== "");
      });
    };

    const generateStatistics = () => {
      const typeCounts = calendarDays.value.reduce((acc, day) => {
        if (day.type !== undefined) {
          acc[day.type] = (acc[day.type] || 0) + 1;
        }
        return acc;
      }, {});

      statistics.value = Object.keys(typeCounts).map((type) => ({
        icon: weatherIcons[type],
        count: typeCounts[type],
        description: descriptions[type],
      }));
    };

    const filteredStatistics = computed(() => {
      return statistics.value
        .filter((stat) => stat.count > 0)
        .sort((a, b) => b.count - a.count);
    });

    const isWeekend = (day) => day === "SAT" || day === "SUN";
    const isWeekendIndex = (index) => index % 7 === 0 || index % 7 === 6;

    onMounted(() => {
      loadCSV();
    });

    return {
      daysOfWeek,
      calendarGrid,
      calendarYearPart1,
      calendarYearPart2,
      statistics,
      filteredStatistics,
      isWeekend,
      isWeekendIndex,
    };
  },
};
</script>

<style scoped>
/* 整個月曆範圍 */
.calendar-wrapper {
  height: 680px;
  width: 100%;
}

.future-weather-calendar {
  display: flex;
  flex-direction: column;
  height: 100%;
  margin-left: 10px;
}

/* 最大的標題 */
.calendar-header {
  margin-left: 10px;
  display: flex;
  align-items: center;
}

/* 年份 */
.year {
  display: flex;
  flex-direction: column;
  font-size: 40px;
  font-weight: bold;
  color: rgb(114, 114, 114);
  margin-right: 10px;
}

.year-part {
  line-height: 1;
}

/* 月份 */
.month {
  font-size: 95px;
  font-weight: bolder;
  color: rgb(58, 92, 75);
}

/* 主內容區域 */
.main-content {
  margin-left: 10px;
  display: flex;
  align-items: flex-start;
}

/* 月曆區域 */
.calendar-section {
  flex-grow: 1;
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 2px;
}

/* 星期格子 */
.day-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: white;
  background-color: rgb(108, 147, 111);
  font-size: 25px;
  font-weight: bold;
  padding: 15px;
}

.day-header.weekend {
  color: rgb(205, 2, 2);
}

/* 日期格子 */
.day-cell {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 3px;
  border: 1px solid rgb(108, 147, 111);
}

.day-content {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.day-cell.weekend .day-number {
  color: rgb(189, 19, 19);
}

/* 沒有日期時 */
.day-cell.no-data {
  background-color: rgb(216, 215, 215);
}

/* 日期數字 */
.day-number {
  font-size: 20px;
  font-weight: bold;
  margin-left: 5px;
  margin-right: 3px;
  color: rgb(58, 92, 75);
  position: relative;
}

.text-red {
  color: rgb(189, 19, 19);
}

.month-label {
  font-size: 15px;
  font-weight: bold;
  color: inherit;
  white-space: nowrap;
}

/* icon */
.weather-icon {
  width: 60px;
  height: 60px;
  margin-left: 10px;
}

/* 免責聲明 */
.note-text {
  margin-left: 10px;
  margin-right: 10px;
  margin-top: 15px;
  font-size: 15px;
}

/* 統計表整個區域 */
.statistics-section {
  margin-left: 25px;
  margin-right: 10px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

/* 統計表標題 */
.statistics-title {
  font-size: 23px;
  font-weight: bold;
  margin-bottom: 10px;
  white-space: nowrap;
}

/* 統計結果整個範圍 */
.stat-item {
  display: flex;
  align-items: center;
  margin: 5px 0;
}

/* 統計圖示 */
.stat-icon {
  width: 40px;
  height: 40px;
  margin-right: 20px;
}

/* 統計圖文字 */
.stat-item span {
  font-weight: bold;
  font-size: 18px;
  margin-right: 5px;
}
</style>
