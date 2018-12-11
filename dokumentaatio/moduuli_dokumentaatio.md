# Metasploit moduuli firefox salasanojen varastukseen.

## Esittely

Tässä on dokumentaatio metasploit moduulista, jonka tarkoitus on varastaa meterpreter yhteyden kautta Firefoxin salasana tiedostot.  
Moduuli on rakennettu enum_files pohjalta ja on kokonaisuudessaan melko yksinkertainen.
Tein mielummin oman moduulin enum_files pohjalta, sillä valmis firefox_cred moduuli lataa paljon ylimääräistä tavaraa ja halusin pitää liikenteen minimaalisena.  


Työkaluina käytin Armitagea moduulin käyttämiseen, Virtualboxia testikoneen pyörittämiseen ja Virtualboxissa aikaisemmassa dokumentissa luotua testikonetta, sekä atomia moduulin tekemiseen.
Käytettyyn testikoneeseen oli valmiiksi syötetty kaksi pseudo käyttäjätunnusta ja salasanaa, jotta moduuli oikeasti hakisin jotain kohteesta.
Koska moduulia käytetään meterpreter yhteyden kautta, niin piti virustorjunnat ottaa pois päältä, että yhteys toimisi ja moduulia pystyisi käyttämään.

## Moduulin rakennus

Aikaisemmassa dokumentaatiossa kävi ilmi, että Firefox säilyttää tallennettuja salasanoja kahdessa tiedostossa, joista toinen on logins.json, jossa Firefox säilyttää itse tunnukset ja salasanat, sekä key*.db jossa Firefox säilyttää avainta jolla salasanat on kryptattu. 
Tarvitsemme siis molemmat tiedostot, jos haluamme avata salasanat.
  
Aloin tutkimaan miten saisin haettua tiedostot helposti, kun törmäsin metasploitissa olevaan post exploittiin nimeltä **enum_files**. Enum_filesilla pystyi hakemaan tietyltä kovalevyltä kaikki tiedostot, jotka vastasivat hakuun annettua tiedoston nimeä.  
Testasin enum_filesin toimintaa antamalla haettavaksi tiedostoksi key*.db. Tähtimerkinnällä viitataan siihen, ettei tiedetä mikä numero key*.db tiedoston nimessä on, joten moduuli hakee kaikki kuvaukseen sopivat tiedostot. Firefoxissa tosin pitäisi olla vain yksi key*.db per profiili, joten tämä ei tuota ongelmia. Haettavaksi kohteeksi asetin koko C:/ aseman. 

![Haku ikkuna](https://github.com/Tommi852/virustestaus/raw/master/media/enum_files_original.png)
  
Enum_files löysikin firefoxin key*.db tiedoston ja latasi sen. Moduulissa oli tosin ongelmana se, että se osasi ladata vain yhden tiedoston kerrallaan. Kokeilin erilaisia tapoja saada sitä lataamaan useamman tiedoston, kuten antamalla haettavaksi kohteeksi **key*.db && logins.json**, mutta tuloksetta. Näin ollen päätin, että minun pitää kustomoida moduulin koodia, jotta se toimii niinkuin haluan.
  
Enum_filesin koodi on kirjoitettu Rubylla ja se löytyy polusta: /usr/share/metasploit-framework/modules/post/windows/gather/enum_files.rb. Polku voi vaihdella, jos metasploit on rakennettu toiselle distrolle tai githubin kautta. 
  
Kopioin enum_files.rb tiedoston polkuun: ~/.msf4/modules/post/windows/gather/, joka on tarkoitettu nimenomaan metasploit moduulien kehittämistä varten ja kolmannen osapuolen moduuleja varten.  
Koska en osaa Rubya, niin koodin tutkiminen ja joten kuten ymmärtäminen tuotti hieman vaikeuksia, mutta lukuisten kokeilujen jälkeen löysin melko yksinkertaisen tekniikan, jolla sain enum_filesin hakemaan kaksi eri tiedostoa kohteesta.
  
Aluksi lisäsin register_options kohtaan uuden asetuksen, johon asetin haettavaksi logins.jsonin.
```
    register_options(
      [
        OptString.new('SEARCH_FROM', [ false, 'Search from a specific location. Ex. C:\\', 'C:/']),	# Tämä kertoo mistä levyltä haetaan.
        OptString.new('FILE_GLOBS',  [ true, 'Filename for decrypt file. Ex. key3.db', 'key*.db']),	# Tämä kertoo tiedoston, jota etsitään kohteesta.
        OptString.new('FILE_GLOBS2',  [ true, 'File name for logins file. Ex. logins.json', 'logins.json'])	# Tämä on itse lisäämäni kopio ylemmästä vaihtoehdosta toiselle tiedostolle.
      ])
```
Tämän jälkeen loin koodin uudelle FILE_GLOBS2 vaihtoehdolle, että sekin haetaan kohteesta:
```
    datastore['FILE_GLOBS2'].split(",").each do |glob|
      begin
        download_files(location, glob.strip)
      rescue ::Rex::Post::Meterpreter::RequestError => e
        if e.message =~ /The device is not ready/
          print_error("#{my_drive} drive is not ready")
          next
        elsif e.message =~ /The system cannot find the path specified/
          print_error("Path does not exist")
          next
        else
          raise e
        end
      end
    end
```
Koodi on siis täysin sama, kuin mikä oli FILE_GLOBS vaihtoehdolle, mutta eri datastorella.  
Tämä ei ole puhtain tai optimaalisin tapa toimia, sillä nyt koodi hakee uudestaan kovalevyltä tätä tosita tiedostoa ja lataa sitten sen, vaikka tiedostot ovat täysin samassa polussa. Jouduin kutienkin tyytymään tähän ratkaisuun, sillä en ymmärrä tarpeeksi Rubya, että osaisin rakentaa tämän optimaalisemmaksi ja näin tämä toimii.
  

## Tiedostojen purku.

Moduulin lataamat tiedostot menevät kansioon polussa: ~/.msf4/loot ja ovat hieman kryptisillä nimillä.  
Tiedostojen nimistä käy ilmi latausaika, IP-osoite josta ladattu, sekä tiedosto pääte. Tiedosto päätteissä on pieni ongelma, sillä koodi ei osaa nimetä .json tiedostoa samalla nimellä vaan muuttaa sen .bin nimiseksi tiedostoksi. Kansiosta löytyy kaksi kumpaakin tiedostoa, eli .db ja .bin tiedostoa on kumpaakin kaksi kappaletta. Tämä johtuu todennäköisesti siitä, että koneelta löytyy kaksi kertaa nämä tiedostot OpenSSH:n luoman linkin kautta.  
Näin ollen muutin tiedostojen nimet hieman ymmärrettäviksi. Nimesin yhden .db tiedoston key4.db tiedostoksi ja yhden .bin tiedoston logins.json tiedostoksi.  
  
Kopioin kummatkin tiedostot uuteen kansioon kotihakemistossa nimeltä "purku", jossa aijon purkaa nämä tiedostot oikeiksi salasanoiksi. Koska purkaminen olisi käsin melko työlästä, niin käytin apuna firefox_decrypt.py skriptiä, joka löytyy sen kehittäjän githubista osoitteesta: https://github.com/unode/firefox_decrypt  
Nopealla silmäilyllä koodi ei näytä haitalliselta itselleni.
  
Tiedostojen purkamisen piti onnistua komennolla: **python firefox_decrypt.py purku/**, mutta itse sain virheeksi: **ERROR - Couldn't initialize NSS, maybe '/root/purku' is not a valid profile?**  
Tiedosto polku oli oikein, mutta ilmeisesti skripti olisi halunnut myös profile.ini tiedoston mukaan purkamiseksi. Kokeilin erilaisia tekniikoita, kuten NSS uudelleen asennuksen, mutta nämä eivät auttaneet.  
Lopuksi löysin firefox_decryptistä tehdun forkin nimeltä ff-password-exporter, joka löytyy osoitteesta: https://github.com/kspearrin/ff-password-exporter
  
Ff-password-exporter, jolla oli mainittu erikseen, että se tukee Firefox58+ versioissa käytettyjä key4.db profiileja. Ff-password-exporter on firefox_decrypt.py pohjautuva spin-offi, jossa on graafinen käyttöliittymä.  
Ff-password-exporterin käyttö oli erittäin suoraviivaista. Ohjelmalle piti aluksi antaa suoritus oikeudet, joka onnistuu esimerkiksi klikkaamalla oikealla kuvaketta ja laittamalla ruksin execute kohtaan permissions välilehdellä.  
Tämän jälkeen ohjelman pystyi suorittamaan ja ohjelmasta valittiin vain kansio, jossa key4.db ja logins.json ovat. Koska käytössä ei ollut master passwordia, niin voi kentän jättää tyhjäksi.  

(kuva)

Tämän jälkeen ohjelma kysyy mihin kansioon tulokset halutaan tallentaa ja missä muodossa (json tai CSV).  
Tiedostossa näkyvät purkamisen jälkeen salasanat ja käyttäjätunnukset selkokielisenä.

(kuva)




