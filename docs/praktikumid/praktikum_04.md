---
tags:
  - Praktikum
  - CCNA
---

# Praktikum 4: Ruuteri seadistamine ja DHCP

Selles praktikumis ehitad komplekssema võrgu kahe alamvõrguga, konfigureerid ruuteri, DHCP serveri ja testid võrkudevahelist ühenduvust. See on seniste oskuste koondpraktikum.

DHCP (*Dynamic Host Configuration Protocol*) võimaldab arvutitel saada IP-aadressi automaatselt, ilma käsitsi seadistamata. DHCP ja DNS teenuseid käsitleb [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md). Ruuteri rolli erinevate võrkude ühendamisel käsitleb [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan ehitada kahe alamvõrguga topoloogia ruuteri, kommutaatorite ja arvutitega
    - Oskan konfigureerida ruuteri mitu liidest erinevate alamvõrkude jaoks
    - Oskan seadistada DHCP serveri ja DHCP poole
    - Oskan konfigureerida `ip helper-address` DHCP relay jaoks
    - Oskan eristada staatilist ja dünaamilist IP konfiguratsiooni
    - Oskan lugeda marsruutimistabelit (`show ip route`)

!!! warning "Eeldused"
    Enne seda praktikumi pead olema läbinud [Praktikum 3](praktikum_03.md) ja tundma ruuteri ning kommutaatori põhiseadistamist. Kasuks tuleb [Peatükk 8: IPv4 adresseerimine](../teooria/08_ipv4.md).

!!! info "Vajalikud materjalid"
    - Cisco Packet Tracer
    - Seadmed: 1× ruuter 2911 (R1), 2× kommutaator 2960 (SW1, SW2), 4× PC, 1× Server

## Osa 1: Topoloogia ja adresseerimistabel

### Topoloogia

```text
                    [Server-DHCP]
                          |
                        Fa0/5
                          |
[PC1]---[PC2]---[SW1]---[R1]---[SW2]---[PC3]---[PC4]
 .10     DHCP           |   |          .30    DHCP
              Gi0/0    Gi0/1          Fa0/1
         192.168.1.0/24      192.168.2.0/24
```

### Adresseerimistabel

| Seade | Liides | IP-aadress | Alamvõrgumask | Default gateway |
|---|---|---|---|---|
| R1 | Gi0/0 | 192.168.1.1 | 255.255.255.0 | — |
| R1 | Gi0/1 | 192.168.2.1 | 255.255.255.0 | — |
| Server-DHCP | NIC | 192.168.1.100 | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.10 (staatiline) | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | DHCP | — | — |
| PC3 | NIC | 192.168.2.30 (staatiline) | 255.255.255.0 | 192.168.2.1 |
| PC4 | NIC | DHCP | — | — |

*Tabel 4.1. Võrgu adresseerimistabel*

### Samm 1: Seadmete lisamine ja ühendamine

| Seade A | Port A | Seade B | Port B |
|---|---|---|---|
| PC1 | FastEthernet0 | SW1 | Fa0/1 |
| PC2 | FastEthernet0 | SW1 | Fa0/2 |
| Server-DHCP | FastEthernet0 | SW1 | Fa0/5 |
| SW1 | Gi0/1 | R1 | Gi0/0 |
| R1 | Gi0/1 | SW2 | Gi0/1 |
| PC3 | FastEthernet0 | SW2 | Fa0/1 |
| PC4 | FastEthernet0 | SW2 | Fa0/2 |

*Tabel 4.2. Seadmete ühendused*

!!! tip "GigabitEthernet pordid"
    Kasuta kommutaatorite ja ruuteri vaheliseks ühenduseks **GigabitEthernet** porte (1 Gbps), mitte FastEthernet (100 Mbps).

## Osa 2: Ruuteri konfigureerimine

### Samm 1: Põhiseadistus

```text
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# enable secret cisco123
R1(config)# banner motd #Volitamata ligipääs keelatud!#
```

### Samm 2: Liideste konfigureerimine

**Gi0/0** (Network 1 pool):

```text
R1(config)# interface gigabitethernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description Connection to SW1 - Network1
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Gi0/1** (Network 2 pool):

```text
R1(config)# interface gigabitethernet 0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# description Connection to SW2 - Network2
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Samm 3: DHCP helper-address

Network 2 arvutid vajavad DHCP teenust, aga server asub Network 1-s. Kuna DHCP-päringud on **broadcast**-paketid ja ruuter vaikimisi broadcast’e ei edasta, vajame `ip helper-address` käsku, mis käsib ruuteril DHCP-päringud unicast’ina serverile edasi suunata. Seda nimetatakse **DHCP relay** funktsionaalsuseks.

```text
R1(config)# interface gigabitethernet 0/1
R1(config-if)# ip helper-address 192.168.1.100
R1(config-if)# exit
```

### Samm 4: Salvestamine ja kontrollimine

```text
R1(config)# exit
R1# copy running-config startup-config
R1# show ip interface brief
```

Mõlemad liidesed peavad olema **up/up**:

```text
Interface              IP-Address      Status    Protocol
GigabitEthernet0/0     192.168.1.1     up        up
GigabitEthernet0/1     192.168.2.1     up        up
```

## Osa 3: DHCP serveri seadistamine

### Samm 1: Serveri staatiline IP

Kliki Server-DHCP → **Desktop** → **IP Configuration** → **Static**:

```text
IP Address:      192.168.1.100
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

### Samm 2: DHCP teenuse konfigureerimine

Kliki Server-DHCP → **Services** → **DHCP**.

**Pool 1 — Network1:**

| Säte | Väärtus |
|---|---|
| Pool Name | Network1 |
| Default Gateway | 192.168.1.1 |
| DNS Server | 8.8.8.8 |
| Start IP Address | 192.168.1.10 |
| Subnet Mask | 255.255.255.0 |
| Max Users | 50 |

*Tabel 4.3. DHCP pool Network1*

Kliki **Save**, seejärel **Add** uue pooli jaoks.

**Pool 2 — Network2:**

| Säte | Väärtus |
|---|---|
| Pool Name | Network2 |
| Default Gateway | 192.168.2.1 |
| DNS Server | 8.8.8.8 |
| Start IP Address | 192.168.2.10 |
| Subnet Mask | 255.255.255.0 |
| Max Users | 50 |

*Tabel 4.4. DHCP pool Network2*

Kliki **Save**. Veendu, et **Service = On**.

## Osa 4: PC-de ja kommutaatorite konfigureerimine

### Staatilised IP-d

**PC1:** IP `192.168.1.10`, mask `255.255.255.0`, gateway `192.168.1.1`

**PC3:** IP `192.168.2.30`, mask `255.255.255.0`, gateway `192.168.2.1`

### Dünaamilised IP-d (DHCP)

**PC2** ja **PC4:** vali **Desktop** → **IP Configuration** → **DHCP**. Oota ~5 sekundit — DHCP server määrab IP-aadressi automaatselt.

!!! info "Staatiline vs dünaamiline"
    Staatiline IP — serveritele, printeritele, võrguseadmetele — nende aadress ei tohi muutuda. Dünaamiline (DHCP) — tavakasutajate arvutitele ja telefonidele, kus konkreetne aadress pole oluline. DHCP tööpõhimõtteid (DORA protsess) käsitleb [Peatükk 13: DHCP ja DNS](../teooria/13_dhcp_dns.md).

### Kommutaatorite põhiseadistus

**SW1:**

```text
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret switch123
SW1(config)# exit
SW1# copy running-config startup-config
```

**SW2:** sama, aga `hostname SW2`.

## Osa 5: Testimine

### Ping testid

**PC1 → Network 1 seadmed:**

```text
ping 192.168.1.1
ping 192.168.1.100
```

**PC1 → Network 2 (läbi ruuteri):**

```text
ping 192.168.2.30
```

Kõik peavad andma vastuse — see tõestab, et ruuter suunab pakette korrektselt.

### Marsruutimistabeli kontrollimine

```text
R1# show ip route
```

```text
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
L    192.168.1.1/32 is directly connected, GigabitEthernet0/0
C    192.168.2.0/24 is directly connected, GigabitEthernet0/1
L    192.168.2.1/32 is directly connected, GigabitEthernet0/1
```

**C** = *Connected* — otseühendatud võrk. Ruuter teab automaatselt, millised võrgud on tema liideste taga. Marsruutimistabeli lugemist käsitleb [Peatükk 12: Marsruutimine](../teooria/12_marsruutimine.md).

### DHCP lease test

```text
C:\> ipconfig /release
C:\> ipconfig /renew
C:\> ipconfig
```

`release` tagastab IP servile, `renew` küsib uue.

## Osa 6: Tõrkeotsing

!!! warning "Ping ei tööta võrkude vahel?"
    1. Kas ruuteri mõlemad liidesed on **up/up**?
    2. Kas PC-de default gateway viitab **ruuteri** IP-le?
    3. Kas tegid `no shutdown` mõlemal liidesel?

!!! warning "PC4 ei saa DHCP IP-d?"
    1. Kas DHCP Service on **On** serveris?
    2. Kas Network2 pool on loodud?
    3. Kas `ip helper-address 192.168.1.100` on R1 Gi0/1 liidesel?

!!! tip "Troubleshooting harjutus"
    Proovi `shutdown` üks ruuteri liides ja testi uuesti. Kasuta `show ip interface brief` vea leidmiseks, seejärel paranda `no shutdown`-iga.

---

## Kokkuvõte

Selles praktikumis sa:

- Ehitasid kahe alamvõrguga topoloogia 8 seadmega
- Konfigureerisid ruuteri kaks liidest erinevatele alamvõrkudele
- Seadistasid DHCP serveri kahe pooliga ja `ip helper-address` relay
- Konfigureerisid staatilised ja dünaamilised IP-aadressid
- Testisid ühenduvust läbi ruuteri ja kontrollisid marsruutimistabelit
- Harjutasid tõrkeotsingut `show ip interface brief` abil
