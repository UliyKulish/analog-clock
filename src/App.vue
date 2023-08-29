<script setup lang="ts">
import AnalogClock from "@/components/clock/AnalogClock.vue";
import TextClock from "@/components/clock/TextClock.vue";
import type {TimeType} from "@/types/clock/TimeType";
import {ref, onMounted, reactive} from "vue";
let tick = ref<number>(0)
let time = reactive<TimeType>({ hours: 0, minutes: 0, seconds: 0 })
const updateTime = (date: Date) => {
  tick.value++
  time = {
    hours: date.getHours(),
    minutes: date.getMinutes(),
    seconds: date.getSeconds()
  }

  setTimeout(() => updateTime(new Date()), 1000 - new Date().getMilliseconds())
}
onMounted(() => {
  updateTime(new Date())
})
</script>

<template>
  <main id="clock">
    <template v-if="tick">
      <analog-clock :minute="time.minutes" :tick="tick"></analog-clock>
      <text-clock :time="time"></text-clock>
    </template>
  </main>
</template>