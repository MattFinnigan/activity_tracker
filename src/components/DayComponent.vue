<template>
  <div class="day-container">
    <div class="top-row">
      <h3>{{ day.dateStr }}</h3>
      <a @click="showing = !showing">{{ showing ? 'Hide' : 'Show' }}</a>
    </div>
    <div v-show="showing" class="entries-contain">
      <div class="controls">
        <div class="totals">{{ totals }}</div>
        <button class="success" @click="newEntry = baseEntry">New Entry</button>
      </div>
      <div v-if="newEntry" class="new-entry">
        <div class="input-contain">
          <div>Description</div>
          <input type="text" name="desc" id="desc" v-model="newEntry.desc"/>
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
      <EntryComponent
        v-for="(e, i) in day.entries"
        :entry="e"
        :index="i"
      />
      <em v-if="!day.entries.length">No entries for this day</em>
    </div>
  </div>
</template>

<script>
import EntryComponent from './EntryComponent.vue'
import VueDatePicker from '@vuepic/vue-datepicker'
import '@vuepic/vue-datepicker/dist/main.css'

export default {
  components: {
    VueDatePicker,
    EntryComponent
  },
  props: {
    day: {}
  },
  data () {
    return {
      showing: true,
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
    if (this.isPast()) {
      this.showing = false
    }
  },
  computed: {
    totals () {
      return 'Totals: '
    }
  },
  methods: {
    isPast () {
      const today = new Date()
      today.setHours(0, 0, 0, 0)

      const check = new Date(this.day.date)
      check.setHours(0, 0, 0, 0)

      return check.getTime() < today.getTime()
    },
    createEntry () {
      if (this.newEntry.desc && this.newEntry.from && this.newEntry.to) {
        const fromHrs = this.newEntry.from.hours + (this.newEntry.from.minutes / 60)
        const toHrs = this.newEntry.to.hours + (this.newEntry.to.minutes / 60)
        this.newEntry.hrs = toHrs - fromHrs
        this.newEntry.total = this.newEntry.hrs * this.newEntry.rate
        this.newEntry = null
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
      padding-top: 0.5em;
      padding-left: 1em;
      .controls {
        display: flex;
        align-items: center;
        justify-content: space-between;
      }
      .new-entry {
        margin: 0.5em 0;
        padding: 0.5em;
        border: 1px solid grey;
        border-radius: 0.5em;
        .buttons-contain {
          > button {
            margin-right: 0.5em;
          }
        }
      }
    }
  }
</style>