<template>
  <button @click="record" :class="['record', isRecording && 'active']">
    <slot v-if="!isRecording" />
    <slot v-if="isRecording" name="isRecording"></slot>
  </button>
</template>

<script>
import RecordRTC from "recordrtc";

export default {
  data() {
    return {
      isRecording: false,
      hasMediaDeviceCaps: false,
      startedRecording: null,
      mediaRecorder: null,
      isEdge:
        navigator.userAgent.match(/Edge/) !== -1 &&
        (!!navigator.msSaveOrOpenBlob || !!navigator.msSaveBlob),
      isSafari: !!navigator.userAgent.match(/^((?!chrome|android).)*safari/i),
      isWin:
        navigator.platform &&
        navigator.platform
          .toString()
          .toLowerCase()
          .indexOf("win") === -1
    };
  },
  mounted() {
    this.grantPermissions();
  },
  methods: {
    async grantPermissions() {
      let t = this;
      try {
        await navigator.mediaDevices.getUserMedia({ audio: true });
        this.hasMediaDeviceCaps = true;
      } catch (err) {
        // could be used to show a hint that the device/browser does not support WebRTC.
        // should potentially never be the case but better be safe
        this.hasMediaDeviceCaps = false;
      }
    },
    async record() {
      if (!this.isRecording) {
        this.isRecording = true;
        const stream = await navigator.mediaDevices.getUserMedia({
          audio: this.isEdge
            ? true
            : {
                echoCancellation: false
              }
        });

        var options = {
          type: "audio",
          numberOfAudioChannels: this.isEdge ? 1 : 2,
          checkForInactiveTracks: true,
          bufferSize: 16384,
          // disble webRTC logs
          disableLogs: true
        };

        if (this.isSafari || this.isEdge) {
          options.recorderType = RecordRTC.StereoAudioRecorder;
        }

        if (this.isSafari) {
          options.sampleRate = 44100;
          options.bufferSize = 4096;
        }

        this.mediaRecorder = new RecordRTC(stream.clone(), options);
        this.mediaRecorder.startRecording();
        this.startedRecording = performance.now();
      } else {
        this.isRecording = false;

        if (this.mediaRecorder) {
          this.mediaRecorder.stopRecording(this.handleDataAvailable);
        }
      }
    },
    handleDataAvailable() {
      if (this.mediaRecorder) {
        const blob = this.mediaRecorder.getBlob();
        this.$emit("result", {
          blob: blob,
          duration: performance.now() - this.startedRecording
        });
      }
    }
  }
};
</script>
