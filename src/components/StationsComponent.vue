<template>
  <div class="modal" @click="closeModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="player-controls__item">
          <button class="close-button">
            <svg class="icon">
              <close-icon/>
            </svg>
          </button>
        </div>
        <h2>Select Station</h2>
      </div>
      <div class="stations">
        <ul class="stations-list">
          <li
              v-for="(station, index) in stations"
              :key="index"
              class="station-item"
              @click="selectStation(station)">
            <div class="station-content">
              <img
                  class="station-img"
                  :src="station?.cover"
                  :alt="station.name">
              <span class="station-name">{{ station.name }}</span>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import CloseIcon from "@/components/icons/close-icon.vue";

const props = defineProps({
  stations: {
    type: Array,
  },
  closeModal: {
    type: Function
  },
  selectStation: {
    type: Object
  },
});

const emit = defineEmits(['update:selected-station']);

const selectStation = (selectedStation) => {
  emit('update:selected-station', selectedStation)
};
</script>

<style scoped lang="scss">
$font-family: 'Roboto', sans-serif;
$primary-color: #6D5BD0;
$secondary-color: #F2F2FE;
$hover-color: #E2DEFE;
$modal-backdrop-color: #141c22;
$text-color: #ffffff;
$shadow-color: rgba(109, 91, 208, 0.3);
$fade-height: 35px;
$gradient-end-color: rgba(242, 242, 254, 0);
$modal-border-radius: 15px;
$modal-padding: 24px;
$transition-duration: 0.3s;
$backdrop-filter-blur: 12px;

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin glassmorphic {
  background: $modal-backdrop-color;
  backdrop-filter: blur($backdrop-filter-blur);
  border: 2px solid transparent;
  background-clip: padding-box;
  border-radius: $modal-border-radius;
  box-shadow: 0 8px 32px 0 $shadow-color;
}

.modal {
  @include flex-center;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);

  .modal-content {
    @include glassmorphic;
    padding: $modal-padding;
    width: 90%;
    max-width: 600px;
    transform: translateY(-50%) scale(0.5);
    opacity: 0;
    animation: modalEntrance $transition-duration ease-out forwards;
    overflow: hidden;

    .modal-header {
      font-family: $font-family;
      border-bottom: 1px solid $hover-color;
      padding-bottom: $modal-padding * 0.5;
      margin-bottom: $modal-padding;

      h2 {
        font-size: 1.75rem;
        color: $text-color;
        margin: 0;
        text-align: center;
      }

      .close-button {
        background: none;
        border: none;
        cursor: pointer;
        font-size: 1.5rem;
        color: #acb8cc;

        &:hover {
          color: darken($primary-color, 10%);
          box-shadow: 0 5px 10px 0 rgba(76, 70, 124, 0.2);
          transition: all 0.4s cubic-bezier(0.35, 0.57, 0.13, 0.88);
        }

        &.close-animate {
          animation: modalExit $transition-duration ease-in forwards;
        }
      }
    }

    .stations {
      max-height: 60vh;
      overflow-y: auto;

      .stations-list {
        list-style: none;
        margin: 0;
        padding: 0;
        cursor: pointer;

        .station-item {
          @include flex-center;
          transition: background-color $transition-duration;
          padding: $modal-padding * 0.5;
          border-radius: $modal-padding * 0.5;
          margin-bottom: 8px;
          justify-content: start;

          &:hover {
            background-color: $hover-color;
            border-radius: $modal-padding * 0.75;
          }

          .station-content {
            @include flex-center;

            .station-img {
              width: 64px;
              height: 64px;
              border-radius: 50%;
              object-fit: cover;
              margin-right: $modal-padding;
            }

            .station-name {
              font-size: 1.2rem;
              color: $text-color;
            }
          }
        }
      }
    }

    .stations {
      &::-webkit-scrollbar {
        width: 5px;
      }

      &::-webkit-scrollbar-thumb {
        background-color: $primary-color;
        border-radius: 10px;
      }

      &:after {
        content: '';
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        height: $fade-height;
        pointer-events: none;
      }
    }
  }
}

@keyframes modalEntrance {
  0% {
    transform: scale(0.5);
    opacity: 0;
  }
  50% {
    transform: scale(1.05);
    opacity: 0.5;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes modalExit {
  0% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(0.9);
    opacity: 0;
  }
}

@supports (backdrop-filter: blur($backdrop-filter-blur)) {
  .modal-content {
    background: $modal-backdrop-color;
    background: linear-gradient(145deg, rgba(255, 255, 255, 0.65), rgba(255, 255, 255, 0.45));
    border-radius: $modal-border-radius;
    border: 1px solid rgba(255, 255, 255, 0.72);
  }
}
</style>