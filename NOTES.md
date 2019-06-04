Es folgt eine gruselige Geschichte aus dem Jahre 2012

# Abstract
* RSA und DSA können katastrophale Fehler aufweisen wenn Zufallszahlen nicht korrekt berechnet werden
* Es wurde die größte Netzwerk "Umfrage" von TLS und SSH servern durchgeführt:
  * Angreifbare Keys sind weit verbreitet
  * 0,75% der TLS Zertifikate teilen sich Schlüssel wegen mangelnder Entropie bei der Schlüsselgenerierung
  * Es werden weitere 1,7% vermutet
  * RSA-Privatschlüssel von 0,5% der TLS und 0,03% der SSL Hosts konnten abgegriffen werden
    * Aufgrund von non-trivialen faktoren (Entropieproblem)
  * DSA-Privatschlüssel von 1,03% der SSH Hosts konnten ebenfalls abgegriffen werden
    * Mangel an Signaturzufall
  * Die meisten der angreifbaren Hosts sind "Headless" oder "Embedded" Geräte
  * Es konnten im Experiment durch die verwendung von 3 oft verwendeten Programmen die Problemursachen reproduziert werden
    * Eins der Probleme ist die Boot-Time Entropie des Linux Zufallsgenerators
  
# Introduction and Roadmap
* Moderne Kryptografie basiert heutzutage sehr oft auf auf Zufall und deren gleichmäßige Verteilung
* Es ist bereits bekannt, dass eine Hand voll Probleme bei der generierung von Zufallszahlen Konsequenzen hat
* Auch wenn man annehmen möchte, dass heutzutage die verbreiteten Betriebssysteme ihre Zufallszahlen möglichst sicher generieren, soll dieser Artikel  das Gegenteil zeigen

## Internetweite Untersuchung

* Der erste Teil des Papers ist die größte Internetweite Untersuchung von TLS und SSH (zwei der wichtigsten Crypto-Protokolle)
* Es wurde der gesamte IPv4 Bereich gescannt
* 5,8 Millionen einzigartige TLS Zertifikate von 12,8 Millionen Hosts
* 6,2 Millionen einzigartige SSH host keys von 10,2 Millionen Hosts
* Die verwendete Technik benötigte weniger als 24 Stunden um Hosts für die Protokolle zu finden
* Keys von den gefundenen Hosts zu beziehen dauerte weniger als 96 Stunden

## Analyse der Keys und Zertifikate
  
* Bei der Untersuchung auf Hinweise/Beweise auf Probleme bei der Generierung durch Zufallsfaktoren wurden
  * Bei 5,57% der TLS hosts
  * und bei 9,6% der SSH hosts
  * die gleichen Schlüssel verwendet (Kapitel 4.1)
* Bei mindestens 5,23% der TLS Hosts wurde der Standardschlüssel der Hersteller verwendet
* 0,34% Haben zwar einen neuen Schlüssel generiert, jedoch am Ende einen identischen Schlüssel durch fehlerhafte Zufallsgeneratoren erhalten
* Nur eine handvoll der angreifbaren TLS Zertifikate wurden von Browser-Trusted Zertifikatsstellen signiert
* Bei 64000(0,5%) TLS Hosts und 108000(1,06%) der SSH Hosts konnte der Privatschlüssel berechnet werden 
  * NUR durch ausnutzen der fehlerhaften Zufallsgeneratoren
* Wenn bei RSA zwei privatschlüssel sich nur einen Primfaktor teilen, dann wirken die öffentlichen Schlüssel erst einmal unterschiedlich, jedoch kann der GGT berechnet werden um den Faktor zu berechnen
  * Hierfür wurde ein Algorithmus entwickelt welcher alle 11 Millionen eindeutigen öffentlichen RSA Schlüssel auf ihren GGT prüft
  * ... In unter 2 Stunden
  * So wurden Privatschlüssel von 0,5% der TLS hosts und 0,03% der SSH hosts berechnet
* Bei DSA konnte bei Verwendung des gleichen flüchtigen Schlüssel für unterschiedliche Nachrichten effizient der private Schlüssel berechnet werden
  * Es wurden dadurch 1,6% der privaten Schlüssel von SSH DSA Hosts berechnet

## Verstehen warum das ganze passiert

* Manuelle Untersuchung von hunderten angreifbaren Hosts welche repräsentativ für die am häufigsten auftreteten privaten Schlüssel waren
* Es scheinen headless oder embedded Systeme zu sein. Unter anderem auch Router, Server Managment Karten, Firewalls und andere Netzwerkgeräte
  * Diese Geräte generieren oft beim ersten Start einen Schlüssel und haben jedoch eine geringe Entropie im Vergleich zu regulären Rechnern
  * Weitere Untersuchungen zeigten, dass geteilte private Schlüssel damit zusammenhingen welche Modelle und Hersteller es waren
    * Sprich: 
      * Gleiche Hersteller -> Gleiche Schlüssel wurden wahrscheinlicher
      * Gleiche Modelle -> Gleiche Schlüssel noch wahrscheinlicher
  * Dies führte die Autoren zu dem Ergebnis, dass die Probleme den fehlerhaften implementierungen zu schulden sind und vor allem nicht genug Entropie für die Berechnung der Schlüssel gesammelt wurde
  * Es wurden Geräte und Software von 54 Herstellern gefunden (einschließlich einige der größten Hersteller)
    
## Experimentelle Ursachenforschung

  * (Kapitel 5)
  * Experimente zur Ermittlung der Kernprobleme
    * Eine Untersuchung der am weitesten verbreiteten Open-Source Software Module unter den angreifbaren Geräten
  * Die Geräte die untersucht wurden zeigen, dass es nicht an "genau einer Implementierung" liegt
    * Die Schwachstellen konnten jedoch plausibel reproduziert werden durch Softwarekonfiguration
    * Jedes untersuchte Paket berief sich auf den Pfad "/dev/urandom" zur Generierung von Crypto-Schlüsseln
    * Das Problem: Dieser Zufallsgenerator (RNG) produziert vor allem auf headless und embedded geräten einen deterministischen Output
  * In einem Experiment mit OpenSSL und Dropbear SSH konnte gezeigt werden wie wiederholender Output des Generators nicht nur zu sich wiederholenden privaten Schlüssel führte, sondern auch zu faktorisierbaren RSA Keys und wiederholenden flüchtigen DSA Keys
 
## Das Problem der Lösung

* Durch die starke Verbreitung des Problems benötigt die Lösung ein ahndeln von vielen Parteien
* Kapitel 7 zeigt einige Lösungen
* In Kapitel 8 wird ein online key-check service vorgestellt um seinen schlüssel auf Sicherheit zu prüfen
* Hinterfragen der Sicherheit von RSA und DSA ist in diesem Zusammenhang verständlich
  * Die "Margin of Safety" ist deutlich dünner als man meinen möchte
  * Jedoch ist die Sicherheit von korrekt generierten Schlüsseln vonn traditionellen PCs aus sicht der Autoren nicht gemindert
  * Es wurde zwar ein ganzt spezieller Teil der kryptographischen Algorithmen ausgenutztn, jedoch liegt die Schwachstelle eindeutig in der Implementierung
* Abschließend machen die Authoren darauf aufmerksam dass ihre Arbeit ein Wake-Up Call für sichere Zufallsgeneratoren als ungelöstes Problem sein soll

# Hintergrund

## RSA Beurteilung

* Öffentlicher Schlüssel besteht aus 2 Zahlen
  * Der Exponent `e`
  * Der Modulus `N`
    * `N` besteht aus 2 zufällig gewählten Primzahlen `q` und `p`
  * Der private Schlüssel ist
    * ![Formel für d](https://latex.codecogs.com/gif.latex?d%20%3D%20e%5E%7B-1%7D%5C%20mod%5C%20%28p-1%29%28q-1%29)
  * Ist die Faktorisierung von N möglich so kann sehr einfach der private schlüssel jedes öffentlichen Schlüssels berechnet werden
* Faktorisierbare RSA Schlüssel
  * Ein 1024 Bit Modulus ist bisher nicht faktorisiert worden
  * Der größte bekannte Schlüssel war 768 Bits lang und wurde 2009 nach mehreren Jahren von einem Rechnernetzwerk faktorisiert
  * Im Vergleich dazu kann der GGT von zwei 1024-Bit Zahlen im Bruchteil einer Sekunde berechnet werden
  * Dies führt zu einer sehr bekannten Schwachstelle:
   * Findet ein Angreifer zwei unterschiedliche Moduli `N1` und `N2` die einen gemeinsamen Primfaktor `p` teilen
   * So ist der GGT von `N1` und `N2` ... *trommelwirbel* ... `p` 
   * Teilt man `N1` und `N2` durch `p`so erhält man `q1` und `q2`
   * Die nachfolgende Berchnung ist trivial um den privaten Schlüssel zu erstellen
   
## DSA Beurteilung

* Besteht aus 
  * 2 Primzahlen p und q (Domain-Parameter)
  * 1 Generator g aus der Gruppe "q mod p" (Domain-Parameter)
  * und ![Formel für y](https://latex.codecogs.com/gif.latex?y%20%3D%20g%5Ex%5C%20mod%20%5C%20p)
    * `x`ist hierbei der private Schlüssel
  * Die sog. Domain-Parameter können zzwischen mehreren öffentlichen Schlüsseln geteilt werden ohne die Sicherheit zu gefährden
* Die Signatur besteht aus einem Zahlenpaar von `r`und `s`
  * ![Formeln für r und s](https://latex.codecogs.com/gif.latex?r%20%3D%20g%5Ek%5C%20mod%5C%20p%5C%5C%20s%20%3D%20%28k%5E%7B-1%7D%28H%28m%29&plus;xr%29%29%5C%20mod%20%5C%20q)
  * `k` wird hierbei zufällig gewählt und ist ein flüchtiger privater Schlüssel
  * `H(m)` ist ein Hash der Nachricht die es zu verschlüsseln gilt
* Die Gefahr
  * Ist `k` bekannt so kann der private Schlüssel `x` berechnet werden
    * ![Formel für x](https://latex.codecogs.com/gif.latex?x%20%3D%20r%5E%7B-1%7D%28ks-H%28m%29%29%5C%20mod%20%5C%20q)
  * Noch schlimmer wird es wenn `k`doppelt verwendet wird, denn dann kann auch ohne das wissen über k der private Schlüssel berechnet werden
    * Hierzu wird `k` zuerst mit Hilfe beider Nachrichten und den öffentlichen Schlüsseln berechnet
    * ![Formel für k](https://latex.codecogs.com/gif.latex?k%20%3D%20%28H%28m_1%29-H%28m_2%29%29%28s_1-s_2%29%5E%7B-1%7D%5C%20mod%5C%20q)
    * Im Anschluss kann dann mit `k` wie oben beschrieben der private Schlüssel `x`berechnet werden
    * Hinweis hier: r1 und r2 sind laut formel durch gleiches k identisch. Es kann also dem r angesehen werden ob k mehrfach verwendet wurde
   
## Angriffsszenario
TLS und SSH verwenden normalerweise RSA oder DSA zur Authorisierung zwischen Server und Client

### TLS
* Ein öffentlicher Schlüssel wird in einem TLS Zertifikat während des Handshakes übermittelt
* Wird ein RSA Schlüsselaustausch verwendet so kann ein passiver Zuhörer mit Hilfe des privaten Schlüssels bereits alle Sessions entschlüsseln
* Wird ein Diffie-Hellmann Schlüsselaustausch verwendet so reicht es nicht nur das Transkript der Verbindung zu erhalten
* In beiden Fällen ist mit Hilfe des privaten Schlüssels ein aktivier Angreifer bspw als man-in-the-middle in der Lage den gesamten Datenverkehr zu entschlüsseln, zu lesen, zu manipulieren und verschlüsselt weiterzuleiten

### SSH
Hier gibt es 2 Unterscheidungen von SSH Versionen
* SSH-1
 * Der Client verschlüsselt Session Key Material mit dem öffentlichen Schlüssel des Servers
* SSH-2
 * Diffie-Hellmann Schlüsselaustausch
 * Der Benutzer bestätigt manuell den Fingerprint des Servers bei der ersten Verbindung
 * Die meisten Clients speichern den Schlüssel dann unter "known_hosts" und trauen dem host ab dort automatisch

Die Konsequenz
* SSH-1
 * Ein passiver Zuhörer kann mit dem privaten Schlüssel die gesamte Session abhören
* SSH-2
 * Hier ist dank Diffie-Hellmann nur ein Man-in-the-Middle Angriff möglich
 
Eine Zeitbombe hierbei:
SSH überträgt nach Aufbau der vermeidlich sicheren Verbindung das vom Benutzer eingegebene Passwort im Klartext über den "sicheren" Kanal, welcher jedoch abgehört werden könnte. Dies kann selbst einem passiven Zuhörer mit genug Geduld Zugang zu dem System des Servers verschaffen.
