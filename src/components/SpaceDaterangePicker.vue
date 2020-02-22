<template>
  <div class="space-daterange-picker">
    <div class="custom-display" ref="customDisplay">
      <slot></slot>
    </div>
    <div class="default-inputs">
      <input type="text"
        v-if="!hasDefaultSlot"
        v-model="rangeStartDateLabel"
        :placeholder="placeholder"
        @click="openPopover"
        @blur="onBlur"
        :disabled="disabled"
        ref="spaceDatetimePickerInput"
        class="sapce-datetime-picker-input start-date space-input" />
      <span class="separator"> ~ </span>
      <input type="text"
        v-if="!hasDefaultSlot"
        v-model="rangeEndDateLabel"
        :placeholder="placeholder"
        @click="openPopover"
        @blur="onBlur"
        :disabled="disabled"
        ref="spaceDatetimePickerInput"
        class="sapce-datetime-picker-input end-date space-input" />
    </div>

    <!-- <div class="space-datetime-popover"
      ref="spaceDatetimePopover"
      :style="popoverStyle"
      v-show="displayPopover">

    </div> -->
    <div class="space-date-range-popover">
      <div class="start-date">
        <SpaceDatePicker v-if="startDatetimeMode === modeEnum.DayPick"
          v-model="rangeDatetimes.startDatetime"
          :showTime="showTime"
          :showHome="false"
          :weekday="weekday"
          :locale="locale"
          :disabled="disabled"
          :startingDate="defaultRangeStartDate"
          :startingDay="startingDay"
          :mode="'date-range'"
          :rangeDatetimes="rangeDatetimes"
          :leftRangeDatePicker="true"
          @modeChange="(mode) = startDatetimeMode = mode;"
          @select="onSelectStartDatetime"></SpaceDatePicker>
        <SpaceMonthPicker
          v-else-if="startDatetimeMode === modeEnum.MonthPick"
          v-model="rangeDatetimes.startDatetime"
          :locale="locale"
          :disabled="disabled"
          :month="month"
          @change="(datetime) => onChange(datetime, modeEnum.DayPick)">
        </SpaceMonthPicker>
        <SpaceYearPicker v-else-if="startDatetimeMode === modeEnum.YearPick"
          :disabled="disabled"
          @change="(datetime) => onChange(datetime, modeEnum.DayPick)"
          v-model="rangeDatetimes.startDatetime"></SpaceYearPicker>
      </div>
      <div class="end-date">
        <SpaceDatePicker v-if="endDatetimeMode === modeEnum.DayPick"
          v-model="rangeDatetimes.endDatetime"
          :showTime="showTime"
          :showHome="false"
          :weekday="weekday"
          :locale="locale"
          :disabled="disabled"
          :startingDay="startingDay"
          :mode="'date-range'"
          :startingDate="defaultRangeEndDate"
          :rangeDatetimes="rangeDatetimes"
          :rightRangeDatePicker="true"
          @modeChange="(mode) = endDatetimeMode = mode;"
          @select="onSelectEndDatetime"></SpaceDatePicker>
        <SpaceMonthPicker
          v-else-if="endDatetimeMode === modeEnum.MonthPick"
          v-model="rangeDatetimes.endDatetime"
          :locale="locale"
          :disabled="disabled"
          :month="month"
          @change="(datetime) => onChange(datetime, modeEnum.DayPick)">
        </SpaceMonthPicker>
        <SpaceYearPicker v-else-if="endDatetimeMode === modeEnum.YearPick"
          :disabled="disabled"
          @change="(datetime) => onChange(datetime, modeEnum.DayPick)"
          v-model="rangeDatetimes.endDatetime"></SpaceYearPicker>
      </div>
    </div>
  </div>
</template>
<script>

import ModeEnum from './ModeEnum';

import {
  reverseFuncs,
  parseFuncs,
  isValidDate,
} from './helper';

import SpaceDatePicker from './SpaceDatePicker.vue';
import SpaceMonthPicker from './SpaceMonthPicker.vue';
import SpaceYearPicker from './SpaceYearPicker.vue';

export default {
  components: {
    SpaceDatePicker,
    SpaceMonthPicker,
    SpaceYearPicker,
  },

  data() {
    return {
      startDatetimeMode: ModeEnum.DayPick,
      endDatetimeMode: ModeEnum.DayPick,
      modeEnum: ModeEnum,
      label: null,
      rangeStartDateLabel: null,
      rangeEndDateLabel: null,
      displayPopover: false,
      popoverStyle: null,
      rangeDatetimes: {
        startDatetime: null,
        endDatetime: null,
      },
    };
  },

  props: {
    value: {
      type: Date,
      required: false,
      default: null,
    },

    placeholder: {
      type: String,
      required: false,
      default: null,
    },

    format: {
      type: String,
      required: false,
      default: 'YYYY-MM-DD',
    },

    locale: {
      type: String,
      required: false,
      default: undefined,
    },

    showTime: {
      type: Boolean,
      required: false,
      default: false,
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

    year: {
      type: String,
      required: false,
      default: 'numeric', // numeric (ex: 2012) | 2-digit (ex: 12)
    },

    month: {
      type: String,
      required: false,
      default: 'short', // numeric | 2-digit | long | short | narrow
    },

    day: {
      type: String,
      required: false,
      default: '2-digit', // numeric (ex: 1) | 2-digit (ex: 01)
    },

    hour: {
      type: String,
      required: false,
      default: '2-digit', // numeric | 2-digit
    },

    minute: {
      type: String,
      required: false,
      default: '2-digit', // numeric | 2-digit
    },

    second: {
      type: String,
      required: false,
      default: '2-digit', // numeric | 2-digit
    },

    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },

    // First date in week days, for exemple Monday is the default first day of week,
    // but can be Sunday for some countries
    startingDay: {
      type: Number,
      required: false,
      default: 1,
      validator: (value) => value >= 0 && value < 7,
    },

    defaultRangeStartDate: {
      type: Date,
      required: false,
      default: () => new Date(),
    },

    defaultRangeEndDate: {
      type: Date,
      required: false,
      default: () => {
        const now = new Date();
        return new Date(now.getFullYear(), now.getMonth() + 1, 1);
      },
    },
  },

  beforeMount() {
    // if (this.value && isValidDate(this.value)) {
    //   this.datetime = this.value;
    // }
  },

  watch: {
    rangeDatetimes: {
      deep: true,
      handler(value) {
        if (value.startDatetime) {
          this.rangeStartDateLabel = this.formatDatetime(value.startDatetime);
        } else {
          this.rangeStartDateLabel = null;
        }
        if (value.endDatetime) {
          this.rangeEndDateLabel = this.formatDatetime(value.endDatetime);
        } else {
          this.rangeEndDateLabel = null;
        }
      },
    },
  },

  computed: {
    hasDefaultSlot() {
      return this.$slots.default;
    },
  },

  methods: {

    formatDatetime(datetime) {
      let tmp = this.format;
      for (let i = 0; i < parseFuncs.length; i += 1) {
        const parseFunc = parseFuncs[i];
        if (this.format.includes(parseFunc.key)) {
          tmp = parseFunc.handler(datetime, tmp);
        }
      }
      return tmp;
    },

    onStartDatetimeChange(datetime, mode) {
      this.rangeDatetimes.startDatetime = datetime;
      this.startDatetimeMode = mode;
    },

    onEndDatetimeChange(datetime, mode) {
      this.rangeDatetimes.endDatetime = datetime;
      this.endDatetimeMode = mode;
    },

    isSameDay(d1, d2) {
      return (d1.getFullYear() === d2.getFullYear()
          && d1.getMonth() === d2.getMonth()
          && d1.getDate() === d2.getDate());
    },

    isSameRangeStartDate(datetime) {
      return this.rangeDatetimes.startDatetime
        && this.isSameDay(this.rangeDatetimes.startDatetime, datetime);
    },

    isSameRangeEndDate(datetime) {
      return this.rangeDatetimes.endDatetime
        && this.isSameDay(this.rangeDatetimes.endDatetime, datetime);
    },

    onSelectStartDatetime(datetime) {
      if (this.isSameRangeStartDate(datetime)) {
        this.rangeDatetimes.startDatetime = null;
        this.rangeDatetimes.endDatetime = null;
        return;
      }
      // if the range start datetime is already set, and the end datetime is blank,
      // assign the value to end datetime if the new selected one is newer,
      if (this.rangeDatetimes.startDatetime
        && datetime.getTime() > this.rangeDatetimes.startDatetime.getTime()) {
        if (this.isSameRangeEndDate(datetime)) {
          this.rangeDatetimes.endDatetime = null;
          return;
        }
        this.rangeDatetimes.endDatetime = datetime;
        return;
      }
      this.rangeDatetimes.startDatetime = datetime;
    },

    onSelectEndDatetime(datetime) {
      if (this.isSameRangeEndDate(datetime)) {
        this.rangeDatetimes.endDatetime = null;
        return;
      }
      if (this.isSameRangeStartDate(datetime)) {
        this.rangeDatetimes.startDatetime = null;
        this.rangeDatetimes.endDatetime = null;
        return;
      }
      if (!this.rangeDatetimes.startDatetime) {
        this.rangeDatetimes.startDatetime = datetime;
        return;
      }
      if (this.rangeDatetimes.startDatetime.getTime() > datetime.getTime()) {
        this.rangeDatetimes.startDatetime = datetime;
        return;
      }
      this.rangeDatetimes.endDatetime = datetime;
    },

    onBlur() {
      let startDatetime = new Date();
      let endDatetime = new Date();
      if (this.disabled) {
        return;
      }
      if (!this.rangeStartDateLabel
        && !this.rangeEndDateLabel) {
        this.$emit('input', null);
        return;
      }
      for (let i = 0; i < reverseFuncs.length; i += 1) {
        const parseFunc = reverseFuncs[i];
        if (this.format.includes(parseFunc.key)) {
          if (this.rangeStartDateLabel) {
            const tmp = parseFunc.handler(this.rangeStartDateLabel, this.format, startDatetime);
            startDatetime = new Date(tmp);
          } else {
            this.rangeDatetimes.startDatetime = null;
          }
          if (this.rangeEndDateLabel) {
            const tmp = parseFunc.handler(this.rangeEndDateLabel, this.format, endDatetime);
            endDatetime = new Date(tmp);
          } else {
            this.rangeDatetimes.endDatetime = null;
          }
        }
      }
      if (this.rangeStartDateLabel && isValidDate(startDatetime)) {
        this.rangeDatetimes.startDatetime = startDatetime;
      }
      if (this.rangeEndDateLabel && isValidDate(endDatetime)) {
        this.rangeDatetimes.endDatetime = endDatetime;
      }
      // The end datetime is smaller than the start datetime,
      // set to null
      if (this.rangeDatetimes.startDatetime
        && this.rangeDatetimes.endDatetime
        && this.rangeDatetimes.endDatetime.getTime()
          < this.rangeDatetimes.startDatetime.getTime()) {
        this.rangeDatetimes.startDatetime = null;
        this.rangeDatetimes.endDatetime = null;
      }
      this.$emit('input', this.rangeDatetimes);
    },

    onClickHandler(e) {
      if (!this.$refs.spaceDatetimePopover) {
        return;
      }
      const viewportOffset = this.$refs.spaceDatetimePopover.getBoundingClientRect();
      const zoneX = viewportOffset.x + viewportOffset.width;
      const zoneY = viewportOffset.y + viewportOffset.height;
      if (e.clientX < viewportOffset.x
          || e.clientX > zoneX
          || e.clientY < viewportOffset.y
          || e.clientY > zoneY) {
        this.displayPopover = false;
        document.removeEventListener('click', this.onClickHandler);
        if (isValidDate(this.datetime) && !this.disabled) {
          this.$emit('input', this.datetime);
        }
        this.mode = ModeEnum.DayPick;
      }
    },

    openPopover(e) {
      this.displayPopover = true;
      const el = this.$slots.default
        ? this.$refs.customDisplay : this.$refs.spaceDatetimePickerInput;
      if (!el) {
        return;
      }
      const viewportOffset = el.getBoundingClientRect();
      const popoverPositionLeft = viewportOffset.left;
      const popoverPositionTop = viewportOffset.top + viewportOffset.height;
      this.popoverStyle = {
        position: 'absolute',
        left: `${popoverPositionLeft}px`,
        top: `${popoverPositionTop}px`,
      };
      e.stopPropagation();
      e.preventDefault();
      document.addEventListener('click', this.onClickHandler);
    },
  },
};
</script>
<style lang="scss">

  .space-daterange-picker {
    padding: 15px;
    input.space-input {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      list-style: none;
      position: relative;
      display: inline-block;
      width: 100%;
      max-width: 300px;
      height: 32px;
      padding: 4px 11px;
      color: rgba(0, 0, 0, 0.65);
      font-size: 14px;
      line-height: 1.5;
      background-color: #fff;
      border: 1px solid #d9d9d9;
      border-radius: 4px;

      &.start-date,
      &.end-date {
        border: 0;
        flex: 1 1 auto;
      }
    }

    .default-inputs {
      border: 1px solid #d9d9d9;
      border-radius: 4px;
      max-width: 600px;
      display: flex;
    }

    .separator {
      background-color: #fff;
      display: inline-block;
      line-height: 32px;
      padding: 0 5px;
    }
  }

  .space-date-range-popover {
    display: flex;
  }
</style>
