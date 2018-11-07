# Viikkoraportti viikolle 44, Virusturvan ja IDS:n ohittaminen  

## Tehdyt tunnit  

### Tommi
#### Perjantai 02.11.2018
* Klo 14:30-16:30 **yht: 2h**  
Smartscreenin ohittamisen tutkimista. SigThief ei toiminut.  


#### Lauantai 03.11.2018
* Klo 16:00-20:30 **yht: 4h 30min** 
Koitin liittää haittaohjelmasta tehdyn exe tiedoston Msfvenomin payloadilla generic/custom putty.exeen, mutta virustorjunta havaitsi tämän.  
sudo msfvenom --payload generic/custom PAYLOADFILE=/home/anon/haittaohjelma/varasPerus.exe -a x64 --template putty.exe --keep -f exe --out puttyVaras.exe  
Kokeilin myös shikataganailla enkoodata 570 kertaa, mutta ei toiminut silti vaan kaspersky huomasi samantien.  
Tutkittu veil-frameworkkia ja sen käyttö haittaohjelman piilottamiseen.

#### Yhteensä: 6h 30min


### Aleksi  

#### Torstai 01.11.2018
* Klo 08:50-09:50 **yht: 1h**  
Setookitin kolmannen osapuolen moduulin käytön lukemista

#### Yhteensä: 1h


## Viikon yhteenveto:
Projekti eteni tällä viikolla normaalia hitaammin. Ei edistystä smartscreenin ohituksen kannalta yrityksistä huolimatta.  
