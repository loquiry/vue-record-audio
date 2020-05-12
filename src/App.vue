<template>
  <button
    @click="recorderClick"
    :class="[isRecording && 'active', !isInitiated && 'needsInitiation']"
  >
    <slot v-if="!isRecording" />
    <slot v-if="isRecording" name="isRecording"></slot>
  </button>
</template>

<script>
import Recorder from "recorder-js";

const DEFAULT_OPTIONS = { sampleRate: 44100, bufferSize: 16384 };

export default {
  props: {
    options: {
      type: Object,
      required: false
    }
  },
  data() {
    return {
      isInitiated: false,
      isRecording: false,
      recordingStartedAt: null,
      recordingEndedAt: null,
      recorder: null,
      recorderOptions: this.options || DEFAULT_OPTIONS
    };
  },
  methods: {
    async recorderClick() {
      let t = this;
      if (!t.isInitiated) {
        const audioContext = new (window.AudioContext ||
          window.webkitAudioContext)();

        await navigator.mediaDevices
          .getUserMedia({ audio: true })
          .then((stream) => {
            t.recorder = new Recorder(audioContext, t.recorderOptions);
            t.recorder.init(stream);
          })
          .then(() => (t.isInitiated = true))
          .catch((err) => {
            throw err;
          });
      } else {
        if (!t.isRecording) {
          t.recorder.start().then(() => {
            t.isRecording = true;
            t.recordingStartedAt = performance.now();
          });
        } else {
          t.recorder.stop().then(({ blob, buffer }) => {
            t.recordingEndedAt = performance.now();
            t.$emit("result", {
              blob: blob,
              duration: t.recordingEndedAt - t.recordingStartedAt,
              type: blob.type
            });
            t.isRecording = false;
          });
        }
      }
    }
  }
};
</script>
