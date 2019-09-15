# VCV Prototype

Scripting language host for [VCV Rack](https://vcvrack.com/) containing:
- 6 inputs
- 6 outputs
- 6 knobs
- 6 lights (RGB LEDs)
- 6 switches with RGB LEDs

[Discussion thread](https://community.vcvrack.com/t/vcv-prototype/3271)

## Scripting API

This is the reference API for the JavaScript script engine, along with default property values.
Other script engines may vary in their syntax (e.g. `block.inputs[i]` vs `block.getInput(i)` vs `input(i)`), but the functionality should be similar.

```js
/** Display message on LED display.
*/
display(message)

/** Skip this many sample frames before running process().
For CV generators and processors, 256 is reasonable.
For sequencers, 32 is reasonable since process() will be called every 0.7ms with a 44100kHz sample rate, which will capture 1ms-long triggers.
For audio generators and processors, 1 is recommended, but use `bufferSize` below.
If this is too slow for your purposes, consider writing a C++ plugin, since native VCV Rack plugins have 10-100x better performance.
*/
config.frameDivider // 32

/** Instead of calling process() every sample frame, hold this many input/output voltages in a buffer and call process() when it is full.
This decreases CPU usage, since processing buffers is faster than processing one frame at a time.
The total latency of your script is `config.frameDivider * config.bufferSize * block.sampleTime`.
*/
config.bufferSize // 1

/** Called when the next block is ready to be processed.
*/
function process(block) {
	/** Engine sample rate in Hz. Read-only.
	*/
	block.sampleRate

	/** Engine sample timestep in seconds. Equal to `1 / sampleRate`. Read-only.
	Note that the actual time between process() calls is `block.sampleTime * config.frameDivider`.
	*/
	block.sampleTime

	/** The actual size of the input/output buffers.
	*/
	block.bufferSize

	/** Voltage of the input port of row `rowIndex`. Read-only.
	*/
	block.inputs[rowIndex][bufferIndex] // 0.0

	/** Voltage of the output port of row `rowIndex`. Writable.
	*/
	block.outputs[rowIndex][bufferIndex] // 0.0

	/** Value of the knob of row `rowIndex`. Between 0 and 1. Read-only.
	*/
	block.knobs[rowIndex] // 0.0

	/** Pressed state of the switch of row `rowIndex`. Read-only.
	*/
	block.switches[rowIndex] // false

	/** Brightness of the RGB LED of row `rowIndex`, between 0 and 1. Writable.
	*/
	block.lights[rowIndex][0] // 0.0 (red)
	block.lights[rowIndex][1] // 0.0 (green)
	block.lights[rowIndex][2] // 0.0 (blue)

	/** Brightness of the switch RGB LED of row `rowIndex`. Writable.
	*/
	block.switchLights[rowIndex][0] // 0.0 (red)
	block.switchLights[rowIndex][1] // 0.0 (green)
	block.switchLights[rowIndex][2] // 0.0 (blue)
}
```

## Adding a script engine

- Add your scripting language library to the build system so it builds with `make dep`, following the Duktape example in the `Makefile`.
- Create a `MyEngine.cpp` file (for example) in `src/` with a `ScriptEngine` subclass defining the virtual methods, possibly using `src/DuktapeEngine.cpp` as an example.
- Add your engine to the "List of ScriptEngines" in `src/ScriptEngine.cpp`.
- Build and test the plugin.
- Add a few example scripts and tests to `examples/`. These will be included in the plugin package for the user.
- Add your name to the Contributers list below.
- Send a pull request. Once merged, you will be added as a repo maintainer. Be sure to "watch" this repo to be notified of bugs in your engine.

## Contributers

- [Wes Milholen](https://grayscale.info/): panel design
- [Andrew Belt](https://github.com/AndrewBelt): host code, Duktape (JavaScript)
- add your name here

## License

All **source code** is copyright © 2019 VCV Prototype Contributers and licensed under the [BSD-3-Clause License](https://opensource.org/licenses/BSD-3-Clause).

The **panel graphics** in the `res` directory are copyright © 2019 [Grayscale](http://grayscale.info/) and licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/).
You may not distribute modified adaptations of these graphics.

**Dependencies** included in the binary distributable may have other licenses.
For example, if a GPL library is included in the distributable, the entire work is covered by the GPL.
See [LICENSE-dist.txt](LICENSE-dist.txt) for a full list.