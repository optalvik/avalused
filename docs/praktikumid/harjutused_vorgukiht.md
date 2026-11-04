---
tags:
  - Harjutused
  - Võrgukiht
---

# Tahvliharjutused: IPv4-aadress, võrk ja default gateway

Osa 1 kordab [praktikumi 7](praktikum_07.md) ja [peatüki 6: Võrgukiht](../teooria/06_vorgukiht.md) teemasid: ruuter ja default gateway. Osad 2–7 on uue teema [IPv4 adresseerimine](../teooria/08_ipv4.md) harjutused: IPv4-aadressi ülesehitus, võrk ja aadressid samas võrgus. Iga osa eel selgitab õpetaja teooria lühidalt. Osad 1–5 on **individuaalsed**: iga õppija lahendab tahvlil ühe ülesande, järgmine jätkab või parandab ja klass kontrollib. Osad 6–7 on **rühmatööd**. Vastused on kokkuvolditud plokkides. Ava need alles pärast seda, kui tahvlil on lahendus olemas.

!!! abstract "Eesmärgid"
    Harjutuste lõpuks:

    - Oskan öelda, kas IPv4-aadress on kehtiv ning kas see on privaatne või avalik
    - Oskan öelda, kas kaks IP-aadressi on samas võrgus
    - Oskan valida arvutile sobiva IP-aadressi ja default gateway
    - Oskan selgitada, kuidas pakett liigub ühest võrgust teise
    - Oskan leida topoloogiast vea, mille tõttu ping ei õnnestu

Kõigis harjutustes on mask `255.255.255.0`. See tähendab, et IP-aadressi **kolm esimest numbrit näitavad võrku** ja viimane number seadet selles võrgus. Näiteks aadressid `192.168.1.10` ja `192.168.1.50` on samas võrgus `192.168.1.0`. Teistsuguseid maske käsitleme [peatükis 10: Alamvõrgustamine](../teooria/10_alamvorgustamine.md).

## Osa 1: Kiire kordamine (individuaalne)

Vasta suuliselt. Iga küsimus on ühe õppija jaoks.

??? question "1. Mis vahe on kommutaatoril ja ruuteril?"
    Kommutaator ühendab seadmeid **ühes võrgus** ja kasutab MAC-aadresse. Ruuter ühendab **erinevaid võrke** ja kasutab IP-aadresse.

??? question "2. Mis on default gateway?"
    Ruuteri liidese IP-aadress arvuti enda võrgus. Sinna saadab arvuti kõik paketid, mille sihtkoht on teises võrgus.

??? question "3. Mis käsuga lülitad ruuteri liidese sisse?"
    `no shutdown`. Ruuteri liidesed on vaikimisi välja lülitatud.

??? question "4. Mis käsk näitab, kas ruuteri liidesed töötavad? Mida väljundis otsid?"
    `show ip interface brief`. Töötava liidese real on `up up`.

??? question "5. Mida tähendab `C` käsu `show ip route` väljundis?"
    *Connected*: võrk on otse ruuteri liidese taga. Need read tekivad ise, kui liidesel on IP-aadress ja liides on `up`.

## Osa 2: IPv4-aadress (individuaalne)

IPv4-aadressis on neli numbrit ehk **oktetti**, mis on eraldatud punktidega. Iga oktett on 8 bitti ja saab olla 0–255. Kokku on aadress 32 bitti ([peatükk 8](../teooria/08_ipv4.md)).

### Samm 1: Kas aadress on kehtiv?

| Nr | Aadress |
|---|---|
| a | 192.168.1.1 |
| b | 192.168.1.256 |
| c | 10.0.0 |
| d | 172.16.300.4 |
| e | 8.8.8.8 |
| f | 192.168.1.1.5 |

*Tabel H6.1. Kas IPv4-aadress on kehtiv?*

??? question "Vastused"
    a – kehtiv. b – ei, `256` on suurem kui 255. c – ei, oktette on kolm, peab olema neli. d – ei, `300` on suurem kui 255. e – kehtiv (Google'i DNS-server). f – ei, oktette on viis.

### Samm 2: Privaatne või avalik?

Privaatsed aadressid on vahemikes `10.x.x.x`, `172.16.x.x` kuni `172.31.x.x` ja `192.168.x.x`. Neid kasutatakse kodu-, kooli- ja kontorivõrkudes ning internetis neid ei marsruudita ([peatükk 8](../teooria/08_ipv4.md#avalikud-ja-privaatsed-ip-aadressid)).

| Nr | Aadress |
|---|---|
| a | 192.168.0.15 |
| b | 8.8.4.4 |
| c | 10.20.30.40 |
| d | 172.20.1.1 |
| e | 172.40.1.1 |
| f | 193.40.5.10 |

*Tabel H6.2. Privaatne või avalik aadress?*

??? question "Vastused"
    a – privaatne. b – avalik. c – privaatne. d – privaatne (`172.16`–`172.31`). e – avalik, sest `40` on vahemikust väljas. f – avalik.

    Lõks on `172` puhul: privaatne on ainult siis, kui teine oktett on 16–31.

### Samm 3: Mida see aadress tähendab?

Mida näitab arvutis olev aadress ja mida sa teed?

1. `127.0.0.1`
2. `169.254.12.7`
3. `255.255.255.255`

??? question "Vastused"
    1. Localhost: arvuti ise. `ping 127.0.0.1` kontrollib, kas arvuti võrgutarkvara töötab.
    2. DHCP server ei vastanud ja arvuti võttis endale ajutise aadressi. Internetti ei ole. Kontrolli kaablit või WiFi-ühendust ja ruuterit.
    3. Leviaadress: pakett läheb kõigile seadmetele samas võrgus. Arvutile seda aadressi anda ei saa.

## Osa 3: Arvutused (individuaalne)

Kirjuta vastus tahvlile ja selgita üks lause, kuidas selle said.

### Samm 1: Kas samas võrgus?

| Nr | Aadress A | Aadress B |
|---|---|---|
| a | 192.168.1.10 | 192.168.1.200 |
| b | 192.168.1.10 | 192.168.2.10 |
| c | 10.0.5.1 | 10.0.5.254 |
| d | 172.16.10.5 | 172.16.11.5 |
| e | 192.168.100.1 | 192.168.10.1 |

*Tabel H6.3. Kas aadressid on samas võrgus? Mask on 255.255.255.0*

??? question "Vastused"
    a – jah (`192.168.1`), b – ei, c – jah (`10.0.5`), d – ei (`10` ja `11`), e – ei (`100` ja `10`).

    Viimane on lõks: numbrid näevad sarnased välja, aga kolmas number on erinev.

### Samm 2: Võrgu aadress ja leviaadress

Kirjuta iga aadressi kohta võrgu aadress (viimane number `0`) ja leviaadress (viimane number `255`).

1. `192.168.5.37`
2. `10.1.1.200`
3. `172.20.3.1`

??? question "Vastused"
    | Aadress | Võrgu aadress | Leviaadress |
    |---|---|---|
    | 192.168.5.37 | 192.168.5.0 | 192.168.5.255 |
    | 10.1.1.200 | 10.1.1.0 | 10.1.1.255 |
    | 172.20.3.1 | 172.20.3.0 | 172.20.3.255 |

    Neid kahte aadressi ei saa arvutile anda. Arvutitele jäävad aadressid `.1` kuni `.254`.

### Samm 3: Kas sobib arvutile?

Võrk on `192.168.1.0`, ruuteri liides on `192.168.1.1`. Kas need aadressid sobivad arvutile?

| Nr | Aadress | Sobib? |
|---|---|---|
| a | 192.168.1.0 | |
| b | 192.168.1.1 | |
| c | 192.168.1.25 | |
| d | 192.168.1.255 | |
| e | 192.168.2.25 | |
| f | 192.168.1.256 | |

*Tabel H6.4. Kas aadress sobib arvutile?*

??? question "Vastused"
    a – ei, see on võrgu aadress. b – ei, see on ruuteri aadress, tekiks konflikt. c – jah. d – ei, see on leviaadress. e – ei, see on teine võrk. f – ei, ühes numbris saab olla kõige rohkem `255`.

### Samm 4: Mitu seadet mahub?

Mitu arvutit saab panna võrku `192.168.1.0`, kui mask on `255.255.255.0` ja ruuter kasutab ühte aadressi?

??? question "Vastus"
    Viimane number saab olla 0–255, kokku 256 aadressi. Võrgu aadress ja leviaadress lähevad maha: jääb 254. Ruuter kasutab ühe, arvutitele jääb **253**.

## Osa 4: Topoloogia selgitamine (individuaalne)

Joonista see topoloogia tahvlile. Sama topoloogia oli praktikumis 7.

```text
 PC1 ─── SW1 ─── Fa0/0 [ R1 ] Fa0/1 ─── SW2 ─── PC2
```

### Samm 1: Täida aadresside tabel

Üks õppija täidab ühe rea. Kasuta võrke `192.168.1.0` (vasak pool) ja `192.168.2.0` (parem pool). Ruuter saab igas võrgus aadressi `.1`.

| Seade | Liides | IP-aadress | Mask | Default gateway |
|---|---|---|---|---|
| R1 | Fa0/0 | | | — |
| R1 | Fa0/1 | | | — |
| PC1 | FastEthernet0 | | | |
| PC2 | FastEthernet0 | | | |

*Tabel H6.5. Täidetav aadresside tabel*

??? question "Vastus"
    | Seade | Liides | IP-aadress | Mask | Default gateway |
    |---|---|---|---|---|
    | R1 | Fa0/0 | 192.168.1.1 | 255.255.255.0 | — |
    | R1 | Fa0/1 | 192.168.2.1 | 255.255.255.0 | — |
    | PC1 | FastEthernet0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
    | PC2 | FastEthernet0 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |

    Arvuti aadress võib olla ka muu vaba aadress samas võrgus. Gateway peab olema ruuteri liides **sama arvuti võrgus**.

### Samm 2: Kirjuta ruuteri käsud

Kirjuta tahvlile käsud, millega R1 liides `Fa0/0` saab aadressi ja lülitatakse sisse.

??? question "Vastus"
    ```text
    R1> enable
    R1# configure terminal
    R1(config)# interface FastEthernet0/0
    R1(config-if)# ip address 192.168.1.1 255.255.255.0
    R1(config-if)# no shutdown
    R1(config-if)# exit
    ```

### Samm 3: Selgita paketi teekonda

PC1 pingib PC2-t. Üks õppija räägib ühe sammu, järgmine jätkab. Mis toimub igas sammus?

??? question "Vastus"
    1. PC1 võrdleb aadresse: `192.168.2.10` ei ole tema võrgus `192.168.1.0`.
    2. PC1 saadab paketi oma default gateway'le `192.168.1.1`. Selleks küsib ta ARP-iga ruuteri MAC-aadressi.
    3. SW1 viib kaadri MAC-aadressi järgi ruuteri liidesesse `Fa0/0`.
    4. R1 vaatab sihtaadressi ja marsruutimistabelit: võrk `192.168.2.0` on liidese `Fa0/1` taga (`C`-rida).
    5. R1 saadab paketi välja liidesest `Fa0/1`. SW2 viib selle PC2-ni.
    6. PC2 vastab ja sama tee läbitakse tagasi. Vastus läheb PC2 gateway'le `192.168.2.1`.

    IP-aadressid paketis ei muutu kogu tee jooksul. MAC-aadressid vahetuvad ruuteri juures.

### Samm 4: Ennusta

Mis juhtub, kui PC1-l on IP-aadress ja mask õiged, aga default gateway puudub? Kas PC1 saab pingida ruuterit `192.168.1.1`? Kas PC1 saab pingida PC2-t?

??? question "Vastus"
    Ruuteri `192.168.1.1` pingimine õnnestub, sest see on PC1-ga samas võrgus. PC2 pingimine ei õnnestu, sest PC1 ei tea, kuhu teise võrku minevad paketid saata. Seda proovisite praktikumis 7 osas „Lõhu meelega: gateway puudub“.

## Osa 5: Leia viga (individuaalne)

Igas olukorras ei õnnestu ping PC1-st PC2-le. Topoloogia on sama mis osas 4. Õppija leiab vea, ütleb põhjuse ja kirjutab parandatud väärtuse tahvlile.

| Nr | Mis on seadistatud |
|---|---|
| 1 | PC1: IP `192.168.1.10`, gateway `192.168.2.1` |
| 2 | PC2: IP `192.168.1.20`, gateway `192.168.2.1` |
| 3 | R1 `show ip interface brief`: `Fa0/1 192.168.2.1 administratively down down` |
| 4 | PC1: IP `192.168.1.10`, gateway `192.168.1.1`; R1 Fa0/0: `192.168.1.10` |
| 5 | Kõik aadressid on õiged, aga R1 ja SW2 vahel ei ole kaablit |

*Tabel H6.6. Leia viga*

??? question "Vastused"
    1. PC1 gateway on teise võrgu aadress. Õige on `192.168.1.1`, ruuteri liides PC1 enda võrgus.
    2. PC2 aadress on vales võrgus. Õige on näiteks `192.168.2.10`.
    3. Liides on välja lülitatud. Parandus: `interface Fa0/1` ja `no shutdown`.
    4. PC1 ja ruuteri liides kasutavad sama aadressi, tekib aadressikonflikt. Ruuteri liides peab olema `192.168.1.1`.
    5. Liides `Fa0/1` ei lähe olekusse `up`, kui kaabli teises otsas pole töötavat seadet. Marsruutimistabelis puudub siis ka rida `C 192.168.2.0`.

## Osa 6: Rühmatöö: lahenda ja esitle

Jagunege 3–4-liikmelistesse rühmadesse. Iga rühm saab ühe ülesande, joonistab lahenduse tahvlile ja selgitab selle teistele. Igal rühmal on 10 minutit ettevalmistuseks ja 3 minutit esitluseks.

!!! example "Näidisstsenaarium"
    **Rühm 1.** Väikeses kontoris on kaks ruumi: raamatupidamine (4 arvutit) ja klienditeenindus (3 arvutit). Ruumid peavad olema eri võrkudes, aga suhtlema omavahel. Joonista topoloogia ja koosta aadresside tabel.

    **Rühm 2.** Koolis on kolm klassi ja igal klassil oma võrk. Kõik klassid on ühendatud ühe ruuteriga, millel on kolm liidest. Joonista topoloogia, vali võrgud ja kirjuta iga klassi ühe arvuti aadress ja gateway.

    **Rühm 3.** Arvuti `192.168.10.15` ei pääse internetti, aga printer `192.168.10.20` vastab pingile. Ruuteri aadress on `192.168.10.1`. Koosta loetelu: mida ja mis järjekorras kontrollid ning mis käsuga.

    **Rühm 4.** Selgita teistele Haapsalu ja Tallinna näite abil, mis on default gateway ([peatükk 6](../teooria/06_vorgukiht.md#default-gateway-uks-valismaailma)). Joonista tahvlile pilt, kus on näha arvuti, gateway ja teine võrk.

    **Rühm 5.** Kodus on ruuter, telefon, sülearvuti ja nutiteler. Kirjuta igale seadmele privaatne IPv4-aadress ja selgita, kuidas need seadmed ühe avaliku aadressiga internetti pääsevad ([NAT](../teooria/08_ipv4.md#nat-kuidas-privaatne-vork-internetti-paaseb)).

    **Rühm 6.** IPv4-aadresse on umbes 4,3 miljardit, aga internetti ühendatud seadmeid on rohkem. Selgita, miks see ikkagi töötab, ja too kolm põhjust ([peatükk 8](../teooria/08_ipv4.md)).

??? question "Mida rühmatöö lahendusest otsida"
    - Igas võrgus on oma võrgu aadress, näiteks `192.168.1.0`, `192.168.2.0` ja `192.168.3.0`.
    - Ruuteril on igas võrgus üks liides ja selle aadress on selle võrgu gateway.
    - Arvutite aadressid on gateway'ga samas võrgus.
    - Rühm 3 alustab arvutist: IP-aadress ja gateway (`ipconfig`), siis ping gateway'le, siis ping väljapoole.
    - Rühm 5: kõik kodu seadmed on samas privaatvõrgus (nt `192.168.1.x`), ruuter on gateway ja NAT asendab väljuva paketi aadressi ruuteri avaliku aadressiga.
    - Rühm 6: privaatsed aadressid ja NAT, IPv6 ning see, et vabad aadressid ostetakse ja müüakse.

## Osa 7: Rühmatöö: loe ja selgita

Iga rühm loeb 10 minutit ühte osa [peatükist 6: Võrgukiht](../teooria/06_vorgukiht.md) või [peatükist 8: IPv4 adresseerimine](../teooria/08_ipv4.md). Seejärel selgitab rühm selle tahvlil teistele 3 minutiga. Selgituses peab olema üks joonis ja üks näide.

| Rühm | Mida loete |
|---|---|
| 1 | [Miks kommutaatorist ei piisa](../teooria/06_vorgukiht.md#miks-kommutaatorist-ei-piisa) |
| 2 | [Ruuter — võrkude vaheline teejuht](../teooria/06_vorgukiht.md#ruuter-vorkude-vaheline-teejuht) |
| 3 | [Marsruutimistabel](../teooria/06_vorgukiht.md#marsruutimistabel-ruuteri-kaart) |
| 4 | [Paketi teekond — kogu pilt](../teooria/06_vorgukiht.md#paketi-teekond-kogu-pilt) |
| 5 | [Võrgu- ja hostiosa](../teooria/08_ipv4.md#vorgu-ja-hostiosa) |
| 6 | [Avalikud ja privaatsed IP-aadressid](../teooria/08_ipv4.md#avalikud-ja-privaatsed-ip-aadressid) |
| 7 | [Erilise tähendusega aadressid](../teooria/08_ipv4.md#erilist-tahendusega-aadressid) |

*Tabel H6.7. Lugemise jaotus rühmadele*

Kuulajad esitavad igale rühmale ühe küsimuse.

---

## Kokkuvõte

Nendes harjutustes sa:

- kontrollisid, kas IPv4-aadress on kehtiv ning kas see on privaatne või avalik;
- kontrollisid, kas kaks aadressi on samas võrgus;
- leidsid võrgu aadressi, leviaadressi ja arvutile sobivad aadressid;
- täitsid aadresside tabeli ja kirjutasid ruuteri liidese käsud;
- selgitasid paketi teekonda läbi ruuteri;
- leidsid vead, mille tõttu ping ei õnnestu.

Järgmisena teed [praktikumi 8](praktikum_08.md), kus võrkude vahel on kaks ruuterit.
