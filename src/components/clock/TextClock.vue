<script setup lang="ts">
import { computed } from 'vue'
import type { TimeType } from '@/types/clock/TimeType'
import type { MeridiemType } from '@/types/clock/MeridiemType'
const timeMap = {
  20: 'twenty',
  30: 'thirty',
  40: 'forty',
  50: 'fifty',
  0: [null, 'one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten'],
  10: [
    null,
    'eleven',
    'twelve',
    'thirteen',
    'fourteen',
    'quarter',
    'sixteen',
    'seventeen',
    'eighteen',
    'nineteen'
  ],
  hours: [
    null,
    'one',
    'two',
    'three',
    'four',
    'five',
    'six',
    'seven',
    'eight',
    'nine',
    'ten',
    'eleven',
    'twelve'
  ]
}
const parsers = [
  {
    test: (n) => !n,
    parse: () => ({ m: `o'clock` })
  },
  {
    test: (n) => n <= 10,
    parse: (n) => ({ r: 1, m: `${timeMap[0][n]} ${!(n % 5) ? 'after' : 'past'}` })
  },
  {
    test: (n) => ~[15, 30, 45].indexOf(n),
    parse: (n) => ({ r: 1, m: `${n === 30 ? 'half' : 'quarter'} ${n <= 30 ? 'past' : 'to'}` })
  },
  {
    test: (n) => ~[50, 55].indexOf(n),
    parse: (n) => ({ r: 1, m: `${timeMap[0][-(n % 10) + 10]} to` })
  },
  {
    test: (n) => n < 20,
    parse: (n) => ({ m: timeMap[10][n % 10] })
  },
  {
    test: (n) => n,
    parse: (n) => ({
      m: `${timeMap[((n * 0.1) | 0) * 10]}${n % 10 ? `-${timeMap[0][n % 10]}` : ''}`
    })
  }
]
const props = defineProps({
  time: Object as TimeType
})
let meridiem = computed((): MeridiemType => {
  return props.time.hours < 12 ? 'a' : 'p'
})
let output = computed((): string[] => {
  const { hours: h, minutes: m } = props.time
  const { r: reverse, m: minutes }: { r?: any; m: string } = parsers
    .find(({ test }) => test(m))
    .parse(m)

  const hours = [h]
    .map((h) => (h > 12 ? h - 12 : h || 12))
    .map((h) => (/\sto$/.test(minutes) ? h + 1 : h))
    .map((h) => (h > 12 ? 1 : h))
    .map((h) => timeMap.hours[h])
    .reduce((h) => h)

  return reverse ? [minutes, hours] : [hours, minutes]
})
</script>

<template>
  <output class="text-clock">
    <hgroup class="text-clock__col -meridiem">
      <h2>{{ meridiem }}</h2>
      <h2>m</h2>
    </hgroup>

    <time class="text-clock__col">
      <h3 v-for="(part, i) in output" :key="i" class="text-clock__row">{{ part }}</h3>
    </time>
  </output>
</template>

<style scoped lang="scss">
.text-clock {
  display: flex;
  font-size: 3rem;
  color: #99dcf9;
  white-space: nowrap;

  &__col {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    text-align: center;
    line-height: 1.1;
    padding: 0.5rem 0.5rem 0.65rem;
    &.-meridiem {
      text-transform: uppercase;
      font-size: 0.85em;
      background-color: #7392a0;
      color: white;
      font-weight: 500;
    }
  }

  &__row {
    flex: 1 0 0;
    display: flex;
    align-items: center;
  }
}
</style>
