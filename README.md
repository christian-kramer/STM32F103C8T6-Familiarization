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


Next, I wanted to try exposing the STM32 to the PC as a Human Interface Device. Building your own peripherals is a seriously powerful skill, and one that I definitely wanted to learn.

To do this, we'll need to make our own custom USB HID class. We're doing custom because I want to try making my own gamepad device in the near future, instead of a boring old regular keyboard or mouse. Here is how I setup CubeMX:

![Custom HID](https://i.imgur.com/B3hr0ZI.png)

I've provisioned Channel 0 of ADC1, and GPIO pin A1 as our two inputs... one analog axis, and one push-button.

Next, we need to write our USB Device Descriptor. Since we're doing this custom, this is something we'll need to figure out ourselves.

For now, I'm going to start off easy and describe just the axis.

![1axis gamepad](https://i.imgur.com/nyPE8WF.png)

Here, we can see that I'm defining an axis that has values from 0-4095 (12 bits, because that's the resolution of our ADC) and because USB HID requires us to byte-align our values, we're going to put that 12-bit integer inside a 16-bit space. I've defined this axis as "steering", as in "car steering" because I'm planning on using this custom gamepad device for driving and racing simulators, eventually.

If we go to File -> Info, we'll see that this descriptor is 22 bytes.

![22](https://i.imgur.com/QlhOw4Q.png)

And... that's all there is to it! Let's fill this in.

After generating our CubeMX code, we're going to find the file called `usbd_conf.h` inside our Includes folder. This header file contains our definitions for the descriptor. The  only thing we care about at this very moment in here is the `USBD_CUSTOM_HID_REPORT_DESC_SIZE` definition. It got generated as 2, but our descriptor is 22 bytes. We gotta change it to:

![Descriptor Length](https://i.imgur.com/33pENwL.png)


Next, we're going to find the file called `usbd_custom_hid_if.c` in our source folder. This is going to be where we plug the numbers from our descriptor in.

![Descriptor Transcription](https://i.imgur.com/WZeeFeb.png)

And, yes, `END_COLLECTION` is included in the 22 bytes of the descriptor. It was nice of CubeMx to do that one for us.

Let's write our main loop.

![Main Loop](https://i.imgur.com/EEU1Qti.png)

Originally I had the STM32 generate random ADC values before I actually hooked up a potentiometer to the ADC. (My first driving AI??) Next, to make it less sporadic, I had it swep through the full ADC range. Finally, I wrote the non-commented part which actually polls Channel 0 of ADC1, and sends an HID report after each reading.

Compile, and...

![HID Test](https://i.imgur.com/njEPSQ6.png)

Nice! It shows up exactly how we expected it to. We've got our GamePad icon with "Christian's Custom Human interface", it's recognized as a controller with an OK status, and our Steering axis shows up! Now, let's add that button.

First, we need to change our descriptor. It's going to look a lot more complicated, but I promise it's easy to interpret.

![Button and Axis](https://i.imgur.com/Dn5qQjT.png)

The only change we really made was adding another "collection" underneath our axis input word. Inside this collection we've defined the following as "buttons", and we're using the `USAGE_MINIMUM` and `USAGE_MAXIMUM` words to confine the number of buttons that will show up as 1. This will make sense in a moment. `LOGICAL_MINIMUM` and `LOGICAL_MAXIMUM` are to indicate that the buttons can either be "on" or "off". `REPORT_SIZE` is going to be 1 byte, because  that's all we need for "on" or "off". However, we need to transmit at least 8 of these bytes, because otherwise Windows will complain that our descriptor is not "byte aligned". There may be a more efficient way to do this, but for now this is what I've found to work with no noticeable performance hit. So, now you can see why we need to indicate that our `USAGE_MINIMUM` and `USAGE_MAXIMUM` both need to be 1... if we made the minimum 1 and the maximum 8, Windows would see 8 buttons. Which we could totally do, but I just want one for now.

Alright, let's implement this new descriptor.

Remember to change the config.

![USB HID Config](https://i.imgur.com/sN4L0Po.png)

![New descriptor](https://i.imgur.com/VIxZggS.png)

Next,  our main loop:

![Main loop](https://i.imgur.com/issb99z.png)

I'm taking advantage here of the fact that `HAL_GPIO_ReadPin` returns either 1 or 0, depending on the state of that pin. The rest of the bits in that byte are the rest of the 8 buttons if you so choose to implement. If you need more than 8 buttons, adjust the `REPORT_SIZE` and `REPORT_COUNT` as needed.

Let's see how it shows up.

[Gamepad Demo](https://i.imgur.com/ZbpbU08.mp4)

Awesome! Both "Steering" and "Button 1" work great!

![Interface](https://i.imgur.com/KuqTbTO.jpg)

Here's the interface I came up with, for the button and steering axis. The 1µF/100nF capacitors and resistor on the potentiometer help to filter the noise going to the ADC.

Let's try it in an actual game.

[Driving a truck with the STM32](https://i.imgur.com/QYTzxKI.mp4)

Ignore that ending, that... that wasn't supposed to happen...
