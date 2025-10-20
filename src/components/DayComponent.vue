<template>
  <div class="day-container">
    <div class="top-row">
      <h3>{{ day.dateStr }}</h3>
      <a @click="showing = !showing">{{ showing ? 'Hide' : 'Show' }}</a>
    </div>
    <div v-show="showing" class="entries-contain">
      <div class="controls">
        <div class="totals">{{ totals }}</div>
        <button class="success" @click="newEntry = JSON.parse(JSON.stringify(baseEntry))">New Entry</button>
      </div>
      <div v-if="newEntry" class="new-entry">
        <div class="input-contain">
          <div>Description</div>
          <textarea type="text" name="desc" id="desc" v-model="newEntry.desc"/>
        </div>
        <div class="input-contain">
          <label>
            From
            <VueDatePicker
              v-model="newEntry.from"
              :clearable="false"
              :time-picker="true"
              :dark="true"
            />
          </label>
        </div>
        <div class="input-contain">
          <label>
            To
            <VueDatePicker
              v-model="newEntry.to"
              :clearable="false"
              :time-picker="true"
              :dark="true"
            />
          </label>
        </div>
        <div class="input-contain">
          <div>Rate</div>
          <input type="number" name="rate" id="rate" v-model="newEntry.rate"/>
        </div>
        <div class="buttons-contain">
          <button class="success" @click="createEntry">Create</button>
          <button class="danger" @click="newEntry = null">Cancel</button>
        </div>
      </div>
      <table v-if="day.entries.length" class="entries-wrapper">
        <thead>
          <tr>
            <th>Description</th>
            <th>From</th>
            <th>To</th>
            <th>Hours</th>
            <th>Rate</th>
            <th>Total</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <EntryComponent
            v-for="(e, i) in day.entries"
            :entry="e"
            :index="i"
            @delete-entry="deleteEntry(i)"
          />
        </tbody>
      </table>
      <em v-else>No entries for this day</em>
    </div>
  </div>
</template>

<script>
import EntryComponent from './EntryComponent.vue'
import VueDatePicker from '@vuepic/vue-datepicker'
import '@vuepic/vue-datepicker/dist/main.css'

export default {
  emits: ['update-activity'],
  components: {
    VueDatePicker,
    EntryComponent
  },
  props: {
    day: {}
  },
  data () {
    return {
      showing: false,
      newEntry: null,
      baseEntry: {
        desc: '',
        from: null,
        to: null,
        hrs: 0,
        rate: 75,
        total: 0
      }
    }
  },
  mounted () {
    if (this.isToday()) {
      this.showing = true
    }
  },
  computed: {
    totals () {
      const hrs = this.day.entries.reduce((sum, e) => sum + (e.hrs || 0), 0)
      const earnings = this.day.entries.reduce((sum, e) => sum + ((e.hrs || 0) * (e.rate || 0)), 0)

      return `Hours: ${hrs}, Earnings: $${earnings.toFixed(2)}`
    }
  },
  methods: {
    isToday () {
      const today = new Date()
      today.setHours(0, 0, 0, 0)

      const check = new Date(this.day.date)
      check.setHours(0, 0, 0, 0)

      return check.getTime() === today.getTime()
    },
    createEntry () {
      if (this.newEntry.desc && this.newEntry.from && this.newEntry.to) {
        const fromHrs = this.newEntry.from.hours + (this.newEntry.from.minutes / 60)
        const toHrs = this.newEntry.to.hours + (this.newEntry.to.minutes / 60)
        this.newEntry.hrs = parseFloat((toHrs - fromHrs).toFixed(1))
        this.newEntry.total = `${(this.newEntry.hrs * this.newEntry.rate).toFixed(2)}`

        const activity = JSON.parse(localStorage.getItem('mf_activity_tracker'))
        activity.days.forEach(d => {
          if (d.date === this.day.date) {
            d.entries.push(this.newEntry)
          }
        })
        this.newEntry = null
        this.$emit('update-activity', activity)
      }
    },
    deleteEntry (index) {
      if (confirm('Are you sure you want to delete this entry?')) {
        const activity = JSON.parse(localStorage.getItem('mf_activity_tracker'))
        activity.days.forEach(d => {
          if (d.date === this.day.date) {
            d.entries.splice(index, 1)
          }
        })
        this.$emit('update-activity', activity)
      }
    }
  }
}
</script>

<style scoped>
  .day-container {
    margin: 1em 0;
    .top-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      border-bottom: 1px solid hsla(160, 100%, 37%, 1);
      padding: 0.5em 0;
    }
    .entries-contain {
      padding-left: 1em;
      .controls {
        margin: 1em 0;
        display: flex;
        align-items: center;
        justify-content: space-between;
      }
      .new-entry {
        margin-bottom: 1em;
        padding: 0.5em;
        border: 1px solid #2f2f2f;
        border-radius: 0.5em;
        .buttons-contain {
          > button {
            margin-right: 0.5em;
          }
        }
      }
      .entries-wrapper {
        margin-top: 1em;
        width: 100%;
        border: 1px solid #2f2f2f;
      }
    }
  }
</style>