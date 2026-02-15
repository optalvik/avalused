---
tags:
  - Praktikum
  - Ruutimine
  - CCNA
---

# Praktikum 8: Staatiline marsruutimine

Selles praktikumis konfigureerid staatilise marsruutimise kahe ruuteri vahel, et ühendada kaks eraldiseisvat kohtvõrku. Õpid, kuidas käsitsi määrata marsruudid ja testid pakettide liikumist võrkude vahel.

Staatiline marsruutimine tähendab, et võrguadministraator lisab marsruudid käsitsi — ruuter ise naaberruuteritelt teavet ei küsi. See sobib väikestele võrkudele, aga suurematel muutub haldamine keeruliseks. Marsruutimise teooriat käsitleb [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md), IP-adresseerimist [Peatükk 8: IPv4](../teooria/08_ipv4.md) ja alamvõrgustamist [Peatükk 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan selgitada staatilise marsruutimise tööpõhimõtet
    - Oskan konfigureerida staatilised marsruudid mõlemal ruuteril
    - Oskan testida ühenduvust kahe kohtvõrgu vahel
    - Oskan lugeda marsruutimistabelit ja eristada C- ja S-marsruute

!!! warning "Eeldused"
    Enne seda praktikumi pead olema läbinud [Praktikum 7: Ruuteri avastamine](praktikum_07.md) ja tutvunud [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer
    - Seadmed: 2× ruuter, 2× kommutaator, 4× PC

## Osa 1: Topoloogia ja adresseerimine

### Topoloogia

```text
[PC0]--[PC1]--[SW0]--[R0]---G0/0---[R1]--[SW1]--[PC2]--[PC3]
                     G0/1   G0/0  G0/0  G0/1
       LAN 1                Link              LAN 2
  192.168.70.0/24      221.123.1.0/30    172.230.10.0/24
```

!!! note "Oluline: mis liidest sa kasutad?"
    Selle juhendi konfiguratsioon kasutab ruuterite vahel **Ethernet-liidest G0/0 ↔ G0/0** (nagu adresseerimistabelis).
    Kui ehitad Packet Traceris ruuterite vahele **Serial** lingi, pead kasutama `s0/0/0` tüüpi liideseid ja DCE poolel määrama ka `clock rate`.

### Adresseerimistabel

| Seade | Liides | IP-aadress | Alamvõrgumask |
|---|---|---|---|
| R0 | G0/0 | 221.123.1.1 | 255.255.255.252 |
| R0 | G0/1 | 192.168.70.1 | 255.255.255.0 |
| R1 | G0/0 | 221.123.1.2 | 255.255.255.252 |
| R1 | G0/1 | 172.230.10.1 | 255.255.255.0 |
| PC0 | NIC | 192.168.70.2 | 255.255.255.0 |
| PC1 | NIC | 192.168.70.3 | 255.255.255.0 |
| PC2 | NIC | 172.230.10.2 | 255.255.255.0 |
| PC3 | NIC | 172.230.10.3 | 255.255.255.0 |

*Tabel 8.1. Võrgu adresseerimistabel*

!!! info "Miks /30 mask lingi jaoks?"
    Ruuteritevahelisel lingil on vaja ainult 2 IP-aadressi. /30 mask annab täpselt 2 kasutatavat aadressi — IP-aadresside kokkuhoid. Alamvõrgumaskidest lähemalt [Peatükk 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md) ja [Peatükk 11: VLSM](../teooria/11_vlsm.md).

## Osa 2: Seadmete konfigureerimine

### Samm 1: R0 konfiguratsioon

```text
Router> enable
Router# configure terminal
Router(config)# hostname R0
R0(config)# interface g0/0
R0(config-if)# ip address 221.123.1.1 255.255.255.252
R0(config-if)# no shutdown
R0(config-if)# exit
R0(config)# interface g0/1
R0(config-if)# ip address 192.168.70.1 255.255.255.0
R0(config-if)# no shutdown
R0(config-if)# exit
```

### Samm 2: R1 konfiguratsioon

```text
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface g0/0
R1(config-if)# ip address 221.123.1.2 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface g0/1
R1(config-if)# ip address 172.230.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Samm 3: PC-de ja default gateway konfigureerimine

PC0 ja PC1: default gateway **192.168.70.1** (R0 G0/1).

PC2 ja PC3: default gateway **172.230.10.1** (R1 G0/1).

## Osa 3: Staatiliste marsruutide konfigureerimine

Ilma staatiliste marsruutideta teab iga ruuter ainult oma otseühendatud võrke. R0 ei tea, kuidas jõuda LAN 2-sse ja R1 ei tea, kuidas jõuda LAN 1-sse.

### R0: marsruut LAN 2 suunas

```text
R0(config)# ip route 172.230.10.0 255.255.255.0 221.123.1.2
```

See ütleb R0-le: *"Võrk 172.230.10.0/24 on kättesaadav läbi 221.123.1.2 (R1)."*

### R1: marsruut LAN 1 suunas

```text
R1(config)# ip route 192.168.70.0 255.255.255.0 221.123.1.1
```

!!! danger "Marsruudid mõlemas suunas!"
    Staatilised marsruudid peavad olema konfigureeritud **mõlemal** ruuteril. Kui ainult R0 teab LAN 2-st, saab ta paketi küll kohale saata, aga vastus ei tule tagasi.

### Salvestamine

```text
R0# copy running-config startup-config
R1# copy running-config startup-config
```

## Osa 4: Kontrollimine ja testimine

### Marsruutimistabeli kontrollimine

```text
R0# show ip route
```

```text
C    192.168.70.0/24 is directly connected, GigabitEthernet0/1
C    221.123.1.0/30 is directly connected, GigabitEthernet0/0
S    172.230.10.0/24 [1/0] via 221.123.1.2
```

**C** = *Connected* (otseühendatud), **S** = *Static* (käsitsi lisatud).

### Ping testid

**Ruuterite vahel:**

```text
R0# ping 221.123.1.2
```

**LAN-ide vahel (PC0 → PC2):**

```text
ping 172.230.10.2
```

Kui ping töötab — paketid liiguvad edukalt LAN 1-st LAN 2-sse ja tagasi.

### Paketi teekond

```text
PC0 (192.168.70.2) → ping → PC2 (172.230.10.2)

1. PC0: "Kas 172.230.10.2 on minu võrgus?" → EI
2. PC0: saadab paketi default gateway'le (192.168.70.1 = R0)
3. R0: vaatab marsruutimistabelit → S 172.230.10.0 via 221.123.1.2
4. R0: saadab paketi R1-le
5. R1: "Kas 172.230.10.2 on minu võrgus?" → JAH (G0/1)
6. R1: saadab paketi otse PC2-le
```

Vastus liigub sama loogikaga tagasi.

## Osa 5: Tõrkeotsing

!!! warning "Ping LAN-ide vahel ei tööta?"
    1. Kas ruuteritevahelise lingi ping töötab? (`R0# ping 221.123.1.2`)
    2. Kas staatilised marsruudid on **mõlemal** ruuteril?
    3. Kas PC-de default gateway on õige?
    4. Kas kõik liidesed on **up/up**?

---

## Kokkuvõte

Selles praktikumis sa:

- Ehitasid kahe ruuteri ja kahe kohtvõrguga topoloogia
- Konfigureerisid staatilised marsruudid mõlemal ruuteril
- Testisid ühenduvust kahe eraldiseisva kohtvõrgu vahel
- Lugesid marsruutimistabelit ja eristasid C- ja S-marsruute
- Jälgisid paketi teekonda läbi mitme ruuteri
