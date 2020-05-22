<template>
  <button
    @click="recorderClick"
    :disabled="creatingResult || disabled"
    :class="[
      isRecording && 'active',
      !isInitiated && 'needsInitiation',
      creatingResult && 'creating'
    ]"
  >
    <slot v-if="isInitiated && !isRecording && !creatingResult" />
    <slot v-if="!isInitiated" name="isInitiating" />
    <slot v-if="creatingResult" name="isCreating" />
    <slot v-if="isInitiated && isRecording" name="isRecording"></slot>
  </button>
</template>

<script>
import Recorder from "recorder-js";

const DEFAULT_OPTIONS = { sampleRate: 44100, bufferSize: 16384 };

export default {
  props: {
    disabled: Boolean,
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
      creatingResult: false,
      recordingStartedAt: null,
      recordingEndedAt: null,
      recorder: null,
      audioContext: null,
      strema: null,
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
      this.audioContext = new (window.AudioContext ||
        window.webkitAudioContext)();

      this.isInitiated = true;
      this.$emit("initiated");
    },
    async recorderClick() {
      if (!this.isInitiated) {
        await this.initiatePlayer();
      } else {
        if (!this.isRecording) {
          this.stream = await navigator.mediaDevices.getUserMedia({
            audio: true
          });
          this.recorder = new Recorder(this.audioContext, this.recorderOptions);
          this.recorder.init(this.stream);
          await this.recorder.start();
          this.recordingStartedAt = performance.now();
          this.isRecording = true;
        } else {
          this.recordingEndedAt = performance.now();
          this.isRecording = false;
          this.creatingResult = true;
          const tracks = this.stream.getTracks();
          tracks.forEach((track) => {
            track.stop();
          });

          const { blob, buffer } = await this.recorder.stop();

          this.$emit("result", {
            blob: blob,
            duration: this.recordingEndedAt - this.recordingStartedAt,
            type: blob.type
          });
          this.creatingResult = false;
        }
      }
    }
  }
};
</script>
