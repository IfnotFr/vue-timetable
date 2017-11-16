<template>
  <div class="vue-timetable vue-timetable-list-mode">
    <div v-for="day in date.list" v-if="day.events.length > 0" class="vue-timetable-day-container">
      <slot name="header" :day="day">
        <header>
          {{ day.date | moment("dddd DD") }}
          {{ day.date | moment("MMMM") }}
          <span>{{ day.date | moment("DD/MM/YYYY") }}</span>
        </header>
      </slot>
      <ul>
        <li v-for="event in day.events">
          <span class="vue-timetable-event-color" :style="{ backgroundColor: event.color }"></span>
          <span class="vue-timetable-event-link">
            <strong class="vue-timetable-event-time">{{ event.date.from | moment('HH:mm') }}</strong>
            <span class="vue-timetable-event-name">
              <slot :event="event">
                {{ event.text }}
              </slot>
            </span>
          </span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import moment from 'moment'

  export default {
    name: 'list-timetable',
    props: ['from', 'to', 'events', 'reverse'],
    data () {
      return {
        date: {
          from: moment(this.from),
          to: typeof this.to === 'undefined' ? moment().add(30, 'days') : moment(this.to),
          list: []
        }
      }
    },
    watch: {
      events () {
        this.fillDaysList()
      },
      from () {
        this.date.from = moment(this.from)
        this.fillDaysList()
      },
      to () {
        this.date.to = moment(this.to)
        this.fillDaysList()
      }
    },
    methods: {
      fillDaysList () {
        this.date.list = []

        let now = this.date.from.clone()
        while (now.isBefore(this.date.to) || now.isSame(this.date.to)) {
          this.date.list.push({
            date: now.format(),
            events: this.getDayEvents(now)
          })
          now.add(1, 'days')
        }

        if (this.reverse) {
          this.date.list.reverse()
        }
      },
      getDayEvents (day) {
        if (typeof this.events === 'undefined') {
          return []
        }

        return this.events.filter((event) => {
          return day.isSame(event.date.from, 'day')
        }).sort((a, b) => {
          return moment(b.date.from).isBefore(a.date.from)
        })
      }
    },
    created () {
      this.fillDaysList()
    }
  }
</script>
