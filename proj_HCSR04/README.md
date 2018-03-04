# Sonar with HC-SR04

### Working with the HC-SR04 
The HC-SR04 is a sonar sensor module, which was really widespread in the Arduino community for robots or cars that need to detect walls or obstacles. I have one lying around, so I decided to see if I can get it working with my BP. This module is really easy to use, and I'm sure there are throngs of tutorials out there already, but this is MY REPOSITORY. In any case, I'll be brief.
HC-SR04 has 4 pins, VCC, Trig, Echo, and GND. You can probably guess what they do. To fire the sonic burst wave, you need to supply a 10&micro;s TTL signal to the Trig pin. When the burst is fired, the Echo pin goes high and returns to low once the echo of the burst is detected.

![Source: www](https://github.com/bot1131357/stm32_cortex/blob/master/img/HCSR04_timing_diagram.png "Timing diagram for HC-SR04")

#### 3.3V and 5V tolerance
The module works on 5V TTL logic, so this is something worth paying attention to when connecting to something that operates at a different voltage level. Fortunately for us, STM32F030F4 plays well with 5V TTL logic (at least from the spec it should.)

For the input side: the STM32F030F4 datasheet mentions that the I/Os are 5V tolerant, so we should be able to read the echo signal from the 5V TTL HC-SR04 without cooking the chip. I decided to play safe so I connected a 2k7 resistor anyway. What's the worst that could happen? (Dear universe: no, that's not an invitation for you to do your worst. Sincerely, your humble constituent) 

As for the output to trigger the HC-SR04, is the voltage level of STM32F030F4P6 sufficient? Turns out it is. At V<sub>DD</sub>=3.3V, the minimum output voltage level would be about 2.3V which is still safely higher than TTL's V<sub>IH</sub> of 2V. So spec-wise we're fine.

All good to go.
 
#### Coding
I must confess, the last time I really got my hands dirty with STM32 was 2013. Lots have changed since, and in a good way too: now we have STM32CubeMX, which really helps getting the bulk of the code generated for your project needs. Need a GPIO output? BAM! Need a pull-up? BAM! Need a PWM? BAM! I'm pleased to say that the GUI interface makes configuring your microcontroller as challenging as ordering a meal from ~McDona~ a fast food restaurant. One you're happy with your configurations, just generate all the boiler plate code. 

#### Here's the plan

```
___________
|          |
|          |
|      VCC====----5V 
|     TRIG====----PA3 (GPIO_Output)
|     ECHO====----PA2 (EXTI2 GPIO_MODE_IT_RISING_FALLING)
|      GND====----GND
|          |
|          |
------------
```

When the ECHO pin goes on high, a timer will start counting until the pin falls to low again. The number of counts will be used to calculate the distance.


#### 10&micro;s pulse 
To generate the pulse, I could make waste time with NOPs. Here's an microsecond delay routine in assembly I've copied from [STM32 tutorial](https://www.carminenoviello.com/2015/09/04/precisely-measure-microseconds-stm32/) :
```C++
#define delayUS_ASM(us) do {\
	asm volatile (	"MOV R0,%[loops]\n\t"\
			"1: \n\t"\
			"SUB R0, #1\n\t"\
			"CMP R0, #0\n\t"\
			"BNE 1b \n\t" : : [loops] "r" (16*us) : "memory"\
		      );\
} while(0)
```

...or we could use one of the many hardware timers available on this little badass. Which one to use? I guess it just depends on how badly you need to optimise. If you need to MCU to sleep or do something else during that 10&micro;s, then hardware timer would be the way to go, me thinks. Using the delay routine for now.

***Actually...***
The 10&micro;s pulse is probably just the shortest pulse allowed to trigger the burst. I have tried with 1ms, and it still worked fine. So HAL_Delay(1) it is.


#### External interrupt
On Echo's rising edge, TIM3->CNT is restarted and timer is started, and then stopped on falling edge.

```C++
void EXTI2_3_IRQHandler(void)
{
	if (HAL_GPIO_ReadPin(GPIOA,GPIO_PIN_2))
	{
		// on rising edge
		sonarBegin();
	}
	else
	{
		// on falling edge
		sonarEnd();
	}
	HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_2);
}
```

#### Utility functions
Some functions that I used : 

```C++
void sonarPing(); 		// start blasting sonar wave
void sonarBegin(); 		// when rise is detected, let the counter run
void sonarEnd(); 		// when the fall is detected, stop the counter and calculate distance
void sonarTimeout(); 	// called when timeout happens
```

Source file:
```C++
#include <stdbool.h>
uint32_t ticks=0;
float distance=0;

bool timeout=false;

void calculateDistance()
{
	ticks = TIM3->CNT;
	distance = 343*ticks / 2e6 / 2;
}

#define delayUS_ASM(us) do {\
	asm volatile (	"MOV R0,%[loops]\n\t"\
			"1: \n\t"\
			"SUB R0, #1\n\t"\
			"CMP R0, #0\n\t"\
			"BNE 1b \n\t" : : [loops] "r" (16*us) : "memory"\
		      );\
} while(0)


void sendPulse()
{
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_3,GPIO_PIN_SET);
//	delayUS_ASM(1);
	HAL_Delay(1);
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_3,GPIO_PIN_RESET);

}
void setPWM()
{
	TIM14->ARR = ticks >> 4;
}

void sonarPing()
{
	sendPulse();
}
void sonarBegin()
{
	HAL_TIM_Base_Start_IT(&htim3);
	TIM3->CNT = 0;
}
void sonarEnd()
{
	volatile int test = GPIOA->IDR & GPIO_PIN_2;
	HAL_TIM_Base_Stop_IT(&htim3);
	if(!timeout)
	{
		calculateDistance();
	//	setPWM();
	}
	else
	{
		distance = 0.123456789;
	}
	timeout=false;
}

void sonarTimeout()
{
	volatile int test = GPIOA->IDR & GPIO_PIN_2;
	volatile int test2 = TIM3->CNT;
	timeout=true;
	// if over 30 ms
	HAL_TIM_Base_Stop_IT(&htim3);
	TIM3->CNT = 0;
}
```



#### Timeout
As I've mentioned earlier, the Echo pin goes to high and returns to low once the echo of the burst is detected. But what if the echo wasn't detected? 

![Source: www](https://github.com/bot1131357/stm32_cortex/blob/master/img/sonar_no_echo.png "Timing diagram for HC-SR04")

Unfortunately, HC-SR04 doesn't ever time out on its own, and it would remain high. 

![Source: www](https://github.com/bot1131357/stm32_cortex/blob/master/img/high.jpg "I'm so high...")

What can you do about this? I suppose you could implement a timeout, using TIM3 update interrupt, then reset the HC-SR04 by having the GND pin connected to an NPN transistor switched by the BP. Let me know if that works for you. Because I doubt I'll write anymore in a long time.

Until the day where my life depends on the reliability of this sonar system, I'm perfectly contented assuming that the echo will always return. Doubt that day will ever come, but never say never. ;-)
 
