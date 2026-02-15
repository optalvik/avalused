---
tags:
  - Praktikum
  - CCNA
---

# Praktikum 3: Kommutaatori ja ruuteri seadistamine

Selles praktikumis lood väikese kohtvõrgu (LAN), kus kaks arvutit on samas alamvõrgus ja suhtlevad omavahel **kommutaatori kaudu**. Ruuter lisatakse võrku selleks, et harjutada ruuteri liidese seadistamist ning anda kommutaatorile halduseks (management) vaikimisi lüüs (default gateway). Konfigureerid mõlema seadme põhiseadistused ja testid ühenduvust.

Kommutaator töötab [andmesidekihis](../teooria/04_andmesidekiht.md) (Layer 2) ja edastab kaadreid ühe võrgu sees. Ruuter töötab [võrgukihis](../teooria/06_vorgukiht.md) (Layer 3) ja ühendab **erinevaid** võrke omavahel — sellest lähemalt [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan üles ehitada võrgu ruuteri, kommutaatori ja arvutitega
    - Oskan konfigureerida ruuteri liidese IP-aadressi
    - Oskan konfigureerida kommutaatori VLAN 1 haldusaadressi ja default gateway'd
    - Oskan testida ühenduvust (PC↔PC, PC↔ruuter, PC↔kommutaator (VLAN 1))

!!! warning "Eeldused"
    Enne seda praktikumi pead olema läbinud [Praktikum 2: CLI ja kommutaatori seadistamine](praktikum_02.md) ning tutvunud [võrguseadmetega](../teooria/00a_vorguseadmed.md).

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer
    - Seadmed: 1× Cisco 4221 ruuter (R1), 1× Cisco 2960 kommutaator (S1), 2× PC

## Osa 1: Topoloogia ja adresseerimistabel

### Topoloogia

```text
[PC-A] -------- [S1 Kommutaator] -------- [PC-B]
                       |
                       |
                  [R1 Ruuter]
```

### Adresseerimistabel

| Seade | Liides | IP-aadress | Alamvõrgumask | Default gateway |
|---|---|---|---|---|
| R1 | G0/0/0 | 192.168.0.1 | 255.255.255.0 | — |
| S1 | VLAN 1 | 192.168.0.2 | 255.255.255.0 | 192.168.0.1 |
| PC-A | NIC | 192.168.0.3 | 255.255.255.0 | 192.168.0.1 |
| PC-B | NIC | 192.168.0.4 | 255.255.255.0 | 192.168.0.1 |

*Tabel 3.1. Võrgu adresseerimistabel*

### Samm 1: Seadmete ühendamine

1. PC-A → S1 port Fa0/1
2. PC-B → S1 port Fa0/2
3. R1 (G0/0/0) → S1 port Fa0/5

Kõik ühendused Copper Straight-Through kaabliga.

### Samm 2: PC-de IP-aadresside konfigureerimine

Seadista mõlemad PC-d vastavalt adresseerimistabelile (**Desktop** → **IP Configuration** → **Static**).

## Osa 2: Ruuteri seadistamine

Ruuteri liidese IP-aadress määrab, millise võrgu lüüsiks (*gateway*) ta saab. Selles topoloogias on ruuteri G0/0/0 liides kogu LAN-i värav välismaailma. IP-adresseerimist käsitleb [Peatükk 8: IPv4 adresseerimine](../teooria/08_ipv4.md).

```text
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface g0/0/0
R1(config-if)# ip address 192.168.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# exit
R1# copy running-config startup-config
```

!!! tip "no shutdown"
    Cisco ruuteri liidesed on vaikimisi **välja lülitatud** (*administratively down*). Ilma `no shutdown` käsuta liides ei tööta, isegi kui IP-aadress on seadistatud.

## Osa 3: Kommutaatori seadistamine

Kommutaator edastab kaadreid automaatselt ilma IP-aadressita, kuid halduse jaoks (Telnet/SSH) vajab ta IP-aadressi VLAN 1 liidesel. `ip default-gateway` määrab, kuhu kommutaator saadab **oma** halduspakette, kui sihtkoht on väljaspool kohalikku võrku.

```text
Switch> enable
Switch# configure terminal
Switch(config)# hostname S1
S1(config)# interface vlan 1
S1(config-if)# ip address 192.168.0.2 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.0.1
S1(config)# exit
S1# copy running-config startup-config
```

!!! info "ip default-gateway"
    Kommutaator vajab `ip default-gateway` käsku selleks, et **haldusliiklus** (nt SSH/Telnet) saaks väljuda kohalikust võrgust. See ei mõjuta läbi kommutaatori liikuvat kasutajaliiklust. Täpsemalt käsitleb lüüside teemat [Peatükk 6: Võrgukiht](../teooria/06_vorgukiht.md).

## Osa 4: Ühenduvuse testimine

`ping` kasutab ICMP (*Internet Control Message Protocol*) pakette, et kontrollida, kas sihtseade on kättesaadav. See on kõige lihtsam tööriist võrguühenduvuse testimiseks — törkeotsingu põhimõtteid käsitleb [Peatükk 14: Tõrkeotsing](../teooria/14_torkeotsing.md).

Ava PC-A → **Command Prompt**:

```text
ping 192.168.0.1
ping 192.168.0.4
```

Ava PC-B → **Command Prompt**:

```text
ping 192.168.0.1
ping 192.168.0.3
```

Kõik neli ping'i peaksid andma vastuse.

### Kontrollikäsud

```text
R1# show ip interface brief
S1# show ip interface brief
S1# show vlan brief
```

## Osa 5: Tõrkeotsing

!!! warning "Ping ruuterile ei tööta?"
    1. Kas ruuteri liides on **up/up**? (`show ip interface brief`)
    2. Kas tegid `no shutdown`?
    3. Kas PC-de default gateway viitab ruuteri IP-le (`192.168.0.1`)?

!!! warning "PC-d ei näe teineteist?"
    1. Kas mõlemad PC-d on samas alamvõrgus?
    2. Kas kommutaatori pordid on **up**?
    3. Kas kaablid on **rohelised**?

---

## Kokkuvõte

Selles praktikumis sa:

- Ehitasid võrgu ruuteri, kommutaatori ja kahe arvutiga
- Konfigureerisid ruuteri G0/0/0 liidese IP-aadressi
- Konfigureerisid kommutaatori VLAN 1 haldusaadressi ja default gateway
- Testisid ühenduvust läbi kogu võrgu kõigi seadmete vahel
