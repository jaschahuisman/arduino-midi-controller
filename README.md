# Arduino midi controller
## Table of contents
1. [About this project](#about-this-project)
2. [Hardware](#hardware)
    + [Shopping list](#shopping-list)
3. [Circuit](#circuit)
    + [Midi connector circuit](#midi-connector-circuit)
    + [Button matrix circuit](#button-matrix-circuit)
4. [Code](#code)
    + [Midi connector code](#midi-connector-code)
    + [Button matrix code](#button-matrix-code)
5. [Midi communication](#midi-communication)
    + [Hexadecimal en/decoding](#hexadecimal-en/decoding)
    + [Midi send commands](#midi-send-commands)
    + [Status byte](#status-byte)
    + [Data byte](#data-byte)
        + [List of midi keys](#list-of-midi-keys)
6. [Design](#design)
7. [File downloads](#file-downloads)

## About this project

*[▲ Back to top ▲](#arduino-midi-controller)*

## Hardware
### Shopping list
To create an Arduino midi controller, you'll need the following:
* [1x Arduino/Genuino Uno](https://store.arduino.cc/arduino-uno-rev3) 
* [1x Midi connector female](https://www.sparkfun.com/products/9536)
* [1x Midi cable (and instrument/interface to plugin the cable)](https://www.thomann.de/intl/the_sssnake_sk366-3-blk_midi_kabel.htm)
* [2x 220 Ohm resistors](https://www.amazon.com/slp/220-ohm-resistor/pwc2jfx3cwoh9sf)
* [1x 9V Adapter/Battery (or just use USB for power)](https://www.sparkfun.com/products/15314)
* [Diodes](https://www.sparkfun.com/products/14884)
* [Pushbuttons](https://www.sparkfun.com/products/97)
* [Jumperwires](https://www.sparkfun.com/products/11026)
* [Finishing material](https://www.makercase.com/#/)

*[▲ Back to top ▲](#arduino-midi-controller)*

## Circuit
### Midi connector circuit
The female midi connector has 5 pins. We only use pin 4, 2 and 5. (Note that the pins are in the order of 1, 4, 2, 5, 3).
* Connect pin 4 to a 220 Ohm resistor, to Arduino's 5V pin.
* Connect pin 2 to ground.
* Connect pin 5 to a 220 Ohm resistor, to Arduino's pin 1 (TXD).

<img src="" alt="Circuit – 2x2 button matrix" width="500"/>

### Button matrix circuit
The button matrix seems a bit more complicated. Never connect anything to pin 13, because it's pullup input does not work well. The rows are being used as outputs, the columns as pullup inputs. Make sure to ground the columns with a diode, so that they don't interconnect with each other. This will prevent errors.

The formula for the amount of on-board pins you need is: `(cols + rows)`. The formula for the amount of buttons you get is: `(cols * rows)`.

The following image shows a simple 2x2 matrix circuit diagram using on-board pins 2-5:

<img src="" alt="Circuit – 2x2 button matrix" width="500"/>

*[▲ Back to top ▲](#arduino-midi-controller)*

## Code
### Midi connector code
Use the simple midi send script below to send midi commands to the midi output. Note that no midi pin setup is necessary because all `Serial.write()' commands will output to pin 1 (TXD). To learn more about midi communication check:
5. [Midi communication](#midi-communication)
  + [Hexadecimal en/decoding](#hexadecimal-en/decoding)
  + [Midi send commands](#midi-send-commands)
  + [Status byte](#status-byte)
  + [Data byte](#data-byte)
    + [List of midi keys](#list-of-midi-keys)

``` c#
// SIMPLE MIDI SEND SCRIPT
// Jascha Huisman, The Netherlands, Dordrecht & Breda

void setup(){
    Serial.begin(31250);          // Midi needs a 31250 baud-rate
}

void midi(int statusByte, int dataByte1, int dataByte2){
    Serial.write(statusByte);     // Send the midi command (status byte)
    Serial.write(dataByte1);      // Send the first parameter (data byte)
    Serial.write(dataByte2);      // Send the second parameter (data byte)
}

void update(){
    midi(0x90, 0x3C, 0x50);
    // Status byte: 0x90 – note on on channel 1
    // Data byte 1: 0x3C – note C4
    // Data byte 2: 0x50 – velocity of 80

    delay(1000);                  // Wait one second
}
```

*[▲ Back to top ▲](#arduino-midi-controller)*

### Button matrix code
``` c#
// SIMPLE 2X2 BUTTON MATRIX SCRIPT
// Jascha Huisman, The Netherlands, Dordrecht & Breda

// Initialize button matrix parameters
byte rows = {2, 3};                 // Pins for row inputs {r1, r2, r3, etc}
byte cols = {4, 5};                 // Pins for column inputs {c1, c2, c3, etc}
const int rowCount = sizeof(rows)/sizeof(rows[0]);                // Amount of rows
const int colCount = sizeof(cols)/sizeof(cols[0]);                // Amount of columns
const int buttonCount = rowCount * colCount;                      // Amount of buttons
int button[buttonCount];                                          // Button array
byte keys[rowCount][colCount];      // Coördinate based dataslot (keys[0][0] will be the first button etc.)

void setup(){
    // Set the pinModes for the rows as inputs
    for(int x=0; x<rowCount; x++) {
        pinMode(rows[x], INPUT);
    }

    // Set the pinModes for the columns as pullup inputs
    for (int x=0; x<colCount; x++) {
        pinMode(cols[x], INPUT_PULLUP);
    }
    
    Serial.begin(31250);
}

void update(){
    // Iterate trough columns
    for (int colIndex=0; colIndex < colCount; colIndex++) {
        byte curCol = cols[colIndex];         // Save current column temporarily in a byte
        pinMode(curCol, OUTPUT);              // Set current column's pin as a temporarily output
        digitalWrite(curCol, LOW);            // Set temporarily output to low

        // Iterate trough rows in column
        for (int rowIndex=0; rowIndex < rowCount; rowIndex++) {
            byte rowCol = rows[rowIndex];           // Save current row related to temporarily in a byte
            pinMode(rowCol, INPUT_PULLUP);          // Set the current row as pullup input to save the corresponding pressed button value
            keys[rowIndex][colIndex] = digitalRead(rowCol);     // Save the button state in the ROW/COL key byte
        }

        pinMode(curCol, INPUT);               // Reset the current column from output to input
    }

    // Iterate trough buttons
    for (int buttonIndex = 0; buttonIndex < buttonCount; buttonIndex++) {
        button[buttonIndex] = keys[(int)(buttonIndex / colCount)][buttonIndex - ((int)(buttonIndex / colCount) * colCount)];
        if (button[buttonIndex] == 0){
            // Execute code to the corresponding button below
        }
    } 
  }
}
```
*[▲ Back to top ▲](#arduino-midi-controller)*

## Midi communication
Midi is a digital musical communication protocol (Musical Instrument Digital Interface) used to exchange data between multiple electronical instruments, such as keyboards and synthesizers. *Note: midi does not transmit sound, but operations only.* Although the midi commands in code language seem somewhat complicated, they can look like this textually:

* `Put Note C4 on with a velocity of 75 on channel 1`
* `Select Patch (instrument) 4 on channel 16`
* `Add a slight vibrato on channel 4`

Midi is transmitted trough a – you guessed it – midi cable. The cable connects from midi device A to B from out to in, or from thru to out (which gets out of midi device B immideately). More about midi signal flow can be found here. A midi cable has 5 connectors, and often is male on both sides. It looks like this:

![alt text](https://images-na.ssl-images-amazon.com/images/G/01/musical-instruments/detail-page/B000068NTU_img1.jpg "Midi Cable Male")

A midi controller sends 3 commands, notated in bytes: midi action (status byte), parameter 1 (data byte), parameter 2 (data byte). These bytes can be written down in binary, decimal, or hexadecimal.

### Hexadecimal en/decoding
In this project we're working with hexadecimal numbers, because they are easier to read than binary. Although decimal numbers are quite readable, I use hexadecimal values because it makes the command and midi channel of the [status byte](#status-byte) visible immediately. In the table below you can see each number from 1 to 16 is represented by its hexadecimal number from 0 to F. For decimal numbers above 15 the first digit of the hexadecimal number changes to the next decimal number, so for example: 17 (dec) is 11 in hexadecimal numbers. After 0F (15) comes 10 (16) and after 10 comes 11 (17), etc. If this seems complicated you can always use [a converter tool](https://www.rapidtables.com/convert/number/hex-to-decimal.html).

| Number             	| 0 	| 1 	| 2 	| 3 	| 4 	| 5 	| 6 	| 7 	| 8 	| 9 	| 10 	| 11 	| 12 	| 13 	| 14 	| 15 	|
|--------------------	|---	|---	|---	|---	|---	|---	|---	|---	|---	|----	|----	|----	|----	|----	|----	|----	|
| Hexadecimal Number 	| 00 	| 01 	| 02 	| 03 	| 04 	| 05 	| 06 	| 07 	| 08 	| 09  	| 0A  	| 0B  	| 0C  	| 0D  	| 0E  	| 0F  	|

*[▲ Back to top ▲](#arduino-midi-controller)*

### Midi send commands
A list of all possible midi commands with corresponding data byte types.
| **Midi Action**      | **[Status Byte](#status-byte)** | **[Data Byte 1](#data-bytes)**  | **[Data Byte 2](#data-bytes)**                    |
|----------------------|-----------------|-------------------------------------|------------------------------------|
| Note Off             | 0x80            | Key                                 | Note Velocity                      |
| Note On              | 0x90            | Key                                 | Note Velocity                      |
| Aftertouch           | 0xA0            | Key                                 | Pressure Value                     |
| Control Change       | 0xB0            | Control                             | Control Value                      |
| Program/patch Change | 0xC0            | Program/patch                       | –                                  |
| Channel Pressure     | 0xD0            | Aftertouch Pressure                 | –                                  |
| Pitch Bend           | 0xE0            | Bend Value (Least Significant Byte) | Bend Value (Most Significant Byte) |
| Non-musical Commands | 0xF0            | –                                   | –                                  |

A little example for a complete send action (command):
* Command: `0x92 0x5A 0x66`
* `0x92` represents the midi action (status byte) and can be translated to: _Note On on Channel 3_.
* `0x5A` represents the midi key (data byte 1) and can be translated to: _F# 6_.
* `0x66` represents the midi velocity (data byte 2) and can be translated to: _102 (forte)_.

*[▲ Back to top ▲](#arduino-midi-controller)*

### Status byte
To send a midi command it needs to be defined what action to execute. The status byte is the first thing being read by a midi receiver, so it's the first thing being sent from the Arduino board. The status byte contains the command a midi device will execute. The status byte hex code is made up of two characters: the action character and the channel character. In decimal numbers the status byte is a number from 128 to 255. Thats 80 to FF in hexadecimal numbers.

*For example: in the case of sending a note on midi channel 1, the action character is 9, and the channel character is 0. 
The final HEX code in the status byte will look like this: `0x90`*
| Midi Action | Channel (number) | HEX | Midi Action (Character) | Channel (Character) | Complete HEX Byte |
|-------------|------------------|-----|-------------------------|---------------------|-------------------|
| Note On     | 1                | 0x  | 9                       | 0                   | 0x90              |
| Note Off    | 3                | 0x  | 8                       | 2                   | 0x82              |

*[▲ Back to top ▲](#arduino-midi-controller)*

### Data byte
Data bytes are being used to send data of parameters from the status byte. A maximum of 2 data bytes can be sent with a status byte. A data byte in decimal numbers can be in a range from 0 to 127. Each status byte has corresponding data byte types, that can be found [in this table of midi send actions](#midi-send-actions). In the table below you can find some corresponding units of the parameters.  

| Data Byte Parameter | Unit              | Unit Range                          |
|---------------------|-------------------|-------------------------------------|
| Key                 | Note              | C-1 – G9                            |
| Velocity            | Velocity/Dynamics | 0 – 127 / Silent – Forte Fortissimo |

#### List of midi keys
| DEC | HEX | NOTE |  | DEC | HEX | NOTE |  | DEC | HEX | NOTE |  | DEC | HEX | NOTE |  | DEC | HEX | NOTE |  | DEC | HEX | NOTE |
|-----|-----|------|---|-----|-----|------|---|-----|-----|------|---|-----|-----|------|---|-----|-----|------|---|-----|-----|------|
| 0 | 00 | C-1 |  | 21 | 15 | A0 |  | 42 | 2A | F#2 |  | 63 | 3F | D#4 |  | 84 | 54 | C6 |  | 105 | 69 | A7 |
| 1 | 01 | C#-1 |  | 22 | 16 | A#0 |  | 43 | 2B | G2 |  | 64 | 40 | E4 |  | 85 | 55 | C#6 |  | 106 | 6A | A#7 |
| 2 | 02 | D-1 |  | 23 | 17 | B0 |  | 44 | 2C | G#2 |  | 65 | 41 | F4 |  | 86 | 56 | D6 |  | 107 | 6B | B7 |
| 3 | 03 | D#-1 |  | 24 | 18 | C1 |  | 45 | 2D | A2 |  | 66 | 42 | F#4 |  | 87 | 57 | D#6 |  | 108 | 6C | C8 |
| 4 | 04 | E-1 |  | 25 | 19 | C#1 |  | 46 | 2E | A#2 |  | 67 | 43 | G4 |  | 88 | 58 | E6 |  | 109 | 6D | C#8 |
| 5 | 05 | F-1 |  | 26 | 1A | D1 |  | 47 | 2F | B2 |  | 68 | 44 | G#4 |  | 89 | 59 | F6 |  | 110 | 6E | D8 |
| 6 | 06 | F#-1 |  | 27 | 1B | D#1 |  | 48 | 30 | C3 |  | 69 | 45 | A4 |  | 90 | 5A | F#6 |  | 111 | 6F | D#8 |
| 7 | 07 | G-1 |  | 28 | 1C | E1 |  | 49 | 31 | C#3 |  | 70 | 46 | A#4 |  | 91 | 5B | G6 |  | 112 | 70 | E8 |
| 8 | 08 | G#-1 |  | 29 | 1D | F1 |  | 50 | 32 | D3 |  | 71 | 47 | B4 |  | 92 | 5C | G#6 |  | 113 | 71 | F8 |
| 9 | 09 | A-1 |  | 30 | 1E | F#1 |  | 51 | 33 | D#3 |  | 72 | 48 | C5 |  | 93 | 5D | A6 |  | 114 | 72 | F#8 |
| 10 | 0A | A#-1 |  | 31 | 1F | G1 |  | 52 | 34 | E3 |  | 73 | 49 | C#5 |  | 94 | 5E | A#6 |  | 115 | 73 | G8 |
| 11 | 0B | B-1 |  | 32 | 20 | G#1 |  | 53 | 35 | F3 |  | 74 | 4A | D5 |  | 95 | 5F | B7 |  | 116 | 74 | G#8 |
| 12 | 0C | C0 |  | 33 | 21 | A1 |  | 54 | 36 | F#3 |  | 75 | 4B | D#5 |  | 96 | 60 | C7 |  | 117 | 75 | A8 |
| 13 | 0D | C#0 |  | 34 | 22 | A#1 |  | 55 | 37 | G3 |  | 76 | 4C | E5 |  | 97 | 61 | C#7 |  | 118 | 76 | A#8 |
| 14 | 0E | D0 |  | 35 | 23 | B1 |  | 56 | 38 | G#3 |  | 77 | 4D | F5 |  | 98 | 62 | D7 |  | 119 | 77 | B8 |
| 15 | 0F | D#0 |  | 36 | 24 | C2 |  | 57 | 39 | A3 |  | 78 | 4E | F#5 |  | 99 | 63 | D#7 |  | 120 | 78 | C9 |
| 16 | 10 | E0 |  | 37 | 25 | C#2 |  | 58 | 3A | A#3 |  | 79 | 4F | G5 |  | 100 | 64 | E7 |  | 121 | 79 | C#9 |
| 17 | 11 | F0 |  | 38 | 26 | D2 |  | 59 | 3B | B3 |  | 80 | 50 | G#5 |  | 101 | 65 | F7 |  | 122 | 7A | D9 |
| 18 | 12 | F#0 |  | 39 | 27 | D#2 |  | 60 | 3C | C4 |  | 81 | 51 | A5 |  | 102 | 66 | F#7 |  | 123 | 7B | D#9 |
| 19 | 13 | G0 |  | 40 | 28 | E2 |  | 61 | 3D | C#4 |  | 82 | 52 | A#5 |  | 103 | 67 | G7 |  | 124 | 7C | E9 |
| 20 | 14 | G#0 |  | 41 | 29 | F2 |  | 62 | 3E | D4 |  | 83 | 53 | B5 |  | 104 | 68 | G#7 |  | 125 | 7D | F9 |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | 126 | 7E | F#9 |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | 127 | 7F | G9 |

*[▲ Back to top ▲](#arduino-midi-controller)*

## Design
Will be added soon...

## File downloads
Will be added soon...
