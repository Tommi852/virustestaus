# Rapid7 Evasion module tutkimista

Rapid7 on julkaissut uusimmassa metasploit versiossaan evasion moduulin, jonka tarkoituksena on piilottaa haittaohjelmat niin hyvin, että virustorjunnat eivät havaitse niitä.  
Lyhyt kuvaus uudesta moduulista löytyy täältä: https://blog.rapid7.com/2018/10/09/introducing-metasploits-first-evasion-module/  
  
Lisää tietoa haittaohjelman piilottamisesta löytyy täältä: https://blog.rapid7.com/2018/05/03/hiding-metasploit-shellcode-to-evade-windows-defender/  
  
Jälkimmäisessä blogi postauksessa käydään hyvin läpi itsekkin miettimääni tekniikkaa, jossa haittaohjelma kryptattaisiin avaimella, mutta ohjelma osaisi itse avata kryptauksen. Postauksessa todetaankin, että tämä on hyvä taktiikka virustorjunnan staattisen skannauksen ehkäisemiseksi, mutta ohjelmaa pyörittäessä virustorjunnat havaitsevat ohjelman käyttäytymisen ja osaavat estää sen.  
Toisena hyvänä pointtina postauksessa oli base64 väärinkäytön määrä, jonka takia kaikki powershell.exeen syötettävät base64 enkoodatut komennot ovat välittömiä red flageja haittaohjelmasta.
  
Tarkempi tekninen kuvaus evasion moduulin tekniikoista löytyy täältä: https://www.rapid7.com/globalassets/_pdfs/whitepaperguide/rapid7-whitepaper-metasploit-framework-encapsulating-av-techniques.pdf  
  
  
Valitettavasti msf5 on toistaiseksi vielä työn alla, eikä sitä ole jaettu vielä yleiseen jakeluun.
  
