# Deadtime signal, cf. https://www.plctalk.net/qanda/showthread.php?t=123595

    Summary
    1) Detect a rising edge on a discrete input
    2) Twenty seconds after that detection, make an output go high for one second.

    Caveats
    - The twenty-second delay will be an HMI setpoint; here it is controlled by two inputs, decreasing or increasing 5s at a time
    - (In-;)Accuracy
      - Probably +/- one, or a few, scan times
      - Plus any non-deterministic effects of using a TON timer
      - Plus truncation of the time of the the input rising edge detection to the most recent whole second of the TON timer

    Table of Contents
    - Rung 0000-0000   Initialization
    - Rung 0001-0001   1Hz timer (TON; T4:0) repeating every 128s
    - Rung 0002-0004   Set or clear the delayed 1s output pulse; increment the next pulse pointer
    - Rung 0005-0005   Detect an input rising edge
    - Rung 0006-0008   Validate a proposed new output pulse
    - Rung 0009-0009   Add a new validated future output pulse
    - Rung 0010-0011   Decrease or increase delay

    Inputs
    - INPUT_TO_BE_DELAYED:  discrete input, the rising edge of which will produce a delayed, one-second output pulse
    - DECREASE_DELAY:  discrete input, the rising edge of which will decrease the delay by 5s
    - INCREASE_DELAY:  discrete input, the rising edge of which will increase the delay by 5s

    Outputs
    - OUTPUT_PULSE:  discrete output, one-second pulse delayed approximately DELAY_S from rising edge of INPUT_TO_BE_DELAYED

    Parameters and internal variables
    - PULSE_FIFO:  256-word circular buffer holding times to turn on output
    - 1HZ_REPEATING_TIMER:  TON 1Hz time base; resets every 128s (preset); preset must not exceed size (256) of PULSE_FIFO.
    - T_NOW:  1HZ_REPEATING_TIMER.ACCumulated time
    - DELAY_S:  approximate delay, seconds; delay must not exceed preset of 1HZ_REPEATING_TIMER
    - PTR_NEXT:  indirect addressing pointer into circular buffer PULSE_FIFO of time of next output pulse
    - PTR_FUTURE:  indirect addressing pointer into circular buffer PULSE_FIFO of time of the next future output pulse to be added
    - LAST_FUTURE_TIME:  the last valid time of a future output pulse PULSE_FIFO[(PTR_FUTURE+255) MODULO 256], or -1 if no such valid time exists
    - NEW_FUTURE_TIME:  the time of a proposed new future output pulse
    - NEW_FUTURE_PTR:  the value PTR_FUTURE will become if a proposed new future output pulse is valid

    First-pass initialization
    - Reset the 1Hz repeating timer
    - Set delay to 20s
    - Set PTR_NEXT and PTR_FUTURE pointers to zero i.e. to start of circular buffer PULSE_FIFO
    - Set LAST_FUTURE_TIME to -1

