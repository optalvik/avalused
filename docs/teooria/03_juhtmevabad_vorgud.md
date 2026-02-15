---
tags:
  - Ethernet
  - Võrgud
---

# Juhtmevabad võrgud

!!! abstract "Eesmärgid"
    Selle peatüki läbimise järel:

    - Oskan selgitada WiFi tööpõhimõtet ja eristada sagedusalasid (2.4 GHz, 5 GHz, 6 GHz)
    - Oskan võrrelda WiFi põlvkondi (WiFi 4–7)
    - Oskan selgitada WiFi turvaprotokolle (WEP, WPA, WPA2, WPA3)
    - Oskan eristada mobiilsidevõrkude põlvkondi (2G–5G)
    - Oskan kirjeldada Bluetooth, NFC ja LoRaWAN kasutusalasid
    - Oskan nimetada levinumaid juhtmevabasid turvaohte
    - Oskan selgitada, kuidas raadiosagedusi Eestis reguleeritakse

## Kõik algas Hawaiilt

Eelmises peatükis rääkisime, et füüsiline kiht kannab bitte kolmel viisil: elekter vaskkaablis, valgus fiibris ja raadiolained õhus. Esimesed kaks käsitlesime põhjalikult — nüüd on raadiolainete kord.

Aga enne kui WiFi ruuteriteni jõuame, peame minema tagasi 1970. aastasse Hawaii saartele. Professor **Norman Abramson** Hawaii Ülikoolist seisis silmitsi probleemiga: kuidas ühendada arvuteid neljal erineval saarel, ilma et peaks merepõhjakaablit vedama? Tema lahendus oli **ALOHAnet** — maailma esimene juhtmevaba andmesidevõrk. Seitse arvutit neljal saarel suhtlesid raadiolainete kaudu keskse arvutiga Oahu saarel.[^alohanet]

ALOHAnet'i tugevus oli lihtsuses: kui sul on andmeid saata, saada need lihtsalt eetrisse. Kui keegi teine saadab samal ajal, tekib kollisioon — oota juhuslik aeg ja proovi uuesti. Tundub kaootilisena? Oli ka. Süsteemi efektiivsus oli ainult 18%. Aga see lihtne idee — "saada ja oota kinnitust" — muutis maailma. Sellest kasvas välja Ethernet (kaablite peatükist tuttav) ja hiljem WiFi. Iga kord, kui su telefon WiFi-ga ühendub, kasutab ta ALOHAnet'i järeltulijat.

## Kuidas raadiosagedusi jagatakse

Enne WiFi ja mobiilside juurde minekut peame rääkima ühest olulisest asjast: raadiosagedused on **piiratud ressurss**. Sama sagedust ei saa kasutada korraga kaks erinevat saatjat — muidu tekib müra. Sellepärast reguleerib riik, kes tohib millisel sagedusel saata.

Eestis jagab sagedusi **TTJA** (*Tarbijakaitse ja Tehnilise Järelevalve Amet*). Eesti raadiosagedusplaan on kooskõlas ITU (Rahvusvaheline Telekommunikatsiooni Liit), Euroopa Liidu ja NATO nõuetega — Eesti ei saa lihtsalt ise otsustada, milliseid sagedusi kuidas kasutada.[^ttja]

Sagedusi on kahte tüüpi: **loaga** ja **loata**. Mobiilsideoperaatorid (Telia, Elisa, Tele2) ostavad sageduslube riiklikel oksjonitel — 5G sagedusload 3.5 GHz alas läksid Eestis miljonite eurode eest. WiFi ja Bluetooth aga kasutavad **loata sagedusalasid** (2.4 GHz, 5 GHz) — igaüks tohib seal saata, aga väikese võimsusega. Sellepärast su koduruuter ei sega naabri oma — noh, enamasti.

!!! info "Miks mikrolaineahi segab WiFi-d?"
    2.4 GHz sagedusala on ISM-riba (*Industrial, Scientific, Medical*) — algselt mõeldud tööstusseadmetele, teadusele ja meditsiinile. Mikrolaineahi kasutab täpselt sama sagedust toidu soojendamiseks. Kui 1985. aastal USA telekommunikatsiooni regulaator (FCC) lubas selle riba ka andmesideks kasutada, sai WiFi võimalikuks — aga nii sai ka "su kohvilkäik segab su videokõne" probleem.

## WiFi — kuidas see töötab?

WiFi on ametlikult IEEE 802.11 standardiperekond.[^ieee80211] Esimene versioon ilmus 1997. aastal ja pakkus toredaid 2 Mbit/s. Suure läbimurde tegi WiFi 1999. aastal, kui Apple lisas oma iBook sülearvutile WiFi toe ja nimetas selle **AirPort**. See oli esimene massiline tarbetoode, mis WiFi populaarseks tegi.

Põhimõte on lihtne: pääsupunkt (*access point*, AP) saadab ja võtab vastu raadiolaineid, ja kõik seadmed suhtlevad AP kaudu. Aga raadiolained on jagatud keskkond — kõik "räägivad" samas õhus. Mäletad ALOHAnet'i? WiFi kasutab selle täiustatud versiooni: **CSMA/CA** (*Carrier Sense Multiple Access with Collision Avoidance*). Enne saatmist kuulab seade, kas kanal on vaba. Kui on, saadab. Kui ei ole, ootab juhusliku aja ja proovib uuesti. See on üks põhjus, miks WiFi on alati aeglasem kui kaabel — pool ajast kulub "viisakale ootamisele".

### WiFi põlvkonnad

| Põlvkond | Standard | Aasta | Max kiirus | Sagedus |
|---|---|---|---|---|
| WiFi 4 | 802.11n | 2009 | 600 Mbps | 2.4 / 5 GHz |
| WiFi 5 | 802.11ac | 2014 | 3.5 Gbps | 5 GHz |
| WiFi 6 | 802.11ax | 2019 | 9.6 Gbps | 2.4 / 5 GHz |
| WiFi 6E | 802.11ax | 2021 | 9.6 Gbps | 2.4 / 5 / 6 GHz |
| WiFi 7 | 802.11be | 2024+ | 46 Gbps | 2.4 / 5 / 6 GHz |

*Tabel 3.1. WiFi põlvkondade võrdlus*

!!! info "Tabelikiirused vs reaalsus"
    Tabelis olevad maksimaalkiirused on teoreetilised ideaaltingimustes. Päriselus saad WiFi 6-ga heal juhul 500–800 Mbps ja WiFi 5-ga 200–400 Mbps. Aga ka see on kordades parem kui WiFi 4. WiFi Alliance andis varem igavad 802.11-numbrid sõbralikumaks ümber: 802.11n = WiFi 4, 802.11ac = WiFi 5 jne.

### Sagedusalad — füüsikareegel, mida sa pead teadma

WiFi sagedusalade puhul kehtib lihtne füüsikareegel: **mida madalam sagedus, seda kaugemale ulatub ja seda paremini läbib takistusi**. Mida kõrgem sagedus, seda kiirem ühendus, aga seda lühem ulatus. See reegel kehtib kõigi raadiolainete puhul — WiFist 5G-ni.

**2.4 GHz** on vana ja tuttav — ulatub kaugele ja läbib seinad paremini. Aga seda kasutavad peale WiFi ka Bluetooth, mikrolaineahjud ja beebifoonid. 13 kanalit, kuid mittekattuvaid ainult 3 (kanalid 1, 6, 11). Kortermajas, kus igal naabritel on oma ruuter, on 2.4 GHz nagu rahvarohke tänav — kõik üritavad korraga rääkida.

**5 GHz** on kiirem ja puhtam — rohkem kanaleid, vähem konflikte. Aga seinad nõrgestavad signaali tugevamini. Kui su ruuter on ühes toa otsas ja sülearvuti teises, võib 5 GHz signaal juba nõrk olla seal, kus 2.4 GHz veel kenasti töötab.

**6 GHz** on uusim, avatud WiFi 6E ja WiFi 7 jaoks. Tohutu hulk uut spektrit ja väga laiad kanalid tähendavad pöörast kiirust. Ulatus on aga kõige lühem — sobib kõrge tihedusega keskkondadesse nagu kontorid ja staadionid.

!!! tip "Praktiline reegel"
    2.4 GHz IoT seadmetele ja kaugematesse tubadesse. 5 GHz striimimiseks ja igapäevaseks tööks. 6 GHz ainult siis, kui seadmed ja ruuter seda toetavad ja oled ruuteri lähedal.

### Miks WiFi vannitoas ei tööta?

Mitte kõik takistused pole võrdsed. Kipsplaatsein — väike probleem. Betoonisein — keskmiselt probleem. Vannitoaplaadid ja metallist torud — suur probleem, sest metall peegeldab raadiolaineid tagasi. Akvaarium ruuteri kõrval? Vesi on raadiolainete jaoks peaaegu läbipaistmatu. Ja töötav mikrolaineahi? See kasutab sama 2.4 GHz sagedust ja on sisuliselt WiFi segaja, mis su köögis istub.

## WiFi turvalisus

Raadiolaineid saab pealt kuulata igaüks, kellel on vastuvõtja. Kaablivõrgus peab häkker füüsiliselt kaabli külge pääsema. WiFi puhul piisab, kui istud autoparklas ruuteri leviala piires. Sellepärast on WiFi turvaprotokollid kriitilise tähtsusega.

| Protokoll | Aasta | Turvalisus | Märkus |
|---|---|---|---|
| Avatud | — | Puudub | Igaüks näeb liiklust |
| WEP | 1999 | Murtav minutitega | Ära kasuta! |
| WPA | 2003 | Ajutine parandus | Aegunud |
| WPA2 | 2004 | Tugev (AES) | Praegune miinimum |
| WPA3 | 2018 | Väga tugev | Parim valik |

*Tabel 3.2. WiFi turvaprotokollide evolutsioon*

WEP oli WiFi esimene turvaprotokoll ja ta osutus katastroofiliselt nõrgaks — murtav minutitega vabalt kättesaadava tarkvaraga. WPA oli kiirparandus, mis asendati peagi WPA2-ga. WPA3 on praegune tipp: isegi kui keegi homme su parooli ära arvab, ei saa ta dekrüpteerida täna pealtkuulatud liiklust. Seda nimetatakse *forward secrecy* põhimõtteks.[^wpa3]

!!! warning "Avalik WiFi"
    Kohviku, lennujaama ja hotelli WiFi on häkkerite lemmik jahimaa. Avatud WiFi-võrgus näeb igaüks su liiklust. Kasuta alati VPN-i või veendu, et külastad ainult HTTPS-lehti.

## Mobiilsidevõrgud

Mobiilside ajalugu on ühe suure hüppe lugu iga kümne aasta tagant. Esimesed mobiiltelefonid (1G, 1980ndad) olid puhtalt analoogsed — hääl ja piiks. 2G tõi digitaalse hääle ja SMS-i. 3G tegi mobiilse interneti võimalikuks. 4G tegi selle kiireks. Ja 5G? See muudab kõike uuesti.

| Põlvkond | Aasta | Kiirus | Peamine muutus |
|---|---|---|---|
| 2G (GSM) | 1991 | 14.4 kbps | Digitaalne hääl + SMS |
| 3G (UMTS) | 2001 | 2 Mbps | Mobiilne internet |
| 4G (LTE) | 2010 | 100+ Mbps | HD video, äpid |
| 5G (NR) | 2020 | 1+ Gbps | IoT, madal latentsus |

*Tabel 3.3. Mobiilsidevõrkude põlvkonnad*

5G tõi kolm olulist muutust. Esiteks **kiirus** — kuni 10 Gbps teoorias, praktikas 200–500 Mbps. Teiseks **madal latentsus** — alla 10 ms, mis teeb võimalikuks reaalajarakendused nagu isejuhtivad autod. Kolmandaks **mahutavus** — kuni miljon seadet ruutkilomeetri kohta.

### 5G Eestis

Eestis pakuvad 5G teenust kolm operaatorit: **Telia**, **Elisa** ja **Tele2**. Sagedusload jagas TTJA riiklikel oksjonitel, kus operaatorid ostsid 700 MHz, 2.3 GHz, 3.5 GHz ja 26 GHz sagedusalade kasutusõiguse.

2024. aasta lõpuks kattis Tele2 5G võrk umbes 70% Eesti elanikkonnast. Elisa saavutas Ookla testides Eesti kiireima 5G-võrgu tiitli mediaankiirusega 336 Mbit/s. Elisa eesmärk on 2026. aastaks jõuda 5G katvusega 99% Eesti rahvastikust. Samal ajal suletakse vanad võrgud: Tele2 sulges 2025. aastal oma 3G võrgu.

Põnev projekt on ka **5G-BALTICS** — Euroopa Liidu poolt osaliselt rahastatav 16.5 miljoni euro suurune projekt, mis ehitab 5G katvuse kogu Via Baltica kiirtee äärde Tallinnast Poola piirini (663 km). Projektis osalevad kõik kolm Eesti operaatorit ja see peaks valmima 2027. aastaks.

!!! info "Miks 5G vajab nii palju maste?"
    Kõrgem sagedus = lühem levikaugus. 4G mast 800 MHz sagedusel katab terve linnaosa. 5G mast 3.5 GHz sagedusel katab mõned kvartalid. 5G mmWave (26 GHz) katab ühe tänava. Sellepärast näed uusi väikseid antenne ilmumas laternapostide ja hoonete külge — ja sellepärast on 5G ehitamine nii kallis.

## Bluetooth

WiFi ühendab sind internetiga. Bluetooth ühendab su seadmed omavahel — kõrvaklapid telefoniga, hiir sülearvutiga, aktiivsusmonitor äpiga. See on PAN (*Personal Area Network*), millest rääkisime võrgutüüpide peatükis.

Bluetooth sai nime Taani kuningas **Harald Sinihammas** (*Harald Bluetooth*) järgi, kes 10. sajandil ühendas Skandinaavia hõimud. Bluetooth'i logo on tema initsiaalide ruunikiri. Sobiv nimi tehnoloogiale, mis ühendab seadmeid.

| Versioon | Aasta | Ulatus | Peamine uuendus |
|---|---|---|---|
| Bluetooth 4.0 | 2010 | ~50 m | Low Energy (LE) — akud kestavad nädalaid |
| Bluetooth 5.0 | 2016 | ~200 m | 4× ulatus, 2× kiirus |
| Bluetooth 5.3 | 2021 | ~200 m | Parem energiasääst, stabiilsus |

*Tabel 3.4. Bluetooth versioonide võrdlus*

Oluline eristus: **Bluetooth Classic** on mõeldud pidevaks andmevooks nagu muusika striimimine — tarbib rohkem energiat. **Bluetooth Low Energy (LE)** on mõeldud harvadeks, väikesteks andmeedastusteks — aktiivsusmonitorid, andurid, uksesensorid. Töötab kuude kaupa ühe patareiga.

## NFC

Near Field Communication töötab ainult puudutades — kuni 4 sentimeetri kaugusel. See on meelega nii lühikese ulatusega: turvalisuse huvides. Keegi ei saa su makseinfot üle toa varastada.

Kasutusalad: kontaktivaba maksmine (telefon terminali vastu), hotellitoakaardid, ühistranspordi kaardid (Tallinna Ühiskaart!), digitaalsed visiitkaardid. Kui puudutad telefoni terminaliga ja piiks käib — see on NFC.

## LoRaWAN

LoRaWAN on juhtmevabasid võrkude "maratonijooksja" — aeglane (alla 50 kbps), kuid ülipika ulatusega (kuni 15 km linnades, kuni 40+ km maapiirkondades) ja aastatepikkuse aku kestvusega. Üks lüüs (*gateway*) katab terve linna.

Kus seda Eestis kasutatakse? Parkimisandurid (kas koht on vaba?), prügikastide täituvuse jälgimine, farmi mullasensorid (niiskus, temperatuur), veelekete tuvastamine veetorudes. Kõik need vajavad edastada vaid väikeseid andmepakette pika vahemaa tagant — ja akut vahetada iga paari aasta tagant, mitte iga nädal.

## Juhtmevabad turvaohud

Raadiolaineid saab pealt kuulata — see on juhtmevabade võrkude suurim nõrkus võrreldes kaabliga. Siin on ründed, mida peaksid teadma:

| Rünne | Kirjeldus | Kaitse |
|---|---|---|
| Evil Twin | Häkker loob võlts-WiFi sama nimega | VPN avalikes kohtades |
| Deauth Attack | Viskab sind WiFi-st välja võltsitud pakettidega | WPA3 (Protected Management Frames) |
| KRACK | WPA2 krüpteeringu nõrkus (2017) | Uuenda firmware, kasuta WPA3 |
| Wardriving | WiFi-võrkude kaardistamine autoga sõites | Kasuta WPA3, ära jäta võrku avatuks |
| BlueBorne | Bluetooth-i kaudu seadme kaaperdamine | Hoia Bluetooth välja lülitatud, kui ei kasuta |

*Tabel 3.5. Levinumad juhtmevabad turvaohud*

!!! info "Uuri ise"
    - [inSSIDer: WiFi analüüsitööriist](https://www.metageek.com/products/inssider/) — tasuta tööriist, mis näitab su ümbruses WiFi kanaleid reaalajas
    - [How does 5G work?](https://www.youtube.com/watch?v=GEx_d0SjnIE) — selge selgitus 5G kolmest kihist
    - [KRACK rünne demo](https://www.krackattacks.com/) — 2017. aasta WPA2 haavatavuse leht originaaluurijatelt
    - [5G Eestis](https://5geestis.ee/) — TTJA info 5G kohta Eestis

---

## Kokkuvõte

Juhtmevabad tehnoloogiad said alguse 1970. aasta Hawaii saarelt, kus ALOHAnet näitas, et andmeid saab saata raadiolainete teel. Sellest kasvas WiFi (IEEE 802.11), mis on tänapäeval peamine siseruumide internetiühendus. Raadiosagedused on piiratud ressurss, mida Eestis reguleerib TTJA kooskõlas ITU, EL ja NATO nõuetega. Mobiilsidevõrgud (2G → 5G) arenevad iga kümne aasta tagant — Eestis pakuvad 5G teenust Telia, Elisa ja Tele2, katvusega juba üle 70% elanikest. Bluetooth ühendab seadmeid omavahel, NFC võimaldab kontaktivaba maksmist ja LoRaWAN teenindab IoT andureid. Turvalisus on juhtmevabastes võrkudes kriitiliselt oluline — kasuta alati WPA3 ja VPN-i avalikes võrkudes.

---

## Enesekontroll

??? question "1. Mis on ALOHAnet ja miks ta oluline on?"
    ALOHAnet oli maailma esimene juhtmevaba andmesidevõrk, loodud 1970. aastal Hawaii Ülikoolis. Selle "saada ja oota kinnitust" põhimõte sai aluseks nii Ethernetile kui WiFi-le.

??? question "2. Kes reguleerib raadiosagedusi Eestis ja miks?"
    TTJA (Tarbijakaitse ja Tehnilise Järelevalve Amet) kooskõlas ITU, EL ja NATO nõuetega. Raadiosagedused on piiratud ressurss — sama sagedust ei saa kasutada korraga kaks saatjat. Mobiilsideoperaatorid ostavad sageduslube oksjonitel, WiFi kasutab loata sagedusalasid.

??? question "3. Mis vahe on 2.4 GHz ja 5 GHz sagedusalal?"
    2.4 GHz läbib seinad paremini ja ulatub kaugemale, kuid on aeglasem ja ülekoormatud. 5 GHz pakub kiiremat ühendust ja vähem häireid, kuid lühemat levikaugust.

??? question "4. Miks on WEP ohtlik ja mida kasutada selle asemel?"
    WEP on murtav minutitega vabalt kättesaadava tarkvaraga. Minimaalselt tuleks kasutada WPA2-d (AES), ideaalis WPA3-e, mis pakub ka forward secrecy kaitset.

??? question "5. Nimeta 5G kolm peamist eelist 4G ees."
    Suurem kiirus (kuni 10 Gbps vs 100 Mbps), madalam latentsus (alla 10 ms vs 30–50 ms) ja suurem mahutavus (miljon seadet/km² vs kümned tuhanded).

??? question "6. Miks on WiFi alati aeglasem kui kaabel?"
    WiFi kasutab jagatud keskkonda (õhk) ja CSMA/CA protokolli — seadmed peavad "ootama oma korda" enne saatmist. Kaabelvõrgus (full-duplex kommutaator) saab iga seade saata ja vastu võtta samaaegselt, ilma ootamiseta.

??? question "7. Mis on Evil Twin rünne?"
    Häkker loob võlts-WiFi pääsupunkti sama nimega (SSID) kui legitiimne võrk. Kasutaja ühendub teadmatult valega ja häkker näeb kogu liiklust. Kaitse: VPN avalikes kohtades.

[^alohanet]: Abramson, N. (1970). *The ALOHA System*. University of Hawaii. https://en.wikipedia.org/wiki/ALOHAnet

[^ieee80211]: IEEE 802.11 Working Group. (2021). *IEEE 802.11ax-2021: Wireless LAN Medium Access Control*. IEEE. https://standards.ieee.org/ieee/802.11ax/7354/

[^wpa3]: Wi-Fi Alliance. (2018). *Wi-Fi CERTIFIED WPA3: Security*. https://www.wi-fi.org/discover-wi-fi/security

[^ttja]: Eesti raadiosagedusplaan. Riigi Teataja. https://www.riigiteataja.ee/akt/101112017001
