<template>
  <div class="wrapper">
    <div class="player" v-if="!isModalVisible">
      <div class="player__top">
        <div class="player-cover">
          <transition-group
              :name="transitionName"
              v-for="(station, i) in stations" :key="i"
          >
            <div
                :class="{ glitch: applyGlitch }"
                class="player-cover__item"
                v-if="i === currentStationIndex"
                :style="{ backgroundImage: `url(${station?.cover})`}"
            >
              <div class="channel r"></div>
              <div class="channel g"></div>
              <div class="channel b"></div>
            </div>
          </transition-group>
        </div>
        <div class="player-controls">

          <a :href="currentStation?.url" target="_blank" class="player-controls__item">
            <svg class="icon">
              <link-icon/>
            </svg>
          </a>

          <div class="player-controls__item" @click="prevTrack">
            <svg class="icon">
              <prev-icon/>
            </svg>
          </div>

          <div class="player-controls__item" @click="nextTrack">
            <svg class="icon">
              <next-icon/>
            </svg>
          </div>

          <div class="player-controls__item" @click="toggleMute">
            <svg class="icon">
              <volume-plus-icon v-if="!isMuted"/>
              <volume-minus-off v-if="isMuted"/>
            </svg>
          </div>

          <div class="player-controls__item" @click="openModal">
            <svg class="icon">
              <antenna-icon/>
            </svg>
          </div>

          <div class="player-controls__item -xl js-play" @click="play">
            <svg class="icon">
              <pause-icon v-if="isTimerPlaying"/>
              <play-icon v-else/>
            </svg>
          </div>
        </div>

      </div>

      <div class="progress" ref="progress">
        <div class="progress__top">
          <div class="album-info" v-if="currentStation">
            <div class="album-info__track">{{ currentStation?.name }}</div>
          </div>
        </div>
        <div
            class="progress__time"
            v-if="currentTime"
        >
          {{ currentTime }}
        </div>
      </div>
      <div v-cloak></div>
    </div>
    <StationsComponent
        v-if="isModalVisible"
        @update:selected-station="onSelectStation"
        :stations="stations"
        :close-modal="closeModal"
    />
  </div>
</template>

<script setup>
import {ref, onMounted, watchEffect, reactive} from 'vue';
import PlayIcon from "@/components/icons/play-icon.vue";
import PauseIcon from "@/components/icons/pause-icon.vue";
import NextIcon from "@/components/icons/next-icon.vue";
import PrevIcon from "@/components/icons/prev-icon.vue";
import LinkIcon from "@/components/icons/link-icon.vue";
import AntennaIcon from "@/components/icons/antenna-icon.vue";
import stationsData from '@/db/stations.json';
import StationsComponent from "@/components/StationsComponent.vue";
import VolumePlusIcon from "@/components/icons/volume-plus-icon.vue";
import VolumeMinusOff from "@/components/icons/volume-minus-off.vue";

const audio = ref(null);
const duration = ref(null);
const currentTime = ref(null);
const isTimerPlaying = ref(false);
const currentStation = ref('');
const currentStationIndex = ref(0);
const transitionName = ref(null);
const applyGlitch = ref(false)
const stations = ref([]);
const isModalVisible = ref(false);
const isMuted = ref(false);
const volume = ref(0.5);
const previousVolume = ref(0.5);

const openModal = () => {
  isModalVisible.value = true;
};

const closeModal = () => {
  isModalVisible.value = false;
};

const onSelectStation = (selectedStation) => {
  currentStation.value = selectedStation;
  const index = stations.value.findIndex((station) => station.id === selectedStation.id);
  currentStationIndex.value = index !== -1 ? index : 0;
  resetPlayer();
  closeModal();
};

const play = () => {
  if (audio.value.paused) {
    const playPromise = audio.value.paused ? audio.value.play() : audio.value.pause();
    if (playPromise !== undefined) {
      playPromise
          .then(() => {
            isTimerPlaying.value = true;
            applyGlitch.value = true;
          })
          .catch(error => {
            console.error('Playback error:', error);
          });
    }
  } else {
    audio.value.pause();
    isTimerPlaying.value = false;
    applyGlitch.value = false;
  }
};

const prevTrack = () => {
  transitionName.value = 'scale-in';
  if (currentStationIndex.value > 0) {
    currentStationIndex.value--;
  } else {
    currentStationIndex.value = stations.value.length - 1;
  }
  currentStation.value = stations.value[currentStationIndex.value];
  resetPlayer();
};

const nextTrack = () => {
  transitionName.value = 'scale-out';
  if (currentStationIndex.value < stations.value.length - 1) {
    currentStationIndex.value++;
  } else {
    currentStationIndex.value = 0;
  }
  currentStation.value = stations.value[currentStationIndex.value];
  resetPlayer();
};

const toggleMute = () => {
  if (!isMuted.value) {
    previousVolume.value = volume.value;
    audio.value.volume = 0;
    volume.value = 0;
    isMuted.value = true;
  } else {
    volume.value = previousVolume.value || 0.5;
    audio.value.volume = volume.value;
    isMuted.value = false;
  }
};

const resetPlayer = () => {
  audio.value.currentTime = 0;
  audio.value.src = currentStation.value?.source;
  setTimeout(() => {
    isTimerPlaying.value ? audio.value.play() : audio.value.pause();
  }, 300);
};

const generateTime = () => {
  let durMin = Math.floor(audio.value.duration / 60);
  let durSec = Math.floor(audio.value.duration - durMin * 60);
  let curMin = Math.floor(audio.value.currentTime / 60);
  let curSec = Math.floor(audio.value.currentTime - curMin * 60);
  durMin = durMin < 10 ? '0' + durMin : durMin;
  durSec = durSec < 10 ? '0' + durSec : durSec;
  curMin = curMin < 10 ? '0' + curMin : curMin;
  curSec = curSec < 10 ? '0' + curSec : curSec;
  duration.value = durMin + ':' + durSec;
  currentTime.value = curMin + ':' + curSec;
};

onMounted(() => {
  stations.value = stationsData;
  currentStation.value = stations.value[0];
  audio.value = new Audio();

  audio.value.src = currentStation.value?.source;
  audio.value.ontimeupdate = () => {
    generateTime();
  };
  audio.value.onloadedmetadata = () => {
    generateTime();
  };
  for (let index = 0; index < stations.value.length; index++) {
    const element = stations.value[index];
    const link = document.createElement('link');
    link.rel = 'prefetch';
    link.href = element?.cover;
    link.as = 'image';
    document.head.appendChild(link);
  }
});
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Roboto:wght@100&display=swap");

* {
  box-sizing: border-box;
}

html, body {
  font-family: 'Roboto', sans-serif;
  background: linear-gradient(135deg, rgb(206, 177, 121) 0%, rgb(96, 153, 184) 100%) no-repeat;
  background-size: auto;
  margin: 0;
  padding: 0;
  height: 100%;
}

.icon {
  display: inline-block;
  width: 1em;
  height: 1em;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}

.wrapper {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding-top: 40px;
  padding-bottom: 40px;
  height: calc(100vh - 80px); /* Adjust the height to account for padding */
  box-sizing: border-box; /* Include padding and borders in the element's total height */
  @media screen and (max-width: 700px), (max-height: 500px) {
    flex-wrap: wrap;
    flex-direction: column;
  }
}

.player {
  background: #141c22;
  width: 410px;
  min-height: 480px;
  box-shadow: 0 55px 75px -10px rgba(76, 70, 124, 0.5);
  border-radius: 15px;
  padding: 30px;
  @media screen and (max-width: 576px), (max-height: 500px) {
    width: 95%;
    margin-top: 75px;
    min-height: initial;
    padding: 20px 20px 30px;
    max-width: 400px;
  }

  &__top {
    display: flex;
    align-items: flex-start;
    position: relative;
    z-index: 4;
    @media screen and (max-width: 576px), (max-height: 500px) {
      flex-wrap: wrap;
    }
  }

  &-cover {
    width: 300px;
    height: 300px;
    margin-left: -70px;
    flex-shrink: 0;
    position: relative;
    z-index: 2;
    border-radius: 15px;

    @media screen and (max-width: 576px), (max-height: 500px) {
      width: 290px;
      height: 230px;
      margin: -70px auto 25px;
    }

    &__item {
      background-repeat: no-repeat;
      background-position: center;
      background-size: cover;
      width: 100%;
      height: 100%;
      border-radius: 15px;
      position: absolute;
      left: 0;
      top: 0;

      &:before {
        content: "";
        background: inherit;
        width: 100%;
        height: 100%;
        box-shadow: 0 10px 40px 0 rgba(76, 70, 124, 0.5);
        display: block;
        z-index: 1;
        position: absolute;
        top: 30px;
        transform: scale(0.9);
        filter: blur(10px);
        opacity: 0.9;
        border-radius: 15px;
      }

      &:after {
        content: "";
        background: inherit;
        width: 100%;
        height: 100%;
        box-shadow: 0 10px 40px 0 rgba(76, 70, 124, 0.5);
        display: block;
        z-index: 2;
        position: absolute;
        border-radius: 15px;
      }
    }

    &__img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      border-radius: 15px;
      box-shadow: 0 10px 40px 0 rgba(76, 70, 124, 0.5);
      user-select: none;
      pointer-events: none;
    }
  }

  &-controls {
    flex: 1;
    padding-left: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;

    @media screen and (max-width: 576px), (max-height: 500px) {
      flex-direction: row;
      padding-left: 0;
      width: 100%;
      flex: unset;
    }

    &__item {
      display: inline-flex;
      font-size: 30px;
      padding: 5px;
      margin-bottom: 10px;
      color: #acb8cc;
      cursor: pointer;
      width: 50px;
      height: 50px;
      align-items: center;
      justify-content: center;
      position: relative;
      transition: all 0.3s ease-in-out;

      @media screen and (max-width: 576px), (max-height: 500px) {
        font-size: 26px;
        padding: 5px;
        margin-right: 10px;
        color: #acb8cc;
        cursor: pointer;
        width: 40px;
        height: 40px;
        margin-bottom: 0;
      }

      &::before {
        content: "";
        position: absolute;
        width: 100%;
        height: 100%;
        transform: scale(0.5);
        opacity: 0;
        transition: all 0.4s cubic-bezier(0.35, 0.57, 0.13, 0.88);
      }

      @media screen and (min-width: 500px) {
        &:hover {
          color: #532ab9;

          &::before {
            opacity: 1;
            transform: scale(1.3);
          }
        }
      }

      @media screen and (max-width: 576px), (max-height: 500px) {
        &:active {
          color: #532ab9;

          &::before {
            opacity: 1;
            transform: scale(1.3);
          }
        }
      }

      .icon {
        position: relative;
        z-index: 2;
      }

      &.-xl {
        margin-bottom: 0;
        font-size: 95px;
        filter: drop-shadow(0 11px 6px rgba(172, 184, 204, 0.45));
        color: #fff;
        width: auto;
        height: auto;
        display: inline-flex;
        @media screen and (max-width: 576px), (max-height: 500px) {
          margin-left: auto;
          font-size: 75px;
          margin-right: 0;
        }

        &:before {
          display: none;
        }

        &:hover {
          color: #532ab9;
        }
      }
    }
  }
}


[v-cloak] {
  display: none;
}

[v-cloak] > * {
  display: none;
}

.progress {
  display: flex;
  align-items: center;
  justify-content: start;
  flex-direction: row;
  width: 100%;
  margin-top: 10px;
  user-select: none;

  &__top {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;
  }

  &__duration {
    color: #71829e;
    font-weight: 700;
    font-size: 20px;
    opacity: 0.5;
  }

  &__time {
    margin-top: 2px;
    color: #fff;
    font-weight: 700;
    font-size: 16px;
    opacity: 0.7;
  }
}

.progress__bar {
  height: 6px;
  width: 100%;
  cursor: pointer;
  background-color: #d0d8e6;
  display: inline-block;
  border-radius: 10px;
}

.progress__current {
  height: inherit;
  width: 0;
  background-color: #a3b3ce;
  border-radius: 10px;
}

.album-info {
  color: #fff;
  flex: 1;
  padding-right: 60px;
  user-select: none;

  @media screen and (max-width: 576px), (max-height: 500px) {
    padding-right: 30px;
  }

  &__name {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 12px;
    line-height: 1.3em;
    @media screen and (max-width: 576px), (max-height: 500px) {
      font-size: 18px;
      margin-bottom: 9px;
    }
  }

  &__track {
    font-weight: 400;
    font-size: 20px;
    opacity: 0.7;
    line-height: 4.3em;
    min-height: 52px;
    @media screen and (max-width: 576px), (max-height: 500px) {
      font-size: 18px;
      min-height: 50px;
    }
  }
}

.scale-out-enter-active {
  transition: all .35s ease-in-out;
}

.scale-out-leave-active {
  transition: all .35s ease-in-out;
}

.scale-out-enter {
  transform: scale(.55);
  pointer-events: none;
  opacity: 0;
}

.scale-out-leave-to {
  transform: scale(1.2);
  pointer-events: none;
  opacity: 0;
}

.scale-in-enter-active {
  transition: all .35s ease-in-out;
}

.scale-in-leave-active {
  transition: all .35s ease-in-out;
}

.scale-in-enter {
  transform: scale(1.2);
  pointer-events: none;
  opacity: 0;
}

.scale-in-leave-to {
  transform: scale(.55);
  pointer-events: none;
  opacity: 0;
}

// glitch

$animation-duration: 5s;
$glitch-duration: 40%;
$glitch-frequency: 10;
$glitch-interval: $glitch-duration / $glitch-frequency;

@function rand($min, $max) {
  @return random() * ($max - $min) + $min;
}

@mixin rgb-shift($name) {
  @keyframes rgb-shift-#{$name} {
    @for $i from 0 to $glitch-frequency {
      #{$i * $glitch-interval} {
        transform: translate(
                #{rand(-2, 2) * 1%},
                #{rand(-0.5, 0.5) * 1%}
        );
      }
    }

    #{$glitch-duration}, 100% {
      transform: none;
    }
  }

  animation: rgb-shift-#{$name} $animation-duration steps(1, jump-end) infinite alternate both;
}

@mixin glitch($name) {
  @keyframes glitch-#{$name} {
    @for $i from 0 to $glitch-frequency {
      $left: 0%;
      $right: 100%;
      $top: rand(0, 90) * 1%;
      $bottom: $top + rand(1, 10) * 1%;

      #{$i * $glitch-interval} {
        clip-path: polygon(
                $left $top,
                $right $top,
                $right $bottom,
                $left $bottom
        );
        transform: translate(
                #{rand(-8, 8) * 1%},
                #{rand(-0.5, 0.5) * 1%}
        );
      }
    }

    #{$glitch-duration}, 100% {
      clip-path: none;
      transform: none;
    }
  }
  animation: glitch-#{$name} $animation-duration linear infinite alternate both;
}

.glitch {
  &::before,
  &::after,
  .channel {
    background: inherit;
    background-size: cover;
    bottom: 0;
    left: 0;
    position: absolute;
    right: 0;
    top: 0;
  }

  &::before {
    @include glitch(before);
    content: "";
  }

  &::after {
    @include glitch(after);
    content: "";
  }

  .channel {
    mix-blend-mode: screen;
    border-radius: 15px;

    &::before {
      bottom: 0;
      content: "";
      display: block;
      mix-blend-mode: multiply;
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
    }
  }

  .r {
    @include rgb-shift(r);

    &::before {
      background: #f00;
      border-radius: 15px;
    }
  }

  .g {
    @include rgb-shift(g);

    &::before {
      background: #0f0;
      border-radius: 15px;
    }
  }

  .b {
    @include rgb-shift(b);

    &::before {
      background: #00f;
      border-radius: 15px;
    }
  }
}
</style>