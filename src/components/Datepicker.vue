<template>
  <div class="svd-outer">

    <!-- Day View -->
    <div :class="cc.svdCalWrapper" v-show="showDayView" :style="calendarStyle">
        <header>
            <span
                @click="previousMonth"
                :class="cc.svdCalPrev"
            ></span>
            <span class="up">{{ currMonthName }} {{ currYear }}</span>
            <span
                @click="nextMonth"
                :class="cc.svdCalNext"
            ></span>
        </header>

        <div :class="cc.svdCalDayNames" v-for="d in daysOfWeek">
          <span>{{ d }}</span>
        </div>

        <div v-for="day in displaySlots" :class="dayClasses(day)" @click="selectDate(day)">
          <span>{{ day.date }}</span>
        </div>

    </div>
  </div>
</template>

<script>
import DateUtils from '@/utils/DateUtils.js'
import DateLanguages from '@/utils/DateLanguages.js'

var defaultClasses = {
  svdCalWrapper: 'svd-cal-wrapper',
  svdCalPrev: 'svd-cal-prev',
  svdCalNext: 'svd-cal-next',
  svdCalDayNames: ['svd-cal-cell', 'svd-cal-day-names'],
  svdCalDays: ['svd-cal-cell', 'svd-cal-day'],

  svdDayBlank: 'svd-day-blank',
  svdDaySelected: 'svd-day-selected',
  svdDayDisabled: 'svd-day-disabled',
  svdDayHighlighted: 'svd-day-highlighted',
  svdDayToday: 'svd-day-today'
}

export default {
  props: {
    value: {
      validator: function (val) {
        return val === null || val instanceof Date || typeof val === 'string'
      }
    },
    initialViewDate: {
      type: Date,
      default () {
        return new Date()
      }
    },
    language: {
      value: String,
      default: 'en'
    },
    disabled: {
      type: Object
    },
    inline: {
      type: Boolean
    },
    classes: {
      type: Object,
      default () {
        return {}
      }
    },
    mondayFirst: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      // selected date {Date}
      selectedDate: null,
      // which month is being displayed in the calendar
      viewDate: this.setViewDate(new Date()),
      showDayView: false
    }
  },

  computed: {
    // calendar classes
    cc () {
      let c = Object.assign({}, defaultClasses, this.classes)
      c.svdCalPrev = [{'disabled': this.prevMonthDisabled(this.viewDate)}, c.svdCalPrev]
      c.svdCalNext = [{'disabled': this.nextMonthDisabled(this.viewDate)}, c.svdCalNext]
      return c
    },

    translation () {
      return DateLanguages.translations[this.language]
    },
    currMonthName () {
      return DateUtils.getMonthNameAbbr(this.viewDate, this.translation.months.abbr)
    },
    currYear () {
      return this.viewDate.getFullYear()
    },
    /**
     * Returns the day number of the week less one for the first of the current month
     * Used to get number of empty cells before the first day in the month
     * @return {Number}
     */
    blankDays () {
      const d = this.viewDate
      let day = new Date(d.getFullYear(), d.getMonth(), 1).getDay()
      return this.mondayFirst ? (day + 6) % 7 : day
    },

    daysOfWeek () {
      let dayNames = this.translation.days.slice()

      if (!this.mondayFirst) return dayNames

      return dayNames.slice(1).concat(dayNames[0])
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
          isBlank: true,
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
      let dObj = new Date(
        this.viewDate.getFullYear(), this.viewDate.getMonth(), 1
      )
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
      }
      if (typeof date === 'string' || typeof date === 'number') {
        let d = new Date(date)
        return isNaN(d.valueOf()) ? null : d
      }
      return null
    },

    setViewDate (date) {
      let d = date.getTime()
      this.viewDate = new Date(d - d % 86400000)
      return this.viewDate
    },

    showDayCalendar () {
      this.showDayView = true
      this.$emit('opened')
    },

    setDate (timestamp) {
      this.selectedDate = new Date(timestamp)
      this.$emit('selected', new Date(timestamp))
      this.$emit('input', this.selectedDate)
    },

    clearDate () {
      this.selectedDate = null
      this.$emit('selected', null)
      this.$emit('input', null)
    },

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
      if (this.prevMonthDisabled()) {
        return false
      }

      let d = new Date(this.viewDate)
      d.setMonth(d.getMonth() - 1)
      this.setViewDate(d)
      this.$emit('changedMonth', d)
    },

    prevMonthDisabled () {
      return false
    },

    nextMonth () {
      if (this.nextMonthDisabled()) {
        return false
      }
      let d = new Date(this.viewDate)
      let daysInMonth = DateUtils.daysInMonth(d.getFullYear(), d.getMonth())
      d.setDate(d.getDate() + daysInMonth)
      this.setViewDate(d)
      this.$emit('changedMonth', d)
    },

    nextMonthDisabled () {
      return false
    },

    //
    // normalize a date, i.e., adjust time to 00:00:00.000
    //
    norm (date) {
      return new Date(
        date.getFullYear(),
        date.getMonth(),
        date.getDate(),
        0, 0, 0
      )
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
      if (!this.disabled) {
        return false
      }

      if (this.disabled.dates) {
        if (this.disabled.dates.some(d => d.getTime() === date.getTime())) {
          return true
        }
      }

      if (this.disabled.to && date.getTime() < this.norm(this.disabled.to).getTime()) {
        return true
      }

      if (this.disabled.from && date.getTime() > this.norm(this.disabled.from).getTime()) {
        return true
      }

      if (this.disabled.days) {
        return this.disabled.days.indexOf(date.getDay()) !== -1
      }

      return false
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

      if (!this.highlighted) {
        return false
      }

      let highlighted = false

      if (this.highlighted.dates) {
        this.highlighted.dates.forEach(d => {
          if (date.toDateString() === d.toDateString()) {
            highlighted = true
          }
        })
      }

      if (this.highlighted.from && this.highlighted.to) {
        highlighted = date >= this.highlighted.from && date <= this.highlighted.to
      }

      if (this.highlighted.days && this.highlighted.days.indexOf(date.getDay()) !== -1) {
        highlighted = true
      }
      return highlighted
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

      if (!this.disabled) {
        return false
      }

      if (this.disabled.to) {
        if (
          (date.getMonth() < this.disabled.to.getMonth() && date.getFullYear() <= this.disabled.to.getFullYear()) ||
          date.getFullYear() < this.disabled.to.getFullYear()
        ) {
          disabled = true
        }
      }
      if (this.disabled.from) {
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
     * Whether a year is selected
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
      if (!this.disabled) {
        return false
      }

      if (this.disabled.to) {
        if (date.getFullYear() < this.disabled.to.getFullYear()) {
          disabled = true
        }
      }
      if (this.disabled.from) {
        if (date.getFullYear() > this.disabled.from.getFullYear()) {
          disabled = true
        }
      }

      return disabled
    },

    dayClasses (day) {
      let c = this.cc.svdCalDays.slice()
      let conditionalClasses = {
        [this.cc.svdDayBlank]: day.isBlank,
        [this.cc.svdDaySelected]: day.isSelected,
        [this.cc.svdDayDisabled]: day.isDisabled,
        [this.cc.svdDayHighlighted]: day.isHighlighted,
        [this.cc.svdDayToday]: day.isToday
      }
      c.push(conditionalClasses)
      return c
    }

  },

  mounted () {
    if (this.value) {
      let d = this.getDateObject(this.value)
      if (d) {
        this.setDate(d.getTime())
        this.setViewDate(d)
      } else {
        this.setViewDate(new Date())
      }
    } else if (this.initialViewDate) {
      this.setViewDate(this.getDateObject(this.initialViewDate))
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

.svd-outer
    position relative
    text-align left
    *
        box-sizing border-box

.svd-cal-wrapper
    background white
    width $width
    header
        display flex
        line-height 40px
        span
            display block
            text-align center
            width (100 - (100/7)*2)%

        .svd-cal-prev
        .svd-cal-next
            width (100/7)%
            position relative

        .svd-cal-prev
            &:after
                content '<'
            &.disabled:after
                opacity 0.5
        .svd-cal-next
            &:after
                content '>'
            &.disabled:after
                opacity 0.5
        .svd-cal-prev:not(.disabled)
        .svd-cal-next:not(.disabled)
        .up:not(.disabled)
            cursor pointer
            &:hover
                background #ddd

    .svd-day-disabled
    .disabled
        color #ddd
        cursor default

    .svd-cal-cell
        display inline-block
        width (100/7)%
        height 40px
        line-height 40px
        text-align center
        vertical-align middle
        border 1px solid transparent
        &:not(.blank):not(.disabled).svd-cal-day
            cursor pointer
            &:hover
                border-color #ddd
        &.svd-day-selected
            background #ddd
            &.highlighted
                background #ddd
        &.svd-day-highlighted
            background #ddd
        &.grey
            color #888

            &:hover
                background inherit
        &.svd-day-today
            border-color #ddd

        &.svd-cal-day-names
            font-size 75%
            white-space nowrap

    .month,
    .year
        width 33.333%

.vdp-datepicker__clear-button, .vdp-datepicker__calendar-button
    cursor pointer
    font-style normal
</style>
