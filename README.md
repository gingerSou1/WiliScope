# WiliScope v001

A standalone FREE-WILi 2 (FX0141) application for viewing slow analog voltage changes. **This version is a voltage trend monitor, not a high-speed oscilloscope.** It uses real TLA2024 voltage readings through the stock MAIN firmware. It contains no simulated waveforms.

Status: compiled and checked as a RAM-loadable DISPLAY application. Not yet loaded or tested on physical hardware. Voltage accuracy, channel mapping, touch, buttons, power behavior, frame rate, and recovery still require device testing.

## Features

- CH1 through CH4 selection, corresponding to API channels A0 through A3; one channel plotted at a time.
- Back-to-back split-phase voltage requests, one in flight at a time, without the former 50 ms throttle. The screen reports the observed returned-reading rate. Points use actual receipt times, not ADC conversion timestamps; repeated values may represent the same conversion.
- 2,048 four-channel records; timebase 50, 100, 250, 500, 1,000, or 5,000 ms per horizontal division (10 divisions). Available history duration depends on the observed reply rate; an unfilled part of the window stays empty.
- 0.1, 0.2, 0.5, 1, 2, or 5 V per vertical division. Zero volts is at the bottom; CLIP identifies values outside the vertical display range.
- Run/Hold; minimum, maximum, average, and peak-to-peak over the visible time window.
- Waveform, Measurements, and Settings pages; MAIN response status and last-reading age.
- A prominent live voltage readout and waveform grid. Framebuffer rendering uses asynchronous DMA for the chart and redraws header/footer bands only when needed. Routine chart updates are limited to 10 per second; control changes can update immediately. Actual flicker and responsiveness need hardware verification.

## Connections

Use the **10-position analog connector**, not the digital GPIO connector. Connect the signal to the input labeled A0 (CH1), and connect the circuit ground to the device's labeled GND. Select CH2/A1, CH3/A2, or CH4/A3 as needed. Confirm labels against your board before wiring: the public pinout describes the analog connector but does not provide a numbered ten-pin table, so no physical pin numbers are guessed here.

The documented analog input range is **0 to 5 V**. The ADC's selected 6.144 V full-scale setting does not increase the connector's voltage limit. Use low-voltage, ground-referenced circuits within that range; do not connect negative voltage or mains. Dupont leads have no probe attenuation or isolation.

## Controls

| Control | Action |
| --- | --- |
| Left / Right | Select channel, voltage scale, or timebase |
| Up / Down | Cycle selected value forward / backward |
| Check / navigation center | Cycle selected value forward |
| X / Cancel or short Home | Return to Waveform page |
| Page | Cycle Waveform / Measurements / Settings |
| Hold Page for 5 seconds | Show version and repository; release to return |
| Hold Home for 5 seconds | Normal watchdog restart back to stock DISPLAY firmware |
| Grey | Run/Hold |
| Yellow | Next voltage scale |
| Green | Next timebase |
| Blue | Next channel |
| Red | Next page |

Touch the five colored labels for the matching actions. Touch a channel, scale, or timebase tile to select and cycle it. Touch LESS / MORE to adjust the selected value in either direction. Touch the top-right Run/Hold button to pause or resume. Changing channel, scale, or timebase reuses the existing four-channel history, including while held. Resuming Run starts a new history. Hold stops new measurement requests; an already in-flight reply is drained without adding it to the held history.

## Installation

1. Keep the existing stock MAIN and DISPLAY firmware installed. MAIN must support the OneWili DISPLAY bridge and TLA2024 commands.
2. Copy `WiliScope-v001.uf2` to `/apps/` on the device's SD card using the normal SD file-transfer procedure, then return card ownership to MAIN.
3. Launch it through the stock App Explorer. This file is a RAM application, not a replacement firmware image. Do not install it through a MAIN or DISPLAY firmware-update workflow.
4. Hold Home for five seconds to return to stock firmware.

No firmware flashing or deployment was performed during development. No MAIN firmware changes are required or included.

## Acquisition limitations

The TLA2024 API returns the latest four voltages. v001 configures channels for single-ended A0-GND through A3-GND, 6.144 V full scale, and the maximum documented 3300 SPS converter setting. That nominal converter setting is not the application's delivered sample rate. The app sends the next read after the previous reply, subject to the event loop and display work; it does not impose a fixed 20 Hz polling limit. The documented converter settings span 128 to 3300 SPS; the four-channel scan, stock firmware update cadence, and command transport further constrain observations. No maximum trustworthy waveform frequency is claimed.

The stock OneWili API also exposes buffered analog capture through its logic-analyzer subsystem: an analog channel mask, sampling interval, 8/16-bit storage, and capture-report metadata. However, the official OneWili DISPLAY transport explicitly does **not** mirror logic-analyzer reports. No supported buffered analog retrieval path to this standalone app was found. The inspected public interfaces do not establish a usable DISPLAY capture buffer size or sustained capture rate.

Consequently v001 has no buffered waveform acquisition, hardware sample timestamps, pretrigger data, hardware trigger, or software edge-trigger mode. Trigger is labeled FREE/unavailable. The separate analog-output/window-comparator controls are not enabled. A high-speed standalone version needs an official MAIN-to-DISPLAY analog-buffer transfer interface; adding that upstream would be the smallest practical hardware-preserving route, but it is outside this build and was not attempted.

The desktop GUI's PicoScope integration requires external PicoScope hardware plus PicoSDK/runtime drivers. It is not an acquisition implementation for the Wili's own inputs.

## Hardware behavior and first test

The app requests DISPLAY and ANALOG power zones. An app-local adapter preserves inherited VIO selection, antenna routing, and unrelated expander settings while preparing the LCD. It does not enable DAC outputs, programmable power, GPIO outputs, or glitching. It does not release unrelated power zones, so power consumption depends partly on inherited device state. ADC configuration is not restored on exit.

For the first physical test, start with A0 tied to GND, confirm a near-zero reading, then apply a known safe DC voltage and compare with a meter. Verify each channel, Run/Hold, all buttons, touch controls, and five-second Home recovery. Only then try a very slow changing input. Confirm the input is actually connected: MAIN OK means a successful API response, not probe/contact detection. An unsupported API or lost response shows API REJECTED, NO REPLY, or LINK ERROR and triggers delayed reconnection. MAIN OK describes the most recent successful voltage response; while held it is not a live connection probe. The Settings page shows the age of that response.

## Implementation reference

The available [WiliGlitch source](https://github.com/gingerSou1/WiliGlitch) supplied the application-loop structure, split-phase OneWili adapter pattern, small shared protocol buffers, press-edge touch behavior, and retained UI-update pattern. WiliScope adapts these for read-only analog acquisition and extends the physical-button mappings. GitHub WiliGlitch v003 is an older passive diagnostic; the owner identifies v010 as the DMM-verified output build on another machine. v010 source was not available for this build. WiliGlitch itself and the official dependencies were left unchanged.

## Release contents

This repository is intended to contain only `README.md` and `WiliScope-v001.uf2`. Development source, upstream dependencies, build trees, helper tools, and validation notes are retained locally under an ignored `.local/` directory. The local `.gitignore` is itself ignored.

Built with official WiliBSP / OneWili, Raspberry Pi Pico SDK 2.3.0, official Pico 2.3.0 Windows helpers, and Arm GNU Toolchain 15.2.Rel1. Build validation checks app metadata and RAM-only UF2 targets; these checks do not prove on-device operation.

## Official references

- [Application launch and FWApp contract](https://docs.freewili.com/files-and-apps/running-apps/)
- [WiliBSP app framework](https://github.com/freewili/wilibsp)
- [OneWili DISPLAY transport and its capture-report restriction](https://github.com/freewili/onewili/tree/main/wilibsp)
- [Analog input API](https://docs.freewili.com/features/analog-in/)
- [Buffered analog capture commands](https://docs.freewili.com/features/logic-analyzer/)
- [Hardware pinout and analog voltage range](https://docs.freewili.com/hardware/pinout/)
- [Connectors](https://docs.freewili.com/hardware/connectors/)
- [Desktop GUI requirements](https://github.com/freewili/freewili-gui)

## Third-party notices

The binary includes components from WiliBSP and OneWili (MIT), the Raspberry Pi Pico SDK (BSD-3-Clause), and SEGGER RTT. Upstream license texts and attribution notices follow.


### WiliBSP (MIT)

MIT License

Copyright (c) 2026 Dave Robins

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

### OneWili (MIT)

MIT License

Copyright (c) 2026 Free-Wili

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

### Raspberry Pi Pico SDK (BSD-3-Clause)

Copyright 2020 (c) 2020 Raspberry Pi (Trading) Ltd.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the
following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following
   disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following
   disclaimer in the documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products
   derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES,
INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

### SEGGER RTT attribution

Copyright SEGGER Microcontroller GmbH. SEGGER RTT: https://github.com/SEGGERMicro/RTT. Included unmodified from the official WiliBSP dependency.



