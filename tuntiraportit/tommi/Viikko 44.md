# Tommin tuntiraportti viikko 44, IDS ja virusturvan ohittaminen

#### Perjantai 02.11.2018
* Klo 14:30-16:30 **yht: 2h**  
Smartscreenin ohittamisen tutkimista. SigThief ei toiminut.  


#### Lauantai 03.11.2018
* Klo 16:00-20:30 **yht: 4h 30min** 
Koitin liittää haittaohjelmasta tehdyn exe tiedoston Msfvenomin payloadilla generic/custom putty.exeen, mutta virustorjunta havaitsi tämän.  
sudo msfvenom --payload generic/custom PAYLOADFILE=/home/anon/haittaohjelma/varasPerus.exe -a x64 --template putty.exe --keep -f exe --out puttyVaras.exe  
Kokeilin myös shikataganailla enkoodata 570 kertaa, mutta ei toiminut silti vaan kaspersky huomasi samantien.  
Tutkittu veil-frameworkkia ja sen käyttö haittaohjelman piilottamiseen.
