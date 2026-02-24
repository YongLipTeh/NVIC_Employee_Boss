# NVIC: The Employee-Boss Relationship
In this experiment we will investigate the relationship of different priority levels in interrupt controllers. This is a concept that only exists in the hardware level of microcontrollers. We will see how a higher priority prompt overrides the low priority ones and how we can apply primask to enable the employee to temporarily ignore its boss before serving them.

NVIC is unquestionably one of the most important hardware in the silicon die of arm Cortex-M series processors. It enables the CPU, which is single-cored, the freedom to perform more important logic tasks most of the time. Whether it is an alarm detecting a fire or a drone taking a picture, NVIC plays an important role as an alarm clock for the CPU so that it can respond only when it is necessary.
# Priority Levels
NVIC in STM32F446RE supports 4 bits of priority and 16 different priority levels (0 is the highest and 15 is the lowest). A high priority signal can override a low priority one. In this project, we are going to prove it with simple LEDs to demonstrate the employee-boss relationship.

<img width="1310" height="797" alt="Screenshot 2026-02-25 at 2 36 45 AM" src="https://github.com/user-attachments/assets/95cb1b12-1718-44f8-a968-313aeb01fb0d" />

Figure 1 shows a circuit diagram of STM32 F446RE. The green LED and first button show the employee; the red LED and the second button show the boss.

In the circuit diagram (Figure 1) We have two LED and two buttons, the first button controls the first (green) LED – employee while the second LED controls the second (red) LED – boss. The green LED will turn on for a longer time compared to the red LED. We placed an empty for loop incrementing variable to simulate a delay for the LED. The first button (connected to PA0) has a priority level of 1 while the second button has a priority of 0 (highest).

<img width="763" height="989" alt="image5" src="https://github.com/user-attachments/assets/0166c4f5-37ce-4779-8390-13a36d368116" />

Figure 2 shows the employee and the boss both lit up at the same time.

Pressing the first button will light up the green LED while pressing the second will light up the second LED. Since the boss (red LED) has a higher preemption priority than the employee (green LED). It can interrupt the employee any time. During the loop, the red LED is able to turn on when the second button was pressed. The red LED has overridden the green LED’s loop and turn the red LED on. The boss is superior here.

<img width="521" height="247" alt="image6" src="https://github.com/user-attachments/assets/f87cd770-a102-4528-b6b8-a0b8c52ac1bc" />

Figure 3 shows the second LED being pressed first and the first LED pressed later.

Despite both buttons being pressed, the green LED does not turn on when it is pressed after the second button. The green LED has to wait for the red LED to finish its loop before interrupting. The employee cannot interrupt the boss. Note that the green LED will still turn on after the red button turns off (loop is over), it means the CPU is just holding on to the first button’s request (not ignoring it).

The downside to this type of interrupt is it relies on the Interrupt Set-Pending Register (ISPR) to record pending event. Thus, even when I pressed the button several times, all of these requests are going to be read as a single interrupt. Therefore, multiple requests will only yield a single interrupt because the CPU is checking the value of the flag that was turned on by NVIC.
# Primask
We can allow the employee to hold off the boss’s interrupt request using *__disable_irq()*, the employee is allowed to temporarily finish all its critical tasks before responding to the boss using *__enable_irq()*. Therefore, even the boss has to wait for the employee before overriding it. Once *__enable_irq()* is called, the CPU check for any pending bit, if there were requests during the pause, the CPU will service it (the boss).

To demonstrate it, we can set put a simple wait for loop inside the primask and see the boss (red LED) wait for its employee to finish its tasks before taking over. *HAL_Delay()* should not be used because it is a low priority interrupt, so if it is called by a higher-priority interrupts, the system can never cut in and check uwTick, thus causing it to be stuck forever. 

<img width="477" height="280" alt="image7" src="https://github.com/user-attachments/assets/e5c0ff6a-62f6-44bb-996b-8bca590a9351" />

Figure 4 shows button 2 being pressed after the first button was pressed. However, the red LED does not light up immediately. It only lights up after a few seconds.

To prove that the boss is still superior, after the red LED turns off, we can press the second button and see the red LED lights up immediately. This shows that the *__disable_irq()* and *__enable_irq()* primask in action, and the boss’s interrupts are not ignored, but hold off in the pending bit.


