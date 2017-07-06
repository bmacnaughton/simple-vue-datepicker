<template>
  <div class="vdp-datepicker" :class="wrapperClass">

    <!-- Day View -->
    <div class="vdp-datepicker__calendar" v-show="showDayView" :style="calendarStyle">
        <header>
            <span
                @click="previousMonth"
                class="prev"
                v-bind:class="{ 'disabled' : previousMonthDisabled(currDate) }">&lt;</span>
            <span class="up">{{ currMonthName }} {{ currYear }}</span>
            <span
                @click="nextMonth"
                class="next"
                v-bind:class="{ 'disabled' : nextMonthDisabled(currDate) }">&gt;</span>
        </header>
        <span class="cell day-header" v-for="d in daysOfWeek">{{ d }}</span>

        <div class="cell day" v-for="day in displaySlots" :class="dayClasses(day)" @click="selectDate(day)">{{ day.date }}</div>
    </div>
  </div>
</template>

<script>
import DateUtils from '@/utils/DateUtils.js'
import DateLanguages from '@/utils/DateLanguages.js'

export default {
  props: {
    value: {
      validator: function (val) {
        return val === null || val instanceof Date || typeof val === 'string'
      }
    },
    initialDate: {
      type: Date,
      default: null
    },
    name: {
      value: String
    },
    id: {
      value: String
    },
    format: {
      value: String,
      default: 'dd MMM yyyy'
    },
    language: {
      value: String,
      default: 'en'
    },
    disabled: {
      type: Object
    },
    highlighted: {
      type: Object
    },
    placeholder: {
      type: String
    },
    inline: {
      type: Boolean
    },
    inputClass: {
      type: [String, Object]
    },
    wrapperClass: {
      type: [String, Object]
    },
    clearButton: {
      type: Boolean,
      default: false
    },
    clearButtonIcon: {
      type: String,
      default: ''
    },
    calendarButton: {
      type: Boolean,
      default: false
    },
    calendarButtonIcon: {
      type: String,
      default: ''
    },
    initialView: {
      type: String,
      default: 'day'
    },
    disabledPicker: {
      type: Boolean,
      default: false
    },
    required: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      /*
       * Vue cannot observe changes to a Date Object so date must be stored as a timestamp
       * This represents the first day of the current viewing month
       * {Number}
       */
      currDate: new Date(new Date().getFullYear(), new Date().getMonth(), 1).getTime(),
      /*
       * Selected Date
       * {Date}
       */
      selectedDate: null,
      viewDate: new Date(new Date().getFullYear(), new Date().getMonth(), 1),
      /*
       * Flags to show calendar views
       * {Boolean}
       */
      showDayView: false,
      showMonthView: false,
      showYearView: false,
      /*
       * Positioning
       */
      calendarHeight: 0
    }
  },
  /*
  watch: {
    value (value) {
      this.setValue(value)
    }
  },
  // */
  computed: {
    viewMonth () {
      // return this.selectedDate ? this.selectedDate : this.viewDate
      return this.viewDate
    },

    formattedValue () {
      if (!this.selectedDate) {
        return null
      }
      return DateUtils.formatDate(new Date(this.selectedDate), this.format, this.translation)
    },
    translation () {
      return DateLanguages.translations[this.language]
    },
    currMonthName () {
      return DateUtils.getMonthNameAbbr(this.viewMonth, this.translation.months.abbr)
    },
    currYear () {
      const d = new Date(this.currDate)
      return d.getFullYear()
    },
    /**
     * Returns the day number of the week less one for the first of the current month
     * Used to get number of empty cells before the first day in the month
     * @return {Number}
     */
    blankDays () {
      const d = this.viewMonth
      return new Date(d.getFullYear(), d.getMonth(), 1, d.getHours(), d.getMinutes()).getDay()
    },

    daysOfWeek () {
      return this.translation.days
    },

    displaySlots () {
      return [...this.blankSlots, ...this.days]
    },

    blankSlots () {
      let slots = []
      for (let i = 0; i < this.blankDays; i++) {
        slots.push({
          date: '',
          timestamp: i,
          isSelected: false,
          isDisabled: true,
          isHighlighted: false,
          isToday: false
        })
      }
      return slots
    },

    days () {
      let days = []
      // first insert the blank days (previous month's days)
      // now set up the days of the month. start with the first day of the month
      // based on this.currDate
      // const d = new Date(this.dayCalendarDate)
      const d = this.viewMonth
      let dObj = new Date(d.getFullYear(), d.getMonth(), 1, d.getHours(), d.getMinutes())
      let dim = DateUtils.daysInMonth
      let daysInMonth = dim(dObj.getFullYear(), dObj.getMonth())
      for (let i = 0; i < daysInMonth; i++) {
        days.push({
          date: dObj.getDate(),
          timestamp: dObj.getTime(),
          isSelected: this.isSelectedDate(dObj),
          isDisabled: this.isDisabledDate(dObj),
          isHighlighted: this.isHighlightedDate(dObj),
          isToday: DateUtils.dmyEqual(new Date(), dObj)
        })
        dObj.setDate(dObj.getDate() + 1)
      }
      return days
    },

    calendarStyle () {
      let styles = {}

      if (this.isInline) {
        styles.position = 'static'
      }

      return styles
    },
    isOpen () {
      return this.showDayView
    },
    isInline () {
      return this.inline
    }
  },
  methods: {
    getDateObject (date) {
      if (date instanceof Date) {
        return date
      } else if (typeof date === 'string' || typeof date === 'number') {
        let d = new Date(date)
        return isNaN(d.valueOf()) ? null : d
      } else {
        return null
      }
    },

    setViewDate (date) {
      this.viewDate = date
    },

    showDayCalendar () {
      this.showDayView = true
      this.$emit('opened')
    },

    setDate (timestamp) {
      this.selectedDate = new Date(timestamp)
      this.currDate = new Date(this.selectedDate.getFullYear(), this.selectedDate.getMonth(), 1).getTime()
      this.$emit('selected', new Date(timestamp))
    },

    clearDate () {
      this.selectedDate = null
      this.$emit('selected', null)
    },

    /**
     * @param {Object} day
     */
    selectDate (day) {
      if (day.isSelected) {
        this.clearDate()
        return
      }
      if (day.isDisabled) {
        return
      }
      this.setDate(day.timestamp)

      if (this.isInline) {
        return this.showDayCalendar()
      }
    },

    previousMonth () {
      if (this.previousMonthDisabled()) {
        return false
      }

      let d = new Date(this.viewMonth)
      d.setMonth(d.getMonth() - 1)
      this.viewDate = d
      // this.selectedDate = null
      this.$emit('changedMonth', d)
    },

    previousMonthDisabled () {
      return false
    },

    nextMonth () {
      if (this.nextMonthDisabled()) {
        return false
      }
      let d = new Date(this.viewDate)
      let daysInMonth = DateUtils.daysInMonth(d.getFullYear(), d.getMonth())
      d.setDate(d.getDate() + daysInMonth)
      this.viewDate = d
      // this.selectedDate = null
      this.$emit('changedMonth', d)
    },

    nextMonthDisabled () {
      return false
    },

    /**
     * Whether a day is selected
     * @param {Date}
     * @return {Boolean}
     */
    isSelectedDate (dObj) {
      return this.selectedDate && this.selectedDate.getTime() === dObj.getTime()
    },

    /**
     * Whether a day is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isDisabledDate (date) {
      let disabled = false

      if (typeof this.disabled === 'undefined') {
        return false
      }

      if (typeof this.disabled.dates !== 'undefined') {
        this.disabled.dates.forEach((d) => {
          if (date.toDateString() === d.toDateString()) {
            disabled = true
            return true
          }
        })
      }
      if (typeof this.disabled.to !== 'undefined' && this.disabled.to && date < this.disabled.to) {
        disabled = true
      }
      if (typeof this.disabled.from !== 'undefined' && this.disabled.from && date > this.disabled.from) {
        disabled = true
      }
      if (typeof this.disabled.days !== 'undefined' && this.disabled.days.indexOf(date.getDay()) !== -1) {
        disabled = true
      }
      return disabled
    },

    /**
     * Whether a day is highlighted (only if it is not disabled already)
     * @param {Date}
     * @return {Boolean}
     */
    isHighlightedDate (date) {
      if (this.isDisabledDate(date)) {
        return false
      }

      let highlighted = false

      if (typeof this.highlighted === 'undefined') {
        return false
      }

      if (typeof this.highlighted.dates !== 'undefined') {
        this.highlighted.dates.forEach((d) => {
          if (date.toDateString() === d.toDateString()) {
            highlighted = true
            return true
          }
        })
      }

      if (this.isDefined(this.highlighted.from) && this.isDefined(this.highlighted.to)) {
        highlighted = date >= this.highlighted.from && date <= this.highlighted.to
      }

      if (this.highlighted.days && this.highlighted.days.indexOf(date.getDay()) !== -1) {
        highlighted = true
      }
      return highlighted
    },

    /**
     * Helper
     * @param  {mixed}  prop
     * @return {Boolean}
     */
    isDefined (prop) {
      return typeof prop !== 'undefined' && prop
    },

    /**
     * Whether the selected date is in this month
     * @param {Date}
     * @return {Boolean}
     */
    isSelectedMonth (date) {
      return (this.selectedDate &&
        this.selectedDate.getFullYear() === date.getFullYear() &&
        this.selectedDate.getMonth() === date.getMonth())
    },

    /**
     * Whether a month is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isDisabledMonth (date) {
      let disabled = false

      if (typeof this.disabled === 'undefined') {
        return false
      }

      if (typeof this.disabled.to !== 'undefined' && this.disabled.to) {
        if (
          (date.getMonth() < this.disabled.to.getMonth() && date.getFullYear() <= this.disabled.to.getFullYear()) ||
          date.getFullYear() < this.disabled.to.getFullYear()
        ) {
          disabled = true
        }
      }
      if (typeof this.disabled.from !== 'undefined' && this.disabled.from) {
        if (
          this.disabled.from &&
          (date.getMonth() > this.disabled.from.getMonth() && date.getFullYear() >= this.disabled.from.getFullYear()) ||
          date.getFullYear() > this.disabled.from.getFullYear()
        ) {
          disabled = true
        }
      }
      return disabled
    },

    /**
     * Whether a year is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isSelectedYear (date) {
      return this.selectedDate && this.selectedDate.getFullYear() === date.getFullYear()
    },

    /**
     * Whether a month is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isDisabledYear (date) {
      let disabled = false
      if (typeof this.disabled === 'undefined' || !this.disabled) {
        return false
      }

      if (typeof this.disabled.to !== 'undefined' && this.disabled.to) {
        if (date.getFullYear() < this.disabled.to.getFullYear()) {
          disabled = true
        }
      }
      if (typeof this.disabled.from !== 'undefined' && this.disabled.from) {
        if (date.getFullYear() > this.disabled.from.getFullYear()) {
          disabled = true
        }
      }

      return disabled
    },

    dayClasses (day) {
      return {
        'selected': day.isSelected,
        'disabled': day.isDisabled,
        'highlighted': day.isHighlighted,
        'today': day.isToday
      }
    }

  },

  mounted () {
    if (this.value) {
      this.selectDate(this.getDateObject(this.value))
      this.setViewDate(this.getDateObject(this.value))
    } else if (this.initialDate) {
      this.setViewDate(this.getDateObject(this.initialDate))
    } else {
      this.setViewDate(new Date())
    }
    if (this.isInline) {
      this.showDayCalendar()
    }
  }
}
</script>

<style lang="stylus">

$width = 300px

.vdp-datepicker
    position relative
    text-align left
    *
        box-sizing border-box

.vdp-datepicker__calendar
    position absolute
    z-index 100
    background white
    width $width
    border 1px solid #ccc
    header
        display block
        line-height 40px
        span
            display inline-block
            text-align center
            width (100 - (100/7)*2)%
            float left

        .prev
        .next
            width (100/7)%
            float left
            text-indent -10000px
            position relative
            &:after
                content ''
                position absolute
                left 50%
                top 50%
                transform translateX(-50%) translateY(-50%)
                border 6px solid transparent

        .prev
            &:after
                border-right 10px solid #000
                margin-left -5px
            &.disabled:after
                border-right 10px solid #ddd
        .next
            &:after
                border-left 10px solid #000
                margin-left 5px
            &.disabled:after
                border-left 10px solid #ddd

        .prev:not(.disabled)
        .next:not(.disabled)
        .up:not(.disabled)
            cursor pointer
            &:hover
                background #eee

    .disabled
        color #ddd
        cursor default

    .cell
        display inline-block
        padding 0 5px
        width (100/7)%
        height 40px
        line-height 40px
        text-align center
        vertical-align middle
        border 1px solid transparent
        &:not(.blank):not(.disabled).day
        &:not(.blank):not(.disabled).month
        &:not(.blank):not(.disabled).year
            cursor pointer
            &:hover
                border 1px solid #4bd
        &.selected
            background #4bd
            &:hover
                background #4bd
            &.highlighted
                background #4bd
        &.highlighted
            background #cae5ed
        &.grey
            color #888

            &:hover
                background inherit
        &.today
            border 2px solid yellow

        &.day-header
            font-size 75%
            white-space no-wrap
            cursor inherit
            &:hover
                background inherit

    .month,
    .year
        width 33.333%

.vdp-datepicker__clear-button, .vdp-datepicker__calendar-button
    cursor pointer
    font-style normal
</style>
