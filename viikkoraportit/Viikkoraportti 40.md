# Viikkoraportti viikolle 40, Virusturvan ja IDS:n ohittaminen  

## Tehdyt tunnit  

### Tommi
#### Keskiviikko 03.10.2018
* Klo 12:00-15:15 **yht: 3h 15min**  
Msfvenomilla troijalaisen teko ja enkoodauksen testaaminen, sekä Unicornilla koitettu piilottaa ohjelma virustorjunnalta.  
Kummatkin haittaohjelmat toimivat ilman virustorjuntaa ja antoivat etähallinan, paitsi Avastilla varustettu virtuaalikone ei antanut virusten suoriutua vaikka suojaus oli otettu pois päältä.  
Virustorjunnat päällä ei troijalainen toiminut kummallakaan virtuaalikoneella.  
  

#### Torstai 04.10.2018
* Klo 12:00-12:30 **yht: 30min**  
Pikainen katsaus PowerShell Empire toimintaan koulussa, mutta luokkassa, jossa töitä teimme alkoikin tunti ja jouduimme keskeyttämään.  

#### Perjantai 05.10.2018  
* Klo 14:00-19:00 **yht: 5h**  
Kalin asennus pysyvästi koneelle, Powershell Empiren ja VirtualBoxin asennus koneelle.  
Leikitty PowerShell Empirellä. Testattu erilaisia moduuleja ja kuuntelijoita. Meterpreter kuuntelija moduuli on buginen ja sitä ei kannata käyttää.  
HTTP agentti toimii hyvin, mutta Kaspersky havaitsee käynnistys scriptin. Itse yhteyttä Kaspersky ei tosin huomaa.  
Virusskannerin tarkistus moduuli toimi hyvin ja Kaspersky ei huomannut sen pyöräytystä moduulien kautta.  


#### Sunnuntai 07.10.2018  
* Klo 16:00-18:30 **yht: 2h 30min**  
Onnistuneesti ohitettu Avast virustorjunta Unicornin avulla. Dokumentaatio pitää hioa vielä.  
  
#### Tunnit yhteensä: 11h 15min

### Aleksi  
# Aleksin tuntiraportti viikko 40, IDS ja virusturvan ohittaminen

#### Keskiviikko 03.10.2018  
* Klo 12:00-15:09 **yht: 3h9min**  
msfvenomin käyttö, viikkokokous  
Sain tehtyä yksinkertaisen hyökkäyksen virusturvan ollessa pois käytöstä msfvenomilla reverse tcp:tä käyttäen ja tommin   ohjeistuksella  

#### Torstai 04.10.2018
* Klo 12:00-12:30 **yht: 30min**  
Pikainen katsaus PowerShell Empire toimintaan koulussa, mutta luokkassa, jossa töitä teimme alkoikin tunti ja jouduimme keskeyttämään.

#### Perjantai 05.10.2018
* klo 12:00-14:30 **yht: 2h30min**
Powershell empirellä leikkimistä livetikun kanssa kotona ja harjoittelua ylipäättäänsä  
  
#### Tunnit yhteensä: 6h 9min

## Viikon yhteenveto:
Projekti on saanut tuulta enemmän alleen ja ensimmäinen virusturvan ohitus on tapahtunut Unicornin avustuksella.  
Projekti on muutamasta kovalevyn rikkomisesta huolimatta hyvässä vauhdissa.

