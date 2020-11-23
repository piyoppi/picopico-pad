# picopico-pad

Gamepad Api helper library (capturing + key-mapping)

## Usage

### Detecting gamepad status

The following sample code detects a change of gamepad status.
[Sample](https://piyoppi.github.io/picopico-pad/sample/basic.html)

```js
import { GamePads } from '@piyoppi/picopico-pad';

const gamePads = new GamePads();

gamePads.addEventHandler('connected', async (e) => {
  console.log('Connected');

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

### Selecting a gamepad

The following sample code can select a gamepad.
[Sample](https://piyoppi.github.io/picopico-pad/sample/gamePadSelectable.html)

```js
import { GamePads } from '@piyoppi/picopico-pad';

const gamePads = new GamePads();

// "gamePadListElem" is a "select" element.
// ex. <select id="gamepads"></select>
const gamePadListElem = document.getElementById('gamepads');

const buildGamePadSelector = () => {
  gamePadListElem.innerHTML = '';

  // Build gamepad list
  gamePads.pads.forEach( item => {
    const padItemElem = document.createElement('option');
    padItemElem.setAttribute('value', item.index);
    padItemElem.innerText = item.gamepad.id;
    gamePadListElem.appendChild(padItemElem);
  });
};

gamePadListElem.onchange = async (e) => {
  // Stop capturing
  await gamePads.stop();

  // Setup selected gamepad
  await gamePads.setIndex(parseInt(e.target.value));

  // Restart capturing
  gamePads.capture();
};

gamePads.addEventHandler('connected', async (e) => {
  console.log('Connected');

  buildGamePadSelector();

  await gamePads.setFirstPad();

  // Start capturing
  gamePads.capture();

  gamePads.addEventHandler('buttonChanged', e => {
    console.log(`Button ${e.index} : ${e.value.pressed}`);
  });
  gamePads.addEventHandler('axisChanged', e => {
    console.log(`Axis ${e.index} : ${e.value}`);
  });
});
```

### Mapping buttons or axes to name

The following sample code name each buttons (or axes).
[Sample](https://piyoppi.github.io/picopico-pad/sample/gamePadMapper.html)

```js
import { GamePads, GamePadMapper, keyType } from '@piyoppi/picopico-pad';

// Button-mapping array
//
// name: the name of key
// type: a type of key (button or axis)
// index: button(axis) index
const keys = [
  { name: 'buttonA', type: keyType.button, index: 0 },
  { name: 'buttonB', type: keyType.button, index: 1 },
  { name: 'buttonC', type: keyType.button, index: 2 },
  { name: 'buttonD', type: keyType.button, index: 3 },
  { name: 'axis-x', type: keyType.axis, index: 0 },
  { name: 'axis-y', type: keyType.axis, index: 1 }
];

const gamePads = new GamePads();

// give a GamePad and key-mapping informations
const padMapper = new GamePadMapper(gamePads, keys);

gamePads.addEventHandler('connected', async (e) => {
  console.log('Connected');
  await gamePads.setFirstPad();

  gamePads.capture();

  padMapper.addEventHandler('buttonChanged', e => {
    console.log(`Button ${e.index} | ${e.name} | ${e.value.pressed}`);
  });
  padMapper.addEventHandler('axisChanged', e => {
    console.log(`Axis ${e.index} | ${e.name} | ${e.value}`);
  });
});
```

### Mapping configuration

The following sample code configure key-mapping.
[Sample](https://piyoppi.github.io/picopico-pad/sample/gamePadMapperConfiguration.html)

```js
import { GamePads, GamePadMapper, keyType } from '@piyoppi/picopico-pad';

// Initial key-mapping
const keys = [
  { name: 'button-1', type: keyType.button, index: 0 },
  { name: 'button-2', type: keyType.button, index: 1 },
  { name: 'button-3', type: keyType.button, index: 2 },
  { name: 'button-4', type: keyType.button, index: 3 },
  { name: 'axis-x', type: keyType.axis, index: 0 },
  { name: 'axis-y', type: keyType.axis, index: 1 }
];

const gamePads = new GamePads();
const padMapper = new GamePadMapper(gamePads, keys);

// captureButton and stopButton are button elements.
const captureButton = document.getElementById('capture');
const stopButton = document.getElementById('stop');

stopButton.style.display = 'none';

// Cancel key-mapping
stopButton.addEventListener('click', () => {
  console.log('Stopped');
  padMapper.stop();
});

// Set event hander to each buttons
// targetButton is button element for selecting a key for setup.
// ex)
// <button id="setup-button-1">Set</button>
// <button id="setup-button-2">Set</button>
// <button id="setup-button-3">Set</button>
// ...
keys.map(key => key.name).forEach(keyName => {
  const targetButton = document.getElementById(`setup-${keyName}`);
  targetButton.addEventListener('click', async () => {
    await padMapper.register(keyName);
  });
});

// Show the result of key mappings
padMapper.addEventHandler('applied', e => {
  console.log(`${e.name} is configured (key index: ${e.index})`);
});

gamePads.addEventHandler('connected', async (e) => {
  console.log('Connected');

  // Select using a gamepad
  await gamePads.setFirstPad();

  gamePads.capture();

  captureButton.addEventListener('click', () => {
    console.log('Capturing...');
    padMapper.resetStepBy();
    padMapper.registerAll();
    stopButton.style.display = '';
    captureButton.style.display = 'none';
  });

  padMapper.addEventHandler('buttonChanged', e => {
    console.log(`Button ${e.index} ${e.name} : ${e.value.pressed}`);
  });
  padMapper.addEventHandler('axisChanged', e => {
    console.log(`Axis ${e.index} ${e.name} : ${e.value}`);
  });
});

padMapper.addEventHandler('cursorChanged', e => {
  if( e.cursor >= 0 ) console.log(`Waiting for input ${e.cursor}th key...`);
});

padMapper.addEventHandler('registerCompleted', e => {
  console.log('Captured');
  stopButton.style.display = 'none';
  captureButton.style.display = '';
});

```
