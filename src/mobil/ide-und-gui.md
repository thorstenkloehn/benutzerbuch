# IDE und GUI

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Wer eine **App** für Mobilgeräte mit dem Betriebssystem **Android** schreiben will – also für die meisten Smartphones und Tablets außerhalb der Apple-Welt –, steht am Anfang vor zwei Fragen: In welchem Programm schreibt man den Code? Und wie baut man die sichtbare Oberfläche, die der Nutzer auf dem Bildschirm antippt? Dieses Kapitel beantwortet beide für die heute übliche Programmiersprache **Kotlin**.

Kotlin ist eine vergleichsweise junge Sprache der Firma JetBrains. Google hat sie 2017 offiziell für Android zugelassen und 2019 zur bevorzugten Sprache erklärt: Neue Anleitungen, Beispiele und Werkzeuge gehen seither von Kotlin aus, die ältere Sprache Java bleibt weiter möglich. Kotlin-Code läuft auf derselben Laufzeitumgebung wie Java (siehe [Programmiersprachen](../entwicklungs-rechner/programmiersprachen.md)), ist aber kürzer und fängt eine häufige Fehlerquelle – den Zugriff auf einen „leeren“ Wert – schon beim Übersetzen ab.

Als Grundlage dient wie im übrigen Buch **Ubuntu**. Ein **Terminal** ist ein Fenster, in das man Anweisungen als Text tippt; ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen, das System fragt dann nach dem Passwort.

## Android Studio – die IDE für Android

Eine **IDE** („Integrated Development Environment“, integrierte Entwicklungsumgebung) ist ein Programm, in dem Editor, Fehlersuche, Versionsverwaltung und Werkzeuge zum Ausführen von Code in einem Fenster zusammenstecken (siehe [IDE](../entwicklungs-rechner/ide.md)). Für Android gibt es dafür einen klaren Standard: **Android Studio**.

Android Studio wird von Google herausgegeben, ist kostenlos und läuft auf Windows, macOS und Linux. Es baut auf **IntelliJ IDEA** von JetBrains auf – der IDE aus dem Kapitel [IDE](../entwicklungs-rechner/ide.md) – und ergänzt sie um alles, was für Android nötig ist:

- das **Android SDK** („Software Development Kit“), also die fertigen Bausteine und Werkzeuge, aus denen eine Android-App gebaut wird;
- einen **Emulator**, ein am Bildschirm nachgebildetes Smartphone, auf dem sich die App ohne echtes Gerät ausprobieren lässt;
- eine **Vorschau** der Oberfläche, die Änderungen sofort anzeigt;
- das Bauwerkzeug **Gradle**, das den Weg vom Quellcode zur fertigen App-Datei steuert und Bausteine aus dem Internet nachlädt.

Kotlin ist von Haus aus eingebaut; eine gesonderte Installation der Sprache ist für die App-Entwicklung nicht nötig.

### Installation auf Ubuntu

Am einfachsten geht es über ein **Snap** – ein Paketformat von Canonical, bei dem das Programm stärker vom übrigen System abgeschottet läuft:

```bash
sudo snap install android-studio --classic
```

`--classic` hebt die Abschottung so weit auf, dass die IDE den vollen Zugriff auf das System bekommt, den sie zum Bauen und Ausführen braucht.

Wer die **Toolbox App** von JetBrains bereits nutzt (siehe [IDE](../entwicklungs-rechner/ide.md)), kann Android Studio auch darüber installieren und aktuell halten. Ein dritter Weg ist das Archiv von der offiziellen Seite `developer.android.com/studio`:

```bash
# Archiv nach /opt entpacken (dort liegen zusätzlich installierte Programme)
sudo tar -xzf android-studio-*.tar.gz -C /opt

# Einmalig starten; danach trägt sich Android Studio ins Anwendungsmenü ein
/opt/android-studio/bin/studio.sh
```

Beim ersten Start lädt ein Assistent das Android SDK herunter – mehrere Gigabyte – und richtet ein erstes Emulator-Gerät ein.

### Den Emulator beschleunigen

Der Emulator ist deutlich schneller, wenn er die Virtualisierungs-Unterstützung des Prozessors nutzt. Unter Linux läuft das über **KVM**. Ob der eigene Rechner das kann, zeigt:

```bash
sudo apt install cpu-checker
kvm-ok
```

Meldet der Befehl fehlende Rechte, wird der eigene Benutzer der Gruppe `kvm` hinzugefügt; danach einmal ab- und wieder anmelden:

```bash
sudo adduser "$USER" kvm
```

Alternativ testet man die App auf einem echten Telefon, das per USB-Kabel angeschlossen ist. Dazu müssen auf dem Telefon einmalig die „Entwickleroptionen“ und darin das „USB-Debugging“ eingeschaltet werden.

### Andere Editoren

**IntelliJ IDEA** kann mit dem Android-Zusatzmodul dasselbe wie Android Studio – kein Wunder, da Android Studio darauf aufbaut. Android Studio hat jedoch alles vorkonfiguriert und ist der Weg, den die Google-Anleitungen beschreiben.

**Visual Studio Code** (siehe [IDE](../entwicklungs-rechner/ide.md)) hat Erweiterungen für Kotlin und eignet sich für kleine Änderungen. Für die volle Android-Entwicklung fehlen aber Emulator, Oberflächen-Vorschau und die vorbereitete Verzahnung mit dem Android SDK. Als Hauptwerkzeug für Android ist es nicht zu empfehlen.

## GUI – die sichtbare Oberfläche bauen

**GUI** steht für „Graphical User Interface“, grafische Benutzeroberfläche: die Knöpfe, Textfelder, Listen und Bilder, die der Nutzer sieht und antippt. Auf Android gibt es dafür zwei Wege, die beide aus Kotlin heraus benutzbar sind.

### XML-Layouts mit Views – der klassische Weg

Bei diesem Weg wird die Oberfläche in **XML**-Dateien beschrieben. XML ist ein Textformat aus verschachtelten Marken (Tags); jedes Bedienelement – ein Knopf, ein Textfeld, ein Bild – ist eine solche Marke und heißt im Android-Sprachgebrauch **View**. Der Kotlin-Code füllt diese Elemente dann mit Inhalt und reagiert auf Berührungen.

Dieser Weg ist seit den Anfängen von Android da. Dadurch gibt es sehr viele Anleitungen und Beispiele, und jede Android-Version unterstützt ihn. Der Nachteil: Oberfläche und Ablauf-Logik liegen an zwei getrennten Stellen – in der XML-Datei und im Kotlin-Code –, und es fällt vergleichsweise viel gleichförmiger Verbindungscode an.

### Jetpack Compose – der heutige Standard

**Jetpack Compose** ist der neuere Ansatz von Google, seit 2021 einsatzreif. Hier gibt es keine XML-Dateien mehr: Die Oberfläche wird direkt in Kotlin geschrieben, als besondere Funktionen (im Fachjargon „composable functions“).

Compose arbeitet **beschreibend** (deklarativ). Statt Schritt für Schritt anzuweisen, wie sich die Anzeige ändern soll, beschreibt man nur, wie der Bildschirm bei einem bestimmten Zustand aussehen soll – etwa „Liste mit diesen fünf Einträgen“. Ändert sich der Zustand, zeichnet Compose die betroffenen Teile von selbst neu. Das ergibt spürbar weniger Code als der XML-Weg, und Android Studio zeigt eine Live-Vorschau direkt neben dem Editor. Für neue Apps empfiehlt Google heute Compose.

Beide Wege lassen sich mischen. In eine bestehende XML-App kann man einzelne Compose-Bereiche einsetzen und so nach und nach umstellen, ohne alles auf einmal neu zu schreiben.

### Material Design – die Gestaltungsbausteine

**Material Design** ist kein dritter Weg, sondern eine Ergänzung: Googles Gestaltungssammlung liefert fertige Elemente – Knöpfe, Karten, Dialoge, Menüs – in einem einheitlichen, aufeinander abgestimmten Aussehen. Es gibt sie sowohl für den XML-Weg als auch für Compose, sodass eine App nicht jedes Bedienelement selbst entwerfen muss.

### Compose Multiplatform – eine Oberfläche für mehrere Systeme

**Compose Multiplatform** ist eine Erweiterung von JetBrains, die Jetpack Compose über Android hinaus öffnet. Zusammen mit **Kotlin Multiplatform** – der Technik, mit der sich Kotlin-Code für mehrere Betriebssysteme übersetzen lässt – läuft derselbe in Kotlin geschriebene Oberflächencode auf Android, auf dem iPhone (iOS), auf dem Desktop (Windows, macOS, Linux) und im Browser. Die Oberfläche für iOS gilt seit Mai 2025 als stabil.

Das lohnt sich, wenn dieselbe App mehrere Plattformen bedienen soll und man sie nur einmal schreiben möchte. Für eine reine Android-App ist der Zusatzaufwand nicht nötig.

### Nicht Kotlin: Flutter und React Native

Zwei bekannte Baukästen für plattformübergreifende Apps arbeiten nicht mit Kotlin: **Flutter** von Google nutzt die Sprache **Dart**, **React Native** nutzt **JavaScript** beziehungsweise **TypeScript**. Sie sind nur eine Überlegung wert, wenn man sich nicht auf Kotlin festlegen will, und werden hier genannt, damit die Auswahl vollständig ist.

## Kurzvergleich

| Ansatz | Sprache | Oberfläche beschrieben in | Wann geeignet |
| --- | --- | --- | --- |
| XML-Layouts mit Views | Kotlin | XML-Dateien, getrennt vom Code | Arbeit an einer bestehenden App, die es schon so nutzt |
| Jetpack Compose | Kotlin | direkt in Kotlin, beschreibend | neue Android-Apps (Empfehlung von Google) |
| Compose Multiplatform | Kotlin | direkt in Kotlin, beschreibend | eine Oberfläche für Android, iOS, Desktop und Web zugleich |
| Flutter | Dart | direkt in Dart, beschreibend | plattformübergreifend, ohne Festlegung auf Kotlin |
| React Native | JavaScript / TypeScript | direkt im Code, beschreibend | plattformübergreifend, mit Web-Kenntnissen im Team |

## Für dieses Buch

Die Entwicklung einer eigenen Android-App ist nicht Kern dieses Handbuchs. Entsteht aber eine App als Ergänzung – etwa als mobiler Zugang zur Wissenssammlung (siehe [Wissenssystem](../web-stack/wissensystem.md)) –, dann ist der Weg klar: **Android Studio** als IDE, installiert über Snap, und **Jetpack Compose** für die Oberfläche. Den XML-Weg wählt man nur, wenn man an einer vorhandenen App mitarbeitet, die bereits darauf aufbaut. **Compose Multiplatform** kommt in Frage, sobald neben Android auch das iPhone aus einem gemeinsamen Code bedient werden soll.

## Fazit

Für Android-Apps mit **Kotlin** ist **Android Studio** die passende IDE: kostenlos, von Google, auf IntelliJ IDEA aufgebaut und mit Android SDK, Emulator und Bauwerkzeug fertig eingerichtet. Auf Ubuntu wird es am einfachsten über ein Snap installiert; der Emulator läuft flüssig, sobald **KVM** freigeschaltet ist. Für die sichtbare Oberfläche gibt es zwei Kotlin-Wege: den klassischen über **XML-Layouts mit Views** und den heutigen Standard **Jetpack Compose**, bei dem die Oberfläche beschreibend direkt in Kotlin entsteht. Für neue Apps ist Compose die erste Wahl; **Compose Multiplatform** dehnt denselben Code auf iOS, Desktop und Web aus. Baukästen wie **Flutter** oder **React Native** sind nur ohne Festlegung auf Kotlin eine Alternative.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
