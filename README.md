# STM32 Cortex: Stuff I do, and things I learn

## STM32F030F4: The Blue Pill (shall be referred to as BP)
Back story: I chanced upon this STM32 board a while back on AliExpress. For $2.48 you get STM32F030F4P6 (20pins, 16K flash, TSSOP, -40 to 85 &deg;C)  32bit microcontroller choke full of features? I just had to give it a go. The item description was:

*48MHz STM32F030F4P6 Small Systems Development Board STM32 CORTEX-M0 Core Mini System Development PCB Panels ARM 32 Bit DIY AU$ 2.48*

The Micro-USB jack on board seems to only provide power, so you will need to have your own programmer/debugger. Luckily, you can also get one for a very cheap price:     

*1PCS ST LINK Stlink ST-Link V2 Mini STM8 STM32 Simulator Download Programmer Programming With Cover AU$ 2.39*

It's a shame not to get my grubby hands on one of this so... I got one. 

Admittedly, this isn't a tutorial. Just a place to jot down notes on projects I have done so that I can retrace my steps. So don't expect properly stuctured codes. Yea, I'm sloppy like that. 


Contents:
- [Sonar with HC-SR04](https://github.com/bot1131357/stm32_cortex/tree/master/proj_HCSR04/)
- 
- 

## Notes
### Clock stuff
Clock sources: 
- High Speed Internal (HSI)
- High Speed External (HSE)
- Phase-Locked Loop (PLL)

Clocks in the system
- SYSCLK (System) - Derived from HSI, HSE and PLL
- HCLK (Host?) - Signal to the peripherals
- FCLK (Free-running) - Runs even when sleeping


