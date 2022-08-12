<template>
  <div class="orientation">
      Please turn your device to landscape mode
  </div>
  <div class="container">
    <div v-if="settings.showSetup" class="clock-setup">
      <div class="text-align-center">
        <input type="number" class="minutes" v-model="settings.left.minutes">:<input type="number" v-model="settings.left.seconds">
      </div>
    </div>
    <div v-else class="clock" @click="startRight">
      <span class="left-clock place-self-center">
        {{ leftClock() }}
      </span>
    </div>
    <div>
      <button @click="changeTheme">{{ theme.charAt(0).toUpperCase() + theme.slice(1) }}</button>
      <button @click="toggleSetup">Setup</button>
      <button @click="pause">Pause</button>
      <button @click="reset">Reset</button>
    </div>
    <div v-if="settings.showSetup" class="clock-setup">
      <div class="text-align-center">
        <input type="number" class="minutes" v-model="settings.right.minutes">:<input type="number" v-model="settings.right.seconds">
      </div>
    </div>
    <div v-else class="clock" @click="startLeft">
      <span class="right-clock place-self-center">
        {{ rightClock() }}
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Ref } from 'vue'

type Clock = {
  minutes: number
  seconds: number
  milliseconds: number
}

const defaultMinutes = 7

const settings = ref({
  left: {
    minutes: defaultMinutes,
    seconds: 0
  },
  right: {
    minutes: defaultMinutes,
    seconds: 0
    },
  theme: 'light',
  showSetup: false
})
const left = ref({
  minutes: defaultMinutes,
  seconds: 0,
  milliseconds: 0
})
const right = ref({
  minutes: defaultMinutes,
  seconds: 0,
  milliseconds: 0
})
let timer = 0

const theme = computed({
  get() {
    if (settings.value.theme === 'light') {
      return 'Dark'
    }
    return 'Light'
  },
  set(theme: string) {
    settings.value.theme = theme
  },
})

function toString(clock: Ref<Clock>) {
  return (clock.value.minutes < 10 ? `0${clock.value.minutes}` : `${clock.value.minutes}`)
    + ':'
    + (clock.value.seconds < 10 ? `0${clock.value.seconds}` : `${clock.value.seconds}`)
}

function leftClock() {
  return toString(left)
}

function rightClock() {
  return toString(right)
}

function tick(clock: Clock) {
  clock.milliseconds = clock.milliseconds - 10
  if (clock.milliseconds < 0) {
    clock.milliseconds = 1000
    clock.seconds--
  }
  if (clock.seconds < 0) {
    clock.seconds = 59
    clock.minutes--
  }
  // No time left
  if (clock.minutes < 0) {
    clock.seconds = 0
    clock.minutes = 0
    clearInterval(timer)
  }
}

function pause() {
  clearInterval(timer)
}

function reset() {
  pause()
  left.value = { ...settings.value.left, milliseconds: 0 }
  right.value = { ...settings.value.right, milliseconds: 0 }
}

function startLeft() {
  pause()
  timer = setInterval(() => tick(left.value), 10)
}

function startRight() {
  pause()
  timer = setInterval(() => tick(right.value), 10)
}

function changeTheme() {
  const root = document.querySelector(':root') as HTMLElement;
  if (settings.value.theme === 'light') {
    settings.value.theme = 'dark'
    root.style.setProperty('--color-background', '#1a1a1a')
    root.style.setProperty('--color-text', '#fefefe')
  }
  else {
    settings.value.theme = 'light'
    root.style.setProperty('--color-background', '#fefefe')
    root.style.setProperty('--color-text', '#1a1a1a')
  }
}

function toggleSetup() {
  settings.value.showSetup = !settings.value.showSetup
  left.value.minutes = settings.value.left.minutes
  left.value.seconds = settings.value.left.seconds
}
</script>