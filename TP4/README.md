# TP 4 - Spéléologie réseau: descente dans les couches

# I. Mise en place du lab
## 1. Création des VMs
## 3. Mise en place du routage statiqueuh

`ip route show` de **`router1`**:
```bash
[louis@router1 ~]$ ip route show
10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s9 proto kernel scope link src 10.2.0.254 metric 101
```


`ping` de **`client1`** vers **`server1`**:
```bash
[louis@client1 ~]$ ping server1
PING server1 (10.2.0.10) 56(84) bytes of data.
64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=0.737 ms
64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=1.38 ms
64 bytes from server1 (10.2.0.10): icmp_seq=3 ttl=63 time=1.41 ms
^C
--- server1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.737/1.180/1.419/0.315 ms
```

`ping` de **`server1`** vers **`client1`**:
```bash
[louis@server ~]$ ping client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.762 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=1.51 ms
64 bytes from client1 (10.1.0.10): icmp_seq=3 ttl=63 time=1.42 ms
^C
--- client1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2011ms
rtt min/avg/max/mdev = 0.762/1.236/1.519/0.338 ms
```

`traceroute` depuis **`client1`**:
```bash
[louis@client1 ~]$ traceroute server1
traceroute to server1 (10.2.0.10), 30 hops max, 60 byte packets
 1  router1 (10.1.0.254)  0.316 ms  0.235 ms  0.170 ms
 2  router1 (10.1.0.254)  0.110 ms !X  0.190 ms !X  0.283 ms !X
```
# II. Spéléologie réseau

# 1. ARP

## A. Manip 1
2) sur `client1`
```bash
[louis@client1 /]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1b DELAY
```
La ligne correspond à l'adresse ip de la carte reseau sur laquelle est connecté le SSH

3) sur `server1`
```bash
[louis@server ~]$ sudo ip neigh show
10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0a DELAY
```
La ligne correspond à l'adresse ip de la carte reseau sur laquelle est connecté le SSH.

4) sur `client1`
- Ping vers le `server1`:
    ```bash
    [louis@client1 ~]$ ping server1
    PING server1 (10.2.0.10) 56(84) bytes of data.
    64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=1.22 ms
    64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=1.65 ms
    64 bytes from server1 (10.2.0.10): icmp_seq=3 ttl=63 time=1.46 ms
    ^C
    --- server1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2004ms
    rtt min/avg/max/mdev = 1.229/1.450/1.654/0.176 ms
    ```
- Afficher la table ARP :
    ```console
    [louis@client1 ~]$ ip neigh show
    10.1.0.254 dev enp0s8 lladdr 08:00:27:15:52:20 STALE
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1b DELAY
    ```
- > *Comment ça se fait qu'il y ai une ligne qui se soit rajouter ?*

    La ligne qui s'est rajouté ? C'est juste les adresses **ip** et **mac** du `router1` sur le réseau du `client1`.

    > *Mais comment le client a fait pour connaitre l'adresse du router ?*

    Quand on a ping le `server1` depuis le `client1`, ce ping passe par le `router1` et c'est à ce moment là que les deux machines s'échangent leurs adresses **ip** et **mac**.

5) sur `server1`
- > *Est-ce que la table ARP du server a changé elle aussi ?*
    
    Il suffit juste de vérifier :
    ```console
    [louis@server1 ~]$ ip neigh show
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0a REACHABLE
    10.2.0.254 dev enp0s8 lladdr 08:00:27:6e:66:a2 STALE
    ```
- > *Ah oui elle a changé ! Mais qu'est-ce qui s'est rajouté ?*

    Comme pour le client, le `server1` et le `routeur1` se sont échangés leurs adresses mais cette fois-ci c'était pour recevoir le ping et non pas l'envoyer.

## B. Manip 2

2) sur `router1`
- Afficher la table ARP
    ```console
    [louis@router1 etc]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1b DELAY
    ```
- > *A quoi correspond cette ligne ?*

    Cette ligne correpond à l'adresse **ip** du gateway du `router1`.

3) sur `client1`
    ```console
    [louis@client1 ~]$ ping server1
    PING server1 (10.2.0.10) 56(84) bytes of data.
    64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=1.42 ms
    64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=0.750 ms
    64 bytes from server1 (10.2.0.10): icmp_seq=3 ttl=63 time=1.11 ms
    ^C
    --- server1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 0.750/1.094/1.422/0.277 ms
    ```
4) sur `router1`
- Afficher la table ARP :

    ```console
    [louis@router1 etc]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1b DELAY
    10.1.0.10 dev enp0s8 lladdr 08:00:27:5b:cb:9f STALE
    10.2.0.10 dev enp0s9 lladdr 08:00:27:b0:4e:d5 STALE
    ```
- > *Qu'est-ce qu'il s'est passé encore ?*
     
    Ce n'est pas compliqué ! En effectuant le ping, ce dernier est passé par le `router1` qui a donc enregistré les adresses **ip** et **mac** du `client1` et du `router1`.

## C. Manip 3

2) sur le PC
    - Afficher la table ARP
        ```powershell
        PS C:\WINDOWS\system32> arp -a

        Interface : 192.168.102.1 --- 0x8
        Adresse Internet      Adresse physique      Type
        192.168.102.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 192.168.188.1 --- 0x9
        Adresse Internet      Adresse physique      Type
        192.168.188.254       00-50-56-e7-d7-78     dynamique
        192.168.188.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique
        255.255.255.255       ff-ff-ff-ff-ff-ff     statique

        Interface : 10.2.0.1 --- 0xa
        Adresse Internet      Adresse physique      Type
        10.2.0.10             08-00-27-b0-4e-d5     dynamique
        10.2.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 192.168.79.1 --- 0xb
        Adresse Internet      Adresse physique      Type
        192.168.79.254        00-50-56-e5-58-9c     dynamique
        192.168.79.255        ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique
        255.255.255.255       ff-ff-ff-ff-ff-ff     statique

        Interface : 192.168.1.21 --- 0x15
        Adresse Internet      Adresse physique      Type
        192.168.1.1           30-7c-b2-87-9e-c6     dynamique
        192.168.1.10          58-48-22-71-fb-75     dynamique
        192.168.1.18          24-7f-20-d9-8e-1c     dynamique
        192.168.1.26          e8-40-f2-b9-23-4f     dynamique
        192.168.1.255         ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique
        255.255.255.255       ff-ff-ff-ff-ff-ff     statique

        Interface : 192.168.56.1 --- 0x18
        Adresse Internet      Adresse physique      Type
        192.168.56.255        ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        230.0.0.1             01-00-5e-00-00-01     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 10.1.0.1 --- 0x1b
        Adresse Internet      Adresse physique      Type
        10.1.0.10             08-00-27-5b-cb-9f     dynamique
        10.1.0.254            08-00-27-15-52-20     dynamique
        10.1.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique
        ```
        ---
    - Après l'avoir effacé voici la nouvelle table ARP :
        ```powershell
        PS C:\WINDOWS\system32> arp -a

        Interface : 192.168.102.1 --- 0x8
        Adresse Internet      Adresse physique      Type
        224.0.0.22            01-00-5e-00-00-16     statique

        Interface : 192.168.188.1 --- 0x9
        Adresse Internet      Adresse physique      Type
        224.0.0.22            01-00-5e-00-00-16     statique
        239.192.152.143       01-00-5e-40-98-8f     statique

        Interface : 10.2.0.1 --- 0xa
        Adresse Internet      Adresse physique      Type
        224.0.0.22            01-00-5e-00-00-16     statique

        Interface : 192.168.79.1 --- 0xb
        Adresse Internet      Adresse physique      Type
        224.0.0.22            01-00-5e-00-00-16     statique
        239.192.152.143       01-00-5e-40-98-8f     statique

        Interface : 192.168.1.21 --- 0x15
        Adresse Internet      Adresse physique      Type
        192.168.1.1           30-7c-b2-87-9e-c6     dynamique
        224.0.0.2             01-00-5e-00-00-02     statique

        Interface : 192.168.56.1 --- 0x18
        Adresse Internet      Adresse physique      Type
        224.0.0.22            01-00-5e-00-00-16     statique
        230.0.0.1             01-00-5e-00-00-01     statique

        Interface : 10.1.0.1 --- 0x1b
        Adresse Internet      Adresse physique      Type
        224.0.0.22            01-00-5e-00-00-16     statique
        ```
        ---
    - Après avoir un peu attendu, voici la table ARP remise à jour (toute seule comme une grande) :
        ```powershell
        PS C:\WINDOWS\system32> arp -a

        Interface : 192.168.102.1 --- 0x8
        Adresse Internet      Adresse physique      Type
        192.168.102.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 192.168.188.1 --- 0x9
        Adresse Internet      Adresse physique      Type
        192.168.188.254       00-50-56-e7-d7-78     dynamique
        192.168.188.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 10.2.0.1 --- 0xa
        Adresse Internet      Adresse physique      Type
        10.2.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 192.168.79.1 --- 0xb
        Adresse Internet      Adresse physique      Type
        192.168.79.254        00-50-56-e5-58-9c     dynamique
        192.168.79.255        ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 192.168.1.21 --- 0x15
        Adresse Internet      Adresse physique      Type
        192.168.1.1           30-7c-b2-87-9e-c6     dynamique
        192.168.1.18          24-7f-20-d9-8e-1c     dynamique
        192.168.1.255         ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 192.168.56.1 --- 0x18
        Adresse Internet      Adresse physique      Type
        192.168.56.255        ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        230.0.0.1             01-00-5e-00-00-01     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique

        Interface : 10.1.0.1 --- 0x1b
        Adresse Internet      Adresse physique      Type
        10.1.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.2             01-00-5e-00-00-02     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        232.0.32.15           01-00-5e-00-20-0f     statique
        239.192.152.143       01-00-5e-40-98-8f     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        239.255.255.253       01-00-5e-7f-ff-fd     statique
        ```
        ---
    - > *Mais comment ça se fait ?*

    En fait, notre `router1` peut se connecter à internet par l'intermediaire de notre PC en utilisant une passerelle et les machines `client1` et `server1` se servent du `router1` comme passerelle. Cela explique le fait qu'il y ait tant d'adresses dans la table ARP du PC

    > *Oui mais quand on efface la table ARP, elles reviennent quand-même !*

    Elles reviennent parce que les machines sont connectées au PC et aussi allumées.
