<template>
  <div id="app">

    <h1>Datepicker Examples</h1>
    <!--
    <div class="example">
      <h3>Default datepicker</h3>
      <datepicker placeholder="Select Date"></datepicker>
      <code>
          &lt;datepicker&gt;&lt;/datepicker&gt;
      </code>
    </div>

    <div class="example">
      <h3>Format datepicker</h3>
      <datepicker :format="format"></datepicker>
      <code>
        &lt;datepicker :format="format"&gt;&lt;/datepicker&gt;
      </code>
      <div class="settings">
        <h5>Settings</h5>
        <div class="form-group">
          <label>Format</label>
          <select v-model="format">
            <option value="d MMM yyyy" selected>d MMM yyyy - e.g 12 Feb 2016</option>
            <option value="d MMMM yyyy">d MMMM yyyy - e.g 12 February 2016</option>
            <option value="yyyy-MM-dd">yyyy-MM-dd - e.g 2016-02-12</option>
            <option value="dsu MMM yyyy">dsu MMM yyyy - e.g 12th Feb 2016</option>
            <option value="D dsu MMM yyyy">D dsu MMM yyyy - e.g Sat 12th Feb 2016</option>
          </select>
        </div>
      </div>
    </div>

    <div class="example">
      <h3>With minimum and maximum date range</h3>
      <datepicker :disabled="disabled"></datepicker>
      <code>
        &lt;datepicker :disabled="disabled"&gt;&lt;/datepicker&gt;
      </code>
      <div class="settings">
        <h5>Settings</h5>
        <div class="form-group">
          <label>Disabled to:</label>
          <datepicker v-on:selected="disableTo"></datepicker>
        </div>
        <div class="form-group">
          <label>Disabled from:</label>
          <datepicker v-on:selected="disableFrom"></datepicker>
        </div>
        <pre>disabled: {{ disabled }}</pre>
      </div>
    </div>

    <div class="example">
      <h3>Translations</h3>
      <h5>{{ languages[language].language }} datepicker</h5>

      <datepicker :language="language" format="d MMMM yyyy"></datepicker>
      <code>
          &lt;datepicker language="{{ language }}"&gt;&lt;/datepicker&gt;
      </code>
      <div class="settings">
        <h5>Settings</h5>
        <select v-model="language">
          <option :value="key" v-for="(language, key) in languages">{{ language.language }}</option>
        </select>
      </div>
    </div>
    -->
    <div class="example">
      <h3>Inline datepicker</h3>
      <datepicker
        v-model="date"
        :inline="true"
        :mondayFirst="true"
        :initialViewDate="initialViewDate"
        :disabled="disabled"
        @selected="dateClicked"
      ></datepicker>
    </div>
    <div>
      {{ panelTitle }}
    </div>

  </div>
</template>

<script>
import Datepicker from '@/components/Datepicker'
import DateLanguages from '@/utils/DateLanguages'

const state = {
  date1: new Date()
}

export default {
  name: 'app',
  components: {
    Datepicker
  },
  data () {
    return {
      format: 'd MMMM yyyy',
      eventMsg: null,
      state: state,
      language: 'en',
      languages: DateLanguages.translations,
      initialViewDate: null,
      selectedDate: null,
      date: null,
      disabled: {
        days: [0],
        to: new Date(),
        from: this.endDate(),
        dates: this.foldSlots([
          new Date(1499954400000),
          new Date(1499943600000),
          new Date(1499950800000),
          new Date(1499947200000),
          new Date(1499958000000),
          new Date(1499961600000),
          new Date(1499965200000),
          new Date(1499968800000)
        ])
      }
    }
  },
  computed: {
    panelTitle () {
      return this.selectedDate ? this.selectedDate.toDateString() : 'Select a date to see available booking times'
    }
  },

  methods: {
    dateClicked (date) {
      this.selectedDate = date
    },

    endDate () {
      let d = new Date()
      d.setMonth((d.getMonth() + 3) % 12)
      return d
    },

    foldSlots (timestamps) {
      let bookedSlotsMap = {}
      if (!timestamps) return bookedSlotsMap

      timestamps.forEach(s => {
        let d = new Date(s)
        let text = d.toISOString()
        let date = text.slice(0, 10)
        let slot = text.slice(11, 16)
        if (!(date in bookedSlotsMap)) {
          bookedSlotsMap[date] = []
        }
        bookedSlotsMap[date].push(slot)
      })

      let fullyBookedDates = []
      for (let d in bookedSlotsMap) {
        if (bookedSlotsMap[d].length === 8) {
          let ymd = d.split('-')
          fullyBookedDates.push(new Date(ymd[0], ymd[1] - 1, ymd[2]))
        }
      }
      return fullyBookedDates
    }

  },

  watch: {
    date (newDate, oldDate) {
      console.log('demo watch date', newDate)
    }
  }
}
</script>

<style>

body {
    font-family: 'Helvetica Neue Light', Helvetica, sans-serif;
    padding: 1em 2em 2em;
}
input, select {
    padding: .75em .5em;
    font-size: 100%;
    border: 1px solid #ccc;
    width: 100%
}

select {
    height: 2.5em;
}

.example {
    background: #f2f2f2;
    border: 1px solid #ddd;
    padding: 0em 1em 1em;
    margin-bottom: 2em;
}

code,
pre {
    margin: 1em 0;
    padding: 1em;
    border: 1px solid #bbb;
    display: block;
    background: #ddd;
    border-radius: 3px;
}

.settings {
    margin: 2em 0;
    border-top : 1px solid #bbb;
    background: #eee;
}

h5 {
    font-size:100%;
    padding: 0;
}

.form-group {
    margin-bottom: 1em;
}

.form-group label {
    font-size: 80%;
    display: block;
}
</style>
