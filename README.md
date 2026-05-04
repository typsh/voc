    #include <htc.h>
    #include <stdio.h>
    #include "lcd.h"

    // Guna 20MHz ikut standard SK40C, pastikan sama dengan crystal fizikal
    #define _XTAL_FREQ 20000000
    __CONFIG(0x2CF2);

    unsigned int m = 1;      // Nilai mula
    unsigned int last_m = 0; // Untuk kesan perubahan

    void main (void)
    {
    // 1. Setting I/O
    ANSEL = 0x00;
    ANSELH = 0x00;

    TRISB = 0b00000001; // RB0 sebagai Input (Butang)
    TRISC = 0b00000000; // PORTC sebagai Output (LED)
    TRISD = 0b00000000; // PORTD sebagai Output (LCD)

    PORTB = 0;
    PORTC = 0;
    PORTD = 0;

    // 2. Setting Interrupt
    INTE = 1;           // Enable External Interrupt RB0
    GIE = 1;            // Enable Global Interrupt
    INTEDG = 1;         // Trigger pada Rising Edge (Butang ke 5V)

    // 3. Initialize LCD
    __delay_ms(100);    // Tunggu LCD stabil
    lcd_init();
    lcd_clear();

    while (1)
    {
        // Hanya update skrin jika nilai m berubah (elak flicker)
        if (m != last_m)
        {
            lcd_clear();
            __delay_ms(5); // Delay sikit selepas clear

            // Logik LED dan Paparan LCD
            if(m==1) {
                PORTC = 0b00000000;
                lcd_printstring("Count: 0");
            }
            else if(m==2) {
                PORTC = 0b00000001;
                lcd_printstring("Count: 1");
            }
            else if(m==3) {
                PORTC = 0b00000011;
                lcd_printstring("Count: 2");
            }
            else if(m==4) {
                PORTC = 0b00000111;
                lcd_printstring("Count: 3");
            }
            else if(m==5) {
                PORTC = 0b00001111;
                lcd_printstring("Count: 4");
            }

            last_m = m; // Update status terakhir
        }
    }
    }

    // Fungsi Interrupt
    void interrupt intMain()
    {
    if(INTF) // Jika interrupt RB0 berlaku
    {
        __delay_ms(20); // Debounce ringkas
        if(RB0 == 1)    // Sahkan butang masih ditekan
        {
            m++;
            if(m > 5) m = 1;
        }

        INTF = 0; // Reset flag
    }
    }
