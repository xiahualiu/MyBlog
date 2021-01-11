---
title: "Bearcat EV Tasklist"
date: "2021-01-10"
draft: false
---

This file is the TODO list for the Bearcat EV software team, and also the I/O document. 

## State machine

> The state machine model needs to be a [Moore machine](https://en.wikipedia.org/wiki/Moore_machine), i.e. the control strategy only depends on the current state, not the inputs. Inputs only trigger the transition from one state to another.

### State list and transition condition

Except all `\*_WAIT` state and `ERROR` state, at each state, a watchdog timer must be created at the beginning, to prevent any dead loop error. 

#### INIT state

ECU enters this state when:

* The ECU was just powered on (GLVMS was switched on).

In this state:

* `ECU_OK` stays **LOW**. `PRECHARGE_FINISHED-` stays **HIGH**.
* ECU at this stage executes the initialzation tasks, such as interrupt table initialzation.

ECU leaves this state and moves to `RESET` when the initialization process has been finished.

#### RESET state

ECU enter this state when:

* ECU was successfully reset.
* ECU left `INIT` state.

In this state:

* `ECU_OK` stays **LOW**. `PRECHARGE_FINISHED-` stays **HIGH**.
* ECU resets all memory blocks.

#### RESET\_CONFIRM state

ECU enter this state when:

* The ECU was in `ERROR` state, AND the reset switch outside the cockpit was pressed **DOWN**. *Input Falling-Edge Interrrupt*.
  
This is the intermediate state from `ERROR` to `RESET`. In this state:

* the ECU **should NOT** change output signals.  

A count down timer is assigned in this state, when the reset button is released half way, ECU return to `ERROR` state.

If the time `RESET_HOLD_DURATION` runs out, and the reset botton is still pressed **DOWN**. The ECU enters `RESET` state. This is intended to prevent any accidental press.

**Note**: During the confirm, ECU still does routine check, if routine check fails, ECU immediately returns to ERROR state.
 
#### ERROR state

ECU enters this state when:

* An error occured in the shutdown circuit. *Input Rising-Edge Interrupt*
* The watchdog timer ran off. *Timer Interrupt*
* The reset was not successful.

In this state, `ECU_OK` stays **LOW**. `PRECHARGE_FINISHED-` stays **HIGH**.

Upon entering this state, ECU is trapped here for 10 seconds or more at a least (to ensure the state does not bounce back), after that in order to leave this state, (to `RESET_CONFIRM` state) the shutdown reset button has to be pressed **DOWN**. *Input Falling-Edge Interrupt*

#### ROUTINE\_CHECK state

ECU enters this state when:

* The scheduled data check was triggered.

In this state:

* ECU collects data from BMS, BSPD and motor controller.
* ECU checks itself for any memory errors. 
* the ECU **should NOT** change output signals. 

If data are not good, ECU enters the `ERROR` state. Transition to this state is scheduled with a timer during any `\*_WAIT` states. (Proper trigger interval 10ms~100ms).

To avoid false alarm, multiple data fetches can be made consecutively. Error will only be raised when the error data persist. 

#### PRE\_HV\_CHECK state

ECU enters this state when left `RESET` state.

In this state:

* ECU does similar job as `ROUTINE_CHECK` state. 

#### HV\_READY\_WAIT state

ECU enters this state when `PRE_HV_CHECK` state has finished.

In this state:

* ECU `HV_READY` outputs **HIGH**, `ECU_OK` outputs **HIGH**.

ECU leaves this state to `PRECHARGE` state, when `SHUTDOWN_TTL_OK-` shows a falling edge.

#### PRECHARGE\_WAIT state

ECU enters this state from `PRE_HV_CHECK` state.

In this state:

* ECU assigns a count down timer.
* ECU unmask `SHUTDOWN_TTL_OK` falling edge interrupt, from this point, if the shutdown circuit is open by any mean, ECU goes to `ERROR` state.

After the precharge timer runs out, meaning precharge is finished, ECU goes to `READY_TO_GO_WAIT` state.

#### READY\_TO\_GO\_WAIT state

ECU enters this state from `PRECHAGE_WAIT` state.

In this state:

* ECU plays the ready to go sound once.
* ECU `READY_TO_GO` outputs **HIGH**.

In this state, the driver is able to drive the car.

## ECU Peripheral Devices

* Brake system plausibility device
* Battery management system
* Motor controller

## Signal Table

| NAME | DIRECTION | OUTPUT TYPE | VOLTAGE LEVEL | COMMENTS |
| :--- | :---: | :---: | :---: | :--- |
| ECU\_OK | OUTPUT | PUSH PULL | TTL | Should be **LOW** ASAP once error is found. |
| PRECHARGE\_FINISHED- | OUTPUT | PUSH PULL | TTL | **Low** means precharge finished. |
| READY\_TO\_GO | OUTPUT | PUSH PULL | TTL | High means car is able to move. |
| SHUTDOWN\_TTL\_OK | INPUT | - | TTL | SHUTDOWN\_OK- signal's inverse, but at TTL level. |

