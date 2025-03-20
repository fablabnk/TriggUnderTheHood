# Making Sound on the Trigg

## What is I2S?

## What are .pio files?

## What is a 'tune'?

Example from an existing game (Asteroids)

const gamePlayTune = tune`
75,
37.5: D5^37.5,
37.5: D5^37.5,
37.5: C5^37.5,
37.5: C5^37.5,
37.5: C5^37.5,
37.5: G4^37.5,
37.5: G4^37.5,
37.5: G4^37.5,
37.5,
37.5: A5^37.5,
75,
37.5: B4^37.5,
150,
37.5: A5^37.5,
37.5: A5^37.5,
37.5,
37.5: C4^37.5,
37.5,
37.5: A5^37.5,
37.5: A5^37.5,
37.5,
37.5: A5^37.5,
37.5: A5^37.5,
112.5`// Game tune.
const gameEndTune = tune`
133.92857142857142,
133.92857142857142: B5^133.92857142857142,
133.92857142857142: A5^133.92857142857142,
133.92857142857142: G5^133.92857142857142,
133.92857142857142: F5^133.92857142857142,
133.92857142857142: E5^133.92857142857142,
133.92857142857142: D5^133.92857142857142,
133.92857142857142: C5^133.92857142857142,
133.92857142857142: B4^133.92857142857142,
133.92857142857142: A4^133.92857142857142,
133.92857142857142: G4^133.92857142857142,
133.92857142857142: F4^133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142: E4^133.92857142857142,
133.92857142857142,
133.92857142857142: D4^133.92857142857142,
133.92857142857142: D4^133.92857142857142,
133.92857142857142,
133.92857142857142: C4^133.92857142857142,
133.92857142857142: C4^133.92857142857142,
535.7142857142857`// Wa wa wa wa. You lost!

## How does sprig engine make sound (code example)?

game-engine side:	sprig/engine/src/web/tune.ts
spade side:			sprig/blob/main/firmware/spade/src/shared/audio/piano.c
rpi side: 			sprig/blob/main/firmware/spade/src/rpi/main.c
hardware side:		sprig/blob/main/firmware/sprig_hal/src/HAL.c

## How is I2S stereo converted to mono by the amplifier board

## Does it make sense to go through the amplifier board to make a mono 3.5mm jack output?

Using the MAX98357A amplifier to create a 3.5mm jack output is not recommended due to its design for direct speaker driving and high-power PWM output. For a proper mono 3.5mm jack output, it's better to use a dedicated DAC chip with appropriate filtering and buffering.

## Method for outputting digital audio from a Pico PWM pin:

The Raspberry Pi Pico can output audio through PWM (Pulse-Width Modulation) using a simple circuit. Here's how to set up a mono 3.5mm jack output:

Create a basic PWM audio output circuit:

Connect GPIO pin (e.g., GP0) to a 220Ω resistor

Connect the resistor to a 100Ω resistor

Connect a 0.1μF capacitor between the 100Ω resistor and ground

Connect a 47μF capacitor in parallel with the 0.1μF capacitor

Connect the junction of the 100Ω resistor and capacitors to the tip of the 3.5mm jack

Connect the sleeve of the 3.5mm jack to ground

Use CircuitPython to implement audio playback:

```
python
import board
from audiocore import WaveFile
from audiopwmio import PWMAudioOut

audio_out = PWMAudioOut(board.GP0)

def play_wave(filename):
    with open(filename, "rb") as wave_file:
        sample = WaveFile(wave_file)
        audio_out.play(sample, loop=True)
This setup provides a simple mono audio output through a 3.5mm jack. While it may not deliver high-fidelity sound, it's a straightforward method for basic audio projects using the Raspberry Pi Pico.
```

## Could we add a potentiometer instead of a resistor on the GAIN pin to smoothly control output volume?

No, it's better to add a volume control at the microcontroller stage - so potentiometer coming into a Pico Pin that digitally controls the volume of sound in code

More detail: The MAX98357A's GAIN pin is designed for fixed gain settings via specific resistor configurations, not analog voltage control. While a potentiometer can be physically connected to the GAIN pin, it won't provide smooth volume adjustment due to the amplifier's discrete gain levels (3dB, 6dB, 9dB, 12dB, 15dB) triggered by predefined resistor values or direct connections