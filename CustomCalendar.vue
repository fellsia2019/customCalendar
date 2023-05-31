<template>
  <div
    ref="customCalendar"
    class="custom-calendar"
    :class="{
      'custom-calendar--is-opened': !isHiddenCalendar,
      'custom-calendar--form-theme': isFormTheme,
      'custom-calendar--is-error': isError,
    }"
  >
    <div
      class="custom-calendar__error"
      :class="{'custom-calendar__error--is-hidden': !errorText}"
    >
      {{ errorText }}
    </div>
    <div v-if="subtitle" class="custom-calendar__subtitle">
      {{ subtitle }}
    </div>
    <div class="custom-calendar__label" @click="openCalendar()">
      <input
        :placeholder="placeholder"
        :value="activeDate"
        @input="onChangeDate"
        class="custom-calendar__input"
      >
      <div class="custom-calendar__label-close" @click.stop="closeCalendar()">
        <inline-svg class="custom-calendar__label-icon" src="u-arr-down"/>
      </div>
    </div>
    <div
      class="custom-calendar__menu"
      :class="{'custom-calendar__menu--is-hidden': isHiddenCalendar}"
    >
      <div class="custom-calendar__controls">
        <div
          class="custom-calendar__month"
          :class="{'custom-calendar__month--is-opened': !isHiddenMonthMenu}"
          ref="dropdownMonth"
        >
          <div
            class="custom-calendar__control-month"
            @click="isHiddenMonthMenu = !isHiddenMonthMenu"
          >
            <span>{{ getCurrentMonthInLocale }}</span>
            <inline-svg class="custom-calendar__month-icon" src="u-arr-down"/>
          </div>
          <div
            class="custom-calendar__month-menu"
            :class="{'custom-calendar__month-menu--is-hidden': isHiddenMonthMenu}"
          >
            <div class="custom-calendar__month-wrapper">
              <div
                v-for="month in getMonthList"
                :key="`custom-calendar__month-item-${month.code}`"
                class="custom-calendar__month-item"
                :class="{'custom-calendar__month-item--is-active': month.code === getCurrentMonth}"
                @click="selectMonth(month.code)"
              >
                {{ month.name }}
              </div>
            </div>
          </div>
        </div>
        <div
          class="custom-calendar__month"
          :class="{'custom-calendar__month--is-opened': !isHiddenYearMenu}"
          ref="dropdownYear"
        >
          <div
            class="custom-calendar__control-month"
            @click="isHiddenYearMenu = !isHiddenYearMenu"
          >
            <span>{{ getCurrentYear }}</span>
            <inline-svg class="custom-calendar__month-icon" src="u-arr-down"/>
          </div>
          <div
            class="custom-calendar__month-menu"
            :class="{'custom-calendar__month-menu--is-hidden': isHiddenYearMenu}"
          >
            <div class="custom-calendar__month-wrapper">
              <div
                v-for="(year, i) in getYearList"
                :key="`custom-calendar__year-item-${i}`"
                class="custom-calendar__month-item"
                :class="{'custom-calendar__month-item--is-active': year === getCurrentYear}"
                @click="selectYear(year)"
              >
                {{ year }}
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="custom-calendar__content">
        <div
          v-for="week in getCalendarMartix"
          :key="`custom-calendar__week-${week}`"
          class="custom-calendar__week"
        >
          <div
            v-for="day in week"
            :key="`custom-calendar__day-${day}`"
            class="custom-calendar__day"
            :class="{
              'custom-calendar__day--is-active': isActiveDay(day),
              'custom-calendar__day--is-disabled': isDisabledDay(day)
            }"
            @click="selectDay(day)"
          >
            {{ day }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    placeholder: {
      type: String,
      default: 'Выберите дату'
    },
    minDate: {
      type: [String, Date],
      default: ''
    },
    maxDate: {
      type: [String, Date],
      default: ''
    },
    value: {
      type: [String, Date],
      require: true
    },
    subtitle: {
      type: String,
      default: ''
    },
    isFormTheme: {
      type: Boolean,
      default: false
    },
    isError: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      currentDate: null,
      activeDate: this.formatDate(this.value),
      isHiddenCalendar: true,
      isHiddenMonthMenu: true,
      isHiddenYearMenu: true,
      errorText: '',
      errorsStack: null
    }
  },
  computed: {
    getCalendarMartix() {
      const calendarMatrix = [];
      let week = []
      const firstDay = this.getISOWeekDay(this.currentDate)
      let limitInWeek = 7 - firstDay + 1
      const countDaysInMonth = this.getCountDaysInMonth(this.currentDate)

      for (let i = 1; i <= countDaysInMonth; i++) {
        week.push(i)

        if (week.length === limitInWeek || (i === countDaysInMonth && week.length >= 1)) {
          calendarMatrix.push(week);
          week = []
          limitInWeek = 7
        }
      }

      return calendarMatrix
    },
    getCurrentMonth() {
      return this.parseStrDate(this.currentDate).getMonth()
    },
    getCurrentYear() {
      return this.parseStrDate(this.currentDate).getFullYear()
    },
    getCurrentMonthInLocale() {
      return this.parseStrDate(this.currentDate).toLocaleString('ru', { month: 'long' });
    },
    getMonthList() {
      const months = []

      for (let i = 0; i <= 11; i++) {
        const month = {
          code: i,
          name: new Date(this.getCurrentYear, i+1, 0).toLocaleString('ru', { month: 'long' })
        }
        months.push(month)
      }

      return months
    },
    getYearList() {
      const minYear = this.parseStrDate(this.minDate).getFullYear();
      const maxYear = this.parseStrDate(this.maxDate).getFullYear();
      const countYears = maxYear - minYear
      const yearsList = []

      for (let i = 0; i <= countYears; i++) {
        yearsList.push(maxYear - i)
      }

      return yearsList
    }
  },
  methods: {
    getISOWeekDay(date) {
      const currentDate = this.parseStrDate(date)
      currentDate.setDate(1)
      const isoWeekDay = currentDate.getDay()

      return isoWeekDay === 0 ? 7 : isoWeekDay
    },
    getCountDaysInMonth(date) {
      const currentDate = this.parseStrDate(date)
      const countDaysInMonth = new Date(currentDate.getFullYear(), currentDate.getMonth()+1, 0).getDate()

      return countDaysInMonth
    },
    selectDay(day) {
      this.activeDate = this.formatDay(day)
      this.currentDate = this.activeDate
      this.closeCalendar()
    },
    selectMonth(month) {
      const thisDate = this.parseStrDate(this.currentDate)
      thisDate.setDate(1)
      thisDate.setMonth(month)

      this.currentDate = this.formatDate(thisDate)
      this.isHiddenMonthMenu = true
    },
    selectYear(year) {
      const thisDate = this.parseStrDate(this.currentDate)
      thisDate.setFullYear(year)

      this.currentDate = this.formatDate(thisDate)
      this.isHiddenYearMenu = true
    },
    selectDate(date) {
      this.activeDate = this.formatDate(date)
      this.currentDate = this.formatDate(date)
    },
    isActiveDay(day) {
      if (!this.activeDate) {
        return false
      }
      const activeDate = this.parseStrDate(this.activeDate)
      const thisDate = this.parseStrDate(this.currentDate)
      thisDate.setDate(day)

      return Date.parse(activeDate) === Date.parse(thisDate)
    },
    isDisabledDay(day) {
      const maxDate = this.parseStrDate(this.maxDate)
      const minDate = this.parseStrDate(this.minDate)
      const thisDate = this.parseStrDate(this.currentDate)
      thisDate.setDate(day)

      return thisDate > maxDate || thisDate < minDate
    },
    openCalendar() {
      this.isHiddenCalendar = false
    },
    closeCalendar() {
      this.isHiddenCalendar = true
    },
    validateDate(date) {
      let isValid = true
      const thisDate = this.parseStrDate(date)
      const year = thisDate.getFullYear()
      const month = thisDate.getMonth() + 1
      const day = thisDate.getDate()

      if (
        !thisDate
        || !(thisDate instanceof Date && !isNaN(thisDate))
        || !year
        || !month
        || !day
        || String(year).length > 4
        || month > 12
        || month < 0
      ) {
        this.setError()
        isValid = false
      } else {
        this.clearError()
        isValid = true
      }

      return isValid
    },
    setError(text = 'Введенная дата должна быть в формате ДД.ММ.ГГГГ') {
      this.errorsStack = clearTimeout(this.errorsStack)
      this.errorText = text
      this.errorsStack = setTimeout(this.clearError, 5000);
    },
    clearError() {
      this.errorText = ''
      this.errorsStack = clearTimeout(this.errorsStack)
    },
    onChangeDate(e) {
      if (!this.validateDate(e.target.value)) {
        return
      }

      this.selectDate(e.target.value)
    },
    formatDay(day, date = this.currentDate) {
      let thisDate = this.parseStrDate(date)
      thisDate.setDate(day)

      return this.formatDate(thisDate)
    },
    formatDate(date) {
      if (!date) {
        return
      }
      let thisDate = this.parseStrDate(date)
      const transformDate = thisDate.toLocaleDateString('ru-RU')

      return transformDate
    },
    parseStrDate(date) {
      let thisDate = date

       if (!(date instanceof Date && !isNaN(date))) {
        const partialsDate = date.split('.')
        const dd = partialsDate[0]
        const mm = partialsDate[1]
        const yy = partialsDate[2]

        thisDate = new Date(yy, mm-1, dd)
      }

      return thisDate
    },
    closeCalendarFromClickOutside(e) {
      const options = {
        target: e.target,
        containsQuery: 'custom-calendar',
        slosestClass: '.custom-calendar',
        ref: this.$refs.customCalendar,
        closeFunction: () => this.isHiddenCalendar = true
      }

      this.closeFromClickOutside(options)
    },
    closeMonthFromClickOutside(e) {
      const options = {
        target: e.target,
        containsQuery: 'custom-calendar__month',
        slosestClass: '.custom-calendar__month',
        ref: this.$refs.dropdownMonth,
        closeFunction: () => this.isHiddenMonthMenu = true
      }

      this.closeFromClickOutside(options)
    },
    closeYearFromClickOutsie(e) {
       const options = {
        target: e.target,
        containsQuery: 'custom-calendar__month',
        slosestClass: '.custom-calendar__month',
        ref: this.$refs.dropdownYear,
        closeFunction: () => this.isHiddenYearMenu = true
      }

      this.closeFromClickOutside(options)
    },
    closeFromClickOutside(options) {
      if (
        !(options.target.classList.contains(options.containsQuery) ||
        (Boolean(options.target.closest(options.slosestClass)) &&
        options.target.closest(options.slosestClass) === options.ref) ||
        options.target === options.ref)
      ) {
        options.closeFunction()
      }
    }
  },
  created() {
    if (this.value) {
      this.currentDate = this.formatDate(this.value)
    } else {
      this.currentDate = this.formatDate(new Date())
    }
  },
  mounted() {
    document.addEventListener('click', this.closeCalendarFromClickOutside)
    document.addEventListener('click', this.closeMonthFromClickOutside)
    document.addEventListener('click', this.closeYearFromClickOutsie)
  },
  beforeDestroy() {
    document.removeEventListener('click', this.closeCalendarFromClickOutside)
    document.removeEventListener('click', this.closeMonthFromClickOutside)
    document.removeEventListener('click', this.closeYearFromClickOutsie)
  },
  watch: {
    currentDate(newDate) {
      this.$nextTick(() => {
        const thisDate = this.parseStrDate(newDate)
        const minDate = this.parseStrDate(this.minDate)
        const maxDate = this.parseStrDate(this.maxDate)

        if (Date.parse(thisDate) < Date.parse(minDate)) {
          this.setError('Минимальная дата: ' + this.minDate)
          this.selectDate(this.minDate)
        } else if (Date.parse(thisDate) > Date.parse(maxDate)) {
          this.setError('Максимальная дата: ' + this.maxDate)
          this.selectDate(this.maxDate)
        } else {
          this.$emit('input', this.activeDate)
        }
      })
    },
    activeDate(newDate) {
      this.$nextTick(() => {
        if (!this.validateDate(newDate)) {
          return
        }
        this.selectDate(newDate)
        this.$emit('input', this.activeDate, newDate)
      })
    }
  }
}
</script>

<style lang="scss">
@import './scss/base/u-includes';

$b: '.custom-calendar';

$size-day-cell: 50px;
$size-day-cell-mob: 42px;
$size-day-cell-mob-sm: 35px;
$size-day-gap: 4px;

@mixin size-calendar-week($size-cell, $size-gap) {
  width: calc(($size-cell * 7) + ($size-gap * 6));
}

#{$b} {
  width: 400px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  font-family: $font-family-inter;
  font-size: 16px;
  line-height: 1.2;
  font-weight: 500;
  color: $black-true;
  position: relative;
  user-select: none;

  @include tablet {
    width: 344px;
  }

  @include mobile-md {
    width: 100%;
  }

  @include mobile {
    max-width: 100%;
  }

  // .custom-calendar--form-theme
  &--form-theme {
    width: 100% !important;
  }

  // .custom-calendar__subtitle
  &__subtitle {
    font-family: $font-family-inter;
    font-size: 12px;
    line-height: 1.3;
    text-transform: initial;
    margin-bottom: 8px;
    margin-left: 8px;
    font-weight: normal;
  }

  // .custom-calendar__week
  &__week {
    display: flex;
    @include size-calendar-week($size-day-cell, $size-day-gap);

    &:first-child {
      justify-content: flex-end;
    }
    &:last-child {
      justify-content: flex-start;
    }

    @include tablet {
      @include size-calendar-week($size-day-cell-mob, $size-day-gap);
    }

    @include mobile-sm {
      @include size-calendar-week($size-day-cell-mob-sm, $size-day-gap);
    }
  }

  // .custom-calendar__day
  &__day {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    width: $size-day-cell;
    height: $size-day-cell;
    border-radius: 50%;

    transition: $transtion-default;
    cursor: pointer;

    @include tablet {
      width: $size-day-cell-mob;
      height: $size-day-cell-mob;
    }

    @include mobile-sm {
      width: $size-day-cell-mob-sm;
      height: $size-day-cell-mob-sm;
    }

    &:not(:last-child) {
      margin-right: $size-day-gap;
    }

    &:not(#{$b}__day--is-active):hover {
      @include mobile-min {
        color: $color-base;
      }
    }

    &--is-active {
      color: $white-true;
      background-color: $color-base;
    }

    // .custom-calendar__day--is-disabled
    &--is-disabled {
      opacity: 0.3;
      pointer-events: none;
      touch-action: none;
    }
  }

  // .custom-calendar__error
  &__error {
    font-size: 14px;
    color: $color-error;
    max-height: 40px;
    margin-bottom: 10px;
    pointer-events: none;
    touch-action: none;
    transition: all 0.5s ease;

    // .custom-calendar__error--is-hidden
    &--is-hidden {
      opacity: 0;
      max-height: 0;
      margin: 0;
    }
  }

  // .custom-calendar__label
  &__label {
    position: relative;
    width: 100%;
    height: 48px;
    display: flex;
    align-items: center;
    overflow: hidden;
    background-color: transparent;
    transition: all 0.5s ease;
    border-bottom: 1px solid $color-base-origin;

    #{$b}--form-theme & {
      height: 35px;
    }

    #{$b}--is-error & {
      border-color: $color-error;
    }

    // .custom-calendar__label-close
    &-close {
      position: absolute;
      right: 0;
      top: 0;
      height: 100%;
      width: 50px;
      display: flex;
      align-items: center;
      justify-content: flex-end;
      padding-right: 10px;
      pointer-events: none;
      touch-action: none;

      #{$b}--is-opened & {
        cursor: pointer;
        pointer-events: unset;
        touch-action: unset;
        z-index: 2;
      }
    }

    // .custom-calendar__label-icon
    &-icon {
      width: 14px;
      height: 7px;
      color: $color-base-origin;

      #{$b}--form-theme & {
        width: 16px;
        height: 10px;
      }

      svg {
        fill: transparent !important;

        & > * {
          fill: transparent !important;
          stroke: currentColor;
        }
      }
    }
  }

  // .custom-calendar__input
  &__input {
    font-family: $font-family-fugue;
    font-weight: 500;
    padding: 8px 50px 16px 8px;
    height: 100%;
    width: 100%;
    &::placeholder {
      color: rgba($black-true, 0.5)
    }

    #{$b}--form-theme & {
      font-family: $font-family-inter;
      font-weight: 400;
      padding: 0 50px 8px 8px;
    }
  }

  // .custom-calendar__menu
  &__menu {
    position: absolute;
    left: 0;
    top: 100%;
    max-height: 500px;
    width: 100%;
    background-color: $white-true;
    padding: 24px 0 12px;
    transition: all 0.5s ease;
    z-index: 5;
    box-shadow: $shadow-default;


    @include removeScrollBar;

    // .custom-calendar__menu--is-hidden
    &--is-hidden {
      max-height: 0;
      padding: 0;
      opacity: 0;
      pointer-events: none;
      touch-action: none;
      overflow: hidden;
    }
  }

  // .custom-calendar__controls
  &__controls {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    padding: 0 16px;
    margin-bottom: 10px;

    // .custom-calendar__control-labels
    &-labels {
      font-weight: 700;
    }
  }

  // .custom-calendar__month
  &__month {
    position: relative;

    // .custom-calendar__month-icon
    &-icon {
      position: absolute;
      right: 16px;
      top: 50%;
      width: 14px;
      height: 7px;
      color: $black-true;
      transform: translateY(-50%);

      svg {
        fill: transparent !important;

        & > * {
          fill: transparent !important;
          stroke: currentColor;
        }
      }
    }

    // .custom-calendar__month-menu
    &-menu {
      position: absolute;
      left: 0;
      top: 100%;
      width: 100%;
      background-color: $color-secondary;
      padding: 0px 6px 10px 16px;

      transition: all 0.5s ease;
      z-index: 1;
      border-bottom-left-radius: 24px;
      border-bottom-right-radius: 24px;

      &:after {
        content: "";
        position: absolute;
        top: 0;
        left: 12px;
        width: calc(100% - (12px *2));
        height: 1px;
        background-color: rgba($black-true, 0.5);
      }

      // .custom-calendar__month-menu--is-hidden
      &--is-hidden {
        opacity: 0;
        pointer-events: none;
        touch-action: none;
        transform: translateY(-10px);
      }
    }

    // .custom-calendar__month-wrapper
    &-wrapper {
      max-height: 220px;
      overflow-x: hidden;
      overflow-y: auto;
      padding-right: 10px;
      transition: all 0.5s ease;

      @include customScrollBar(3px, rgba($color-base,0.5), rgba($black-true, 0.1));

      @include tablet {
        max-height: 200px;
      }

      @include mobile-sm {
        max-height: 170px;
      }

      #{$b}__month-menu--is-hidden & {
        overflow: hidden;
        max-height: 0;
      }
    }

    // .custom-calendar__month-item
    &-item {
      padding: 5px 0;
      text-transform: capitalize;
      cursor: pointer;
      transition: $transtion-default;

      &:not(#{$b}__month-item--is-active):hover {
        @include mobile-min {
          color: $color-base;
        }
      }

      &--is-active {
        pointer-events: none;
        touch-action: none;
        color: rgba($black-true, 0.5);
      }
    }
  }

  // .custom-calendar__control
  &__control {
    width: 30px;
    height: 20px;
    display: flex;
    align-items: center;
    cursor: pointer;

    &-prev {
      justify-content: flex-start;
    }
    &-next {
      justify-content: flex-end;
    }

    // .custom-calendar__control-month
    &-month {
      position: inherit;
      display: flex;
      align-items: center;
      text-transform: capitalize;
      height: 40px;
      background-color: $color-secondary;
      border-radius: 24px;
      padding: 5px 40px 5px 14px;
      color: rgba($black-true, 0.5);
      transition: all 0.5s ease;
      cursor: pointer;

      #{$b}__month--is-opened & {
        border-bottom-left-radius: 0;
        border-bottom-right-radius: 0;
      }
    }

    // .custom-calendar__control-icon
    &-icon {
      width: 8px;
      height: 13px;

      svg {
        fill: transparent !important;

        & > * {
          fill: transparent !important;
          stroke: currentColor;
        }
      }
    }
  }

  // .custom-calendar__content
  &__content {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 0 12px;
  }

}
</style>
