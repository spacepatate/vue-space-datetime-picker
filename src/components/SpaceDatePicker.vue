<template>
  <div class="space-date-day-picker">
    <div class="header">
      <a class="prev-year-btn" @click="selectPrevYear"></a>
      <a class="prev-month-btn" @click="selectPrevMonth"></a>
      <div class="current-month-year-label">
        <div @click="changeToMonthPicker" class="month-label">
          {{ currentMonthLabel }}</div>
        <div @click="changeToYearPicker" class="year-label">
          {{ datetime.getFullYear() }}</div>
      </div>
      <a class="next-month-btn" @click="selectNextMonth"></a>
      <a class="next-year-btn" @click="selectNextYear"></a>
      <a v-if="showHome" class="icon home" @click="gotoCurrentDdate"></a>
    </div>
    <div class="week-days">
      <div class="week-day" v-for="(weekday, index) of weekDays" :key="`week-day-${index}`">
        {{ weekday }}
      </div>
    </div>
    <div class="days-grid">
      <div class="weeks" v-for="(week, index) of calendar" :key="`week-${index}`">
        <div class="day"
          v-for="(day, index) of week"
          @click="clickDay(day)"
          :key="`day-${index}`"
          :class="{
            'prev-month': (day.getMonth() < datetime.getMonth()
                && day.getFullYear() === datetime.getFullYear())
              || (day.getFullYear() < datetime.getFullYear()),
            'next-month': day.getMonth() > datetime.getMonth()
              && day.getFullYear() >= datetime.getFullYear(),
            'current-day': day.getFullYear() === currentDatetime.getFullYear()
                          && day.getMonth() === currentDatetime.getMonth()
                          && day.getDate() === currentDatetime.getDate(),
            'selected': day.getFullYear() === datetime.getFullYear()
                          && day.getMonth() === datetime.getMonth()
                          && day.getDate() === datetime.getDate(),
          }">
          <span>{{ formatedDayLabel(day) }}</span>
        </div>
      </div>
    </div>
    <div class="time-picker" v-if="showTime">
      <SpaceTimePicker :disabled="disabled"
        :value="value" :hour12="hour12" @ok="onTimeSelect"></SpaceTimePicker>
    </div>
  </div>
</template>
<script>

import {
  Monday,
  Tuesday,
  Wednesday,
  Thursday,
  Friday,
  Saturday,
  Sunday,
} from './WeekDayEnum';

import ModeEnum from './ModeEnum';
import SpaceTimePicker from './SpaceTimePicker.vue';

const daysOfWeekOrder = [Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday];

export default {
  components: {
    SpaceTimePicker,
  },
  props: {
    value: {
      type: Date,
      required: true,
    },

    locale: {
      type: String,
      required: false,
      default: undefined,
    },

    startingDay: {
      type: Number,
      required: false,
      default: 1,
      validator: (value) => value >= 1 && value <= 7,
    },

    showTime: {
      type: Boolean,
      required: false,
      default: true,
    },

    day: {
      type: String,
      required: false,
      default: '2-digit', // numeric (ex: 1) | 2-digit (ex: 01)
    },

    hour12: {
      type: Boolean,
      required: false,
      default: false,
    },

    weekday: {
      type: String,
      required: false,
      default: 'narrow', // long | short | narrow
    },

    month: {
      type: String,
      required: false,
      default: 'short', // numeric | 2-digit | long | short | narrow
    },

    showHome: {
      type: Boolean,
      required: false,
      default: true,
    },

    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      calendar: [], // days of calendar
      currentWeek: [],
      weekDays: [],
      datetime: null,
      weekDaysOrder: [],
    };
  },

  beforeMount() {
    this.datetime = this.value;
    this.init();
    this.initWeekdDays();
  },

  computed: {
    firstDayOfWeek() {
      const index = daysOfWeekOrder.findIndex((day) => day === this.startingDay);
      return daysOfWeekOrder[index];
    },

    firstDayOfMonth() {
      return new Date(this.datetime.getFullYear(), this.datetime.getMonth(), 1);
    },

    lastDayOfMonth() {
      return new Date(this.datetime.getFullYear(), this.datetime.getMonth() + 1, 0);
    },

    currentMonthLabel() {
      return this.datetime.toLocaleString(this.locale, { month: this.month });
    },

    currentDatetime() {
      return new Date();
    },
  },

  watch: {
    value() {
      this.datetime = this.value;
    },

    datetime() {
      this.calendar = [];
      this.currentWeek = [];
      this.init();
    },
  },

  methods: {
    diffDatesInDays(date1, date2) {
      const timeDiff = date2.getTime() - date1.getTime();
      return timeDiff / (1000 * 3600 * 24);
    },

    initWeekdDays() {
      const tmpDaysOfWeekOrder = [...daysOfWeekOrder];
      const index = tmpDaysOfWeekOrder.findIndex((day) => day === this.startingDay);
      const tmp = tmpDaysOfWeekOrder.splice(index);
      this.weekDaysOrder = [...tmp];
      if (index > 0) {
        const tmp2 = tmpDaysOfWeekOrder.splice(0, index);
        this.weekDaysOrder = [...tmp, ...tmp2];
      }
      const currentDay = new Date();
      for (let i = 0; i < this.weekDaysOrder.length; i += 1) {
        const day = currentDay.getDay();
        const distance = this.weekDaysOrder[i] - day;
        const tmp3 = new Date(currentDay.setDate(currentDay.getDate() + distance));
        const label = tmp3.toLocaleString(this.locale, { weekday: this.weekday });
        this.weekDays.push(label);
      }
    },

    insertPrevMonthDays() {
      const firstDayOfMonthDay = this.firstDayOfMonth.getDay() === 0
        ? 7
        : this.firstDayOfMonth.getDay();
      if (this.firstDayOfWeek !== firstDayOfMonthDay) {
        let diff = firstDayOfMonthDay - this.firstDayOfWeek;
        while (diff > 0) {
          // get first day of the month
          const datetime = new Date(this.datetime.getFullYear(),
            this.datetime.getMonth(),
            1,
            this.datetime.getHours(),
            this.datetime.getMinutes(),
            this.datetime.getSeconds());
          // get the starting day of calendar from diff
          datetime.setDate(datetime.getDate() - diff);
          // check if days pushed fullfill a week days
          if (this.currentWeek.length >= 7) {
            this.calendar.push(this.currentWeek);
            this.currentWeek = [];
          }
          this.currentWeek.push((datetime));
          diff -= 1;
        }
        if (this.currentWeek.length >= 7) {
          this.calendar.push(this.currentWeek);
          this.currentWeek = [];
        }
      }
    },

    insertCurrentMonthDays() {
      for (let i = this.firstDayOfMonth.getDate();
        i <= this.lastDayOfMonth.getDate(); i += 1) {
        const datetime = new Date(this.datetime.getFullYear(),
          this.datetime.getMonth(),
          i,
          this.datetime.getHours(),
          this.datetime.getMinutes(),
          this.datetime.getSeconds());
        if (this.currentWeek.length >= 7) {
          this.calendar.push(this.currentWeek);
          this.currentWeek = [];
        }
        this.currentWeek.push((datetime));
      }
      if (this.currentWeek.length >= 7) {
        this.calendar.push(this.currentWeek);
        this.currentWeek = [];
      }
    },

    insertNextMonthDays() {
      const lastDayOfMonthDay = this.lastDayOfMonth.getDay() === 0
        ? 7
        : this.lastDayOfMonth.getDay();
      if (lastDayOfMonthDay < 7) {
        const diff2 = 7 - lastDayOfMonthDay;
        for (let i = 0; i <= diff2; i += 1) {
          const datetime = new Date(this.datetime.getFullYear(),
            this.datetime.getMonth() + 1,
            0,
            this.datetime.getHours(),
            this.datetime.getMinutes(),
            this.datetime.getSeconds());
          datetime.setDate(datetime.getDate() + i + 1);
          if (this.currentWeek.length >= 7) {
            this.calendar.push(this.currentWeek);
            this.currentWeek = [];
          }
          this.currentWeek.push((datetime));
        }
        if (this.currentWeek.length >= 7) {
          this.calendar.push(this.currentWeek);
          this.currentWeek = [];
        }
      }
    },

    init() {
      this.insertPrevMonthDays();
      this.insertCurrentMonthDays();
      this.insertNextMonthDays();
    },

    formatedDayLabel(day) {
      if (this.day === 'numeric') {
        return day.getDate();
      }
      return (day.getDate() < 10) ? `0${day.getDate()}` : day.getDate();
    },

    clickDay(day) {
      if (this.disabled) {
        return;
      }
      if (this.showTime) {
        this.$emit('change', day);
      } else {
        this.$emit('select', day);
      }
    },

    selectMonth(selectedMonth) {
      const tmp = this.datetime.setMonth(selectedMonth);
      this.datetime = new Date(tmp);
    },

    selectPrevMonth() {
      const currentMonth = this.datetime.getMonth();
      if (currentMonth - 1 < 0) {
        this.datetime = new Date(this.datetime.getFullYear() - 1,
          11,
          this.datetime.getDate(),
          this.datetime.getHours(),
          this.datetime.getMinutes(),
          this.datetime.getSeconds());
      }
      return this.selectMonth(currentMonth - 1);
    },

    selectNextMonth() {
      const currentMonth = this.datetime.getMonth();
      if (currentMonth + 1 > 11) {
        this.datetime = new Date(this.datetime.getFullYear() + 1,
          0,
          this.datetime.getDate(),
          this.datetime.getHours(),
          this.datetime.getMinutes(),
          this.datetime.getSeconds());
      }
      return this.selectMonth(currentMonth + 1);
    },

    selectPrevYear() {
      const tmp = this.datetime.setYear(this.datetime.getFullYear() - 1);
      this.datetime = new Date(tmp);
    },

    selectNextYear() {
      const tmp = this.datetime.setYear(this.datetime.getFullYear() + 1);
      this.datetime = new Date(tmp);
    },

    changeToMonthPicker() {
      if (this.disabled) {
        return;
      }
      this.$emit('modeChange', ModeEnum.MonthPick);
    },

    changeToYearPicker() {
      if (this.disabled) {
        return;
      }
      this.$emit('modeChange', ModeEnum.YearPick);
    },

    gotoCurrentDdate() {
      this.datetime = new Date();
    },

    onTimeSelect(datetime) {
      if (this.disabled) {
        return;
      }
      this.$emit('select', datetime);
    },
  },
};
</script>
<style lang="scss" scoped>
  $primaryColor: #a4d0ff;
  $secondaryColor: #bfdeff;
  /* Home Icon */
  .icon {
    margin: 12px;
    background-color: $secondaryColor;
    border: 2px solid $primaryColor;
    display: inline-block;
    position: relative;
    vertical-align: top;
  }
  .icon:after,
  .icon:before {
    background: $secondaryColor;
    border: 2px solid $primaryColor;
    content: '';
    position: absolute;
  }

  .icon.home {
    height: 8px;
    width: 10px;
  }
  .icon.home:after,
  .icon.home:before,
  .icon.home:hover:after,
  .icon.home:hover:before {
    background: none;
  }

  .icon.home:after {
    height: 2px;
    left: 8px;
    top: -8px;
  }

  .icon.home:before {
    border-bottom: 8px solid #bfdeff;
    border-left: 8px solid transparent;
    border-right: 8px solid transparent;
    border-top: 12px solid transparent;
    height: 0;
    left: -3px;
    top: -21px;
    width: 0;
    z-index: 1;
  }

  input.space-input {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    list-style: none;
    position: relative;
    display: inline-block;
    width: 100%;
    height: 32px;
    padding: 4px 11px;
    color: rgba(0, 0, 0, 0.65);
    font-size: 14px;
    line-height: 1.5;
    background-color: #fff;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
  }

  .space-date-day-picker {
    display: flex;
    flex-direction: column;
    width: 300px;
    box-shadow: 1px 2px 6px 0px #bfbfbf;

    a.icon.home {
      position: absolute;
      right: 0;
      cursor: pointer;
    }

    .days-grid {
      height: 200px;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .weeks {
      display: flex;
      .day {
        flex: 1 1 auto;
        padding: 5px;
        cursor: pointer;
        span {
          padding: 3px 5px;
          border: 1px solid transparent;
          &:hover {
            border-radius: 25px;
            background: #bfdeff;
          }
        }

        &.selected {
          span {
            border-radius: 25px;
            background: #bfdeff;
          }
        }

        &.current-day {
          span {
            border-radius: 25px;
            border: 1px solid #a4d0ff;
          }
        }
      }
    }

    .header {
      display: inline-flex;
      justify-content: center;
      position: relative;
      border-bottom: 1px solid #ececec;
      .current-month-year-label {
        display: inline-flex;
        line-height: 33px;
        .month-label {
          min-width: 60px;
        }
        .month-label,
        .year-label {
          cursor: pointer;
        }
      }
      .prev-month-btn,
      .next-month-btn,
      .prev-year-btn,
      .next-year-btn {
        padding: 0 10px;
        color: rgba(0, 0, 0, 0.45);
        font-size: 20px;
        cursor: pointer;
        &:hover {
          color: #a4d0ff;
        }
      }
      .prev-month-btn::after {
        content: '\2039';
      }
      .next-month-btn::after {
        content: '\203A';
      }
      .prev-year-btn::after {
        content: '\00AB';
      }
      .next-year-btn::after {
        content: '\00BB';
      }
    }

    .week-days {
      display: flex;
      width: 100%;
      height: 32px;
      font-size: small;
      justify-content: center;
      align-items: flex-end;
      .week-day {
        flex: 1;
      }
    }

    .prev-month,
    .next-month {
      color: #b9b9b9;
    }
  }
</style>
