<template>
  <header>
    <div>
      <h1>Activity Tracker</h1>
    </div>
  </header>

  <main v-if="activity">
    <div class="period-container">
      <label>Period: ({{ activity.days.length }} days): </label>
      <div class="period-input">
        <VueDatePicker
          :model-value="period"
          range
          :clearable="false"
          :enable-time-picker="false"
          :dark="true"
          :format-locale="enGB"
          @update:model-value="updatePeriod"
        />
      </div>
      <button class="danger" @click="clearPeriod"> Clear Period</button>
    </div>
    <div class="activity-wrapper">
      <DayComponent
        v-for="d in activity.days"
        :day="d"
      />
    </div>
  </main>
</template>

<script>
import { enGB } from 'date-fns/locale'
import VueDatePicker from '@vuepic/vue-datepicker'
import '@vuepic/vue-datepicker/dist/main.css'
import DayComponent from './components/DayComponent.vue'

export default {
  components: {
    DayComponent,
    VueDatePicker
  },
  data () {
    return {
      enGB,
      period: null,
      activity: null
    }
  },
  mounted () {
    let data = localStorage.getItem('mf_activity_tracker')
    if (data) {
      this.activity = JSON.parse(data)
      this.period = this.activity.period
    } else {
      this.startNewActivity()
    }
  },
  methods: {
    startNewActivity (from = new Date(), days = 30) {
      localStorage.removeItem('mf_activity_tracker')
      this.activity = null
      this.period = [from, new Date(new Date().setDate(from.getDate() + days))]

      const daysArr = []
      for (let i = 0; i < (days + 1); i++) {
        const date = new Date(new Date().setDate(from.getDate() + i))
        daysArr.push({
          date,
          dateStr: date.getDate() + '/' + date.getMonth(),
          entries: []
        })
      }
      const toStore = {
        period: this.period,
        days: daysArr
      }
      localStorage.setItem('mf_activity_tracker', JSON.stringify(toStore))
      this.activity = toStore
    },
    updatePeriod (val) {
      if (val[0] && val[1]) {
        const diffTime = Math.abs(val[1] - val[0])
        const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
        this.startNewActivity(val[0], diffDays)
      }
    },
    clearPeriod () {
      if (confirm('Are you sure you want to clear all work items for this period?')) {
        this.startNewActivity()
      }
    }
  }
}
</script>

<style scoped>
  .period-container {
    display: flex;
    align-items: center;
    .period-input {
      width: 250px;
      margin: 0 10px;
    }
  }
</style>
