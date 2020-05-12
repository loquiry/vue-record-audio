<template>
  <div>
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
    <h2>result:</h2>
    <div v-if="duration">duration: {{ duration }}ms</div>
    <div v-if="type">type: {{ type }}</div>
  </div>
</template>

<script>
import VueRecord from "./App";

export default {
  components: { VueRecord },
  data() {
    return {
      duration: null,
      type: null
    };
  },
  methods: {
    applyAudio(data) {
      console.log(data);
      this.$refs.audio.src = URL.createObjectURL(data.blob);
      this.duration = data.duration;
      this.type = data.type;
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
.record.needsInitiation {
  background: black;
}
</style>
