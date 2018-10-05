# PowerShell Empire
PowerShell Empire on post-exploitation työkalu, jolla voidaan luoda tehokkaita haittaohjelmia, jotka hyödyntävät Windowsin PowerShelliä.  
Ohjelma tarjoaa kryptologisesti suojatun yhteyden ja joustavan arkkitehtuurin haittaohjelmille.  

PowerShell Empiren Git repository löytyy osoitteesta: https://github.com/EmpireProject/Empire  


## Asennus  

Aluksi kloonataan PowerShell Empiren git repository komennolla:
```
git clone https://github.com/EmpireProject/Empire.git
```

Repon latauduttua navigoidaan kansioon Empire/setup  
Tässä kansiossa tiedosto "install.sh", jonka ajamalla voimme asentaa PowerShell Empiren.
Ajamme siis asennuksen komennolla: ***./install.sh***
Asennus kysyy jonkin ajan päästä salasanaa, jota käytetään palvelimen neuvotteluun.  
Tämän jälkeen asennus onkin valmis ja pääsemme käyttämään PowerShell Empireä ja voimme käynnistää sen Empire kansiosta komennolla: ***./empire***  

