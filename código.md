; Archivo: Proyecto.s
; Dispositivo: PIC16F887
; Autor: Angela Figueroa
; Compilador: pic-a (v2.30), MPLABX V5.40
;
; Programa: GENERADOR DE FRECUENCIAS 
; Hardware: LEDs, push buttons, displays
;
; Creado: 28 de febrero, 2023
; Ultima modificacion: 19 de marzo, 2023
PROCESSOR 16F887
#include <xc.inc>
;CONFIGURACIÓN 1 
    CONFIG FOSC=INTRC_NOCLKOUT //OSCILADOR INTERNO SIN SALIDAS
    CONFIG WDTE=OFF // WDT DISABLE (REINICIO REPETITIVO DE PIC)
    CONFIG PWRTE=OFF // PWRT ENABLE (ESPERA DE 72MS AL INICIAR)
    CONFIG MCLRE=ON //EL PIN DE MCLR SE USA COMO I/O
    CONFIG CP=OFF //SIN PROTECCIÓN DE CÓDIGO
    CONFIG CPD=OFF //SIN PROTECCIÓN DE DATOS    
    CONFIG BOREN=OFF //SIN REINICIO CUANDO EL V DE ALIMENTACIÓN BAJA A 4V
    CONFIG IESO=OFF // REINICIO SIN CAMBIO DE RELOJ INTERNO A EXTERNO
    CONFIG FCMEN=OFF // CAMBBIO DE RELOJ EXTERNO A INTERNO EN CASO DE FALLO
    CONFIG LVP=OFF //PROGRAMACIÓN EN BAJO VOLTAJE PERMITIDA
    
 ;CONFIGURACIÓN 2
    CONFIG WRT=OFF //PROTECCIÓN DE AUTOESCRITURA POR EL PROGRAMA DESACTIVADA
    CONFIG BOR4V=BOR40V //REINICIO ABAJO DE 4, (BOR21V=2.1V)
    PSECT udata_bank0 ;common memory
;----------MEMORIA DE LAS VARIABLES----------------------------------------	
        w_tem: DS 1
	status_tem: DS 1
;----------HZ CUADRADA----------------------------------------    
	CONTADOR_HZ: DS 1
	ValhzC: DS 2
        ValhzT: DS 2
;----------DISPLAYS(4)----------------------------------------
	banderas: DS 1
	display0: DS 1
	display1: DS 1
	display2: DS 1
	display3: DS 1
	cont_tmr2: DS 1
;----------DIGITOS----------------------------------------
	unidades: DS 1
	decenas: DS 1
	centenas: DS 1
	miles: DS 1
;----------HZ diplay y ondas----------------------------------------
	ValHzDIS: DS 2
	banderas_ondas: DS 1
    ;----BOTONES----
    UP   EQU 0; PARA SUBIR
    DOWN EQU 1;PARA BAJAR
   

    PSECT resVect, class=CODE, abs, delta=2 
    ;--------------RESET-------------------------
    ORG 00h	
    resetVec:
	PAGESEL main
	goto main
    ;------------Rutina de interupción-------------------
    PSECT code, delta=2, abs
    ORG 004h	;Poscición 
    
    push:
	movwf w_tem
	swapf STATUS,0
	movwf status_tem
    isr:
	banksel PIR1
	banksel INTCON
	banksel PORTB
	
	btfss PORTB,2
	call cambiar_ondas
	
	btfsc PIR1, 1
	call generar_ondas
	
	btfsc INTCON,2
	call display_var
    pop:
	   swapf    status_tem,0
	   movwf    STATUS
	   swapf    w_tem,1
	   swapf    w_tem,0
	   retfie
     
    cambiar_ondas:;NOS PERMITE HACER EL CAMBIO DE ONDA DE C a T o DE T a C
	btfss PORTB,2
	goto $-1
	banksel banderas_ondas
	btfsc banderas_ondas,2
	goto $+3
	bsf   banderas_ondas,2
	goto $+2
	bcf   banderas_ondas,2
	return
	
    generar_ondas:;NOS AYUDA A GENERAR LAS ONDAS TRIANGULAR Y CUADRADA
	btfss banderas_ondas,2
	goto $+3
	call triangular
	goto $+2
	call cuadrada
	return
	
    cuadrada:;AYUDA A VER LOS LIMITES DE LA CUADRADA PARA GENERARLA
	banksel TMR2
	clrf TMR2
	bcf  PIR1, 1
	banksel PORTA
	btfsc banderas_ondas,1
	goto apagar_cuadrada
	movlw 255;TOPE
	movwf PORTA
	bsf banderas_ondas,1
	return
;--------------------------------------------------------------	
	apagar_cuadrada:;PONER EN MODO APAGADA
	clrf PORTA
	bcf banderas_ondas,1
	return
;------------------------------------------------------------	
    triangular:;AYUDA A VER LOS LIMITES DE LA TRIANGULAR PARA GENERARLA
	banksel TMR2
	clrf TMR2
	;bcf  PIR1, 1
	banksel PORTA
	btfsc banderas_ondas,0
	goto decremenrtar_triangular
	incf PORTA
	movf PORTA,0
	sublw 255;TOPE
	btfss STATUS, 2
	return
	bsf banderas_ondas,0
	return
;----------------------------------------------------------------------
	decremenrtar_triangular:;QUE PUEDA BAJAR 
	decfsz PORTA,1
	return
	bcf banderas_ondas,0
	return
;-----------------------------------------------------------------------
    ;------------Main----------------
     PSECT code, delta=2, abs
    ORG 100h	;Poscición 
    ;-----------Tabla para display 7 segmentos, anodo común----------------
    ;TABLA PARA LA CUADRADA EN HZ
    TABLA_HZ_CUADRADA:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 125;100hz
	retlw 83;150hz
	retlw 63;200hz
	retlw 50;250hz
	retlw 42;300hz
	retlw 36;350hz
	retlw 31;400hz
	retlw 28;450hz
	retlw 25;500hz
	retlw 23;550hz
	retlw 21;600hz
	retlw 19;650hz
	retlw 18;700hz
	retlw 17;750hz
	retlw 16;800hz
	retlw 15;850hz
	retlw 14;900hz
	retlw 13;950hz
	
    TABLA_HZ_TRI:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 125;100hz
	retlw 83;150hz
	retlw 63;200hz
	retlw 50;250hz
	retlw 42;300hz
	retlw 36;350hz
	retlw 31;400hz
	retlw 28;450hz
	retlw 25;500hz
	retlw 23;550hz
	retlw 21;600hz
	retlw 19;650hz
	retlw 18;700hz
	retlw 17;750hz
	retlw 16;800hz
	retlw 15;850hz
	retlw 14;900hz
	retlw 13;950hz	
;TABLA PARA LOS VALORES DEL DISPLAY DE LA CUADRADA EN HZ	
    VALORES_DISPLAY_CUADRADA_HZ:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 100
	retlw 150
	retlw 200
	retlw 250
	retlw 300
	retlw 350
	retlw 400
	retlw 450
	retlw 500
	retlw 550
	retlw 600
	retlw 650
	retlw 700
	retlw 750
	retlw 800
	retlw 850
	retlw 900
	retlw 950
	
    VALORES_DISPLAY_TRI_HZ:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 100
	retlw 150
	retlw 200
	retlw 250
	retlw 300
	retlw 350
	retlw 400
	retlw 450
	retlw 500
	retlw 550
	retlw 600
	retlw 650
	retlw 700
	retlw 750
	retlw 800
	retlw 850
	retlw 900
	retlw 950
;-----------CONFIGURACION DEL DISPLAY DE 7 SEGMENTOS----------------	
    TABLA_DISPLAY:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 01000000B;0
	retlw 11111001B;1
	retlw 00100100B;2
	retlw 00110000B;3
	retlw 00011001B;4
	retlw 00010010B;5
	retlw 00000010B;6
	retlw 01111000B;7
	retlw 00000000B;8
	retlw 00010000B;9
	
	
    main:
	call pines
	call ciocb
	call tmr0_config
	call tmr2_config
	call clock
    loop:
	banksel PORTA
	call variar_frecuencia 
	call preparar_hz_cuadrada
	call preparar_hz_tri
	call preparar_valores_display_hz_cuadrada
	call preparar_valores_display_hz_tri
	call obtener_centenas
	call obtener_decenas
	call obtener_unidades
	call displays
	goto loop
	
;-----------VALORES DE LA CUADRADA EN HZ----------------
    preparar_hz_cuadrada:
	clrf ValhzC
        movf CONTADOR_HZ,0
	call TABLA_HZ_CUADRADA
	call reset_tmr2
	movwf ValhzC
	return
	
    preparar_hz_tri:
	clrf ValhzT
        movf CONTADOR_HZ,0
	call TABLA_HZ_TRI
	call reset_tmr2
	movwf ValhzT
	return	
;-----------VALORES DEL DISPLAY EN HZ----------------
    preparar_valores_display_hz_cuadrada:
	banksel PORTA
	clrf ValHzDIS
	movf CONTADOR_HZ,0
	call VALORES_DISPLAY_CUADRADA_HZ
	movwf ValHzDIS
	return
	
    preparar_valores_display_hz_tri:
	banksel PORTA
	clrf ValHzDIS
	movf CONTADOR_HZ,0
	call VALORES_DISPLAY_TRI_HZ
	movwf ValHzDIS
	return	
;-----------OPERACION PARA VARIAR LA FRECUENCIA----------------	
    variar_frecuencia:
	btfss PORTB,0
	call incrementar
	btfss PORTB, 1
	call decrementar
	RETURN
;-----------OPERACION PARA INCREMENTAR----------------	
	incrementar:
	    btfss PORTB,0
	    goto $-1
	    incf CONTADOR_HZ
	    incf PORTA
	return
;-----------OPERACION PARA DECREMENTAR----------------	
	decrementar:
	    btfss PORTB,1
	    goto $-1
	    decf CONTADOR_HZ
	    decf PORTA
	return
;-----------RESETEAMOS EL TIMER 2----------------	
    reset_tmr2:
	banksel TRISA
	movwf	PR2
	return
;-----------DONDE SE VAN A COLOCAR LOS BOTONES----------------	
    pines:
	BANKSEL PORTD
	clrf	PORTD
	clrf	PORTA
	clrf	PORTC
	clrf	PORTB
	clrf	PORTE
	banksel TRISD
	bsf	TRISB,UP
	bsf	TRISB,DOWN
	clrf	TRISD
	clrf	TRISA
	clrf	TRISC
	clrf	TRISE
	banksel ANSEL
	clrf ANSEL
	clrf ANSELH
	
	;--------Confuración de PULL UP del PORTB------------------
	banksel	TRISA
	bcf	OPTION_REG,7 ;RBPU
	bsf	WPUB,UP
	bsf	WPUB,DOWN
	;----------------------------------------------------------
	return
	
;-----------CONFIGURACION DE LAS ENTRADAS Y SALIDAS----------------	
     ciocb:
	banksel TRISA
	bsf	IOCB,UP
	bsf	IOCB,DOWN
	BSF	PIE1, 1
	banksel PORTA
	bsf	INTCON, 7
	bsf	INTCON, 6
	bcf	PIR1,1
    return
    
  ;-----------EL RELOJ----------------  
    clock:
	banksel OSCCON;Oscillator Control Register 
	bsf IRCF2	  ;-------Reloj de 1 Mhz
	bsf IRCF1	  
	bsf IRCF0	  
	bsf SCS		  ;Internal oscillator is used
	return
	
;-----------CONFIGURACION DEL TIMER 0----------------	
     tmr0_config:
	banksel TRISA
	Movlw 01010111B	    ;Configuración
	movwf OPTION_REG    ;Cargar la configuración
	banksel PORTA
	banksel TMR0
	movlw	176;
	movwf	TMR0
	bcf	INTCON,2
	return
	
;-----------CONFIGURACION DEL TIMER 2----------------	
    tmr2_config:
	banksel T2CON
	movlw 1001111B
	movwf T2CON
	banksel TRISA
	movlw	125
	movwf	PR2
	banksel	PORTA
	clrf	TMR2
	bcf	PIR1,1
	return
;OPERACIONES PARA OBTENER LOS DIGITOS EN CENTENAS, DECENAS Y UNIDADES
     obtener_centenas:
	clrf centenas
	movlw 100
	subwf ValHzDIS, 1
	incf centenas
	btfsc STATUS, 0
	goto $-3
	decf centenas
	movlw 100
	addwf ValHzDIS, 1
    return
 ;OPERACIONES PARA OBTENER LOS DIGITOS EN CENTENAS, DECENAS Y UNIDADES   
    obtener_decenas:
	clrf decenas
	movlw 10
	subwf ValHzDIS, 1
	incf decenas
	btfsc STATUS, 0
	goto $-3
	decf decenas
	movlw 10
	addwf ValHzDIS, 1
    return
 ;OPERACIONES PARA OBTENER LOS DIGITOS EN CENTENAS, DECENAS Y UNIDADES   
    obtener_unidades:
	clrf unidades
	movf ValHzDIS,0
	movwf unidades
	
    return
 ;COLOCAMOS CUANTOS DISPLAYS USAREMOS (EN ESTE CASO 4)   
     displays:
	 movf	miles, W
	call	TABLA_DISPLAY
	movwf	display0
	
	movf	centenas, W
	call	TABLA_DISPLAY
	movwf	display1
	
	movf	decenas, W
	call	TABLA_DISPLAY
	movwf	display2
	
	movf	unidades, W
	call	TABLA_DISPLAY
	movwf	display3
	
	return
	
;DISPLAY VAR NOS AYUDA A REVISAR LAS BANDERAS Y COLOCAR LAS CEN,UNI,DEC	
     display_var:
	clrf	PORTD
	btfsc	banderas, 0
	goto    display_centenas
	btfsc	banderas, 1
	goto    display_decenas
	btfsc	banderas, 2
	goto    display_unidades
;-----MILES------------	
    display_miles:
	movf	display0, 0
	movwf	PORTC
	BSF	PORTD, 0
	goto	toggle
;-----CENTENAS------------	
    display_centenas:
	movf	display1, 0
	movwf	PORTC
	BSF	PORTD, 1
	goto	toggle
;-----DECENAS------------	
    display_decenas:
	movf	display2, 0
	movwf	PORTC
	BSF	PORTD, 2
	goto	toggle
;-----UNIDADES------------	
    display_unidades:
	movf	display3, 0
	movwf	PORTC
	BSF	PORTD, 3
	
;---REVISAMOS LAS BANDERAS------	
    toggle:
	incf	banderas
	return
END
   
