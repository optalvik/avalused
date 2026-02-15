---
tags:
  - Praktikum
  - Ruutimine
---

# Praktikum 10: Dünaamiline marsruutimine

Selles praktikumis seadistad kolme ruuteriga võrgu ja katsetad erinevaid dünaamilise marsruutimise protokolle — RIP, OSPF ja EIGRP. Testid, kuidas võrk reageerib lingi katkemisele.

Erinevalt staatilisest marsruutimisest ([Praktikum 8](praktikum_08.md)), kus administraator lisab marsruudid käsitsi, vahetavad dünaamilised protokollid teavet **automaatselt** naaberruuteritega. See tähendab, et võrk suudab ise rikete korral alternatiivse tee leida. Marsruutimisprotokollide teooriat käsitleb [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan konfigureerida RIP, OSPF ja EIGRP protokolle
    - Oskan kasutada `tracert` käsku marsruudi jälgimiseks
    - Oskan selgitada dünaamilise marsruutimise reageerimist lingi rikke korral
    - Oskan võrrelda erinevaid marsruutimise protokolle

!!! warning "Eeldused"
    Enne seda praktikumi pead olema läbinud [Praktikum 8: Staatiline marsruutimine](praktikum_08.md) ja tutvunud [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md) — sealt leiad RIP, OSPF ja EIGRP teooria.

## Topoloogia

!!! note "Portide valik Packet Traceris"
    Cisco 2811 ruuteril on vaikimisi **FastEthernet0/0**, **FastEthernet0/1** ja **GigabitEthernet0/0**. See juhend kasutab WAN-linkide jaoks FastEthernet porte ja LAN-i jaoks GigabitEthernet porti.

```text
                    [R2]
                   /    \
            Fa0/0 /      \ Fa0/1
   Link A        /        \        Link B
   10.0.0.0/8   /          \       11.0.0.0/8
              Fa0/0        Fa0/1
               [R1]--------[R3]
                Fa0/1    Fa0/0
                   Link C
                  12.0.0.0/8
```

Iga ruuteri küljes on LAN:

- R1 → SW1 → PC1 (192.168.1.0/24)
- R2 → SW2 → PC2 (192.168.2.0/24)
- R3 → SW3 → PC3 (192.168.3.0/24)

| Seade | Liides | Võrk | IP-aadress |
|---|---|---|---|
| R1 | Fa0/0 | Link A | 10.0.0.1 |
| R1 | Fa0/1 | Link C | 12.0.0.1 |
| R1 | G0/0 | LAN 1 | 192.168.1.1 |
| R2 | Fa0/0 | Link A | 10.0.0.2 |
| R2 | Fa0/1 | Link B | 11.0.0.2 |
| R2 | G0/0 | LAN 2 | 192.168.2.1 |
| R3 | Fa0/0 | Link C | 12.0.0.3 |
| R3 | Fa0/1 | Link B | 11.0.0.3 |
| R3 | G0/0 | LAN 3 | 192.168.3.1 |

*Tabel 10.1. IP-aadresside plaan*

## Osa 1: Topoloogia ja põhikonfiguratsioon

### Samm 1: Lisa seadmed

- 3× Router 2811, 3× Switch 2960, 3× PC

### Samm 2: Kaabeldamine ja IP-aadressid

Konfigureeri kõik ruuteri liidested ja PC-de IP-aadressid vastavalt tabelile. Ära unusta `no shutdown`!

### Samm 3: Lokaalne test

Testi iga LAN-i sisest ühenduvust (PC → oma ruuter).

## Osa 2: RIP konfigureerimine

RIP (*Routing Information Protocol*) on vanim ja lihtsaim dünaamiline marsruutimisprotokoll. See kuulub **distantsvektorprotokollide** hulka — ruuterid jagavad perioodiliselt (iga 30 sek) oma marsruutimistabeleid naabritega. RIPv2 toetab klassivaba marsruutimist (CIDR), mistõttu kasutame alati `version 2` ja `no auto-summary`.

### R1

```text
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 10.0.0.0
R1(config-router)# network 12.0.0.0
R1(config-router)# network 192.168.1.0
R1(config-router)# no auto-summary
R1(config-router)# exit
```

### R2

```text
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 10.0.0.0
R2(config-router)# network 11.0.0.0
R2(config-router)# network 192.168.2.0
R2(config-router)# no auto-summary
R2(config-router)# exit
```

### R3

```text
R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# network 11.0.0.0
R3(config-router)# network 12.0.0.0
R3(config-router)# network 192.168.3.0
R3(config-router)# no auto-summary
R3(config-router)# exit
```

### Kontrolli

```text
R1# show ip route
```

**R** = RIP-marsruut. Oota ~30 sekundit, kuni ruuterid vahetavad infot.

### Traceroute test

PC1 → PC3:

```bash
tracert 192.168.3.2
```

Tulemus peaks näitama otseteed R1 → R3 (läbi Link C, 1 hop).

## Osa 3: Link failure test

Eesmärk: näha, kuidas RIP leiab automaatselt alternatiivse tee, kui otsene link katkeb.

### Samm 1: Katkesta Link C (R1 ↔ R3)

```text
R1(config)# interface Fa0/1
R1(config-if)# shutdown
```

### Samm 2: Oota ~60 sekundit ja testi uuesti

```bash
tracert 192.168.3.2
```

Nüüd peaks traceroute näitama alternatiivset teed: R1 → R2 → R3 (läbi Link A + Link B, 2 hopi). RIP leidis automaatselt ümbersõidutee!

### Samm 3: Taasta link

```text
R1(config)# interface Fa0/1
R1(config-if)# no shutdown
```

Oota ~30 sekundit ja käivita tracert uuesti — marsruut peaks taastuma otsetee peale (1 hop).

## Osa 4: OSPF konfigureerimine

OSPF (*Open Shortest Path First*) on **lingi-oleku** protokoll, mis ehitab kogu võrgust täieliku "kaardi" ja arvutab lühima tee Dijkstra algoritmiga. Erinevalt RIP-ist ei saada OSPF perioodiliselt tervet marsruutimistabelit, vaid ainult **muudatuste teateid** — see teeb ta oluliselt efektiivsemaks ja kiiremaks.

Eemalda esmalt RIP:

```text
R1(config)# no router rip
```

Korda kõigil kolmel ruuteril, seejärel seadista OSPF:

### R1

```text
R1(config)# router ospf 1
R1(config-router)# network 10.0.0.0 0.255.255.255 area 0
R1(config-router)# network 12.0.0.0 0.255.255.255 area 0
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# exit
```

### R2 ja R3

Korda sama mustriga, asendades võrguaadressid.

### Kontrolli

```text
R1# show ip route
R1# show ip ospf neighbor
```

**O** = OSPF-marsruut.

## Osa 5: EIGRP konfigureerimine

EIGRP (*Enhanced Interior Gateway Routing Protocol*) on Cisco hübriidprotokoll, mis ühendab distantsvektor- ja lingi-oleku protokollide parimad omadused. Ta kasutab **DUAL** (*Diffusing Update Algorithm*) algoritmi ja saavutab konvergentsi kõige kiiremini. EIGRP on Cisco proprietary, kuigi uuemad versioonid toetavad ka teisi tootjaid.

Eemalda OSPF ja seadista EIGRP:

```text
R1(config)# no router ospf 1
R1(config)# router eigrp 100
R1(config-router)# network 10.0.0.0
R1(config-router)# network 12.0.0.0
R1(config-router)# network 192.168.1.0
R1(config-router)# no auto-summary
R1(config-router)# exit
```

Korda R2 ja R3 jaoks.

```text
R1# show ip route
```

**D** = EIGRP-marsruut.

## Osa 6: Protokollide võrdlus

| Omadus | RIP | OSPF | EIGRP |
|---|---|---|---|
| Tüüp | Distance-vector | Link-state | Hybrid |
| Metric | Hop count | Cost (bandwidth) | Composite |
| Max hop | 15 | Piiranguta | 224 |
| Uuendused | 30 sek | Muutuste põhine | Muutuste põhine |
| Konvergents | Aeglane | Kiire | Väga kiire |
| Admin Distance | 120 | 110 | 90 |
| Routing table kood | **R** | **O** | **D** |

*Tabel 10.2. Marsruutimisprotokollide võrdlus*

---

## Kokkuvõte

Selles praktikumis sa:

- Konfigureerrisid RIP, OSPF ja EIGRP dünaamilise marsruutimise protokolle
- Testisid lingi katkemise mõju ja automaatset taastumist
- Kasutasid `tracert` käsku marsruudi jälgimiseks
- Võrdlesid protokollide erinevusi (metric, konvergents, admin distance)
