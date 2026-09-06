# Postfix

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

MediaWiki – die Grundlage der Wissenssammlung dieses Buchs – und Drupal, ein weiteres Inhaltssystem, das auf demselben Server laufen kann, verschicken an vielen Stellen E-Mails: Ein neues Benutzerkonto bekommt einen Bestätigungslink, ein vergessenes Passwort einen Link zum Zurücksetzen, eine beobachtete Seite eine Änderungsmeldung. Damit diese Nachrichten den Server verlassen, fehlt ein Programm, das sie annimmt und an den Mailanbieter des Empfängers zustellt. Dieses Programm heißt **Mail Transfer Agent** (MTA). Der auf Linux mit Abstand verbreitetste ist **Postfix**.

Das Schwierige ist nicht die Installation – die ist ein einziger Befehl. Das Schwierige ist, dass große Anbieter wie **Gmail** (Google Mail) sehr genau prüfen, welche Post sie annehmen. Ein Server, der einfach „irgendwie" sendet, landet im Spam oder wird ganz abgewiesen. Dieses Kapitel zeigt Schritt für Schritt, wie man Postfix auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) so einrichtet, dass eine Gmail-Adresse die Post im Posteingang annimmt: mit verschlüsselten Verbindungen und einem Zertifikat von **Certbot** (siehe [Webserver](./webserver.md)) sowie den drei DNS-Einträgen SPF, DKIM und DMARC.

Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen. Platzhalter wie `meine-domain.de` und die Beispiel-IP `203.0.113.10` müssen überall durch die eigenen Werte ersetzt werden.

## Was Postfix macht

Man kann sich Postfix wie die Poststelle einer Firma vorstellen. Mitarbeiter (die Programme auf dem Server) legen ihre fertigen Briefe in ein Fach. Die Poststelle nimmt sie heraus, schaut auf die Anschrift, sucht den richtigen Zustellweg und übergibt den Brief nach draußen. Um die Zustellung im Haus kümmert sie sich getrennt davon.

Postfix kann in zwei Betriebsarten laufen:

- **Nur senden.** Postfix nimmt Post ausschließlich von Programmen auf demselben Server an und leitet sie nach außen weiter. Es gibt keine Postfächer, und von außen ist nichts erreichbar. Genau das brauchen MediaWiki und Drupal – und genau das richtet dieses Kapitel ein.
- **Vollständiger Mailserver.** Zusätzlich nimmt Postfix Post aus dem Internet an, legt sie in Postfächern ab, macht sie über IMAP abrufbar (mit dem Zusatzprogramm Dovecot), filtert Spam und so weiter. Das ist deutlich aufwendiger, ein eigenes Thema und hier nicht nötig.

## Warum Gmail so wählerisch ist

Der Grund ist Spam. Bei jeder eingehenden Nachricht prüft Gmail unter anderem:

- **Rückwärtsauflösung (PTR).** Hat die IP-Adresse des sendenden Servers einen Namen, und zeigt dieser Name auf dieselbe IP zurück? Post von IPs ohne passenden PTR-Eintrag wird fast immer abgewiesen.
- **SPF.** Darf dieser Server überhaupt Post für diese Domain verschicken? Das legt ein DNS-Eintrag fest.
- **DKIM.** Ist die Nachricht digital unterschrieben, und passt die Unterschrift zu einem Schlüssel, der im DNS veröffentlicht ist? Das beweist, dass die Nachricht unterwegs nicht verändert wurde und wirklich von der Domain stammt.
- **DMARC.** Ein DNS-Eintrag, der dem Empfänger sagt, was er tun soll, wenn SPF oder DKIM nicht passen – und der Berichte anfordert.
- **Verschlüsselung (TLS).** Wird die Verbindung zwischen den Servern verschlüsselt?
- **Ruf der IP-Adresse.** Kam von dieser IP schon einmal Spam?

Seit Februar 2024 verlangt Gmail auch von kleinen Absendern mindestens SPF **oder** DKIM; bei größeren Mengen SPF **und** DKIM **und** DMARC. Am einfachsten richtet man von Anfang an alles ein.

## Voraussetzungen

- **Eine eigene Domain** mit Zugang zu den DNS-Einträgen (meist beim Anbieter, bei dem die Domain registriert ist).
- **Ein Server mit fester öffentlicher IPv4-Adresse** – ein gewöhnlicher gemieteter Server genügt (siehe [Server mieten](./server-mieten.md)).
- **Ausgehender Port 25 offen.** Viele Anbieter sperren Port 25 zunächst, um Spam zu verhindern. Prüfen:

  ```bash
  nc -zv gmail-smtp-in.l.google.com 25
  ```

  Bleibt der Befehl hängen oder meldet „timed out", muss man den Anbieter über den Support bitten, Port 25 freizuschalten. Ohne offenen Port 25 verlässt keine einzige Nachricht den Server.
- **Die Möglichkeit, den PTR-Eintrag** für die Server-IP zu setzen – meist im Kundenbereich des Anbieters unter „Reverse DNS", „rDNS" oder „PTR".
- Eine frische IP ohne schlechte Vorgeschichte hilft. Prüfen lässt sich das unter `check.spamhaus.org`.

## Schritt 1: Den Hostnamen des Servers festlegen

Der Server braucht einen vollständigen Namen, mit dem er sich bei anderen Mailservern meldet. Üblich ist `mail.meine-domain.de`.

```bash
sudo hostnamectl set-hostname mail.meine-domain.de
```

Danach die Datei `/etc/hosts` ergänzen, damit der Name auch lokal auflösbar ist. Eine Zeile hinzufügen:

```
203.0.113.10  mail.meine-domain.de  mail
```

Kontrolle:

```bash
hostname --fqdn
# muss  mail.meine-domain.de  ausgeben
```

## Schritt 2: DNS-Einträge für den Mailnamen

Beim DNS-Anbieter für die Domain anlegen:

| Name/Host | Typ | Wert |
| --- | --- | --- |
| `mail` | A | `203.0.113.10` |
| `@` (die Domain selbst) | MX | `10 mail.meine-domain.de.` |

Der A-Eintrag lässt `mail.meine-domain.de` auf den Server zeigen. Der MX-Eintrag legt fest, welcher Server für Post an `@meine-domain.de` zuständig ist. Das ist nötig, damit Antworten und DMARC-Berichte irgendwo ankommen.

Da dieser Aufbau nur sendet und keine Post empfängt, sollte der MX-Eintrag **auf ein vorhandenes Postfach** zeigen – zum Beispiel das Postfach, das beim Domain-Anbieter mitgeliefert wird, oder ein anderes Mailkonto. Dorthin gehen dann Antworten und Berichte. Wer die Domain-Post selbst auf dem Server empfangen will, braucht den vollständigen Mailserver aus dem Abschnitt oben – ein eigenes Thema.

## Schritt 3: Rückwärtsauflösung (PTR) setzen

Im Kundenbereich des Server-Anbieters die Rückwärtsauflösung für `203.0.113.10` auf `mail.meine-domain.de` einstellen. Prüfen:

```bash
dig +short -x 203.0.113.10
# muss  mail.meine-domain.de.  ausgeben
```

Die Änderung kann einige Zeit dauern. Ein fehlender oder falscher PTR-Eintrag ist der häufigste Grund, warum Gmail Post abweist – diesen Schritt also nicht überspringen.

## Schritt 4: Postfix installieren

```bash
sudo apt update
sudo apt install postfix
```

Es erscheint ein blauer Einrichtungsdialog:

- **General type of mail configuration:** **Internet Site** auswählen.
- **System mail name:** **meine-domain.de** eintragen (ohne `mail.` davor). Das ist die Domain, die hinter dem `@` in der Absenderadresse lokaler Post steht.

Erscheint der Dialog nicht oder wurde etwas falsch gewählt, holt man ihn mit `sudo dpkg-reconfigure postfix` zurück.

Dann noch ein Hilfsprogramm für Testnachrichten installieren:

```bash
sudo apt install mailutils
```

`mailutils` stellt den Befehl `mail` bereit, mit dem man später eine Testmail verschickt.

## Schritt 5: Postfix als reinen Sendedienst einrichten

Die Haupteinstellungen stehen in `/etc/postfix/main.cf`. Statt die Datei von Hand zu öffnen, setzt man die Werte mit dem Befehl `postconf -e` – das vermeidet Tippfehler:

```bash
sudo postconf -e 'myhostname = mail.meine-domain.de'
sudo postconf -e 'myorigin = /etc/mailname'
sudo postconf -e 'mydestination = localhost'
sudo postconf -e 'inet_interfaces = loopback-only'
sudo postconf -e 'inet_protocols = ipv4'
```

Was die einzelnen Zeilen bedeuten:

- **`myhostname`** – der vollständige Name, mit dem Postfix sich meldet.
- **`myorigin = /etc/mailname`** – die Datei `/etc/mailname` enthält nach der Installation bereits `meine-domain.de`. Absenderadressen ohne Domain (etwa `www-data`) bekommen diesen Teil angehängt.
- **`mydestination = localhost`** – Postfix betrachtet nur `localhost` als „für mich bestimmt"; alles andere wird nach außen weitergeleitet.
- **`inet_interfaces = loopback-only`** – Postfix lauscht nur auf dem Server selbst (`127.0.0.1`), nie im Internet. Kein Fremder kann ihm Post übergeben. Damit ist ein Missbrauch als Spam-Verteiler ausgeschlossen.
- **`inet_protocols = ipv4`** – gesendet wird über IPv4. Nur wenn der Server eine funktionierende IPv6-Adresse **mit passendem PTR-Eintrag** hat, kann man hier `all` setzen. Ein fehlender IPv6-PTR führt zu Abweisungen durch Gmail, deshalb ist IPv4 die sichere Voreinstellung.

Kurz prüfen, dass die Datei mit dem Mailnamen stimmt:

```bash
cat /etc/mailname
# meine-domain.de
```

Danach Postfix neu starten:

```bash
sudo systemctl restart postfix
```

## Schritt 6: Absenderadressen sauber umschreiben

MediaWiki und Drupal setzen ihre Absenderadresse meist selbst sinnvoll. Systemnachrichten aber – etwa von zeitgesteuerten Aufgaben oder Fehlermeldungen – kommen von Adressen wie `root@mail.meine-domain.de` oder `www-data@mail.meine-domain.de`. Gmail mag Absender nicht, die es gar nicht gibt. Zwei Handgriffe helfen.

**Weiterleitung für `root`.** In `/etc/aliases` eine Zeile ergänzen:

```
root: postmaster@meine-domain.de
```

Danach:

```bash
sudo newaliases
```

**Adressen beim Versand umschreiben.** Eine neue Datei `/etc/postfix/generic` anlegen:

```
root@mail.meine-domain.de        postmaster@meine-domain.de
www-data@mail.meine-domain.de    noreply@meine-domain.de
@mail.meine-domain.de            noreply@meine-domain.de
```

Dann in ein von Postfix lesbares Format übersetzen und aktivieren:

```bash
sudo postmap /etc/postfix/generic
sudo postconf -e 'smtp_generic_maps = hash:/etc/postfix/generic'
sudo systemctl restart postfix
```

`smtp_generic_maps` schreibt Adressen nur auf dem Weg nach draußen um. So trägt jede Nachricht, die den Server verlässt, eine echte Adresse an der eigenen Domain. Die Adressen `postmaster@meine-domain.de` und `noreply@meine-domain.de` müssen bei dem in Schritt 2 gewählten Postfach tatsächlich existieren (`postmaster` wird ohnehin erwartet; `noreply` kann ein Sammelpostfach sein).

## Schritt 7: Verbindungen zu anderen Mailservern verschlüsseln

Beim **Senden** verschlüsselt Postfix in aktuellen Versionen bereits von sich aus, wann immer die Gegenseite es anbietet. Man setzt es trotzdem ausdrücklich:

```bash
sudo postconf -e 'smtp_tls_security_level = may'
sudo postconf -e 'smtp_tls_loglevel = 1'
```

`may` bedeutet: verschlüsseln, sobald die Gegenseite es kann – Gmail kann es immer. Ein **eigenes Zertifikat** braucht der Server dafür nicht, denn er stellt beim Senden keinen Ausweis vor.

Ein eigenes Zertifikat wird erst wichtig, sobald andere Server oder Mailprogramme sich **mit** dem Server verbinden – also wenn man später Post empfangen oder den Versand für entfernte Programme öffnen will. Wer das vorbereiten möchte, holt sich jetzt ein Zertifikat mit **Certbot** (Einrichtung siehe [Webserver](./webserver.md)):

```bash
sudo certbot certonly --nginx -d mail.meine-domain.de
```

Danach in Postfix eintragen:

```bash
sudo postconf -e 'smtpd_tls_cert_file = /etc/letsencrypt/live/mail.meine-domain.de/fullchain.pem'
sudo postconf -e 'smtpd_tls_key_file = /etc/letsencrypt/live/mail.meine-domain.de/privkey.pem'
sudo postconf -e 'smtpd_tls_security_level = may'
```

Damit Postfix ein erneuertes Zertifikat übernimmt, legt man einen kleinen Haken für Certbot an. Datei `/etc/letsencrypt/renewal-hooks/deploy/postfix-neuladen.sh`:

```sh
#!/bin/sh
systemctl reload postfix
```

Ausführbar machen:

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/postfix-neuladen.sh
```

## Schritt 8: SPF-Eintrag im DNS

Ein TXT-Eintrag an der Wurzel der Domain:

| Name/Host | Typ | Wert |
| --- | --- | --- |
| `@` (die Domain selbst) | TXT | `v=spf1 mx a ip4:203.0.113.10 -all` |

Das bedeutet: Post für diese Domain darf von den MX-Servern, vom A-Server und von dieser IP kommen; alles andere (`-all`) soll abgewiesen werden. Wer sich zu Beginn unsicher ist, nimmt statt `-all` zunächst `~all` (nur Verdacht statt Abweisung) und stellt nach erfolgreichen Tests auf `-all` um.

Es darf nur **einen** SPF-Eintrag pro Domain geben. Prüfen:

```bash
dig +short TXT meine-domain.de
```

## Schritt 9: DKIM mit OpenDKIM einrichten

DKIM unterschreibt jede ausgehende Nachricht digital. Das Programm dafür heißt **OpenDKIM**.

```bash
sudo apt install opendkim opendkim-tools
```

### Schlüssel erzeugen

```bash
sudo mkdir -p /etc/opendkim/keys/meine-domain.de
sudo opendkim-genkey -b 2048 -d meine-domain.de -D /etc/opendkim/keys/meine-domain.de -s default -v
sudo chown -R opendkim:opendkim /etc/opendkim
sudo chmod 600 /etc/opendkim/keys/meine-domain.de/default.private
```

Dabei entstehen zwei Dateien: `default.private` – der geheime Schlüssel, der auf dem Server bleibt – und `default.txt` – der öffentliche Schlüssel für das DNS. `default` ist der frei wählbare Name des Schlüssels (der „Selektor").

### OpenDKIM einstellen

In der Datei `/etc/opendkim.conf` folgende Zeilen setzen bzw. ergänzen (vorhandene gleichnamige Zeilen anpassen):

```
Domain                  meine-domain.de
Selector                default
KeyFile                 /etc/opendkim/keys/meine-domain.de/default.private
Socket                  local:/var/spool/postfix/opendkim/opendkim.sock
Mode                    sv
Canonicalization        relaxed/simple
OversignHeaders         From
SubDomains              no
UMask                   002
InternalHosts           refile:/etc/opendkim/TrustedHosts
ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
```

Falls in `/etc/default/opendkim` eine Zeile `SOCKET=` steht, wird sie mit `#` auskommentiert, damit die Angabe aus `opendkim.conf` gilt.

Die Datei `/etc/opendkim/TrustedHosts` anlegen mit dem Inhalt:

```
127.0.0.1
::1
localhost
mail.meine-domain.de
meine-domain.de
```

### Platz für den Verbindungspunkt schaffen

Postfix läuft in einem abgeschotteten Verzeichnis (`/var/spool/postfix`). Der Verbindungspunkt zu OpenDKIM muss darin liegen:

```bash
sudo mkdir -p /var/spool/postfix/opendkim
sudo chown opendkim:postfix /var/spool/postfix/opendkim
sudo chmod 750 /var/spool/postfix/opendkim
sudo adduser postfix opendkim
```

### Postfix mit OpenDKIM verbinden

```bash
sudo postconf -e 'milter_default_action = accept'
sudo postconf -e 'milter_protocol = 6'
sudo postconf -e 'smtpd_milters = local:opendkim/opendkim.sock'
sudo postconf -e 'non_smtpd_milters = local:opendkim/opendkim.sock'
```

Der Pfad ist ohne `/var/spool/postfix` davor angegeben, weil Postfix aus seinem abgeschotteten Verzeichnis heraus arbeitet. Entscheidend ist hier `non_smtpd_milters`: MediaWiki und Drupal übergeben ihre Post lokal, nicht über eine SMTP-Verbindung.

Dienste neu starten und für den Systemstart vormerken:

```bash
sudo systemctl restart opendkim
sudo systemctl enable opendkim
sudo systemctl restart postfix
```

### Öffentlichen Schlüssel im DNS veröffentlichen

```bash
sudo cat /etc/opendkim/keys/meine-domain.de/default.txt
```

Die Ausgabe sieht ungefähr so aus:

```
default._domainkey  IN  TXT  ( "v=DKIM1; h=sha256; k=rsa; "
  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA..." )
```

Daraus wird ein DNS-Eintrag:

| Name/Host | Typ | Wert |
| --- | --- | --- |
| `default._domainkey` | TXT | `v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A...` |

Der gesamte Inhalt ab `v=DKIM1` wird zu **einer** Zeichenkette zusammengesetzt – die Anführungszeichen und Klammern aus der Datei entfallen, der `p=`-Teil wird ohne Leerzeichen aneinandergehängt. Viele DNS-Oberflächen nehmen den langen Wert direkt an.

Nach ein paar Minuten prüfen:

```bash
sudo opendkim-testkey -d meine-domain.de -s default -vvv
# die Meldung  key OK  bedeutet: passt
```

## Schritt 10: DMARC-Eintrag im DNS

| Name/Host | Typ | Wert |
| --- | --- | --- |
| `_dmarc` | TXT | `v=DMARC1; p=none; rua=mailto:postmaster@meine-domain.de; fo=1` |

`p=none` heißt: zu Beginn nur beobachten, nichts abweisen. An die `rua`-Adresse schicken die Empfänger regelmäßig Berichte. Sind diese Berichte nach ein bis zwei Wochen sauber, verschärft man auf `p=quarantine` (Verdachtspost in den Spam-Ordner) und später auf `p=reject` (Abweisung).

## Schritt 11: Testen

Eine Testnachricht an eine eigene Gmail-Adresse senden:

```bash
echo "Dies ist ein Test von meinem eigenen Server." | mail -s "Postfix-Test" -a "From: noreply@meine-domain.de" meine-adresse@gmail.com
```

Dabei das Protokoll mitlesen:

```bash
sudo tail -f /var/log/mail.log
```

Gesucht wird eine Zeile mit `status=sent` und `250 2.0.0 OK`. (Gibt es die Datei `/var/log/mail.log` nicht, hilft `journalctl -u postfix -f`.)

In Gmail die Nachricht öffnen, oben rechts auf die drei Punkte, dann **Original anzeigen**. Dort sollte stehen:

```
SPF:    PASS
DKIM:   'PASS' with domain meine-domain.de
DMARC:  'PASS'
```

Zusätzlich die Nachricht durch **mail-tester.com** schicken: Die Seite zeigt eine Adresse an, an die man eine Mail sendet; nach dem Neuladen erscheint eine Bewertung. Ziel ist 10 von 10.

Landet die Post trotz dreier „PASS" im Spam, einmal „Kein Spam" wählen und einer neuen IP einige Tage Zeit geben, einen guten Ruf aufzubauen.

## Schritt 12: MediaWiki anbinden

MediaWiki nutzt von Haus aus die `mail()`-Funktion von PHP, die die Nachricht über `/usr/sbin/sendmail` an Postfix übergibt. Es muss also nichts zusätzlich installiert werden – nur Adressen eingetragen. In `LocalSettings.php`:

```php
$wgEnableEmail      = true;
$wgEnableUserEmail  = true;
$wgEmergencyContact = 'postmaster@meine-domain.de';
$wgPasswordSender   = 'noreply@meine-domain.de';
$wgNoReplyAddress   = 'noreply@meine-domain.de';
```

Die Einstellung `$wgSMTP` bleibt auf `false` (Voreinstellung), damit das lokale Postfix verwendet wird. Zum Prüfen die Seite `Spezial:E-Mail_senden` aufrufen oder ein Passwort zurücksetzen lassen.

## Schritt 13: Drupal anbinden

Auch Drupal verschickt im Kern über die `mail()`-Funktion von PHP und damit über Postfix – ein Zusatzmodul ist nicht nötig. Die Absenderadresse der Website wird unter **Verwaltung → Konfiguration → System → Basiseinstellungen** (`/admin/config/system/site-information`) auf `noreply@meine-domain.de` gesetzt.

Wer später mehr Einfluss auf das Aussehen der E-Mails braucht (etwa HTML-Nachrichten), kann das Zusatzmodul **Symfony Mailer** nachrüsten; es kann das lokale Postfix als „Sendmail"-Weg weiterverwenden. Für Anmelde- und Passwortmails genügt der eingebaute Weg.

Zum Prüfen ein Testkonto anlegen oder `/user/password` verwenden.

## Schritt 14: Laufender Betrieb

- **Protokoll ansehen:** `sudo tail -f /var/log/mail.log`. `deferred` heißt: vorübergehend hängen geblieben, Postfix versucht es erneut; `bounced` heißt: endgültig abgewiesen.
- **Warteschlange:** `mailq` zeigt wartende Post an, `sudo postqueue -f` stößt einen sofortigen neuen Versuch an.
- **Kein offenes Relay:** Durch `inet_interfaces = loopback-only` nimmt der Server Post nur von sich selbst an. Diese Einstellung nicht ohne Anmeldung und weitere Regeln auf `all` ändern.
- **Aktuell halten:** `sudo apt upgrade` versorgt Postfix und OpenDKIM mit Sicherheitsupdates.
- **Ruf der IP prüfen:** gelegentlich unter `check.spamhaus.org` und `mxtoolbox.com` nachsehen.
- **DMARC verschärfen:** nach sauberen Berichten `p=none` → `p=quarantine` → `p=reject`.
- **Zertifikat:** Wurde das Certbot-Zertifikat eingerichtet, bestätigt `sudo certbot renew --dry-run`, dass die Erneuerung funktioniert.

## Fehlersuche

| Symptom | Ursache | Lösung |
| --- | --- | --- |
| Post bleibt in der Warteschlange, `connect to ...:25: Connection timed out` | Anbieter sperrt ausgehenden Port 25 | beim Anbieter freischalten lassen |
| Gmail: `550 ... does not have a valid PTR record` | keine oder falsche Rückwärtsauflösung | PTR im Anbieter-Panel auf `mail.meine-domain.de` setzen |
| Gmail-Header `SPF: SOFTFAIL` oder `FAIL` | SPF-Eintrag fehlt oder IP nicht enthalten | TXT-Eintrag `v=spf1 ... ip4:203.0.113.10 -all` prüfen |
| Header `DKIM: FAIL` oder `NEUTRAL` | öffentlicher Schlüssel im DNS falsch, oder die Unterschrift greift nicht | `opendkim-testkey` ausführen; ist `non_smtpd_milters` gesetzt? Socket-Pfad richtig? |
| `warning: connect to Milter service local:opendkim/opendkim.sock: No such file or directory` | Verzeichnis für den Verbindungspunkt fehlt oder falsche Rechte | `/var/spool/postfix/opendkim` anlegen, `chown opendkim:postfix`, OpenDKIM neu starten |
| Post landet trotz „PASS" im Spam | Ruf der IP noch niedrig | „Kein Spam" wählen, abwarten, Menge langsam steigern |

## Für dieses Buch

Für den beschriebenen Aufbau ist ein Postfix, das nur sendet, genau das richtige Maß: MediaWiki und Drupal müssen Post verschicken, nicht empfangen. Der Server bleibt zum Internet hin geschlossen (`loopback-only`), womit das größte Risiko entfällt – der Missbrauch als Spam-Verteiler. Die Arbeit steckt nicht in Postfix selbst, sondern in den vier Nachweisen auf der DNS-Seite: Rückwärtsauflösung, SPF, DKIM und DMARC. Sind alle vier gesetzt, stellt Gmail in den Posteingang zu; fehlt einer, verschwindet die Post im Spam. Das Empfangen von Post (Postfächer, IMAP, Spam-Filter) ist eine eigene, größere Aufgabe; bis dahin zeigt der MX-Eintrag der Domain auf ein vorhandenes Postfach, damit Antworten und Berichte trotzdem ankommen.

## Fazit

**Postfix** ist das Programm, das die E-Mails von MediaWiki und Drupal annimmt und zustellt. Auf Ubuntu 26.04 ist es mit einem `apt`-Befehl installiert und mit wenigen `postconf`-Zeilen als reiner Sendedienst eingerichtet: `inet_interfaces = loopback-only` hält den Server dabei nach außen dicht. Damit **Gmail** die Post annimmt, zählen vier Dinge im DNS: ein passender **PTR**-Eintrag für die IP, ein **SPF**-Eintrag, eine **DKIM**-Unterschrift über **OpenDKIM** mit veröffentlichtem Schlüssel und ein **DMARC**-Eintrag. Die Verbindung zu anderen Mailservern verschlüsselt Postfix beim Senden von selbst; ein eigenes Zertifikat von **Certbot** wird erst beim Empfangen nötig. MediaWiki und Drupal brauchen keine Zusatzsoftware – sie geben ihre Post über die `mail()`-Funktion von PHP direkt an das lokale Postfix. Ein Test an eine Gmail-Adresse und ein Blick in „Original anzeigen" auf die Zeilen `SPF: PASS`, `DKIM: PASS` und `DMARC: PASS` bestätigen, dass alles greift.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
