#ifndef CLOCK_H
#define CLOCK_H


// Define Digits
#define DIG_1   1 // connected to PORTC0
#define DIG_2   2 // connected to PORTC1
#define DIG_3   4 // connected to PORTC2
#define DIG_4   8 // connected to PORTC3
#define DIG_5   16 // connected to PORTC4
#define DIG_6   32 // connected to PORTC5
#define DIG_ALL 0x3F

// define segments
#define SEG_A   1 // connected to PORTD0
#define SEG_B   2 // connected to PORTD1
#define SEG_C   4 // connected to PORTD2
#define SEG_D   8 // connected to PORTD3
#define SEG_E   16 // connected to PORTD4
#define SEG_F   32 // connected to PORTD5
#define SEG_G   64 // connected to PORTD6
#define SEG_ALL 0x7F

// define modes
#define DISPLAY_TIME_MODE       0
#define SET_TIME_MODE           1
#define SET_ALARM_MODE          2
#define SET_AMPM_MODE           3
#define SET_MILITARY_MODE       4
#define DISPLAY_ALARM_MODE      5
#define SNOOZE_MODE             6

// Ease of use macros
//   Numbers
#define NUMBER_ZERO SEG_ALL-SEG_G
#define NUMBER_ONE SEG_B+SEG_C
#define NUMBER_TWO SEG_ALL-SEG_F-SEG_C
#define NUMBER_THREE SEG_ALL-SEG_F-SEG_E
#define NUMBER_FOUR SEG_F+SEG_G+SEG_B+SEG_C
#define NUMBER_FIVE SEG_ALL-SEG_B-SEG_E
#define NUMBER_SIX SEG_ALL-SEG_B
#define NUMBER_SEVEN SEG_A+SEG_B+SEG_C
#define NUMBER_EIGHT SEG_ALL
#define NUMBER_NINE SEG_ALL-SEG_E
#define NUMBER_BLANK SEG_ALL-SEG_ALL
#define LETTER_A SEG_ALL-SEG_D
#define LETTER_P SEG_ALL-SEG_C-SEG_D
#define LETTER_H SEG_ALL-SEG_A-SEG_D

//  Digits
#define DIGIT_ONE DIG_ALL-DIG_1
#define DIGIT_TWO DIG_ALL-DIG_2
#define DIGIT_THREE DIG_ALL-DIG_3
#define DIGIT_FOUR DIG_ALL-DIG_4
#define DIGIT_FIVE DIG_ALL-DIG_5
#define DIGIT_SIX DIG_ALL-DIG_6

// set up function pre-defines
void initialize_ports(void);  // set up port connects for AVR 
void display_time(uint8_t hrs, uint8_t mins, uint8_t secs);  // function to display the time across all digits
void display_number(uint8_t number, uint8_t digit); // function to display a single number on a single digit
void display_letter(uint8_t number, uint8_t digit); // function to display a single number on a single digit
void display_set_time(uint8_t hrs, uint8_t mins, uint8_t secs); //display the alarm setting feature
void display_nothing(void); // pretty self explainatory
void flash_number(uint8_t number, uint8_t digit); // funciton for flashing a number every second
void display_alarm_sequence(void); // what to do when the alarm is tripped
void display_set_military(void); // display setting military mode
uint8_t normalize_hours(uint8_t); // will fix the hours based upon 12/24 hour clock
void debounce_buttons(void); // function to check for buttons being pressed and debounce them


// set up global vars
static uint16_t confidence_both_pressed, confidence_not_both_pressed, set_ctr;
static uint16_t confidence_pin_b1_pressed, confidence_not_pin_b1_pressed;
static uint16_t confidence_pin_b2_pressed, confidence_not_pin_b2_pressed;
static uint16_t confidence_pin_b3_pressed, confidence_not_pin_b3_pressed;
static uint8_t seconds, minutes, hours, display_ctr, set_position, set_seconds, set_minutes, set_hours, incrementer;
static uint8_t alarm_hours, alarm_minutes, alarm_seconds;
static uint8_t mode, is_pm;
static uint8_t both_pressed, not_both_pressed, pin_b1_pressed, not_pin_b1_pressed, pin_b2_pressed, not_pin_b2_pressed;
static uint8_t pin_b3_pressed, not_pin_b3_pressed;
static uint8_t military_mode, display_set_hours, display_hours;  // true = 24 hour false

#endif
