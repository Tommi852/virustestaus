# IDS esto säännöt

## Esittely
  
Tässä dokumentissa käydään läpi aikaisemmin luodun haittaohjelman tunnistaminen Snortilla ja sen estäminen.

## Estäminen

Aluksi pitää tietää minkälaista dataa haittaohjelma lähettää. Tähän tuli avuksi Wireshark, joka näyttää kaikki verkossa liikkuvat paketit.  
Koska tässä tapauksessa tiesin, että ohjelma käyttää ftp protokollaa, niin suodatin vain ftp protokollaa käyttävät paketit.

(kuva)

Tarkemmin katsottuna yksi FTP paketeista näyttääkin infona, että se on lähettämässä salasanat.zip tiedostoa.
Kopioin kyseisen pätkän HEX tunnuksen, jotta sitä voi käyttää paketin tunnistamiseen.

(kuva)

```
0000   00 00 00 01 00 06 08 00 27 6a 59 ab 74 61 08 00   ........'jY«ta..
0010   45 00 00 3c 1b dd 40 00 80 06 0f 98 50 dd 91 4d   E..<.Ý@.....PÝ.M
0020   50 dd 9c 3f c2 e7 00 15 a2 93 7c 19 da ce f0 13   PÝ.?Âç..¢.|.ÚÎð.
0030   50 18 1e ef 27 25 00 00 53 54 4f 52 20 73 61 6c   P..ï'%..STOR sal
0040   61 73 61 6e 61 74 2e 7a 69 70 0d 0a               asanat.zip..
```


Tämän jälkeen asensin snortin komennolla:
```
Sudo apt-get -y install snort
```

Kokeilin, että snort toimii nopeasti avaamalla sen komennolla: "snort".  
Snort näyttiki, että paketteja liikkuu verkossa jonkin verran.
  
Tämän jälkeen siirryin tekemään sääntöä, jolla estää itse tekemäni ohjelma.  
Avasin snortin sääntö tiedoston komennolla:
```
sudo gedit /etc/snort/rules/local.rules
```

Tein sinne testiksi varoitus säännön ICMP varoituksen:
```
alert icmp any any -> $HOME_NET any (msg:”ICMP test”; sid:1000001; rev:1; classtype:icmp-event;)
```
Tämän jälkeen otin kyseisen säännön käyttöön komennolla: 
```
sudo snort -T -i eth0 -c /etc/snort/snort.conf
```
Nyt sääntö oli käytössä ja pystyin käynnistämään snortin tällä säännöllä komennolla:
```
sudo snort -A console -q -c /etc/snort/snort.conf -i eht0
```
  
Nyt täytyi vain triggeröidä tuo sääntö, jonka loin joten toisessa konsoli ikkunassa laitoin komennon: **ping 80.221.xx.xx**  

Kun laitoin pingin pyörimään niin Snort alkoikin pienellä viiveellä näyttämään, että se havaitsi säännön mukaisen ICMP paketin.

(kuva)


Tämän jälkeen loin säännön, joka tunnistaa haittaohjelman luoman tiedoston lähetys pyynnön.  
Sääntö jolla tuo viesti havaitaan näyttää tältä:
```
alert tcp any any -> any any (msg:"Salasana varas havaittu"; content:"STOR salasanat.zip"; sid:1000003; rev:1;)
```

Ja säännön kohtaaminen näyttää tältä:
(kuva varas havaittu)

Seuraavaksi siirryin tutkimaan miten tämä estetään.
  
Muutin aikaisempaa havaitsemis sääntöäni siten, että muutin alertin rejectiksi. Sääntö näytti siis tältä:
```
reject tcp any any -> any any (msg:"Salasana varas havaittu"; content:"STOR salasanat.zip"; sid:1000003; rev:1;)
```
Sain edelleen ilmoituksen, että paketti on havaittu, mutta jostain syystä se ei estynyt.

Pienen tutkiskelun jälkeen löysin keskustelua, että Snortin pitää olla inline tilassa, jotta se voi estää paketteja.  
Oletuksena oleva pcap tila ei tue inlineä, joten otin käyttöön afpacket tilan. Snortin käynnistys komento afpacket tilaan pääsemiseksi näytti tältä:
```
sudo snort -A console -q -c /etc/snort/snort.conf --daq afpacket -i eth0
```
Tässä tilassa snort kuitenkin kaatui **Segmentation fault** virheeseen, jos se kohtasi haittaohelmani paketin.
  
Löysin myös lopulta, että Snortin config tiedostoon pitää lisätä rivit:
```
config daq: afpacket
config daq_mode: inline
```
Näillä asetuksilla Snort toimisi inline tilassa afpacketilla.  
  
Tämän jälkeen kun käynnistin snortin, niin sain virheen:
```
ERROR: Can't initialize DAQ afpacket (-1) - afpacket_daq_initialize: Invalid interface specification: 'eth0'!
```
Pitkään tutkittuani löysin, tietoa että inline tilassa pitää olla kaksi interfacea, jos haluaa käyttää IPS tilaa.  
Nämä kaksi interface siis sillattaisiin snortin kautta, jotta snort pystyisi pudottamaan paketit. Valitettavasti tämä ei nyt onnistunut, mutta valmiiksi luodun säännön pitäisi toimia ja estää pakettini oikeassa ympäristössä.
  
Näin ollen tyydyimme vain paketin havaitsemiseen.




