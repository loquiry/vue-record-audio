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
    <template slot="isInitiating">
      Grant microphone permissions
    </template>
    <template slot="isRecording">
      Stop
    </template>
    <template slot="isCreating">
      Creating Sound...
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

### Result Object

The result is an object containing the blob and the duration of the recording in ms

```js
result = {
  blob: blob,
  duration: 1515.9850000000006,
  type: "audio/wav"
};
```

### Events

- result: fired after recording | returns the [Result Object](#result-object)
- initiated: fired when permissions are successfully granted | returns nothing
- error: fired when an error happens and | returns the error message

### Content

As stated in [Usage](#usage) The component has three slots that will be switched if the button is initiated/recording

### Css Classes

- `.active` when the button is recording
- `.needsInitiation` when the button has never been pressed
- `.creating` when the record stopped and the result is being created

### Props

- `options`: pass an options object as prop `options`

  default:

  ```
  { sampleRate: 44100, bufferSize: 16384 }
  ```

- `isAlreadyInitiated`: immediatly initiation the recorder (helpful when you know a user has already given permissions to use the microphone)

  default: `false`

## Development

```
yarn install
```

### Compiles and hot-reloads for development

```
yarn run serve
```

## Dependencies

This is only working thanks to

- [recorder-js](https://www.npmjs.com/package/recorder-js)
