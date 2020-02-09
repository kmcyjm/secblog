> As you enjoy this music even more, another executable be-quick-or-be-dead-2 shows up. 
> Can you run this fast enough too? You can also find the executable in /problems/be-quick-or-be-dead-2_0_04f4c579185361da6918bbc2fc9dcb7b.

Executing the program emits the following output,

```
$ ./be-quick-or-be-dead-2
Be Quick Or Be Dead 2
=====================

Calculating key...
You need a faster machine. Bye bye.
```

Looking at `main`, we can see there is a function `set_timer`. It's job is to send a [SIGALRM](http://man7.org/linux/man-pages/man7/signal.7.html) to a signal handler afer a few seconds, which is to terminate the program.

```assembly
000000000040085f <main>:
  40085f:	55                   	push   rbp
  400860:	48 89 e5             	mov    rbp,rsp
  400863:	48 83 ec 10          	sub    rsp,0x10
  400867:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
  40086a:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
  40086e:	b8 00 00 00 00       	mov    eax,0x0
  400873:	e8 a9 ff ff ff       	call   400821 <header>
  400878:	b8 00 00 00 00       	mov    eax,0x0
  40087d:	e8 f8 fe ff ff       	call   40077a <set_timer>
  400882:	b8 00 00 00 00       	mov    eax,0x0
  400887:	e8 42 ff ff ff       	call   4007ce <get_key>
  40088c:	b8 00 00 00 00       	mov    eax,0x0
  400891:	e8 63 ff ff ff       	call   4007f9 <print_flag>
  400896:	b8 00 00 00 00       	mov    eax,0x0
  40089b:	c9                   	leave  
  40089c:	c3                   	ret    
  40089d:	0f 1f 00             	nop    DWORD PTR [rax]
```

With `gdb-peda`, we can use `deactive alarm` to deactivate the alarm function, so the program won't quit after the time preset in the program. This allows us enough time to play with the program later on.

The `get_key` function looks quite relevant to our challenge, after diving into it we can see it calls another function `calculate_key`, which in turn calls another function `fib` with one parameter `0x3f7` (1015 in decimal).

```assembly
000000000040074b <calculate_key>:
  40074b:	55                   	push   rbp
  40074c:	48 89 e5             	mov    rbp,rsp
  40074f:	bf f7 03 00 00       	mov    edi,0x3f7
  400754:	e8 ad ff ff ff       	call   400706 <fib>
  400759:	5d                   	pop    rbp
  40075a:	c3                   	ret    
```

Further looking at the `fib` function, we can see it is used to calculate the fibonassi sequence.

It basically calculates the 1015th number in the fibonassi sequence.

```assembly
0000000000400706 <fib>:
  400706:	55                   	push   rbp
  400707:	48 89 e5             	mov    rbp,rsp
  40070a:	53                   	push   rbx
  40070b:	48 83 ec 28          	sub    rsp,0x28
  40070f:	89 7d dc             	mov    DWORD PTR [rbp-0x24],edi
  400712:	83 7d dc 01          	cmp    DWORD PTR [rbp-0x24],0x1
  400716:	77 08                	ja     400720 <fib+0x1a>
  
  400718:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24] ; when n = 0 | 1
  40071b:	89 45 ec             	mov    DWORD PTR [rbp-0x14],eax ; move n directly to $rbp-0x14 where the sum is stored
  40071e:	eb 21                	jmp    400741 <fib+0x3b>
  
  400720:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24] ; when n > 1, move n to eax
  400723:	83 e8 01             	sub    eax,0x1 ; n-1
  400726:	89 c7                	mov    edi,eax ; set 1st parameter to n-1
  400728:	e8 d9 ff ff ff       	call   400706 <fib>
  
  40072d:	89 c3                	mov    ebx,eax ; save result of fib(n-1) to ebx
  
  40072f:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24] ; fib(n-2)
  400732:	83 e8 02             	sub    eax,0x2 ; n-2
  400735:	89 c7                	mov    edi,eax ; set 1st parameter to n-2
  400737:	e8 ca ff ff ff       	call   400706 <fib>
  
  40073c:	01 d8                	add    eax,ebx ; fib(n-2) + fib(n-1)
  40073e:	89 45 ec             	mov    DWORD PTR [rbp-0x14],eax
  
  400741:	8b 45 ec             	mov    eax,DWORD PTR [rbp-0x14]
  400744:	48 83 c4 28          	add    rsp,0x28
  400748:	5b                   	pop    rbx
  400749:	5d                   	pop    rbp
  40074a:	c3                   	ret    
```

If we let the program continue at this point, it will take ages to finish. 

At this point, we can actually look up online to see what the 1015th fibonassi number is. It turns out is a quit huge number,

```
59288416551943338727574080408572281287377451615227988184724603969919549034666922046325034891393072356252090591628758887874047734579886068667306295291967872198822088710569576575629665781687543564318377549435421485

Hex
0xb457201f4f2ec4e640ca1f499492e0246038fe0c8d43e5c09a32f1c5cf705417439dfd6390c7c73d984ce0d2701dca8aca1858a08fdcbd64c83ed2594d50080c62f0f9c0bc7b00554806f0daa0a63229f51f7eb6d73ec32d
```

From the disassembly of `fib` we can see this final number is stored in `eax` when `fib` returns. However, `eax` only can store 4 bytes, therefore, only the least significant 4 bytes on above number is returned, which is `0xd73ec32d`.

This hex number is then being stored at `0x6010c0`.

```assembly
00000000004007ce <get_key>:
  4007ce:	55                   	push   rbp
  4007cf:	48 89 e5             	mov    rbp,rsp
  4007d2:	bf b8 09 40 00       	mov    edi,0x4009b8
  4007d7:	e8 54 fd ff ff       	call   400530 <puts@plt>
  4007dc:	b8 00 00 00 00       	mov    eax,0x0
  4007e1:	e8 65 ff ff ff       	call   40074b <calculate_key>
  4007e6:	89 05 d4 08 20 00    	mov    DWORD PTR [rip+0x2008d4],eax        # 6010c0 <__TMC_END__>, result of fib(1015)
  4007ec:	bf cb 09 40 00       	mov    edi,0x4009cb ; "Done caculating key"
  4007f1:	e8 3a fd ff ff       	call   400530 <puts@plt>
  4007f6:	90                   	nop
  4007f7:	5d                   	pop    rbp
  4007f8:	c3                   	ret    
```

It is then in turn passed to `decrypt_flag` function to calculate the flag.

```assembly
00000000004007f9 <print_flag>:
  4007f9:	55                   	push   rbp
  4007fa:	48 89 e5             	mov    rbp,rsp
  4007fd:	bf e0 09 40 00       	mov    edi,0x4009e0
  400802:	e8 29 fd ff ff       	call   400530 <puts@plt>
  400807:	8b 05 b3 08 20 00    	mov    eax,DWORD PTR [rip+0x2008b3]        # 6010c0 <__TMC_END__>
  40080d:	89 c7                	mov    edi,eax
  40080f:	e8 82 fe ff ff       	call   400696 <decrypt_flag>
  400814:	bf 80 10 60 00       	mov    edi,0x601080
  400819:	e8 12 fd ff ff       	call   400530 <puts@plt>
  40081e:	90                   	nop
  40081f:	5d                   	pop    rbp
  400820:	c3                   	ret    
```

A brief overview of the `decrypt_flag` function reveals that it does some shifts and xors to calculate the flag, and the `puts` function above will probably print the flag out.

Therefore, at this point, it is not necessary to understand what this function does. We just need to set the eax to the correct value shown above, and let it run to finish.

It will then print out the flag,
`picoCTF{the_fibonacci_sequence_can_be_done_fast_73e2451e}`
