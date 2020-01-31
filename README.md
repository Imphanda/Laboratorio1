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
// PIC16F887 Configuration Bit Settings

// 'C' source line config statements

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
uint8_t semaforo;//contador numeros para display y semaforo
uint8_t contador1;//puntos P1
uint8_t contador2;//puntos P2
uint8_t banderaP1;//Antirrebote1
uint8_t banderaP2;//Antirrebote2
uint8_t banderaE;//Antirrebote boton inicio
uint8_t numeros[5]={79,91,6,63,118};//mostrar en el display
uint8_t puntos[9]={0,1,2,4,8,16,32,64,128};//aumento decadas
uint8_t leds123[5]={128,128,64,32,224};//aumento semaforo

void inicio(void);
void display(void);
void puntajes(void);

void main(void){
    inicio();
    banderaE=0;//Habilitar inicio
    contador1=0;//inicio puntos
    contador2=0;//inicio puntos
    semaforo=0;
    PORTA=puntos[contador1];//Set puertos en 0
    PORTB=puntos[contador2];
    PORTC = numeros[4];//Letra H en el display (Hola)
    PORTD = leds123[semaforo];//Rojo al inicio
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
         banderaE=1;//control
    }
    if(PORTEbits.RE0==0&&banderaE==1){
        contador1=0;//inicio puntos
        contador2=0;//inicio puntos
        PORTA=puntos[contador1];//Reinicia los puertos 
        PORTB=puntos[contador2];
        PORTC = numeros[semaforo];//Display y Leds
        PORTD = leds123[semaforo];
        __delay_ms(600);//tarda 0.6 segundos en cada numero
        if((++semaforo)==4){//al terminar la cuenta regresiva el contador queda en 0
            banderaP1=0;//Ya puede jugar
            banderaP2=0;//Ya puede jugar
            banderaE=0;//termina display
            contador1=0;//inicio puntos
            contador2=0;//inicio puntos
    }
    }
}  
void puntajes(void){
    if(PORTEbits.RE1==1&&banderaP1==0){
        banderaP1=1;//bandera de control
    }
    if(PORTEbits.RE1==0&&banderaP1==1){
        contador1++;//aumento de puntos P1
        PORTA=puntos[contador1];
        banderaP1=0;
        if(contador1==9&&contador1>contador2){
        PORTA=puntos[8];    
        banderaP1=2;//Bloquea botones
        banderaP2=2;
        contador2=0;//Apaga leds 2do lugar
        PORTB=puntos[contador2];
        PORTC = numeros[2];//Numero jugador ganador 1
        PORTD = leds123[5];
        PORTDbits.RD3=0;//Las 3 luces de semaforo encendidas
        PORTDbits.RD1=1;//enciende led ganador
        PORTDbits.RD6=1;//enciende led ganador
        }
    }
    if(PORTEbits.RE2==1&&banderaP2==0){
        banderaP2=1;//bandera de control
    }
    if(PORTEbits.RE2==0&&banderaP2==1){
        contador2++;//aumento de puntos P2
        PORTB=puntos[contador2];
        banderaP2=0;
        if(contador2==9&&contador2>contador1){
            PORTB=puntos[8];
            banderaP1=2;//bloquea botones
            banderaP2=2;
            contador1=0;//Apaga leds 2do lugar
            PORTA=puntos[contador1];
            banderaP1=2;
            banderaP2=2;
            PORTC = numeros[1];//Numero jugador ganador 2
            PORTD = leds123[5];
            PORTDbits.RD3=1;//Las 3 luces de semaforo encendidas
            PORTDbits.RD1=0;//enciende led ganador
            PORTDbits.RD6=1;//enciende led ganador
        }
    }
}
