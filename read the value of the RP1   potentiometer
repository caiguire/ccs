/**********************************************************************
 * Modificacion de prueba
 **********************************************************************
 * PICkit 2 C Compiler Demo for 44-Pin Demo Board.
 * 
 * This code uses the PIC16F887 ADC to read the value of the RP1
 * potentiometer, and displays a "bar graph" on the LEDs of the 
 * relative potentiometer value.
 * Pressing the switch SW1 reverses the bar graph.  See the 
 * "Quick Start" > "Debug Express" > "Getting Started in C" section
 * of the PICkit CD.
 **********************************************************************/

#include <16F877.h>             // header file for the PIC16F887
                                // includes built-in functions and constants
                                // for arguments/returns.

#use fast_io(A)                 // all other ports default to 'standard_io'
#use fast_io(B)                 // fast_io means the tris bits only change
                                // when we explicitly set them.

// FUSES sets the PIC16F877 Configuration Words.  See top of the header file
// 16F877.h for fuse option constants.
#Fuses LP,XT,HS,RC,NOWDT,WDT,NOPUT,PUT,PROTECT,PROTECT_5%

struct adc_result
    {
    int8 value;         // ADC built-in functions default to 8 bit result.
    int1 new_flag;      // 1-bit falg to indicate a new (fresh) value
    } adc_conversion;

void init_io()
{
    // set up PORTA so RA0 is an input for the ADC, RA1-RA7 are outputs
    SET_TRIS_A(0x01);   // bit 0 = output, 1 = input
    
    // set up PORTB so RB0 is an input for the switch SW1, RB1-RB7 are outputs
    SET_TRIS_B(0x01);   // bit 0 = output, 1 = input

    // set up remaining ports to be all outputs
    OUTPUT_C(0x00);
    OUTPUT_D(0x00);
    OUTPUT_E(0x00);
}

void init_adc()
{
    // Set up RA0 as the only analog input, using VDD and VSS 
    // as the references
    SETUP_ADC_PORTS(AN0 | 0x0000);

    // Turn on ADC and use Fosc/8 as the conversion clock
    // (Fosc = 4MHz)
    SETUP_ADC(ADC_CLOCK_DIV_8);   

    // Set channel for conversion to AN0 (on RA0)
    SET_ADC_CHANNEL(0);

    // start the first conversion
    READ_ADC(ADC_START_ONLY);
}

#INT_RTCC   // This function will be called on a Timer0 interrupt
void timer0_interrupt_service()
{
    // channel is always set to AN0.  Otherwise we might change
    // it here
    
    // Get last conversion result.
    adc_conversion.value = READ_ADC(ADC_READ_ONLY);
    adc_conversion.new_flag = 1;    // new value
    
    // start next conversion
    READ_ADC(ADC_START_ONLY);
}


void main()
{
    int1 led_bar_right = 0; // default is led bar graph starting on the left (DS0)   
    int switch_count = 0;   // used for debouncing the switch
    int bars = 0;           // used to compute the number of LED "bars" to display
    int temp1 = 0;          // temporary working variable
    int led_display = 0;    // led display value

    // initialize global variables
    adc_conversion.value = 0;
    adc_conversion.new_flag = 0;

    // Set up MCU
    init_io();
    init_adc();
        // Timer 0 runs off internal clock with 1:256 prescaler
        // This means it should run over every ((256*256)/(4MHz/4)) = 65.5ms
        // It will be used to start an ADC conversion.
    SETUP_TIMER_0(RTCC_INTERNAL | RTCC_DIV_256);
    ENABLE_INTERRUPTS(GLOBAL);          // enable interrupts in general to be active
    ENABLE_INTERRUPTS(INT_RTCC);        // enable the interrupt for Timer 0

    while(1)
    {
        // look for a new conversion result
        if (adc_conversion.new_flag == 1)
        { 
            bars = adc_conversion.value;
            adc_conversion.new_flag = 0;        // reset flag
            // We'll use the 3 most significant bits of the conversion result
            // to determine whether to display 1 to 8 "bars" on the LED display
            bars = (bars >> 5) + 1; // add one to make it 1 to 8 (vs 0 to 7)
            led_display = 0; // clear display variable
            for (temp1 = 0; temp1 < bars; temp1++)
            {
                // shift in a '1' for each bar
                led_display = (led_display << 1) + 1; 
            }
        }

        // check for a switch press & debounce
        if (INPUT_STATE(PIN_B0) == 0)
        { // switch is low when pressed
            if (switch_count < 8)
            { // increment the count on all consecutive checks where switch is pressed
              // to a max of 8
                switch_count ++;
                // if we've seen 8 consecutive checks where the switch is pressed
                // it's considered a valid switch press.  Reverse the bar graph
                if (switch_count == 8)
                {
                    led_bar_right = ~led_bar_right;
                }
            }
        }
        else
        { // anytime switch is detected as not pressed, reset the count
            switch_count = 0;
        }


        // update display
        if (led_bar_right == 0)
        {  // from left
            OUTPUT_D(led_display);
        }
        else
        { // from right
            // we have to shift it so it displays 1-8 bars.  Just complementing
            // would display 0-7.
            OUTPUT_D(~(led_display >> 1));
        }

    }

}       


