---
tags:
  - Praktikum
  - CCNA
---

# Praktikum 2: CLI ja kommutaatori seadistamine

Selles praktikumis õpid Cisco kommutaatori põhilisi CLI käske ja seadistusi. Konfigureerid kommutaatori nime, paroolid, haldusaadressi ja testid ühenduvust. Kommutaator on [andmesidekihi](../teooria/04_andmesidekiht.md) seade, mis edastab Ethernet-kaadreid MAC-aadresside põhjal — selle praktikumi fookus on aga seadme **haldamine** käsurealt.

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan navigeerida Cisco IOS-i erinevate režiimide vahel
    - Oskan seadistada kommutaatori hostnime, paroole ja bännerit
    - Oskan konfigureerida VLAN 1 haldusaadressi
    - Oskan salvestada konfiguratsiooni püsimällu
    - Oskan kasutada `show` käske konfiguratsiooni kontrollimiseks

!!! warning "Eeldused"
    Enne seda praktikumi pead olema läbinud [Peatükk 0a: Võrguseadmed](../teooria/00a_vorguseadmed.md) — kommutaatori roll võrgus.

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer
    - Topoloogia: 1× Cisco 2960 kommutaator (S1), 1× PC (PC-A)

## Osa 1: Topoloogia ja seadmete ettevalmistamine

### Samm 1: Topoloogia ülesehitamine

```text
[PC-A] ---- [S1]
            Fa0/6
```

1. Lisa Packet Traceris 1× Switch 2960 ja 1× PC
2. Ühenda PC-A kommutaatori porti **Fa0/6** (Copper Straight-Through)

### Samm 2: PC-A seadistamine

Kliki PC-A → **Desktop** → **IP Configuration** → **Static**:

```text
IP Address:      192.168.1.2
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

## Osa 2: Kommutaatori põhiseadistamine

### Samm 1: CLI avamine ja režiimid

Kliki kommutaatoril → **CLI** tab. Kui küsib *"Continue with configuration dialog?"*, vasta **no** ja vajuta Enter.

Cisco IOS (*Internetwork Operating System*) on operatsioonisüsteem, mis jookseb enamikul Cisco kommutaatoritel ja ruuteritel. Seadme haldamine toimub käsurealt (CLI — *Command Line Interface*), kus liigud erinevate režiimide vahel vastavalt sellele, mida parasjagu tegema pead.

IOS-i kolm põhirežiimi:

| Režiim | Prompt | Sisenemine |
|---|---|---|
| User EXEC | `Switch>` | Algne režiim |
| Privileged EXEC | `Switch#` | `enable` |
| Global Config | `Switch(config)#` | `configure terminal` |

*Tabel 2.1. Cisco IOS-i režiimid*

### Samm 2: Privilegeeritud režiim ja konfiguratsiooni vaatamine

```text
Switch> enable
Switch# show running-config
```

### Samm 3: Hostnime ja parooli seadistamine

Hostinimi aitab seadet tuvastada — eriti oluliseks muutub see siis, kui võrgus on palju seadmeid. `enable secret` parool kaitseb privilegeeritud režiimi, kusjuures `secret` salvestab parooli **MD5 räsina**, mitte lihttekstina (erinevalt vanast `enable password` käsust).

```text
Switch# configure terminal
Switch(config)# hostname S1
S1(config)# enable secret class
S1(config)# no ip domain-lookup
```

!!! tip "no ip domain-lookup"
    See käsk takistab kommutaatoril valesti sisestatud käske DNS-ist otsimast. Ilma selleta ootab kommutaator iga kirjavea puhul ~30 sekundit DNS vastust.

### Samm 4: Konsooliparooli seadistamine

Konsooliport (`line console 0`) on füüsiline ühendus, mille kaudu seadet kohapeal hallatakse (nt sülearvuti + konsoolikaabel). Parool kaitseb seda juurdepääsuteed.

```text
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
```

### Samm 5: VTY (kaugjuurdepääsu) parooli seadistamine

VTY (*Virtual Terminal*) liinid võimaldavad seadmele ligi pääseda üle võrgu (Telnet/SSH). Cisco kommutaatoril on vaikimisi 16 VTY liini (0–15), mis tähendab, et korraga saab kuni 16 administraatorit kaugühenduse luua.

```text
S1(config)# line vty 0 15
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
```

### Samm 6: Paroolide krüpteerimine ja bänner

`service password-encryption` krüpteerib kõik lihttekstina salvestatud paroolid konfiguratsioonis (Cisco tüüp 7 — nõrk, aga parem kui mitte midagi). Bänner (*Message of the Day*) kuvatakse igale sisselogijale ja on oluline nii juriidiliselt kui ka hoiatuseks.

```text
S1(config)# service password-encryption
S1(config)# banner motd #
************************************************
HOIATUS! Volitamata ligipääs on keelatud!
************************************************
#
```

## Osa 3: Haldusaadressi seadistamine

### Samm 1: VLAN 1 konfigureerimine

Kommutaator on Layer 2 seade ja tal pole tavaliselt IP-aadressi vaja andmete edastamiseks. Küll aga vajab ta IP-aadressi **halduse jaoks** — et sa saaksid talle üle võrgu ligi pääseda (Telnet/SSH). Selleks kasutatakse virtuaalset liidest VLAN 1. IP-aadresside tööd käsitleb põhjalikumalt [Peatükk 8: IPv4 adresseerimine](../teooria/08_ipv4.md).

```text
S1(config)# interface vlan 1
S1(config-if)# ip address 192.168.1.1 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit
```

!!! info "Miks VLAN 1?"
    VLAN 1 on kommutaatori vaikimisi haldusliides. `no shutdown` on vajalik, sest vaikimisi on liides välja lülitatud.

### Samm 2: Konfiguratsiooni salvestamine

```text
S1(config)# exit
S1# copy running-config startup-config
Destination filename [startup-config]?
```

Vajuta Enter. Ilma salvestamiseta kaob konfiguratsioon taaskäivitamisel.

## Osa 4: Kontrollimine ja testimine

### Samm 1: Konfiguratsiooni kontrollimine

```text
S1# show running-config
```

Kontrolli, et hostinimi, paroolid, bänner ja VLAN 1 aadress on paigas.

### Samm 2: Liideste staatus

```text
S1# show ip interface brief
```

Oodatav tulemus — VLAN 1 on **up/up**:

```text
Interface              IP-Address      OK? Method Status Protocol
Vlan1                  192.168.1.1     YES manual up     up
FastEthernet0/1        unassigned      YES unset  down   down
FastEthernet0/6        unassigned      YES unset  up     up
...
```

### Samm 3: Ühenduvuse testimine

Ava PC-A → **Command Prompt**:

```text
ping 192.168.1.1
```

Kui saad vastuse — kommutaatori haldusaadress on korrektne ja ühendus töötab.

## Osa 5: Tõrkeotsing

!!! warning "Ping kommutaatorile ei tööta?"
    1. Kas VLAN 1 on **up/up**? (`show ip interface brief`)
    2. Kas tegid **no shutdown** VLAN 1 liidesel?
    3. Kas PC-A ja S1 on **samas alamvõrgus**?
    4. Kas salvestasid konfiguratsiooni?

---

## Kokkuvõte

Selles praktikumis sa:

- Navigeerisid IOS-i kolme režiimi vahel (User → Privileged → Global Config)
- Seadistasid kommutaatori hostnime, enable secret parooli ja bänneri
- Konfigureerisid konsool- ja VTY paroolid kaugjuurdepääsu kaitseks
- Seadistasid VLAN 1 haldusaadressi ja aktiveerisid liidese
- Salvestasid konfiguratsiooni püsimällu käsuga `copy running-config startup-config`
