# nmap

nmap -n -Pn -p- 192.168.91.26
nmap -n -Pn -A -p 22,80 192.168.91.26
nmap -n -Pn -sC -sV -O -p 22,80 192.168.91.26
nmap -n -Pn --script vuln -p 80,135,139,445,49664-49676 192.168.89.45
nmap -A -Pn -p- 192.168.91.46
	-sS	TCP SYN scan
	-sC	scripts default
			--script default, all, auth, vuln
	-sV	service versions
	-O		OS Detection
	-A		-O -sV --traceroute
	-T5	insane timing template
	-Pn	no ping scan
	-p-		all ports
sudo nmap -sS -sC -sV -O -p- 192.168.91.46

# nc

nc.traditional -n -w 1 -z -v 192.168.89.19 180-190 2>&1 | grep open
	-w		timeout
	-z		just scan (dont send data)
	-v		verbose
	-n		no dns lookup
echo "" | nc -zv -w1 <alvo> <range portas> 2>&1 | grep succeeded

# wpscan
wpscan --url http://192.168.90.90/wp/

# Brute Force

## Wordlists
ls -lh /usr/share/wordlists/

locate rockyou.txt
gunzip /usr/share/wordlists/rockyou.txt.gz

wordlists -h

## Tools

medusa -h 192.168.90.90 -u someone -P /home/path/rockyou.txt -M ssh

# HTTP

python3 -m http.server 8080
python2 -m SimpleHTTPServer 8080
ruby -run -e httpd . -p 8080
php -S 0.0.0.0:8080
php -S 0.0.0.0:8080 -c php.ini

php.ini
> disable_functions =exec
> upload_max_filesize = 1000M;
> post_max_size = 1000M;

# FTP

Servidor FTP na porta 2121
python3 -m pyftpdlib

Servidor FTP na porta 21
sudo python3 -m pyftpdlib -p21

Servidor FTP com permissão de escrita
python3 -m pyftpdlib -w

# Metsploit

