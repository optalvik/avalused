---
tags:
  - Praktikum
  - Ruutimine
---

# Praktikum 9: Staatiline marsruutimine (iseseisev harjutus)

See praktikum kinnistab [Praktikumis 8](praktikum_08.md) õpitut. Ehitad sama tüüpi topoloogia nullist, aga teiste ruuterite ja aadressidega. Käske juhend ette ei anna: iga ülesande juures on kokkuvolditud vihje, mida ava ainult siis, kui jääd kinni.

Teooria kordamiseks: [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md), [Peatükk 8: IPv4](../teooria/08_ipv4.md), [Peatükk 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan ilma juhisteta seadistada kahe ruuteri liidesed ja staatilised marsruudid
    - Oskan kasutada /30 maski ruuterite vahelises lingis
    - Oskan lugeda IOS 15 marsruutimistabelit, kus on ka `L`-read
    - Oskan kontrollida marsruutimist käskudega `show ip route` ja `ping`

## Topoloogia

```text
PC0 ─┐                                         ┌─ PC2
     ├── SW0 ──── R0 ─────────── R1 ──── SW1 ──┤
PC1 ─┘      G0/1    G0/0   G0/0    G0/1        └─ PC3
      LAN 1              link              LAN 2
192.168.70.0/24    221.123.1.0/30    172.230.10.0/24
```

| Seade | Liides | Võrk | IP-aadress | Mask |
|---|---|---|---|---|
| R0 | G0/0 | link | 221.123.1.1 | 255.255.255.252 |
| R0 | G0/1 | LAN 1 | 192.168.70.1 | 255.255.255.0 |
| R1 | G0/0 | link | 221.123.1.2 | 255.255.255.252 |
| R1 | G0/1 | LAN 2 | 172.230.10.1 | 255.255.255.0 |
| PC0 | NIC | LAN 1 | 192.168.70.2 | 255.255.255.0 |
| PC1 | NIC | LAN 1 | 192.168.70.3 | 255.255.255.0 |
| PC2 | NIC | LAN 2 | 172.230.10.2 | 255.255.255.0 |
| PC3 | NIC | LAN 2 | 172.230.10.3 | 255.255.255.0 |

*Tabel 9.1. IP-aadresside plaan*

## Osa 1: Topoloogia

Lisa seadmed: 2 ruuterit **2911** (R0, R1), 2 kommutaatorit 2960 (SW0, SW1) ja 4 arvutit (PC0–PC3).

| Ühendus | Port A | Port B | Kaabel |
|---|---|---|---|
| R0 ↔ R1 | G0/0 | G0/0 | Copper Cross-Over |
| R0 ↔ SW0 | G0/1 | Gig0/1 | Copper Straight-Through |
| R1 ↔ SW1 | G0/1 | Gig0/1 | Copper Straight-Through |
| PC0 ↔ SW0 | Fa0 | Fa0/1 | Copper Straight-Through |
| PC1 ↔ SW0 | Fa0 | Fa0/2 | Copper Straight-Through |
| PC2 ↔ SW1 | Fa0 | Fa0/1 | Copper Straight-Through |
| PC3 ↔ SW1 | Fa0 | Fa0/2 | Copper Straight-Through |

*Tabel 9.2. Kaabeldus*

Seadista arvutitele aadressid tabeli 9.1 järgi. LAN 1 arvutite gateway on `192.168.70.1`, LAN 2 arvutite gateway `172.230.10.1`.

## Osa 2: Ruuterite liidesed

**Ülesanne:** anna mõlemale ruuterile hostinimi ja seadista liidesed tabeli 9.1 järgi.

**Kontrolli:** `show ip interface brief` näitab mõlemal ruuteril kahte liidest olekus `up up`. PC0-st `ping 192.168.70.3` ja PC2-st `ping 172.230.10.3` õnnestuvad.

??? tip "Vihje: liidese seadistamine"
    ```text
    R0(config)# interface gigabitethernet 0/0
    R0(config-if)# ip address 221.123.1.1 255.255.255.252
    R0(config-if)# no shutdown
    ```

    Teise liidese ja R1 liidesed seadista samamoodi.

## Osa 3: Staatilised marsruudid

**Ülesanne:** lisa kummalegi ruuterile staatiline marsruut teise ruuteri taga olevasse LAN-i.

**Kontrolli:** R0 tabelis on `S`-rida võrgu `172.230.10.0` jaoks (`via 221.123.1.2`) ja R1 tabelis `S`-rida võrgu `192.168.70.0` jaoks (`via 221.123.1.1`).

??? tip "Vihje: käsu kuju"
    `ip route <sihtvõrk> <mask> <naaberruuteri aadress lingis>`, globaalses konfiguratsioonirežiimis. Vaata [Praktikum 8, Osa 3](praktikum_08.md#osa-3-staatilised-marsruudid).

!!! info "IOS 15 marsruutimistabel"
    Ruuter 2911 kasutab IOS 15. Erinevalt praktikumi 8 ruuterist 2811 on iga `C`-rea all ka `L`-rida (*Local*): liidese enda aadress maskiga /32, näiteks `L 192.168.70.1/32 is directly connected, GigabitEthernet0/1`. Tabelis on ka rühmade pealkirjad (`is subnetted`, `is variably subnetted`), mis näitavad, mitu alamvõrku ja maski on ühe klassipõhise võrgu sees.

## Osa 4: Lõpptest

**Kontrolli:** PC0-st `ping 172.230.10.2` ja PC2-st `ping 192.168.70.2` õnnestuvad. PC0-st `tracert 172.230.10.2` näitab kolme hüpet.

Salvesta mõlema ruuteri konfiguratsioon (`copy running-config startup-config`).

??? warning "Ping ei õnnestu"
    1. Kas `ping 221.123.1.2` R0-st õnnestub? Kui ei, on link maas: kontrolli kaablit ja `show ip interface brief`.
    2. Kas marsruut on olemas **mõlemal** ruuteril ja next-hop on naaberruuteri aadress?
    3. Kas arvutite gateway on nende enda LAN-i ruuteri liides?

---

## Kokkuvõte

Selles praktikumis sa:

- Ehitasid kahe ruuteri ja kahe kohtvõrguga topoloogia ilma juhisteta
- Kasutasid /30 maski ruuterite vahelises lingis
- Seadistasid staatilised marsruudid mõlemal ruuteril
- Lugesid IOS 15 marsruutimistabelit koos `L`-ridadega
- Testisid ühendust kahe kohtvõrgu vahel
