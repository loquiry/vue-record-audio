<template>
  <button
    @click="recorderClick"
    :class="[isRecording && 'active', !isInitiated && 'needsInitiation']"
  >
    <slot v-if="isInitiated && !isRecording" />
    <slot v-if="!isInitiated" name="isInitiating" />
    <slot v-if="isInitiated && isRecording" name="isRecording"></slot>
  </button>
</template>

<script>
import Recorder from "recorder-js";

const DEFAULT_OPTIONS = { sampleRate: 44100, bufferSize: 16384 };

export default {
  props: {
    isAlreadyInitiated: {
      type: Boolean,
      default: false
    },
    options: {
      type: Object,
      required: false
    }
  },
  data() {
    return {
      isInitiated: this.isAlreadyInitiated || false,
      isRecording: false,
      recordingStartedAt: null,
      recordingEndedAt: null,
      recorder: null,
      recorderOptions: this.options || DEFAULT_OPTIONS
    };
  },
  async mounted() {
    this.isInitiated &&
      (await this.initiatePlayer().catch((err) => {
        this.isInitiated = false;
        this.causeError(err);
      }));
  },
  methods: {
    causeError(error) {
      console.error("@loquiry/vue-record-audio error");
      this.$emit("error", error);
      throw error;
    },
    async initiatePlayer() {
      let t = this;

      const audioContext = new (window.AudioContext ||
        window.webkitAudioContext)();

      await navigator.mediaDevices
        .getUserMedia({ audio: true })
        .then((stream) => {
          t.recorder = new Recorder(audioContext, t.recorderOptions);
          t.recorder.init(stream);
        })
        .then(() => {
          t.$emit("initiated");
          t.isInitiated = true;
        })
        .catch((err) => {
          t.causeError(err);
        });
    },
    async recorderClick() {
      let t = this;
      if (!t.isInitiated) {
        await this.initiatePlayer().catch((err) => {
          t.causeError(err);
        });
      } else {
        if (!t.isRecording) {
          await t.recorder
            .start()
            .then(() => {
              t.isRecording = true;
              t.recordingStartedAt = performance.now();
            })
            .catch((err) => {
              t.causeError(err);
            });
        } else {
          t.recorder
            .stop()
            .then(({ blob, buffer }) => {
              t.recordingEndedAt = performance.now();
              t.$emit("result", {
                blob: blob,
                duration: t.recordingEndedAt - t.recordingStartedAt,
                type: blob.type
              });
              t.isRecording = false;
            })
            .catch((err) => {
              t.causeError(err);
            });
        }
      }
    }
  }
};
</script>
