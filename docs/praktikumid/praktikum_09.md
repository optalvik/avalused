---
tags:
  - Praktikum
  - Ruutimine
---

# Praktikum 9: Staatiline marsruutimine (iseseisev harjutus)

See praktikum kinnistab [Praktikumis 8](praktikum_08.md) õpitut — ehitad sama tüüpi topoloogia nullist ja ilma detailsete juhisteta. Eesmärk on kontrollida, kas suudad staatilise marsruutimise seadistada iseseisvalt.

Teooria kordamiseks: [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md), [Peatükk 8: IPv4](../teooria/08_ipv4.md), [Peatükk 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan iseseisvalt planeerida IP-aadressid mitmele võrgule
    - Oskan ilma juhisteta konfigureerida staatilisi marsruute käsuga `ip route`
    - Oskan kasutada /30 maski punkt-punkt ühendustes
    - Oskan testida ja kontrollida marsruutimist `show ip route` ja `ping` abil
    - Oskan selgitada staatilise marsruutimise eeliseid ja puudusi

## Topoloogia

```text
[PC0]--[PC1]--[SW0]--[R0]---link---[R1]--[SW1]--[PC2]--[PC3]
                     G0/1  G0/0  G0/0  G0/1
              LAN 1              Link              LAN 2
         192.168.70.0/24   221.123.1.0/30   172.230.10.0/24
```

| Seade | Liides | Võrk | IP-aadress | Mask |
|---|---|---|---|---|
| R0 | G0/0 | Link | 221.123.1.1 | 255.255.255.252 |
| R0 | G0/1 | LAN 1 | 192.168.70.1 | 255.255.255.0 |
| R1 | G0/0 | Link | 221.123.1.2 | 255.255.255.252 |
| R1 | G0/1 | LAN 2 | 172.230.10.1 | 255.255.255.0 |
| PC0 | NIC | LAN 1 | 192.168.70.2 | 255.255.255.0 |
| PC1 | NIC | LAN 1 | 192.168.70.3 | 255.255.255.0 |
| PC2 | NIC | LAN 2 | 172.230.10.2 | 255.255.255.0 |
| PC3 | NIC | LAN 2 | 172.230.10.3 | 255.255.255.0 |

*Tabel 9.1. IP-aadresside plaan*

!!! info "/30 mask link-võrgus"
    Ruuterite vaheline ühendus vajab ainult 2 IP-aadressi. /30 mask annab täpselt 2 kasutatavat aadressi — aadressiruumi optimaalne kasutus. Vt [Peatükk 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md).

## Osa 1: Topoloogia loomine

### Samm 1: Lisa seadmed

- 2× Router 2911 (R0, R1)
- 2× Switch 2960 (SW0, SW1)
- 4× PC (PC0–PC3)

### Samm 2: Kaabeldamine

| Ühendus | Port A | Port B |
|---|---|---|
| R0 ↔ R1 | G0/0 | G0/0 |
| R0 ↔ SW0 | G0/1 | G0/1 |
| R1 ↔ SW1 | G0/1 | G0/1 |
| PC0 ↔ SW0 | Fa0 | Fa0/1 |
| PC1 ↔ SW0 | Fa0 | Fa0/2 |
| PC2 ↔ SW1 | Fa0 | Fa0/1 |
| PC3 ↔ SW1 | Fa0 | Fa0/2 |

*Tabel 9.2. Kaabeldamise plaan*

### Samm 3: Seadista PC-de IP-aadressid

PC0–PC1: gateway **192.168.70.1**

PC2–PC3: gateway **172.230.10.1**

## Osa 2: Ruuterite konfigureerimine

### R0

```text
Router> enable
Router# configure terminal
Router(config)# hostname R0
R0(config)# interface GigabitEthernet0/0
R0(config-if)# ip address 221.123.1.1 255.255.255.252
R0(config-if)# no shutdown
R0(config-if)# exit
R0(config)# interface GigabitEthernet0/1
R0(config-if)# ip address 192.168.70.1 255.255.255.0
R0(config-if)# no shutdown
R0(config-if)# exit
```

### R1

```text
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 221.123.1.2 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 172.230.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

## Osa 3: Lokaalne test

Enne marsruutimise lisamist testi LAN-ide sisest ühenduvust:

PC0 → PC1:

```bash
ping 192.168.70.3
```

PC2 → PC3:

```bash
ping 172.230.10.3
```

Mõlemad peavad töötama — see kinnitab, et Layer 2 (kommutaator) ja IP-seadistused on korras.

## Osa 4: Staatiliste marsruutide lisamine

Praegu teab R0 ainult oma otseühendatud võrke (192.168.70.0 ja 221.123.1.0). Ta **ei tea**, kuidas jõuda 172.230.10.0 võrku. Sama R1 puhul — ta ei tea 192.168.70.0 kohta.

### R0 — marsruut LAN 2 suunas

```text
R0(config)# ip route 172.230.10.0 255.255.255.0 221.123.1.2
```

See ütleb: "Kui tahad jõuda 172.230.10.0/24 võrku, saada pakett next-hop aadressile 221.123.1.2 (R1)."

### R1 — marsruut LAN 1 suunas

```text
R1(config)# ip route 192.168.70.0 255.255.255.0 221.123.1.1
```

### Salvesta mõlemal

```text
R0# copy running-config startup-config
R1# copy running-config startup-config
```

## Osa 5: Marsruutimistabeli kontrollimine

### R0

```text
R0# show ip route
```

```text
C    192.168.70.0/24 is directly connected, GigabitEthernet0/1
C    221.123.1.0/30 is directly connected, GigabitEthernet0/0
S    172.230.10.0/24 [1/0] via 221.123.1.2
```

**C** = Connected (otse ühendatud), **S** = Static (käsitsi lisatud).

### R1

```text
R1# show ip route
```

```text
C    172.230.10.0/24 is directly connected, GigabitEthernet0/1
C    221.123.1.0/30 is directly connected, GigabitEthernet0/0
S    192.168.70.0/24 [1/0] via 221.123.1.1
```

## Osa 6: Lõpptest

PC0 → PC2 (LAN 1 → LAN 2):

```bash
ping 172.230.10.2
```

PC2 → PC0 (LAN 2 → LAN 1):

```bash
ping 192.168.70.2
```

Mõlemad peavad töötama — paketid liiguvad R0 → link → R1 ja tagasi.

!!! danger "Ping ei tööta?"
    1. Kas mõlemal ruuteril on staatiline marsruut? (mõlemal suunal!)
    2. Kas PC-de default gateway on ruuteri LAN-liidese IP?
    3. Kas kõik liidested on up/up? (`show ip interface brief`)

---

## Kokkuvõte

Selles praktikumis sa:

- Planeerisid IP-aadressid kolmele võrgule (2 LAN + 1 link)
- Kasutasid /30 maski punkt-punkt ühenduses
- Konfigureerrisid staatilised marsruudid käsuga `ip route`
- Kontrollisid marsruutimistabelit (`show ip route`)
- Testisid LAN-idevahelist ühenduvust läbi kahe ruuteri
