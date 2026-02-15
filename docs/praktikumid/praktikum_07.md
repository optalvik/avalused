---
tags:
  - Praktikum
  - CCNA
---

# Praktikum 7: Ruuteri avastamine ja seadistamine

Selles praktikumis tutvud ruuteri füüsilise ülesehituse ja CLI-ga. Konfigureerid ruuteri liideste IP-aadressid, kontrollid liideste staatust ja uurid marsruutimistabelit.

Ruuter on [võrgukihi](../teooria/06_vorgukiht.md) (Layer 3) seade, mis suunab IP-pakette erinevate võrkude vahel marsruutimistabeli alusel. Erinevalt kommutaatorist, mis töötab [andmesidekihis](../teooria/04_andmesidekiht.md) MAC-aadressidega, kasutab ruuter otsuste tegemiseks **IP-aadresse**. Marsruutimise põhimõtteid käsitleb [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan selgitada ruuteri ja kommutaatori erinevust (Layer 2 vs Layer 3)
    - Oskan uurida ruuteri riistvara infot (`show version`)
    - Oskan konfigureerida ruuteri liideste IP-aadressid ja kirjeldused
    - Oskan lugeda `show ip interface brief` ja `show ip route` väljundit
    - Oskan testida ruuteri liideste tööd `ping` käsuga

!!! warning "Eeldused"
    Enne seda praktikumi pead tundma CLI põhitööd [Praktikumist 2](praktikum_02.md) ja olema läbi lugenud [Peatükk 6: Võrgukiht](../teooria/06_vorgukiht.md) ning [võrguseadmete peatüki](../teooria/00a_vorguseadmed.md).

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer (või füüsiline ruuter + konsoolikaabel)
    - Seadmed: 1× ruuter (Cisco 2811 või 4321)

## Osa 1: Ruuter vs kommutaator

| Omadus | Kommutaator | Ruuter |
|---|---|---|
| OSI kiht | Layer 2 (andmeside) | Layer 3 (võrgu) |
| Kasutab | MAC-aadresse | IP-aadresse |
| Edastab | Kaadreid (*frames*) | Pakette (*packets*) |
| Ühendab | Seadmeid **samas** võrgus | **Erinevaid** võrke |
| Liidesed vaikimisi | Sees (*up*) | Väljas (*administratively down*) |

*Tabel 7.1. Kommutaatori ja ruuteri võrdlus*

!!! info "Oluline erinevus"
    Ruuteri liidesed on vaikimisi **välja lülitatud** — turvakaalutlustel. Iga liides tuleb käsitsi aktiveerida käsuga `no shutdown`.

## Osa 2: Ruuteri uurimine

### Samm 1: Ühendamine ja riistvara info

Ühenda ruuteriga (PT: kliki ruuteril → CLI tab; füüsiline: konsoolikaabel + PuTTY).

```text
Router> enable
Router# show version
```

Märgi üles: ruuteri mudel, IOS versioon, RAM ja Flash maht.

### Samm 2: Liideste loetelu

```text
Router# show ip interface brief
```

```text
Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/0        unassigned      YES unset  administratively down down
FastEthernet0/1        unassigned      YES unset  administratively down down
```

Kõik liidesed on `unassigned` (IP puudub) ja `administratively down` (välja lülitatud).

## Osa 3: Ruuteri konfigureerimine

### Samm 1: Põhiseadistus

```text
Router# configure terminal
Router(config)# hostname R1
R1(config)# enable secret cisco123
R1(config)# banner motd #Volitamata ligipääs keelatud!#
```

### Samm 2: Konsool- ja VTY paroolid

```text
R1(config)# line console 0
R1(config-line)# password console123
R1(config-line)# login
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password vty123
R1(config-line)# login
R1(config-line)# exit
```

### Samm 3: Liideste konfigureerimine

**FastEthernet0/0** (gateway):

```text
R1(config)# interface fastethernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description Gateway kohtvorgule
R1(config-if)# no shutdown
R1(config-if)# exit
```

**FastEthernet0/1** (teine võrk):

```text
R1(config)# interface fastethernet 0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# description Teine alamvork
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Samm 4: Salvestamine

```text
R1(config)# exit
R1# copy running-config startup-config
```

## Osa 4: Kontrollimine

### Liideste staatus

```text
R1# show ip interface brief
```

```text
Interface              IP-Address      OK? Method Status    Protocol
FastEthernet0/0        192.168.1.1     YES manual up        up
FastEthernet0/1        192.168.2.1     YES manual up        up
```

Mõlemad liidesed peavad olema **up/up**.

### Marsruutimistabel

```text
R1# show ip route
```

```text
C    192.168.1.0/24 is directly connected, FastEthernet0/0
L    192.168.1.1/32 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet0/1
L    192.168.2.1/32 is directly connected, FastEthernet0/1
```

**C** = *Connected* — otseühendatud võrk. Ruuter teab automaatselt, millised võrgud on tema liideste taga. **L** = *Local* — liidese enda IP-aadress (/32 mask). Marsruutimistabeli lugemist käsitleb põhjalikumalt [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

### Ping test

```text
R1# ping 192.168.1.1
R1# ping 192.168.2.1
```

Mõlemad peavad vastama (`!!!!!` = 5 edukat pingi).

## Osa 5: Tõrkeotsing

!!! warning "Liides jääb down?"
    - `administratively down` → unustasid `no shutdown`
    - `down/down` (ilma "administratively") → kaabel pole ühendatud või port on rikki

!!! warning "Ping ei tööta?"
    1. Kontrolli IP-aadresse: `show ip interface brief`
    2. Kontrolli marsruutimistabelit: `show ip route`
    3. Kas salvestasid? `show running-config` vs `show startup-config`

---

## Kokkuvõte

Selles praktikumis sa:

- Uurisid ruuteri riistvara infot ja liideste loetelu
- Konfigureerisid ruuteri hostnime, paroole ja bänneri
- Seadistasid kaks liidest IP-aadresside ja kirjeldustega
- Aktiveerisid liidesed `no shutdown` käsuga
- Kontrollisid liideste staatust ja marsruutimistabelit
- Testisid liideste tööd `ping` käsuga
