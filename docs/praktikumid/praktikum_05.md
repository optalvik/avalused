---
tags:
  - Praktikum
---

# Praktikum 5: Ethernet-kaabli valmistamine

Selles praktikumis valmistad oma esimese Ethernet-kaabli. See on füüsiline labor, kus õpid Cat5e kaabli ettevalmistamist, kiudude järjendamist T-568B standardi järgi, RJ-45 otsiku paigaldamist ja kaabli testimist.

Kaablid on [füüsilise kihi](../teooria/02_fuusiline_kiht.md) põhikomponent — ilma nendeta pole võrku. Erinevaid kaablitüüpe (vask, optika) ja nende omadusi käsitleb teooria. Selles laboris töötad konkreetselt UTP (*Unshielded Twisted Pair*) kaabliga, mis on enim levinud kohtvõrgustiku standard.

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan ette valmistada ja koorida Cat5e kaablit
    - Oskan järjendada kiude T-568B standardi järgi
    - Oskan paigaldada ja krimpida RJ-45 otsikut
    - Oskan kasutada kaablitesterit valmis kaabli kontrollimiseks

!!! warning "Eeldused"
    Enne seda praktikumi pead olema tutvunud [Peatükk 2: Füüsiline kiht](../teooria/02_fuusiline_kiht.md) — kaablitüübid, keerdpaarid, kategooriad.

!!! info "Vajalikud materjalid"
    - Cat5e kaabel (~1 m)
    - RJ-45 otsikud (läbipaistvad, vähemalt 4 tk)
    - Kaablitangid (*crimping tool*)
    - Kaablikoorija või nuga
    - Kaablitester

## Osa 1: Kaabli ettevalmistamine

### Samm 1: Kaabli koorimine

Mõõda kaabli otsast ~3–4 cm ja kasuta kaablikoorijat väliskile eemaldamiseks. Lõika ringjooneliselt **ainult väliskile** — ära kahjusta sisemisi kiude.

!!! danger "Levinud viga"
    Liiga sügav lõige kahjustab vaskkiude ja kaabel ei tööta. Harjuta enne prügitüki peal.

### Samm 2: Keerdpaaride lahtiharutamine

Näed 4 keerdpaari (8 kiudu kokku). Haruata igaüks ettevaatlikult lahti ja siruta kiud täiesti sirgeks. Kasuta sõrmi või lauaserva kiudude sirgendamiseks — kõik kiud peavad olema ühel tasapinnal.

## Osa 2: Kiudude järjendamine (T-568B)

T-568B on üks kahest levinud Ethernet-kaablite värvistandardist (teine on T-568A). Eestis ja enamikus Euroopa riikides kasutatakse valdavalt **T-568B** standardit. Kui mõlemas otsas on sama standard, on tulemuseks **straight-through** kaabel (arvuti ↔ kommutaator). Kui otstes on erinevad standardid, saad **crossover** kaabli (arvuti ↔ arvuti).

Järjenda kiud vasakult paremale T-568B standardi järgi:

| PIN | Värvus |
|---|---|
| 1 | Valge/oranž |
| 2 | Oranž |
| 3 | Valge/roheline |
| 4 | Sinine |
| 5 | Valge/sinine |
| 6 | Roheline |
| 7 | Valge/pruun |
| 8 | Pruun |

*Tabel 5.1. T-568B kiudude järjekord*

!!! tip "Meeldetrikk"
    Oranž paar (1–2), roheline kiud (3), sinine paar (4–5), roheline kiud (6), pruun paar (7–8). Mõlemad otsad kasutavad **sama** standardit — see annab straight-through kaabli.

### Kiudude lõikamine

Hoia kõik 8 kiudu tihedalt koos õiges järjekorras ja lõika ristlõikega. Jäta pikkuseks **12–13 mm** kooritud otsast.

## Osa 3: RJ-45 otsiku paigaldamine

### Samm 1: Otsiku orienteerimine

Hoia otsikut nii, et **metallkontaktid on üleval** ja plastist lukk (*clip*) all. Avatud ots on sinu poole.

### Samm 2: Kiudude sisestamine

Lükka kiud otsikusse kindlalt ja sirgjooneliselt. Kontrolli läbipaistva otsiku kaudu:

- Kõik 8 kiudu on näha otsiku esiosas
- Värvijärjekord on õige
- Väliskest jõuab ~5 mm otsiku sisse
- Kiud on täiesti otsiku lõpus

### Samm 3: Krimpimine

Aseta otsikuga kaabel kaablitangide krimpimispessa ja pigista **tugevalt** mõlema käega. Tunned vastupanu kadumist — krimpimine on valmis.

!!! warning "Kontrolli enne krimpimist!"
    Pärast krimpimist ei ole võimalik otsikut enam eemaldada. Kontrolli värvijärjekorda **enne** tangide kasutamist.

### Samm 4: Teine ots

Korda kõiki samme teise otsa jaoks. Kasuta **sama standardit** (T-568B).

## Osa 4: Kaabli testimine

### Kaablitesteri kasutamine

1. Ühenda üks kaabliots **TX** (Master) pessa
2. Ühenda teine ots **RX** (Remote) pessa
3. Lülita tester sisse
4. Vaata LED-tulesid

**Õige tulemus:** kõik 8 LED-tuld süttivad **järjestikku** (1→2→3→4→5→6→7→8) rohelist värvi.

### Vigade diagnostika

| LED käitumine | Probleem | Lahendus |
|---|---|---|
| LED ei sütti üldse | Üldine rike | Kontrolli otsikud, alusta uuesti |
| Üks LED puudub | Kiud ei kontakti | Lõika ots maha ja krimpi uuesti |
| LED-id vales järjekorras | Kiud vales järjekorras | Kontrolli värvijärjekorda |
| Töötab 100 Mbps, mitte 1 Gbps | PIN 4, 5, 7, 8 ei tööta | Kõik 8 kiudu peavad kontaktis olema ([füüsiline kiht](../teooria/02_fuusiline_kiht.md) nõuab seda 1 Gbps jaoks) |

*Tabel 5.2. Kaablitesteri vigade diagnostika*

## Osa 5: Kontrollnimekiri

Visuaalne kontroll enne esitamist:

- [ ] Kõik 8 kiudu nähtavad läbi läbipaistva otsiku
- [ ] Õige värvijärjekord (T-568B) mõlemas otsas
- [ ] Väliskest sisestatud otsikusse
- [ ] Ei ole paljastatud vaskkiudu enne otsikut
- [ ] Otsik on korralikult krimpitud (ei tule maha)
- [ ] Kaablitester näitab kõik 8 LED-tuld õiges järjekorras

---

## Kokkuvõte

Selles praktikumis sa:

- Valmistasid ette ja koorisid Cat5e kaabli
- Järjendasid kiud T-568B standardi järgi
- Paigaldasid ja krimppisid RJ-45 otsikud mõlemasse otsa
- Testisid valmis kaablit kaablitesteriga
