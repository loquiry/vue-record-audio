<template>
  <div>
    <div class="error" v-if="error">{{ error }}</div>
    <VueRecord
      class="record"
      @result="applyAudio"
      @start="cLog('started')"
      @stop="cLog('stopped')"
    >
      Record
      <span slot="isRecording">
        Stop
      </span>
    </VueRecord>
    <audio ref="audio" controls></audio>
    <div v-if="duration">{{ duration }}ms</div>
  </div>
</template>

<script>
import VueRecord from "./App";

export default {
  components: { VueRecord },
  data() {
    return {
      duration: null,
      error: null
    };
  },
  methods: {
    async applyAudio(data) {
      let t = this;
      this.$refs.audio.src = await URL.createObjectURL(data.blob).catch(
        (err) => (t.error = err)
      );
      this.duration = data.duration;
    },
    cLog(content) {
      console.log(content);
    }
  }
};
</script>

<style>
.record {
  color: white;
  width: 90px;
  height: 90px;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 90px;
  background: var(--blue);
  transition: background 120ms ease-in-out, transform 120ms ease-in-out;
}

.record:focus {
  outline: none;
}
.record.active {
  background: var(--red);
  transform: scale(1.05);
  animation: none;
}
.error {
  padding: 1em;
  border-bottom: 2px solid red;
}
</style>
