;---------------------------------------
;--- Copyright (c) 2019 Greg Nacu ------
;---------------------------------------

;----[ presenter.a ]--------------------

         .include "/s/:basic.s"

         .include "/s/:io.s"
         .include "/s/:io_joy.s"

         .include "/s/ker/:file.s"
         .include "/s/ker/:io.s"
         .include "/s/ker/:key.s"
         .include "/s/ker/:mem.s"

         .include "/s/:pointer.s"

lfn      = $05
clrscr   = $93

cr       = $0d ;Carriage Return

c_bang   = "!" ;New Slide
c_pause  = "," ;Pause Output
c_end    = "." ;End Marker

ptr      = $fb ;$fc
ystore   = $07

slideptr = $fd ;$fe
slideidx = $02

slidehi  = $ce00 ;Ptr's page byte
slidelo  = $cf00 ;Y Index into page

         ;Configure IRQ
         #rdxy $0314
         #stxy romirq+1

         #ldxy scanbut
         sei
         #stxy $0314
         cli

;Prepare Screen for Output
;---------------------------------------

         lda #clrscr
         jsr chrout

         lda #$0e      ;Upper/Lower
         jsr chrout

         lda #$97      ;DkGrey PETSCII
         jsr chrout

         lda #1        ;White
         sta vic+$20   ;Border
         sta vic+$21   ;Background


         ;Clear Slide Pointer Buffer

         lda #0
         ldx #$ff
         stx slideidx

         inx
         sta slidehi,x
         sta slidelo,x
         inx
         bne *-7



;Load Presentation Data
;---------------------------------------

         lda #lfn ;Logical File #
         ldx $ba  ;Current Dev #
         ldy #$05 ;Secondary Address

         jsr setlfs

         lda #14 ;strlen(introc64os.pet)
         #ldxy filename

         jsr setnam
         jsr open

         ldx #lfn
         jsr chkin

         #ldxy buffer
         #stxy ptr

         ;Read data into memory

         .block
         ldy #0
next     jsr chrin
         sta (ptr),y

         iny
         bne *+4
         inc ptr+1

         jsr readst
         beq next

         lda #0        ;Data Terminator
         sta (ptr),y
         .bend

         lda #lfn
         jsr close

         jsr clrchn


;Output Presenter Data
;---------------------------------------

         #ldxy buffer
         #stxy ptr

         .block
         ldy #0

next     lda (ptr),y
         beq done      ;Data End

         sty ystore
         jsr procchr
         ldy ystore

         iny
         bne next
         inc ptr+1
         bne next
done


;Restore screen properties
;---------------------------------------

         lda #clrscr
         jsr chrout

         lda #$8e      ;Graphix/Upper
         jsr chrout

         lda #$9a      ;Blue PETSCII
         jsr chrout

         lda #14       ;Lt.Blue
         sta vic+$20   ;Border

         lda #6        ;Blue
         sta vic+$21   ;Background

         rts
         .bend

;Fallthrough if no end marker present.


endslide ;End Slide Marker Encountered
         .block
iloop    jsr getbut
         beq iloop

         cmp #jbut_up
         bne *+5
         jmp slidetop

         cmp #jbut_lt
         bne *+5
         jmp slidebak

         sei
         jsr restor
         cli

         rts
         .bend

newslide ;New Slide Marker Encountered
         .block
         ldx slideidx
         bmi first

iloop    jsr getbut
         beq iloop

         cmp #jbut_up
         bne *+5
         jmp slidetop

         cmp #jbut_lt
         bne *+5
         jmp slidebak

first    inc slideidx
         ldx slideidx

         lda ptr+1
         sta slidehi,x
         lda ystore
         sta slidelo,x

         lda #clrscr
         jmp chrout
         .bend

pause    ;Output Pause Encountered
         .block
iloop    jsr getbut
         beq iloop

         cmp #jbut_up
         bne *+5
         jmp slidetop

         cmp #jbut_lt
         bne *+5
         jmp slidebak

         rts
         .bend

slidebak ;Go back one slide
         .block
         ldx slideidx
         dex
         bpl *+3
         inx
         stx slideidx

         ;Fallthrough to slidetop
         .bend

slidetop ;Go to top of current slide
         .block
         ldx slideidx

         lda slidehi,x
         sta ptr+1
         lda slidelo,x
         sta ystore

         lda #clrscr
         jmp chrout
         .bend

procchr  ;Process Character
         ;A -> Character
         .block
         cmp #cr
         bne notcr

         ldx #1
         stx codechk
         bne output

notcr    ldx codechk
         beq output

         ldx #0
         stx codechk

         ;Check for Control Code
         cmp #c_bang
         bne *+5
         jmp newslide

         cmp #c_pause
         bne *+5
         jmp pause

         cmp #c_end
         bne *+5
         jmp endslide

output   jmp chrout

codechk  .byte 0
         .bend

;filename .null "introc64os.pet"
filename .null "video1541u.pet"

getbut   ;Get Joystick Button Event
         .block
         ;A <- Button Event

         ldx #0

         lda butevt
         and #jbut_f1
         cmp #jbut_f1
         beq done

         lda butevt
         and #jbut_up
         cmp #jbut_up
         beq done

         lda butevt
         and #jbut_lt
         cmp #jbut_lt
         beq done

         lda #0
         rts

done     cmp #0
         stx butevt
         rts
         .bend

scanbut  ;Scan the Joystick Buttons
         .block
;port = 1 for control port 1
;port = 0 for control port 2

port     = 0

         lda cia1+port     ;Scan PRB
         cmp cia1+port
         bne *-6           ;Debounce

         eor #$ff          ;Invert
         and #%00011111    ;Mask

         cmp butstat
         beq romirq

         sta butstat
         cmp #0
         beq romirq
         sta butevt

         .bend
romirq   jmp $ffff         ;Self Mod

butstat  .byte 0
butevt   .byte 0

;Adjust to page align the buffer, but
;keep it a minimum distance from the end
;of the presenter main code.

         *= $1000

buffer ;Free Memory following the binary
