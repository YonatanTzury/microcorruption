43f0: 0000 0000 0000 0000 0000 0000 0000 4e44   ..............ND
4400: 3140 0044 1542 5c01 75f3 35d0 085a 3f40   1@.D.B\.u.5..Z?@
4410: 7c00 0f93 0724 8245 5c01 2f83 9f4f 3845   |....$.E\./..O8E
4420: 0024 f923 3f40 0001 0f93 0624 8245 5c01   .$.#?@.....$.E\.
4430: 1f83 cf43 7c24 fa23 3e40 2045 0f4e 3e40   ...C|$.#>@ E.N>@
4440: f800 3f40 0024 b012 8644 b012 0024 0f43   ..?@.$...D...$.C
4450: 32d0 f000 fd3f 3040 3645 1e41 0200 0212   2....?0@6E.A....
4460: 0f4e 8f10 024f 32d0 0080 b012 1000 3241   .N...O2.......2A
4470: 3041 5468 6973 4973 5365 6375 7265 5269   0AThisIsSecureRi
4480: 6768 743f 0000 0b12 0a12 0912 0812 0d43   ght?...........C
4490: cd4d 7c24 1d53 3d90 0001 fa23 3c40 7c24   .M|$.S=....#<@|$
44a0: 0d43 0b4d 684c 4a48 0d5a 0a4b 3af0 0f00   .C.MhLJH.Z.K:...
44b0: 5a4a 7244 8a11 0d5a 3df0 ff00 0a4d 3a50   ZJrD...Z=....M:P
44c0: 7c24 694a ca48 0000 cc49 0000 1b53 1c53   |$iJ.H...I...S.S
44d0: 3b90 0001 e723 0b43 0c4b 183c 1c53 3cf0   ;....#.C.K.<.S<.
44e0: ff00 0a4c 3a50 7c24 684a 4b58 4b4b 0d4b   ...L:P|$hJKXKK.K
44f0: 3d50 7c24 694d cd48 0000 ca49 0000 695d   =P|$iM.H...I..i]
4500: 4d49 dfed 7c24 0000 1f53 3e53 0e93 e623   MI..|$...S>S...#
4510: 3841 3941 3a41 3b41 3041 0e4f 0f4e 3041   8A9A:A;A0A.O.N0A
4520: 7768 6174 2773 2074 6865 2070 6173 7377   what's the passw
4530: 6f72 643f 0000 0013 4c85 1bc5 80df e9bf   ord?....L.......
4540: 3864 2bc6 4277 62b8 c3ca d965 a40a c1a3   8d+.Bwb....e....
4550: bbd1 a6ea b3eb 180f 78af ea7e 5c8e c695   ........x..~\...
4560: cb6f b8e9 333c 5aa1 5cee 906b d1aa a1c3   .o..3<Z.\..k....
4570: a986 8d14 08a5 a22c baa5 1957 192d abe1   .......,...W.-..
4580: 66b9 ef23 4a08 e95c d919 8069 07a5 ef01   f..#J..\...i....
4590: caa2 a30d f344 815e 3e10 e765 2bc8 2837   .....D.^>..e+.(7
45a0: abad ab3f 8cfa 754d 8ff0 b083 6b3e b3c7   ...?..uM....k>..
45b0: aefe b409 0000 0000 0000 0000 0000 0000   ................
45c0: 0000 0000 0000 0000 0000 0000 0000 0000   ...............

0b12 0412 0441 2452 3150 e0ff 3b40 2045
073c 1b53 8f11 0f12 0312 b012 6424 2152
6f4b 4f93 f623 3012 0a00 0312 b012 6424
2152 3012 1f00 3f40 dcff 0f54 0f12 2312
b012 6424 3150 0600 b490 88cd dcff 0520
3012 7f00 b012 6424 2153 3150 2000 3441
3b41 3041 1e41 0200 0212 0f4e 8f10 024f
32d0 0080 b012 1000


Assembled Objects
0b12           push	r11
0412           push	r4
0441           mov	sp, r4
2452           add	#0x4, r4
3150 e0ff      add	#0xffe0, sp
3b40 2045      mov	#0x4520, r11

// Print what's the password?
073c           jmp	$+0x10 // relative +10      r4 43fe, sp 43da, r15 24f8, r11 4520 
1b53           inc	r11    /////////// -0x12
8f11           sxt	r15
0f12           push	r15
0312           push	#0x0
b012 6424      call	#0x2464 // putchar
2152           add	#0x4, sp
6f4b           mov.b	@r11, r15 ////////// +0x10
4f93           tst.b	r15
f623           jnz	$-0x12

3012 0a00      push	#0xa
0312           push	#0x0 // putchar
b012 6424      call	#0x2464
2152           add	#0x4, sp
3012 1f00      push	#0x1f
3f40 dcff      mov	#0xffdc, r15
0f54           add	r4, r15
0f12           push	r15
2312           push	#0x2 // gets
b012 6424      call	#0x2464
3150 0600      add	#0x6, sp
b490 88cd dcff cmp	#0xcd88, -0x24(r4) /// r04 43fe -> 0x43fe - 0x24 = 0x43da -> Pass in hex: 88 cd
0520           jnz	$+0xc // not equal return
3012 7f00      push	#0x7f 
b012 6424      call	#0x2464 //UNLOCK_DOOR
2153           incd	sp
3150 2000      add	#0x20, sp
3441           pop	r4
3b41           pop	r11
3041           ret
1e41 0200      mov	0x2(sp), r14
0212           push	sr
0f4e           mov	r14, r15
8f10           swpb	r15
024f           mov	r15, sr
32d0 0080      bis	#0x8000, sr
b012 1000      call	#0x10 // Return to trap


4438 <main>
4438:  3e40 2045      mov	#0x4520, r14
443c:  0f4e           mov	r14, r15
443e:  3e40 f800      mov	#0xf8, r14 // Shellcode length 0xf8 (248)
4442:  3f40 0024      mov	#0x2400, r15
4446:  b012 8644      call	#0x4486 <enc>
444a:  b012 0024      call	#0x2400
444e:  0f43           clr	r15

4486 <enc>
4486:  0b12           push	r11
4488:  0a12           push	r10
448a:  0912           push	r9
448c:  0812           push	r8
448e:  0d43           clr	r13
4490:  cd4d 7c24      mov.b	r13, 0x247c(r13)
4494:  1d53           inc	r13
4496:  3d90 0001      cmp	#0x100, r13
449a:  fa23           jnz	$-0xa <enc+0xa>
449c:  3c40 7c24      mov	#0x247c, r12
44a0:  0d43           clr	r13
44a2:  0b4d           mov	r13, r11
44a4:  684c           mov.b	@r12, r8
44a6:  4a48           mov.b	r8, r10
44a8:  0d5a           add	r10, r13
44aa:  0a4b           mov	r11, r10
44ac:  3af0 0f00      and	#0xf, r10
44b0:  5a4a 7244      mov.b	0x4472(r10), r10
44b4:  8a11           sxt	r10
44b6:  0d5a           add	r10, r13
44b8:  3df0 ff00      and	#0xff, r13
44bc:  0a4d           mov	r13, r10
44be:  3a50 7c24      add	#0x247c, r10
44c2:  694a           mov.b	@r10, r9
44c4:  ca48 0000      mov.b	r8, 0x0(r10)
44c8:  cc49 0000      mov.b	r9, 0x0(r12)
44cc:  1b53           inc	r11
44ce:  1c53           inc	r12
44d0:  3b90 0001      cmp	#0x100, r11
44d4:  e723           jnz	$-0x30 <enc+0x1e>
44d6:  0b43           clr	r11
44d8:  0c4b           mov	r11, r12
44da:  183c           jmp	$+0x32 <enc+0x86>
44dc:  1c53           inc	r12
44de:  3cf0 ff00      and	#0xff, r12
44e2:  0a4c           mov	r12, r10
44e4:  3a50 7c24      add	#0x247c, r10
44e8:  684a           mov.b	@r10, r8
44ea:  4b58           add.b	r8, r11
44ec:  4b4b           mov.b	r11, r11
44ee:  0d4b           mov	r11, r13
44f0:  3d50 7c24      add	#0x247c, r13
44f4:  694d           mov.b	@r13, r9
44f6:  cd48 0000      mov.b	r8, 0x0(r13)
44fa:  ca49 0000      mov.b	r9, 0x0(r10)
44fe:  695d           add.b	@r13, r9
4500:  4d49           mov.b	r9, r13
4502:  dfed 7c24 0000 xor.b	0x247c(r13), 0x0(r15)
4508:  1f53           inc	r15
450a:  3e53           add	#-0x1, r14
450c:  0e93           tst	r14
450e:  e623           jnz	$-0x32 <enc+0x56>
4510:  3841           pop	r8
4512:  3941           pop	r9
4514:  3a41           pop	r10
4516:  3b41           pop	r11
4518:  3041           ret