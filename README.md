# Pimhyp4

This repository contains an experimental kernel driver for the Pimoroni Hyperpixel4 touchscreen display.

It is a hybrid driver that combines a minimally modified version of the Goodix 911 touchscreen driver, plus initialisation code for the LCD based on Pimoroni's python initialisation code.

The 911 touchscreen chip can appear at 2 different i2c addresses. Due to insufficient GPIO pins, the RESET line is not accessible and the i2c address can therefore vary between 0x14 and 0x5d. Typically it is 0x14 on a Pi4 and 0x5d on other Pis.

The pimhyp4-overlay.dts file describes both of these addresses, so that a driver will be loaded for either address that appears, assuming only one will appear at a time.

The LCD is initialised over a bitbanged SPI bus. This is because one of the pins is shared with the INTERRUPT pin of the touch controller.

The config.txt file shows 4 sets of configuration options to allow the 4 full rotations of the display and the touchscreen. This display is natively a 480x800 portrait orierntation, so rotate_1 or rotate_3 is normally required for landscape orientation.

# Status

Currently, the driver detects the touchscreen driver on a Pi3 or Pi4 and initialises the display correctly for framebuffer mode.

On the Pi4, with dtoverlay=vc4-fkms-v3d enabled, the display does NOT show the desktop, but continues to display the last boot messages on the LCD. However, if VNC is enabled, the VNC screen shows the screen that shoudl be ont he LCD, and the touchscreen mouse operates OK.

However, after a while of dragging my finger on the LCD display, an unhandled IRQ error occurs and the IRQ becomes disabled such that the mouse no longer works.

# Intitialisation messages

Here is the output of dmesg when the kernel driver is starting on the different models:

## Pi3
```
[    5.627591] pimhyp4-TS 3-0014: I2C Address: 0x14
[    5.631261] pimhyp4-TS 3-0014: i2c test failed: -6
[    5.661723] pimhyp4-TS 3-0014: I2C communication failure: -6
[    5.661909] pimhyp4-TS 3-005d: I2C Address: 0x5d
[    5.663882] pimhyp4-TS 3-005d: ID 911, version: 1060
[    5.663889] Detected: pimhyp4
[    5.663910] pimhyp4-TS 3-005d: touchscreen-refresh-rate not found
[    5.663923] pimhyp4-TS 3-005d: touchscreen-size-x found 480
[    5.663934] pimhyp4-TS 3-005d: touchscreen-size-y found 800
[    5.663944] pimhyp4-TS 3-005d: requested size (480, 800)
[    5.685759] pimhyp4-TS 3-005d: Current size = (480, 800), X2Y = 0, refresh = 5
[    5.685773] pimhyp4-TS 3-005d: Checking firmware
[    5.685784] pimhyp4-TS 3-005d: Requested X size = 480, orig = 480
[    5.685795] pimhyp4-TS 3-005d: Requested Y size = 800, orig = 800
[    5.685804] pimhyp4-TS 3-005d: Firmware already updated with correct values
[    5.685816] pimhyp4-TS 3-005d: Current size = (480, 800), X2Y = 0, refresh = 5
[    5.686112] input: Pimoroni Hyperpixel4 Capacitive TouchScreen as /devices/platform/i2c@0/i2c-3/3-005d/input/input1
[    5.686907] Using: pimhyp4
```
## Pi4
```
[    4.733531] pimhyp4-TS 7-0014: I2C Address: 0x14
[    4.735525] pimhyp4-TS 7-0014: ID 911, version: 1060
[    4.735537] Detected: pimhyp4
[    5.436178] Using: pimhyp4
[    5.436206] pimhyp4-TS 7-0014: touchscreen-refresh-rate not found
[    5.436222] pimhyp4-TS 7-0014: touchscreen-size-x found 480
[    5.436236] pimhyp4-TS 7-0014: touchscreen-size-y found 800
[    5.436248] pimhyp4-TS 7-0014: requested size (480, 800)
[    5.463059] pimhyp4-TS 7-0014: Current size = (480, 800), X2Y = 0, refresh = 5
[    5.463073] pimhyp4-TS 7-0014: Checking firmware
[    5.463086] pimhyp4-TS 7-0014: Requested X size = 480, orig = 480
[    5.463098] pimhyp4-TS 7-0014: Requested Y size = 800, orig = 800
[    5.463110] pimhyp4-TS 7-0014: Firmware already updated with correct values
[    5.463122] pimhyp4-TS 7-0014: Current size = (480, 800), X2Y = 0, refresh = 5
[    5.463380] input: Pimoroni Hyperpixel4 Capacitive TouchScreen as /devices/platform/i2c@0/i2c-7/7-0014/input/input1
[    5.464177] pimhyp4-TS 7-005d: I2C Address: 0x5d
[    5.464229] pimhyp4-TS 7-005d: Failed to get irq GPIO: -16
[    5.464291] pimhyp4-TS 7-005d: Failed to get mosi GPIO: -16
[    5.464329] pimhyp4-TS 7-005d: Failed to get cs GPIO: -16
[    5.464938] pimhyp4-TS 7-005d: i2c test failed: -6
[    5.501501] pimhyp4-TS 7-005d: I2C communication failure: -6
```
Here is the DMESG output when the IRQ error occurs:

## Pi4 Error
```
[   83.191891] i2c i2c-7: sendbytes: NAK bailout.
[   83.191912] pimhyp4-TS 7-0014: I2C transfer error: -5
[   83.976064] i2c i2c-7: sendbytes: NAK bailout.
[   83.976084] pimhyp4-TS 7-0014: I2C transfer error: -5
[   88.355184] i2c i2c-7: sendbytes: NAK bailout.
[   88.355204] pimhyp4-TS 7-0014: I2C transfer error: -5
[   89.717497] irq 57: nobody cared (try booting with the "irqpoll" option)
[   89.717508] CPU: 0 PID: 0 Comm: swapper/0 Tainted: G         C        4.19.75-v7l #6
[   89.717511] Hardware name: BCM2835
[   89.717533] [<c0212c74>] (unwind_backtrace) from [<c020d2cc>] (show_stack+0x20/0x24)
[   89.717542] [<c020d2cc>] (show_stack) from [<c097d4f0>] (dump_stack+0xcc/0x110)
[   89.717551] [<c097d4f0>] (dump_stack) from [<c02851e8>] (__report_bad_irq+0x4c/0xd0)
[   89.717558] [<c02851e8>] (__report_bad_irq) from [<c0284fcc>] (note_interrupt+0x124/0x2bc)
[   89.717565] [<c0284fcc>] (note_interrupt) from [<c0281e78>] (handle_irq_event_percpu+0x88/0x90)
[   89.717572] [<c0281e78>] (handle_irq_event_percpu) from [<c0281ed4>] (handle_irq_event+0x54/0x78)
[   89.717579] [<c0281ed4>] (handle_irq_event) from [<c0286148>] (handle_edge_irq+0xcc/0x1fc)
[   89.717585] [<c0286148>] (handle_edge_irq) from [<c0280ca4>] (generic_handle_irq+0x34/0x44)
[   89.717596] [<c0280ca4>] (generic_handle_irq) from [<c0628ba8>] (bcm2835_gpio_irq_handle_bank+0x94/0xcc)
[   89.717605] [<c0628ba8>] (bcm2835_gpio_irq_handle_bank) from [<c0628c54>] (bcm2835_gpio_irq_handler+0x74/0x12c)
[   89.717611] [<c0628c54>] (bcm2835_gpio_irq_handler) from [<c0280ca4>] (generic_handle_irq+0x34/0x44)
[   89.717618] [<c0280ca4>] (generic_handle_irq) from [<c0281408>] (__handle_domain_irq+0x6c/0xc4)
[   89.717625] [<c0281408>] (__handle_domain_irq) from [<c0202234>] (gic_handle_irq+0x4c/0x88)
[   89.717632] [<c0202234>] (gic_handle_irq) from [<c02019bc>] (__irq_svc+0x5c/0x7c)
[   89.717635] Exception stack(0xc1001ed0 to 0xc1001f18)
[   89.717640] 1ec0:                                     c0209a54 00000000 40000093 40000093
[   89.717645] 1ee0: ffffe000 c1004db8 c1004e00 00000001 00000001 c10962da c0b87ef4 c1001f2c
[   89.717649] 1f00: c1000000 c1001f20 00000000 c0209a58 40000013 ffffffff
[   89.717656] [<c02019bc>] (__irq_svc) from [<c0209a58>] (arch_cpu_idle+0x34/0x4c)
[   89.717664] [<c0209a58>] (arch_cpu_idle) from [<c0999158>] (default_idle_call+0x40/0x48)
[   89.717673] [<c0999158>] (default_idle_call) from [<c0254200>] (do_idle+0x134/0x174)
[   89.717681] [<c0254200>] (do_idle) from [<c0254500>] (cpu_startup_entry+0x28/0x2c)
[   89.717688] [<c0254500>] (cpu_startup_entry) from [<c09927e4>] (rest_init+0xb8/0xbc)
[   89.717697] [<c09927e4>] (rest_init) from [<c0e01000>] (start_kernel+0x4b0/0x4e0)
[   89.717702] handlers:
[   89.717707] [<5927cfa3>] irq_default_primary_handler threaded [<8d7f360e>] pimhyp4_ts_irq_handler [pimhyp4]
[   89.717728] Disabling IRQ #57
```