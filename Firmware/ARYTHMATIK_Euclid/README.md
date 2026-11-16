# EUCLID + XOX — A-RYTH-MATIK firmware

A modified version of the Modulove **Euclid** firmware for the A-RYTH-MATIK (Arduino Nano / ATmega328P, SSD1306 OLED). It keeps the euclidean sequencer and adds step programming per channel, an autosaved working state, a mute mode for live use, and a few fixes.

Branch `eucli-xox` of [jkocontreras/A-RYTH-MATIK](https://github.com/jkocontreras/A-RYTH-MATIK), based on [modulove/A-RYTH-MATIK](https://github.com/modulove/A-RYTH-MATIK).

## Quick guide

### Main menu
Turn the encoder to move through the menu on the right column: channels **1–6**, **RND**, **X**, **S**, **L**, **T**, **R** and **M**.

### Channel menu
On a channel, a click moves to the next setting: **HITS → OFFS → LIMI → RESET → RANDOM → PROB → XOX**, then back to the top. A long click goes back one setting. Turning changes the value of the selected setting.

- Pressing and turning the encoder on a channel at the top level changes its HITS directly.
- A long click on a channel at the top level mutes or unmutes it.
- When the left column is empty, it shows the current BPM, written vertically.

### XOX (step programming)
XOX is the last setting of the channel menu.

- Turn to move the cursor, a filled diamond, through the steps up to the LIMI length.
- Click to turn the step under the cursor on or off. The diamond blinks when the step is on.
- Long click to leave XOX.

The first edit switches the channel to XOX and starts from the euclidean pattern that was playing. OFFS rotates the drawn pattern. Changing HITS, or a random change of the hits, returns the channel to euclidean mode; HITS starts from the number of active steps of the drawing.

### Ring numbers
- In LIMI, each ring shows its number of steps at the center instead of the polygon.
- With more than 8 hits, the ring shows the number of hits at the center. In XOX it counts the active steps.

### Mute (M)
- Click on **M** to enter mute mode. Turn to choose a channel and click to mute or unmute it. Long click to leave.
- Long click on **M** outside mute mode mutes all channels and shows **MUTE** on screen. A click unmutes them all.
- A muted channel is drawn as a white circle with a black M.

### Memory
- **Working state:** the current state is saved automatically 10 seconds after the last encoder action, and it comes back at power-up. It is written one byte per loop pass, so playback keeps running while it saves.
- **S / L:** click to open, turn to choose one of 20 slots, long click to save or load.
- Slots use a compact 41-byte format that includes the XOX patterns. Slots from the previous Euclid format are converted automatically at the first boot.
- The 20 factory presets were removed. A Nano without saved slots starts with 4 hits in 16 steps on every channel.

### Clock
- **T:** click to see the tempo, turn to change it (40–240 BPM). Long click switches between internal and external clock.
- With an external clock, the BPM is measured in microseconds and averaged over about 4 pulses, so the reading stays steady.

## Changes from the original Euclid firmware

**Sequencing**
- XOX step programming mode per channel.
- MUTE was removed from the channel settings; channels are muted from mute mode or with a long click on the channel.

**Memory**
- Autosaved working state, written without stopping playback.
- Compact slot format (`EUCLID11`) with XOX patterns and automatic migration from `EUCLID10`.
- Factory presets and the preset menu removed to free flash memory.

**Display**
- Full-screen EUCLID + XOX splash.
- Step and hit numbers at the center of the rings.
- BPM in the empty left column.
- Numbers drawn with the JARA pixel font (`JaraNumeros.h`, digits only). Text and menu letters use the system font.
- SAVE, LOAD and TEMPO screens left-aligned with more line spacing.
- Removed the small animated pixel noise in the lower right corner. It came from the stack reaching the display buffer; RAM use was reduced to fix it.

**Clock and LEDs**
- Steadier external BPM reading.
- Dimmer LEDs: the LED flash lasts 5 ms and the screen redraws after it. Output triggers keep their original length (about 30 ms).
- `ENCODER_REVERSED` is enabled. Comment it out at the top of the `.ino` if your encoder turns the wrong way.

**Fixes**
- RND cycles through 2, 4, 6, 8, 12 and 16 bars. The original never reached 2 and read outside the table after 16.
- With LIMI above 16, the extra steps repeat the pattern instead of reading the next channel's data.
- Steps are stored as 16-bit masks per channel and polygons are drawn without a point buffer, which frees RAM.

## Building and flashing

Requires the Arduino AVR core and the libraries FastGPIO, Adafruit GFX, Adafruit SSD1306 and EncoderButton.

```bash
arduino-cli compile -b arduino:avr:nano ARYTHMATIK_Euclid
arduino-cli upload  -b arduino:avr:nano -p <port> ARYTHMATIK_Euclid
```

Nanos with the old bootloader use `-b arduino:avr:nano:cpu=atmega328old`. The firmware uses 30180 of the 30720 bytes available with the old bootloader, so there is little room for new features.

Flash the module with only the USB cable connected, without Eurorack power.
