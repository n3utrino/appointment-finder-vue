<template>
  <div id="app">
    <h1>{{searchDescription}}</h1>
    <div class="appointments">
      <div class="appointments__day">
        <span class="appointments__day-header">Optimalen Tag wählen</span>
        <ul class="appointments__slots">
          <li
            v-for="(selection,index) in dayFilters"
            v-bind:key="index"
            @click="toggleDay(index)"
            class="appointments__slot appointments__slot-time-selection"
            :class="{'appointments__slot-time-selection--active':optimalDays.includes(index)}"
          >
            <span>{{selection}}</span>
          </li>
        </ul>
      </div>
      <div class="appointments__day">
        <span class="appointments__day-header">Optimale Zeit wählen <br/>+/- 1h</span>
        <ul class="appointments__slots">
          <li
            v-for="selection in selections"
            v-bind:key="selection.time"
            @click="toggleTime(selection)"
            class="appointments__slot appointments__slot-time-selection"
            :class="{'appointments__slot-time-selection--active':optimalHours.includes(selection.time)}"
          >
            <span>{{selection.time}}:00</span>
          </li>
        </ul>
      </div>
      <transition-group name="list" tag="ul" class="appointments__day--list">
        <li class="appointments__day list-item" v-for="day in availableDays" v-bind:key="day.id">
          <div class="appointments__day-header">
            <span>{{day.date | relativeFilter}} am&nbsp;</span>
            <br />
            <span @click="toggleDay(day.date)">{{day.date | dayFilter}}</span>
          </div>
          <ul class="appointments__slots">
            <li
              v-for="slot in day.slots"
              class="appointments__slot"
              v-bind:class="{
              'appointments__slot--available': slot.available,
              'appointments__slot--optimal':  optimalHours.includes(slot.from.getHours())
              }"
              v-bind:key="slot.id"
            >
              <span>Um {{slot.from | timeFilter}} bis {{slot.to | timeFilter}}</span>
            </li>
          </ul>
        </li>
      </transition-group>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import HelloWorld from "./components/HelloWorld.vue";

interface Day {
  id: number;
  date: Date;
  slots: Slot[];
}

interface Slot {
  id: number;
  from: Date;
  to: Date;
  day: string;
  available: boolean;
}

let numberOfSlots = 11;
let slotStart = 8;

const numericHourFormat = new Intl.DateTimeFormat("de-CH", { hour: "numeric" });
const weekdayFormat = new Intl.DateTimeFormat("de-CH", {
  weekday: "long"
});

const dateFormat = new Intl.DateTimeFormat("de-CH", {
  month: "long",
  day: "numeric"
});

let generateDayFilters = () => {
  let weekdayDate = new Date(Date.UTC(2017, 0, 2)); // Any monday
  let weekdays = [];

  for (let i = 0; i < 7; i++) {
    weekdays[weekdayDate.getDay()] = weekdayFormat.format(weekdayDate);
    weekdayDate.setDate(weekdayDate.getDate() + 1);
  }

  return weekdays;
};

let days = generateDayFilters();

let generateSelections = () => {
  let selections = [];
  for (let i = 0; i <= numberOfSlots; i++) {
    selections.push({ time: i + slotStart });
  }
  return selections;
};

let generateDays = (): Day[] => {
  let days: Day[] = [];
  let numberOfDays = 60;

  let today = new Date();
  today.setHours(slotStart);
  today.setMinutes(0);
  today.setSeconds(0);

  let dayFormat = new Intl.DateTimeFormat("de-DE", { weekday: "long" });

  for (let i = 0; i <= numberOfDays; i++) {
    let currentDayDate = new Date(today);
    currentDayDate.setTime(today.getTime() + i * 24 * 60 * 60 * 1000);
    let day: Day = { id: i, slots: [], date: currentDayDate };

    for (let j = 0; j <= numberOfSlots; j++) {
      let startDate = new Date(currentDayDate);
      startDate.setHours(slotStart + j);

      let endDate = new Date(startDate);
      endDate.setHours(startDate.getHours() + 1);

      let available = Math.random() > 0.7;

      day.slots.push(<Slot>{
        id: j,
        day: dayFormat.format(currentDayDate),
        from: startDate,
        to: endDate,
        available: available
      });
    }

    days.push(day);
  }

  return days;
};

const dateFilter = Vue.filter("timeFilter", (date: Date) => {
  return numericHourFormat.format(date);
});

const dayFilter = Vue.filter("dayFilter", (date: Date) => {
  return weekdayFormat.format(date) + " " + dateFormat.format(date);
});

const relativeFilter = Vue.filter("relativeFilter", (date: Date) => {
  const offset = (date.getTime() - new Date().getTime()) / 1000;

  let minute = 60,
    hour = minute * 60,
    day = hour * 24,
    week = day * 7;

  if (Math.round(offset / day) == 1) {
    return `Morgen`;
  }
  if (Math.round(offset / day) == 7) {
    return `In einer Woche`;
  }
  if (offset > day) {
    return `In ${Math.round(offset / day)} Tagen`;
  }
  return "Heute";
});

@Component({
  components: {
    HelloWorld
  }
})
export default class App extends Vue {
  private days: Day[] = generateDays();
  private optimalHours: number[] = [8, 12, 18];
  private optimalDays: number[] = [];

  private dayFilters = days;
  private selections = generateSelections();

  toggleTime(selection: any) {
    let index = this.optimalHours.indexOf(selection.time);

    if (index > -1) {
      this.optimalHours.splice(index, 1);
    } else {
      this.optimalHours.push(selection.time);
    }
  }

  toggleDay(day: number) {
    let index = this.optimalDays.indexOf(day);

    if (index > -1) {
      this.optimalDays.splice(index, 1);
    } else {
      this.optimalDays.push(day);
    }
  }

  get searchDescription(): string {
    let text = "Den nächsten möglichen Termin finden.";

    if (this.optimalHours.length > 0 || this.optimalDays.length > 0) {
      text = "Termin ";

      this.optimalHours.forEach((hour, index) => {
        if (index > 0) {
          text += " oder um ";
        } else if (index == 0) {
          text += " um ";
        }
        text += `${hour}:00`;

        if (index == this.optimalHours.length - 1) {
          text += " Uhr ";
        }
      });

      this.optimalDays.forEach((day, index) => {
        if (index > 0) {
          text += " oder am ";
        } else if (index == 0) {
          text += " am ";
        }
        text += days[index];
      });

      text += " finden";
    }

    return text;
  }

  get availableDays() {
    return this.days
      .filter((day: Day) => {
        let slots = day.slots.filter(
          slot =>
            (this.optimalDays.length == 0 ||
              this.optimalDays.includes(slot.from.getDay())) &&
            (this.optimalHours.length == 0 ||
              this.optimalHours.includes(slot.from.getHours()) ||
              this.optimalHours.includes(slot.from.getHours() + 1) ||
              this.optimalHours.includes(slot.from.getHours() - 1)) &&
            slot.available
        );
        return slots.length > 0;
      })
      .slice(0, 7);
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  box-sizing: content-box;
}

ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.appointments {
  display: flex;

  $selectionColor: rgb(0, 60, 255);

  &__day--list {
    display: flex;
  }

  &__day {
    width: 12rem;
    margin-right: 0.1rem;

    &-header {
      min-height: 2.5rem;
      display: inline-block;
    }
  }

  &__slot {
    background-color: rgba(255, 0, 0, 0.5);
    padding: 0.5rem;
    &--available {
      background-color: rgba(0, 128, 0, 0.5);
    }

    &--optimal {
      color: $selectionColor;
    }
  }

  &__slot-time-selection {
    cursor: pointer;
    border-radius: 4px;
    margin-bottom: 0.25rem;
    padding: 0.38rem;
    width: 10rem;
    background-color: rgb(0, 231, 255);

    &--active {
      color: $selectionColor;
      border-right: 5px solid $selectionColor;
    }

    &:hover {
      color: $selectionColor;
    }
  }
}

.list {
  &-item {
    transition: all 0.5s;
  }

  &-enter,
  &-leave-to {
    transform: translateY(30px);
    opacity: 0;
  }

  &-leave-active {
    transition: none;
    opacity: 0;
    position: absolute;
  }
}
</style>
