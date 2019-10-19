## picoCTF 2018
### quackme - Points: 200

>Can you deal with the Duck Web? Get us the flag from [this](https://2018shell.picoctf.com/static/197682216dbdd2c67ae043dc531d4cfc/main) program. You can also find the program in /problems/quackme_2_45804bbb593f90c3b4cefabe60c1c4e2.

The way to solve this challenge is inspired by this [blog](https://ctf-wiki.github.io/ctf-tools/binary_core_tools/instrumentation/intel_pin/).

Basically it uses a tool called [pin](https://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool) from Intel which has various plugins (.so), one of which can show the instructions executed given an input to a program.

In this challenge, the program being benchmarked is `main`, and **_the theory that below code is based upon is the fact that, in almost all cases that we've seen, the program tends to execute more instructions when it is given the correct input._**

For this binary, given the correct input, the program executes one more instruction than the wrong input.

when the program execution reaches to the following point,

`0x080486f3 <+177>:   jne    0x8048707 <do_magic+197>`

**if ne is true, then the program will execute +197,201,204,207,209**

**otherwise it will do +179,182,187,192,195,209**

```assembly
 0x080486f3 <+177>:   jne    0x8048707 <do_magic+197>
 0x080486f5 <+179>:   sub    esp,0xc
 0x080486f8 <+182>:   push   0x80488ab
 0x080486fd <+187>:   call   0x8048470 <puts@plt>
 0x08048702 <+192>:   add    esp,0x10
 0x08048705 <+195>:   jmp    0x8048713 <do_magic+209>
 0x08048707 <+197>:   add    DWORD PTR [ebp-0x18],0x1
 0x0804870b <+201>:   mov    eax,DWORD PTR [ebp-0x18]
 0x0804870e <+204>:   cmp    eax,DWORD PTR [ebp-0x10]
 0x08048711 <+207>:   jl     0x80486bd <do_magic+123>
 0x08048713 <+209>:   leave
 0x08048714 <+210>:   ret
```

Based upon this observation, we can use the following code to solve this challenge.

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import datetime
from subprocess import Popen, PIPE

max_counter = []
flag = ""
output = ""

pinPath = "/home/ubuntu/intel-pin-ctf/pin-3.11-97998-g7ecce2dac-gcc-linux/pin"
pinInit = lambda tool, elf: Popen([pinPath, '-t', tool, '--', elf], stdin = PIPE, stdout = PIPE)
pinWrite = lambda cont: pin.stdin.write(cont)
pinRead = lambda : pin.communicate()[0]

print("Start time: ", datetime.datetime.now())

# stop when the "You are winner!" message is printed in the output
while ("You are winner!" not in output):
    for i in range(32, 127):
        ascii_char = chr(i)
        pin = pinInit("./inscount0.so", "./main")

        pinWrite((flag+ascii_char).encode("utf-8"))
        output = pinRead().decode("utf-8")
        if ("You are winner!" in output):
            print("This is the winner output: " + output)
            flag_candidate = ascii_char
            break

        with open("./inscount.out", "r") as f:
            line = f.readline()

        counter = line.split()[1]
        print("Testing ", ascii_char, ", The corresponding counter is: ", counter)

        if len(max_counter) == 0:
            max_counter.append(counter)
            continue

        # if the next counter is greater than the previous one, overwrite the previous counter
        if counter > max_counter[0]:
            max_counter[0] = counter
            flag_candidate = ascii_char

    flag += flag_candidate

    print("#" * 10 + "\n" + "Flag is now: " + flag + "\n" + "#" * 10)

print("Flag found! " + flag)
print("End time: ", datetime.datetime.now())
```
