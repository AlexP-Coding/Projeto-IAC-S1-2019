# Projeto-IAC-S1-2019
;############################################################
;	Projeto 1o Semestre IAC - Pulsares
;	Por: 	Alexandra Saraiva Pato  - 97375
;		João Pedroso Raposo    - 97377
;	Grupo: 42
;
;	Descrição: Jogo onde o jogador tem que
;	sobreviver o máximo tempo possível sem
;	ser ficar sem energia ou ser abatido
;
;	Género: Shooter/Bullethell
;
;############################################################






;############################################################
;	Constantes
;############################################################

;	Stack pointer

PLACE 3D00H
BTEInicio:			; Endereço da tabela de interrupções
	WORD	RotInt00
	WORD	RotInt01
	WORD	RotInt02
	WORD	RotInt03
	
BTEFim:
Pilha:	
TABLE 100H
PilhaFim:   			; último endereço da RAM + 1
;********************************************

;	Display energia

DISPLAYS	EQU 0A000H  ; endereço dos displays
GIFINICIO	EQU 0		; imagem do gif de fundo
GIFFIM		EQU 7	; última imagem do gif de fundo
;********************************************

;	Teclado

TECLIN		EQU 0C000H  	; endereço das linhas 	do teclado
TECCOL		EQU 0E000H  	; endereço das colunas do teclado
LINHA 		EQU 0001b   	; linha 0 do teclado
;********************************************

;	Pixel screen

PSBASE		EQU 6000H    	; endereço da componente vermelha da cor
GREEN		EQU 6002H		; endereço da componente verde da cor
BLUE		EQU	6004H		; endereço da componente azul da cor
ALPHA		EQU	6006H		; endereço da transparência
IOPIXEL		EQU	6008H		; ligar/desligar pixel
PSLIN		EQU	600AH		; Nº linha
PSCOL		EQU	600CH		; Nº coluna
PSCENA		EQU	600EH		; Nº cenário a usar
PSCLEAR		EQU	6010H		; Limpar ecrã / Nº de cenários definidos
PSSOM		EQU	6012H		; Nº do som a tocar 

; EXCLUSIVAMENTE DE LEITURA
PSNUMSOM	EQU 6014H		; Nº do sons definidos
PSNUMCOL	EQU 6016H		; Nº de colunas do ecrã 
PSNUMLIN	EQU 6018H		; Nº de linhas do ecrã
;********************************************
; 	Valores relacionados com funcoes
LARGURA_X_NAVE		EQU 08H
COMPRIMENTO_Y_NAVE	EQU 0AH

;*******************************************************************************************
;	Definicoes dos templates dos objetos
;*******************************************************************************************
PLACE 2000H
Nave: 						; Setup é igual para todos os templates
		WORD 75	   	; Red
		WORD 0	   		; Green
		WORD 125	   		; Blue
		WORD 255			; Alpha
		WORD 3				; Largura total (x)
		WORD 3				; Comprimento total (y)
		WORD 0E000H  		; Linha 1 da Nave -> 1110 0000 0000 0000 b
		WORD 0A000H 		; Linha 2 da Nave -> 1010 0000 0000 0000 b
		WORD 0E000H 		; Linha 3 da Nave -> 1110 0000 0000 0000 b
		WORD -1			; Stop do vetor
Escudo:
		WORD 107		
		WORD 255	
		WORD 203	
		WORD 255
		WORD 1
		WORD 1	
		WORD 0A000H 
 		WORD 04000H	 
		WORD 0A000H
		WORD -1	

apagarEscudo:
		WORD 0	
		WORD 0	
		WORD 0	
		WORD 0
		WORD 1
		WORD 1	
		WORD 0E000H 
 		WORD 0E000H	 
		WORD 0E000H
		WORD -1	



PulsarMau:	
		WORD 255		
		WORD 0		
		WORD 0		
		WORD 255
		WORD 3
		WORD 3	
		WORD 0E000H
 		WORD 0E000H
		WORD 0E000H
		WORD -1	 
		
PulsarMauSemiVis:	
		WORD 255		
		WORD 0		
		WORD 0		
		WORD 100
		WORD 3
		WORD 3	
		WORD 0E000H
 		WORD 0E000H
		WORD 0E000H
		WORD -1	 
			
PulsarBom:	
		WORD 0		
		WORD 255		
		WORD 0		
		WORD 255
		WORD 3
		WORD 3	
		WORD 0E000H
 		WORD 0E000H
		WORD 0E000H
		WORD -1	
		

PulsarBomSemiVis:	
		WORD 0		
		WORD 255		
		WORD 0		
		WORD 100
		WORD 3
		WORD 3	
		WORD 0E000H
 		WORD 0E000H
		WORD 0E000H
		WORD -1	
	
		
RaioMau:	
		WORD 255		
		WORD 0		
		WORD 0		
		WORD 255
		WORD 1
		WORD 1		
		WORD 8000H	
		WORD -1	
		
RaioBom:	
		WORD 0		
		WORD 255		
		WORD 0		
		WORD 255
		WORD 1
		WORD 1
		WORD 8000H		
		WORD -1		


;	As 7 cores do arco-iris, para referencia da Nave
Cor0:	; vermelho
	WORD 255
	WORD 0
	WORD 0
Cor1:	; laranja
	WORD 255
	WORD 127
	WORD 0
Cor2:	; amarelo
	WORD 255
	WORD 255
	WORD 0
Cor3:	; verde
	WORD 0
	WORD 255
	WORD 0
Cor4:	; azul
	WORD 0
	WORD 0
	WORD 255
Cor5:	; indigo
	WORD 75
	WORD 0
	WORD 130
Cor6:	; roxo
	WORD 139
	WORD 0
	WORD 255






;############################################################
;	Variáveis
;############################################################


;*************************************************************************************
;*************************************************************************************
;	As primeiras 5 words correspondem as mesmas componentes
;	em todas as variaveis dos objetos pulsar/raio
;	
;	Variaveis dos objetos
;*************************************************************************************
;*************************************************************************************


naveVar:	WORD 32		; posição no eixo x
			WORD 16 	; posição no eixo y
			WORD 32 	; ultima posicao de desenho x
			WORD 16 	; ultima posicao de desenho y
			WORD -1		; se = -1  nao desenhar, =1 desenhar Nave
naveEscudo: WORD -1		; escudo ativo = 1, inativo = -1
naveEnergia:WORD 105	; energia

escudoVar:	WORD 32		; posição no eixo x
			WORD 16 	; posição no eixo y
			WORD 32 	; ultima posicao de desenho x
			WORD 16 	; ultima posicao de desenho y
			WORD -1		; se = -1  nao desenhar, =1 desenhar escudo
			

pulsares:		
pulsar1Var:	
		WORD 0	; posição no eixo x
		WORD 0	; posição no eixo y
		WORD 0  ; ultima posicao de desenho x
		WORD 0  ; ultima posicao de desenho y
		WORD 1	; se = -1  nao desenhar, =1 desenhar
		WORD 1  ; se = -1 o pulsar é mau, =1 o pulsar é bom
pulsar2Var:	
		WORD 61	; posição no eixo x
		WORD 0	; posição no eixo y
		WORD 61 ; ultima posicao de desenho x
		WORD 0  ; ultima posicao de desenho y
		WORD 0	; se = -1  nao desenhar, =1 desenhar
		WORD 1  ; se = -1 o pulsar é mau, =1 o pulsar é bom
pulsar3Var:	
		WORD 0	; posição no eixo x
		WORD 29	; posição no eixo y
		WORD 0  ; ultima posicao de desenho x
		WORD 29 ; ultima posicao de desenho y
		WORD 1	; se = -1  nao desenhar, =1 desenhar
		WORD 1  ; se = -1 o pulsar é mau, =1 o pulsar é bom
pulsar4Var:	
		WORD 61	; posição no eixo x
		WORD 29	; posição no eixo y
		WORD 61 ; ultima posicao de desenho x
		WORD 29 ; ultima posicao de desenho y
		WORD 1	; se = -1  nao desenhar, =1 desenhar
		WORD 1  ; se = -1 o pulsar é mau, =1 o pulsar é bom
		
raios:
raio1Var:	
		WORD 1	; posição no eixo x
		WORD 1	; posição no eixo y
		WORD 1  ; ultima posicao de desenho x
		WORD 1  ; ultima posicao de desenho y
		WORD 1 	; direcao x
		WORD 1 	; direcao y
		WORD 0  ; se = -1  nao desenhar
		WORD 0  ; se = -1 o raio é mau, =1 o raio é bom
		WORD 1	; posição no eixo x inicial
		WORD 1	; posição no eixo y inicial
raio2Var:	
		WORD 62	; posição no eixo x
		WORD 1	; posição no eixo y
		WORD 62 ; ultima posicao de desenho x
		WORD 1  ; ultima posicao de desenho y
		WORD -1 ; direcao x
		WORD 1 	; direcao y
		WORD 0  ; se = -1  nao desenhar
		WORD 0  ; se = -1 o raio é mau, =1 o raio é bom
		WORD 62	; posição no eixo x inicial
		WORD 1	; posição no eixo y inicial
raio3Var:	
		WORD 1	; posição no eixo x
		WORD 30	; posição no eixo y
		WORD 1  ; ultima posicao de desenho x
		WORD 30 ; ultima posicao de desenho y
		WORD 1 	; direcao x
		WORD -1 ; direcao y
		WORD 0  ; se = -1  nao desenhar
		WORD 0  ; se = -1 o raio é mau, =1 o raio é bom
		WORD 1	; posição no eixo x inicial
		WORD 30	; posição no eixo y inicial
raio4Var:	
		WORD 62	; posição no eixo x
		WORD 30	; posição no eixo y
		WORD 62 ; ultima posicao de desenho x
		WORD 30 ; ultima posicao de desenho y
		WORD -1 	; direcao x
		WORD -1 	; direcao y
		WORD 0  ; se = -1  nao desenhar
		WORD 0  ; se = -1 o raio é mau, =1 o raio é bom
		WORD 62	; posição no eixo x inicial
		WORD 30	; posição no eixo y inicial
		
		
		
;*************************************************************************************
;	Variaveis nao correlacionadas com objetos
;*************************************************************************************
distRaio: 	WORD 0EH	; distancia entre as variaveis dos diversos pulsares/raios
distPulsaresVar: 	WORD 0CH	; distancia entre os pulsares/raios
distDirecaoRaio: WORD 08H ; distancia para as variaveis do movimento dos raios
geradorVar1: WORD 415H		; variavel do Gerador_ pseudoaleatorio
bitsEmWord: WORD 10H	; No de bits numa word
distCor:	WORD 06H	; distancia entre a base das estruturas da cor
distWord:   WORD 02H    ; distancia entre words
noTotalCores:	WORD 7		; No de cores definidas para a nave
AlphaToVForma: 	WORD 06H		; distancia da componente alpha para o vetor forma
distVetorForma: WORD 0CH	; distancia da base do template até ao vetor forma
distUltimaPosx: WORD 04H	; distancia da base das vars ate ao ultimo x de desenho
cenaInicio:		WORD 8	; ATENÇAO No de imagens de fundo
cenaFinal:		WORD 9	
somMovimento:	WORD 1	; som ouvido ao mover a nave
ultimoInputTratado: WORD 0
lastInput: 			WORD 0
numCor: WORD 0H
noCena: WORD 0000H
UEUltimoEscudo: WORD -1	; 0 = nao apagou escudo, 1 = apagou escudo
NoRaioColisao: WORD 0; numero do raio que colidiu com a nave
RaiosOutOfBounds: WORD -1 ; -1 == raios dentro do ecra, 1 == raios fora do ecra
estadoPulsaçao: WORD 1; -1 == semi-transparentes, 1 == opacos


LOC_TIPOPULSAR EQU 0CH
LOC_TIPO_RAIO EQU 0EH
TIPO_RAIO_MAU EQU 0FFFFH
VAR_DESENHAR_RAIO EQU 0CH
ENERGIA_RAIOBOM_C_ESCUDO EQU 30
ENERGIA_RAIOBOM_S_ESCUDO EQU 0
ENERGIA_RAIOMAU_C_ESCUDO EQU 0FFECH	; -20
ENERGIA_RAIOMAU_S_ESCUDO EQU 0FB00H ; -500
;############################################################
;	Loop principal (Main)

;############################################################
PLACE 0

;*************************************************************************************
;*************************************************************************************
;	Inicializacoes
;*************************************************************************************
;*************************************************************************************

	MOV SP, PilhaFim	  	; inicializar stack pointer
	MOV BTE, BTEInicio		; iniciar pointer BTE (interrupcoes)



;*******************************************************************
;	Setup do ecra inicial quando se carrega o .asm
;*******************************************************************
EcraInicial:
;	Mover a cena inicial para o background do pixel screen
	MOV R0, PSCENA
	MOV R1, cenaInicio				; mover para R1 o endereço do cenário do ecrã de jogo
	MOV R1, [R1]					; ler número de imagem a utilizar
	MOV [R0], R1					; mostrar imagem a utilizar como fundo
;	Tocar o som inicial do jogo
	MOV R0, PSSOM				
	MOV R1, 0	; som de início do jogo
	MOV [R0], R1		; guardar o som de início do jogo
EcraInicialLoop1:
	CALL Input_					; verificar se há input
	MOV R0, lastInput				; endereço do último input
	MOV R0, [R0]					; ler valor do último input
	MOV R1, 0CH	
	CMP R0, R1					
	JEQ OutEcraInicial				; iniciar jogo se se premir C
	JMP EcraInicialLoop1				; permanecer no ecrã inicial caso não se prima C
	
OutEcraInicial:
;*******************************************************************
;	Reinicializacao das variaveis para o valor no inicio do jogo
;*******************************************************************
Inicio:
;	Repor energia e posicao da nave, desativar o escudo (valores iniciais da nave)
	MOV R0, naveEnergia
	MOV R1, 100
	MOV [R0], R1
	MOV R0, naveVar
	MOV R1, 32
	MOV [R0], R1
	ADD R0, 2
	MOV R1, 16
	MOV [R0], R1
	MOV R0, naveEscudo
	MOV R1, -1
	MOV [R0], R1
	
;	Repor valores iniciais do escudo
	MOV R0, escudoVar
	MOV R1, 32
	MOV [R0], R1
	ADD R0, 2
	MOV R1, 16
	MOV [R0], R1
	ADD R0, 6
	MOV R1, -1
	MOV [R0], R1
;	Restaurar flags iniciais
	MOV R0,flagInput_	   
	MOV R1, 1
	MOV [R0], R1
	MOV R0, flagUpdate_Nave_Ecra
	MOV R1, 1
	MOV [R0], R1
	MOV R0, flagGerador_Add
	MOV R1, 1
	MOV [R0], R1
	MOV R0, flagTratar_Input
	MOV R1, 1
	MOV [R0], R1
	MOV R0, flagMudar_Cor_Nave
	MOV R1, 1
	MOV [R0], R1
	MOV R0, flagLoop_BG
	MOV R1, 1
	MOV [R0], R1
	MOV R0, flagTick_Energia
	MOV R1, 0
	MOV [R0], R1
	MOV R0, flagPiscar_Pulsares
	MOV R1, 1
	MOV [R0], R1
;********************************************
;	Iniciar nave, pulsares e background
	CALL Reset_Ecra
	CALL Loop_BG
	MOV R0, Nave
	MOV R1, naveVar
	MOV R10, 0
	CALL Desenhar_Objeto
	MOV R10, 0
	CALL Piscar_Pulsares
;***********************************
	JMP main0

;*************************************************************************************
;	flags que definem se a funcao é executada na main, sao alteradas nos interrupts
;*************************************************************************************
flagInput_:				WORD	1	   	
flagUpdate_Nave_Ecra:	WORD	1   	
flagUpdate_Escudo:		WORD	1
flagGerador_Add:		WORD	1
flagTratar_Input:		WORD	1
flagMudar_Cor_Nave:		WORD	1
flagLoop_BG:			WORD	1
flagTick_Energia:		WORD	0
flagPiscar_Pulsares:	WORD	1
flagPausa:				WORD 	0
flagReinicio:			WORD 	0 

;*************************************************************************************
;	Loop de jogo que chama constantemente as diversas funções
;*************************************************************************************
Main:
	EI
	EI0
	EI1
	EI2
	EI3
	DI
main0:
	CALL Input_
	CALL Gerador_Add	
main1:	
	MOV R0, flagTratar_Input
	MOV R0, [R0]
	CMP R0, 0
	JEQ main2
	CALL Tratar_Input
	MOV R0, flagTratar_Input
	MOV R1, 0
	MOV [R0], R1
main2:	
	MOV R0, flagPiscar_Pulsares
	MOV R0, [R0]
	CMP R0, 0
	JEQ main3
	;	preparar R10 como argumento para Piscar_Pulsares
	MOV R10, 0
	CALL Piscar_Pulsares
	MOV R0, flagPiscar_Pulsares
	MOV R1, 0
	MOV [R0], R1
main3:	
	MOV R0, flagMudar_Cor_Nave
	MOV R0, [R0]
	CMP R0, 0
	JEQ main4
	CALL Mudar_Cor_Nave
	MOV R0, flagMudar_Cor_Nave
	MOV R1, 0
	MOV [R0], R1
main4:	
	MOV R0, flagUpdate_Nave_Ecra
	MOV R0, [R0]
	CMP R0, 0
	JEQ main5
	CALL Update_Nave_Ecra
	MOV R0, flagUpdate_Nave_Ecra
	MOV R1, 0
	MOV [R0], R1
main5:	
	MOV R0, flagLoop_BG
	MOV R0, [R0]
	CMP R0, 0
	JEQ main6
	CALL Loop_BG
	MOV R0, flagLoop_BG
	MOV R1, 0
	MOV [R0], R1
main6:	
	MOV R0, flagTick_Energia
	MOV R0, [R0]
	CMP R0, 0
	JEQ main7
	CALL Tick_Energia
	MOV R0, flagTick_Energia
	MOV R1, 0
	MOV [R0], R1
main7:	
	MOV R0, flagUpdate_Escudo
	MOV R0, [R0]
	CMP R0, 0
	JEQ main8
	CALL Update_Escudo
	MOV R0, flagUpdate_Escudo
	MOV R1, 0
	MOV [R0], R1
main8:
	
;********************************
;	Caso a energia seja 0, acabar o jogo
	MOV R0, naveEnergia
	MOV R0, [R0]
	MOV R1, 0
	CMP R0, R1
	JEQ GameOver
;********************************
OutMain:
	EI
	JMP Main







;****************************************************************************
Pausa:
	PUSH RE
	DI
MainPausa:
	CALL Input_
	MOV R0, lastInput
	MOV R1, ultimoInputTratado
	MOV R0, [R0]
	MOV R1, [R1]
	CMP R0, R1
	JEQ MainPausa
	CALL Tratar_Input
	CALL Input_
	MOV R0, lastInput
	MOV R0, [R0]
	MOV R1, 0BH
	CMP R0, R1
	JEQ OutPausa
	JMP MainPausa

OutPausa:
	POP RE
	RET
;****************************************************************************
GameOver:
	DI
	CALL Reset_Ecra
;	Mover a cena final para o background do pixel screen 
	MOV R0, PSCENA
	MOV R1, cenaFinal
	MOV R1, [R1]
	MOV [R0], R1
;	Tocar o som final do jogo
	MOV R0, PSSOM
	MOV R1, 4	; som do fim do jogo
	MOV [R0], R1
GameOverLoop1:
	CALL Input_
	MOV R0, lastInput
	MOV R0, [R0]
	MOV R1, 0CH
	CMP R0, R1
	JEQ OutGameOver
	JMP GameOverLoop1
	
OutGameOver:
	JMP Inicio
;****************************************************************************
	
	
	
	
	
;############################################################
;############################################################
;	Interrupcoes do PEPE
;############################################################
;############################################################

;	Clock de 1000ms
RotInt00:
	PUSH RE
	PUSH R0
	PUSH R1
	CALL Gerador_Add
	POP R1
	POP R0
	POP RE
	RFE
;	Clock de 300ms	
RotInt01:
	PUSH RE
	PUSH R0
	PUSH R1
	MOV R1, 1
	MOV R0, flagLoop_BG
	MOV [R0], R1
	MOV R0, flagMudar_Cor_Nave
	MOV [R0], R1
	CALL Gerador_Add
	POP R1
	POP R0
	POP RE
	RFE
;	Clock de 3000ms
RotInt02:
	PUSH RE
	PUSH R0
	PUSH R1
	MOV R1, 1
	MOV R0, flagTick_Energia
	MOV [R0], R1
	MOV R0, flagPiscar_Pulsares
	MOV [R0], R1
	CALL Gerador_Add
	POP R1
	POP R0
	POP RE
	RFE
;	Clock de 100ms
RotInt03:
	PUSH RE
	PUSH R0
	PUSH R1
	MOV R1, 1
	MOV R0, flagTratar_Input
	MOV [R0], R1
	MOV R0, flagUpdate_Nave_Ecra
	MOV [R0], R1
	MOV R0, flagUpdate_Escudo
	MOV [R0], R1
	CALL Gerador_Add
	POP R1
	POP R0
	POP RE
	RFE






;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Tick_Energia: Rotina que da update da energia segundo o clock
;	Args: Nenhum	
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Tick_Energia:
	PUSH R0						; guardar valor dos registos a alterar
	PUSH R1
	
	MOV R0, naveEscudo				; endereço das posi
	MOV R0, [R0]
	MOV R1, 1
	CMP R0, R1
	JEQ TE1
	JMP TE2
TE1:
	MOV  R0, -10 ; escudo ativo
	JMP TE3
TE2:
	MOV R0, -5	; escudo inativo
	JMP TE3
TE3:
	CALL Update_Energia
	
OutTick_Energia:
	POP R1
	POP R0
	RET
;*********************************

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Play_Som: Rotina que toca o som cujo No e passado em R0 
;	Args: R0 - Numero do som a tocar 
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Play_Som:
	PUSH R0		; guardar valor dos registos a alterar
	PUSH R1
	MOV R1, PSSOM	; Move para R1, o endereco dos sons no PS
	MOV [R1], R0	; 
	POP R1		; repor valor dos registos alterados
	POP R0
	RET

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Update_Energia: Rotina que da update á energia da nave
;	e renova a valor nos displays
;	Args: R0 - Numero a adicionar (subtrair se for negativo)
;			   ao valor da energia da nave
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Update_Energia:
	PUSH R1					; guardar valor dos registos a alterar
	PUSH R2
;	Guardar em R1 a energia atual da nave e adiciona-se-lhe 
;	o valor de R0 
	MOV R1, naveEnergia			; endereço da energia da nave
	MOV R1, [R1]					; ler energia atual da nave
	ADD R0, R1					; adicionar à energia o valor de R0
;	Verificar se a energia excedeu 100, caso se verifique, 
;	saltar para codigo onde se maximiza a energia a 100
	MOV R2, 100
	CMP R0, R2
	JGE UEnergia1				; se a energia final é superior ou igual a 100, maximizar 
;********************************
	CMP R0, 0					; se a energia final é inferior a zero, 
	JLT UEnergia2
;************************************
	MOV R2, R0					; atualizar valor da energia da nave
	CALL Display_Em_DEC			; atualizar os displays
	JMP OutUEnergia
UEnergia1:
;	Maximizar o valor da energia a 100
	MOV R2, 100	; maximizar energia
	MOV R1, R0	; R0 contem o valor da energia + valor novo
	SUB R1, R2	; o excesso superior a 100 é guardado em R1
	SUB R0, R1	; subtrai-se o excesso e fica somente 100
	MOV R1, R0	; atualizar energia		
	MOV R2, R0	; ?atualizar energia (cópia)
	CALL Display_Em_DEC			; atualizar os displays
	JMP OutUEnergia
	
UEnergia2:
	MOV R2, 0					; minimizar energia a zero
	MOV R0, R2 					; atualizar energia
	CALL Display_Em_DEC			; atualizar os displays
	JMP OutUEnergia
	
OutUEnergia:
	MOV R1, naveEnergia			; endereço da energia da nave
	MOV [R1], R2					; atualizar energia da nave
	POP R2					; repor valor dos registos alterados
	POP R1
	RET

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Update_Escudo: Rotina que verifica se ouve mudanca na 
;	posicao do escudo, e faz o draw respetivo 
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Update_Escudo:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4 
	PUSH R5
	PUSH R6
	PUSH R10
	
	
;**************************************************************
	
	;	Comparar a posicao atual com as antigas posicoes de desenho
	;	As variaveis sao movidas para os registos da seguinte forma:
	;	X atual: R1				Y atual: R2
	;	X antigo: R3			Y antigo: R4
	MOV R1, naveVar
	MOV R2, R1
	ADD R2, 2
	MOV R3, escudoVar
	ADD R3, 4
	MOV R4, R3 
	ADD R4, 2 
	MOV R1, [R1]
	MOV R2, [R2]
	MOV R3, [R3]
	MOV R4, [R4]
	;	Guardar as posicoes atuais para mais tarde
	MOV R5, R1	
	MOV R6, R2 
;*************************************************************
		
	MOV R0, naveEscudo
	MOV R0, [R0]
	MOV R1, 0FFFFH
	CMP R0, R1
	JEQ UEscudo3	
	
	CMP R1, R3	;	Caso sejam iguais, o escudo nao mudou o x
	JEQ UEscudo1
	JMP UEscudo2

;	Verificar se houve mudanca de y
UEscudo1:
	CMP R2, R4
	JEQ UEscudo3
	JMP UEscudo2
	
;	Havendo mudanca de x ou y, apagar o escudo antigo, e desenhar
UEscudo2:
	MOV R0, Escudo
	MOV R1, naveVar
	MOV R10, 1
	CALL Desenhar_Objeto
	JMP OutUEscudo


;	Caso o escudo esteja desativado, apagar o escudo e redesenhar a nave
UEscudo3:
		MOV R0, UEUltimoEscudo
		MOV R0, [R0]
	MOV R1, naveEscudo
	MOV R1, [R1]
	CMP R0, R1
	JEQ OutUEscudo	; caso na ultima iteracao já se tenha saltado para UEscudo3,
					; sair logo da rotina
;	Apagar todos os pixels na area 3x3 em cima da nave
	MOV R0, apagarEscudo
	MOV R1, naveVar
	MOV R10, 1
	CALL Desenhar_Objeto
;	redesenhar a nave
	MOV R0, Nave
	MOV R1, naveVar
	MOV R10, 1
	CALL Desenhar_Objeto
	
	JMP OutUEscudo




;	Caso nao haja mudancas nas variaveis, nao se desenha, acabando a funcao
;	Retornar Update_Escudo
OutUEscudo:
	MOV R1, R5 
	MOV R2, R6 
	MOV R3, escudoVar		;	Mover para R3 o endereco do x antigo
	ADD R3, 04H			
	MOV R4, escudoVar		;	Mover para R4 o endereco do y antigo
	ADD R4, 06H	
	;	Atualizar a var das ultimas posicoes de desenho com a variavel atual
	MOV [R3], R1
	MOV [R4], R2
	
;	Guardar o valor de naveEscudo na variavel que define o ultimo escudo processado
	MOV R0, naveEscudo
	MOV R0, [R0]
	MOV R1, UEUltimoEscudo
	MOV [R1], R0

	POP R10
	POP R6 
	POP R5 
	POP R4 
	POP R3
	POP R2 
	POP R1 
	POP R0 
	RET	

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Update_Nave_Ecra: Rotina que verifica se ouve mudanca na 
;	posicao da nave, e faz o draw respetivo 
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Update_Nave_Ecra:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4 
	PUSH R5
	PUSH R6

	;	Comparar a posicao atual com as antigas posicoes de desenho
	;	As variaveis sao movidas para os registos da seguinte forma:
	;	X atual: R1				Y atual: R2
	;	X antigo: R3			Y antigo: R4
	MOV R1, naveVar
	MOV R2, R1
	ADD R2, 2
	MOV R3, R2
	ADD R3, 2
	MOV R4, R3
	ADD R4, 2 
	MOV R1, [R1]
	MOV R2, [R2]
	MOV R3, [R3]
	MOV R4, [R4]
	;	Guardar as posicoes atuais para mais tarde
	MOV R5, R1
	MOV R6, R2 
	;*************************************************************
	
	CMP R1, R3	;	Caso sejam iguais, a nave nao mudou o x
	JEQ UNE1
	JMP UNE2

;	Verificar se houve mudanca de y
UNE1:
	CMP R2, R4
	JEQ UNE3
	JMP UNE2
	
;	Havendo mudanca de x ou y, apagar a nave antiga, e desenhar
UNE2:
	MOV R0, Nave
	MOV R1, naveVar
	MOV R10, 1
	CALL Desenhar_Objeto
	JMP OutUNE

;	Caso nao haja mudancas nas variaveis, nao se desenha, acabando a funcao

UNE3:	
;	Retornar Update_Nave_Ecra
OutUNE:
	MOV R1, R5 
	MOV R2, R6 
	MOV R3, naveVar		;	Mover para R3 o endereco do x antigo
	ADD R3, 04H			
	MOV R4, naveVar		;	Mover para R4 o endereco do y antigo
	ADD R4, 06H	
	;	Atualizar a var das ultimas posicoes de desenho com a variavel atual
	MOV [R3], R1
	MOV [R4], R2
	POP R6 
	POP R5 
	POP R4 
	POP R3
	POP R2 
	POP R1 
	POP R0 
	RET

	
	
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Gerador_Add - Aumenta o Gerador_ um valor 
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
geradorVar2: WORD 1
Gerador_Add:
; guardar valor dos registos a alterar
	PUSH R0				
	PUSH R1
	PUSH R2
	
	MOV R0, geradorVar1
	MOV R1, [R0]
	MOV R2, 1
	ADD R1, R2
	MOV [R0], R1
	MOV R0, geradorVar2
	MOV R1, [R0]
	MOV R2, 3
	ADD R1, R2
	MOV [R0], R1

; repor valor dos registos a alterar	
	POP R2
	POP R1				
	POP R0
	RET
	
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Gerador_ - Retorna um numero pseudoaleatorio
;	Args: R0- n -> retorna u numero entre 0-(n-1)
;	Out: R0 - Valor aleatorio
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Gerador_:
; guardar registos na stack
	PUSH R1
	PUSH R2
	PUSH R3
	
	MOV R1, geradorVar1
	MOV R2, geradorVar2
	MOV R1, [R1]
	MOV R2, [R2]
	XOR R1, R2
	MOV R2, 53
	MOD R1, R2
	MOD R1, R0
	MOV R0, R1

; repor registos
	CALL Gerador_Add
	POP R3
	POP R2
	POP R1			
	RET
	
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Input_ - Retorna o No da tecla premida,caso contrario 
;	retorna 0EFH
;	Args: Nenhum
;	Out: lastInput - Guarda o valor da ultima tecla premida
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Input_: 
	PUSH R0		; guardar valor dos registos a alterar
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
;	Inicializar registos
    MOV  R2, TECLIN   	; endereço das linhas
    MOV  R3, TECCOL   	; endereco das colunas
    MOV  R4, DISPLAYS  	; endereco dos displays
	MOV R1, LINHA      	; iniciar linha 0
	
;	Leitura
Read:
	MOVB [R2], R1	; escrever linha no periférico de entrada
	MOVB R0, [R3]	; ler coluna no periferico de saida
	CMP R0, 0H		   
	JEQ InputEQ0	   	; caso não haja input na linha salta
	JMP nLin
	
;	Processar linha e coluna para obter número da tecla
;	Começar por obter número de linha
;	Esta obtem-se através de contar o nº de shifts right necessarios
;	para R1(Entrada de perférico das linhas) se tornar 0
;	Ex: SHL 0001b, 1 => R4 = 0, logo a linha é 0, pois o bit menos
;	significativo das linhas é o direito

; Processar linha
nLin:
	MOV R4, 0			; número de vezes que se multiplicou a entrada dos periféricos das linhas por 2
nLinLoop:
	SHR R1, 1			; multiplicar a entrada do periférico das linhas por 2
	CMP R1, 0			
	JEQ nCol			; processar coluna se o valor corresponder a uma linha
	ADD R4, 1			; aumentar número de vezes que se multiplicou a entrada dos periféricos das linhas por 2
	JMP nLinLoop			; repete ciclo
	
;	Repetimos o processo usado em nLin para obter o Nº da coluna
; Processar coluna	
nCol:   
	MOV R5, 0       		; número de vezes que se multiplicou a entrada dos periféricos das colunas por 2

nColLoop:	
	SHR R0, 1			; multiplicar a entrada do periférico das colunas por 2
	CMP R0, 0			
	JEQ nKey	; obter número da tecla se o valor corresponder a uma coluna
	ADD R5,1	; aumentar número de vezes que se multiplicou a entrada dos periféricos das linhas por 2

	JMP nColLoop		; repete ciclo

;	Obter-se Nº da tecla, nKey = 4*nLin + nCol
nKey: 
	MOV R6, R4
	SHL R6, 2         ; R6 * 4 <=> SHR R6, 2
	ADD R6, R5
	JMP EndInput
	
;	Caso a coluna seja igual a 0
;	ler a próxima linha
InputEQ0:
	SHL R1, 1
	MOV R5, 10H		   ; 10H = linha 4 (nao existe) > linha 3 
	CMP R1, R5
	JEQ NoInput		   ; se linha(R1) > linha 3 e nao tem input salta
	JMP Read
	
	
;	Inexistencia de input
NoInput:
	MOV R6, 0EFH      ; sinalizar falta de input
	JMP EndInput	  ; retornar a funcao
	

;	Fim de input
;	Repor registos e retornar
EndInput:
	MOV R5, lastInput
	MOV [R5], R6			
	POP R6
	POP R5
	POP R4
	POP R3
	POP R2
	POP	R1
	POP	R0
	RET

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Tratar_Input: Funcao que trata o input obtido
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Tratar_Input:
	
;	Guardar registos
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3

;	Guardar a posicao x e y da Nave em R2, e R3 respetivamente
	MOV R0, naveVar			; tabela da posição e desenho/não desenho da nave
	MOV R0, [R0]				; ler posição x da nave
	MOV R1, naveVar			; tabela da posição e desenho/não desenho da nave
	ADD R1, 2			
	MOV R1, [R1]				; ler posição y da nave
	MOV R2, R0				; guardar posição x da nave
	MOV R3, R1				; guardar posição y da nave

;	Guardar a ultima tecla premida em R0
	MOV R0, lastInput			; última tecla premida
	MOV R0, [R0]				; guardar última tecla premida

;	Caso onde nao existe tecla premida -> 0EFH
	MOV R1, 0EFH			
	CMP R0, R1
	JEQ OutTratarInput			
	
;	Caso onde se ativa o escudo
	MOV R1, 05H
	CMP R0, R1				; ativar escudo se for premida a tecla 5
	JEQ TIUpdateEscudoNave
	
;	Caso onde se pausa o jogo
	MOV R1, 15
	CMP R0, R1				; pausar o jogo se for premida a tecla B
	JEQ TIPausa
	
	
;	Os nomes das etiquetas seguem o seguinte principio:
;	L- Left (sub x, 1)		R- Right (add x, 1)
;	U- Up (sub y, 1)	D-Down (add y, 1)
;	sendo x a posicao x da nave, e o mesmo para y.
;	Compara o input com as varias possiveis teclas que 
;	representam movimento.
;*******************************************
	MOV R1, 0H
	CMP R0, R1				; mover a nave para cima e para a esquerda se for premida a tecla 0
	JEQ LUMovNave
	MOV R1, 1H
	CMP R0, R1				; mover a nave para cima se for premida a tecla 1
	JEQ UMovNave
	MOV R1, 2H
	CMP R0, R1				; mover a nave para cima e para a direita se for premida a tecla 2
	JEQ RUMovNave
	MOV R1, 4H
	CMP R0, R1				; mover a nave para a esquerda se for premida a tecla 4
	JEQ LMovNave
	MOV R1, 6H			
	CMP R0, R1				; mover a nave para a direita se for premida a tecla 6
	JEQ RMovNave
	MOV R1, 8H
	CMP R0, R1				; mover a nave para baixo e para a esquerda se for premida a tecla 8
	JEQ LDMovNave
	MOV R1, 9H				
	CMP R0, R1				; mover a nave para baixo se for premida a tecla 9
	JEQ DMovNave
	MOV R1, 0AH				
	CMP R0, R1
	JEQ RDMovNave			; mover a nave para baixo e para a direita se for premida a tecla A

	JMP OutTratarInput
;**********************************************	
;	Caso onde se reinicia o jogo
	;MOV R1, 0FH
	;CMP R0, R1
	;JEQ 
	;JMP OutTratarInput
	

TIPausa:
	MOV R0, lastInput
	MOV R1, ultimoInputTratado
	MOV R0, [R0]				; ler valor do último input 
	MOV R1, [R1]				; ler valor do último input tratado
	CMP R0, R1
	JEQ OutTratarInput			; tratar do último input se ainda não o fez
	MOV R0, lastInput			
	MOV R0, [R0]				; ler valor do último input
	MOV R1, ultimoInputTratado		
	MOV [R1], R0				; atualizar último input trataro
	CALL Pausa				; pausar o jogo
	JMP OutTratarInput


;	Update do escudo 
;********************************************
TIUpdateEscudoNave:
	MOV R0, lastInput				
	MOV R1, ultimoInputTratado
	MOV R0, [R0]					; ler valor da última tecla premida
	MOV R1, [R1]					; ler valor do último input tratado
	CMP R0, R1
	JEQ OutTratarInput				; terminar ciclo se o input tiver sido tratado
	MOV R0, naveEscudo
	MOV R1, [R0]					; ler valor do escudo (ativo/inativo)
	NEG R1					; alterar estado de ativação do escudo
	MOV [R0], R1					; atualizar estado de ativação do escudo
	MOV R0, PSSOM
	MOV R1, 2		; som do escudo
	MOV [R0], R1
	JMP OutTratarInput

;	Update da variavel que contem o x e y da nave
;************************************
LUMovNave:
	SUB R2, 1				; nave retrocede uma coluna
	SUB R3, 1				; nave sobe uma linha
	JMP TISomMov
UMovNave:
	SUB R3, 1				; nave desce uma linha				
	JMP TISomMov 
RUMovNave:
	ADD R2, 1				; nave avança uma coluna
	SUB R3, 1				; nave sobe uma linha
	JMP TISomMov
LMovNave:
	SUB R2, 1				; nave retrocede uma coluna
	JMP TISomMov			
RMovNave:
	ADD R2, 1				; nave avança uma coluna
	JMP TISomMov
LDMovNave:
	SUB R2, 1					; nave retrocede uma coluna
	ADD R3, 1					; nave desce uma linha
	JMP TISomMov
DMovNave:	
	ADD R3, 1					; nave desce uma linha
	JMP TISomMov
RDMovNave:
	ADD R2, 1					; nave retrocede uma coluna
	ADD R3, 1					; nave desce uma linha
	JMP TISomMov
;************************************
;	Tocar som  do movimento
;************************************
TISomMov:
	MOV R0, lastInput
	MOV R1, ultimoInputTratado
	MOV R0, [R0]					; ler valor da última tecla premida
	MOV R1, [R1]					; ler valor do último input tratado
	CMP R0, R1					
	JEQ TIValPos			
	MOV R0, PSSOM				; número do som a utilizar		
	MOV R1, somMovimento
	MOV R1, [R1]					; ler som do movimento
	MOV [R0], R1					; tocar som do movimento
	JMP TIValPos
	
;***************************************************
;	Verificar se a posicao da nave e dentro do ecra
;***************************************************
TIValPos:
	MOV R0, 0 
	CMP R2, 0
	JLT TIVP1
	CMP R3, R0
	JLT TIVP2
	MOV R0, 61
	CMP R2, R0
	JGT TIVP3
	MOV R0, 29
	CMP R3, R0
	JGT	TIVP4
	JMP OutTratarInput

TIVP1:
	MOV R2, 0
	JMP TIValPos
TIVP2:
	MOV R3, 0
	JMP TIValPos
TIVP3:
	MOV R2, 61
	JMP TIValPos
TIVP4:
	MOV R3, 29
	JMP TIValPos
	
	
;************************************

OutTratarInput:
;	Guardar as novas posicoes nas variaveis
	MOV R0, naveVar
	MOV R1, naveVar
	ADD R1, 2					; passar para posição y
	MOV [R0], R2					; gravar posição x atual
	MOV [R1], R3					; gravar posição y atual
	MOV R0, lastInput
	MOV R0, [R0]
	MOV R1, ultimoInputTratado
	MOV [R1], R0					; atualizar último input tratado
	;Repor registos e retornar
	POP R3
	POP R2
	POP R1
	POP R0
	RET




;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Piscar_Pulsares: Rotina que da update aos pulsares 
;	e redesenha-os
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
TIPO_PULSAR EQU 0AH
Piscar_Pulsares:
; Gravar valor dos registos a alterar
	PUSH R0				
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R10

; Loop que itera pelos raios e redesenha com opacidade diferente
	MOV R0, 3
LoopPP1:
	CMP R0, 0
	JLT OutLoopPP1
	PUSH R0
	
	MOV R1, pulsares
	MOV R2, distPulsaresVar
	MOV R2, [R2]
	MUL R2, R0
	ADD R1, R2
	MOV R3, R1
	MOV R2, TIPO_PULSAR		; mover para R1 o tipo do pulsar (bom/mau)
	ADD R1, R2
	MOV R0, 13	
	;CALL Gerador_Add
	CALL Gerador_
	MOV [R1], R0
	CMP R0, 2	; Verificar que tipo de pulsar é
	JLE PPBom
	JMP PPMau

PPMau:
	MOV R2, estadoPulsaçao
	MOV R2, [R2]
	CMP R2, 1
	JEQ PPMauOpaco	
	JMP PPMauSemi
	
PPMauOpaco:
	MOV R10, 1
	MOV R0, PulsarMau
	MOV R1, R3
	CALL Desenhar_Objeto
	JMP EndLoopPP1
PPMauSemi:	
	MOV R10, 1
	MOV R0, PulsarMauSemiVis
	MOV R1, R3
	CALL Desenhar_Objeto
	JMP EndLoopPP1

PPBom:	
	MOV R2, estadoPulsaçao
	MOV R2, [R2]
	CMP R2, 1
	JEQ PPBomOpaco
	JMP PPBomSemi

PPBomOpaco:
	MOV R10, 1
	MOV R0, PulsarBom
	MOV R1, R3
	CALL Desenhar_Objeto
	JMP EndLoopPP1
PPBomSemi:	
	MOV R10, 1
	MOV R0, PulsarBomSemiVis
	MOV R1, R3
	CALL Desenhar_Objeto
	JMP EndLoopPP1	
	
EndLoopPP1:
	POP R0
	SUB R0, 1
	JMP LoopPP1
	
OutLoopPP1:
	MOV R0, estadoPulsaçao
	MOV R1, [R0]
	NEG R1
	MOV [R0], R1
OutPiscar_Pulsares:
; repor valor dos registos alterados
	POP R10	
	POP R3
	POP R2
	POP R1
	POP R0
	RET

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Update_Movimento_Raios: Rotina que da update a posicao dos raios
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
flagCriarRaios: WORD -1
Update_Movimento_Raios:
;	Guardar registos
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	
; Loop que itera os varios raios
	MOV R0, 3
LoopUMR1:
	CMP R0, 0
	JLT OutLoopUMR1
	PUSH R0
;----------------------------------------	
	MOV R1, raios	; base dos raios
	MOV R2, distRaio
	MOV R2, [R2]
	MUL R2, R0		; calcular a base correta para o raio em questao
	MOV R3,R2		; guardar base do raio em R3
	MOV R0, distDirecaoRaio
	MOV R0, [R0]
	ADD R2, R0		; mover o ponteiro para a direcao x do raio
	MOV R0, [R2]	; R0 fica com direcao x
	ADD R2, 2
	MOV R1, [R2]	; R1 fica com direcao y
	MOV R2, R3		; repor a base em R2
	MOV R3, [R2]	; obter posicao x do raio
	ADD R3, R0		; alterar posicao x de acordo com direcao x
	CMP R3, 0
	JLT UMR_MudarFlag
	CMP R3, 64
	JGE UMR_MudarFlag
	MOV [R2], R3	; guardar o valor na variavel
	ADD R2, 2
	MOV R3, [R2]	; obter posicao y do raio
	ADD R3, R1		; alterar posicao y de acordo com direcao y
	CMP R3, 0
	JLT UMR_MudarFlag
	CMP R3, 32
	JGE UMR_MudarFlag
	CALL UMR_ForaEcra
	MOV [R2], R3	; guardar o valor na variavel
;---------------------------------------	
	POP R0
	SUB R0, 1
	JMP LoopUMR1
	
UMR_MudarFlag:
		
	MOV R1, flagCriarRaios
	MOV [R1], 1

OutLoopUMR1:

	CALL Detecao_Colisao
OutUpdate_Movimento_Raios:
;	Repor registos
	POP R3
	POP R2
	POP R1
	POP R0
	RET

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Detecao_Colisao: Rotina que verifica se houve colisao dos raios
;	NoRaioColisao == 0 -> nao houve colisao
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Detecao_Colisao:
;	Guardar registos
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
	PUSH R7
; Loop que itera os varios raios
	MOV R0, 3
LoopDC1:
	CMP R0, 0
	JLT OutLoopDC1
	
	MOV R1, raios	; base dos raios
	MOV R2, distRaio
	MOV R2, [R2]
	MUL R2, R0		; calcular a base correta para o raio em questao
	MOV R1, R2		; guardar base do raio em R1
	MOV R2, [R2]	; guardar pos x do raio
	MOV R4, naveVar
	MOV R4, [R4]	; guardar pos x da nave
	MOV R6, Nave
	MOV R7, LARGURA_X_NAVE
	ADD R6, R7	; localizacao de largura x na base do objeto
	MOV R6, [R6]	; guardar a largura x da nave
	
; Verificar se o raios se encontra na faixa da largura da nave
	MOV R3, R4
	MOV R5, R2
	SUB R5, R3		; se o x do raio for menor que a nave, o numero da negativo
	CMP R5, 0
	JLT EndLoopDC1	; verificamos entao se o valor e menor que o valor x max da nave
	MOV R3, R4
	ADD R3, R6
	SUB R3, 1 		; para compensar o facto de posx da nave ser ja um pixel da nave
	MOV R5, R2
	SUB R5, R3
	CMP R5, 0
	JGT EndLoopDC1	; caso o valor seja superior a 0, posx raio esta para la de posx da nave
;-------------------------------------------------------------
; Preparar argumentos denovo
	MOV R2, R1
	ADD R2, 2 	
	MOV R2, [R2]	; posy do raio
	MOV R4, naveVar
	ADD R4, 2
	MOV R4, [R4]	; pos y da nave
	MOV R6, Nave
	MOV R7, COMPRIMENTO_Y_NAVE 
	ADD R6, R7		; localizacao de comprimento y na base do objeto
	MOV R6, [R6]
	
; Verificar se o raio se encontra na faixa de comprimento da nave
	MOV R3, R4
	MOV R5, R2
	SUB R5, R3		; se o y do raio for menor que a nave, o numero da negativo
	CMP R5, 0
	JLT EndLoopDC1	; verificamos entao se o valor e menor que o valor y max da nave
	MOV R3, R4
	ADD R3, R6
	SUB R3, 1 		; para compensar o facto de posy da nave ser ja um pixel da nave
	MOV R5, R2
	SUB R5, R3
	CMP R5, 0
	JGT EndLoopDC1	; caso o valor seja superior a 0, posx raio esta para la de posy da nave
;---------houve colisao---------------------
	MOV R3, NoRaioColisao
	MOV [R3], R0		; guardamos o valor do raio que colidiu para tratamento
	CALL Tratar_Colisao
	JMP OutDetecao_Colisao
	
EndLoopDC1:
	SUB R0, 1
	JMP LoopDC1
	
OutLoopDC1:
	MOV R3, NoRaioColisao
	MOV R4, 0
	MOV [R3], R4			; nao havendo colisao move-se 0 para a variavel, indicando
						; assim que nao ocorreu colisao
OutDetecao_Colisao:

;	Repor registos
	POP R7
	POP R6
	POP R5
	POP R4
	POP R3
	POP R2
	POP R1
	POP R0
	RET
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Tratar_Colisao: Rotina que faz alteracoes ao estado de jogo
;	devido á colisao de acordo com o estado das variaveis no momento
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Tratar_Colisao:
;	Guardar registos
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	
	MOV R0, NoRaioColisao
	MOV R0, [R0]
	CMP R0, 0			; se o valor for zero, nao houve colisao(ver funcao de detecao)
	JEQ OutTratar_Colisao	
;	bloco que altera as variaveis para o raio nao ser desenhado até á proxima onda
	MOV R1, raios
	MOV R2, distRaio
	MOV R2, [R2]
	MUL R2, R0
	ADD R1, R2
	MOV R3, R1
	MOV R4, VAR_DESENHAR_RAIO
	ADD R1, R4
	MOV R2, [R1]
	NEG R2
	MOV [R1], R2
;-----------------	
	MOV R1, R3
	MOV R4, LOC_TIPO_RAIO
	ADD R1, R4	; mover para R1 o tipo de raio (bom/mau)
	MOV R1, [R1]
	MOV R4, TIPO_RAIO_MAU
	CMP R1, R4	; se o raio for mau, saltar para a label correta
	JEQ TratarRaioMau
	JMP TratarRaioBom
	
TratarRaioBom:
	MOV R1, naveEscudo
	MOV R1, [R1]
	CMP R1, 1
	JEQ OutTratar_Colisao

	PUSH R0
	MOV R0, ENERGIA_RAIOBOM_S_ESCUDO
	CALL Update_Energia
	POP R0
	JMP OutTratar_Colisao
; Para o caso em que o escudo está ativado, a energia nao varia
TratarRaioMau:
	MOV R1, naveEscudo
	MOV R1, [R1]
	CMP R1, 1
	JEQ TratarRaioMauCEscudo
	JMP TratarRaioMauSEscudo
	
TratarRaioMauSEscudo:
	PUSH R0
	MOV R0, ENERGIA_RAIOMAU_S_ESCUDO
	CALL Update_Energia
	POP R0
	JMP OutTratar_Colisao

TratarRaioMauCEscudo:
	PUSH R0
	MOV R0, ENERGIA_RAIOMAU_C_ESCUDO
	CALL Update_Energia
	POP R0
	JMP OutTratar_Colisao


OutTratar_Colisao:
;	Repor registos
	POP R4
	POP R3
	POP R2
	POP R1
	POP R0
	RET



	
;############################################################
;	Desenho (Render)
;############################################################

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Loop_BG: Esta funcao itera o gif
;	Args: Nenhum
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Loop_BG:
;	Salvar registos
	PUSH R0
	PUSH R1
;	Mover noCena antigo e adicionar 1, para iterar o gif
	MOV R0, noCena
	MOV R0, [R0]					; ler número de imagem anterior
	ADD R0, 1					; passar para imagem seguinte
;	Caso o noCena seja maior ao no de imagens do gif
;	retornar para o inico do gif
	CMP R0, GIFFIM				; retornar ao início do gif caso não haja imagem seguinte
	JGT ResetGif
	JMP RetGif
ResetGif:
	MOV R0, GIFINICIO				; retornar ao início do gif
	JMP RetGif
;	Mover o valor da cena para a variavel e chamar Background
;	para atualizar a cena, e de seguida retornar
RetGif:
	MOV R1, noCena				;
	MOV [R1], R0					; escrever valor da imagem do gif
	POP R1					; repor valor dos registos alterados
	POP R0
	CALL Background_
	RET
;******************************************************************	

;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
;	Background:	Rotina que altera o fundo do pixel screen
;	Args: noCena - No da cena a definir
;	Out: Nenhum
;$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Background_:
;	Salvar registos
	PUSH R0
	PUSH R1
;	Mover noCena para um registo
	MOV R0, noCena
	MOV R0, [R0]
;	Calcular endereco para especificar a cena por comando
	MOV R1, PSCENA
;	Mover No de cena para a memória do comando
	MOV [R1], R0
;	Restaurar registos
	POP R1
	POP R0
	RET

; **********************************************************************
; Apaga_Objeto - Rotina que apaga um objeto na linha e coluna
; do ultimo desenho, no PixelScreen (PS)
; Argumentos:   R0- Endereco do template
;				R1- Endereco das variaveis do objeto
;***********************************************************************
Apagar_Objeto:
	PUSH R0				; guardar registos
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	PUSH R5
	MOV R3, distUltimaPosx
	MOV R3, [R3]				; ler distância da base da variável até à última posição x de desenho
	ADD R1, R3				; 
	MOV R2, R1				; 
	ADD R2, 2
	MOV R1, [R1]
	MOV R2, [R2]
	MOV R3, distVetorForma
	MOV R3, [R3]
	ADD R0, R3
	MOV R5, 0

AOCallDesLin:
	CALL Desenhar_Linha
	ADD R0, 2		; Incrementar a endereco do vetor/ 'linha' da forma
	MOV R3, [R0]	; Comparar o valor da linha de forma com '-1', caso
	MOV R4, -1
	CMP R3, R4		; seja igual a -1, significa que o objeto terminou
	JEQ OutApagaObjeto ; logo, retorna-se da funçao
	ADD R2, 1		; Passa para a proxima linha do vetor/ forma 
	CMP R1, 0
	JLT OutApagaObjeto
	CMP R2, 0
	JLT OutApagaObjeto
	
	JMP DOCallDesLin
	
OutApagaObjeto:
	POP R5
	POP R4
	POP R3
	POP R2
	POP R1
	POP R0
	RET
; *********
	
; **********************************************************************
; Desenhar_Objeto - Rotina que escreve um objeto na linha e coluna
; recentes, no PixelScreen (PS)
; Argumentos:   R0- Endereco do template
;				R1- Endereco das variaveis do objeto
;				R10- 0 = desenha objeto
;					0 != apaga desenho antigo e desenha novo
;***********************************************************************
Desenhar_Objeto:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
	PUSH R7
	PUSH R10
	
;	Guardar R0 e R1 em R6 e R7 como backup dos registos
	MOV R6, R0
	MOV R7, R1
	
;	Comparacao que define se o programa apaga o desenho antigo ou nao
	CMP R10, 0 
	JNE DOApagarPrepArgs
	JMP DONovo
	
DONovo:
	MOV R0, R6
	; Define componente vermelha do objeto	
	MOV R4, [R0]	; Guardar componente vermelha em R0
	MOV R3, PSBASE	; Guardar em R3 o endereco da componente Red do PS
	MOV [R3], R4	; Passa a componente por comando para o PS
	ADD R0, 2		; Passar para endereco da componente verde
	MOV R4, [R0]	; (...)
	MOV R3, GREEN
	MOV [R3], R4
	ADD R0, 2		
	MOV R4, [R0]		
	MOV R3, BLUE
	MOV [R3], R4
	ADD R0, 2		; (...)
	MOV R4, [R0]	; Guardar componente alpha
	MOV R3, ALPHA
	MOV [R3], R4
	ADD R0, 6		; Passar para endereco inicial do vetor de forma
	JMP DONovoPrepArgs
	
	
;	Prepara os argumentos para chamar Desenha_Linha ( Ver declaracao da rotina)
DONovoPrepArgs:
	MOV R1, R7	; restaurar o endereco das variaveis/ pos x
	MOV R2, R1
	ADD R2, 2	; mover endereco pos y para R2
	MOV R1, [R1]	; mover os valores reais para o registo
	MOV R2, [R2]
	MOV R10, 0
	MOV R5, 1
	JMP DOCallDesLin
	
	
;	Passar para R1 e R2 as posicoes do ultimo desenho, e move o valor 0 para R5, o que
;	significa que Desenhar_Linha ira desligar os pixels correspondentes ao desenho na
;	posicao antiga
DOApagarPrepArgs:
	MOV R0, R6 
	MOV R1, R7
	MOV R3, distUltimaPosx
	MOV R3, [R3]
	ADD R1, R3
	MOV R2, R1
	ADD R2, 2
	MOV R1, [R1]
	MOV R2, [R2]
	MOV R3, distVetorForma
	MOV R3, [R3]
	ADD R0, R3
	MOV R5, 0
	JMP DOCallDesLin
	
	
	
	
; Este loop itera as linhas do vetor, e enquanto nao forem
; iguais a '-1', desenha a respetiva representacao em bits
DOCallDesLin:
	CALL Desenhar_Linha
	ADD R0, 2		; Incrementar a endereco do vetor/ 'linha' da forma
	MOV R3, [R0]	; Comparar o valor da linha de forma com '-1', caso
	MOV R4, -1
	CMP R3, R4		; seja igual a -1, significa que o objeto terminou
	JEQ OutEscreveObjeto ; logo, retorna-se da funçao
	ADD R2, 1		; Passa para a proxima linha do vetor/ forma 
	
	CMP R1, 0
	JLT OutEscreveObjeto
	CMP R2, 0
	JLT OutEscreveObjeto
	JMP DOCallDesLin
	
	
	
	
OutEscreveObjeto:
;	Como se salta o desenho de novo quando o argumento R10 e 0, salta-se para DONovo, que desenhara objeto e tornara o valor de R10 em 0, permitindo retornar a funcao
	CMP R10, 1
	JEQ DONovo
	
;*********************************

	POP R10
	POP R7
	POP R6
	POP R5
	POP R4
	POP R3
	POP R2
	POP R1
	POP R0
	RET
	

; **********************************************************************
; Desenhar_Linha - Rotina que desenha uma linha do template
; Argumentos:   R0- Endereco do vetor forma
;				R1- x
;				R2- y
;				R5- 0 = apaga, 1 = desenha
;***********************************************************************
Desenhar_Linha:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
;	Caso x e y sejam inferiores a 0, o objeto encontra-se fora do ecra, logo nao se desenha
	CMP R1, 0
	JLT OutDesLinha
	CMP R2, 0
	JLT OutDesLinha
	; Comparar a linha do vetor com '0', caso seja igual, significa que a linha
	; nao representa nenhum pixel, retornando-se assim a funcao
	MOV R3, [R0]
	CMP R3, 0
	JEQ OutDesLinha
	
; Este loop dá SHL á linha, caso a operacao de carry, significa que se deve 
; desenhar um pixel, caso contrario incrementa-se a coluna e repete-se o loop
DLLoop1:
	SHL R3, 1
	JC DLLoop2
	ADD R1, 1
	CMP R3, 0	; Caso a linha seja igual a '0', desenhou-se todos os pixeis
	JEQ OutDesLinha	; representados na linha, retornando-se a funcao
	JMP DLLoop1
	
; Este loop desenha o pixel na coluna e linha respetiva
DLLoop2:
	; Definir coluna e linha
	MOV R4, PSCOL
	MOV [R4], R1
	MOV R4, PSLIN
	MOV [R4], R2
;**************************************
	; 'Ligar' o pixel
	MOV R4, IOPIXEL
	MOV [R4], R5	
	; Incrementar a coluna (x)
	ADD R1, 1
	JMP DLLoop1
	
;	Restaurar registos e retornar
OutDesLinha:
	POP R6
	POP R5
	POP R4
	POP R3
	POP R2
	POP R1
	POP R0
	RET
	
; **********************************************************************
; Mudar_Cor_Nave - Rotina que altera a cor da nave
; Argumentos: Nenhum
; Out: Nenhum
;***********************************************************************

Mudar_Cor_Nave:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R3
	
	
	MOV R0, numCor
	MOV R0, [R0]	; guardar em R0 o numero da cor
	MOV R3, distCor
	MOV R3, [R3]	; guardar em R4 a distancia entre as bases da cor
	MOV R1, R3		; multiplicar o numCor pela distancia, para obter a base correta
	MUL R0, R1		; multiplicar o numero da cor pela distancia entre bases
	MOV R1, Cor0
	ADD R1, R0		; mover para R1 a base correta da cor pretendida
	MOV R0, R1		; iterar pelas componentes RBG da cor pretendida
	ADD R1, 2; 
	MOV R2, R1
	ADD R2, 2;**********************************************
;	mover para os registos os valores das componentes
	MOV R0, [R0]
	MOV R1, [R1]
	MOV R2, [R2]

;	guardar as componentes rgb no template da nave
	MOV R3, Nave
	MOV [R3], R0
	ADD R3, 2
	MOV [R3], R1
	ADD R3, 2
	MOV [R3], R2
	
;	incrementar a cor, caso o numCor seja maior que o no maximo de cores
;	definir  numCor como a cor de indice 0
	MOV R0, numCor
	MOV R0, [R0]
	ADD R0, 1
	CMP R0, 7
	JEQ	MCN2	
	
;	guardar o novo numCor na variavel respetiva
MCN1: 
	MOV R1, numCor
	MOV [R1], R0
	JMP OutMudarCorNave
;	tornar o numCor igual a 0
MCN2:
	MOV R0, 0
	JMP MCN1
	
;	retornar da funcao
OutMudarCorNave:
	POP R3
	POP R2
	POP R1
	POP R0
	RET
	

	
	
; **********************************************************************
; Reset_Ecra - Rotina que limpa o ecra
;***********************************************************************
Reset_Ecra:
	PUSH R0
	PUSH R1
	MOV R0, PSCLEAR	; comando do Pixel screen para apagar o ecra
	MOV R1, 1
	MOV [R0], R1
	POP R1
	POP R0
	RET
	
	
; **********************************************************************
; Display_Em_DEC - Esta funcao da update aos displays de 7 segmentos
; Argumentos: R0- Numero a por nos displays
; Out: Nenhum
;***********************************************************************
Display_Em_DEC:
	PUSH R0	;entrada
	PUSH R1 ; 1o algarismo
	PUSH R2 ; 2o algarismo
	PUSH R3 ; 3o algarismo
	PUSH R4
;	Usar o modulo por 10 para obter o valor dos digitos decimais 
;	do valor guardado em hexadecimal
	MOV R4, 10
	MOV R1, R0
	MOD R1, R4
	DIV R0, R4	; nao esquecer que se tem que dividir por 10
			; para fazer ?shift? do algarismo das unidades para fora
			; Ex: 123 -> 12
	MOV R2, R0
	MOD R2, R4
	DIV R0, R4
	MOV R3, R0
	MOD R3, R4
	DIV R0, R4
	
;	Adicionar os digitos do maior para o menor a R1, adicionando o digito a R1, 
;	fazendo um SHL de 4 bits para preservar a ?representacao decimal? do digito
; 	que tem a forma de um numero hexadecimal, repetindo para os varios digitos
	MOV R4, 0
	ADD R4, R3
	SHL R4, 4
	ADD R4, R2
	SHL R4, 4
	ADD R4, R1
;	Mover o registo que contem as representacoes decimais para o display
	MOV R1, DISPLAYS
	MOV [R1] , R4
;	Restaurar registos e retornar
	POP R4 
	POP R3
	POP R2
	POP R1
	POP R0
	RET
