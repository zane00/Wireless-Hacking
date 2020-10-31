# Crack WPA/WPA2 Wi-Fi Routers

Wi-Fi Protected Access (WPA) e Wi-Fi Protected Access II (WPA2) sono due protocolli di sicurezza per proteggere le reti di
computer wireless.

Sono stati sviluppati come rimedio alle vulnerabiltà riscontrate nel sistema precedentemente utilizzato: WEP (Wired 
Equivalent Privacy).

Dal 2018 è disponibile anche il protocollo WAP3.

***ATTENZIONE***

***Il seguente articolo è a puro scopo informativo e il suo contenuto non deve essere utilizzato per alcuna attività
illegale.***

***L'autore non è responsabile di alcuna attività malevole causata dalla lettura di questo post e ne prende le distanze.***

## Introduzione

Prima di iniziare è necessario chiarire che la piattaforma da cui verranno inviati i comandi è Kali Linux e i tool utilizzati
sono i seguenti:
  * Aircrack-ng

Per installare `aircrack-ng` è necessario digitare il seguente comando\
`sudo apt-get install aircrack-ng`

## Cracking a Wi-Fi Network

Iniziamo mostrando tutte le interfacce che supportano la monitor mode\

`airmon-mg`

Da questo punto in poi utilizzeremo l'interfaccia _wlan0_.

`airmong-ng start wlan0`

Eseguendo `iwconfig` troveremo quindi l'interfaccia _wlan0_ tra quelle elencate.
Solitamente è presentata tramite la stringa `mon0` o `wlan0mon`.

### Target

Proseguiamo quindi nel trovare il bersaglio dell'attacco.\
Interroghiamo il beacon frame dello standard 802.11: uno dei frame di gestione dello standard IEEE 802.11 che contiene tutte le informazioni della rete. Viene lanciato periodicamente e serve per annunciare la presenza di una VLAN.

`airodump-ng mon0`

Verranno quindi elencate tutte le VLAN presenti.
Necessario ricordarsi il MAC BSSID e il numero del canale che utilizza (`CH`) della VLAN che si desidera attaccare.

### Catturare il 4-way Handshake

I protocolli WPA e WPA2 utilizzano un 4-way handshake per autenticare i disponsitivi nella rete.
Viene effettuato ogni volta che un dispositivo si connette alla rete.
Utliizzando `airodump-ng` procediamo a catturare l'handshake andando a digitare il seguente comando:

`airodump -ng -c x -bssid XX:XX:XX:XX:XX:XX -w . mon0`

Facciamo un po' di chiarezza sui flag utilizzati:
-c e -bssid permettono di indicare il canale e il BSSID trovati in precedenza.
-w permette di salvare l'output contenente il pacchetto catturato

Ora sarà necessario aspettare finchè non sarà catturato l'handshake.
Dovresti vedere qualcosa di simile a `[WPA handshake: xx:xx:xx:xx:xx:xx]` in alto a destra del tuo monitor.
Terminando il processo di `airodump-ng` e recandosi presso la cartella di output dovremmo trovare un file con estensione `.cap`.

### Crack della password con John the Ripper

L'utlimo step è quello di crackare la password del router.
D'ora in poi utilizzeremo il tool `John the Ripper` ma è possibile svolgere questo passo anche tramite `hashcat`.

Per prima cosa è necessario convertire il formato del file da `.cap` a `.hccap` tramite `cap2hccapx`:\
`cap2hccapx file.cap converted-file.hccap`\

Possibile la traduzione online tramite i seguenti link:
 * http://sourceforge.net/projects/cap2hccap/files/
 * https://hashcat.net/cap2hccap/

Una volta ricavato il file `.hccap` è necessaria un'ulteriore conversione nell'estensione di input comprensibile da `john`.\
Utilizzando il tool `hccap2john` la sintassi è la seguente:\
`hccap2john converted-file.hccap > crackme`\

Andiamo quindi ad utlizzare `john` per la vera e propria operazione di crack:\
`./john -w = passwordlist.txt -form = wpaspk-opencl crackme`

Se la password sarà contenuta nel file `passwordlist.txt`, john non dovrebbe impiegare troppo tempo per trovarla e stamparla in console.

## Letture consigliate

Ci tengo a citare alcune pagine da cui ho preso spunto per la scrittura del seguente articolo e altre in cui è possibile approfondire alcuni argomenti citati nel documento.
 * Crack WPA/WPA2 Wi-Fi Router with Aircrack-ng and Hashcat: https://medium.com/@brannondorsey/crack-wpa-wpa2-wi-fi-routers-with-aircrack-ng-and-hashcat-a5a5d3ffea46
 * Beacon frame: https://en.wikipedia.org/wiki/Beacon_frame
 * Standard IEEE 802.11 e 4-way handshake: https://it.wikipedia.org/wiki/IEEE_802.11i
 * Airodump-ng: https://www.aircrack-ng.org/doku.php?id=airodump-ng
 * Cracking WPA-PSK/WPA2-PSK with John the Ripper: https://openwall.info/wiki/john/WPA-PSK
 * Cracking WPA/WPA2 with hashcat: https://hashcat.net/wiki/doku.php?id=cracking_wpawpa2
 * Converting .cap in .hccap: https://stuffjasondoes.com/2018/07/18/converting-aircrack-ng-hashes-to-hccapx-format-and-cracking-with-hashcat/

