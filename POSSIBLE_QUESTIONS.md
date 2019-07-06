# Mögliche Fragen in der mndl. Prüfung
## Transport Layer Security

### Was bedeutet ECDHE-RSA-AES128-GCM-SHA256?
  - Ist eine Ciphersuite mit folgenden Eigenschaften:
    - Schlüsselaustausch mit ECDHE
    - Authentifizierung mit RSA
    - AES128-GCM für Datenverschlüsselung 
    - SHA256 als Hashfunktion

### Beschreiben Sie den TLS Handshake und die ausgetauschten Nachrichten.
#### TLS 1.2
![TLS 1.2](https://tlseminar.github.io/images/firstfew/tls-hs-ecdhe.png)
#### TLS 1.3
![TLS 1.3](https://timtaubert.de/images/tls13-hs-ecdhe.png)
https://www.youtube.com/watch?v=yPdJVvSyMqk

## TLS-Nachrichten, Formate, etc.
### Beschreiben Sie die Inhalte in den einzelnen TLS-Nachrichten, z. B. Client Hello, Certificate, Client KEX, etc.
https://www.youtube.com/watch?v=cuR05y_2Gxc
#### TLS 1.2
- ClientHello: 
  - Unterstütze Ciphersuites
  - Versionsnummer
- ServerHello:
  - Wählt erstmal Ciphersuite aus die genutzt werden soll
  - Enthält Zertifikat
    - Enthält wiederum den Pubic Key für die Verschlüsselung
  - Enthält Ciphersuite
- ServerHelloDone
- Client generiert Pre-Master Secret (PMS)
- Client Key Exchange + Change Cipher Spec
  - Sendet PMS verschlüsselt (mit hilfe des Public Keys) an Server
- ClientFinished
- Server kann den Key Exchange entschlüssen und nun haben beide das gleiche PMS
  - Ab hier ist verschlüsselung möglich da beide einen symmetrischen Schlüssel haben
- Server Change Cipher Spec
- Server Finished
- Application Data kann verschlüsselt übertragen werden
  - "Bulk Data"

#### TLS 1.3
- ClientHello
  - Versionsnummer
  - Unterstütze Chipersuites
  - Key Agreements TODO: Klären
  - Keyshare: Vorberechnete Keys für die unterstützen Ciphersuites
- ServerHello
  - Gibt die Ciphersuite an
  - Schickt Keyshare mit
  - Server kann an dieser Stelle verschlüsseln
  - Zertifikat wird mit übertragen
  - Finished Nachricht kann auch direkt mitgesendet werden
- ClientFinished
  - Fnished
  - Kann ab hier auch verschlüsseln
  - Application Data
- ServerHelloRetryRequest
  - Wird gesendet wenn keine Ciphersuite vom Server unterstützt wird
  - Schickt Cipersuite-Liste mit
### Welche Auswirkungen auf die Sicherheit des Protokolls hätte es, wenn man einzelne Nachrichten rausstreichen würde?

## Schlüsselaustausch
### Wie funktioniert der RSA-basierte Schlüsselaustausch?
- Alice und Bob generieren privaten und öffentlichen Schlüssel
- Public Keys werden ausgetausch
- Daten werden zum senden mit dem Public Key verschlüsselt
- Privater Schlüssel kann verwendet werden um zu entschlüsseln
#### Berechnung der Schlüssel
- Wähle p und q Primzahlen
- n = p*q (1. Teil des Public Keys)
- phi(n) = (p-1)(q-1)
- Wähle e mit ggT(E,phi(n))=1
=> Public Key = (n,e)
- Wähle d mit 0<d<phi(n), sodass gilt e*d kongruent zu 1 (mod phi(n))
=> Private Key = d
- Verschlüsseln:
  - c = m^e mod n
- Entschlüsseln
  - m = c^d mod n
### Wie funktioniert der Diffie-Hellman-basierte Schlüsselaustausch?
https://www.youtube.com/watch?v=Yjrfm_oRO0w
- Alice und Bob generieren privaten Schlüssel
- Generator g
- Primzahl n ( 2kBit-4bit )
- Öffentlich: g und n
- g wird mit den privaten Schlüsseln verrechnet:  g^a mod n
  - Werden im Anschluss ausgetauscht (öffentlich und unverschlüsselt)
- Alice und Bob berechnen Schlüssel mit Hilfe der empfangenen Werte
  - g^a^b mod n
### Wie unterscheiden sich die beiden Schlüsselaustauschverfahren aus Sicherheitssicht?
- Beide stützen sich auf die schwierigkeit der Berechnung des diskreten Logarithmus
- Im Falle von DH haben am Ende beide Parteien den gleichen Schlüssel
  - Es gibt auch nur diesen Schlüssel
  - Asymmetrische Verschlüsselung
    - Man Endet jedoch mit gleichem Key für evtl. symmetrische Verschlüsselungen
- Bei RSA existiert ein Public und ein Private Key.
  - Public Key zum verschlüsseln
  - Private Key zum entschlüseln
  - Asymmetrische Verschlüsselung
## Forward Secrecy (FS)
### Was ist FS?
### Welche Eigenschaften hat FS?
### Mit welchen Algorithmen wird FS erreicht?

## Elliptische Kurven
### Wie funktioniert ECDHE?
### Was bedeutet [a]G? 
  - a ist die Anzahl der Kopien eines Punktes, G ist der Punkt, [a]G nennt man Skalarmultplikation
  - Ist der public key
### Was ist G?
  - Ein Punkt
### Was ist typischerweise a?
  - Eine große Primzahl
  - 
### Welche Formen von elliptischen Kurve kennen Sie (keine Gleichungen, nur Namen)?
### Warum gibt es verschiedene Formen?
### Welche Kurve haben Sie im Praktikum benutzt?

## Symmetrische Verschlüsselung und Verschlüsselungsmodi:
### Was ist der Unterschied zwischen symmetrischer und asymmetrischer Verschlüsselung? 
### Wofür wird symm. und asymm. Verschlüsselung verwendet? Was setzt TLS ein?
### Welche Schlüssellängen hat AES?
  - 128,192,256
### Welche Blockgrößen?
  - 128 (16 Byte)
### Wie verschlüsselt man Nachrichten, die größer als die Blockgröße des Verschlüsselungsalgorithmus sind?
### Was ist ECB? Ist es sicher?
  - Electronic Code Book: Nachricht in 16 Byte Blöcke schneiden und jeweils ver/entschlüsseln. Unabhängig voneinander
  - Letzter Block wird ggf mit Padding aufgefüllt PKCS 7
  Gleiche Klartextblöcke => Gleiche Chiffratblöcke 
### Was ist CBC? Ist es sicher?
  - Cahin Block Cipher
  - Initialvektor (zufällig) wird mit erstem Klartextblock verXORt und verschlüsselt
  - Chiffrat wird für nächsten Block als IV(Initialvektor) benutzt
  - IV wird unverschlüsselt mitübertragen
  - Entschlüsseln: Erst dec und dann XOR
### Was ist CTR? Ist es sicher?
 - Counter Mode
 - Stromchiffre aus Nonce (Wird mit jedem Block Hochgezählt)
 - Verschlüsselte Nonce wird mit Klartext-Block verXORt
 - Entschlüsseln genau so (auch mit der Verschlüsselung(!!) der Nonce)
 - Nonce wird mit übertragen
### Was ist GCM und was ist AEAD? Ist das sicher?
  - AEAD:
    - Authenticated Encryption with Associated Data
    - Entschlüsselung und Authentifizierung in einem Schritt
    - Verhindert Padding-Oracles
  -GCM:
    - Galois/Counter Mode
    - Sehr verbreitet und schnell!
      - AES Hardwareunterstützung AES-NI
      - Carryless Multiply
  - AES-GCM ist schwierig zu implementieren
  - Schwer Seitenkanäle auszuschließen
  - Nonce-Problematik
  - Sehr effizient auf Plattformen mit passender Hardware-Unterstützung
    

## Hashes
### Welche Eigenschaften haben kryptografische Hashverfahren? 
### Was bedeutet schwache Kollisionsresistenz?
### Was bedeutet starke Kollisionsresistenz?
### Nennen Sie zwei Szenarien, bei welchen jeweils schwache/starke Kollisionsresistenz benötigt wird. 
### Welche Konstruktion nutzt z.B. SHA1? Welche Angriffe gegen SHA1 gibt es?

## Zertifikate, Zertifikatsketten und PKI
### Wofür braucht man Zertifikate?
### Welche Inhalte sind in einem Zertifikat?
### Wie funktioniert eine Zertifikatskette, von der CA-Root zum TLS-Leaf-Zertifikat.
### Wie stellt man fest, dass Zertifikate authentisch sind?
### Was ist eine digitale Signatur?
### Was ist ASN.1 und was ist DER/BER?

## Keyed Hashes und MAC
### Wie kann man Hashes nutzen um Authentizität zu erlangen? 
  - Ein Hash aus Key und Plaintext ergibt eine Art Prüfsumme.
  - Diese kann nur von dem originalen Autoren stammen und kann von dem Empfänger gegengeprüft werden
### Was ist ein MAC?
  - Ein MAC ist eine Art Signatur einer Nachricht und die Quelle und Korrektheit der Nachricht zu garantieren
  - Das Ergebnis einer Hashfunktion welche den Key und den Plaintext beinhaltet
### Was ist ein HMAC?
### Was ist MAC-and-Encrypt?
  - MAC mit Plaintext + Key generieren
  - Plaintext verschlüsseln und danach MAC anhängen
### Was ist MAC-then-Encrypt?
  - MAC wird an den Plaintext angehangen und im Anschluss gemeinsam mit dem Plaintext verschlüsselt
  - Dadurch ist der MAC ebenfalls verschlüsselt
### Was ist Encrypt-then-MAC?
  - Erst verschlüsseln und dann Ciphertext + Key nutzen um MAC zu erstellen
### Was verwendet TLS?
  - Je nach Ciphersuite
    - MAC-then-Encrypt
  - Encrypt-and-MAC ist nicht gut
  - MAC-then-encrypt führt immer wieder zu Problemen
  - Enrypt-then-MAC ist das was man machen möchte (Ciphertext soll authentisch sein)

## Spezielle Frage(n) zu bearbeiteten Thema.

## Allgemeine Frage(n) zu einem anderen Thema.
