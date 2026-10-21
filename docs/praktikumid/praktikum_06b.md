---
tags:
  - Praktikum
  - Kommutaator
---

# Praktikum 6b: Andra Switch — päris kommutaator serveriruumis

Andra on uus IT-toe praktikant. Tema esimene ülesanne: serveriruumis on Cisco kommutaator, mille seadistust keegi ei tea. Andra peab ühendama oma arvuti kommutaatoriga, sisse logima ja seadistama selle nii, et kõik teaksid, mis seade see on, ja et suvaline inimene sinna sisse ei pääseks.

Seni tegid seda Packet Traceris. Täna oled sina Andra ja teed sama **päris kommutaatoriga serveriruumis**.

!!! abstract "Eesmärgid"
    Selle praktikumi läbimise järel:

    - Oskan ühendada arvuti päris kommutaatoriga konsoolikaabli ja PuTTY kaudu
    - Oskan leida Windowsis COM-pordi numbri
    - Oskan seadistada kommutaatorile nime ja paroolid ning neid kontrollida
    - Oskan kommutaatori seadistuse kustutada (järgmise grupi jaoks)

!!! warning "Enne tundi vaata"
    - [Video: PuTTY ühendus](https://drive.google.com/file/d/1y2phgdB0MZ0k9rTrmo_ES2ae5jxHEo1a/view?usp=sharing)
    - [Juhend: klassi arvuti, konsoolikaabel, PuTTY](https://docs.google.com/document/d/12NH_-liylELLgUhspfoTXxHba21JFe0_eW3Jok0Zffk/edit?usp=sharing)
    - [Praktikum 2: CLI ja kommutaatori seadistamine](praktikum_02.md) (samad käsud Packet Traceris)

!!! note "Grupitöö"
    Grupis on 2–3 inimest. Grupid töötavad korraga, igal grupil oma kommutaator. **Poole pealt vahetate rollid**, et igaüks trükiks ise käske.

    | Roll | Kus | Mida teeb |
    |---|---|---|
    | Serveriruum | serveriruumis, kapp K1 | ühendab kaabli patch-paneelilt kommutaatori CONSOLE porti, teeb foto `kaabel.jpg` |
    | Konsool | klassis arvuti taga | PuTTY, trükib käsud |
    | Kirjutaja (3. inimene) | klassis | teeb ekraanipildid `putty.png` ja `show-run.png`, märgib üles kommutaatori numbri |

    Esitab **igaüks** oma Classroomi repos ([Lab 06b - Andra Switch](https://classroom50.org/hkhk-avalused/it26-avalused/assignments)). Grupi pildid võivad olla samad, vastused kirjutab igaüks ise.

**Parool on kõikjal `1234`.**

## Kuidas ühendus käib

```text
Sinu arvuti ──(konsoolikaabel, USB → RJ-45)── laua pesa (nt 21.1)
      … seina sees …
Serveriruum, kapp K1: patch-paneeli pesa 21.1 ──(sinine kaabel)── kommutaatori CONSOLE port
```

*Joonis 6b.1. Konsooliühenduse tee klassist serveriruumi*

**Konsoolikaabel** (helesinine) ei ole võrgukaabel. Selle kaudu räägid kommutaatoriga otse, ka siis, kui tal pole veel IP-aadressi.

!!! info "Mis seade see on?"
    Kommutaator on **2. kihi** (andmesidekiht) seade: ta saadab kaadreid **MAC-aadressi** järgi õigesse porti. Seda nägid praktikumis 6. IP-aadressidega töötab **ruuter** (3. kiht, võrgukiht). Loe juurde: [OSI mudel](../teooria/01_protokollid_mudelid.md#osi-mudel), [Kommutaator](../teooria/04_andmesidekiht.md#kommutaator-tark-postkontor), [Ruuter](../teooria/06_vorgukiht.md#ruuter-vorkude-vaheline-teejuht).

## Samm 1: Ühenda kaabel

1. **Klassis:** konsoolikaabli USB-ots arvutisse, RJ-45 ots laua pessa. Kirjuta üles pesa number (nt 21.1).
2. **Serveriruumis:** leia kapist **K1** patch-paneelilt sama number ja ühenda sinine kaabel sealt kommutaatori **CONSOLE** porti.

Kommutaatori **SYST** tuluke põleb roheliselt. Kaabel peab olema katkematu: arvuti → sein → paneel → kommutaatori konsool.

📷 **Tõend:** foto kommutaatorist, kus on näha konsoolikaabel CONSOLE pordis → `kaabel.jpg`

!!! warning "Serveriruumis"
    Puutu ainult oma grupi kommutaatorit ja sinist konsoolikaablit. Teisi kaableid ja seadmeid ära tõmba välja: nendest käib kooli võrk.

## Samm 2: Leia COM-port

Windowsis: paremklõps **Start** → **Device Manager** → **Ports (COM & LPT)**.

Näed rida nagu `USB Serial Port (COM3)`. Number võib olla muu. PuTTY peab teadma, millise pordi kaudu kaabel arvutiga ühendatud on.

## Samm 3: Ava PuTTY

**Connection type:** Serial · **Serial line:** COM3 (sinu number) · **Speed:** 9600 · **Open**.

Avaneb must aken, esialgu võib see olla tühi. Cisco konsool räägib kiirusega 9600 bitti sekundis. Vale kiiruse korral näed segast teksti või mitte midagi.

📷 **Tõend:** ekraanipilt PuTTY seadetest enne **Open** vajutamist → `putty.png`

## Samm 4: Ärata terminal

Vajuta **Enter** paar korda. Näed `Switch>` või küsimust `Would you like to enter the initial configuration dialog? [yes/no]:`. Kui tuleb küsimus, kirjuta `no` ja Enter. Kui küsitakse parooli, kirjuta `1234`.

!!! tip "Kui midagi ei tule"
    - Kontrolli COM-numbrit (Device Manager) ja kiirust **9600**.
    - Kas kaabel on serveriruumis **CONSOLE** pordis, mitte tavalises pordis?
    - Kas laua pesa number ja paneeli number on samad?

## Samm 5: Seadista kommutaator

Kirjuta käsud ükshaaval. `XX` asemel kirjuta oma eesnimi.

```text
enable
configure terminal
hostname SW-XX
enable secret 1234
line console 0
 password 1234
 login
 exit
banner motd # Andra switch - ainult IT-26 #
end
show running-config
```

Prompt muutub: `SW-XX#`. `show running-config` näitab sinu hostname'i ja rida `enable secret 5 ...` (parool on krüpteeritud). Hostname näitab, millise seadmega töötad. Paroolid kaitsevad kommutaatorit.

📷 **Tõend:** ekraanipilt `show running-config` algusest → `show-run.png`

## Samm 6: Kontrolli paroole

Kirjuta `exit` ja vajuta Enter. Kommutaator küsib nüüd parooli: `1234`. Siis `enable` ja jälle `1234`.

💭 **Mõtle:** Kumb parool küsiti esimesena ja kumb pärast `enable`-it? Miks on vaja kahte?

**Vahetage nüüd rollid** ja tehke sammud 4–6 uuesti oma nimega (`hostname SW-teine-nimi`).

## Samm 7: Tee kommutaator puhtaks

Järgmise grupi jaoks:

```text
enable
write erase
reload
```

Kui küsitakse `Proceed with reload? [confirm]`, vajuta Enter. Kui küsitakse `Save? [yes/no]`, kirjuta `no`.

!!! tip "Kui unustasid parooli: MODE-nupp"
    Kommutaatori esiküljel vasakul on nupp **MODE**. Hoia seda **15 sekundit** all, kuni tuled vilguvad oranžilt, ja lase lahti. Kommutaator kustutab seadistuse ja taaskäivitub. Alusta uuesti sammust 4.

## Esitamine

Oma Classroomi repos brauseris:

1. **Add file → Upload files:** `kaabel.jpg`, `putty.png`, `show-run.png`
2. Ava `README.md`, vajuta pliiatsit, täida nimi, paariline, kommutaatori number, read `show running-config`-ist ja vasta neljale küsimusele. **Commit changes**.

## Kokkuvõte

- Ühendusid päris kommutaatoriga konsooli kaudu, ilma IP-aadressita
- Seadistasid nime, paroolid ja bänneri ning kontrollisid neid
- Tegid kommutaatori järgmise grupi jaoks puhtaks
