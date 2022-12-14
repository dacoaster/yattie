<template>
  <div v-if="item" class="audio-wrapper">
    <div ref="waveform" class="mb-2 waveform"></div>
    <v-row>
      <v-col cols="12" class="progress-bar">
        <span class="time">00:00</span>
        <div class="progress">
          <v-slider
            color="primary"
            step="1"
            thumb-label
            :thumb-size="20"
            hide-details
            hide-spin-buttons
            track-color="#D1D5DB"
          ></v-slider>
        </div>
        <span class="time">{{ durationTime }}</span>
      </v-col>
    </v-row>
    <v-row class="d-flex justify-space-between">
      <v-col cols="auto" class="pa-0 btn-panel">
        <v-btn
          id="btn_rewind"
          icon
          color="#6b7280"
          @click="wavesurfer.skipBackward(1)"
        >
          <v-icon>mdi-rewind</v-icon>
        </v-btn>
        <v-btn
          id="btn_play"
          v-if="!isPlaying"
          icon
          color="#6b7280"
          @click="wavesurfer.playPause()"
        >
          <v-icon>mdi-play-circle</v-icon>
        </v-btn>
        <v-btn
          id="btn_pause"
          v-if="isPlaying"
          icon
          color="#6b7280"
          @click="wavesurfer.playPause()"
        >
          <v-icon>mdi-pause-circle</v-icon>
        </v-btn>
        <v-btn
          id="btn_forward"
          icon
          color="#6b7280"
          @click="wavesurfer.skipForward(1)"
        >
          <v-icon>mdi-fast-forward</v-icon>
        </v-btn>
      </v-col>
      <v-col cols="auto" class="pa-0">
        <div class="volume-control">
          <div class="volume-btn">
            <v-btn
              id="btn_volume_off"
              v-if="!isMute"
              icon
              color="#111827"
              @click="wavesurfer.toggleMute()"
            >
              <v-icon>mdi-volume-high</v-icon>
            </v-btn>
            <v-btn
              id="btn_volume_on"
              v-if="isMute"
              icon
              color="#111827"
              @click="wavesurfer.toggleMute()"
            >
              <v-icon>mdi-volume-off</v-icon>
            </v-btn>
          </div>
          <div class="volume-progress">
            <v-slider
              v-model="volume"
              color="primary"
              step="1"
              thumb-label
              :thumb-size="20"
              hide-details
              hide-spin-buttons
              track-color="#D1D5DB"
              @change="setVolume()"
            ></v-slider>
          </div>
        </div>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import WaveSurfer from "wavesurfer.js";
import { IPC_HANDLERS, IPC_FUNCTIONS } from "../modules/constants";
export default {
  name: "AudioWrapper",
  props: {
    item: {
      type: Object,
      default: () => {},
    },
    triggerSave: {
      type: Boolean,
      default: () => false,
    },
  },
  data: () => ({
    wavesurfer: null,
    volume: 20,
  }),
  watch: {
    triggerSave: function (oldValue, newValue) {
      if (oldValue !== newValue) {
        this.handleAudio(true);
      }
    },
  },
  computed: {
    isPlaying() {
      if (!this.wavesurfer) return false;

      return this.wavesurfer.isPlaying();
    },
    isMute() {
      if (!this.wavesurfer) return false;

      return this.wavesurfer.getMute();
    },
    durationTime() {
      let duration;
      if (!this.wavesurfer) {
        duration = 0;
      } else {
        duration = parseFloat(this.wavesurfer.getDuration()).toFixed(2);
      }

      duration = 0;

      const date = new Date(null);
      date.setSeconds(duration);
      const result = date.toISOString().substr(14, 5);
      return result;
    },
    currentTime() {
      if (!this.wavesurfer) return 0;

      // return this.wavesurfer.getCurrentTime();
      const duration = parseInt(this.wavesurfer.getDuration());
      const current = parseInt(this.wavesurfer.getCurrentTime());
      const result = Math.round((current / duration) * 100);
      return result;
    },
  },
  mounted() {
    if (!this.wavesurfer) {
      this.$nextTick(async () => {
        await this.createWaveSurfer();
      });
    }
  },
  methods: {
    async createWaveSurfer() {
      this.wavesurfer = WaveSurfer.create({
        container: this.$refs.waveform,
        // hideScrollbar: false,
        waveColor: "#6B7280",
        progressColor: "hsla(200, 100%, 30%, 0.5)",
        cursorColor: "#000",
        barWidth: 3,
      });
      this.wavesurfer.load(`file://${this.item.filePath}`);
    },
    setVolume() {
      const volume = Math.round(this.volume);
      if (volume === 0) {
        this.wavesurfer.setMute(true);
      } else {
        this.wavesurfer.setMute(false);
      }
      this.wavesurfer.setVolume(volume);
    },
    async handleAudio() {
      let item;
      const uri = this.wavesurfer.exportImage("image/png", 1, "dataURL");

      await window.ipc
        .invoke(IPC_HANDLERS.CAPTURE, {
          func: IPC_FUNCTIONS.CREATE_IMAGE,
          data: { url: uri },
        })
        .then(({ filePath }) => {
          item = { ...this.item, poster: filePath };
        });

      this.$root.$emit("save-data", item);
    },
  },
};
</script>
<style scoped>
.audio-wrapper {
  padding: 0 5px;
}
.progress-bar {
  display: flex;
  flex-direction: row;
  align-items: center;
  column-gap: 5px;
}
.progress-bar .time {
  font-style: normal;
  font-weight: 500;
  font-size: 14px;
  line-height: 24px;
  color: #1f2937;
}
.progress-bar .progress {
  flex-grow: 1;
}
.volume-control {
  display: flex;
  align-items: center;
}
.volume-control .volume-progress {
  width: 120px;
}
</style>
