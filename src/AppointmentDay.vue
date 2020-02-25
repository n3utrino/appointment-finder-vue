<template>
    <li class="appointments__day list-item">
        <div  class="appointments__day-header">
            <span>{{relativeDate}} am&nbsp;</span>
            <br/>
            <span>{{$d(day.date,'short')}}</span>
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
</template>

<script lang="ts">
    import {Component, Prop, Vue} from "vue-property-decorator";

    export interface Slot {
        id: number;
        from: Date;
        to: Date;
        day: string;
        available: boolean;
    }

    export interface Day {
        id: number;
        date: Date;
        slots: Slot[];
    }

    @Component({})
    export default class AppointmentDay extends Vue {

        @Prop({required: true}) day!: Day;
        @Prop({required: true}) optimalHours!: number[];

        get relativeDate() {
            const offset = (this.day.date.getTime() - new Date().getTime()) / 1000;

            let minute = 60,
                hour = minute * 60,
                day = hour * 24,
                week = day * 7;

            const days = Math.round(offset / day);
            if (days == 1) {
                return `Morgen`;
            }
            if (days == 7) {
                return `In einer Woche`;
            }
            if (offset > day) {
                return `In ${days} Tagen`;
            }
            return "Heute";
        }

    }

</script>