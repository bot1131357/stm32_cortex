# STM32 Cortex: Stuff I do, and things I learn

## STM32F030F4: The Blue Pill (shall be referred to as BP)
Back story: I chanced upon this STM32 board a while back on AliExpress. For $2.48 you get a 32bit microcontroller, choke full of features? I just had to give it a go. The item description was:

48MHz STM32F030F4P6 Small Systems Development Board STM32 CORTEX-M0 Core Mini System Development PCB Panels ARM 32 Bit DIY AU$ 2.48

The Micro-USB jack on board seems to only provide power, so you will need to have your own programmer/debugger. Luckily, you can also get one for a very cheap price:     

1PCS ST LINK Stlink ST-Link V2 Mini STM8 STM32 Simulator Download Programmer Programming With Cover AU$ 2.39

### Working with the HC-SR04 
The HC-SR04 is a sonar sensor module, which was really widespread in the Arduino community for robots or cars that need to detect walls or obstacles. I have one lying around, so I decided to see if I can get it working with my BP. This module is really easy to use, and I'm sure there are throngs of tutorials out there already, but this is MY REPOSITORY. In any case, I'll be brief.

HC-SR04 has 4 pins, VCC, Trig, Echo, and GND. You can probably guess what they do. 

(Insert image here) 














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


