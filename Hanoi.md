aaaaaaaaaaaaaaaaaaaaaaaaaa
123456789asdfghjklzxcvbnmp
123456789asdfgh®klzxcvbnmp

2400: 3132 3334 3536 3738 3961 7364 6667 686a   123456789asdfghj
2410: 6b6c 7a78 6376 626e 6d70 0000 0000 0000   klzxcvbnmp......

cmp --> equal --> 0
Password = #0x2400
4520 <login>
4520:  c243 1024      mov.b	#0x0, &0x2410
4524:  3f40 7e44      mov	#0x447e "Enter the password to continue.", r15
4528:  b012 de45      call	#0x45de <puts>
452c:  3f40 9e44      mov	#0x449e "Remember: passwords are between 8 and 16 characters.", r15
4530:  b012 de45      call	#0x45de <puts>

4534:  3e40 1c00      mov	#0x1c, r14
4538:  3f40 0024      mov	#0x2400, r15
453c:  b012 ce45      call	#0x45ce <getsn> // Read 0x1c (28) bytes into 0x2400

4540:  3f40 0024      mov	#0x2400, r15
4544:  b012 5444      call	#0x4454 <test_password_valid>

4548:  0f93           tst	r15 // Should be 1 if sucess
454a:  0324           jz	$+0x8 <login+0x32> // 0x4552 False
454c:  f240 8900 1024 mov.b	#0x89, &0x2410

4552:  3f40 d344      mov	#0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call	#0x45de <puts>

455a:  f290 ae00 1024 cmp.b	#0xae, &0x2410
4560:  0720           jnz	$+0x10 <login+0x50> // 0x4570 - Wrong pass
4562:  3f40 f144      mov	#0x44f1 "Access granted.", r15
4566:  b012 de45      call	#0x45de <puts>
456a:  b012 4844      call	#0x4448 <unlock_door>
456e:  3041           ret
4570:  3f40 0145      mov	#0x4501 "That password is not correct.", r15
4574:  b012 de45      call	#0x45de <puts>
4578:  3041           ret


4454 <test_password_valid>
4454:  0412           push	r4
4456:  0441           mov	sp, r4
4458:  2453           incd	r4
445a:  2183           decd	sp
445c:  c443 fcff      mov.b	#0x0, -0x4(r4)
4460:  3e40 fcff      mov	#0xfffc, r14
4464:  0e54           add	r4, r14 // r14 += r4 
4466:  0e12           push	r14 // Password length
4468:  0f12           push	r15 // The password
446a:  3012 7d00      push	#0x7d // Check password by HSM
446e:  b012 7a45      call	#0x457a <INT>
4472:  5f44 fcff      mov.b	-0x4(r4), r15
4476:  8f11           sxt	r15
4478:  3152           add	#0x8, sp
447a:  3441           pop	r4
447c:  3041           ret