
user: 4141 4141 4141 4141 4141 4141 4141 4141 4108 ff41 4141 4141 4141 4141 4141 4141 4141 4141 4141 4141 4141 4a44
pass: AAAAAAAAAAAAAAAAA

# Overwrite pass length test + return address with username, then overwrite the 00 bit for the password overflow test

43a0: 0000 7573 6572 6e61 6d65 0000 0000 0000   ..username......
		     min max
43b0: 0000 0008  1031 3233 3435 3637 3839 7177   .....123456789qw
43c0: 6572 7479  7569 6f61 7364 6667 686a 6b7a   ertyuioasdfghjkz
43d0: 7863 7662  6e6d 6d00 0000 0000 0000 0000   xcvbnmm.........

Ret address location: 43cc

4550 <login>
4550:  0b12           push	r11
4552:  0412           push	r4
4554:  0441           mov	sp, r4
4556:  2452           add	#0x4, r4 				// r4 = sp + 4
4558:  3150 d8ff      add	#0xffd8, sp 			
455c:  c443 faff      mov.b	#0x0, -0x6(r4)			// 0x43c6
4560:  f442 e7ff      mov.b	#0x8, -0x19(r4)			// 0x43b3
4564:  f440 1000 e8ff mov.b	#0x10, -0x18(r4)		// 0x43b4
456a:  3f40 8444      mov	#0x4484 "Authentication now requires a username and password.", r15
456e:  b012 2847      call	#0x4728 <puts>
4572:  3f40 b944      mov	#0x44b9 "Remember: both are between 8 and 16 characters.", r15
4576:  b012 2847      call	#0x4728 <puts>
457a:  3f40 e944      mov	#0x44e9 "Please enter your username:", r15
457e:  b012 2847      call	#0x4728 <puts>
4582:  3e40 6300      mov	#0x63, r14
4586:  3f40 0424      mov	#0x2404, r15
458a:  b012 1847      call	#0x4718 <getsn> 		// Read username 0x63 (99) bytes to 0x2404
458e:  3f40 0424      mov	#0x2404, r15
4592:  b012 2847      call	#0x4728 <puts>
4596:  3e40 0424      mov	#0x2404, r14
459a:  0f44           mov	r4, r15
459c:  3f50 d6ff      add	#0xffd6, r15
45a0:  b012 5447      call	#0x4754 <strcpy> 		// Copy username to 0x43a2 (on stack)
45a4:  3f40 0545      mov	#0x4505 "Please enter your password:", r15
45a8:  b012 2847      call	#0x4728 <puts>
45ac:  3e40 6300      mov	#0x63, r14
45b0:  3f40 0424      mov	#0x2404, r15
45b4:  b012 1847      call	#0x4718 <getsn> 		// Read pass 0x63 (99) bytes to 0x2404
45b8:  3f40 0424      mov	#0x2404, r15
45bc:  b012 2847      call	#0x4728 <puts>
45c0:  0b44           mov	r4, r11
45c2:  3b50 e9ff      add	#0xffe9, r11
45c6:  3e40 0424      mov	#0x2404, r14
45ca:  0f4b           mov	r11, r15
45cc:  b012 5447      call	#0x4754 <strcpy> 		// Copy pass to 0x43b5 (on stack)
45d0:  0f4b           mov	r11, r15
45d2:  0e44           mov	r4, r14
45d4:  3e50 e8ff      add	#0xffe8, r14
45d8:  1e53           inc	r14						// AAA - Compare - 0x43b5 - Find pass last byte
45da:  ce93 0000      tst.b	0x0(r14)
45de:  fc23           jnz	$-0x6 <login+0x88> 		// Loop to AAA
45e0:  0b4e           mov	r14, r11
45e2:  0b8f           sub	r15, r11 				// r11 - pass length
45e4:  5f44 e8ff      mov.b	-0x18(r4), r15
45e8:  8f11           sxt	r15
45ea:  0b9f           cmp	r15, r11 				// Check if pass smaller then 0x10 (16)
45ec:  0628           jnc	$+0xe <login+0xaa> 		// Jump to BBB if good size
45ee:  1f42 0024      mov	&0x2400, r15
45f2:  b012 2847      call	#0x4728 <puts>
45f6:  3040 4044      br	#0x4440 <__stop_progExec__>
45fa:  5f44 e7ff      mov.b	-0x19(r4), r15 			// BBBB
45fe:  8f11           sxt	r15
4600:  0b9f           cmp	r15, r11 				// Check if bigger then 0x8 (8)
4602:  062c           jc	$+0xe <login+0xc0> 		// Jump to CCCC if good size
4604:  1f42 0224      mov	&0x2402, r15
4608:  b012 2847      call	#0x4728 <puts>
460c:  3040 4044      br	#0x4440 <__stop_progExec__>
4610:  c443 d4ff      mov.b	#0x0, -0x2c(r4)			// 0x43a0
4614:  3f40 d4ff      mov	#0xffd4, r15
4618:  0f54           add	r4, r15
461a:  0f12           push	r15 					// CCCC
461c:  0f44           mov	r4, r15
461e:  3f50 e9ff      add	#0xffe9, r15
4622:  0f12           push	r15
4624:  3f50 edff      add	#0xffed, r15
4628:  0f12           push	r15
462a:  3012 7d00      push	#0x7d
462e:  b012 c446      call	#0x46c4 <INT>
4632:  3152           add	#0x8, sp
4634:  c493 d4ff      tst.b	-0x2c(r4)				// 0x43a0
4638:  0524           jz	$+0xc <login+0xf4>		// Jump to wrong pass
463a:  b012 4a44      call	#0x444a <unlock_door>
463e:  3f40 2145      mov	#0x4521 "Access granted.", r15
4642:  023c           jmp	$+0x6 <login+0xf8>
4644:  3f40 3145      mov	#0x4531 "That password is not correct.", r15
4648:  b012 2847      call	#0x4728 <puts>
464c:  c493 faff      tst.b	-0x6(r4)				// Make sure username not overflow
4650:  0624           jz	$+0xe <login+0x10e> 	// Jump to end
4652:  1f42 0024      mov	&0x2400, r15
4656:  b012 2847      call	#0x4728 <puts>
465a:  3040 4044      br	#0x4440 <__stop_progExec__>
465e:  3150 2800      add	#0x28, sp
4662:  3441           pop	r4
4664:  3b41           pop	r11
4666:  3041           ret