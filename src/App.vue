<template>
  <div v-if="!settings.showSetup" class="orientation">
      Please turn your device to landscape mode
  </div>
  <div v-if="settings.showSetup" class="container two-columns">
    <div class="setup-description"><label for="left-time">Left</label></div>
    <div class="clock-setup">
      <div class="text-align-center">
        <input type="text" id="left-time" class="text-align-center" v-model="leftTime">
      </div>
    </div>
    <div class="setup-description"><label for="right-time">Right</label></div>
    <div class="clock-setup">
      <div class="text-align-center">
        <input type="text" id="right-time" class="text-align-center" v-model="rightTime">
      </div>
    </div>
    <div/>
    <div class="setup-description">
      <button @click="toggleSound">{{ sound }}</button>
    </div>
    <div/>
    <div class="setup-description">
      <button @click="changeTheme">{{ theme }}</button>
    </div>
    <div/>
    <div class="setup-description">
      <button @click="toggleSetup">Back</button>
    </div>
  </div>
  <div v-else class="container three-columns">
    <div class="clock" @click="startRight">
      <span class="left-clock place-self-center">
        {{ leftClock() }}
      </span>
    </div>
    <div class="setup">
      <button @click="pause">Pause</button>
      <button @click="reset">Reset</button>
      <button @click="toggleSetup">Setup</button>
    </div>
    <div class="clock" @click="startLeft">
      <span class="right-clock place-self-center">
        {{ rightClock() }}
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Ref } from 'vue'
import click from './assets/click.mp3'

type Clock = {
  minutes: number
  seconds: number
  milliseconds: number
}

const audio = new Audio(click)

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
  sound: true,
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

function validTime(time: string): boolean {
  return (/^([0-9]{1,}):[0-5][0-9]$/).test(time)
}

const leftTime = computed({
  get() {
    return toString(left)
  },
  set(time: string) {
    if (validTime(time)) {
      const timeSplit = time.split(':')
      settings.value.left.minutes = Number(timeSplit[0])
      settings.value.left.seconds = Number(timeSplit[1])
    }
  }
})

const rightTime = computed({
  get() {
    return toString(right)
  },
  set(time: string) {
    if (validTime(time)) {
      const timeSplit = time.split(':')
      settings.value.right.minutes = Number(timeSplit[0])
      settings.value.right.seconds = Number(timeSplit[1])
    }
  }
})

const theme = computed({
  get() {
    if (settings.value.theme === 'light') {
      return 'Dark ???'
    }
    return 'Light ???'
  },
  set(theme: string) {
    settings.value.theme = theme
  }
})

const sound = computed(() => {
  if (settings.value.sound === true) {
      return 'Sound on'
    }
  return 'Sound off'
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
  if (settings.value.sound) {
    audio.load()
    audio.play()
  }
  pause()
  timer = setInterval(() => tick(left.value), 10)
}

function startRight() {
  if (settings.value.sound) {
    audio.load()
    audio.play()
  }
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
  pause()
  settings.value.showSetup = !settings.value.showSetup
  left.value.minutes = settings.value.left.minutes
  left.value.seconds = settings.value.left.seconds
  right.value.minutes = settings.value.right.minutes
  right.value.seconds = settings.value.right.seconds
}

function toggleSound() {
  settings.value.sound = !settings.value.sound
}
</script>