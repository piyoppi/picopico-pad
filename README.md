# picopico-pad

Gamepad Api helper library (capturing + key-mapping)

## Usage

```js
import { GamePads } from './src/main.js';

const gamePads = new GamePads();

gamePads.addEventHandler('connected', async (e) => {
  // Select the first gamepad.
  await gamePads.setFirstPad();

  // Capturing loop
  gamePads.capture();

  // The event when button is pressed.
  gamePads.addEventHandler('buttonChanged', e => {
    console.log(`Button ${e.index} : ${e.value.pressed}`);
  });

  // The event when stich is tilted.
  gamePads.addEventHandler('axisChanged', e => {
    console.log(`Axis ${e.index} : ${e.value}`);
  });
});
```
