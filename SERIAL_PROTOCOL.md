# Serial Protocol Documentation

## General

The serial communication is implemented by sending a command byte followed by optional parameters. A transmission is always ended by a Newline character (`\n`).

## Handshake

Before Raspberry Pi and Microcontroller start communicating, the HELO1 pin must be set to high by the Raspberry. The microcontroller responds by setting HELO2 high. Afterwards, a serial handshake is performed.

## Commands and Parameters
    
### General

        HELLO = b'\x00'
        ALREADY_CONNECTED = b'\x01'
        ERROR = b'\x02'
        RECEIVED = b'\x03'
        EOT = b'\n'
        
### Scrolls

Scroll numbers:

        HORIZONTAL = 0
        VERTICAL = 1

Set scroll positions with

        'M' + <scroll_number (8bit)> + <steps (16bit)> + <speed (8bit)>
        
`speed` is a value between 1 and 4.
        
This only sets the values. Execute with DO_IT (see below).

### Lights

Light numbers:

        BACKLIGHT = 0
        FRONTLIGHT = 1

Set light values with

        'L' + <light_number (8bit)> + <color (32bit)> + <fade_time_ms (32bit)>
        
        color = <white (8bit)> + <red (8bit)> + <green (8bit)> + <blue (8bit)>
        
This only sets the values. Execute with DO_IT (see below).

### Execute

        'D'

### User interaction

USER_INTERACT = U
    
        'U' + <button_bitmask (8bit)> + <timeout (32bit)>
        
Responses depend on button pressed:

        <RECEIVED> + <Blue 0x01 | Red 0x02 | Yellow = 0x04 | Green = 0x08 | Timeout = 0x00 (8bit)>

### Record
  
RECORD = C

        'C' + <timeout (32bit)>
        
Response after timeout or when red button was pressed:

        <RECEIVED>
    
### Rewind/Zero Scrolls

        'R'

### Debugging

Print scroll positions to serial:

        'S'

Print sensor readings to serial:
        
        'Z'



