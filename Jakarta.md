user: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
-> 0x1f - 0x20 = 0xffff -> 0x01ff & 0xfff = 0x01ff (scanf length input)
-> 0x1ff + 0x20 = 0x021f -> 0x1f < 0x21 (Because cmp.b compare only the last byte)
pass: 0x1ff * 'c'
pass: ccccLDccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
LD == 4c44

AAAAAAAAAAAAAAAABBBBBBBBBBBBBBB.....@D
3ff0: 1846 4141 4141 4141 4141 4141 4141 4141   .FAAAAAAAAAAAAAA
4000: 4141 4242 4242 4242 4242 4242 4242 4242   AABBBBBBBBBBBBBB
4010: 4200 0000 0000 4044 0000 0000 0000 0000   B.....@D........

4560 <login>
4560:  0b12           push	r11
4562:  3150 deff      add	#0xffde, sp
4566:  3f40 8244      mov	#0x4482 "Authentication requires a username and password.", r15
456a:  b012 c846      call	#0x46c8 <puts>
456e:  3f40 b344      mov	#0x44b3 "Your username and password together may be no more than 32 characters.", r15
4572:  b012 c846      call	#0x46c8 <puts>
4576:  3f40 fa44      mov	#0x44fa "Please enter your username:", r15
457a:  b012 c846      call	#0x46c8 <puts>

# Read username 0xff (255) bytes to 0x2402
457e:  3e40 ff00      mov	#0xff, r14
4582:  3f40 0224      mov	#0x2402, r15
4586:  b012 b846      call	#0x46b8 <getsn>				
458a:  3f40 0224      mov	#0x2402, r15
458e:  b012 c846      call	#0x46c8 <puts>

# Calc username length - from #0x2401 - r11 holds username length
4592:  3f40 0124      mov	#0x2401, r15
4596:  1f53           inc	r15
4598:  cf93 0000      tst.b	0x0(r15)
459c:  fc23           jnz	$-0x6 <login+0x36>			
459e:  0b4f           mov	r15, r11
45a0:  3b80 0224      sub	#0x2402, r11

# Copy username to stack
45a4:  3e40 0224      mov	#0x2402, r14
45a8:  0f41           mov	sp, r15
45aa:  b012 f446      call	#0x46f4 <strcpy>			

# Check username length be less then 0x21 (32 + 1 for null terminator) - Skip force exit
45ae:  7b90 2100      cmp.b	#0x21, r11					
45b2:  0628           jnc	$+0xe <login+0x60>

45b4:  1f42 0024      mov	&0x2400, r15
45b8:  b012 c846      call	#0x46c8 <puts>
45bc:  3040 4244      br	#0x4442 <__stop_progExec__>

45c0:  3f40 1645      mov	#0x4516 "Please enter your password:", r15
45c4:  b012 c846      call	#0x46c8 <puts>


pc  45cc  sp 3ff2  sr 0000  cg 0000
r04 0000 r05 5a08 r06 0000 r07 0000 
r08 0000 r09 0000 r10 0000 r11 0020 
r12 0000 r13 4012 r14 001f r15 0000 

# Read pass remaining bytes to 0x2402
45c8:  3e40 1f00      mov	#0x1f, r14		// r14 0x1f, r11 0x20
45cc:  0e8b           sub	r11, r14        // -> r14 0xffff
45ce:  3ef0 ff01      and	#0x1ff, r14		// -> r14 0x1ff
45d2:  3f40 0224      mov	#0x2402, r15
45d6:  b012 b846      call	#0x46b8 <getsn>
45da:  3f40 0224      mov	#0x2402, r15
45de:  b012 c846      call	#0x46c8 <puts>

# Copy pass to stack the username end locations
45e2:  3e40 0224      mov	#0x2402, r14
45e6:  0f41           mov	sp, r15
45e8:  0f5b           add	r11, r15					
45ea:  b012 f446      call	#0x46f4 <strcpy>

# Check pass length be less then 0x21 (32 + 1 for null terminator) - R15 -> Skip force exit
45ee:  3f40 0124      mov	#0x2401, r15
45f2:  1f53           inc	r15
45f4:  cf93 0000      tst.b	0x0(r15)
45f8:  fc23           jnz	$-0x6 <login+0x92>			
45fa:  3f80 0224      sub	#0x2402, r15
45fe:  0f5b           add	r11, r15

# Check user+pass length be less then 0x21 (32 + 1 for null terminator)
4600:  7f90 2100      cmp.b	#0x21, r15					
4604:  0628           jnc	$+0xe <login+0xb2>


4606:  1f42 0024      mov	&0x2400, r15
460a:  b012 c846      call	#0x46c8 <puts>
460e:  3040 4244      br	#0x4442 <__stop_progExec__>
4612:  0f41           mov	sp, r15
4614:  b012 5844      call	#0x4458 <test_username_and_password_valid>
4618:  0f93           tst	r15
461a:  0524           jz	$+0xc <login+0xc6>
461c:  b012 4c44      call	#0x444c <unlock_door>
4620:  3f40 3245      mov	#0x4532 "Access granted.", r15
4624:  023c           jmp	$+0x6 <login+0xca>
4626:  3f40 4245      mov	#0x4542 "That password is not correct.", r15
462a:  b012 c846      call	#0x46c8 <puts>
462e:  3150 2200      add	#0x22, sp
4632:  3b41           pop	r11
4634:  3041           ret

4458 <test_username_and_password_valid>
4458:  0412           push	r4
445a:  0441           mov	sp, r4
445c:  2453           incd	r4
445e:  2183           decd	sp
4460:  c443 fcff      mov.b	#0x0, -0x4(r4)
4464:  3e40 fcff      mov	#0xfffc, r14
4468:  0e54           add	r4, r14
446a:  0e12           push	r14
446c:  0f12           push	r15
446e:  3012 7d00      push	#0x7d
4472:  b012 6446      call	#0x4664 <INT>
4476:  5f44 fcff      mov.b	-0x4(r4), r15
447a:  8f11           sxt	r15
447c:  3152           add	#0x8, sp
447e:  3441           pop	r4
4480:  3041           ret