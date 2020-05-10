# @loquiry/vue-record-audio

A styleless audio recorder button as a vue component

## Install

`npm i @loquiry/vue-record-audio` or
`yarn add @loquiry/vue-record-audio`

## Usage

```vue
<template>
  <VueRecord class="record" @result="onResult">
    Record
    <template slot="isRecording">
      Stop
    </template>
  </VueRecord>
</template>

<script>
import VueRecord from "@loquiry/vue-record-audio";

export default {
  components: { VueRecord },
  methods: {
    onResult(data) {
      console.log("record button data:", data.blob);
      console.log("Sound in ms:", data.duration);
    }
  }
};
</script>

<style>
.record.active {
  background: red;
}
</style>
```

## Api

### Result

The result is an object containing the blob and the duration of the recording in ms

```js
result = {
  blob: blob,
  duration: 1515.9850000000006
};
```

### Events

- result: fired after recording, returns a blob

### Content

As stated in [Usage](#Usage) The component has two slots that will be switched if the button is recording

### Classes

- `.active` when the button is recording ## Development

### Develpment

```
yarn install
```

### Compiles and hot-reloads for development

```
yarn run serve
```
