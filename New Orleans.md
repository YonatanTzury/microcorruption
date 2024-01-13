
# Generate password:
	4482:  ff40 7d00 0000 mov.b	#0x7d, 0x0(r15)
	4488:  ff40 2d00 0100 mov.b	#0x2d, 0x1(r15)
	448e:  ff40 6e00 0200 mov.b	#0x6e, 0x2(r15)
	4494:  ff40 5700 0300 mov.b	#0x57, 0x3(r15)
	449a:  ff40 3a00 0400 mov.b	#0x3a, 0x4(r15)
	44a0:  ff40 2900 0500 mov.b	#0x29, 0x5(r15)
	44a6:  ff40 7200 0600 mov.b	#0x72, 0x6(r15)
	44ac:  cf43 0700      mov.b	#0x0, 0x7(r15)
	
	0x2400 = 7d2d 6e57 3a29 7200

@R13 - the entered password: 439c
R14 = Counter
44bc: <check_password>
44bc:  0e43           clr	Counter       // Counter = 0
44be:  0d4f           mov	r15, r13  // r15 = r13
44c0:  0d5e           add	Counter, r13  // r13 += Counter
44c2:  ee9d 0024      cmp.b	@r13, 0x2400(Counter) // Comparer index in password to index in input
44c6:  0520           jnz	$+0xc <check_password+0x16> // 0x44d2 - return
44c8:  1e53           inc	Counter // Counter += 1
44ca:  3e92           cmp	#0x8, Counter // password length is 8
44cc:  f823           jnz	$-0xe <check_password+0x2> // 0x44be - continue loop
44ce:  1f43           mov	#0x1, r15
44d0:  3041           ret
44d2:  0f43           clr	r15
44d4:  3041           ret