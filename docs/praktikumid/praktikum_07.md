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
    - Oskan seadistada arvutile default gateway'i ja pingida läbi ruuteri teise võrku

!!! warning "Eeldused"
    Enne seda praktikumi pead tundma CLI põhitööd [Praktikumist 2](praktikum_02.md) ja olema läbi lugenud [Peatükk 6: Võrgukiht](../teooria/06_vorgukiht.md) ning [võrguseadmete peatüki](../teooria/00a_vorguseadmed.md).

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer (või füüsiline ruuter + konsoolikaabel)
    - Seadmed: 1× ruuter Cisco 2811 (R1), 2× kommutaator Cisco 2960 (SW1, SW2), 2× PC (PC1, PC2)

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
    Ruuteri liidesed on vaikimisi **välja lülitatud** turvakaalutlustel. Iga liides tuleb käsitsi aktiveerida käsuga `no shutdown`.

## Osa 2: Ruuteri uurimine

### Samm 1: Topoloogia

Ruuteri liides läheb olekusse `up` ainult siis, kui kaabli teises otsas on töötav seade. Seepärast ehita kohe terve topoloogia: kaks võrku, kummaski kommutaator ja arvuti, ruuter nende vahel.

| Seade | Kust leiad (Packet Tracer) | Nimi |
|---|---|---|
| Ruuter **2811** | Network Devices → Routers | R1 |
| 2 kommutaatorit **2960** | Network Devices → Switches | SW1, SW2 |
| 2 arvutit **PC** | End Devices | PC1, PC2 |

*Tabel 7.2. Praktikumi seadmed*

```text
PC1 ──── SW1 ──── R1 ──── SW2 ──── PC2
            Fa0/0    Fa0/1
```

Ühenda seadmed kaabliga **Copper Straight-Through**:

| Kust | Kuhu |
|---|---|
| PC1 FastEthernet0 | SW1 Fa0/1 |
| SW1 Fa0/24 | R1 **Fa0/0** |
| R1 **Fa0/1** | SW2 Fa0/24 |
| SW2 Fa0/1 | PC2 FastEthernet0 |

*Tabel 7.3. Kaabeldus*

!!! success "Oodatav tulemus"
    Kommutaatori ja arvutite vahel lähevad tuled kuni 30 sekundiga roheliseks. Ruuteri juures on tuled **punased**: ruuteri liidesed on vaikimisi välja lülitatud.

!!! info "Miks just 2811?"
    Packet Traceri 2811 liidesed on `FastEthernet0/0` ja `FastEthernet0/1`, nagu selles juhendis. Uuematel ruuteritel (4321, 1941) on liidesed `GigabitEthernet0/0/0` või `GigabitEthernet0/0` ja käskudes tuleb liidese nimi vastavalt muuta.

### Samm 2: Ühendamine ja riistvara info

Ühenda ruuteriga (PT: kliki ruuteril → CLI tab; füüsiline: konsoolikaabel + PuTTY).

```text
Router> enable
Router# show version
```

Märgi üles: ruuteri mudel, IOS versioon, RAM ja Flash maht.

### Samm 3: Liideste loetelu

```text
Router# show ip interface brief
```

```text
Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/0        unassigned      YES unset  administratively down down
FastEthernet0/1        unassigned      YES unset  administratively down down
```

Kõik liidesed on `unassigned` (IP puudub) ja `administratively down` (välja lülitatud). 2811 näitab Packet Traceris lisaks rida `Vlan1`, mida selles praktikumis ei kasuta.

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

Mõlemad liidesed peavad olema **up/up**. Esimene `up` tähendab, et kaabel on ühendatud ja teises otsas on töötav seade (siin kommutaator). Teine `up` tähendab, et liides saab andmeid vahetada.

### Marsruutimistabel

```text
R1# show ip route
```

```text
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet0/1
```

**C** = *Connected*: otseühendatud võrk. Ruuter teab automaatselt, millised võrgud on tema liideste taga. Sa ei lisanud ühtegi marsruuti: ruuter õppis need ise, kui liidesed said aadressi ja läksid olekusse `up`.

Packet Traceri 2811 kasutab IOS 12.4 ja näitab ainult `C`-ridu. Uuemad ruuterid (IOS 15, nt 4321 või 1941) näitavad iga `C`-rea all ka `L`-rida (*Local*): liidese enda IP-aadress /32 maskiga, nt `L 192.168.1.1/32`. Marsruutimistabeli lugemist käsitleb põhjalikumalt [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

### Ping test

```text
R1# ping 192.168.1.1
R1# ping 192.168.2.1
```

Mõlemad peavad vastama (`!!!!!` = 5 edukat pingi).

### Arvutite seadistamine ja ping teise võrku

Ruuter töötab. Nüüd anna arvutitele aadressid. Kliki PC-l → **Desktop** → **IP Configuration** → **Static**:

| | PC1 | PC2 |
|---|---|---|
| IPv4 Address | 192.168.1.10 | 192.168.2.10 |
| Subnet Mask | 255.255.255.0 | 255.255.255.0 |
| Default Gateway | 192.168.1.1 | 192.168.2.1 |

*Tabel 7.4. Arvutite IP-seaded*

Iga arvuti [default gateway](../teooria/06_vorgukiht.md) on ruuteri liides **tema enda võrgus**. Sama ruuter, kaks ust.

PC1 → Desktop → **Command Prompt**:

```text
C:\> ping 192.168.1.1
C:\> ping 192.168.2.1
C:\> ping 192.168.2.10
```

!!! success "Oodatav tulemus"
    Kõik kolm pingi õnnestuvad. Viimase väljund on umbes selline:

    ```text
    Pinging 192.168.2.10 with 32 bytes of data:

    Request timed out.
    Reply from 192.168.2.10: bytes=32 time<1ms TTL=127
    Reply from 192.168.2.10: bytes=32 time<1ms TTL=127
    Reply from 192.168.2.10: bytes=32 time<1ms TTL=127
    ```

    Esimene `Request timed out` on normaalne: ruuter otsib ARP-iga alles PC2 MAC-aadressi. `TTL=127` (arvuti alustab 128-st) näitab, et pakett läbis ühe ruuteri.

Vaata, kuidas pakett liikus:

```text
C:\> tracert 192.168.2.10
```

```text
Tracing route to 192.168.2.10 over 30 hops:

  1   0 ms      0 ms      0 ms      192.168.1.1
  2   0 ms      0 ms      0 ms      192.168.2.10

Trace complete.
```

Kaks hüpet: esimene on PC1 gateway (R1), teine on sihtkoht PC2.

### Lõhu meelega: gateway puudub

1. PC2 → IP Configuration → kustuta **Default Gateway**.
2. Ennusta: kas PC1-st `ping 192.168.2.10` töötab? PC1 seaded on ju õiged.
3. Pingi PC1-st PC2-e.

!!! success "Oodatav tulemus"
    Kõik neli pingi annavad `Request timed out`. Päring jõuab PC2-ni, aga vastus ei leia teed tagasi: PC2 ei tea, kuhu saata pakett võrku 192.168.1.0/24. Ping vajab gateway't **mõlemas** otsas.

Pane PC2 gateway tagasi (`192.168.2.1`) ja kontrolli, et ping töötab jälle.

## Osa 5: Tõrkeotsing

!!! warning "Liides jääb down?"
    - `administratively down` → unustasid `no shutdown`
    - `down/down` (ilma "administratively") → kaabel pole ühendatud või port on rikki

!!! warning "Ping ei tööta?"
    1. Ping ruuterile töötab, teise võrgu arvutile mitte → kontrolli **mõlema** arvuti Default Gateway'd (`ipconfig`)
    2. Kontrolli IP-aadresse: `show ip interface brief`
    3. Kontrolli marsruutimistabelit: `show ip route`
    4. Kas salvestasid? `show running-config` vs `show startup-config`

---

## Kokkuvõte

Selles praktikumis sa:

- Uurisid ruuteri riistvara infot ja liideste loetelu
- Konfigureerisid ruuteri hostnime, paroole ja bänneri
- Seadistasid kaks liidest IP-aadresside ja kirjeldustega
- Aktiveerisid liidesed `no shutdown` käsuga
- Kontrollisid liideste staatust ja marsruutimistabelit
- Testisid liideste tööd `ping` käsuga
- Seadistasid arvutitele default gateway'i ja pingisid läbi ruuteri teise võrku
- Nägid, et ilma gateway'ta ei jõua vastus tagasi
