# PIC16F877A Digital Real-Time Clock with DS1307 RTC and HD44780 LCD

## Introduction
This project is a digital real-time clock built using the PIC16F877A microcontroller, DS1307 real-time clock (RTC) IC, and an HD44780 16x2 LCD display. The clock displays current time (hours, minutes, seconds) and date (day, month, year) with high accuracy by leveraging the DS1307’s battery-backed I2C communication. This is a complete embedded system demonstrating microcontroller peripheral interfacing and real-time data display.

---

## Tools Used
- [**MPLAB X IDE**](https://www.microchip.com/en-us/development-tools-tools-and-software/mplab-x-ide) – For firmware development and debugging
- [**Microchip XC8 Compiler**](https://www.microchip.com/en-us/development-tools-tools-and-software/mplab-xc-compilers) – C compiler for PIC microcontrollers
- [**simulIDE**](https://simulide.com/p/) – Schematic capture and simulation software (optional)
- [**Kicad**](https://www.kicad.org/) – Schematic and PCB design
- [**Onshape**](https://www.onshape.com/en/) – For CAD enclosure design

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

### Learn more: [I2C Protocol by foolish engineer](https://youtube.com/playlist?list=PLbGlpmZLQWJceYTFXwBjYnUNN2vyVKYNA&feature=shared)

---

### DS1307 RTC IC [datasheet](https://www.alldatasheet.com/datasheet-pdf/view/123888/DALLAS/DS1307.html)
The **DS1307** is a low-power clock/calendar IC with a built-in 32.768 kHz crystal oscillator. It provides:
- Real-time tracking of seconds, minutes, hours, day, date, month, and year with leap-year compensation
- I2C-compatible serial interface for communication
- Battery backup to maintain time during power outages
- Square wave output (not used in this project)

This IC enables precise 1-second timekeeping and stores the current date and time in Binary-Coded Decimal (BCD) format, which is read and decoded by the PIC MCU.

### Learn more: [DS1307 Datasheet](https://www.alldatasheet.com/datasheet-pdf/view/123888/DALLAS/DS1307.html)

---

### PIC16F877A Microcontroller [datasheet](https://www.alldatasheet.com/datasheet-pdf/view/82338/MICROCHIP/PIC16F877A.html)
The **PIC16F877A** is an 8-bit microcontroller by Microchip featuring:
- 40 pins with multiple I/O ports
- MSSP module supporting I2C hardware communication on RC3 (SCL) and RC4 (SDA)
- Configurable timers and interrupts for precise timing
- Runs at 20 MHz crystal oscillator frequency in this project

The MCU handles reading RTC data via I2C, converting BCD values, and controlling the LCD display.

### Learn more: [SM training academy](https://youtube.com/playlist?list=PL_zvrXFdKgZpTrM99mypGVW5JBZ6tQZiR&feature=shared)

---

### HD44780 LCD Display [datasheet](https://cdn.sparkfun.com/assets/9/5/f/7/b/HD44780.pdf)
A standard **16x2 alphanumeric LCD** based on the HD44780 controller is used for displaying:
- Current time in `HH:MM:SS` format
- Current date in `DD/MM/YY` format

The LCD operates in 8-bit mode, with three control pins connected to PORTB (RS, RW, EN) and data pins connected to PORTD. Contrast and backlight are adjusted with an external potentiometer and resistor respectively.

---
## Firmware code
### Code is written in ** Embedded C language** on **MPLAB X IDE** . Generated _.hex_ file is simulated in ** SimulIDE ** 

```c
/******************************************************
 * PIC16F877A + DS1307 RTC + 16x2 LCD (4-bit interface)
 * Clock + Date display
 * Fosc = 20 MHz (HS), I2C @ ~100 kHz
 * Compiler: XC8
 *
 * Connections
 *   LCD:
 *     RS -> RB0
 *     RW -> RB1
 *     EN -> RB2
 *     D4..D7 -> RD4..RD7
 *
 *   RTC (DS1307):
 *     SDA -> RC4 (SDA)
 *     SCL -> RC3 (SCL)
 *     Vbat -> 3V coin cell
 *
 * Notes:
 *  - DS1307 I2C base address write = 0xD0, read = 0xD1
 *  - Time kept in BCD inside DS1307
 ******************************************************/
#include <xc.h>
#include <stdint.h>
#include <stdbool.h>
//================ Configuration bits ==================
#pragma config FOSC  = HS    // High Speed Oscillator
#pragma config WDTE  = OFF   // Watchdog Timer Disabled
#pragma config PWRTE = OFF   // Power-up Timer Disabled
#pragma config BOREN = OFF   // Brown-out Reset Disabled
#pragma config LVP   = ON    // Low-voltage Programming Enable
#pragma config CPD   = OFF   // Data EEPROM Code Protection off
#pragma config WRT   = OFF   // Flash Program Memory Write Enable off
#pragma config CP    = OFF   // Flash Program Code Protection off
#define _XTAL_FREQ 20000000UL
//================ LCD pin macros (PORTB & PORTD) ======
#define RS   PORTBbits.RB0
#define RW   PORTBbits.RB1
#define EN   PORTBbits.RB2
#define LCD_DATA_PORT   PORTD
#define LCD_DATA_TRIS   TRISD
//================ DS1307 register map =================
#define DS1307_ADDR   0xD0   // write address, read = 0xD1
#define REG_SECOND    0x00
#define REG_MINUTE    0x01
#define REG_HOUR      0x02
#define REG_DAY       0x03
#define REG_DATE      0x04
#define REG_MONTH     0x05
#define REG_YEAR      0x06
#define REG_CONTROL   0x07

//================ Function prototypes =================
void i2c_init(void);
void i2c_start(void);
void i2c_restart(void);
void i2c_stop(void);
bool i2c_write(uint8_t val);
uint8_t i2c_read(bool ack);

void DS1307_write(uint8_t sec, uint8_t min, uint8_t hr,
                  uint8_t day, uint8_t date, uint8_t month, uint8_t year);
void DS1307_read(uint8_t slave_address, uint8_t register_address);

void delay_us_custom(unsigned int Delay);
void lcd_init(void);
void lcd_cmd(uint8_t cmd);
void lcd_data(uint8_t data);
void lcd_goto(uint8_t row, uint8_t col);
void lcd_word(const char *s);
void bcd_to_ascii(uint8_t value);
uint8_t decimal_to_bcd(uint8_t value);

//================ Globals (latest read from RTC) ======
volatile uint8_t __sec, __min, __hr, __day, __date, __month, __yr, __con;

//==================== MAIN ============================
void main(void)
{
    // Set directions
    TRISC = 0xFF;      // RC3=SCL, RC4=SDA are controlled by MSSP; other C as input
    TRISD = 0x00;      // LCD data D4..D7 on RD4..RD7; we drive the whole port
    PORTD = 0x00;
    TRISB = 0x00;      // RB0..RB2 outputs for RS/RW/EN
    PORTB = 0x00;

    // I2C @ ~100 kHz (Fosc=20 MHz => SSPADD = (Fosc/(4*Fscl))-1 = 49)
    SSPADD = 49;
    SSPCON = 0x28;     // I2C Master mode, enable MSSP
    SSPSTAT = 0x00;

    lcd_init();

    // Labels
    lcd_goto(1,1);
    lcd_word("CLOCK:");
    lcd_goto(2,1);
    lcd_word("DATE:");

    // Set initial time: 12:15:00 (HH:MM:SS), Day=1 (Sunday), Date=19, Month=8, Year=25 (2025)
DS1307_write(0, 15, 12, 1, 19, 8, 25);
__delay_ms(300);


    while(1)
    {
        __delay_ms(50);
        DS1307_read(DS1307_ADDR, REG_SECOND); // fills globals

        // Show HH:MM:SS
        lcd_goto(1,8);
        bcd_to_ascii(__hr);
        lcd_data(':');
        bcd_to_ascii(__min);
        lcd_data(':');
        bcd_to_ascii(__sec);

        // Show DD/MM/YY
        lcd_goto(2,7);
        bcd_to_ascii(__date);
        lcd_data('/');
        bcd_to_ascii(__month);
        lcd_data('/');
        bcd_to_ascii(__yr);
    }
}

//================ Helpers =============================
void delay_us_custom(unsigned int Delay)
{
    while(Delay--) { __delay_us(1); }
}

//================ LCD (4-bit) =========================
static void lcd_pulse_enable(void)
{
    EN = 1;
    __delay_us(1);
    EN = 0;
    __delay_us(50);
}

static void lcd_send_nibble(uint8_t nib)
{
    // nibble in lower 4 bits -> map to RD4..RD7
    uint8_t d = LCD_DATA_PORT & 0x0F;
    d |= (nib << 4);
    LCD_DATA_PORT = d;
    lcd_pulse_enable();
}

static void lcd_send_byte(uint8_t val, bool is_data)
{
    RS = (is_data ? 1 : 0);
    RW = 0;

    lcd_send_nibble(val >> 4);
    lcd_send_nibble(val & 0x0F);
}

void lcd_cmd(uint8_t cmd)        { lcd_send_byte(cmd, false); }
void lcd_data(uint8_t data)      { lcd_send_byte(data, true);  }

void lcd_init(void)
{
    // init lines
    RS = 0; RW = 0; EN = 0;
    LCD_DATA_TRIS = 0x00;
    LCD_DATA_PORT = 0x00;
    __delay_ms(20);

    // 4-bit init sequence
    lcd_send_nibble(0x03);
    __delay_ms(5);
    lcd_send_nibble(0x03);
    __delay_us(150);
    lcd_send_nibble(0x03);
    lcd_send_nibble(0x02); // 4-bit

    lcd_cmd(0x28); // 4-bit, 2-line, 5x8 dots
    lcd_cmd(0x0C); // display on, cursor off
    lcd_cmd(0x06); // entry mode
    lcd_cmd(0x01); // clear
    __delay_ms(2);
}

void lcd_goto(uint8_t row, uint8_t col)
{
    uint8_t addr = (row==1 ? 0x00 : 0x40) + (col-1);
    lcd_cmd(0x80 | addr);
}

void lcd_word(const char *s)
{
    while(*s) lcd_data(*s++);
}

void bcd_to_ascii(uint8_t value)
{
    // value is BCD e.g., 0x45 -> '4','5'
    uint8_t b = value;
    uint8_t tens = (b & 0xF0) >> 4;
    uint8_t ones = (b & 0x0F);

    lcd_data(tens + '0');
    lcd_data(ones + '0');
}

uint8_t decimal_to_bcd(uint8_t value)
{
    uint8_t msb = value / 10;
    uint8_t lsb = value % 10;
    return (uint8_t)((msb << 4) | lsb);
}

//================ I2C low-level =======================
void i2c_init(void)
{
    SSPADD = 49;     // ~100 kHz at 20MHz
    SSPCON = 0x28;   // Master mode, enable
    SSPSTAT = 0x00;
}

void i2c_start(void)
{
    SSPCON2bits.SEN = 1;
    while(SSPCON2bits.SEN);
    PIR1bits.SSPIF = 0;
}

void i2c_restart(void)
{
    SSPCON2bits.RSEN = 1;
    while(SSPCON2bits.RSEN);
    PIR1bits.SSPIF = 0;
}

void i2c_stop(void)
{
    SSPCON2bits.PEN = 1;
    while(SSPCON2bits.PEN);
}

bool i2c_write(uint8_t val)
{
    SSPBUF = val;
    while(!PIR1bits.SSPIF);
    PIR1bits.SSPIF = 0;
    if(SSPCON2bits.ACKSTAT)
    {
        i2c_stop();
        return false;
    }
    return true;
}

uint8_t i2c_read(bool ack)
{
    uint8_t val;
    SSPCON2bits.RCEN = 1;
    while(!SSPSTATbits.BF);
    val = SSPBUF;
    SSPCON2bits.ACKDT = (ack ? 0 : 1); // 0=ACK, 1=NACK
    SSPCON2bits.ACKEN = 1;
    while(SSPCON2bits.ACKEN);
    return val;
}

//================ DS1307 routines =====================
void DS1307_write(uint8_t _second, uint8_t _minute, uint8_t _hour,
                  uint8_t _day, uint8_t _date, uint8_t _month, uint8_t _year)
{
    // Start + slave addr (write)
    i2c_start();
    if(!i2c_write(DS1307_ADDR)) return;

    // Start at register 0x00
    if(!i2c_write(REG_SECOND)) return;

    // Write 7 bytes in BCD; ensure CH bit cleared on seconds
    i2c_write(decimal_to_bcd(_second) & 0x7F);
    i2c_write(decimal_to_bcd(_minute));
    i2c_write(decimal_to_bcd(_hour));   // 24-hr assumed
    i2c_write(decimal_to_bcd(_day));
    i2c_write(decimal_to_bcd(_date));
    i2c_write(decimal_to_bcd(_month));
    i2c_write(decimal_to_bcd(_year));

    // Optionally control reg: SQW disabled
    i2c_write(0x00);

    // Stop
    i2c_stop();
}

void DS1307_read(uint8_t slave_address, uint8_t register_address)
{
    // Set pointer
    i2c_start();
    if(!i2c_write(slave_address)) return;
    if(!i2c_write(register_address)) return;

    // Restart and read 8 bytes (sec..control)
    i2c_restart();
    if(!i2c_write(slave_address + 1)) { i2c_stop(); return; }

    __sec   = i2c_read(true);
    __min   = i2c_read(true);
    __hr    = i2c_read(true);
    __day   = i2c_read(true);
    __date  = i2c_read(true);
    __month = i2c_read(true);
    __yr    = i2c_read(true);
    __con   = i2c_read(false); // NACK on last byte

    i2c_stop();
}

```
### Circuit Schematic  
![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/Schematic.png)  
*Figure 1: Circuit schematic showing PIC16F877A, DS1307 RTC, and LCD connections.*

### Circuit Layout Front
![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/layoutf.png)  
*Figure 2: Circuit layout showing PIC16F877A, DS1307 RTC, and LCD connections.*

### Circuit Layout Back  
![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/layoutb.png)  
*Figure 3: Circuit Layout showing PIC16F877A, DS1307 RTC, and LCD connections.*
### Circuit Simulation 
![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/simulation.png)  
*Figure 4: Circuit simulation on simulIDE showing PIC16F877A, DS1307 RTC, and LCD connections.*
### Circuit 3D Model  
![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/onehzchalenge.png)  
*Figure 5: 3D cad model*

![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/onehzchalenge1.png)  
*Figure 6: 3D cad model*

![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/onehzchalenge2.png)  
*Figure 7: 3D cad model*

![Circuit schematic](https://github.com/ShravanaHS/Digital-Clock-PIC16F877A-DS1307-16x2-LCD/raw/main/images/onehzchalenge3.png)  
*Figure 8: 3D cad model*


