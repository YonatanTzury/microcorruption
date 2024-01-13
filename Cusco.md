123456789qwertyuioasdfghjklzxcvbnm123456789qwe
									RET
43e0: 5645 0000 a045 0200 ee43 3000 1e45 3132   VE...E...C0..E12
43f0: 3334 3536 3738 3971 7765 7274 7975 696f   3456789qwertyuio
4400: 6173 6466 6768 6a6b 6c7a 00d0 085a 3f40   asdfghjklz...Z?@

									RET
43e0: 7044 7d00 ee43 e843 0043 0000 2445 3132   pD}..C.C.C..$E12
										 RET
43f0: 3334 3536 3738 3971 7765 7274 7975 696f   3456789qwertyuio
4400: 6173 6466 6768 6a6b 6c7a 7863 7662 6e6d   asdfghjklzxcvbnm
4410: 3132 3334 3536 3738 3971 7765 004f d445   123456789qwe

123456789qwertyuFDasdfghjklzxcvbnm123456789qwe

4500 <login>
4500:  3150 f0ff      add	#0xfff0, sp
4504:  3f40 7c44      mov	#0x447c "Enter the password to continue.", r15
4508:  b012 a645      call	#0x45a6 <puts>
450c:  3f40 9c44      mov	#0x449c "Remember: passwords are between 8 and 16 characters.", r15
4510:  b012 a645      call	#0x45a6 <puts>
4514:  3e40 3000      mov	#0x30, r14
4518:  0f41           mov	sp, r15
451a:  b012 9645      call	#0x4596 <getsn> // Read 0x30 (48) bytes into the stack
451e:  0f41           mov	sp, r15
4520:  b012 5244      call	#0x4452 <test_password_valid>
4524:  0f93           tst	r15
4526:  0524           jz	$+0xc <login+0x32>
4528:  b012 4644      call	#0x4446 <unlock_door>
452c:  3f40 d144      mov	#0x44d1 "Access granted.", r15
4530:  023c           jmp	$+0x6 <login+0x36>
4532:  3f40 e144      mov	#0x44e1 "That password is not correct.", r15
4536:  b012 a645      call	#0x45a6 <puts>
453a:  3150 1000      add	#0x10, sp
453e:  3041           ret

4452 <test_password_valid>
4452:  0412           push	r4
4454:  0441           mov	sp, r4
4456:  2453           incd	r4
4458:  2183           decd	sp
445a:  c443 fcff      mov.b	#0x0, -0x4(r4)
445e:  3e40 fcff      mov	#0xfffc, r14
4462:  0e54           add	r4, r14
4464:  0e12           push	r14
4466:  0f12           push	r15
4468:  3012 7d00      push	#0x7d
446c:  b012 4245      call	#0x4542 <INT>
4470:  5f44 fcff      mov.b	-0x4(r4), r15
4474:  8f11           sxt	r15
4476:  3152           add	#0x8, sp
4478:  3441           pop	r4
447a:  3041           ret