---
tags:
  - CCNA
  - Võrgud
---

# Õppematerjalide info

**Arvutivõrkude alused — teooriast praktikani**

| | |
|---|---|
| **Autor** | Maria Talvik · IT-õpetaja |
| **Õppeasutus** | Haapsalu Kutsehariduskeskus |
| **Sihtrühm** | IT-süsteemide nooremspetsialist (tase 4) |
| **Maht** | ~32 akadeemilist tundi (teooria + praktikumid) |
| **Hindamine** | Kujundav (enesekontrolliküsimused + praktikumid) |
| **Litsents** | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) |

## Eesmärk

Kursuse läbinu mõistab arvutivõrkude toimimise põhimõtteid, tunneb OSI ja TCP/IP mudeleid, oskab adresseerida ja alamvõrgustada IPv4 võrke ning seadistada Cisco seadmeid käsurealt. Kursus valmistab ette CCNA 200-301 eksami võrgutehnoloogia osaks.

## Eeldused

Arvutikasutuse baasteadmised. Eelnevad teadmised võrkude kohta ei ole vajalikud — kursus alustab nullist.

## Seos õppekavaga

Info- ja kommunikatsioonitehnoloogia erialade riiklik õppekava (RT I, 14.04.2020, 3):

| Moodul | EKAP | Seos kursusega |
|---|---|---|
| Arvutivõrgud | 16 | Kogu kursuse põhifookus |
| Küberturvalisus | 8 | WiFi turvalisus, tulemüürid, ARP spoofing |
| Majutuskeskkonna riistvara | 5 | Võrguseadmed, kaabeldus, struktureeritud kaabeldus |
| IT-valdkonna alusteadmised | 10 | OSI/TCP/IP mudelid, IP-adresseerimine |

## Struktuur ja maht

| Osa | Maht | Teemad |
|---|---|---|
| Sissejuhatus (osad 0–0c) | 4 h | Võrkude ajalugu, seadmed, topoloogiad, võrgutüübid, Eesti interneti infrastruktuur |
| Protokollid ja kihid (osad 1–7) | 14 h | OSI/TCP/IP mudelid, füüsiline kiht, WiFi, Ethernet, MAC, ARP, IP, TCP/UDP |
| Adresseerimine (osad 8–11) | 8 h | IPv4, kahendsüsteem, alamvõrgustamine, VLSM |
| Teenused ja marsruutimine (osad 12–14) | 6 h | Marsruutimine, DHCP, DNS, tõrkeotsing |
| Praktikumid (1–11) | ~16 h | Packet Tracer laborid, kaabli ehitamine, seadmete seadistamine |

*Tabel 1. Kursuse struktuur*

## Õpiväljundid

| Nr | Kursuse läbinu... | Bloomi tase | Osad |
|---|---|---|---|
| 1 | selgitab arvutivõrkude ajalugu, eesmärki ja seadmete rolle | mõistmine | 0–0c |
| 2 | kirjeldab OSI ja TCP/IP mudelite kihte ning kapseldamise protsessi | mõistmine | 1 |
| 3 | eristab kaablitüüpe ja juhtmevabasid tehnoloogiaid (WiFi, Bluetooth, 5G) | mõistmine | 2–3 |
| 4 | selgitab Ethernet-kaadri ülesehitust, MAC-aadresse ja kommutaatori tööd | mõistmine | 4 |
| 5 | selgitab ARP tööpõhimõtet (IP → MAC lahendamine) | mõistmine | 5 |
| 6 | kirjeldab ruuteri rolli, default gateway ja marsruutimistabelit | mõistmine | 6, 12 |
| 7 | eristab TCP ja UDP protokolle ning nimetab levinumaid pordinumbreid | mõistmine | 7 |
| 8 | teisendab numbreid kahend- ja kümnendsüsteemi vahel | rakendamine | 9 |
| 9 | arvutab alamvõrgu aadressi, leviedastusaadressi ja kasutatavate hostide arvu | rakendamine | 10–11 |
| 10 | selgitab DHCP ja DNS tööpõhimõtteid | mõistmine | 13 |
| 11 | diagnoosib levinumaid võrguprobleeme süstemaatilise tõrkeotsingu abil | analüüs | 14 |
| 12 | seadistab Cisco kommutaatoreid ja ruutereid käsurealt (CLI) | rakendamine | P1–P11 |

*Tabel 2. Õpiväljundid*

## Üldpädevused

| Pädevus | Seos |
|---|---|
| Digipädevus | Võrgutopoloogiate kavandamine, IP-adresseerimine, seadmete seadistamine |
| Õpipädevus | Iseseisev töö enesekontrolliküsimuste ja praktikumidega |
| Matemaatikapädevus | Kahendsüsteem, alamvõrgumaskide arvutamine |
| Suhtluspädevus | Erialane terminoloogia eesti ja inglise keeles |
| Ettevõtluspädevus | Võrguprojekti planeerimine, seadmete valik |

## Hindamine ja tagasiside

Iga teooriaosa lõpus on enesekontrolliküsimused kokkupandavate vastustega. Praktikumid sisaldavad kontrollitava tulemusega ülesandeid Cisco Packet Tracer simulaatoris. Interaktiivne H5P õpik koondab teooriaküsimused ühtseks testimiskeskkonnaks.

## Kuidas kasutada

- **Järjest** — alusta osast 0, tee praktikumid pärast vastavat teooriat
- **Teemapõhiselt** — mine otse vajaliku osa juurde (iga peatükk on iseseisev)
- **CCNA ettevalmistuseks** — kursus katab CCNA 200-301 võrgupõhimõtete osa
- **Õpetaja juhendamisel** — sobib kontaktõppe toetamiseks ja iseseisvaks tööks

## Pedagoogiline lähenemine

Materjal on loodud narratiivses stiilis — iga teema algab ajaloolise konteksti ja probleemiga, mida konkreetne tehnoloogia lahendab. Bullet-pointide asemel kasutatakse jutustust, analoogiaid ja Eesti konteksti (TLL-IX, merepõhjakaablid, MikroTik, e-Eesti). Iga peatüki lõpus on "Uuri ise" boks interaktiivsete linkide ja tööriistadega iseseisvaks avastamiseks.

Materjal on loodud vendor-neutraalsena — Cisco on esikohal kui CCNA ettevalmistuse osa, aga käsitletakse ka Juniper, Huawei, MikroTik, Ubiquiti ja teiste tootjate seadmeid.

## Tehnilised nõuded

Materjal töötab kõigis kaasaegsetes veebilehitsejates (Chrome, Firefox, Safari, Edge) ja on mobiilisõbralik. Praktikumide jaoks on vajalik [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (tasuta, nõuab Cisco NetAcad kontot).

## Tehisintellekti kasutamine

Kursuse hero-pildid (peatükkide peapildid) on loodud tehisintellekti abil ja märgistatud vastavalt. Tekstiline sisu on autori originaallooming, mida on arendatud koostöös AI-tööriistadega. Kõik faktid on kontrollitud ametlike allikate vastu (RFC-d, IEEE standardid, tootjate dokumentatsioon).
