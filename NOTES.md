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
