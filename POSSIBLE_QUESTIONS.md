1) Transport Layer Security
- Was bedeutet ECDHE-RSA-AES128-GCM-SHA256?
- Beschreiben Sie den TLS Handshake und die ausgetauschten Nachrichten.

2) TLS-Nachrichten, Formate, etc.
- Beschreiben Sie die Inhalte in den einzelnen TLS-Nachrichten, z. B. Client Hello, Certificate, Client KEX, etc.
- Welche Auswirkungen auf die Sicherheit des Protokolls hätte es, wenn man einzelne Nachrichten rausstreichen würde?

3) Schlüsselaustausch
- Wie funktioniert der RSA-basierte Schlüsselaustausch?
- Wie funktioniert der Diffie-Hellman-basierte Schlüsselaustausch?
- Wie unterscheiden sich die beiden Schlüsselaustauschverfahren aus Sicherheitssicht?

4) Forward Secrecy (FS)
- Was ist FS?
- Welche Eigenschaften hat FS?
- Mit welchen Algorithmen wird FS erreicht?

5) Elliptische Kurven
- Wie funktioniert ECDHE?
- Was bedeutet [a]G? 
  - a ist die Anzahl der Kopien eines Punktes, G ist der Punkt, [a]G nennt man Skalarmultplikation
  - Ist der public key
- Was ist G?
  - Ein Punkt
- Was ist typischerweise a?
  - Eine große Primzahl
  - 
- Welche Formen von elliptischen Kurve kennen Sie (keine Gleichungen, nur Namen)?
- Warum gibt es verschiedene Formen?
- Welche Kurve haben Sie im Praktikum benutzt?

6) Symmetrische Verschlüsselung und Verschlüsselungsmodi:
- Was ist der Unterschied zwischen symmetrischer und asymmetrischer Verschlüsselung? 
- Wofür wird symm. und asymm. Verschlüsselung verwendet? Was setzt TLS ein?
- Welche Schlüssellängen hat AES?
  - 128,192,256
- Welche Blockgrößen?
  - 128 (16 Byte)
- Wie verschlüsselt man Nachrichten, die größer als die Blockgröße des Verschlüsselungsalgorithmus sind?
- Was ist ECB? Ist es sicher?
- Was ist CBC? Ist es sicher?
- Was ist CTR? Ist es sicher?
- Was ist GCM und was ist AEAD? Ist das sicher?

7) Hashes
- Welche Eigenschaften haben kryptografische Hashverfahren? 
- Was bedeutet schwache Kollisionsresistenz?
- Was bedeutet starke Kollisionsresistenz?
- Nennen Sie zwei Szenarien, bei welchen jeweils schwache/starke Kollisionsresistenz benötigt wird. 
- Welche Konstruktion nutzt z.B. SHA1? Welche Angriffe gegen SHA1 gibt es?

8) Zertifikate, Zertifikatsketten und PKI
- Wofür braucht man Zertifikate?
- Welche Inhalte sind in einem Zertifikat?
- Wie funktioniert eine Zertifikatskette, von der CA-Root zum TLS-Leaf-Zertifikat.
- Wie stellt man fest, dass Zertifikate authentisch sind?
- Was ist eine digitale Signatur?
- Was ist ASN.1 und was ist DER/BER?

9) Keyed Hashes und MAC
- Wie kann man Hashes nutzen um Authentizität zu erlangen? 
- Was ist ein MAC?
- Was ist ein HMAC?
- Was ist MAC-and-Encrypt?
- Was ist MAC-then-Encrypt?
- Was ist Encrypt-then-MAC?
- Was verwendet TLS?

10) Spezielle Frage(n) zu bearbeiteten Thema.

11) Allgemeine Frage(n) zu einem anderen Thema.
