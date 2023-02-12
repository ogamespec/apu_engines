# Bomberman APU Engine

As an example of APU programming, here are excerpts from the source code of the game Bomberman.

All the source code of the game can be found here: https://github.com/emu-russia/bomberman-nes

## Engine

```
APU_STOP:               
        LDA #0
        STA APU_MUSIC

APU_ABORT:              
        RTS

; =============== S U B R O U T I N E =======================================

; Play a tune

APU_PLAY_MELODY:            
        LDA APU_DISABLE
        BNE APU_ABORT
        LDA APU_MUSIC
        BEQ APU_ABORT
        BMI UPDATE_MELODY
        CMP #$B
        BCS APU_STOP
        STA APU_TEMP
        ORA #$80
        STA APU_MUSIC
        DEC APU_TEMP
        LDA APU_TEMP
        ASL
        ASL
        ASL
        TAY
        LDX #0

START_MELODY:               
        LDA APU_MELODIES_TAB,Y ; 1: TITLE
        STA APU_CHANDAT,X
        INY
        INX
        CPX #6
        BNE START_MELODY
        LDA APU_MELODIES_TAB,Y ; 1: TITLE
        STA byte_D3
        LDA APU_MELODIES_TAB+1,Y ; 1: TITLE
        STA byte_D4
        LDA #0
        STA APU_CNT
        STA APU_CNT+1
        STA APU_CNT+2
        STA byte_D5
        STA byte_CD
        STA byte_CE
        STA byte_CF
        STA byte_D0
        STA byte_D1
        STA byte_D2
        LDA #1
        STA byte_B6
        STA byte_B6+1
        STA byte_B6+2
        STA byte_B9
        STA byte_BA
        STA byte_BB
        STA byte_D6
        STA byte_D7
        STA byte_D8
        LDA #8
        STA byte_D9
        STA byte_DA

UPDATE_MELODY:              
        LDA #2
        STA APU_CHAN

NEXT_CHANNEL:               
        LDX APU_CHAN
        DEC $B6,X
        BEQ PLAY_CHANNEL

ADVANCE_CHANNEL:            
        DEC APU_CHAN
        BPL NEXT_CHANNEL
        RTS
; ---------------------------------------------------------------------------

PLAY_CHANNEL:               
        TXA
        ASL
        TAX
        LDA APU_CHANDAT,X
        STA APU_PTR
        LDA APU_CHANDAT+1,X
        STA APU_PTR+1
        ORA APU_PTR
        BEQ ADVANCE_CHANNEL
        JSR APU_WRITE_REGS
        JMP ADVANCE_CHANNEL


; =============== S U B R O U T I N E =======================================


APU_WRITE_REGS:             
        LDX APU_CHAN
        LDY APU_CNT,X
        LDA (APU_PTR),Y
        STA APU_TEMP
        INC APU_CNT,X
        LDA APU_TEMP
        BMI CONTROL_BYTE
        LDA byte_B9,X
        STA byte_B6,X
        CPX #2
        BEQ FIX_TRIANGLE
        LSR
        LSR
        CMP #$10
        BCC FIX_DELAY
        LDA #$F
        BNE FIX_DELAY

FIX_TRIANGLE:               
        ASL
        BPL FIX_DELAY
        LDA #$7F

FIX_DELAY:              
        STA byte_D6,X
        LDA byte_D0,X
        BEQ loc_E57B
        LSR byte_D6,X

loc_E57B:               
        LDY APU_CHAN_DIS,X
        BNE ABORT_WRITE
        LDA APU_TEMP
        CMP #0
        BEQ ABORT_WRITE
        TXA
        ASL
        ASL
        TAY
        LDA byte_CD,X
        BEQ loc_E59D
        BPL loc_E593
        INC byte_CD,X
        BEQ loc_E59D

loc_E593:               
        LDA #$9F
        CPX #2
        BNE loc_E5A1
        LDA #$7F
        BNE loc_E5A1

loc_E59D:               
        LDA byte_D6,X
        ORA byte_D3,X

loc_E5A1:               
        STA $4000,Y
        LDA byte_CD,X
        CMP #2
        BNE loc_E5AB
        RTS
; ---------------------------------------------------------------------------

loc_E5AB:               
        CPX #2
        BCS SET_WAVELEN
        LDA byte_D9,X
        STA $4001,Y

SET_WAVELEN:                
        LDA APU_TEMP
        ASL
        TAX
        LDA WAVELEN_TAB,X
        STA $4002,Y
        LDA WAVELEN_TAB+1,X
        ORA #8
        STA $4003,Y

ABORT_WRITE:                
        RTS
; ---------------------------------------------------------------------------

CONTROL_BYTE:               
        AND #$F0
        CMP #$F0
        BEQ EXEC_EFFECT
        LDA APU_TEMP
        AND #$7F
        STA byte_B9,X
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------

EXEC_EFFECT:                
        SEC
        LDA #$FF
        SBC APU_TEMP
        ASL
        TAY
        LDA off_E5E6+1,Y
        PHA
        LDA off_E5E6,Y
        PHA
        RTS

; ---------------------------------------------------------------------------
off_E5E6:   .WORD off_E5F4+1    
        .WORD locret_E5FA
        .WORD loc_E5FF+2
        .WORD loc_E60D+2
        .WORD loc_E618+2
        .WORD loc_E62A+2
        .WORD loc_E631+2
off_E5F4:   .WORD loc_E638+2    
; ---------------------------------------------------------------------------
        LDA #0
        STA APU_MUSIC

locret_E5FA:                
        RTS
; ---------------------------------------------------------------------------
        LDA #0
        STA APU_CNT,X

loc_E5FF:               
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------
        LDY APU_CNT,X
        LDA (APU_PTR),Y
        STA unk_CA,X
        INY
        STY APU_CNT,X
        STY unk_C7,X

loc_E60D:               
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------
        DEC unk_CA,X
        BEQ loc_E618
        LDA unk_C7,X
        STA APU_CNT,X

loc_E618:               
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------
        LDA byte_CD,X
        BEQ loc_E626
        LDA #2
        STA byte_CD,X
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------

loc_E626:               
        LDA #1
        STA byte_CD,X

loc_E62A:               
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------
        LDA #$FF
        STA byte_CD,X

loc_E631:               
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------
        LDA #$FF
        STA byte_D0,X

loc_E638:               
        JMP APU_WRITE_REGS
; ---------------------------------------------------------------------------
        LDA #0
        STA byte_D0,X

loc_E63F:
        JMP APU_WRITE_REGS

; =============== S U B R O U T I N E =======================================

; Reset APU state

APU_RESET:              
        LDA #0
        STA APU_DELTA_REG+1
        STA APU_CHAN_DIS
        STA APU_CHAN_DIS+1
        STA APU_CHAN_DIS+2
        STA APU_SQUARE1_REG
        STA APU_SQUARE2_REG
        STA APU_TRIANGLE_REG
        STA APU_NOISE_REG
        LDA #$F
        STA APU_MASTERCTRL_REG
        RTS


; =============== S U B R O U T I N E =======================================

; Play the sound

APU_PLAY_SOUND:             
        LDX #2

MUTE_CHANNEL:               
        LDA APU_CHAN_DIS,X
        BEQ MUTE_NEXT_CHAN
        DEC APU_CHAN_DIS,X

MUTE_NEXT_CHAN:             
        DEX
        BPL MUTE_CHANNEL
        LDA APU_SOUND   ; Play the sound
        BMI UPDATE_SOUND
        CMP #7
        BCS WRONG_SOUND ; >= 7
        CMP #3
        BCS START_SOUND ; >= 3
        LDX APU_PATTERN
        BEQ START_SOUND
        TXA
        ORA #$80
        STA APU_SOUND   ; Play the sound
        BNE UPDATE_SOUND

START_SOUND:                
        STA APU_PATTERN
        ORA #$80
        STA APU_SOUND   ; Play the sound
        LDA #0
        STA APU_SOUND_MOD
        STA APU_SOUND_MOD+1
        STA APU_SOUND_MOD+2
        LDA APU_PATTERN
        ASL
        TAX
        LDA MOD_SOUND_TAB+1,X
        PHA
        LDA MOD_SOUND_TAB,X
        PHA
        RTS
; ---------------------------------------------------------------------------

UPDATE_SOUND:               
        LDA APU_PATTERN
        CMP #7
        BCS WRONG_SOUND
        ASL
        TAX
        LDA CONST_SOUND_TAB+1,X
        PHA
        LDA CONST_SOUND_TAB,X
        PHA

WRONG_SOUND:                
        RTS

; ---------------------------------------------------------------------------
MOD_SOUND_TAB:  
        .WORD APU_RESET-1   ; Reset APU state
        .WORD S1_START-1
        .WORD S2_START-1
        .WORD S3_START-1
        .WORD S4_START-1
        .WORD S5_START-1
        .WORD S6_START-1
CONST_SOUND_TAB:
        .WORD WRONG_SOUND-1 
        .WORD WRONG_SOUND-1
        .WORD WRONG_SOUND-1
        .WORD S3_UPDATE-1
        .WORD S4_UPDATE-1
        .WORD WRONG_SOUND-1
        .WORD S6_UPDATE-1
; ---------------------------------------------------------------------------

S1_START:               
        LDA #4
        BNE loc_E6CF

S2_START:               
        LDA #$C

loc_E6CF:               
        STA APU_NOISE_REG+2
        LDA #0
        STA APU_PATTERN
        STA APU_NOISE_REG
        LDA #$10
        STA APU_NOISE_REG+3
        RTS
; ---------------------------------------------------------------------------

S3_START:               
        LDA #$10
        STA APU_SOUND_MOD+1
        LDA #1
        STA APU_NOISE_REG
        LDA #$F
        STA APU_NOISE_REG+2
        LDA #$10
        STA APU_NOISE_REG+3
        LDA #$FF
        STA APU_SQUARE2_REG
        LDA #$84
        STA APU_SQUARE2_REG+1
        LDA #0
        STA APU_SQUARE2_REG+2
        LDA #2
        STA APU_SQUARE2_REG+3
        LDA #4
        STA APU_SDELAY
        RTS
; ---------------------------------------------------------------------------

S3_UPDATE:              
        DEC APU_SDELAY
        BNE locret_E727
        LDA #$DF
        STA APU_SQUARE2_REG
        LDA #$84
        STA APU_SQUARE2_REG+1
        LDA #0
        STA APU_SQUARE2_REG+2
        LDA #$81
        STA APU_SQUARE2_REG+3
        LDA #0
        STA APU_PATTERN

locret_E727:                
        RTS
; ---------------------------------------------------------------------------

S4_START:               
        LDA #$FF
        STA APU_SOUND_MOD+1
        LDA #0
        STA APU_SDELAY
        LDA #4
        STA APU_SDELAY+1

S4_UPDATE:              
        LDA APU_SDELAY
        BNE S4_PITCH1
        LDA APU_SDELAY+1
        BNE S4_PITCH2
        LDA #0
        STA APU_PATTERN
        STA APU_SOUND_MOD+1
        RTS
; ---------------------------------------------------------------------------

S4_PITCH2:              
        DEC APU_SDELAY+1
        LDA #$84
        STA APU_SQUARE2_REG
        LDA #$8B
        STA APU_SQUARE2_REG+1
        LDX APU_SDELAY+1
        LDA S4_PITCH_TAB,X
        STA APU_SQUARE2_REG+2
        LDA #$10
        STA APU_SQUARE2_REG+3
        LDA #4
        STA APU_SDELAY

S4_PITCH1:              
        DEC APU_SDELAY
        RTS
; ---------------------------------------------------------------------------
S4_PITCH_TAB:   .BYTE $65,$87,$B4,$F0   
; ---------------------------------------------------------------------------

S5_START:               
        LDA #$30 ; '0'
        STA APU_SOUND_MOD+1
        LDA #9
        STA APU_NOISE_REG
        LDA #7
        STA APU_NOISE_REG+2
        LDA #$30 ; '0'
        STA APU_NOISE_REG+3
        LDA #$1F
        STA APU_SQUARE2_REG
        LDA #$8F
        STA APU_SQUARE2_REG+1
        LDA #0
        STA APU_SQUARE2_REG+2
        LDA #$33 ; '3'
        STA APU_SQUARE2_REG+3
        LDA #0
        STA APU_PATTERN
        RTS
; ---------------------------------------------------------------------------

S6_START:               
        LDA #$1D
        STA APU_SDELAY
        LDA #$FF
        STA APU_SOUND_MOD+0
        STA APU_SOUND_MOD+1
        STA APU_SOUND_MOD+2

S6_UPDATE:              
        DEC APU_SDELAY
        BEQ S6_PITCH
        LDA APU_SDELAY
        AND #3
        BNE S6_END
        LDA APU_SDELAY
        LSR
        LSR
        AND #1
        TAX
        LDA S6_SQ1MOD_TAB,X
        STA APU_SQUARE1_REG+2
        LDA S6_SQ2MOD_TAB,X
        STA APU_SQUARE2_REG+2
        LDA #8
        STA APU_SQUARE1_REG
        STA APU_SQUARE2_REG
        STA APU_SQUARE1_REG+1
        STA APU_SQUARE2_REG+1
        STA APU_SQUARE1_REG+3
        STA APU_SQUARE2_REG+3

S6_END:                 
        RTS
; ---------------------------------------------------------------------------

S6_PITCH:               
        LDA #$20 ; ' '
        STA APU_SOUND_MOD+0
        STA APU_SOUND_MOD+1
        STA APU_SOUND_MOD+2
        LDA #0
        STA APU_PATTERN
        RTS
```

## Tables


```
S6_SQ1MOD_TAB:  .BYTE $A9       
        .BYTE $A0
S6_SQ2MOD_TAB:  .BYTE $6A       
        .BYTE $64
WAVELEN_TAB:    .WORD     0, $7F0, $77E, $712, $6AE, $64E, $5F3, $59F, $54D, $501, $4B9, $475, $435, $3F8, $3BF, $389
        .WORD  $357, $327, $2F9, $2CF, $2A6, $280, $25C, $23A, $21A, $1FC, $1DF, $1C4, $1AB, $193, $17C, $167
        .WORD  $152, $13F, $12D, $11C, $10C,  $FD,  $EE,  $E1,  $D4,  $C8,  $BD,  $B2,  $A8,  $9F,  $96,  $8D
        .WORD   $85,  $7E,  $76,  $70,  $69,  $63,  $5E,  $58,  $53,  $4F,  $4A,  $46,  $42,  $3E,  $3A,  $37
        .WORD   $34,  $31,  $2E,  $2B,  $29,  $27,  $24,  $22,  $20,  $1E,  $1C,  $1B,  $1A
APU_MELODIES_TAB:.WORD $EB17,$EB85,$EBEA,$8080 ; 1: TITLE
        .WORD $ECBC,$ECDC,$ECFC,$4040 ; 2: STAGE_SCREEN
        .WORD     0,    0,$ED9C,$8080 ; 3: STAGE
        .WORD $ED4D,$ED7B,$ED18,$8080 ; 4: STAGE2
        .WORD $EC50,$EC7E,$ECAC,$8080 ; 5: GODMODE
        .WORD $EA27,$EA63,$EA9F,$8080 ; 6: BONUS
        .WORD $E8CC,$E91C,$E970,  $40 ; 7: FANFARE
        .WORD $EAE1,$EAFF,    0,$8080 ; 8: DIED
        .WORD $E9A2,$E9D5,$E9FF,$8080 ; 9: GAMEOVER
        .WORD $E903,$E957,$E988,  $40 ; 10: ???
TUNE7_TRI:  .BYTE $B0,$29,$AA,$29,$86,$29,$A4,$30,$86,$30,$35,$F9,$8C,$2E,$F8,$98
        .BYTE $2D,$86,$2D,$2E,$8C,$2D,$C8,$2B,$86,$32,$33,$8C,$32,$B0,$30,$8C
        .BYTE   0,$26,$28,$B0,$29,$AA,$29,$86,$29,$A4,$30,$86,$30,$35,$F9,$8C
        .BYTE $2E,$F8,$98,$2D,$86,$2D,$2E,$8C,$2D,$2B,$32,$33,$30,  0,$92,$30
        .BYTE $83,$32,$34,$8C,$35,  0,$1D,$86,$1D,$1D,$8C,$1D,  0,$98,  0,$FF
TUNE7_SQ2:  .BYTE $B0,$21,$AA,$21,$86,$21,$A4,$2D,$86,$2D,$30,$F9,$8C,$29,$F8,$98
        .BYTE $29,$86,$24,$29,$8C,$24,$C8,$27,$86,$2E,$2B,$8C,$2E,$28,$92,$24
        .BYTE $86,$24,$8C,$24,$24,$24,$24,$B0,$21,$AA,$21,$86,$21,$A4,$2D,$86
        .BYTE $2D,$30,$F9,$8C,$29,$F8,$98,$29,$86,$24,$29,$8C,$29,$26,$2B,$2E
        .BYTE $28,  0,$92,$28,$83,$29,$2B,$8C,$2D,  0,$11,$86,$11,$11,$8C,$11
        .BYTE   0,$98,  0,$FF
TUNE7_SQ1:  .BYTE $F9,$FD,  2,$98,$1D,$1C,$1A,$18,$FC,$1B,$1A,$18,$16,$18,$18,$1A
        .BYTE $1C,$FD,  2,$1D,$1C,$1A,$18,$FC
TUNE10_SQ1: .BYTE $F8,$8C,$1F,$1D,$1B,$2B,$18,  0,$92,$18,$83,$18,$18,$8C,$29,  0
        .BYTE $11,$86,$11,$11,$8C,$11,  0,$98,  0,$FF
TUNE9_TRI:  .BYTE $98,$28,$92,$24,$86,$24,$98,$27,$92,$24,$86,$24,$28,$29,$2A,$2B
        .BYTE $F9,$8C,$28,$24,$F8,$98,$22,$29,$28,$92,$24,$86,$24,$98,$27,$92
        .BYTE $24,$86,$24,$8C,$22,$84,$21,$20,$1F,$F9,$8C,$22,$23,$F8,$24,  0
        .BYTE $98,  0,$FF
TUNE9_SQ2:  .BYTE $E0,  0,$86,$1F,$21,$22,$23,$F9,$8C,$1F,$1C,$F8,$98,$1D,$22,$1F
        .BYTE $92,$1C,$86,$1C,$98,$1E,$92,$1B,$86,$1E,$8C,$16,$84,$15,$14,$13
        .BYTE $F9,$8C,$16,$17,$F8,$18,  0, $C,  0,$FF
TUNE9_SQ1:  .BYTE $8C,$18,$1F,  0,$1F,$18,$1F,  0,$1F,$18,$1F,  0,$1F,$14,$1B,  0
        .BYTE $1B,$18,$1F,  0,$1F,$14,$1B,  0,$1B,$16,$84,$15,$14,$13,$F9,$8C
        .BYTE $16,$17,$F8,$18,  0, $C,  0,$FF
TUNE6_TRI:  .BYTE $94,  0,$85,$2C,$2C,$33,$33,$8A,$33,  0,$85,$2C,$2C,$33,$33,$32
        .BYTE $32,$30,$30,$2E,$2E,$29,$29,$26,$26,$24,$24,$94,$22,  0,$85,$31
        .BYTE $31,$36,$36,$8A,$36,  0,$85,$31,$31,$3A,$3A,$37,$37,$35,$35,$33
        .BYTE $33,$31,$31,$30,$30,$2E,$2E,$2C,$2C,$2B,$2B,$FE
TUNE6_SQ2:  .BYTE $94,  0,$85,$24,$24,$30,$30,$8A,$30,  0,$85,$24,$24,$30,$30,$2E
        .BYTE $2E,$2D,$2D,$29,$29,$26,$26,$22,$22,$1D,$1D,$94,$1A,  0,$85,$2E
        .BYTE $2E,$31,$31,$8A,$31,  0,$85,$2E,$2E,$31,$31,$33,$33,$31,$31,$30
        .BYTE $30,$2E,$2E,$2C,$2C,$2B,$2B,$24,$24,$22,$22,$FE
TUNE6_SQ1:  .BYTE $85,$14,$14,$20,$20,$2C,$2C,$20,$20,$14,$14,$20,$20,$2C,$2C,$20
        .BYTE $20,$16,$16,$22,$22,$2E,$2E,$22,$22,$16,$16,$22,$22,$16,$16,$22
        .BYTE $22,$12,$12,$1E,$1E,$2A,$2A,$1E,$1E,$12,$12,$1E,$1E,$2A,$2A,$1E
        .BYTE $1E, $F, $F,$1B,$1B,$27,$27,$1B,$1B, $F, $F,$1B,$1B,$27,$27,$25
        .BYTE $25,$FE
TUNE8_TRI:  .BYTE $83,$30,$2B,$24,$1F,$18,$13, $C,  7,$18,$17,$16,$15,$14,$15,$16 ; +
        .BYTE $17,$92,$18,$86,$18,$F9,$8C,$1B,$1B,$F8,$98,$18,  0,$FF ;
TUNE8_SQ2:  .BYTE $98,  0,$83, $C, $B, $A,  9,  8,  9, $A, $B,$92, $C,$86, $C,$F9
        .BYTE $8C, $F, $F,$F8,$98, $C,  0,$FF
TUNE1_TRI:  .BYTE $87,$27,$33,$3F,$33,$36,$37,$33,$31,$27,$33,$3F,$33,$36,$37,$33
        .BYTE $31,$16,$16,$F9,$8E,$16,$F8,$87,$22,$22,$F9,$8E,$2E, $D, $F,$F8
        .BYTE $87,$38,$2E,$33,$38,$FD,  3,$F9,$8E,$37,$33,$32,$33,$F8,$9C,$2E
        .BYTE $87,$2C,$2E,$2A,$2B,$8E,  0,$3A,  0,$3A,$87,$33,$33,$F9,$8E,$33
        .BYTE $F8,$87,$38,$2E,$33,$38,$FC,$F9,$8E,$37,$F8,$87,$32,$33,$2C,$2E
        .BYTE $F9,$8E,$2B,$F8,$87,$2C,$2C,$F9,$8E,$2C,$F8,$87,$2B,$2B,$F9,$8E
        .BYTE $2B,$27,$27,  0,$F8,$87,$25,$26,$8E,$27,  0,$9C,  0,$FF
TUNE1_SQ2:  .BYTE $87,$1B,$27,$33,$27,$2A,$2B,$27,$25,$1B,$27,$33,$27,$2A,$2B,$27
        .BYTE $25,$13,$13,$F9,$8E,$13,$F8,$87,$1F,$1F,$F9,$8E,$2B,  1,  2,$F8
        .BYTE $87,$2C,$22,$27,$2C,$FD,  3,$F9,$8E,$31,$2E,$2A,$2B,$F8,$9C,$25
        .BYTE   0,$8E,$27,$22,$25,$2C,$9C,$2B,$87,$35,$22,$30,$33,$FC,$F9,$8E
        .BYTE $33,$F8,$87,$2A,$2B,$26,$27,$F9,$8E,$22,$F8,$87,$27,$27,$F9,$8E
        .BYTE $27,$F8,$87,$27,$27,$F9,$8E,$27,$1F,$1F,  0,$F8,$87,$19,$1A,$8E
        .BYTE $1B,  0,$9C,  0,$FF
TUNE1_SQ1:  .BYTE $87,$1B,$1B,$1B,$1B,$9C,  0,$87,$1B,$1B,$1B,$1B,$9C,  0,$87,$19
        .BYTE $19,$F9,$8E,$19,$F8,$87,$25,$25,$F9,$8E,$32,$F8,$B8,  0,$FD,  3
        .BYTE $87,$1B,$1B,$F9,$8E,$1B,$1E,$F8,$87,$1F,$22,$27,$1B,$1B,$1B,$1E
        .BYTE $F9,$8E,$1F,$F8,$87,$18,$19,$19,$19,$25,$18,$1A,$8E,$16,$87,$25
        .BYTE $19,$19,$19,$9C,  0,$FC,$F9,$8E,$2E,$F8,$87,$2D,$2E,$2A,$2B,$27
        .BYTE $22,$9C,  0,$87,$23,$23,$F9,$8E,$23,$22,$22,$F8,  0,$87,$19,$1A
        .BYTE $8E,$1B,  0,$9C,  0,$FF
TUNE5_TRI:  .BYTE $FD,  2,$90,$25,$88,$27,$29,  0,$29,$2B,  0,$FC,$C0,  0,$88,$22
        .BYTE $19,$22,$20,$1F,$19,$1F,$20,$FD,  2,$27,$29,$27,$2A,  0,$29,$27
        .BYTE   0,$FC,$C0,  0,$88,$20,$17,$20,$1E,$1D,$17,$1D,$1F,$FE
TUNE5_SQ2:  .BYTE $FD,  2,$90,$22,$88,$24,$26,  0,$26,$27,  0,$FC,$FD,  2,$1B,$13
        .BYTE $1B,$19,$18,$13,$18,$19,$FC,$FD,  2,$23,$25,$23,$27,  0,$25,$23
        .BYTE   0,$FC,$FD,  2,$19,$11,$19,$17,$16,$11,$16,$19,$FC,$FE
TUNE5_SQ1:  .BYTE $FD,  4,$90,$1B,$1B,$1B,$1B,$FC,$FD,  4,$19,$19,$19,$19,$FC,$FE
TUNE2_TRI:  .BYTE $85,$33,  0,$32,$33,$32,$33,$32,$33,$2E,  0,$2D,$2E,$2D,$2E,$2D
        .BYTE $2E,$2B,$2C,$2D,$2E,$2B,$2C,$2D,$2E,$2B,  0,$27,  0,$94,$25,$FF
TUNE2_SQ2:  .BYTE $85,$2B,  0,$2A,$2B,$2A,$2B,$2A,$2B,$27,  0,$26,$27,$26,$27,$26
        .BYTE $27,$27,$29,$2A,$2B,$27,$29,$2A,$2B,$27,  0,$1F,  0,$94,$22,$FF
TUNE2_SQ1:  .BYTE $F9,$8A,$1B,$1B,$1F,$22,$1B,$1B,$1F,$22,$F8,$85,$22,$24,$26,$27
        .BYTE $22,$24,$26,$27,$F9,$8A,$22,$1B,$F8,$94,$16,$FF
TUNE4_SQ1:  .BYTE $FD,  2,$87,$27,$27,$F9,$8E,$30,$31,$F8,$87,$22,$22,$F9,$8E,$2E
        .BYTE $30,$F8,$87,$31,$30,$F9,$8E,$2E,$F8,$FC,$FD,  2,$87,$25,$25,$F9
        .BYTE $8E,$2E,$2F,$F8,$87,$20,$20,$F9,$8E,$2C,$2E,$F8,$87,$2F,$2E,$F9
        .BYTE $8E,$2C,$F8,$FC,$FE
TUNE4_TRI:  .BYTE $9C,  0,$87,$25,$95,  0,$B8,  0,$9C,  0,$95,$27,$87,$27,$F9,$8E
        .BYTE $2C,$2B,$F8,$27,  0,$9C,  0,$87,$23,$95,  0,$B8,  0,$9C,  0,$95
        .BYTE $25,$87,$25,$F9,$8E,$2A,$F8,$87,$29,$2A,$8E,$25,  0,$FE
TUNE4_SQ2:  .BYTE $9C,  0,$87,$1D,$95,  0,$B8,  0,$9C,  0,$87,$1D,$95,  0,$B8,  0
        .BYTE $9C,  0,$87,$1B,$95,  0,$B8,  0,$9C,  0,$87,$1B,$95,  0,$B8,  0
        .BYTE $FE
TUNE3_SQ1:  .BYTE $87,$27,$27,$33,$27,$F9,$8E,$2A,$F8,$87,$2E,$30,$F9,$8E,$31,$31
        .BYTE $F8,$30,  0,$87,$25,$25,$31,$25,$F9,$8E,$29,$F8,$87,$2C,$2E,$F9
        .BYTE $8E,$2F,$F8,$87,$2E,$2F,$25,$24,$F9,$8E,$25,$F8,$87,$22,$22,$2E
        .BYTE $22,$F9,$8E,$2C,$F8,$87,$2B,$2C,$F9,$8E,$22,$2E,$F8,$22,  0,$87
        .BYTE $22,$22,$2E,$22,$F9,$8E,$2C,$F8,$87,$2B,$2C,$F9,$8E,$22,$2E,$F8
        .BYTE $87,$22,$22,$24,$26,$FE
```

## NMI Handler

```

NMI:

...

        JSR APU_PLAY_MELODY ; Play a tune
        JSR APU_PLAY_SOUND  ; Play the sound
        LDA BOOM_SOUND
        BEQ SET_SCROLL_REG
        LDA DEMOPLAY
        BNE SET_SCROLL_REG
        LDA #$E
        STA APU_DELTA_REG
        LDA #0
        STA BOOM_SOUND
        LDA #$C0 ; 'L'
        STA APU_DELTA_REG+2
        LDA #$FF
        STA APU_DELTA_REG+3
        LDA #$F
        STA APU_MASTERCTRL_REG
        LDA #$1F
        STA APU_MASTERCTRL_REG

...

SET_SCROLL_REG:

```

## BOOM Sample

```
ORG     $F000

; Digitization of the bomb blast.

BOOM_SAMPLE
    BYTE    $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FC, $00, $00, $1F, $F0, $00, $1F 
    BYTE    $FE, $00, $00, $07, $FF, $FF, $FF, $30, $00, $00, $3F, $FF, $E0, $07, $F0, $3F 
    BYTE    $F8, $1F, $F0, $1F, $FF, $FF, $C0, $00, $03, $FF, $C0, $FF, $FF, $C0, $7F, $C7 
    BYTE    $FF, $00, $00, $00, $FF, $F0, $00, $1F, $FF, $FE, $00, $00, $00, $00, $00, $00 
    BYTE    $00, $7F, $F0, $00, $07, $FF, $FF, $C0, $01, $80, $FF, $00, $02, $80, $3F, $FF 
    BYTE    $F8, $00, $03, $FF, $FF, $FF, $FE, $00, $00, $1F, $FF, $FF, $F0, $01, $FF, $FF 
    BYTE    $F0, $00, $0F, $FF, $FF, $FF, $00, $07, $FF, $F0, $00, $FB, $FF, $F8, $FF, $FF 
    BYTE    $00, $00, $00, $01, $FF, $FF, $FF, $C0, $3F, $F8, $00, $00, $00, $00, $7F, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FE, $00, $00, $00, $00, $0F, $FF, $F0, $00, $03, $FF, $FF 
    BYTE    $FF, $E0, $00, $00, $03, $FF, $FF, $F8, $00, $00, $00, $FF, $FF, $FF, $00, $00 
    BYTE    $00, $00, $1F, $E0, $00, $00, $07, $FF, $FF, $FE, $00, $00, $01, $FF, $FE, $00 
    BYTE    $00, $07, $FF, $FF, $FF, $F8, $00, $00, $00, $1F, $FF, $FF, $E0, $00, $00, $01 
    BYTE    $FF, $FF, $FF, $FF, $FF, $C0, $00, $00, $00, $00, $0F, $FF, $FF, $FF, $FE, $00 
    BYTE    $00, $07, $E0, $00, $00, $00, $00, $7F, $FF, $EF, $FF, $FF, $FF, $FF, $FF, $80 
    BYTE    $00, $00, $3F, $FF, $C0, $00, $00, $00, $0F, $FF, $FF, $FC, $00, $FF, $FF, $FE 
    BYTE    $00, $00, $00, $7F, $FF, $FF, $FF, $C0, $00, $00, $00, $03, $FF, $FF, $FF, $C0 
    BYTE    $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $FE, $00, $07, $FF, $00, $00, $0F, $FF 
    BYTE    $FF, $FC, $00, $00, $00, $00, $3F, $FF, $FF, $F0, $00, $00, $00, $0F, $FE, $00 
    BYTE    $00, $03, $FF, $FF, $C0, $00, $00, $00, $00, $FF, $FF, $FF, $FF, $FF, $80, $00 
    BYTE    $01, $FF, $FF, $FF, $F0, $00, $00, $00, $00, $7F, $FF, $FF, $DA, $2E, $00, $00 
    BYTE    $00, $7F, $FF, $FF, $FF, $FE, $00, $00, $FF, $80, $FF, $FC, $00, $00, $00, $00 
    BYTE    $03, $FF, $FF, $FF, $FF, $FF, $80, $00, $00, $0F, $FF, $FF, $F8, $00, $00, $00 
    BYTE    $03, $FF, $FF, $FF, $C0, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $80, $00, $00 
    BYTE    $01, $FF, $FF, $00, $00, $00, $01, $FF, $FF, $FF, $FF, $FF, $FF, $00, $3F, $FF 
    BYTE    $F0, $00, $00, $00, $07, $FF, $FF, $FF, $FE, $00, $03, $FF, $FE, $00, $00, $07 
    BYTE    $FF, $FF, $FF, $C0, $00, $00, $00, $00, $7F, $FF, $E0, $00, $00, $00, $03, $FF 
    BYTE    $FF, $FF, $FF, $FF, $00, $00, $7C, $00, $00, $00, $0F, $FF, $FF, $FF, $FF, $FF 
    BYTE    $F8, $00, $00, $00, $3F, $FF, $FF, $FF, $C0, $00, $00, $07, $FF, $F0, $00, $00 
    BYTE    $7F, $FF, $80, $00, $07, $FC, $00, $FF, $FF, $FF, $FF, $00, $00, $00, $0F, $FF 
    BYTE    $FF, $FF, $FE, $00, $00, $00, $3F, $FF, $FF, $F8, $00, $00, $00, $00, $3F, $FF 
    BYTE    $C0, $00, $00, $FF, $FF, $FF, $E0, $00, $00, $00, $1F, $FF, $FF, $FF, $FC, $00 
    BYTE    $00, $7F, $FF, $FF, $F8, $00, $00, $00, $3F, $FF, $FF, $FF, $00, $00, $00, $00 
    BYTE    $00, $FF, $FF, $FE, $11, $B0, $3F, $FF, $E0, $00, $00, $00, $00, $07, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $FF, $F8, $00, $00, $00, $00, $00, $00, $00, $0F, $FF 
    BYTE    $FF, $FF, $FF, $FF, $80, $00, $00, $00, $03, $FF, $FE, $00, $0F, $FF, $FF, $80 
    BYTE    $00, $00, $00, $FF, $FF, $FF, $FC, $00, $00, $00, $00, $FF, $FF, $FE, $00, $07 
    BYTE    $FF, $FF, $FF, $FF, $E0, $00, $00, $00, $00, $0F, $FF, $A0, $BF, $FF, $FF, $FE 
    BYTE    $00, $00, $7F, $FC, $00, $00, $00, $7F, $FF, $FF, $E0, $00, $00, $00, $3F, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $FF, $80, $00, $00, $00, $07, $F8, $00, $00, $00, $00 
    BYTE    $03, $FF, $FF, $FF, $FF, $FF, $00, $00, $00, $00, $00, $01, $FF, $FF, $FF, $F0 
    BYTE    $00, $07, $FF, $FF, $FC, $00, $00, $00, $00, $3F, $FF, $FF, $FF, $FF, $0F, $FF 
    BYTE    $FF, $FE, $00, $00, $00, $00, $00, $FF, $FF, $FF, $FF, $F8, $00, $01, $FF, $FC 
    BYTE    $00, $00, $00, $00, $FF, $FF, $FF, $FF, $C0, $00, $00, $00, $00, $03, $FF, $FF 
    BYTE    $80, $00, $00, $00, $00, $3F, $FF, $FF, $FF, $FF, $FF, $C0, $00, $3F, $FF, $FF 
    BYTE    $FF, $F8, $00, $00, $02, $00, $00, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $FF 
    BYTE    $FF, $FF, $E0, $B8, $00, $00, $00, $00, $00, $00, $3F, $FF, $FF, $FF, $FF, $FF 
    BYTE    $F8, $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $80, $00, $00, $00, $00, $0F 
    BYTE    $FF, $FF, $FF, $DF, $FF, $00, $00, $0F, $FF, $FF, $80, $00, $00, $00, $03, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $C0, $00, $00, $00, $01, $FF, $FF, $FF, $00, $00, $00 
    BYTE    $00, $1F, $FF, $FF, $FF, $FF, $FF, $FF, $E0, $00, $FF, $00, $00, $00, $00, $00 
    BYTE    $00, $01, $FF, $FF, $FF, $C0, $00, $00, $03, $FF, $FF, $FF, $FF, $FF, $F0, $00 
    BYTE    $00, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $FF, $80, $00, $00, $00, $00, $0F 
    BYTE    $FF, $FF, $FF, $F0, $00, $3F, $FF, $FF, $00, $00, $17, $FB, $C8, $84, $E5, $3F 
    BYTE    $FF, $20, $00, $14, $19, $D9, $9F, $F6, $00, $00, $17, $FF, $FF, $FF, $FF, $FE 
    BYTE    $00, $00, $00, $00, $00, $03, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FE, $00, $00 
    BYTE    $00, $00, $00, $00, $00, $00, $FF, $FF, $FF, $DC, $80, $00, $26, $FE, $57, $FF 
    BYTE    $FF, $80, $00, $00, $07, $FC, $C0, $3F, $FF, $FF, $7F, $BD, $C4, $61, $4E, $EE 
    BYTE    $67, $20, $46, $64, $EF, $E6, $00, $00, $FD, $A0, $00, $00, $00, $00, $07, $FF 
    BYTE    $FF, $FF, $FF, $FF, $F0, $00, $00, $00, $00, $00, $7F, $FF, $FF, $FF, $FF, $C0 
    BYTE    $00, $00, $00, $00, $0F, $7F, $F8, $DF, $FF, $FF, $C0, $03, $FB, $9F, $AF, $FF 
    BYTE    $FF, $80, $00, $00, $02, $F6, $A5, $6F, $FF, $FF, $FC, $00, $3F, $FF, $C0, $00 
    BYTE    $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $FF, $82, $00, $00, $00, $00, $00 
    BYTE    $00, $03, $FF, $FF, $FF, $FF, $C0, $00, $00, $00, $00, $7F, $FF, $FF, $FF, $FE 
    BYTE    $00, $00, $00, $00, $1E, $7B, $FF, $FF, $FF, $FF, $FF, $F8, $00, $BF, $FF, $FF 
    BYTE    $C0, $00, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $FF, $FE, $00, $00, $00, $00 
    BYTE    $00, $00, $01, $FF, $FF, $FF, $88, $20, $0D, $FF, $F0, $00, $00, $00, $03, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $F8, $00, $00, $00, $00, $00, $00, $FF, $FF, $FF, $FF 
    BYTE    $FF, $F8, $00, $00, $3F, $FF, $FF, $80, $00, $00, $00, $00, $00, $01, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FE, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $FF, $00, $00, $00 
    BYTE    $00, $00, $00, $01, $FF, $FF, $FF, $FF, $FF, $FF, $00, $00, $00, $00, $00, $00 
    BYTE    $07, $FF, $FF, $FF, $FF, $F6, $6F, $FF, $E0, $00, $00, $00, $00, $7F, $FF, $FF 
    BYTE    $FE, $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $FE, $00, $00, $00, $00, $00 
    BYTE    $06, $69, $43, $1A, $ED, $BF, $FF, $FF, $F0, $00, $00, $00, $00, $00, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $F0, $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $C0 
    BYTE    $00, $00, $00, $00, $00, $00, $17, $FF, $FF, $FF, $FF, $02, $29, $99, $DF, $FE 
    BYTE    $A4, $26, $75, $AC, $61, $AF, $6D, $84, $44, $50, $9F, $BB, $79, $D9, $9B, $EC 
    BYTE    $DB, $27, $FE, $00, $00, $00, $00, $07, $FF, $FF, $FF, $FC, $00, $00, $00, $00 
    BYTE    $1F, $FF, $FF, $FF, $80, $00, $0F, $FF, $FF, $FF, $00, $00, $00, $00, $1F, $FF 
    BYTE    $FF, $80, $00, $00, $03, $DC, $DA, $FF, $FF, $FF, $FF, $00, $00, $00, $00, $00 
    BYTE    $00, $3F, $FF, $FF, $FF, $FF, $FF, $F0, $00, $00, $00, $1F, $FF, $FF, $FF, $FF 
    BYTE    $F8, $00, $00, $00, $00, $00, $00, $5E, $FB, $EB, $AC, $C7, $EF, $BB, $79, $BF 
    BYTE    $FF, $E0, $00, $0F, $F8, $00, $00, $00, $FF, $FF, $FF, $FF, $FF, $FF, $FC, $00 
    BYTE    $00, $00, $00, $00, $00, $3F, $FF, $FF, $F7, $FF, $FF, $FF, $FA, $10, $00, $00 
    BYTE    $00, $00, $00, $00, $00, $00, $0F, $FC, $FB, $EE, $FF, $EA, $D9, $DF, $CE, $7F 
    BYTE    $FF, $FF, $80, $00, $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $FF, $FF, $E0 
    BYTE    $00, $00, $00, $00, $00, $7F, $FF, $FF, $FF, $F8, $00, $00, $00, $00, $00, $00 
    BYTE    $0F, $FF, $FF, $FF, $FF, $FF, $FF, $FC, $00, $00, $00, $00, $00, $00, $00, $3F 
    BYTE    $FF, $FF, $FF, $FF, $D3, $24, $25, $33, $CC, $C9, $20, $7F, $FF, $F0, $00, $00 
    BYTE    $00, $07, $FF, $FF, $FF, $FF, $00, $00, $00, $00, $0F, $FF, $FF, $FF, $FF, $FF 
    BYTE    $C6, $30, $40, $04, $20, $08, $61, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $FF 
    BYTE    $FC, $00, $00, $00, $00, $00, $00, $00, $00, $01, $FF, $FF, $FF, $FF, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FF, $62, $01, $00, $00, $80, $00, $00, $77, $5F, $FF, $FD, $76 
    BYTE    $EB, $5B, $7F, $FF, $FC, $00, $00, $00, $00, $00, $00, $3F, $FF, $FF, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $00, $00, $00, $00, $00, $00, $00, $00, $FF, $FF, $FF 
    BYTE    $FF, $FF, $FF, $80, $00, $00, $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $00 
    BYTE    $80, $00, $85, $11, $9E, $CC, $CD, $DB, $DF, $F6, $DF, $FF, $FF, $C0, $00, $00 
    BYTE    $00, $00, $00, $0F, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $80, $00, $00, $00 
    BYTE    $00, $00, $00, $00, $07, $FF, $FF, $FF, $FF, $FF, $FF, $F0, $00, $00, $00, $00 
    BYTE    $00, $00, $00, $00, $1F, $FF, $FF, $C0, $00, $07, $FF, $FF, $FF, $FF, $F0, $06 
    BYTE    $31, $66, $6B, $FD, $FE, $D9, $40, $00, $03, $FF, $FF, $FF, $FF, $FF, $80, $00 
    BYTE    $00, $00, $00, $00, $00, $00, $07, $FF, $FF, $FF, $FD, $F9, $88, $44, $4A, $61 
    BYTE    $4C, $CF, $6C, $E6, $7E, $EE, $69, $98, $4A, $CE, $B2, $61, $04, $66, $64, $CE 
    BYTE    $6D, $FF, $FF, $FE, $00, $00, $00, $00, $00, $01, $FF, $FF, $FF, $FF, $FE, $EF 
    BYTE    $FF, $FF, $E6, $20, $80, $00, $00, $00, $00, $00, $00, $00, $01, $DB, $EF, $FF 
    BYTE    $FF, $FF, $FF, $F9, $8A, $99, $8A, $4C, $99, $9B, $39, $BF, $FF, $C0, $00, $00 
    BYTE    $00, $00, $00, $03, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FC, $00, $00, $00, $00 
    BYTE    $00, $00, $00, $2F, $FF, $DF, $FF, $FF, $FF, $F0, $00, $00, $00, $00, $00, $7F 
    BYTE    $FF, $FF, $FF, $FF, $FF, $FF, $00, $00, $00, $00, $00, $3C, $A0, $00, $00, $00 
    BYTE    $1B, $7F, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FC, $A0, $00, $00, $00, $FF, $FF 
    BYTE    $FE, $61, $00, $00, $00, $00, $00, $00, $00, $3F, $FF, $FF, $FF, $FF, $FB, $DF 
    BYTE    $FF, $FF, $FF, $CC, $00, $00, $00, $00, $00, $00, $00, $00, $00, $00, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $FF, $FF, $F8, $00, $00, $00, $00, $00, $00, $41, $AD 
    BYTE    $26, $EF, $FF, $FF, $FF, $FF, $FF, $E0, $00, $00, $00, $00, $00, $00, $01, $5A 
    BYTE    $FF, $FF, $6F, $FF, $FF, $FF, $00, $00, $00, $05, $FF, $FF, $FF, $FF, $FF, $FA 
    BYTE    $28, $88, $10, $40, $00, $02, $10, $94, $66, $73, $36, $F7, $FF, $FF, $FC, $00 
    BYTE    $00, $00, $00, $FF, $FF, $FF, $FF, $FF, $F8, $00, $00, $00, $0F, $B1, $44, $10 
    BYTE    $00, $00, $65, $33, $DF, $FF, $FF, $F9, $9E, $FF, $FF, $FF, $FF, $FA, $E4, $00 
    BYTE    $00, $00, $00, $00, $00, $00, $00, $01, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $80 
    BYTE    $00, $00, $00, $02, $FB, $DD, $B6, $B7, $77, $DA, $28, $99, $4A, $BF, $FF, $C8 
    BYTE    $00, $00, $00, $00, $00, $1F, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $00, $00, $00 
    BYTE    $00, $00, $00, $00, $0F, $FF, $FF, $FF, $FF, $FF, $FF, $C0, $00, $00, $00, $00 
    BYTE    $00, $00, $00, $20, $45, $6D, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $F4, $00 
    BYTE    $00, $07, $BD, $FF, $BC, $C4, $80, $40, $00, $00, $00, $00, $06, $32, $EE, $CE 
    BYTE    $B6, $DF, $7B, $BF, $BF, $F7, $DC, $EF, $FF, $F2, $00, $00, $00, $00, $00, $00 
    BYTE    $0B, $FF, $FF, $FF, $FF, $FF, $FC, $00, $8F, $7F, $76, $CE, $52, $90, $00, $00 
    BYTE    $00, $7B, $62, $00, $00, $00, $00, $32, $23, $7F, $FF, $FF, $FF, $FF, $FF, $FF 
    BYTE    $FF, $FF, $00, $00, $00, $00, $00, $00, $02, $6F, $FF, $FE, $EF, $32, $98, $8C 
    BYTE    $25, $33, $5E, $F7, $BF, $F5, $99, $DF, $FF, $FE, $06, $6C, $48, $C4, $89, $11 
    BYTE    $04, $45, $14, $33, $BC, $88, $00, $00, $00, $00, $26, $6B, $5D, $FF, $FF, $FF 
    BYTE    $FF, $FF, $FF, $FF, $F0, $04, $00, $00, $00, $08, $88, $C6, $66, $D6, $DD, $BB 
    BYTE    $33, $67, $3B, $BB, $FF, $FD, $0A, $64, $90, $01, $00, $00, $8C, $AB, $BB, $FF 
    BYTE    $FF, $FB, $FF, $5D, $CC, $CA, $53, $33, $14, $21, $08, $84, $11, $09, $14, $45 
    BYTE    $22, $4C, $66, $6B, $32, $10, $93, $BF, $FF, $FF, $FF, $FF, $FF, $C0, $00, $00 
    BYTE    $00, $00, $00, $BF, $FB, $F7, $77, $FF, $E4, $20, $42, $4C, $CD, $67, $75, $EE 
    BYTE    $CE, $7A, $D9, $9D, $B3, $31, $84, $82, $44, $8A, $65, $55, $9D, $B3, $33, $5F 
    BYTE    $B6, $DC, $A0, $00, $00, $00, $00, $03, $FF, $FF, $FF, $FF, $FF, $FF, $F9, $02 
    BYTE    $00, $00, $00, $32, $88, $00, $12, $99, $9C, $EE, $F7, $FF, $FF, $FF, $84, $00 
    BYTE    $00, $04, $50, $8A, $FF, $FF, $FF, $FF, $FC, $00, $86, $7F, $FF, $FF, $E0, $00 
    BYTE    $00, $00, $00, $3F, $BF, $FB, $AE, $D9, $99, $D7, $28, $00, $00, $00, $00, $00 
    BYTE    $00, $7D, $FF, $FF, $FF, $FF, $FF, $FF, $E8, $48, $A4, $C6, $4D, $62, $88, $45 
    BYTE    $5F, $6C, $80, $00, $00, $01, $19, $9A, $DB, $BF, $FF, $FE, $FB, $33, $5E, $FF 
    BYTE    $BF, $76, $DA, $E6, $64, $D6, $00, $00, $00, $00, $00, $00, $DB, $33, $14, $22 
    BYTE    $8C, $44, $18, $EF, $FF, $FF, $FF, $FF, $FF, $FF, $FF, $F0, $00, $00, $00, $00 
    BYTE    $00, $CE, $75, $9D, $9C, $E7, $99, $99, $9A, $6A, $64, $CC, $CC, $CC, $CD, $6B 
    BYTE    $2A, $67, $35, $9B, $33, $19, $8C, $CC, $CC, $CA, $67, $7F, $F2, $00, $02, $19 
    BYTE    $DB, $B7, $5D, $FF, $FF, $FC, $20, $00, $00, $00, $00, $17, $F6, $33, $3E, $FF 
    BYTE    $FF, $FF, $FE, $F5, $8D, $9F, $4C, $00, $00, $00, $00, $00, $00, $00, $3F, $FF 
    BYTE    $FF, $FF, $FF, $FF, $FF, $F4, $80, $00, $00, $00, $00, $7F, $FF, $E6, $50, $80 
    BYTE    $00, $19, $FF, $FF, $FF, $FF, $FF, $EB, $33, $12, $00, $00, $81, $10, $22, $33 
    BYTE    $B5, $22, $12, $53, $5B, $5D, $7B, $F7, $EE, $BA, $CC, $CC, $CD, $99, $98, $C6 
    BYTE    $66, $71, $98, $88, $91, $8C, $4C, $FB, $52, $42, $52, $99, $DD, $D9, $9D, $BF 
    BYTE    $BF, $CC, $44, $01, $01, $00, $00, $14, $EE, $FF, $FF, $FF, $FF, $FD, $77, $7E 
    BYTE    $42, $00, $40, $00, $00, $00, $00, $0A, $33, $3D, $FF, $FF, $FF, $FF, $FF, $FE 
    BYTE    $66, $62, $22, $08, $41, $02, $13, $31, $84, $10, $09, $AF, $7B, $5C, $CC, $D6 
    BYTE    $73, $39, $94, $B3, $BF, $FF, $FF, $FF, $EE, $D6, $66, $28, $A2, $08, $40, $10 
    BYTE    $CA, $40, $00, $00, $10, $50, $CC, $CD, $BB, $F7, $F7, $FD, $FF, $7E, $FA, $D5 
    BYTE    $98, $C5, $33, $67, $AE, $75, $99, $4A, $10, $40, $81, $01, $00, $90, $51, $8C 
    BYTE    $CD, $EF, $F5, $A9, $45, $12, $94, $CC, $C9, $9E, $F7, $FF, $FF, $FF, $FF, $79 
    BYTE    $D9, $CC, $E5, $48, $88, $84, $04, $08, $21, $11, $14, $E3, $1C, $D3, $35, $99 
    BYTE    $91, $00, $09, $36, $FF, $FF, $FF, $FD, $6B, $32, $28, $00, $81, $29, $B7, $7B 
    BYTE    $F7, $EB, $B5, $9B, $19, $4A, $24, $44, $42, $25, $59, $98, $81, $20, $24, $46 
    BYTE    $33, $33, $9E, $EE, $FB, $F7, $FF, $BE, $D9, $9C, $EF, $75, $ED, $EE, $D9, $84 
    BYTE    $41, $04, $00, $81, $01, $09, $11, $14, $4C, $6A, $6B, $39, $CF, $BA, $E6, $B7 
    BYTE    $7F, $DF, $66, $22, $00, $04, $65, $D7, $BF, $BE, $FF, $BD, $CE, $66, $62, $94 
    BYTE    $91, $42, $93, $AC, $C4, $00, $40, $20, $04, $08, $CD, $7F, $FF, $FF, $F7, $FB 
    BYTE    $EF, $76, $A4, $88, $84, $11, $23, $18, $C6, $62, $45, $18, $A4, $CC, $D6, $77 
    BYTE    $BE, $D9, $89, $21, $08, $A2, $98, $DA, $EB, $DB, $BB, $EF, $DE, $ED, $99, $8A 
    BYTE    $6B, $5B, $5D, $D6, $CC, $CC, $E7, $28, $00, $00, $00, $00, $00, $99, $33, $1C 
    BYTE    $DB, $DE, $F7, $7F, $BF, $CE, $75, $9A, $CD, $93, $08, $A6, $67, $36, $B6, $B6 
    BYTE    $B3, $9C, $D9, $9B, $24, $10, $21, $02, $11, $46, $4C, $CD, $9C, $D9, $D9, $CE 
    BYTE    $BD, $FB, $B9, $94, $24, $22, $46, $32, $66, $6D, $75, $ED, $F7, $76, $CD, $B3 
    BYTE    $33, $9A, $DB, $5A, $CD, $67, $30, $84, $21, $11, $04, $22, $10, $A3, $18, $CC 
    BYTE    $DA, $D9, $92, $26, $67, $66, $67, $39, $D6, $EB, $D7, $BB, $BB, $9D, $99, $99 
    BYTE    $99, $9B, $32, $61, $49, $9D, $B3, $8C, $21, $11, $10, $51, $43, $1B, $3D, $FB 
    BYTE    $FE, $FD, $EE, $E7, $66, $66, $29, $46, $22, $89, $14, $28, $4A, $14, $A6, $53 
    BYTE    $3B, $6C, $CC, $44, $08, $46, $67, $5F, $BF, $DF, $77, $BE, $FE, $E6, $62, $28 
    BYTE    $11, $08, $84, $82, $26, $6B, $AF, $7D, $FE, $DD, $66, $66, $33, $18, $C5, $28 
    BYTE    $C5, $31, $4C, $65, $25, $32, $A9, $99, $32, $66, $66, $66, $B3, $D6, $CC, $E7 
    BYTE    $6E, $51, $22, $53, $67, $77, $7D, $73, $4A, $22, $96, $6C, $CC, $CA, $66, $33 
    BYTE    $2C, $CC, $CC, $E6, $6B, $35, $99, $9A, $CB, $33, $32, $51, $24, $CB, $39, $D7 
    BYTE    $39, $CC, $CC, $C6, $63, $0A, $48, $88, $93, $18, $93, $18, $CC, $CE, $73, $9B 
    BYTE    $59, $CD, $75, $DD, $EE, $B3, $19, $66, $73, $6B, $67, $59, $CE, $66, $66, $E7 
    BYTE    $9A, $62, $10, $01, $04, $67, $6D, $99, $33, $33, $9D, $AE, $76, $B6, $AC, $CC 
    BYTE    $CE, $6C, $CA, $22, $08, $10, $44, $49, $12, $93, $3A, $FF, $7F, $AD, $5A, $DA 
    BYTE    $EB, $6B, $39, $CC, $D9, $99, $93, $26, $29, $29, $31, $46, $31, $94, $CC, $CC 
    BYTE    $C6, $53, $99, $99, $AB, $35, $6C, $CE, $66, $CD, $9A, $E6, $CD, $99, $CD, $99 
    BYTE    $B2, $B2, $99, $4C, $66, $31, $98, $CC, $CC, $CD, $99, $98, $D3, $58, $CA, $99 
    BYTE    $9A, $DB, $29, $49, $14, $45, $26, $33, $35, $9B, $39, $B5, $DA, $DA, $EC, $E6 
    BYTE    $CD, $66, $CC, $CC, $C9, $99, $8C, $99, $93, $18, $C9, $8A, $22, $94, $CD, $75 
    BYTE    $EF, $75, $9A, $51, $14, $44, $89, $25, $29, $AC, $D9, $CC, $E6, $D6, $B9, $DB 
    BYTE    $5C, $EB, $33, $33, $39, $D7, $6B, $9D, $9C, $E6, $66, $31, $8A, $46, $14, $48 
    BYTE    $A2, $8C, $62, $63, $26, $66, $66, $D7, $66, $32, $64, $CC, $CF, $39, $CE, $76 
    BYTE    $73, $AD, $6B, $39, $B3, $33, $32, $66, $4C, $C9, $98, $CA, $63, $18, $CA, $66 
    BYTE    $53, $33, $31, $99, $8C, $CC, $CC, $D6, $66, $B3, $33, $33, $39, $B3, $99, $B3 
    BYTE    $94, $99, $32, $66, $29, $8C, $67, $5B, $5C, $CC, $CC, $DA, $E7, $5A, $CC, $D6 
    BYTE    $66, $63, $26, $31, $26, $56, $67, $3A, $DA, $CE, $66, $66, $4C, $63, $18, $C6 
    BYTE    $29, $46, $66, $33, $31, $99, $99, $CE, $73, $31, $85, $14, $CC, $D6, $EB, $AE 
    BYTE    $EB, $D7, $6D, $6B, $36, $6B, $59, $89, $44, $93, $18, $CA, $52, $91, $93, $32 
    BYTE    $C6, $69, $9A, $CC, $D9, $9A, $CC, $D9, $99, $66, $73, $AD, $6E, $B9, $CC, $C6 
    BYTE    $24, $44, $44, $89, $28, $A3, $13, $33, $5B, $AE, $EE, $B5, $CE, $75, $9B, $33 
    BYTE    $8C, $CC, $B1, $8C, $CC, $66, $4C, $CA, $99, $99, $96, $66, $66, $66, $6C, $9C 
    BYTE    $CC, $CC, $E6, $CE, $73, $32, $52, $63, $4E, $66, $B3, $B7, $5C, $CC, $C6, $18 
    BYTE    $A2, $52, $99, $CD, $5A, $E7, $6D, $98, $C9, $93, $26, $6B, $36, $63, $18, $92 
    BYTE    $28, $8A, $66, $4C, $CA, $73, $39, $D7, $3D, $75, $B6, $73, $33, $36, $32, $98 
    BYTE    $CC, $66, $66, $65, $33, $32, $99, $33, $33, $33, $33, $99, $9C, $CC, $D9, $66 
    BYTE    $D6, $CC, $C9, $8C, $A6, $66, $6B, $35, $9B, $35, $9D, $63, $32, $95, $33, $33 
    BYTE    $33, $33, $32, $66, $53, $4C, $A9, $99, $99, $99, $8C, $4A, $63, $35, $9B, $39 
    BYTE    $CC, $C6, $63, $32, $66, $66, $59, $99, $9A, $73, $9D, $66, $67, $39, $CE, $73 
    BYTE    $32, $63, $19, $99, $9B, $35, $9A, $D9, $CC, $C6, $14, $8A, $28, $C5, $26, $33 
    BYTE    $19, $99, $9B, $33, $59, $B3, $AC, $E6, $CE, $67, $33, $99, $B3, $39, $99, $99 
    BYTE    $99, $99, $94, $C5, $31, $AC, $CE, $6C, $CE, $66, $33, $19, $94, $C9, $8C, $65 
    BYTE    $29, $8C, $C6, $66, $73, $66, $65, $25, $33, $33, $39, $AD, $D6, $D6, $B3, $59 
    BYTE    $99, $99, $9B, $33, $33, $39, $9C, $E6, $C9, $9A, $66, $66, $26, $32, $B2, $66 
    BYTE    $9C, $CC, $CC, $CC, $C9, $93, $29, $8C, $A6, $66, $62, $51, $84, $94, $A6, $66 
    BYTE    $CE, $75, $B3, $9E, $73, $9C, $E7, $33, $99, $AC, $CB, $33, $33, $33, $33, $33 
    BYTE    $33, $26, $66, $59, $99, $8C, $CC, $C9, $99, $99, $99, $99, $99, $99, $9A, $B3 
    BYTE    $33, $33, $33, $33, $33, $66, $66, $64, $A6, $33, $33, $33, $26, $55, $33, $33 
    BYTE    $9C, $E6, $B3, $9A, $AC, $CC, $C6, $66, $66, $69, $99, $99, $99, $98, $CC, $C6 
    BYTE    $63, $19, $99, $99, $4C, $CC, $CA, $99, $99, $9C, $CD, $54, $CC, $CC, $CC, $E6 
    BYTE    $73, $39, $AC, $E6, $66, $66, $53, $29, $99, $4C, $CC, $CC, $CE, $66, $73, $35 
    BYTE    $99, $CC, $CC, $CE, $67, $33, $19, $92, $A4, $CC, $6C, $CE, $73, $59, $31, $88 
    BYTE    $A4, $65, $33, $35, $9B, $33, $32, $6B, $33, $31, $93, $26, $B3, $36, $67, $3B 
    BYTE    $35, $99, $98, $CC, $A6, $55, $99, $CD, $6B, $59, $CC, $CD, $4E, $66, $63, $31 
    BYTE    $99, $AC, $CC, $E6, $66, $73, $33, $31, $99, $4C, $A6, $32, $99, $4C, $99, $8C 
    BYTE    $E5, $4C, $C9, $8C, $CC, $CD, $59, $99, $99, $99, $CC, $E6, $73, $39, $9C, $E7 
    BYTE    $33, $99, $4C, $99, $98, $CC, $99, $33, $33, $56, $67, $33, $59, $9B, $31, $A9 
    BYTE    $99, $94, $CC, $CC, $66, $66, $53, $2A, $66, $4D, $33, $35, $59, $4C, $CC, $CC 
    BYTE    $D9, $99, $9A, $67, $36, $6C, $CC, $CC, $CA, $65, $4C, $CC, $CC, $CC, $CD, $99 
    BYTE    $AC, $D9, $9B, $33, $39, $99, $9B, $2A, $66, $54, $CC, $C6, $67, $33, $33, $33 
    BYTE    $33, $19, $4C, $99, $4C, $94, $C6, $64, $CC, $66, $66, $55, $66, $35, $99, $9A 
    BYTE    $CC, $E7, $36, $67, $35, $66, $6B, $33, $66, $73, $5A, $72, $66, $59, $99, $9C 
    BYTE    $E6, $63, $31, $8C, $C6, $33, $29, $99, $99, $54, $CC, $CC, $D9, $99, $99, $99 
    BYTE    $99, $9A, $72, $CC, $CC, $CC, $A9, $9B, $26, $63, $14, $C6, $63, $33, $33, $33 
    BYTE    $33, $99, $9C, $D3, $33, $19, $93, $9A, $B3, $59, $99, $99, $9C, $CE, $66, $67 
    BYTE    $33, $59, $9C, $CC, $CE, $66, $66, $6B, $33, $29, $94, $CC, $CC, $D6, $66, $66 
    BYTE    $66, $63, $33, $19, $32, $99, $4C, $66, $32, $99, $99, $8C, $CC, $CC, $CC, $CC 
    BYTE    $CD, $99, $AC, $E6, $CC, $CC, $AA, $CC, $E6, $6C, $CE, $67, $34, $E5, $99, $99 
    BYTE    $99, $99, $99, $99, $93, $33, $33, $33, $26, $31, $8C, $C6, $66, $66, $73, $33 
    BYTE    $33, $29, $99, $99, $99, $CB, $33, $33, $33, $66, $66, $65, $29, $96, $33, $99 
    BYTE    $99, $9C, $CC, $E6, $66, $66, $66, $66, $66, $CC, $CC, $CC, $66, $4C, $C9, $99 
    BYTE    $99, $66, $66, $67, $33, $66, $66, $CC, $CC, $E6, $66, $71, $AA, $66, $65, $8E 
    BYTE    $67, $33, $94, $CC, $66, $65, $99, $4C, $CC, $A6, $65, $4C, $CC, $CC, $CC, $98 
    BYTE    $C6, $4C, $D3, $95, $99, $99, $9C, $CD, $59, $AC, $CD, $99, $99, $B3, $39, $AB 
    BYTE    $32, $66, $33, $19, $A6, $73, $2C, $CC, $CE, $66, $66, $67, $33, $33, $33, $26 
    BYTE    $53, $18, $C9, $99, $99, $B3, $33, $33, $33, $33, $33, $33, $2A, $CC, $CC, $CC 
    BYTE    $CC, $D9, $99, $99, $65, $35, $99, $AC, $CC, $E6, $66, $99, $99, $99, $98, $C9 
    BYTE    $98, $CA, $66, $66, $53, $53, $36, $55, $99, $99, $B3, $33, $33, $1A, $66, $66 
    BYTE    $66, $66, $66, $6B, $35, $9B, $33, $39, $99, $B3, $59, $99, $9A, $CC, $A9, $CC 
    BYTE    $66, $64, $CC, $A6, $65, $33, $26, $66, $4C, $C6, $66, $64, $CC, $CC, $CC, $CC 
    BYTE    $CC, $CC, $CC, $CD, $99, $99, $99, $B3, $33, $33, $39, $99, $99, $B3, $33, $33 
    BYTE    $39, $99, $9A, $66, $53, $31, $99, $99, $39, $98, $CC, $CC, $CC, $CC, $CE, $66 
    BYTE    $67, $35, $99, $93, $32, $99, $99, $99, $9A, $CC, $CC, $CC, $CC, $CC, $CC, $B1 
    BYTE    $98, $CC, $CC, $66, $66, $66, $66, $66, $66, $66, $66, $66, $69, $A7, $33, $33 
    BYTE    $33, $39, $99, $99, $CC, $CC, $D3, $54, $CC, $CB, $19, $99, $99, $99, $99, $99 
    BYTE    $AB, $33, $33, $33, $33, $33, $39, $99, $00, $00, $00, $00, $00, $00, $00, $00 
```
