<template>
  <header>
    <div>
      <h1>Activity Tracker</h1> 
    </div>
    <button class="success" @click="exportToCSV">Export</button>
  </header>

  <main v-if="activity">
    <div class="period-container">
      <label>Period: ({{ activity.days.length }} days): </label>
      <div class="period-input">
        <VueDatePicker
          v-model="period"
          range
          :clearable="false"
          :enable-time-picker="false"
          :dark="true"
          :format-locale="enGB"
        />
      </div>
      <button class="danger" @click="updatePeriod"> Set Period</button>
    </div>
    <div class="overview">
      <div class="totals" v-html="totals"></div>
      <hr />
      <div class="goals">
        <div>
          <label>Target: </label>
          <input type="number" v-model="goalHrs"/>
        </div>
        <div v-html="goals"></div>
      </div>
    </div>
    <div class="activity-wrapper">
      <DayComponent
        v-for="d in activity.days"
        :day="d"
        @update-activity="updateActivity"
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
      activity: null,
      goalHrs: 100
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
  computed: {
    totals () {
      const hrs = this.activity.days.reduce((sum, day) => {
        return sum + day.entries.reduce((entrySum, e) => entrySum + (e.hrs || 0), 0)
      }, 0)

      const earnings = this.activity.days.reduce((sum, day) => {
        return sum + day.entries.reduce((entrySum, e) => entrySum + ((e.hrs || 0) * (e.rate || 0)), 0)
      }, 0)

      return `Hours: <strong>${hrs}</strong>, Earnings: <strong>$${earnings.toFixed(2)}</strong>`
    },
    goals () {
      const diffTime = Math.abs(new Date(this.period[1]) - new Date())
      const daysRemaining = Math.ceil(diffTime / 1000 / 60 / 60 / 24) - this.weekendCount(new Date())
      const hrs = this.activity.days.reduce((sum, day) => {
        return sum + day.entries.reduce((entrySum, e) => entrySum + (e.hrs || 0), 0)
      }, 0)
      const avgNeeded = ((this.goalHrs - hrs) / daysRemaining).toFixed(1)

      return `Days remaining: ${daysRemaining}. Avg needed: ${avgNeeded}`
    }
  },
  methods: {
    weekendCount (from) {
      const start = new Date(from)
      const end = new Date(this.period[1])
      let count = 0

      start.setHours(0, 0, 0, 0)
      end.setHours(0, 0, 0, 0)

      for (let d = new Date(start); d <= end; d.setDate(d.getDate() + 1)) {
        const day = d.getDay()
        if (day === 0 || day === 6) {
          count++
        }
      }

      return count
    },
    startNewActivity (from = new Date(), days = 30) {
      localStorage.removeItem('mf_activity_tracker')
      this.activity = null
      this.period = [from, new Date(new Date().setDate(from.getDate() + days))]

      const daysArr = []
      for (let i = 0; i < (days + 1); i++) {
        const date = new Date(new Date().setDate(from.getDate() + i))
        daysArr.push({
          date,
          dateStr: date.getDate() + '/' + (date.getMonth() + 1),
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
    updatePeriod () {
      if (this.period[0] && this.period[1] && confirm('Are you sure you want to CLEAR the current period and start a new one?')) {
        const diffTime = Math.abs(new Date(this.period[1]) - new Date(this.period[0]))
        const diffDays = Math.ceil(diffTime / 1000 / 60 / 60 / 24)
        this.startNewActivity(new Date(this.period[0]), diffDays)
      }
    },
    clearPeriod () {
      if (confirm('Are you sure you want to clear all work items for this period?')) {
        this.startNewActivity()
      }
    },
    updateActivity (activity) {
      localStorage.setItem('mf_activity_tracker', JSON.stringify(activity))
      this.activity = activity
    },
    exportToCSV () {
      const rows = [['Date', 'Descriptions', 'Hours', 'Rate', 'Total']]
      this.activity.days.forEach(day => {
        if (!day.entries || day.entries.length === 0) {
          return
        }
        const descriptions = day.entries.map(e => e.desc).join('\n')
        const rate = day.entries[0]?.rate || 0
        const hrs = day.entries.reduce((sum, e) => sum + (e.hrs || 0), 0)
        const total = day.entries.reduce((sum, e) => sum + (parseFloat(e.total) || 0), 0)

        rows.push([day.dateStr, descriptions, hrs, rate, total.toFixed(2)])
      })

      const csvContent = rows.map(r => r.map(v => `"${v}"`).join(',')).join('\r\n')

      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
      const url = URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.setAttribute('href', url)
      link.setAttribute('download', 'activity.csv')
      link.style.visibility = 'hidden'
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
    }

  }
}
</script>

<style scoped>
  header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .overview {
    margin: 2em 0;
    .goals {
      display: flex;
      align-items: center;
      > div {
        margin-right: 0.5em;
      }
    }
  }
  hr {
    margin: 1em 0;
  }
  .period-container {
    display: flex;
    align-items: center;
    .period-input {
      width: 250px;
      margin: 0 10px;
    }
  }
</style>
