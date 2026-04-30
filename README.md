LP11 — Windows Defender Firewall prin PowerShell
Ghid complet cu comenzi în ordine și indicații pentru screenshots
IP Ubuntu: 10.242.11.3 | Rețea locală: 10.242.11.0/24
PASUL 0 — Deschizi PowerShell ca Administrator
Click dreapta pe butonul Start → Windows PowerShell (Administrator) → Yes la UAC.
 SCREENSHOT 1 — fereastra PowerShell cu titlul 'Administrator: Windows PowerShell'
PASUL 1 — Verifici starea inițială a firewall-ului
Rulezi această comandă să vezi cum este configurat firewall-ul înainte de orice modificare:
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction,
DefaultOutboundAction
 SCREENSHOT 2 — cele 3 profile (Domain, Private, Public) cu starea lor inițială
PASUL 2 — Activezi firewall-ul și setezi politica implicită
Rulezi pe rând, câte una:
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True
Set-NetFirewallProfile -Profile Domain,Private,Public -DefaultInboundAction Block
Set-NetFirewallProfile -Profile Domain,Private,Public -DefaultOutboundAction Allow
⚠️ ATENȚIE: Prima comandă activează firewall-ul. A doua blochează tot traficul intrant. A treia
permite traficul ieșit. Nu le confunda!
Verifici că s-a aplicat corect:
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction,
DefaultOutboundAction
 SCREENSHOT 3 — DefaultInboundAction: Block la toate 3 profilele
PASUL 3 — Regula ICMP (ping) permis doar pe profil Private
Această regulă permite ping-ul doar când ești conectat la o rețea privată (LAN). Pe profil Public
ping-ul rămâne blocat.
New-NetFirewallRule -DisplayName "LP11 Allow ICMPv4 In Private" -Direction Inbound -Protocol
ICMPv4 -IcmpType 8 -Action Allow -Profile Private
⚠️ Nu adăuga -ICMPv4 ca parametru separat — va da eroare. Comanda de mai sus este corectă.
 SCREENSHOT 4 — regula creată fără text roșu de eroare
PASUL 4 — Regula RDP permis doar din rețeaua locală
RDP (Remote Desktop Protocol) pe portul 3389. Permitem accesul doar din rețeaua
10.242.11.0/24 pentru a evita expunerea la internet.
New-NetFirewallRule -DisplayName "LP11 Allow RDP LAN" -Direction Inbound -Protocol TCP
-LocalPort 3389 -RemoteAddress 10.242.11.0/24 -Action Allow -Profile Private
 SCREENSHOT 5 — regula LP11 Allow RDP LAN creată
PASUL 5 — Blocare SMB inbound (port 445)
SMB este protocolul de partajare fișiere Windows. A fost exploatat de ransomware (WannaCry,
NotPetya). Îl blocăm pe traficul intrant.
New-NetFirewallRule -DisplayName "LP11 Block SMB In" -Direction Inbound -Protocol TCP
-LocalPort 445 -Action Block
 SCREENSHOT 6 — regula LP11 Block SMB In creată
PASUL 6 — Blocare Telnet outbound (port 23)
Telnet trimite parolele necriptat prin rețea. Blocăm traficul ieșit pe portul 23 ca să prevenim
utilizarea accidentală.
New-NetFirewallRule -DisplayName "LP11 Block Telnet Out" -Direction Outbound -Protocol TCP
-RemotePort 23 -Action Block
 SCREENSHOT 7 — regula LP11 Block Telnet Out creată
PASUL 7 — Blocare IP suspect outbound
Simulăm blocarea unui server C2 (command & control) de malware. IP-ul 203.0.113.10 este
rezervat pentru documentație/teste — nu e real.
New-NetFirewallRule -DisplayName "LP11 Block Suspicious IP Out" -Direction Outbound
-RemoteAddress 203.0.113.10 -Action Block
 SCREENSHOT 8 — regula LP11 Block Suspicious IP Out creată
PASUL 8 — Verifici toate regulile LP11 create
Afișezi doar regulile create de tine (toate încep cu LP11):
Get-NetFirewallRule | Where-Object DisplayName -like "LP11*" | Select-Object DisplayName,
Enabled, Direction, Action
 SCREENSHOT 9 — toate 4 regulile LP11 listate cu Direction și Action
PASUL 9 — Testarea regulilor
Test 1 — RDP (trebuie să meargă din LAN):
Test-NetConnection 10.242.11.3 -Port 3389
Rezultat așteptat: TcpTestSucceeded : True
 SCREENSHOT 10 — test RDP reușit
Test 2 — SMB blocat:
Test-NetConnection 10.242.11.3 -Port 445
Rezultat așteptat: TcpTestSucceeded : False
 SCREENSHOT 11 — test SMB blocat
Test 3 — Telnet outbound blocat:
Test-NetConnection 10.242.11.3 -Port 23
Rezultat așteptat: TcpTestSucceeded : False
 SCREENSHOT 12 — test Telnet blocat
Test 4 — Ping (ICMP) pe profil Private:
ping 10.242.11.3
Rezultat așteptat: Reply from 10.242.11.3 (funcționează pe profil Private)
 SCREENSHOT 13 — ping reușit
PASUL 10 — Sarcina 7: Ștergi o regulă (înainte și după)
Mai întâi faci captură ÎNAINTE:
Get-NetFirewallRule | Where-Object DisplayName -like "LP11*" | Select-Object DisplayName,
Enabled, Direction, Action
 SCREENSHOT 14 (ÎNAINTE) — lista completă cu toate regulile LP11
Ștergi regula cu IP-ul suspect:
Remove-NetFirewallRule -DisplayName "LP11 Block Suspicious IP Out"
Verifici că a dispărut:
Get-NetFirewallRule | Where-Object DisplayName -like "LP11*" | Select-Object DisplayName,
Enabled, Direction, Action
 SCREENSHOT 15 (DUPĂ) — lista cu 3 reguli rămase (fără cea ștearsă)
PASUL 11 — Verifici profilurile finale
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction,
DefaultOutboundAction
 SCREENSHOT 16 — starea finală: Block inbound, Allow outbound la toate profilele
PASUL 12 — Curățare după laborator
Ștergi toate regulile LP11 rămase:
Get-NetFirewallRule | Where-Object DisplayName -like "LP11*" | Remove-NetFirewallRule
Verifici că lista e goală:
Get-NetFirewallRule | Where-Object DisplayName -like "LP11*"
 SCREENSHOT 17 — niciun rezultat = toate regulile LP11 au fost șterse
Rezumat Screenshots Windows
# Ce conține screenshot-ul Pasul
1 PowerShell deschis ca Administrator Pasul 0
2 Profile firewall — starea inițială Pasul 1
3 Profile — DefaultInboundAction: Block confirmat Pasul 2
4 Regula ICMP (ping) creată fără eroare Pasul 3
5 Regula RDP din LAN creată Pasul 4
6 Regula SMB blocat creată Pasul 5
7 Regula Telnet blocat creată Pasul 6
8 Regula IP suspect creată Pasul 7
9 Toate regulile LP11 listate Pasul 8
10 Test RDP — TcpTestSucceeded : True Pasul 9
11 Test SMB — TcpTestSucceeded : False Pasul 9
12 Test Telnet — TcpTestSucceeded : False Pasul 9
13 Ping reușit spre 10.242.11.3 Pasul 9
14 Reguli LP11 ÎNAINTE de ștergere (4 reguli) Pasul 10
15 Reguli LP11 DUPĂ ștergere (3 reguli) Pasul 10
16 Profile firewall — starea finală Pasul 11
17 Lista goală — toate regulile LP11 șterse Pasul 12
TI-242 | Frunza Cristian | LP11 Firewall UFW și PowerShell
