#include <avr/io.h>
#include <avr/interrupt.h>
#include "clock.h"


// 
// set up interrupt vector to handle counting seconds 
// from interrupt.h
ISR(TIMER1_COMPA_vect) 
{
	seconds++;

    //
    // check am/pm and toggle on DP4 every second
    //
    if(is_pm && !military_mode) {
        PORTB &= ~(1 << PINB4);
    }
    else {
        PORTB |= (1 << PINB4);  
    }

    //
    // main time update loop
    //
	if(seconds == 60)
	{
		seconds = 0;
		minutes++;
		if(minutes == 60)
		{
			minutes = 0;
			hours++;
            if(hours > 23) {
                hours = 0;
            }

            // toggle am/pm
            is_pm = (hours > 11) ? 1 : 0;

            // set the hours to display
            display_hours = normalize_hours(hours);
		}
	}
} 


//
// interrupt 
// from interrupt.h
//
ISR(TIMER0_OVF_vect)
{
    // check button states
    debounce_buttons();

    // determine what to display based on mode
    switch (mode) {
        case DISPLAY_TIME_MODE:
            display_time(display_hours, minutes, seconds);
            break;
        case SET_TIME_MODE:
            display_set_time(set_hours, set_minutes, set_seconds);
            break;
        case SET_ALARM_MODE:
            display_set_time(set_hours, set_minutes, set_seconds);
            break;
        case SET_MILITARY_MODE:
            display_set_military();
            break;
        case DISPLAY_ALARM_MODE:
            display_time(alarm_hours, alarm_minutes, alarm_seconds);
            break;
        default:
            display_time(display_hours, minutes, seconds);
            break;
    }
}

//
// debounce handler for buttons
// comments on how things work in line
//
void debounce_buttons(void)
{
    //  
    // Use:  
    //  Debounce handler for pressing right button (PINB2)
    //
    //  Description:  
    //      Button 2 is used in 2 ways
    //      1) In set time or set alarm mode:
    //          It is used to incrememt whichever segment you are on
    //          Press and hold for faster scrolling
    //      2)In set military mode:
    //          It is used to toggle military time (24h clock) press and hold
    //
    if(bit_is_clear(PINB,2) && !(bit_is_clear(PINB,1))) {
        confidence_pin_b2_pressed++;
        confidence_not_pin_b2_pressed = 0;
        if(confidence_pin_b2_pressed >= 75 ) {

            // this is retarded i know, will fix later
            if(pin_b2_pressed == 0 || pin_b2_pressed == 1) {
                if(mode == SET_TIME_MODE || mode == SET_ALARM_MODE) {
                    if(set_position < 3) {
                        set_seconds++;
                        if(set_seconds > 59) {
                            set_seconds = 0;
                        }
                    }
                    if(set_position > 2 && set_position < 5) {
                        set_minutes++;
                        if(set_minutes > 59) {
                            set_minutes = 0;
                        }
                    }
                    if(set_position > 4) {
                        hours++;
                        if(hours > 23) {
                            hours = 0;
                        }

                        // toggle am/pm
                        is_pm = (hours > 11) ? 1 : 0;

                        set_hours = normalize_hours(hours);
                    }
                }

                // changing 12/24h clock and don't allow button to be held down
                else if (mode == SET_MILITARY_MODE && pin_b2_pressed == 0) {
                    military_mode = !military_mode;

                    // toggles am/pm
                    is_pm = (hours > 11) ? 1 : 0;
                }

                pin_b2_pressed = 1;
            }
            confidence_pin_b2_pressed = 0;
        }
    }
    else {
        confidence_not_pin_b2_pressed++;
        confidence_pin_b2_pressed = 0;
        if(confidence_not_pin_b2_pressed > 50) {
            pin_b2_pressed = 0;
            confidence_not_pin_b2_pressed = 0;
        }
    }


    // Use: debounce handler for pressing the alarm / snooze button (PINB0)
    //
    //  Descirtipn:
    //      shit on self
    //
    if(bit_is_clear(PINB,0)) {
        confidence_pin_b3_pressed++;
        confidence_not_pin_b3_pressed = 0;

        // want to display it every cycle its pressed
        if(confidence_pin_b3_pressed >= 1 ) {
            pin_b3_pressed = 1;
            mode = DISPLAY_ALARM_MODE;
        }
        confidence_pin_b3_pressed = 0;
    }
    else {
        confidence_not_pin_b3_pressed++;
        confidence_pin_b3_pressed = 0;
        if(confidence_not_pin_b3_pressed > 50) {
            pin_b3_pressed = 0;
            mode = DISPLAY_TIME_MODE;
            confidence_not_pin_b3_pressed = 0;
        }
    }


    //
    //  Use:
    //      Debounce handler for pressing left button (PINB1)
    //
    //  Description:
    //      Left button is used in the following way
    //      1) In set alarm or set time mode:
    //          Moves the segment you are setting one to the left
    //
    if(bit_is_clear(PINB,1) && !(bit_is_clear(PINB,2))) {
        confidence_pin_b1_pressed++;
        confidence_not_pin_b1_pressed = 0;
        if(mode == SET_ALARM_MODE || mode == SET_TIME_MODE) {
            if(confidence_pin_b1_pressed >= 50) {
                if(pin_b1_pressed == 0) {
                    if(set_position > 6) {
                        set_position = 1;
                    }
                    else {
                        set_position += 2;
                    }
                    pin_b1_pressed = 1;
                }
                confidence_pin_b1_pressed = 0;
            }
        }
        // entering set alarm mode
        else if(mode == DISPLAY_TIME_MODE) { 
            if(confidence_pin_b1_pressed >= 1000) {
                if(pin_b1_pressed == 0) {
                    if(mode == DISPLAY_TIME_MODE) {
                        set_seconds = alarm_seconds;
                        set_minutes = alarm_minutes;
                        set_hours = alarm_hours;
                        mode = SET_ALARM_MODE;
                        pin_b1_pressed = 1;
                    }
                }
                confidence_pin_b1_pressed = 0;
            }
        }
    }
    else {
        confidence_not_pin_b1_pressed++;
        confidence_pin_b1_pressed = 0;
        if(confidence_not_pin_b1_pressed > 50) {
            pin_b1_pressed = 0;
            confidence_not_pin_b1_pressed = 0;
        }
    }


    // 
    // Use:
    //      Debouncye handler for pressing both buttons (PINB1 & PINB2)
    //
    // Description:
    //      This is the action that takes you from mode to mode.
    //      If you're in normal mode (DISPLAY_TIME_MODE) pressing
    //      both buttons will take you into SET_MILITARY_MODE.  If
    //      you are in SET_MILITARY_MODE pressing both buttons will
    //      take you into SET_TIME_MODE.  If you are in SET_TIME_MODE
    //      pressing both buttons will take you back into normal mode
    //      (DISPLAY_TIME_MODE).
    if(bit_is_clear(PINB,1) && bit_is_clear(PINB,2)) {
        confidence_both_pressed++;
        confidence_not_both_pressed = 0;
        if(confidence_both_pressed >= 1000) {
            if(both_pressed == 0) {
                if(mode == DISPLAY_TIME_MODE){
                    mode = SET_MILITARY_MODE;
                }
                else if(mode == SET_MILITARY_MODE) {
                    mode = SET_TIME_MODE;
                    set_hours = normalize_hours(hours);
                    set_minutes = minutes;
                    set_seconds = seconds;
                }
                else if(mode == SET_TIME_MODE) {
                    display_hours = set_hours;
                    minutes = set_minutes;
                    seconds = set_seconds;
                    set_position = 1;
                    mode = DISPLAY_TIME_MODE;
                }
                else if(mode == SET_ALARM_MODE) {
                    alarm_seconds = set_seconds;
                    alarm_minutes = set_minutes;
                    alarm_hours = set_hours;
                    mode = DISPLAY_TIME_MODE;
                }
                    
                both_pressed = 1;
            }
            confidence_both_pressed = 0;
        }
     }
     else {
        confidence_not_both_pressed++;
        confidence_both_pressed = 0;
        if(confidence_not_both_pressed > 500) {
            both_pressed = 0;
            confidence_not_both_pressed = 0;
        }
     }

}

// function to handle altering the hours to the proper format
// can sent it either a 12h or 24h time just make sure am/pm flag is set correctly
// send 24h value while in 12h mode .. needs normalization
uint8_t normalize_hours(uint8_t unnormal_hours)
{

    uint8_t normal_hours;
    normal_hours = 0;

    if(military_mode) {
        normal_hours = unnormal_hours;
    }

    // in 12h mode
    else {
        if(unnormal_hours == 0) {
            normal_hours = 12;
        }
        else if(unnormal_hours > 12) {
            normal_hours = unnormal_hours - 12;
        }
        else {
            normal_hours = unnormal_hours;
        }
    }
    return normal_hours;
}

//
// funtion to change the clock from 12h mode to 24h mode
// 
void display_set_military(void)
{
    switch(++display_ctr) {
        case 1: display_nothing();break;
        case 2: display_nothing();break;
        case 3: display_nothing();break;
        case 4: 
            if(military_mode) {
                display_number(2,4);
            }
            else {
                display_number(1,4);
            }
                break;
        case 5: 
            if(military_mode) { 
                display_number(4,5);
            }
            else {
                display_number(2,5);
            }
            break;
        case 6: 
            display_number(LETTER_H, 6);
            display_ctr=0;
            break;
        default: break;
    }
}


// display funciton to know you tripped the alarm
void display_alarm_sequence(void)
{
	switch(++display_ctr)
	{
		case 1: 
            if(seconds % 2)
                display_number(8, 1);
            else
                display_nothing();
            break;
		case 2:	
            if(seconds % 2)
                display_number(8, 2);
            else
                display_nothing();
            break;
		case 3: 
            if(seconds % 2)
                display_number(8, 3);
            else
                display_nothing(); break;
		case 4: 
            if(seconds % 2)
                display_number(8, 4);
            else
                display_nothing(); break;
		case 5: 
            if(seconds % 2)
                display_number(8, 5);
            else
                display_nothing(); break;
		case 6: 
            if(seconds % 2)
                display_number(8, 6);
            else
                display_nothing(); display_ctr=0; break;
		default:	break;
	}
}

//
// set the time or the alarm
// this will light all segments except the segment you are actively setting
// the segment you are actively setting will be flashed on and off every 
// second unless you are holding down the button
//
void display_set_time(uint8_t hrs, uint8_t mins, uint8_t secs)
{

    switch(++set_ctr) {
        case 1:
            if(set_position < 3) {
                flash_number(secs % 10, 6);
            }
            else {
                display_number(secs % 10, 6);
            }
            break;
        case 2:
            if(set_position < 3) {
                flash_number(secs / 10, 5);
            }
            else {
                display_number(secs / 10, 5);
            }
            break;
        case 3:
            if(set_position > 2 && set_position < 5) {
               flash_number(mins % 10, 4);
            }
            else {
                display_number(mins % 10, 4);
            }
            break;
        case 4:
            if(set_position > 2 && set_position < 5 ) {
                flash_number(mins / 10, 3);
            }
            else {
                display_number(mins / 10, 3);
            }
            break;
        case 5:
            if(set_position > 4) {
                flash_number(hrs % 10, 2);
            }
            else {
                display_number(hrs % 10, 2);
            }
            break;
        case 6:
            if(set_position > 4) {
                flash_number(hrs / 10, 1);
            }
            else {
                display_number(hrs / 10, 1);
            }
            set_ctr = 0;
            break;
        default:
            break;
    }
    return;
}



// display all digits
void display_time(uint8_t hrs, uint8_t mins, uint8_t secs)
{
	switch(++display_ctr)
	{
		case 1: 
            display_number(hrs / 10, 1);
            break;
		case 2:	
            display_number(hrs % 10, 2);
            break;
		case 3: display_number(mins / 10, 3); break;
		case 4: display_number(mins % 10, 4); break;
		case 5: display_number(secs / 10, 5); break; 
		case 6: display_number(secs % 10, 6); display_ctr=0; break;
		default:	break;
	}
}

// used to save PWM shit
void display_nothing(void)
{
    PORTC = DIG_ALL;
    PORTD = SEG_ALL;
    return;
}

void flash_number(uint8_t number, uint8_t digit)
{
    if(((seconds % 2) == 0) || pin_b2_pressed) {
        display_number(number, digit);
    }
    else {
        display_nothing();
    }
}

// display single digit
void display_number(uint8_t number, uint8_t digit)
{
	switch(digit)
	{
		case 1: PORTC = DIGIT_ONE; break;   // select digit 1
		case 2: PORTC = DIGIT_TWO; break;	//Select Digit 2
		case 3: PORTC = DIGIT_THREE; break;	//Select Digit 3
		case 4: PORTC = DIGIT_FOUR; break;	//Select Digit 4
		case 5: PORTC = DIGIT_FIVE; break;  // select DIgit 5
		case 6: PORTC = DIGIT_SIX; break;	//Select Digit 6
	    default: PORTC = digit; break;
	}
	switch(number)
	{
		case 0: PORTD = NUMBER_ZERO; break;
		case 1: PORTD = NUMBER_ONE; break;
		case 2: PORTD = NUMBER_TWO; break;
		case 3: PORTD = NUMBER_THREE; break;
		case 4: PORTD = NUMBER_FOUR; break;
		case 5: PORTD = NUMBER_FIVE; break;
		case 6: PORTD = NUMBER_SIX; break;
		case 7: PORTD = NUMBER_SEVEN; break;
		case 8: PORTD = NUMBER_EIGHT; break;
		case 9: PORTD = NUMBER_NINE; break;
		default: PORTD = number; break;
	}	
}

void initialize_ports()
{
    // initialize ports
    DDRC = DIG_ALL; //PC0-PC5 as outputs
    DDRD = SEG_ALL; //PD0-PD6 as outputs
    DDRB = 0x38; //PB0-2 as output PB3-6 as inputs
                        
    //=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
    //              Initialize Display Timer0
    //=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
    TCCR0B |= (1 << CS01)+(1 << CS00); // Timer0 prescaler to clk/64 via CS01 + CS00 bits
    TIMSK0 |= (1 << TOIE0); // Enable Timer0 (8-bit) overflow interrupt

    //=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
    //              Initialize Time Timer1
    //=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
    TCCR1B |= (1 << CS12)+(1 << CS10); // Timer1 prescaler to clk/1024 via CS10 + CS12 bits
    TCCR1B |= (1 << WGM12); // configure for CTC mode
    TIMSK1 |= (1 << OCIE1A); // CTC interrupt
    OCR1A = 15625; // with prescaler 64, results in 1 Hz @ 1 MHz

    // set up PINB3 to high to shut it off (DP1)
    PORTB |= (1 << PINB3);

    // set up PINB3 to high to shut it off (DP2)
    PORTB |= (1 << PINB4);

    // set up PINB0 for button pressing
    PORTB |= 1 << PINB0; //Set PINB1 to a high reading

    // set up PINB1 for button pressing
    PORTB |= 1 << PINB1; //Set PINB1 to a high reading

    // set up PINB2 for button pressing
    PORTB |= 1 << PINB2; //Set PINB1 to a high reading

    return;
}



// mainloop
int main(void) {    

    initialize_ports();  // initialize port connections
    sei(); //  enable global interrupts

    // initialize timing digits
    display_set_hours = hours = set_hours = display_hours = 12;
    minutes = 00;
    seconds = 00;

    // set up position and mode
    set_position = 1;
    mode = DISPLAY_TIME_MODE;

    incrementer = display_ctr = set_ctr = 0; 
    both_pressed = not_both_pressed = confidence_both_pressed = confidence_not_both_pressed = 0;
    pin_b1_pressed = not_pin_b1_pressed = confidence_pin_b1_pressed = confidence_not_pin_b1_pressed = 0;
    pin_b2_pressed = not_pin_b2_pressed = confidence_pin_b2_pressed = confidence_not_pin_b2_pressed = 0;
    pin_b3_pressed = not_pin_b3_pressed = confidence_pin_b3_pressed = confidence_not_pin_b3_pressed = 0;

    // default to military mode
    is_pm = 00;
    military_mode = 1;


    while(1) {
    } // loop forever

    return 1;
}
