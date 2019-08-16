We're given a binary with these charecteristics:
rabin2.exe -I virtual_toy 
arch     x86                
binsz    15288              
bintype  elf                
bits     32
[...]

Upon execution the binary will print a welcome message along with a pompt to enter the access code, giving a random input "ERROR"
is printed to stdout and the process terminates.

Opening the binary in BINJA reveals a standard main function, along with two interesting functions "PUSH" and "POP", this along
with the challenge title gives a good indication that it's a vm that emulates a program.
The main function is shown to print the aforementioned welcome message followed by a relatively big switch case:
![](https://imgur.com/eJFJmEk)
Finding the starting point of the IP, we can copy out the program code, and start to convert the switchcase to python.
It also becomes evident that the VM is stack based due to all the instructions utilising POP and PUSH, with common operations
inbetween, such as:
![](https://imgur.com/nZLjwfD)
with the switch statement endning in a push eax, inc ip.

Implementing all this into a crude dissasembler produces the following assembly:
```
PUSH: 0
PUSH: 12596   ;; PUSH constants
[... left out for brevity ...]
PUSH: 13072
PUSH: 13240
XXX                  ;; Unknown
TEST 5            ;; Test if stack value is 0
PUSH: 68        ;; Jump to 68 if it is
JMP 68 0         ;;---------------------------
READ:             ;; READ one char of stdin or the equivilant buffer
PUSH: 10
ADD 10 65       ;; ADD 10 to it
PUSH: 1337
MUL 1337 75   ;; Multiply by 1337
PUSH: 191
DIV 191 100275 ;; IDIV by 191
PUSH: 4095
XOR 4095 525   ;; XOR with 4095
NEG 3570        ;; NEGative the number
NOT -3570        ;; perform a bitwise not
PUSH: 2
SHL 2 3569      ;; Shift left by 2
XOR 14276 13240 ;; XOR with the stackvalue as pushed by the first instructions of the program
TEST 1148          ;; Check if the result is 0 (eg, check that 14276 == 13240)
PUSH: 33
JMP 33 0            ;; Loop back if it is
PUSH: 69
PRINT E             ;; Otherwise print "Error" and terminate
PUSH: 114
PRINT r
PUSH: 114
PRINT r
PUSH: 111
PRINT o
PUSH: 114
PRINT r
PUSH: 10
PRINT
```
the XXX instruction i did not disassemble and it turns out to be irrelevant as long as a non zero number is pushed to the stack in
place of it.
All that is left is reversing the above sequence of ops and perform it on the pushed constants to get the password, the following
python code accomplishes that:
```
for i in password:
a = (i>>2)&0xfffffffff
a = ~a
a = 0-a
a = a^4095
a = a*191
a = a//1337
a = a-10
print(chr(a), end='')
```
Doing this and we get the flag: flag{qemu_is_better_09831912393}


This is the disassembler and emulator:
```
ip = 0

stack = []

program = [
0x00000011, 0x00000000, 0x00000011, 0x00003134,
0x00000011, 0x0000394c, 0x00000011, 0x000038a4,
0x00000011, 0x0000394c, 0x00000011, 0x00003968,
0x00000011, 0x00003984, 0x00000011, 0x000038a4,
0x00000011, 0x00003984, 0x00000011, 0x0000394c,
0x00000011, 0x000038c0, 0x00000011, 0x000038a4,
0x00000011, 0x000039a0, 0x00000011, 0x0000347c,
0x00000011, 0x00003268, 0x00000011, 0x000033d4,
0x00000011, 0x00003230, 0x00000011, 0x00003230,
0x00000011, 0x000033d4, 0x00000011, 0x00003428,
0x00000011, 0x0000347c, 0x00000011, 0x0000324c,
0x00000011, 0x00003364, 0x00000011, 0x0000347c,
0x00000011, 0x00003214, 0x00000011, 0x000032f4,
0x00000011, 0x000033d4, 0x00000011, 0x00003284,
0x00000011, 0x0000316c, 0x00000011, 0x0000339c,
0x00000011, 0x00003444, 0x00000011, 0x00003310,
0x00000011, 0x000033b8, 0x00000013, 0xdeadc0de,
0x0000000b, 0xdeadc0de, 0x00000011, 0x00000044,
0x00000010, 0xdeadc0de, 0x00000000, 0xdeadc0de,
0x00000011, 0x0000000a, 0x00000002, 0xdeadc0de,
0x00000011, 0x00000539, 0x00000004, 0xdeadc0de,
0x00000011, 0x000000bf, 0x00000005, 0xdeadc0de,
0x00000011, 0x00000fff, 0x00000009, 0xdeadc0de,
0x0000000c, 0xdeadc0de, 0x0000000a, 0xdeadc0de,
0x00000011, 0x00000002, 0x0000000e, 0xdeadc0de,
0x00000009, 0xdeadc0de, 0x0000000b, 0xdeadc0de,
0x00000011, 0x00000021, 0x00000010, 0xdeadc0de,
0x00000011, 0x00000045, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000072, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000072, 0x00000001, 0xdeadc0de,
0x00000011, 0x0000006f, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000072, 0x00000001, 0xdeadc0de,
0x00000011, 0x0000000a, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000001, 0x00000014, 0xdeadc0de,
0x00000011, 0x00000041, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000063, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000063, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000065, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000073, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000073, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000020, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000067, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000072, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000061, 0x00000001, 0xdeadc0de,
0x00000011, 0x0000006e, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000074, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000065, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000064, 0x00000001, 0xdeadc0de,
0x00000011, 0x0000000a, 0x00000001, 0xdeadc0de,
0x00000011, 0x00000000, 0x00000014, 0xdeadc0de]

inputA = [0x41, 0x41, 0x41, 0x41]
while ip in range(0,len(program)):
   jmped = False
    #print("OP: " + str(hex(program[ip])))
    op = program[ip]
    #print("IMM: " + str(hex(program[ip+1])))
    imm = program[ip+1]

    if op == 0x1:
        print("PRINT " + str(chr(stack.pop())))
    elif op == 0x0:
        print ("READ: ")
        stack.append(inputA.pop())
    elif op == 0x2:
        a = stack.pop()
        b = stack.pop()
        stack.append(a+b)
        print("ADD " + str(a) + " " + str(b))
    
    elif op == 0x4:
        a = stack.pop()
        b = stack.pop()
        stack.append(a*b)
        print("MUL " + str(a) + " " + str(b))
    
    elif op == 0x5:
        a = stack.pop()
        b = stack.pop()
        stack.append(b//a)
        print("DIV " + str(a) + " " + str(b))
    
    elif op == 0x9:
        a = stack.pop()
        b = stack.pop()
        stack.append(a^b)
        print("XOR " + str(a) + " " + str(b))

    elif op == 0xa:
        a = stack.pop()
        stack.append(~a)
        print("NOT " + str(a))

    elif op == 0xb:
        a = stack.pop()
        if a == 0:
            stack.append(1)
        else:
            stack.append(0)
        print("TEST " + str(a))
    elif op == 0xc:
        a = stack.pop()
        stack.append(0-a)
        print("NEG " + str(a))
    elif op == 0xe:
        a = stack.pop()
        b = stack.pop()
        stack.append((b<<a)&0xffffffff)
        print("SHL " + str(a) + " " + str(b))
    elif op == 0x10:
        a = stack.pop()
        b = stack.pop()
        if b != 0:
            ip = a*2
            jmped = True
        else:
            ip = ip+2
            jmped = True
        print("JMP " + str(a) + " " + str(b))

    elif op == 0x11:
        stack.append(program[ip+1])
        print("PUSH: " + str(program[ip+1]))
    elif op == 0x13:
        stack.append(0x5)
        print("Something")
    else: 
        print("Not implemented op:" + str(hex(op)))
        print(stack)
        print("IP: " + str(hex(ip)))
        print("IP->" + str(hex(program[ip])))
        exit()
    if jmped != True:    
        ip = ip + 2
    #print(stack)
```
