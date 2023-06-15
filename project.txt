IDEAL
MODEL small
STACK 100h
DATASEG
; --------------------------
;files
	filehandle dw ?
	Header db 54 dup (0)
	Palette db 256*4 dup (0)
	ScrLine db 320 dup (0)

;game opening
	filename_game_opening db 'game.bmp',0 ;game opening image
	
;instructions
	filename_instructions db 'I.bmp',0 ;game opening image
	
;tutorial opening
	filename_tutorial db 'tutorial.bmp',0 ;game opening image
	
;background level 1
	filename_level1 db 'level_1.bmp',0 ;1st background image

;background level 2
	filename_level2 db 'level_2.bmp',0 ;2nd background image
	
;background level 3
	filename_level3 db 'level_3.bmp',0 ;3rd background image
	
;background EndGame
	filename_EndGame db 'EndGame.bmp',0 ;3rd background image

;-----------------------------------messeges
;Error messege
	ErrorMsg db 'Error', 13, 10 ,'$'
	
;mouse coordes
	corner1_Instructions dw 184, 46
	corner2_Instructions dw 310, 64
	
	corner1_InstructionsExit dw 2, 4
	corner2_InstructionsExit dw 13, 12
	
	corner1_tutorial dw 116, 85
	corner2_tutorial dw 198, 103
	
	corner1_startGame dw 16, 47
	corner2_startGame dw 139, 67
	
	corner1_yes_EndGame dw 47, 70
	corner2_yes_EndGame dw 94, 86
	
	corner1_no_EndGame dw 239, 74
	corner2_no_EndGame dw 262, 86
	
	note dw 6ADh
	
	Clock equ es:6Ch
	
;player settings
	player_x dw 0
	player_y dw 0
	
	player_tutorial_x dw 35
	player_tutorial_y dw 107
	
	player_x1 dw 19
	player_y1 dw 175
	
	player_x2 dw 16
	player_y2 dw 184
	
	player_x3 dw 11
	player_y3 dw 189
	
	x_count dw ?
	y_count dw ?
	
	levelcount db 0
	read_count dw,5
	
	level_color db 7h
	color db 0h

; --------------------------
CODESEG
; --------------------------

;=========================================stat files
proc ReadHeader
	; Read BMP file header, 54 bytes
	mov ah,3fh
	mov bx, [filehandle]
	mov cx,54
	mov dx,offset Header
	int 21h
	ret
endp ReadHeader

proc ReadPalette
	; Read BMP file color palette, 256 colors * 4 bytes (400h)
	mov ah,3fh
	mov cx,400h
	mov dx,offset Palette
	int 21h
	ret
endp ReadPalette

proc CopyPal
	; Copy the colors palette to the video memory
	; The number of the first color should be sent to port 3C8h
	; The palette is sent to port 3C9h
	mov si,offset Palette
	mov cx,256
	mov dx,3C8h
	mov al,0
	; Copy starting color to port 3C8h
	out dx,al
	; Copy palette itself to port 3C9h
	inc dx
	PalLoop :
	; Note: Colors in a BMP file are saved as BGR values rather than RGB .
	mov al,[si+2] ; Get red value .
	shr al,2 ; Max. is 255, but video palette maximal
	; value is 63. Therefore dividing by 4.
	out dx,al ; Send it .
	mov al,[si+1] ; Get green value .
	shr al,2
	out dx,al ; Send it .
	mov al,[si] ; Get blue value .
	shr al,2
	out dx,al ; Send it .
	add si,4 ; Point to next color .
	; (There is a null chr. after every color.)
	loop PalLoop 
	ret
endp CopyPal 

proc CopyBitmap 
	; BMP graphics are saved upside-down .
	; Read the graphic line by line (200 lines in VGA format),
	; displaying the lines from bottom to top.
	mov ax, 0A000h
	mov es, ax
	mov cx,200
	PrintBMPLoop :
	push cx
	; di = cx*320, point to the correct screen line
	mov di,cx
	shl cx,6
	shl di,8
	add di,cx
	; Read one line
	mov ah,3fh
	mov cx,320
	mov dx,offset ScrLine 
	int 21h
	; Copy one line into video memory
	cld ; Clear direction flag, for movsb
	mov cx,320
	mov si,offset ScrLine 
	rep movsb ; Copy line to the screen
	;rep movsb is same as the following code :
	;mov es:di, ds:si
	;inc si
	;inc di
	;dec cx
	;loop until cx=0
	pop cx
	loop PrintBMPLoop 
	ret
endp CopyBitmap 
;=========================================end files

;=========================================stat of game opening
proc OpenFile_game_opening
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_game_opening
	int 21h
	jc openerror_game_opening
	mov [filehandle], ax
	ret
	openerror_game_opening:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_game_opening
;=========================================end of game opening

;=========================================start of instructions
proc OpenFile_instructions
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_instructions
	int 21h
	jc openerrorinstructions
	mov [filehandle], ax
	ret
	openerrorinstructions:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_instructions
;=========================================end of instructions

;=========================================stat of tutorial
proc OpenFile_tutorial
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_tutorial
	int 21h
	jc openerror_tutorial
	mov [filehandle], ax
	ret
	openerror_tutorial:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_tutorial
;=========================================end of tutorial

;=========================================start of background level 1
proc OpenFile_level1
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_level1
	int 21h
	jc openerror1
	mov [filehandle], ax
	ret
	openerror1:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_level1
;=========================================end of background level 1

;=========================================start of background level 2
proc OpenFile_level2
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_level2
	int 21h
	jc openerror2
	mov [filehandle], ax
	ret
	openerror2:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_level2
;=========================================end of background level 2

;=========================================start of background level 3
proc OpenFile_level3
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_level3
	int 21h
	jc openerror3
	mov [filehandle], ax
	ret
	openerror3:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_level3
;=========================================end of background level 3

;=========================================start EndGame
proc OpenFile_EndGame
	; Open file
	mov ah, 3Dh
	xor al, al
	mov dx, offset filename_EndGame
	int 21h
	jc openerrorEndGame
	mov [filehandle], ax
	ret
	openerrorEndGame:
	mov dx, offset ErrorMsg
	mov ah, 9h
	int 21h
	ret
endp OpenFile_EndGame
;=========================================end of EndGame

;=========================================start of player

proc drawpixel; מצייר פיקסל על המסך
	mov bh,0h
	mov cx,[player_x]
	mov dx,[player_y]
	mov al,[color]
	mov ah,0ch
	int 10h
	ret
endp drawpixel

proc player;מצייר את השחקן
	mov [y_count], si
draw1:
	mov [x_count], si
draw2:
	call drawpixel
	inc [player_x]
	cmp [x_count],0 
	dec [x_count]
	jne draw2
	inc [player_y]
	sub [player_x],si
	cmp [y_count],0 
	dec [y_count]
	jne draw1
	sub [player_y],si
	ret
endp player

proc clearpixel; מנקה פיקל על המסך
	mov bh,0h
	mov cx,[player_x]
	mov dx,[player_y]
	mov al,[level_color]
	mov ah,0ch
	int 10h
	ret
endp clearpixel

proc clear_player; מנקה את השחקן
	mov [y_count], si
clear1:
	mov [x_count], si
clear2:
	call clearpixel
	inc [player_x]
	cmp [x_count],0 
	dec [x_count]
	jne clear2
	inc [player_y]
	sub [player_x],si
	cmp [y_count],0 
	dec [y_count]
	jne clear1
	sub [player_y],si
	ret
endp clear_player

;==============================================read player

proc read_player_up; קורא את הצבע
	mov cx,[player_x]
	mov dx,[player_y]
	dec dx
	mov [read_count],si
readUP:
	mov bh,0h
	mov ah,0dh
	int 10h ; return al the pixel value read
	call comper
	inc cx
	cmp [read_count], 0
	dec [read_count]
	jne readUP
	ret
endp read_player_up

proc read_player_right; קורא את הצבע
	mov cx,[player_x]
	mov dx,[player_y]
	add cx,si
	mov [read_count],si
readRight:
	mov bh,0h
	mov ah,0dh
	int 10h ; return al the pixel value read
	call comper
	inc dx
	cmp [read_count], 0
	dec [read_count]
	jne readRight
	ret
endp read_player_right

proc read_player_down; קורא את הצבע
	mov cx,[player_x]
	mov dx,[player_y]
	add dx, si
	mov [read_count],si
readDown:
	mov bh,0h
	mov ah,0dh
	int 10h ; return al the pixel value read
	call comper
	inc cx
	cmp [read_count], 0
	dec [read_count]
	jne readDown
	ret
endp read_player_down

proc read_player_left; קורא את הצבע
	mov cx,[player_x]
	mov dx,[player_y]
	dec cx
	mov [read_count],si
readLeft:
	mov bh,0h
	mov ah,0dh
	int 10h ; return al the pixel value read
	call comper
	inc dx
	cmp [read_count], 0
	dec [read_count]
	jne readLeft
	ret
endp read_player_left

;=========================================end of player

;=========================================sound
proc startsound
	; open speaker
	in al, 61h
	or al, 00000011b
	out 61h, al
	; send control  to change frequency
	mov al, 0B6h
	out 43h, al
	; play sound
	mov ax, [note]
	out 42h, al ; Sending lower byte
	mov al, ah
	out 42h, al ; Sending upper byte
	ret
endp startsound

proc endsound
	; close the speaker
	in al, 61h
	and al, 11111100b
	out 61h, al
	ret
endp endsound

;=========================================sound

;=========================================other
proc timer
    ; wait for first change in timer
    mov ax, 40h
    mov es, ax
    mov ax, [Clock]
    FirstTick:
    cmp ax, [Clock]
    je FirstTick
    mov cx, 3
    DelayLoop:
    mov ax, [Clock]
    Tick :
    cmp ax, [Clock]
    je Tick
    loop DelayLoop
    ret
endp timer

proc sound
	call startsound
	call timer
	call endsound
	ret
endp sound

proc mouse
    ; Show mouse
    mov ax, 1h
    int 33h
    ; Loop until mouse click
    Mouse1:
    mov ax, 3h
    int 33h
    cmp bx, 01h ; check left mouse click
    jne Mouse1
    shr cx, 1; adjust cx to range 0-319, to fit scree
    dec dx 
    ret
endp mouse

proc comper
	cmp al,0h;black
	je Game2
	cmp al,7h;gray
	jne level1;eny other color
	ret
endp comper
;=========================================other
; --------------------------
start:
; --------------------------
mov ax, @data
mov ds, ax
;=========================================backgrounds
;Game opening
game_opening:
; Graphic mode
	mov ax,13h
	int 10h
	
	call OpenFile_game_opening
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	label_mouse:
	call mouse
	
	comper_tutorial:
    cmp cx, [ corner1_tutorial]
    jle comper_Instructions
    cmp dx, [corner1_tutorial+2]
    jle comper_Instructions
    cmp cx, [corner2_tutorial]
    jge comper_Instructions
    cmp dx, [corner2_tutorial+2]
    jge comper_Instructions
    jmp tutorial
	
	comper_Instructions:
    cmp cx, [corner1_Instructions]
    jle comper_startGame
    cmp dx, [corner1_Instructions+2]
    jle comper_startGame
    cmp cx, [corner2_Instructions]
    jge comper_startGame
    cmp dx, [corner2_Instructions+2]
    jge comper_startGame
    jmp Instructions
	
	comper_startGame:
    cmp cx, [corner1_startGame]
    jle label_mouse
    cmp dx, [corner1_startGame+2]
    jle label_mouse
    cmp cx, [corner2_startGame]
    jge label_mouse
    cmp dx, [corner2_startGame+2]
    jge label_mouse
	mov [levelcount],1
    jmp level_1
	
level1:
	jmp level
Game2:
	jmp Game1
Game_open:
	jmp game_opening
;Instructions
Instructions:
; Graphic mode
	mov ax,13h
	int 10h
	
	call OpenFile_instructions
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	label_mouse_instructions:
	call mouse
	
	comper_instructions_page:
    cmp cx, [corner1_InstructionsExit]
    jle label_mouse_instructions
    cmp dx, [corner1_InstructionsExit+2]
    jle label_mouse_instructions
    cmp cx, [corner2_InstructionsExit]
    jge label_mouse_instructions
    cmp dx, [corner2_InstructionsExit+2]
    jge label_mouse_instructions
    jmp game_opening
	
	
	level:
	call sound
	inc [levelcount]
	
	;jump tutorial
	cmp [levelcount],1
	je Game_open
	
	;jump level 2
	cmp [levelcount],2
	je level_2_1
	
	;jump level 3
	cmp [levelcount],3
	je level_3_1
	
	;jump end game
	cmp [levelcount],4
	je end1
	
	Game1:
	jmp Game

exit2:
	jmp exit1
	; Process tutorial file
tutorial:
; Graphic mode
	mov ax,13h
	int 10h
	
	mov [levelcount],0
	mov ax, [player_tutorial_x]; sets player x 
	mov [player_x],ax
	
	mov ax, [player_tutorial_y]; sets player y
	mov [player_y],ax
	
	call OpenFile_tutorial;open file
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	mov si,7
	jmp Game
	
; Process level 1  file
level_1:
; Graphic mode
	mov ax,13h
	int 10h
	
	mov ax, [player_x1]; sets player x 
	mov [player_x],ax
	
	mov ax, [player_y1]; sets player y
	mov [player_y],ax
	
	call OpenFile_level1;open file
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	mov si,5
	jmp Game
	
level_2_1:
	jmp level_2
end1:
	jmp EndGame
level_3_1:
	jmp level_3
	
;Process level 2  file
level_2:
; Graphic mode
	mov ax,13h
	int 10h
	
	mov ax, [player_x2]; sets player x 
	mov [player_x],ax
	
	mov ax, [player_y2]; sets player y
	mov [player_y],ax

	call OpenFile_level2;open file
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	mov si,3
	jmp Game
	
checkpoint_level_1:
	jmp level_1
	
checkpoint2:
	jmp start

;Process level 3  file
level_3:
; Graphic mode
	mov ax,13h
	int 10h
	
	mov ax, [ player_x3]; sets player x 
	mov [player_x],ax
	
	mov ax, [ player_y3]; sets player y
	mov [player_y],ax
	
	call OpenFile_level3;open file
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	mov si,2
	jmp Game
	
;==================================================backgrounds

;==================================================player movement
Game:	;moving and controlling the player

	call player;drawing the player
	mov ah,8h
	int 21h

;=============================================Errow keys
	cmp al,48h
	je moveup
	
	cmp al,4bh
	je moveleft
	
	cmp al,4dh
	je moveright
	
	cmp al,50h
	je movedown
;=============================================Errow keys

	cmp al,27
	je exit1
	jmp Game
	
	moveup:
	call read_player_up
	call clear_player
	inc [color]
	dec [player_y]
	jmp Game
	
	checkpoint1:
	jmp checkpoint2
	
	moveleft:
	call read_player_left
	call clear_player
	dec [color]
	dec [player_x]
	jmp Game
	
	moveright:
	call read_player_right
	call clear_player
	inc [color]
	inc [player_x]
	jmp Game
	
	movedown:
	call read_player_down
	call clear_player
	dec [color]
	inc [player_y]
	jmp Game
	
exit1:
	jmp exit
;==================================================player movement

EndGame:
; Graphic mode
	mov ax,13h
	int 10h
	
	call OpenFile_EndGame
	call ReadHeader
	call ReadPalette
	call CopyPal
	call CopyBitmap
	label_mouse_EndGame:
	call mouse
	
	comper_EndGame_yes:
    cmp cx, [corner1_yes_EndGame]
    jle comper_EndGame_no
    cmp dx, [corner1_yes_EndGame+2]
    jle comper_EndGame_no
    cmp cx, [corner2_yes_EndGame]
    jge comper_EndGame_no
    cmp dx, [corner2_yes_EndGame+2]
    jge comper_EndGame_no
    jmp checkpoint1
	
	comper_EndGame_no:
    cmp cx, [corner1_no_EndGame]
    jle label_mouse_EndGame
    cmp dx, [corner1_no_EndGame+2]
    jle label_mouse_EndGame
    cmp cx, [corner2_no_EndGame]
    jge label_mouse_EndGame
    cmp dx, [corner2_no_EndGame+2]
    jge label_mouse_EndGame
    jmp exit
; --------------------------
exit :
	mov [levelcount],0
text_mode:
; Back to text mode
	mov ah, 0
	mov al, 2
	int 10h
mov ax, 4c00h
int 21h
END start