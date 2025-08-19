# PIC16F877A Digital Real-Time Clock with DS1307 RTC and HD44780 LCD

## Introduction
This project is a digital real-time clock built using the PIC16F877A microcontroller, DS1307 real-time clock (RTC) IC, and an HD44780 16x2 LCD display. The clock displays current time (hours, minutes, seconds) and date (day, month, year) with high accuracy by leveraging the DS1307’s battery-backed I2C communication. This is a complete embedded system demonstrating microcontroller peripheral interfacing and real-time data display.

---

## Tools Used
- **[MPLAB X IDE](https://www.microchip.com/en-us/development-tools-tools-and-software/mplab-x-ide)** – For firmware development and debugging
- **[Microchip XC8 Compiler](https://www.microchip.com/en-us/development-tools-tools-and-software/mplab-xc-compilers)** – C compiler for PIC microcontrollers
- **[simulIDE](https://simulide.com/p/)** – Schematic capture and simulation software (optional)
- **[Kicad](https://www.kicad.org/)** – Schematic and PCB design. 
- **[Onshape](https://www.onshape.com/en/)** – For CAD encloser Design.

---

## Components Used
| Component               | Description                        | Quantity |
|-------------------------|----------------------------------|----------|
| PIC16F877A              | 8-bit microcontroller             | 1        |
| DS1307 RTC Module       | Real-time clock IC with battery  | 1        |
| HD44780 16x2 LCD        | Character LCD display module     | 1        |
| 20 MHz Crystal          | Oscillator for PIC MCU           | 1        |
| 22pF Capacitors         | For crystal stabilization        | 2        |
| 4.7kΩ Resistors         | Pull-ups for I2C communication   | 2        |
| 10kΩ Potentiometer      | LCD contrast adjustment          | 1        |
| Current limiting resistor (220Ω) | For LCD backlight       | 1        |


---

## Project Explanation

### Communication Protocols
This project primarily uses **I2C (Inter-Integrated Circuit)** communication protocol to interface the PIC16F877A with the DS1307 RTC module. I2C is a two-wire, half-duplex serial protocol comprising:
- **SDA (Serial Data Line)** – Bidirectional data line for communication
- **SCL (Serial Clock Line)** – Clock line controlled by the master (PIC MCU)

The PIC acts as an I2C master, initiating read/write operations with the DS1307 slave device. Proper pull-up resistors on SDA and SCL lines ensure reliable signal levels.

---

### DS1307 RTC IC [datasheet](https://www.alldatasheet.com/datasheet-pdf/view/123888/DALLAS/DS1307.html).
The **DS1307** is a low-power clock/calendar IC with a built-in 32.768 kHz crystal oscillator. It provides:
- Real-time tracking of seconds, minutes, hours, day, date, month, and year with leap-year compensation
- I2C-compatible serial interface for communication
- Battery backup to maintain time during power outages
- Square wave output (not used in this project)

This IC enables precise 1-second timekeeping and stores the current date and time in Binary-Coded Decimal (BCD) format, which is read and decoded by the PIC MCU.

---

### PIC16F877A Microcontroller [datasheet](https://www.alldatasheet.com/datasheet-pdf/view/82338/MICROCHIP/PIC16F877A.html).
The **PIC16F877A** is an 8-bit microcontroller by Microchip featuring:
- 40 pins with multiple I/O ports
- MSSP module supporting I2C hardware communication on RC3 (SCL) and RC4 (SDA)
- Configurable timers and interrupts for precise timing
- Runs at 20 MHz crystal oscillator frequency in this project

The MCU handles reading RTC data via I2C, converting BCD values, and controlling the LCD display.

---

### HD44780 LCD Display  [datasheet](https://cdn.sparkfun.com/assets/9/5/f/7/b/HD44780.pdf). 
A standard **16x2 alphanumeric LCD** based on the HD44780 controller is used for displaying:
- Current time in `HH:MM:SS` format
- Current date in `DD/MM/YY` format

The LCD operates in 8-bit mode, with three control pins connected to PORTB (RS, RW, EN) and data pins connected to PORTD. Contrast and backlight are adjusted with an external potentiometer and resistor respectively.

---

## Screenshots  
*(Add images/screenshots below to illustrate key stages)*

- **Circuit Schematic**  
![Schematic](path_to_schematic_image.jpg)

- **LCD Display Output Showing Time and Date**  
![LCD Output](path_to_lcd_output_image.jpg)

- **Code Snippets**  
