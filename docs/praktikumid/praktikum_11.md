---
tags:
  - Praktikum
---

# Praktikum 11: DHCP ja DNS

Selles praktikumis seadistad ruuteri DHCP serverina ja konfigureerid eraldiseisva DNS serveri. Arvutid saavad IP-aadressi automaatselt ja pääsevad veebilehele nimepõhiselt.

DHCP ja DNS on kaks kõige levinumat võrguteenust. DHCP jagab automaatselt IP-aadresse, vältides käsitsi seadistamise vajadust. DNS tõlgib inimloetavad nimed (nt `www.firma.lan`) IP-aadressideks. Mõlema teenuse teooriat käsitleb [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan konfigureerida ruuteri DHCP serveri funktsiooni (pool, excluded addresses)
    - Oskan konfigureerida DNS serverit Packet Traceris
    - Oskan kasutada `nslookup` käsku DNS testimiseks
    - Oskan kontrollida DHCP tööd käsuga `show ip dhcp binding`
    - Oskan selgitada DORA protsessi

!!! warning "Eeldused"
    Enne seda praktikumi pead olema tutvunud [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md) ja tundma ruuteri põhiseadistamist ([Praktikum 7](praktikum_07.md)).

## Topoloogia

```text
                [R1]
              Fa0/0 .1
                 |
           192.168.1.0/24
                 |
              [SW1]
           /    |    \
       Fa0/2  Fa0/3  Fa0/4
      [Server] [PC1]  [PC2]
        .10    DHCP   DHCP
```

| Seade | IP-aadress | Seadistus |
|---|---|---|
| R1 | 192.168.1.1 | Staatiline (gateway + DHCP server) |
| Server | 192.168.1.10 | Staatiline (DNS server) |
| PC1 | DHCP | Dünaamiline |
| PC2 | DHCP | Dünaamiline |

*Tabel 11.1. IP-konfiguratsioon*

## Osa 1: Topoloogia ja põhikonfiguratsioon

### Samm 1: Lisa seadmed ja ühenda

1× Router 2811, 1× Switch 2960, 1× Server, 2× PC

### Samm 2: Ruuteri liidese seadistamine

```text
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface FastEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Samm 3: Serveri staatiline IP

Server → **Desktop** → **IP Configuration**:

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
DNS Server:      192.168.1.10
```

## Osa 2: DHCP konfigureerimine

DHCP töötab **DORA** protsessi kaudu: **D**iscover (klient otsib serverit broadcast’iga) → **O**ffer (server pakub IP-d) → **R**equest (klient aktsepteerib pakkumise) → **A**cknowledge (server kinnitab). Kogu see protsess toimub automaatselt paari sekundiga. Lähemalt [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md).

### Samm 1: Excluded addresses

`excluded-address` määrab vahemiku, mida DHCP server **ei tohi** välja jagada. See on oluline selleks, et staatiliselt seadistatud seadmed (ruuter, server) ei saaks konfliktseid aadresse.

```text
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.49
```

!!! warning "Excluded enne pool-i!"
    Määra excluded addresses enne, kui kliendid IP-d küsivad. Vahemik .1–.49 on reserveeritud ruuterile, serverile ja teistele staatiliselt seadistatavtele seadmetele.

### Samm 2: DHCP pool

```text
R1(config)# ip dhcp pool KONTOR
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 192.168.1.10
R1(dhcp-config)# exit
```

### Samm 3: PC-de seadistamine

PC1 ja PC2 → **Desktop** → **IP Configuration** → **DHCP**

Oota kuni PC saab IP-aadressi (~5 sekundit).

### Samm 4: Kontrolli

```text
R1# show ip dhcp binding
```

Näitab väljajagatud IP-aadressid ja nendega seotud MAC-aadressid.

## Osa 3: DNS konfigureerimine

DNS (*Domain Name System*) on hajutatud andmebaas, mis tõlgib domeeninimed IP-aadressideks. Ilma DNS-ita peaksid kasutajad meeles pidama numbrilisi IP-aadresse. Selles praktikumis lood lihtsustatud lokaalse DNS-serveri, et harjutada põhimõtteid. Päriselt on DNS hierarhiline süsteem (juur- → TLD- → autoritatiivserverid) — seda käsitleb [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md).

### Samm 1: DNS teenuse aktiveerimine

Server → **Services** → **DNS** → lülita **On**

### Samm 2: Lisa DNS-kirjed

| Name | Type | Address |
|---|---|---|
| www.firma.lan | A Record | 192.168.1.10 |
| server.firma.lan | A Record | 192.168.1.10 |
| gateway.firma.lan | A Record | 192.168.1.1 |

*Tabel 11.2. DNS-kirjed*

Iga kirje jaoks täida väljad ja kliki **Add**.

## Osa 4: Testimine

### Samm 1: nslookup

PC1 → **Command Prompt**:

```bash
nslookup www.firma.lan
```

Peaks näitama serveri IP-d ja lahendatud aadressi.

### Samm 2: Ping nimega

```bash
ping www.firma.lan
ping gateway.firma.lan
```

### Samm 3: Veebibrauser

PC1 → **Desktop** → **Web Browser** → sisesta `http://www.firma.lan`

Leht peaks avanema — DNS lahendas nime IP-aadressiks ja HTTP server vastas.

## Osa 5: Tõrkeotsing

### DNS probleemi simuleerimine

1. Lülita DNS server **Off**
2. Testi `nslookup www.firma.lan` — ei tööta
3. Testi `ping 192.168.1.10` — töötab!

!!! info "Miks IP-ga ping töötab aga nimega mitte?"
    DNS ainult tõlgib nimed IP-aadressideks. Kui DNS ei tööta, nimelahendit ei toimu, aga IP-põhine suhtlus jätkub. See näitab, et DNS on **rakendusteenus**, mitte võrguteenuse osa — võrk ise töötab IP-aadressidega. Vt [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md).

---

## Kokkuvõte

Selles praktikumis sa:

- Konfigureerrisid ruuteri DHCP serverina (pool, excluded addresses, DNS)
- Seadistasid DNS serveri A-kirjetega
- Testisid DHCP tööd (`show ip dhcp binding`)
- Testisid DNS-i (`nslookup`, `ping` nimega)
- Avasid veebilehe DNS-nime kaudu
