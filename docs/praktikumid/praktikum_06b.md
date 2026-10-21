---
tags:
  - Praktikum
  - Ethernet
---

# Praktikum 6b: MAC-tabel päris kommutaatoril

Praktikumis 6 nägid Packet Traceris, kuidas kommutaator MAC-aadresse õpib. Nüüd teed sama **päris kommutaatoriga serveriruumis**. Päris seade on teistsugune: pordid on füüsilised, kaablid tuleb ise õigesse kohta panna ja Windows ei vasta pingile enne, kui oled seda lubanud.

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan ühenduda päris kommutaatoriga konsoolikaabli ja PuTTY kaudu
    - Oskan leida arvuti MAC-aadressi Windowsis (`ipconfig /all`)
    - Oskan lugeda päris kommutaatori MAC-tabelit ja seostada iga rea konkreetse kaabli ja arvutiga
    - Oskan selgitada, miks MAC-tabel täitub alles siis, kui arvutid midagi saadavad
    - Oskan näidata `arp -a` abil, kuidas arvuti leiab IP-aadressi järgi MAC-aadressi

!!! warning "Eeldused"
    - [Praktikum 6: MAC-aadresside tabel](praktikum_06.md) (Packet Tracer)
    - [Peatükk 4: Andmesidekiht](../teooria/04_andmesidekiht.md)
    - Konsooliühendus ja PuTTY (Serial, 9600): vaata enne [videot PuTTY ühendusest](https://drive.google.com/file/d/1y2phgdB0MZ0k9rTrmo_ES2ae5jxHEo1a/view?usp=sharing) ja [juhendit: klassi arvuti, konsoolikaabel, PuTTY](https://docs.google.com/document/d/12NH_-liylELLgUhspfoTXxHba21JFe0_eW3Jok0Zffk/edit?usp=sharing)

!!! info "Mis kihil see on?"
    Kommutaator on **2. kihi** (andmesidekiht, *Data Link*) seade: ta loeb kaadrist **MAC-aadressi** ja saadab kaadri õigesse porti. IP-aadressi ta ei vaata. IP-aadressidega töötab **ruuter**, mis on **3. kihi** (võrgukiht, *Network*) seade ja ühendab erinevaid võrke. Täna on PC-A ja PC-B samas võrgus (192.168.1.0/24), seega ruuterit pole vaja: kommutaatorist piisab.

    Loe juurde: [OSI mudel, tabel 1.2](../teooria/01_protokollid_mudelid.md#osi-mudel), [Kommutaator — „tark postkontor“](../teooria/04_andmesidekiht.md#kommutaator-tark-postkontor), [Ruuter](../teooria/06_vorgukiht.md#ruuter-vorkude-vaheline-teejuht).

!!! info "Vajalikud materjalid (ühe grupi kohta)"
    - 1× Cisco kommutaator serveriruumis (number on valgel kleebisel)
    - 1× konsoolikaabel + arvuti PuTTY-ga (**konsooliarvuti**)
    - 2× arvutit või sülearvutit (**PC-A** ja **PC-B**)
    - 2× Ethernet-kaablit (sobib ka praktikum 5 omatehtud kaabel)

!!! note "Grupitöö"
    Grupis on 2–3 inimest. Jagage rollid ära ja vahetage Osa 4 juures:

    | Roll | Mida teeb |
    |---|---|
    | Konsool | Istub PuTTY taga ja trükib kommutaatori käsud |
    | PC-A / PC-B | Seadistab oma arvuti ja teeb pingi |
    | Kirjutaja | Täidab tabelid (kui on 3 inimest) |

## Topoloogia

```text
   [Konsooliarvuti]
          |  konsoolikaabel (CONSOLE port)
          |
   [ SW-xx kommutaator ]
      |            |
   port 1       port 2      Ethernet-kaablid
      |            |
   [PC-A]       [PC-B]
   192.168.1.10 192.168.1.20  /24
```

*Joonis 6b.1. Grupi topoloogia*

## Osa 1: Konsooliühendus ja puhas tabel

### Samm 1: Logi sisse

1. Ühenda konsoolikaabel kommutaatori **CONSOLE** porti.
2. PuTTY: **Serial**, kiirus **9600**, COM-port vaata Device Managerist. Kuidas täpselt: [video](https://drive.google.com/file/d/1y2phgdB0MZ0k9rTrmo_ES2ae5jxHEo1a/view?usp=sharing), [juhend](https://docs.google.com/document/d/12NH_-liylELLgUhspfoTXxHba21JFe0_eW3Jok0Zffk/edit?usp=sharing).
3. Vajuta Enter, kuni näed viipa `SW-xx>`.

```text
SW-xx> enable
Password: 1234
SW-xx#
```

!!! tip "Kui PuTTY aken jääb mustaks"
    Kontrolli COM-numbrit Device Manageris (**Ports (COM & LPT)**). Sageli on see COM3 või COM4, mitte COM1.

### Samm 2: Tühjenda MAC-tabel

Enne teid võis keegi teine samu kaableid kasutada. Alusta puhtalt lehelt:

```text
SW-xx# clear mac address-table dynamic
SW-xx# show mac address-table
```

Tabel peaks olema tühi (või seal on ainult `CPU` read, mis kuuluvad kommutaatorile endale).

💭 **Mõtle:** Packet Traceris oli tabel kohe alguses tühi. Miks pidid siin selle ise tühjaks tegema?

## Osa 2: Arvutite MAC-aadressid

### Samm 1: Leia oma MAC

PC-A ja PC-B mõlemas avage **cmd** ja:

```text
ipconfig /all
```

Leidke Etherneti adapteri (mitte Wi-Fi!) rida **Physical Address**.

| Arvuti | MAC Windowsis (`ipconfig /all`) | Kommutaatori port |
|---|---|---|
| PC-A | | |
| PC-B | | |

*Tabel 6b.1. Täida kohe MAC-veerg. Port tuleb hiljem.*

!!! warning "Wi-Fi vs Ethernet"
    Sülearvutil on mitu adapterit ja igaühel oma MAC. Wi-Fi MAC-i kommutaatori tabelist **ei leia**, sest Wi-Fi ei käi läbi selle kaabli.

## Osa 3: Ühenda, seadista IP, vaata tabelit

### Samm 1: Ühenda ainult PC-A

Ühenda PC-A kommutaatori porti 1. Oota, kuni pordi tuli muutub roheliseks (kuni 30 sekundit).

```text
SW-xx# show mac address-table
```

💭 **Mõtle:** Kas PC-A MAC ilmus tabelisse ilma, et oleksid midagi saatnud? Windows saadab ühendamisel ise mõned kaadrid (DHCP, teadaanded). Kas sinu tabelis on see näha?

### Samm 2: Ühenda PC-B ja anna IP-aadressid

Ühenda PC-B porti 2. Seadista mõlemale staatiline IP (**ncpa.cpl** → Etherneti adapter → IPv4):

| Arvuti | IP-aadress | Mask |
|---|---|---|
| PC-A | 192.168.1.10 | 255.255.255.0 |
| PC-B | 192.168.1.20 | 255.255.255.0 |

*Tabel 6b.2. IP-aadressid*

### Samm 3: Luba ping tulemüüris

Windowsi tulemüür **ei vasta vaikimisi pingile**. Mõlemas arvutis avage cmd **administraatorina**:

```text
netsh advfirewall firewall add rule name="ICMP lubatud" protocol=icmpv4:8,any dir=in action=allow
```

### Samm 4: Ping ja tabel

PC-A-st:

```text
ping 192.168.1.20
```

Kohe pärast seda konsoolis:

```text
SW-xx# show mac address-table
```

Näidis (sinu numbrid on teised):

```text
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    3c52.82a1.0f11    DYNAMIC     Fa0/1
   1    3c52.82a1.77e4    DYNAMIC     Fa0/2
```

Täida nüüd tabelis 6b.1 veerg **Kommutaatori port**.

!!! tip "MAC-aadress näeb teistsugune välja"
    Windows kirjutab `3C-52-82-A1-0F-11`, Cisco aga `3c52.82a1.0f11`. See on **sama aadress**, ainult teises kirjaviisis. Võrdle numbreid ja tähti, mitte kriipse ega punkte.

!!! warning "Kui ping ei vasta"
    | Näed | Kontrolli |
    |---|---|
    | `Request timed out` | Kas tulemüüri reegel lisati mõlemas arvutis? |
    | `Destination host unreachable` | Kas IP ja mask on õiged? Kas ping läks Etherneti, mitte Wi-Fi kaudu? Lülita Wi-Fi korraks välja. |
    | Tabelis on ainult üks arvuti | Kas pordi tuli põleb? Proovi teist kaablit. |

💭 **Mõtle:** Kuidas sa ilma kaablit jälgimata tead, milline rida tabelis on PC-A?

### Samm 5: ARP — kuidas PC-A sai teada PC-B MAC-i?

PC-A teadis ainult PC-B **IP-aadressi** (192.168.1.20). Kaadri saatmiseks on aga vaja **MAC-aadressi**. Selle küsis PC-A enne pingi ise, **ARP**-iga ([peatükk 5: ARP](../teooria/05_arp.md#kuidas-arp-tootab)).

PC-A-s cmd **administraatorina**: tühjenda ARP-tabel, tee ping ja vaata tabelit.

```text
arp -d *
ping 192.168.1.20
arp -a
```

Leia väljundist rida `192.168.1.20`. Selle kõrval on PC-B MAC ja tüüp `dynamic` ([ARP-tabel](../teooria/05_arp.md#arp-tabel-telefoniraamat)).

| Kust | PC-B MAC |
|---|---|
| PC-B `ipconfig /all` (tabel 6b.1) | |
| PC-A `arp -a` | |
| Kommutaatori tabel, port 2 | |

*Tabel 6b.4. Sama MAC kolmes kohas*

💭 **Mõtle:** ARP-päring („kes on 192.168.1.20?“) läheb **kõigile** (broadcast), vastus ainult PC-A-le. Kumb neist õpetas kommutaatorile, kus PC-B on?

### Samm 6: Ping teise võrku

PC-A-st:

```text
ping 8.8.8.8
```

Ping ei õnnestu. 8.8.8.8 ei ole sinu võrgus (192.168.1.0/24), seega ARP-iga sealt MAC-i ei küsi: keegi selles kaablis ei vasta. Teise võrku saadab arvuti paketi **ruuterile** ehk **default gateway**'le. Seda meie topoloogias pole, sest kommutaator on 2. kihi seade ja IP-aadresse ei vaata.

💭 **Mõtle:** Mis peaks sinu topoloogiasse lisanduma, et ping 8.8.8.8 töötaks? Vaata [peatükk 6: Ruuter](../teooria/06_vorgukiht.md#ruuter-vorkude-vaheline-teejuht) ja [Default gateway](../teooria/06_vorgukiht.md#default-gateway-uks-valismaailma).

## Osa 4: Kaabli ümberpanek

Vahetage nüüd rolle.

1. Tõmmake PC-B kaabel pordist 2 välja ja pange **porti 5**.
2. Tehke uuesti ping PC-A-st PC-B-le.
3. Vaadake tabelit uuesti.

| Küsimus | Vastus |
|---|---|
| Mis port on nüüd PC-B MAC-i juures? | |
| Kas vana rida (port 2) on veel alles? | |

*Tabel 6b.3. Pärast ümberpanekut*

💭 **Mõtle:** Kommutaator ei tea midagi kaablitest. Kust ta sai teada, et PC-B kolis teise porti?

## Osa 5: Mis juhtub, kui keegi ei räägi?

```text
SW-xx# show mac address-table aging-time
```

Vaikimisi on see 300 sekundit. Ärge 5 minutit midagi saatke (sulgege brauserid, ärge pingige), seejärel vaadake tabelit uuesti.

💭 **Mõtle:** Miks on kasulik, et kommutaator vanad read ise ära kustutab? Mõtle Osa 4 peale.

## ✅ Lõpukontroll

- [ ] Tabelis 6b.1 on mõlema arvuti MAC ja port ning need klapivad `show mac address-table` väljundiga
- [ ] Oskad näidata, mis rida tabelis kuulub sinu arvutile, ilma kaablit jälgimata
- [ ] Pärast kaabli ümberpanekut nägid tabelis uut porti
- [ ] Oskad öelda, miks Windowsi ja Cisco MAC näevad erinevad välja, kuigi on sama

## Lõpus: korista ära

Järgmine grupp kasutab sama kommutaatorit.

```text
SW-xx# clear mac address-table dynamic
```

- Võta oma arvutist staatiline IP maha (**Obtain an IP address automatically**)
- Kustuta tulemüüri reegel: `netsh advfirewall firewall delete rule name="ICMP lubatud"`
- Pane kaablid tagasi sinna, kust need võtsid

## Kokkuvõte

Selles praktikumis sa:

- Ühendusid päris kommutaatoriga konsooli kaudu
- Leidsid oma arvuti MAC-i Windowsis ja tundsid selle ära Cisco tabelis
- Nägid, et kommutaator õpib aadressi alles siis, kui arvuti saadab kaadri
- Nägid, et kaabli ümberpanekul uuendab kommutaator tabelit ise
