---
tags:
  - Praktikum
  - Ethernet
---

# Praktikum 6: MAC-aadresside tabel

Selles praktikumis uurid, kuidas kommutaator õpib MAC-aadresse ja teeb edastamisotsuseid. Kasutad Packet Tracer Simulation mode'i, et jälgida ARP- ja ICMP-kaadrite liikumist reaalajas.

MAC-aadresside tabel on [andmesidekihi](../teooria/04_andmesidekiht.md) kommutaatori südamik — just see tabel määrab, millisesse porti kaader edastatakse. ARP-protokoll, mis seob IP-aadresse MAC-aadressidega, on käsitletud [Peatükis 5: ARP](../teooria/05_arp.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan vaadata kommutaatori MAC-aadresside tabelit (`show mac address-table`)
    - Oskan selgitada, kuidas kommutaator õpib MAC-aadresse lähte-MAC väljast
    - Oskan eristada flooding'ut (tundmatu sihtkoht) ja forwarding'ut (teadaolev sihtkoht)
    - Oskan kasutada Simulation mode'i kaadrite teekonna jälgimiseks

!!! warning "Eeldused"
    Enne seda praktikumi pead olema tutvunud [Peatükk 4: Andmesidekiht](../teooria/04_andmesidekiht.md) (MAC-aadressid, kaadrite struktuur) ja [Peatükk 5: ARP](../teooria/05_arp.md).

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer
    - Seadmed: 1× Cisco 2960 kommutaator (SW1), 2× PC

## Osa 1: Topoloogia ja IP seadistamine

### Samm 1: Seadmete lisamine

```text
    [PC1]
      |
    [SW1]
      |
    [PC2]
```

Ühenda **Copper Straight-Through** kaabliga:

| Ühendus | PC port | Kommutaatori port |
|---|---|---|
| PC1 ↔ SW1 | FastEthernet0 | Fa0/1 |
| PC2 ↔ SW1 | FastEthernet0 | Fa0/2 |

*Tabel 6.1. Seadmete ühendused*

### Samm 2: IP-aadresside konfigureerimine

| PC | IP-aadress | Alamvõrgumask |
|---|---|---|
| PC1 | 192.168.1.10 | 255.255.255.0 |
| PC2 | 192.168.1.20 | 255.255.255.0 |

*Tabel 6.2. IP-aadresside tabel*

## Osa 2: MAC-aadresside kogumine

### Samm 1: MAC-aadresside vaatamine

Ava mõlema PC **Command Prompt** ja sisesta:

```text
ipconfig /all
```

Kirjuta üles **Physical Address** (MAC-aadress) mõlema PC jaoks.

### Samm 2: Tühi MAC-tabel

Ava kommutaatori CLI:

```text
Switch> enable
Switch# show mac address-table
```

Tabel on **tühi** — kommutaator ei tea veel, millises pordis millised seadmed asuvad.

## Osa 3: Ping ja MAC-tabeli täitumine

### Samm 1: Ping test

Ava PC1 → **Command Prompt**:

```text
ping 192.168.1.20
```

### Samm 2: MAC-tabel pärast pingi

```text
Switch# show mac address-table
```

Nüüd näed:

```text
Vlan    Mac Address       Type      Ports
----    -----------       --------  -----
   1    0001.xxxx.xxxx    DYNAMIC   Fa0/1
   1    0002.xxxx.xxxx    DYNAMIC   Fa0/2
```

Kommutaator **õppis** mõlema PC MAC-aadressi ja teab nüüd, millises pordis kumbki asub.

## Osa 4: Simulation mode — kaadrite jälgimine

### Samm 1: Ettevalmistus

1. Kustuta mõlema PC ARP-tabel: `arp -d`
2. Lülita PT paremas alanurgas **Simulation** režiimi
3. **Edit Filters** → näita ainult **ARP** ja **ICMP**

### Samm 2: Ping saatmine Simulation režiimis

Ava PC1 → **Command Prompt**:

```text
ping 192.168.1.20 -n 1
```

Vajuta PT-s **Play** (▶) ja jälgi kaadrite liikumist.

### Samm 3: Kaadrite analüüs

**Kaader 1 — ARP Request (broadcast):**

Saatja (PC1) ei tea PC2 MAC-aadressi. Saadab ARP Request sihtaadressiga `FF:FF:FF:FF:FF:FF`. Kommutaator teeb **flooding** — saadab kaadri **kõigile portidele** peale selle, kust see tuli.

**Kaader 2 — ARP Reply (unicast):**

PC2 vastab oma MAC-aadressiga. Kommutaator teab juba PC1 asukohta (õppis kaader 1 lähte-MAC-ist) ja saadab vastuse **ainult porti Fa0/1**.

**Kaadrid 3–4 — ICMP (ping):**

Kommutaator teab nüüd mõlema seadme asukohta ja saadab ping request/reply **otse õigesse porti**.

!!! tip "Põhiidee"
    Kommutaator õpib **lähte-MAC** väljast, mitte siht-MAC-ist. Esimene kaader alati flooding, edasi forwarding.

## Osa 5: Lisauurimine

### Aging time

```text
Switch# show mac address-table aging-time
```

Vaikimisi **300 sekundit** (5 minutit). Kui selle aja jooksul seadmelt ühtegi kaadrit ei tule, kustutatakse kirje tabelist.

### Broadcast test

```text
ping 192.168.1.255
```

Broadcast-aadressile saadetud paketid edastatakse alati **kõigile portidele**. `FF:FF:FF:FF:FF:FF` ei salvestata MAC-tabelisse.

---

## Kokkuvõte

Selles praktikumis sa:

- Vaatasid kommutaatori tühja ja täidetud MAC-aadresside tabelit
- Jälgisid Simulation režiimis, kuidas ARP Request on broadcast ja ARP Reply unicast
- Nägid, kuidas kommutaator õpib MAC-aadresse lähte-MAC väljast
- Eristasid flooding'ut (tundmatu sihtkoht) ja forwarding'ut (teadaolev sihtkoht)
- Uurisid MAC-tabeli aging time'i vaikeväärtust
