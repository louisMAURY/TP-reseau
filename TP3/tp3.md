# B1 Réseau 2018 -TP3
# I. Création et utilisation simples d'une VM CentOS
## 4. Configuration réseau d'une machine CentOS

- A FAIRE :
    - > *a) Comment je sais si je suis connecté à Internet sur ma VM ?*

    Il suffit de taper la commande `curl -SL www.google.com` (google est un exemple)

    ```console
    [louis@localhost_etc]$ curl -SL www.google.com
    ```
    Et si la console ne nous renvoie pas une erreur, il y a bien une connection à internet.
    ___
    - > *b) Comment je fais pour que ma VM et mon PC communiquent ?*

    Pour prouver que le PC et la VM peuvent  communiquer il suffit de faire un `ping` du PC vers la VM et inversement. Si les deux fonctionne, alors le PC et la VM peuvent communiquer ensemble.

    Pour le `ping` du **PC** vers la **VW** :
    ```console
        C:\Users\louis>ping 192.168.127.10

        Envoi d'une requête 'ping' 192.168.127.10 avec 32 octets de données:
        Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
        Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
        Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
        Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64

        Statistiques Ping pour 192.168.127.10:
            Paquets : envoyés = 4, reçus = 4, perdus = 0(perte 0%),
        Durée approximative des boucles en millisecondes :
            Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
    ```
        
    Le ping entre le PC et la VM fonctionne.

    - Pour le `ping` de la **VM** vers le **PC** :
        ```console
        [louis@localhost_etc]$ ping 192.168.1.21
        PING 192.168.1.21 (192.168.1.21) 56(84) bytes of data.
        64 bytes from 192.168.1.21: icmp_seq=1 ttl=127 time=0.736 ms
        64 bytes from 192.168.1.21: icmp_seq=2 ttl=127 time=1.00 ms
        64 bytes from 192.168.1.21: icmp_seq=3 ttl=127 time=0.998 ms
        64 bytes from 192.168.1.21: icmp_seq=4 ttl=127 time=0.660 ms

        --- 192.168.1.21 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3007ms
        rtt min/avg/max/mdev = 0.660/0.849/1.004/0.156 ms
        ```
        Le ping entre la VM et le PC fonctionne également.
        Cela veut donc dire que le PC et la VM peuvent communiquer.
    ___
    - > *c) Comment afficher sa table de routage ?*
    
    Pour afficher la table de routage il suffit de rentrer la commande `ip route`:
    ```console
    [louis@localhost ~]$ iproute
    default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
    192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 101
    ```
    > *À quoi ça correspond ce charabiat ?*
    
    Les deux premières lignes correspondent à la connection entre ma VM et mon PC et la troisième ligne correspond à l'adresse de ma VM dans mon réseau.
---
---
# II. Notion de ports et SSH
## 3. Firewall.
---
- A. SSH:
    > *En modifiant le fichier sshd_config, j'ai changé le port 22 sur lequel le serveur SSH écoute en un port 2222. J'ai ensuite redémarrer le serveur SSH pour que le changement se fasse. J'ouvre PuTTy et essai de me connecter au server sur le port 2222 mais ça ne fonctionne pas ! Comment ça se fait ?*

    C'est normal ! C'est parceque, par défaut, le pare-feu de la VM n'autorise pas les connections en SSH sur d'autre ports que le n°22.

    > *Et comment je fais pour qu'il puisse m'autoriser sur un autre port ?*

    Il suffit tout simplement de taper les commandes suivante:
    ```console
    [louis@localhost ~]$ sudo firewall-cmd --add-port=2222/tcp --permanent
    [louis@localhost ~]$ sudo firewall-cmd --reload
    success
    [louis@localhost ~]$ sudo firewall-cmd --list-all
    ports: 2222/tcp
    ```
    > *Qu'est-ce que ça veut dire tout ça !?*

    J'allais expliquer !
    
    Donc la **première commande**, c'est pour dire au pare-feu d'**accepter la communication** sur le port 2222 de façon permanente. La **deuxième commande**, c'est pour **relancer le pare-feu** et faire en sorte que ça fonctionne. La **troisième commande** c'est pour **vérifier** si le pare-feu à bien autorisé la communication sur le port qui nous intéresse.

    > *Ah oui je comprends mieux et ça marche en plus !*

---
---
# III. Routage statique

## 1. Préparation des hôtes (vos PCs)
---

### Ping à travers le cable ethernet

```powershell
PS C:\Users\Notitou> ping 192.168.255.1

Envoi d’une requête 'Ping'  192.168.255.1 avec 32 octets de données :
Réponse de 192.168.255.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.255.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.255.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.255.1 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 192.168.255.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
```

Changement d'ip sur les cartes ethernet pour être dans le réseau 192.168.112.0/30

---
### Préparation Virtual Box

Rémi et Louis ont changé leurs IP de leur carte Host-Only et de leur VM par celle recquise.

---
### Check

Ping PC 1 -> PC 2
```powershell
PS C:\Users\Notitou> ping 192.168.112.2

Envoi d’une requête 'Ping'  192.168.112.2 avec 32 octets de données :
Réponse de 192.168.112.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.112.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.112.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.112.2 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.112.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 2ms, Moyenne = 2ms
```

Ping PC 1 -> VM 1

```powershell
PS C:\Users\Notitou> ping 192.168.101.10

Envoi d’une requête 'Ping'  192.168.101.10 avec 32 octets de données :
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.101.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```

Ping VM1 -> PC1

```bash
[root@localhost ~]# ping 192.168.101.1
PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
64 bytes from 192.168.101.1: icmp_seq=1 ttl=128 time=0.270 ms
64 bytes from 192.168.101.1: icmp_seq=2 ttl=128 time=0.684 ms
64 bytes from 192.168.101.1: icmp_seq=3 ttl=128 time=0.401 ms
64 bytes from 192.168.101.1: icmp_seq=4 ttl=128 time=0.409 ms
64 bytes from 192.168.101.1: icmp_seq=5 ttl=128 time=0.328 ms
64 bytes from 192.168.101.1: icmp_seq=6 ttl=128 time=0.452 ms
^X^C
--- 192.168.101.1 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5002ms
rtt min/avg/max/mdev = 0.270/0.424/0.684/0.130 ms
```

Ping PC2 -> PC1

```powershell
PS C:\Users\louis_tbixp39> ping 192.168.112.1

Envoi d’une requête 'Ping'  192.168.112.1 avec 32 octets de données :
Réponse de 192.168.112.1 : octets=32 temps=3 ms TTL=128
Réponse de 192.168.112.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.112.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.112.1 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.112.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 3ms, Moyenne = 2ms
```

Ping VM1 -> PC1

```bash
[root@localhost ~]# ping 192.168.101.1
PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
64 bytes from 192.168.101.1: icmp_seq=1 ttl=128 time=0.270 ms
64 bytes from 192.168.101.1: icmp_seq=2 ttl=128 time=0.684 ms
64 bytes from 192.168.101.1: icmp_seq=3 ttl=128 time=0.401 ms
64 bytes from 192.168.101.1: icmp_seq=4 ttl=128 time=0.409 ms
64 bytes from 192.168.101.1: icmp_seq=5 ttl=128 time=0.328 ms
64 bytes from 192.168.101.1: icmp_seq=6 ttl=128 time=0.452 ms
^X^C
--- 192.168.101.1 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5002ms
rtt min/avg/max/mdev = 0.270/0.424/0.684/0.130 ms
```
Ping VW2 -> PC2

```bash
[louis@localhost ~]$ ping 192.168.112.2                                         PING 192.168.112.2 (192.168.112.2) 56(84) bytes of data.
64 bytes from 192.168.112.2: icmp_seq=1 ttl=127 time=0.483 ms
64 bytes from 192.168.112.2: icmp_seq=2 ttl=127 time=1.05 ms
64 bytes from 192.168.112.2: icmp_seq=3 ttl=127 time=1.23 ms
64 bytes from 192.168.112.2: icmp_seq=4 ttl=127 time=1.16 ms
^C
--- 192.168.112.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 0.483/0.985/1.238/0.299 ms
```
Ping PC2 -> VM2

```powershell
PS C:\Users\louis_tbixp39> ping 192.168.102.10

Envoi d’une requête 'Ping'  192.168.102.10 avec 32 octets de données :
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.102.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```
---
## 2. Configuration du routage

- PC 1

```Powershell
PS C:\WINDOWS\system32> route add 192.168.102.1/24 mask 255.255.255.0 192.168.112.2
 OK!
PS C:\WINDOWS\system32> ping 192.168.102.1

Envoi d’une requête 'Ping'  192.168.102.1 avec 32 octets de données :
Réponse de 192.168.102.1 : octets=32 temps=2 ms TTL=127
Réponse de 192.168.102.1 : octets=32 temps=2 ms TTL=127
Réponse de 192.168.102.1 : octets=32 temps=2 ms TTL=127
Réponse de 192.168.102.1 : octets=32 temps=2 ms TTL=127

Statistiques Ping pour 192.168.102.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 2ms, Moyenne = 2ms
````

- PC 2

```powershell
PS C:\WINDOWS\system32> route add 192.168.101.1/24 mask 255.255.255.0 192.168.112.1
 OK!
PS C:\WINDOWS\system32> ping 192.168.101.1

Envoi d’une requête 'Ping'  192.168.101.1 avec 32 octets de données :
Réponse de 192.168.101.1 : octets=32 temps=3 ms TTL=127
Réponse de 192.168.101.1 : octets=32 temps=2 ms TTL=127
Réponse de 192.168.101.1 : octets=32 temps=1 ms TTL=127
Réponse de 192.168.101.1 : octets=32 temps=2 ms TTL=127

Statistiques Ping pour 192.168.101.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 3ms, Moyenne = 2ms
```
- VM 1

```bash
[root@localhost ~]$ sudo ip route add 192.168.112.0/30 via 192.168.101.1 dev enp0s8
[root@localhost ~]$ sudo ip route add 192.168.102.0/24 via 192.168.101.1 dev enp0s8
[root@localhost ~]# ping 192.168.112.2
PING 192.168.112.2 (192.168.112.2) 56(84) bytes of data.
64 bytes from 192.168.112.2: icmp_seq=1 ttl=127 time=2.63 ms
64 bytes from 192.168.112.2: icmp_seq=2 ttl=127 time=3.58 ms
64 bytes from 192.168.112.2: icmp_seq=3 ttl=127 time=3.09 ms
64 bytes from 192.168.112.2: icmp_seq=4 ttl=127 time=3.90 ms
64 bytes from 192.168.112.2: icmp_seq=5 ttl=127 time=3.01 ms
^C
--- 192.168.112.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4012ms
rtt min/avg/max/mdev = 2.630/3.245/3.900/0.449 ms
[root@localhost ~]# ping 192.168.102.1
PING 192.168.102.1 (192.168.102.1) 56(84) bytes of data.
64 bytes from 192.168.102.1: icmp_seq=1 ttl=126 time=2.82 ms
64 bytes from 192.168.102.1: icmp_seq=2 ttl=126 time=3.13 ms
64 bytes from 192.168.102.1: icmp_seq=3 ttl=126 time=2.93 ms
64 bytes from 192.168.102.1: icmp_seq=4 ttl=126 time=2.86 ms
^C
--- 192.168.102.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 2.829/2.939/3.130/0.122 ms
```

- VM 2

```bash
[louis@localhost ~]$ sudo ip route add 192.168.112.0/30 via 192.168.102.1 dev enp0s8
[louis@localhost ~]$ sudo ip route add 192.168.101.0/24 via 192.168.102.1 dev enp0s8
[louis@localhost ~]$ ping 192.168.112.1
PING 192.168.112.1 (192.168.112.1) 56(84) bytes of data.
64 bytes from 192.168.112.1: icmp_seq=1 ttl=127 time=5.16 ms
64 bytes from 192.168.112.1: icmp_seq=2 ttl=127 time=2.38 ms
64 bytes from 192.168.112.1: icmp_seq=3 ttl=127 time=2.83 ms
64 bytes from 192.168.112.1: icmp_seq=4 ttl=127 time=3.24 ms
^C
--- 192.168.112.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3010ms
rtt min/avg/max/mdev = 2.384/3.406/5.163/1.060 ms
[louis@localhost ~]$ ping 192.168.101.1
PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
64 bytes from 192.168.101.1: icmp_seq=1 ttl=126 time=2.27 ms
64 bytes from 192.168.101.1: icmp_seq=2 ttl=126 time=2.54 ms
64 bytes from 192.168.101.1: icmp_seq=3 ttl=126 time=2.43 ms
64 bytes from 192.168.101.1: icmp_seq=4 ttl=126 time=2.83 ms
^C
--- 192.168.101.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 2.278/2.522/2.831/0.201 ms
```
---
### Ping VM1 

- Ping VM 1 -> PC 1
```bash
[root@localhost etc]# ping pc1.tp3.b1
PING pc1 (192.168.112.1) 56(84) bytes of data.
64 bytes from pc1 (192.168.112.1): icmp_seq=1 ttl=127 time=0.237 ms
64 bytes from pc1 (192.168.112.1): icmp_seq=2 ttl=127 time=0.477 ms
64 bytes from pc1 (192.168.112.1): icmp_seq=3 ttl=127 time=0.461 ms
64 bytes from pc1 (192.168.112.1): icmp_seq=4 ttl=127 time=0.504 ms
^C
--- pc1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.237/0.419/0.504/0.109 ms
```

- Ping VM 1 -> PC 2

```bash
[root@localhost etc]# ping pc2.tp3.b1
PING pc2 (192.168.112.2) 56(84) bytes of data.
64 bytes from pc2 (192.168.112.2): icmp_seq=1 ttl=127 time=2.66 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=2 ttl=127 time=2.78 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=3 ttl=127 time=2.68 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=4 ttl=127 time=2.73 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=5 ttl=127 time=3.27 ms
^C
--- pc2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4013ms
rtt min/avg/max/mdev = 2.669/2.829/3.279/0.238 ms
```

- Ping VM 1 -> VM 2

```bash
[root@localhost etc]# ping vm2.tp3.b1
PING vm2 (192.168.102.10) 56(84) bytes of data.
64 bytes from vm2 (192.168.102.10): icmp_seq=1 ttl=62 time=2.82 ms
^[[A64 bytes from vm2 (192.168.102.10): icmp_seq=2 ttl=62 time=3.09 ms
64 bytes from vm2 (192.168.102.10): icmp_seq=3 ttl=62 time=3.83 ms
64 bytes from vm2 (192.168.102.10): icmp_seq=4 ttl=62 time=3.21 ms
64 bytes from vm2 (192.168.102.10): icmp_seq=5 ttl=62 time=3.54 ms
^C
--- vm2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 2.829/3.302/3.834/0.357 ms

```

- Ping VM 2 -> PC 1

```bash
[louis@localhost /]$ ping pc1.tp3.b1
PING pc1 (192.168.112.1) 56(84) bytes of data.
64 bytes from pc1 (192.168.112.1): icmp_seq=1 ttl=127 time=2.27 ms
64 bytes from pc1 (192.168.112.1): icmp_seq=2 ttl=127 time=2.62 ms
64 bytes from pc1 (192.168.112.1): icmp_seq=3 ttl=127 time=2.43 ms
64 bytes from pc1 (192.168.112.1): icmp_seq=4 ttl=127 time=2.29 ms
^C
--- pc1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 2.271/2.406/2.622/0.151 ms
```

- Ping VM 2 -> PC 2

```bash
[louis@localhost /]$ ping pc2.tp3.b1
PING pc2 (192.168.112.2) 56(84) bytes of data.
64 bytes from pc2 (192.168.112.2): icmp_seq=1 ttl=127 time=0.356 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=2 ttl=127 time=0.662 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=3 ttl=127 time=0.600 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=4 ttl=127 time=0.625 ms
^C
--- pc2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 0.356/0.560/0.662/0.123 ms
```

- Ping VM 2 -> VM 1

```bash
[louis@localhost /]$ ping vm1.tp3.b1
PING vm1 (192.168.101.10) 56(84) bytes of data.
64 bytes from vm1 (192.168.101.10): icmp_seq=1 ttl=62 time=2.50 ms
64 bytes from vm1 (192.168.101.10): icmp_seq=2 ttl=62 time=2.82 ms
64 bytes from vm1 (192.168.101.10): icmp_seq=3 ttl=62 time=2.26 ms
64 bytes from vm1 (192.168.101.10): icmp_seq=4 ttl=62 time=2.53 ms
^C
--- vm1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 2.261/2.530/2.820/0.207 ms
```
---
### Netcat VM 1 -> VM 2

```bash
[louis@localhost /]$ nc vm1.tp3.b1 5454
salut
je suis la vm1
et moi la vm2
ahahah
^C
```