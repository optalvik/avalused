---
tags:
  - Praktikum
  - Ruutimine
  - CCNA
---

# Praktikum 8: Staatiline marsruutimine

Praktikumis 7 ühendas üks ruuter kaks võrku ja teadis mõlemat ise, sest need olid tema liideste taga. Selles praktikumis on võrkude vahel kaks ruuterit. Kumbki ruuter teab ainult oma liideste taga olevaid võrke. Teise ruuteri taga oleva võrgu lisad marsruutimistabelisse käsitsi käsuga `ip route`. Marsruutimistabelit ja staatilisi marsruute käsitleb [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan selgitada, miks ruuter ei tea teise ruuteri taga olevat võrku
    - Oskan lisada staatilise marsruudi käsuga `ip route` ja selle eemaldada
    - Oskan lugeda `show ip route` väljundis `C`- ja `S`-ridu
    - Oskan selgitada, miks marsruut on vaja lisada mõlemas suunas
    - Oskan asendada staatilise marsruudi vaikemarsruudiga (default route)

!!! warning "Eeldused"
    Enne seda praktikumi pead olema läbinud [Praktikumi 7: Ruuteri avastamine ja seadistamine](praktikum_07.md) ja lugenud [peatükist 12](../teooria/12_marsruutimine.md) osad marsruutimistabelist, staatilistest marsruutidest ja vaikemarsruudist.

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer
    - Praktikumi 7 fail (kui seda pole, ehitad topoloogia nullist)
    - Seadmed: 2× ruuter Cisco 2811 (R1, R2), 2× kommutaator Cisco 2960 (SW1, SW2), 2× PC (PC1, PC2)

## Osa 1: Topoloogia ja aadressid

Praktikumis 7 oli LAN 2 otse R1 taga. Nüüd tõstad selle teise ruuteri R2 taha. R1 ja R2 vahel on eraldi väike võrk (link), milles on ainult kaks aadressi.

```text
PC1 ──── SW1 ──── R1 ─────────── R2 ──── SW2 ──── PC2
            Fa0/0    Fa0/1   Fa0/1   Fa0/0
   LAN 1              link              LAN 2
192.168.1.0/24    10.0.0.0/30      192.168.2.0/24
```

| Seade | Liides | IP-aadress | Mask | Default gateway |
|---|---|---|---|---|
| R1 | Fa0/0 | 192.168.1.1 | 255.255.255.0 | |
| R1 | Fa0/1 | 10.0.0.1 | 255.255.255.252 | |
| R2 | Fa0/1 | 10.0.0.2 | 255.255.255.252 | |
| R2 | Fa0/0 | 192.168.2.1 | 255.255.255.0 | |
| PC1 | FastEthernet0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | FastEthernet0 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |

*Tabel 8.1. Aadressid*

LAN 1, PC1 ja PC2 aadressid on samad, mis praktikumis 7. Muutuvad ainult R1 liides Fa0/1 ja uus ruuter R2.

!!! info "Miks mask /30?"
    Ruuterite vahelises võrgus on ainult kaks seadet. Mask `255.255.255.252` (/30) annab neli aadressi: `10.0.0.0` on võrgu aadress, `10.0.0.3` on leviaadress ja seadmetele jäävad `10.0.0.1` ja `10.0.0.2`. Maskidest täpsemalt [Peatükk 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md).

### Samm 1: Muuda praktikumi 7 topoloogiat

1. Ava praktikumi 7 fail ja salvesta see kohe uue nimega: **File → Save As** → `Praktikum8_Eesnimi_Perenimi.pkt`.
2. Kustuta kaabel R1 Fa0/1 ja SW2 vahelt (tööriist **Delete** või klahv ++delete++ pärast kaabli valimist).
3. Lisa ruuter **2811** (Network Devices → Routers) ja pane selle nimeks R2 (kliki ruuteril → **Config** → **Display Name**).
4. Ühenda kaablid:

| Kust | Kuhu | Kaabel |
|---|---|---|
| R1 **Fa0/1** | R2 **Fa0/1** | **Copper Cross-Over** |
| R2 **Fa0/0** | SW2 Fa0/24 | Copper Straight-Through |

*Tabel 8.2. Uued kaablid*

??? note "Praktikumi 7 faili pole?"
    Ehita topoloogia nullist: 2 ruuterit 2811 (R1, R2), 2 kommutaatorit 2960 (SW1, SW2) ja 2 arvutit (PC1, PC2).

    | Kust | Kuhu | Kaabel |
    |---|---|---|
    | PC1 FastEthernet0 | SW1 Fa0/1 | Copper Straight-Through |
    | SW1 Fa0/24 | R1 Fa0/0 | Copper Straight-Through |
    | R1 Fa0/1 | R2 Fa0/1 | Copper Cross-Over |
    | R2 Fa0/0 | SW2 Fa0/24 | Copper Straight-Through |
    | SW2 Fa0/1 | PC2 FastEthernet0 | Copper Straight-Through |

    Seadista R1 nagu praktikumis 7 (hostinimi, Fa0/0 aadress `192.168.1.1/24`, `no shutdown`) ning PC1 ja PC2 tabeli 8.1 järgi. Seejärel jätka sammuga 2.

!!! info "Miks Cross-Over?"
    Kahe sama tüüpi seadme (ruuter ja ruuter) vahel on vaja ristkaablit. Packet Traceri 2811 FastEtherneti liides ei vaheta saate- ja vastuvõtupaare ise, seega sirge kaabliga jääb link olekusse `down`.

!!! success "Oodatav tulemus"
    R1 ja R2 vahel on tuled punased, sest R2 liidesed on vaikimisi välja lülitatud.

### Samm 2: R1 liides lingi poole

R1 Fa0/1 aadress oli `192.168.2.1`. Anna sellele lingi aadress. Uus `ip address` käsk asendab vana aadressi.

```text
R1> enable
R1# configure terminal
R1(config)# interface fastethernet 0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# description Link R2
R1(config-if)# no shutdown
R1(config-if)# end
```

Kui R1 küsib parooli, kasuta praktikumis 7 seatud paroole: konsool `console123`, käsu `enable` järel `cisco123`.

### Samm 3: R2 seadistus

Kliki R2 → **CLI**. Küsimusele `Would you like to enter the initial configuration dialog? [yes/no]` vasta `no`.

```text
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# interface fastethernet 0/1
R2(config-if)# ip address 10.0.0.2 255.255.255.252
R2(config-if)# description Link R1
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface fastethernet 0/0
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# description LAN 2
R2(config-if)# no shutdown
R2(config-if)# end
```

Kontrolli mõlemal ruuteril:

```text
R2# show ip interface brief
```

!!! success "Oodatav tulemus"
    R2 real `FastEthernet0/0` on `192.168.2.1` ja real `FastEthernet0/1` on `10.0.0.2`, mõlemal olek `up up`. R1 peal on samamoodi `192.168.1.1` ja `10.0.0.1`. Kõik tuled topoloogias on rohelised.

??? warning "R1 ja R2 vaheline liides on `down down`"
    Kaabel on vale tüüpi või vales pordis. Kustuta kaabel ja ühenda uuesti **Copper Cross-Over** kaabliga R1 Fa0/1 ja R2 Fa0/1 vahel.

### Samm 4: Mida ruuterid teavad?

```text
R1# show ip route
```

!!! success "Oodatav tulemus"
    Väljundi lõpus on kaks `C`-rida:

    ```text
         10.0.0.0/30 is subnetted, 1 subnets
    C       10.0.0.0 is directly connected, FastEthernet0/1
    C    192.168.1.0/24 is directly connected, FastEthernet0/0
    ```

    Rida `10.0.0.0/30 is subnetted, 1 subnets` on rühma pealkiri: IOS 12.4 näitab /30 võrgu maski selles reas ja selle all olevas `C`-reas on ainult võrgu aadress. Packet Traceri 2811 ei näita `L`-ridu, need on ainult uuemas IOS 15-s ([peatükk 12](../teooria/12_marsruutimine.md)).

R1 tabelis ei ole võrku `192.168.2.0/24`. Vaata ka R2 tabelit: seal ei ole võrku `192.168.1.0/24`.

## Osa 2: Ping enne marsruute

**Ennusta:** PC1-st pingid kolme aadressi: R1 lingi poolne liides `10.0.0.1`, R2 lingi poolne liides `10.0.0.2` ja PC2 `192.168.2.10`. Millised õnnestuvad?

PC1 → Desktop → **Command Prompt**:

```text
C:\> ping 10.0.0.1
C:\> ping 10.0.0.2
C:\> ping 192.168.2.10
```

??? success "Oodatav tulemus"
    - `ping 10.0.0.1` õnnestub.
    - `ping 10.0.0.2` annab `Request timed out`.
    - `ping 192.168.2.10` annab `Reply from 192.168.1.1: Destination host unreachable`.

**Küsimus:** miks on teise ja kolmanda pingi veateade erinev? Võrdle R1 ja R2 marsruutimistabelit: kumb ruuter ei tea, kuhu pakett saata?

## Osa 3: Staatilised marsruudid

### Samm 1: Marsruut R1-le

R1 peab teadma, et võrk `192.168.2.0/24` on R2 taga. R2 aadress lingis on `10.0.0.2`.

```text
R1# configure terminal
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
R1(config)# end
```

| Käsu osa | Tähendus |
|---|---|
| `192.168.2.0` | sihtvõrk |
| `255.255.255.0` | sihtvõrgu mask |
| `10.0.0.2` | järgmine hüpe (*next-hop*): naaberruuteri aadress, kuhu pakett saata |

*Tabel 8.3. Käsu `ip route` osad*

```text
R1# show ip route
```

!!! success "Oodatav tulemus"
    Tabelisse tuli `S`-rida:

    ```text
         10.0.0.0/30 is subnetted, 1 subnets
    C       10.0.0.0 is directly connected, FastEthernet0/1
    C    192.168.1.0/24 is directly connected, FastEthernet0/0
    S    192.168.2.0/24 [1/0] via 10.0.0.2
    ```

    `S` tähendab staatilist marsruuti. `[1/0]` on haldusdistants (*administrative distance*) ja meetrika, `via 10.0.0.2` on järgmine hüpe.

**Ennusta:** R1 teab nüüd teed PC2 võrku. Kas PC1-st `ping 192.168.2.10` õnnestub?

```text
C:\> ping 192.168.2.10
```

??? success "Oodatav tulemus"
    Ei õnnestu, kõik neli vastust on `Request timed out`.

**Küsimus:** kuhu päring jõudis ja miks vastust ei tulnud?

### Samm 2: Marsruut R2-le

```text
R2# configure terminal
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
R2(config)# end
R2# show ip route
```

!!! success "Oodatav tulemus"
    R2 tabelis on rida `S    192.168.1.0/24 [1/0] via 10.0.0.1`.

### Samm 3: Ping ja tracert läbi kahe ruuteri

PC1 Command Promptis:

```text
C:\> ping 192.168.2.10
C:\> ping 10.0.0.2
C:\> tracert 192.168.2.10
```

!!! success "Oodatav tulemus"
    Mõlemad pingid õnnestuvad. Esimene vastus võib olla `Request timed out`, sest ruuterid otsivad ARP-iga MAC-aadresse. Vastustes on `TTL=126`: arvuti alustab 128-st ja iga ruuter vähendab väärtust ühe võrra.

    Väljundist `tracert` loe, mitu hüpet pakett tegi ja millised aadressid on teel.

Proovi ka teist suunda: PC2-st `ping 192.168.1.10`.

### Samm 4: Salvesta

```text
R1# copy running-config startup-config
R2# copy running-config startup-config
```

Salvesta ka Packet Traceri fail (++ctrl+s++).

??? warning "Ping PC1-st PC2-le ei õnnestu"
    1. `R1# ping 10.0.0.2` ei õnnestu: link on maas. Kontrolli `show ip interface brief` ja kaablit (Cross-Over).
    2. `show ip route` mõlemal ruuteril: kas `S`-rida on olemas ja kas next-hop on **naaberruuteri** aadress (R1-l `10.0.0.2`, R2-l `10.0.0.1`)?
    3. Kas sihtvõrk ja mask on õiged? Vale marsruudi eemaldad käsuga `no ip route ...` (sama käsk, ees `no`) ja lisad õige.
    4. PC-de gateway: PC1 `192.168.1.1`, PC2 `192.168.2.1` (`ipconfig`).

## Osa 4: Lõhu meelega: link maha

Staatiline marsruut kehtib ainult siis, kui next-hop on kättesaadav.

**Ennusta:** lülitad R1 lingi poolse liidese välja. Kas `S`-rida jääb R1 tabelisse?

```text
R1# configure terminal
R1(config)# interface fastethernet 0/1
R1(config-if)# shutdown
R1(config-if)# end
R1# show ip route
```

??? success "Oodatav tulemus"
    Tabelist kadusid nii `C`-rida `10.0.0.0` kui ka `S`-rida `192.168.2.0/24`. Konfiguratsioonis on käsk alles: vaata `show running-config | include ip route`.

**Küsimus:** miks kadus `S`-rida? Millisesse otseühendatud võrku kuulub next-hop `10.0.0.2`?

Lülita liides tagasi sisse ja kontrolli, et `S`-rida tuli tagasi ja ping PC1-st PC2-le õnnestub:

```text
R1(config)# interface fastethernet 0/1
R1(config-if)# no shutdown
```

## Osa 5: Vaikemarsruut R2-l

R2 taga ei ole teisi ruutereid. Kõik, mis ei ole tema enda võrkudes, asub R1 suunas. Sellisel juhul piisab ühest vaikemarsruudist (*default route*): sihtvõrk `0.0.0.0` ja mask `0.0.0.0` sobivad iga aadressiga.

Eemalda R2 staatiline marsruut ja lisa vaikemarsruut:

```text
R2# configure terminal
R2(config)# no ip route 192.168.1.0 255.255.255.0 10.0.0.1
R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1
R2(config)# end
R2# show ip route
```

!!! success "Oodatav tulemus"
    ```text
    Gateway of last resort is 10.0.0.1 to network 0.0.0.0

         10.0.0.0/30 is subnetted, 1 subnets
    C       10.0.0.0 is directly connected, FastEthernet0/1
    C    192.168.2.0/24 is directly connected, FastEthernet0/0
    S*   0.0.0.0/0 [1/0] via 10.0.0.1
    ```

    `S*` on vaikemarsruut. Ping PC1 ja PC2 vahel õnnestub endiselt.

Salvesta R2 konfiguratsioon (`copy running-config startup-config`) ja Packet Traceri fail.

## Osa 6: Tõrkeotsing

??? warning "`% Invalid input detected`"
    Käsk `ip route` töötab ainult globaalses konfiguratsioonirežiimis: prompt peab olema `R1(config)#`. Kui oled liidese režiimis (`R1(config-if)#`), kirjuta enne `exit`.

??? warning "`S`-rida ei ilmu tabelisse"
    Next-hop ei kuulu ühtegi otseühendatud võrku (trükiviga aadressis või link on maas). Kontrolli `show ip interface brief` ja `show running-config | include ip route`.

??? warning "Ping töötab ühes suunas, aga mitte teises"
    Ping vajab mõlemat suunda: päring ja vastus. Kontrolli, et marsruut on olemas **mõlemal** ruuteril ja et mõlemal arvutil on gateway.

---

## Kokkuvõte

Selles praktikumis sa:

- Lisasid topoloogiasse teise ruuteri ja ühendasid ruuterid /30 lingiga
- Nägid, et ruuter teab ainult oma liideste taga olevaid võrke
- Lisasid staatilised marsruudid käsuga `ip route` mõlemal ruuteril
- Nägid, et ühest marsruudist ei piisa, sest vastus vajab teed tagasi
- Nägid, et marsruut kaob tabelist, kui next-hop pole kättesaadav
- Asendasid R2 marsruudi vaikemarsruudiga

Iseseisvaks harjutamiseks on [Praktikum 9](praktikum_09.md).
