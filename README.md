# STM32F103C8T6-Familiarization
Getting started with the Blue Pill STM32F103C8T6 development board, and exploring its more advanced features
___

The STM32F030F4P6 is a great microcontroller, for sure. But because it's a bottom-of-the-barrel STM32, I couldn't help but wonder after using it "what else can I do with the nicer ones?"

Well, I need wonder no more... because I decided it was time to dive take the Blue Pill.

![Matrix](https://i.imgur.com/038OHfp.png?1)

Alright, alright. Not that "blue pill," this one:

![Blue Pill](https://i.imgur.com/Wti2PAJ.png)

This is the STM32F103C8T6 Minimal Development Board, otherwise known as "The Blue Pill". It's called that because, well, it's blue. And the cost is about as easy to swallow as a pill. At about a buck a piece, you definitely get your money's worth.

On it is a voltage regulator, low-speed oscillator, high-speed oscillator, USB port (we'll get to that later), and of course; the STM32F103C8T6. This is a step-up from the F0, based around a (comparatively) ARM Cortex-M3 operating at 72MHz with 128KB of Flash, and 20KB of SRAM (over 10,000 times my original PIC10F220 project has!)

Let's fire up the ol' CubeMX to see what she's got.

![STM32F103](https://i.imgur.com/PhJolin.png)

Not bad! A couple ADCs, a RTC, 4 timers, 2 each of I2C and SPI, 3 UARTs, and the ability to natively support USB. This part is the interesting part to me... Everything else is just more of the F0's peripherals, but this is new. No need for a UART to USB converter with this chip... you have full control over how it shows up to a host device as a peripheral. CubeMX has presets for various commonly-found USB devices, like "Human Interface Device" and "Virtual COM Port". That's awesome, because that means you don't necessarily have to write drivers for whatever device you're plugging into. Windows, Linux, and Android all have built-in drivers for the Virtual COM Port mode, and interacting with it is as simple as plugging it in, opening up a terminal, and sending data.

So how do we send and receive data over USB? Well, it's quite simple.

![USB CDC](https://i.imgur.com/nOt7TnK.png)

In the `usbd_cdc_if.c` file there's a function named `CDC_Receive_FS` that's called whenever data comes across the Virtual COM Port. Here, I'm simply checking if the data's a `1` or a `0`, and then turning on or off the onboard LED accordingly. My code's purposely a little verbose, so it's easy to figure out what's going on.
