448a <check_password>
448a:  bf90 5634 0000 cmp	#0x3456, 0x0(r15)
4490:  0d20           jnz	$+0x1c <check_password+0x22> // return False
4492:  bf90 7370 0200 cmp	#0x7073, 0x2(r15)
4498:  0920           jnz	$+0x14 <check_password+0x22> // return False
449a:  bf90 4367 0400 cmp	#0x6743, 0x4(r15)
44a0:  0520           jnz	$+0xc <check_password+0x22> // return False
44a2:  1e43           mov	#0x1, r14
44a4:  bf90 556e 0600 cmp	#0x6e55, 0x6(r15)
44aa:  0124           jz	$+0x4 <check_password+0x24> // return True
44ac:  0e43           clr	r14
44ae:  0f4e           mov	r14, r15
44b0:  3041           ret

# LITTLE ENDIAN!
pass: 5634 7370 4367 556e