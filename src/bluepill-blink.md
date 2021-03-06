# Blink an LED

In this section we will write an application that will raise the system clock
frequency to 72 MHz and blink the on-board LED connected to the PC13 pin. The
application will involve using multiple threads, futures, streams, memory-mapped
registers, and peripherals.

<video autoplay loop muted width="100%">
<source src="./assets/blink.webm" type="video/webm" />
<source src="./assets/blink.mp4" type="video/mp4" />
</video>

The full code for this example can be found at
[Github](https://github.com/drone-os/bluepill-blink).

## Generate a project

To begin with, let's generate a new Drone project for a Blue Pill board:

```shell
$ drone new \
        --toolchain nightly-2020-04-30 \ # we need to pick a fresh nightly with
                                       \ # all required rustup components
        --device stm32f103 \ # microcontroller identifier
        --flash-size 128K \ # flash memory size in bytes
        --ram-size 20K \ # RAM size in bytes
        bluepill-blink # project name
$ cd bluepill-blink
$ just deps
```

To briefly test the newly generated application, connect a Black Magic Probe to
your PC, and a Blue Pill board to the BMP as in [Hello,
world!](./hello-world.md) chapter. Flash the firmware and check the SWO output:

```shell
$ just flash
$ just log
```

If you can see a "Hello, world!" message, follow to the next chapter.
