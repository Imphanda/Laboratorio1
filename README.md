//------------------------------------------------------------------------------
//                                                                             *
//    Filename:      LaboratorioJuego1                                         *
//    Fecha:         24/01/2020                                                *
//    Version:       v.1                                                       *
//    Author:        Nancy Alejandra Mazariegos                                *
//    Carnet:        17227                                                     *
//    Description:   Laboratorio 1                                             *
//                                                                             *
//------------------------------------------------------------------------------

// CONFIG1
#pragma config FOSC = INTRC_NOCLKOUT// Oscillator Selection bits (INTOSCIO oscillator: I/O function on RA6/OSC2/CLKOUT pin, I/O function on RA7/OSC1/CLKIN)
#pragma config WDTE = OFF       // Watchdog Timer Enable bit (WDT disabled and can be enabled by SWDTEN bit of the WDTCON register)
#pragma config PWRTE = OFF      // Power-up Timer Enable bit (PWRT disabled)
#pragma config MCLRE = ON       // RE3/MCLR pin function select bit (RE3/MCLR pin function is MCLR)
#pragma config CP = OFF         // Code Protection bit (Program memory code protection is disabled)
#pragma config CPD = OFF        // Data Code Protection bit (Data memory code protection is disabled)
#pragma config BOREN = OFF      // Brown Out Reset Selection bits (BOR disabled)
#pragma config IESO = OFF       // Internal External Switchover bit (Internal/External Switchover mode is disabled)
#pragma config FCMEN = OFF      // Fail-Safe Clock Monitor Enabled bit (Fail-Safe Clock Monitor is disabled)
#pragma config LVP = OFF        // Low Voltage Programming Enable bit (RB3 pin has digital I/O, HV on MCLR must be used for programming)

// CONFIG2
#pragma config BOR4V = BOR40V   // Brown-out Reset Selection bit (Brown-out Reset set to 4.0V)
#pragma config WRT = OFF        // Flash Program Memory Self Write Enable bits (Write protection off)

// #pragma config statements should precede project file includes.
// Use project enums instead of #define for ON and OFF.

#include <xc.h>
#define _XTAL_FREQ 8000000
#include<stdint.h>
uint8_t semaforo;
uint8_t contador1;
uint8_t contador2;
uint8_t banderaP1;
uint8_t banderaP2;
uint8_t banderaE;
uint8_t numeros[4]={63,6,91,79};
uint8_t puntos[8]={1,2,4,8,16,32,64,128};
uint8_t leds123[4]={0,1,2,4};

void inicio(void);
void display(void);
void puntajes(void);

void main(void){
    inicio();
    banderaE=0;
    while(1){
    display();
    puntajes();
    }
    
}
void inicio(void){// Se definen los puertos de entrada y salida
    ANSEL = 0;
    ANSELH = 0;
    TRISA = 0;//Salida leds jugador 1
    TRISB = 0;//Salida leds jugador 2
    TRISC = 0;//Salida Display
    TRISD = 0;//Salida semaforo
    TRISE = 1;//Entrada 3 botones
}
void display(void){
    if(PORTEbits.RE0==1&&banderaE==0)//Estado al presionar boton inicio
    {
         banderaP1=2;//No podran jugar mientras este el semaforo
         banderaP2=2;
         semaforo=0;
         banderaE=1;
    }
    if(PORTEbits.RE0==0&&banderaE==1){
        PORTC = numeros[semaforo];
        PORTD = leds123[semaforo];
        __delay_ms(1000);//tarda 1 segundo en cada numero
        if((++semaforo)==4){//al terminar la cuenta regresiva el contador queda en 0
            semaforo =0;
            banderaP1=0;//Ya puede jugar
            banderaP2=0;//Ya puede jugar
            contador1=0;
            contador2=0;
    }
    }
}  
void puntajes(void){
    if(PORTEbits.RE1==1&&banderaP1==0){
        banderaP1=1;
    }
    if(PORTEbits.RE1==0&&banderaP1==1){
        contador1++;
        PORTA=puntos[contador1];
        banderaP1=0;
        if(contador1==8&&contador1>contador2){
            PORTB=0;
            PORTA=0;
            PORTAbits.RA7=1;
        }
    }
    if(PORTEbits.RE2==1&&banderaP2==0){
        banderaP2=1;
    }
    if(PORTEbits.RE2==0&&banderaP2==1){
        contador2++;
        PORTB=puntos[contador2];
        banderaP2=0;
        if(contador2==8&&contador2>contador1){
            PORTA=0;
            PORTB=0;
            PORTBbits.RB7=1;
        }
    }
}
