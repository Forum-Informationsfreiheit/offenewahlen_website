---
layout: page
title: "Tutorial: Ergebnis-Balkendiagramm"
permalink: /tutorial/balkendiagramm/
comments: true
---

Das folgende Tutorial gibt eine Schritt-für-Schritt-Einführung in die Visualisierung von Wahlergebnissen mittels Javascript/d3.js. Ziel ist, ein [Balkendiagramm](http://bl.ocks.org/ginseng666/1b950542b04bdf91d55a846b633bb77b) zu erstellen. Dazu wird mit Javascript ein SVG angelegt, Daten werden in entsprechender Form in eine Variable gespeichert und visuell ausgegeben, sowie ein kleiner interaktiver Effekt ergänzt.

## Voraussetzungen

* eine grobe Vorstellung, wie eine HTML-Seite aussieht
* eine grobe Vorstellung, was eine Variable in Javascript ist und was man damit macht ([Präsentation Peter Grassberger]({{ site.staticurl }}pages/workshop-spektral/JS-Coden-101.pdf))

Eine Einführung in HTML findet sich z.B. [hier](http://de.html.net/tutorials/html/) oder [hier](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Introduction), Tutorials zu Javascript gibt es z.B. [hier](https://wiki.selfhtml.org/wiki/JavaScript/Tutorials/Einf%C3%BChrung) oder [hier](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide).

Vorab kann man auch einen Blick in dieses Tutorial werfen, das einige grundlegende Punkte zum Thema Visualisierung von Wahldaten anschneidet: [Tutorial offenewahlen.at](https://github.com/OKFNat/offenewahlen/blob/gh-pages/visualisierungen.md)


## Nützliches

Der Code kann in jedem Textprogramm geschrieben werden, ein etwas besserer Texteditor hat aber viele Vorteile, wie etwa die Nummerierung der Zeilen. Zwei sehr gute (freie) Programme:

* [Notepad++](https://notepad-plus-plus.org/)
* [Sublime](https://www.sublimetext.com/)

Wenn man Code schreibt, passieren zwangsläufig jede Menge Fehler, vom Tippfehler bis zur vergessenen Klammer. Damit man diese Fehler auch sehen kann, sollte man beim Testen die Javascript-Console im Browser offen haben. Mit F12 öffnet man in verschiedenen Browsern mehrere Entwicklungstools, unter denen sich auch die Console befindet.

![Screenshot Console]({{ site.staticurl }}pages/tutorial-balkendiagramm/console.jpg)

(Console in Firefox 49)

Ebenfalls nützlich ist der Inspector (Firefox)/Elements (Chrome)/Dom Inspector (Explorer): Er zeigt den gesamten Inhalt der Seite an. Wenn man also ein Element darstellen will, es aber nicht erscheint und auch kein Fehler in der Console ausgegeben wird, dann kann man es hier suchen (und findet vermutlich den Grund, warum es nicht dargestellt wird - etwa fehlende geschlossene `<>` oder fehlerhafte Koordinaten).

![Screenshot Inspector]({{ site.staticurl }}pages/tutorial-balkendiagramm/inspector.jpg)

(Inspector in Firefox 49)

Konkret zu d3.js und Balkendiagrammen gibt es sehr gute Einführungen:

* [Let's make a bar chart](https://bost.ocks.org/mike/bar/)
* [Making a bar chart](http://alignedleft.com/tutorials/d3/making-a-bar-chart)

d3.js steht mittlerweile bei Version 4, zahlreiche Beispiele und Umsetzungen im Internet verwenden aber noch Version 3. Für die hier vorgestellten Techniken ergeben sich keine Unterschiede, darüber hinaus gibt es aber zahlreiche [Veränderungen](https://github.com/d3/d3/blob/master/CHANGES.md).

## Das Grundgerüst

```javascript
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
<script src="https://d3js.org/d3.v4.min.js"></script>
</head>
<body>

<script>
</script>

</body>
</html>
```

Dieser Code umreißt die Struktur der Seite: Neben dem `<html>`-Tag am Anfang steht im `<head>` ein Titel und der Zeichensatz wird angegeben (`<meta charset="utf-8">` - siehe [hier](https://developer.mozilla.org/de/docs/Web/HTML/Element/meta#attr-charset)). Danach wird die d3.js-library eingebunden, damit später die entsprechenden Befehle verfügbar sind. Das `</head>`-Tag wird geschlossen, der `<body>` beginnt und enthält noch einen leeren `<script>`-Block. Hier wird der Javascript-Code eingefügt. Anschließend werden `</body>` und `</html>` wieder geschlossen.

Diesen Teil kann man standardmäßig als Ausgangsbasis für eine Seite verwenden.

## SVG
Visualisierungen mit d3.js erzeugen so genannte [SVG](https://developer.mozilla.org/en-US/docs/Web/SVG) oder Scalable Vector Graphics - das sind im Kern einfach geometrische Formen wie Rechtecke, Kreise, Linien usw., die in Kombination zu komplexen Formen zusammengesetzt werden können. Ein großer Vorteil eines SVG: Es kann verlustfrei skaliert werden, man kann es also beliebig vergrößern, ohne dass es unscharf oder "pixelig" wird.

Um ein SVG zu definieren benötigt man noch kein Javascript, das geht direkt im HTML:

```
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
</head>
<body>
<svg width="200" height="200"></svg>
</body>
</html>
```

Das Tag `<svg>` erzeugt eine - noch unsichtbare - Fläche in der Struktur unserer Seite, auf der wir jetzt z.B. ein Rechteck zeichnen können. Die Fläche ist 200 Pixel breit und 200 Pixel hoch. Um das Rechteck zu definieren brauchen wir vier Attribute:

* Breite
* Höhe
* x-Koordinate der linken oberen Ecke
* y-Koordinate der linken oberen Ecke

_Wichtig: Alle Positionsangaben auf dem Bildschirm, wie eben die x/y-Koordinate, werden am Computer immer von der linken oberen Ecke des Bildschirms aus berechnet (dort liegt der Punkt x=0, y=0). Von dort geht es nach links und nach unten._

Wir können nun das Rechteck einfach durch Angabe dieser Attribute zeichnen:

```xml
<svg width="200" height="200">
<rect x="0" y="0" width="50" height="80"></rect>
</svg>
```

Dieser Code erzeugt ein schwarzes Rechteck in der linken oberen Ecke des Bildschirms, das 50 Pixel breit und 80 Pixel hoch ist. Es gibt jede Menge weiterer Attribute, die verwendet werden können, um das Rechteck zu verändern, `fill="red"` färbt es etwa rot ein. Der Blick in die [Dokumentation](https://developer.mozilla.org/en-US/docs/Web/SVG) lohnt sich.

So gesehen kann man alles, was man für ein Balkendiagramm braucht, auch direkt in dieser SVG-Form schreiben. Javascript kann diesen statischen Code aber sehr viel variabler, flexibler und interaktiver machen und einem selbst einige Arbeit abnehmen.

## erste Schritte in Javascript
Wir löschen das SVG-Beispiel wieder und wenden uns Javascript zu. Als erstes ein Test, ob es im Browser auch funktioniert:

```javascript
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
<script src="https://d3js.org/d3.v4.min.js"></script>
</head>
<body>

<script>
console.log("Funktioniert");
</script>

</body>
</html>
```

Wenn wir nun die Console (mit F12) öffnen, dann sollte dort das Wort "Funktioniert" stehen. Ist das nicht der Fall (und es steht auch keine Fehlermeldung dort), dann ist Javascript im Browser blockiert (etwa durch Sicherheitseinstellungen) und muss erst aktiviert werden.

![Screenshot Console Funktioniert]({{ site.staticurl }}pages/tutorial-balkendiagramm/console_funktioniert.jpg)

Hilfreich können auch die Zahlenangaben rechts sein: Sie stehen für die Zeile und die Spalte, in der der Befehl steht, der die Ausgabe erzeugt. Ein Klick darauf zeigt den Code an.

Als nächstes die Vorbereitungen für die Grafik: Wie oben gezeigt brauchen wir zunächst ein SVG, um darauf zeichnen zu können, und dieses SVG braucht eine Breite und eine Höhe. Diese Werte kann man statisch setzen (auf z.B. 400 Pixel). Wenn es allerdings keinen Grund gibt, warum das SVG genau 400 Pixel breit sein soll, dann vergibt man damit viel Flexibilität. Besser ist es, die Werte nach dem verfügbaren Platz festzulegen:

```javascript
<script>
var width = window.innerWidth;
var height = window.innerHeight;
</script>
```

Der Befehl `window.innerWidth` gibt die Breite des Fensters als Zahl zurück (siehe [hier](https://developer.mozilla.org/en/docs/Web/API/window/innerWidth), `window.innerHeight` macht das Gleiche für die Höhe. Zum Testen kann man in der Console diese Befehle eingeben, dann erhält man die entsprechenden Werte.

![Screenshot Console innerWidth]({{ site.staticurl }}pages/tutorial-balkendiagramm/console_width.jpg)

Das SVG würde nun das gesamte Browserfenster ausfüllen. Das kann (browserabhängig) dazu führen, dass Scrollbalken erscheinen, da es minimal über den verfügbaren Platz hinausreicht. Daher reduzieren wir die Breite und Höhe auf jeweils 95% und erzeugen das SVG:

```javascript
<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);
</script>
```

Zunächst müssen wir festlegen, wo das SVG angehängt werden soll. Dafür nutzen wir den Befehl `d3.select`. Wir wählen damit den `<body>` aus und hängen mit `append` ein SVG an. Diesem geben wir die Breite `width` und die Höhe `height`. In umfangreicheren Projekten werden SVGs z.B. in ein `div` gezeichnet, da man sie damit etwa positionieren kann.

Wir speichern das SVG gleich in der Variablen `svg`, um danach einfacher darauf zugreifen zu können. Alternativ könnte man es später auch so aufrufen: `d3.select("body").select("svg")`

Der Inspector im Browser sollte das SVG-Element bereits anzeigen. Am Bildschirm sieht man (noch) nichts.

![Screenshot Inspector SVG]({{ site.staticurl }}pages/tutorial-balkendiagramm/inspector_svg.jpg)

Dieser Schritt zeigt, wie die Befehle in d3.js aufgebaut sind:
* man wählt ein Element aus
* anschließendund hängt man alle Beschreibungen und Befehle an, die auf das Element angewandt werden sollen
* diese Angaben werden mit einem . verbunden

Das kann in einer Zeile passieren, aber auch untereinander geschrieben werden, letzteres ist meistens übersichtlicher.

Schließlich zeichnen wir unser Rechteck:

```javascript
<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);

svg.append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 80)
  .style("fill", "red");

</script>
```

Die Angaben sehen etwas anders aus als oben, entsprechen ihnen aber:

* x und y bezeichnen den Ausgangspunkt des Rechtecks, seine linke obere Ecke
* width und height bestimmen Breite und Höhe
* fill bestimmt die Farbe

Hier wird eine weitere Unterscheidung sichtbar: In d3.js verwendet man `.attr` zur Festlegung von Attributen wie Position und Form, `.style` für die Festlegung der Art und Weise, wie das Element aussehen soll (teilweise funktionieren die Befehle mit der einen und der anderen Bezeichnung gleichermaßen, man sollte aber konsequent versuchen, den Style vom Rest zu trennen).

An dieser Stelle: Am meisten lernt man, wenn man Sachen einfach ausprobiert - also einfach einmal die Werte beim Rechteck verändern und schauen, was passiert, und die Console und den Inspector verwenden. Welchen Effekt hat es, wenn das Rechteck größer als das SVG ist? Was ist, wenn man es außerhalb positioniert?

## ein erstes Balkendiagramm
Nach diesen Grundlagen jetzt (endlich) zum Balkendiagramm. Zum Testen verwenden wir das Ergebnis des ersten Wahlgangs der Bundespräsidentenwahl 2016 vom [BMI](http://www.bmi.gv.at/cms/BMI_wahlen/bundespraes/bpw_2016/Ergebnis.aspx). Ein Balkendiagramm besteht banal aus mehreren Rechtecken, die entweder die Breite oder die Höhe dafür verwenden, Werte anzuzeigen. Das jeweils andere Attribut ist konstant.

Wir machen ein Säulendiagramm, verwenden also die Höhe, um einen Wert auszudrücken. Als Balkenbreite nehmen wir 50 Pixel und zeichnen testweise je ein Rechteck für die drei ersten KandidatInnen (alphabetisch sind das Irmgard Griss, Norbert Hofer und Rudolf Hundstorfer):

```javascript
<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);

svg.append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 18.94)
  .style("fill", "grey");

svg.append("rect")
  .attr("x", 55)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 35.05)
  .style("fill", "grey");

svg.append("rect")
  .attr("x", 110)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 11.28)
  .style("fill", "grey");
</script>
```

Damit die Balken nebeneinander stehen, verschieben wir sie jeweils um die Breite 50 plus 5 weiteren Pixeln, um einen kleinen Abstand zu haben. Beim Attribut `height` verwenden wir den Stimmenanteil der KandidatInnen.

![Screenshot Balken 1]({{ site.staticurl }}pages/tutorial-balkendiagramm/balken_1.jpg)

Das geht schon in die richtige Richtung, intuitiv würde man aber Balken von unten nach oben erwarten. Die Grafik steht auf dem Kopf, da - wie oben geschrieben - das Koordinatensystem oben links beginnt. Nachdem man keine negative Höhe zeichnen kann, müssen wir umdenken: Wir setzen die y-Koordinate (bezeichnet die linke obere Ecke) dorthin, wo die Säule aufhören soll, basierend auf einer gemeinsamen Grundlinie bei 100. Die Höhe bleibt gleich, dafür geben wir jedem Element eine eigene Farbe (für Farbnamen siehe [hier](https://en.wikipedia.org/wiki/Web_colors)):

```javascript
svg.append("rect")
  .attr("x", 0)
  .attr("y", 100 - 18.94)
  .attr("width", 50)
  .attr("height", 18.94)
  .style("fill", "purple");

svg.append("rect")
  .attr("x", 55)
  .attr("y", 100 - 35.05)
  .attr("width", 50)
  .attr("height", 35.05)
  .style("fill", "blue");

svg.append("rect")
  .attr("x", 110)
  .attr("y", 100 - 11.28)
  .attr("width", 50)
  .attr("height", 11.28)
  .style("fill", "red");
```

![Screenshot Balken 2]({{ site.staticurl }}pages/tutorial-balkendiagramm/balken_2.jpg)

## variable Größe
Nachdem wir schon die Größe des SVG variabel gemacht haben, liegt es nahe, dasselbe für die Größe des Diagramms zu machen. So können wir die Balkenbreite aus der verfügbaren Breite berechnen, ebenfalls legen wir einen fixen Abstand zwischen den Balken von 10 Pixeln fest:

```javascript
var abstand = 10;
var balken_breite = width / 6 - abstand;
```

Die Zahl 6 steht hier für die sechs KandidatInnen, die das Diagramm letzten Endes darstellen soll. Da wir Prozentangaben verwenden, können wir die Höhe direkt als Maximalhöhe verwenden - jeder Balken soll demnach X Prozent dieser Maximalhöhe einnehmen:
```javascript
var max_hoehe = height;
```

Fügen wir diese Code-Schnipsel in unser bisheriges Script ein, dann passen sich die Balken an den verfügbaren Platz an. Verändert man die Fenstergröße und lädt die Seite neu, dann hat sich die Balkengröße entsprechend verändert (über diesen Weg lassen sich später responsive Visualisierungen umsetzen). Zu beachten: Wir dividieren die Werte jeweils durch 100, um den Prozentwert zu erhalten.

```javascript
svg.append("rect")
  .attr("x", 0)
  .attr("y", max-hoehe - max_hoehe * 18.94 / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * 18.94 / 100)
  .style("fill", "purple");

svg.append("rect")
  .attr("x", balken_breite + abstand)
  .attr("y", max-hoehe - max_hoehe * 35.05 / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * 35.05 / 100)
  .style("fill", "blue");

svg.append("rect")
  .attr("x", balken_breite * 2 + abstand * 2)
  .attr("y", max_hoehe - max_hoehe * 11.28 / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * 11.28 / 100)
  .style("fill", "red");
```

## Automatisierung
Bisher haben wir, trotz einiger Berechnungen, die Balken praktisch von Hand gezeichnet. Wir haben drei Blöcke für drei Säulen, die großteils identisch sind und sich nur durch die Werte unterscheiden. Höchste Zeit also, Arbeit an den Computer zu übergeben.

Um das zu tun, müssen wir unsere Ergebnisse zuerst in einem anderen Variablen-Typ speichern, einem so genannten _Array_. Das ist im Grunde nichts anderen als eine Liste von Werten und sieht so aus:
```javascript
var ergebnis = [18.94, 35.05, 11.28, 11.12, 2.26, 21.34];
```

Die Werte stehen zwischen `[]` (nun alle sechs KandidatInnen). Arrays können nicht nur Zahlen, sondern beliebige Inhalte speichern, das wird später noch wichtig. Ein großer Vorteil von Arrays ist, dass man direkt den Wert an Position x ansprechen kann, und zwar so: `ergebnis[2]`

Gibt man das in die Console ein, dann erhält man den Wert an Position zwei, konkret 11.28. Achtung: Arrays nummerieren ihren Inhalt immer beginnend mit 0, nicht mit 1.

Um den Array gut nutzen zu können, brauchen wir noch eine so genannte for-Schleife oder for-loop (siehe [hier](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Statements/for)). Damit sagt man dem Programm, dass es einen Block Code hintereinander mehrmals ausführen soll. Angewandt auf unser Programm sieht das so aus:

```javascript
var ergebnis = [18.94, 35.05, 11.28, 11.12, 2.26, 21.34];
var farben = ["purple", "blue", "red", "black", "yellow", "green"];

for (var i = 0; i < ergebnis.length; i++)
  {
  svg.append("rect")
    .attr("x", balken_breite * i + abstand * i)
    .attr("y", max_hoehe - max_hoehe * ergebnis[i] / 100)
    .attr("width", balken_breite)
    .attr("height", max_hoehe * ergebnis[i] / 100)
    .style("fill", farben[i]);
  }
```

Hier passiert Folgendes: Wir definieren zunächst den Array mit den Ergebnissen, und auch einen zweiten Array mit Farben für die KandidatInnen. Die Farben sind gleich geordnet wie die Ergebnisse. Dann beginnt die for-Schleife: Wir legen zunächst eine Zählvariable `i` an und setzen sie auf 0 (die Anfangsposition im Array). Dann müssen wir der Schleife sagen, wie lange sie laufen soll - so lange, so lange ihr Wert kleiner als die Länge des Ergebnis-Arrays ist, sprich bis alle Werte abgearbeitet sind.

Schließlich muss man noch definieren, was mit der Zählvariablen in jedem Durchgang passieren soll: Wir wollen den Array Wert für Wert durcharbeiten, also erhöhen wir `i` um eins, dafür steht `i++` (alternativ könnte man schreiben: `i = i + 1`). Anschließend öffnet man eine geschwungene Klammer und ergänzt den Code, der wiederholt werden soll - und schließt die geschwungene Klammer wieder.

Der Code ist wiederum identisch mit den vorherigen Beispielen, wir ersetzen nur die zuvor statischen Werte durch variable Angaben: `balkenbreite * i` multipliziert die Variable balken_breite mit dem Wert der Zählvariablen, um jeden Balken weiter nach rechts zu rücken. Beim ersten Durchlauf ist `i` 0, x ist also insgesamt auch 0, beim zweiten 1, x also 60 usw.

Bei `y` ersetzen wir den fixen Wert durch den Wert im Array, der an Stelle `i` steht, das gleiche machen wir bei der Höhe. Abschließend holen wir uns noch die Farbe aus dem Farben-Array, wiederum über die Position.

![Screenshot Balken 3]({{ site.staticurl }}pages/tutorial-balkendiagramm/balken_3.jpg)

Mit der Schleife haben wir den Code erheblich verkürzt. Zusätzlich brauchen wir nur die Werte im Array ändern, um andere Ergebnisse abzubilden, unser Diagramm passt sich automatisch an die Zahl der Werte an. Um das zu perfektionieren passen wir noch die Berechnung der Balkenbreite an:

```javascript
var balken_breite = width / ergebnis.length - abstand;
```

Achtung, diese Definition muss im Code unterhalb des Ergebnis-Arrays stehen, da Javascript sonst eine Variable sucht, die noch nicht definiert ist. Jetzt wäre es Zeit, ein wenig zu experimentieren, die Werte zu ändern, weitere einzugeben oder vorhandene zu löschen und zu testen, was dann passiert. Auch die Farben kann man ändern, es gibt sehr viel schönere Farben als die archetypischen rot/blau/grün-Töne, z.B. [hier](http://tristen.ca/hcl-picker/#/hlc/6/1/15534C/E2E062).

Man kann auch probieren, was passiert, wenn man die Zeile `.style("fill", farben[i]);` durch folgende drei Zeilen ersetzt:

```javascript
    .style("fill", "none")
    .style("stroke", farben[i])
    .style("stroke-width", "5px");
```

## Objects
Das Schöne (oder Schreckliche) am Programmieren ist, dass man praktisch immer Sachen verbessern oder ausbauen kann. Wir haben zwar schon einiges erreicht, aber ein paar Sachen sind noch unbefriedigend. So sind die Balken etwa nicht nach Größe sortiert, was bei einer Ergebnis-Visualisierung aber sinnvoll wäre. Das kann man händisch machen, muss dann aber auch die Farben händisch anpassen. Es wäre also gut, wenn wir das Ergebnis und die dazugehörige Farbe verbinden könnten.

Das lässt sich in Javascript über ein [Object](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Object) erreichen. Sehr vereinfacht ist das ein, naja, Objekt mit beliebigen Eigenschaften, z.B.:

```javascript
var mensch = {"name": "Sepp", "alter":17, "hobby":"Daten visualisieren"};
```

Auf diese Weise haben wir ein Object `mensch` definiert. Dieses Object hat den Namen "Sepp" und das Alter 17. Man kann alle einzelnen Eigenschaften ansprechen, indem man Object.Eigenschaft schreibt - oder konkret `mensch.name`, `mensch.alter` oder `mensch.hobby`. Auch das kann man testweise in die Console eingeben.

Ein Object besteht immer aus einem oder mehreren key/value-Paaren, die durch Beistrich voneinander getrennt sind: Unter Anführungszeichen steht zunächst ein key, hier eben z.B. `"name"`, dann folgt ein `:` und dann der value `"Sepp"`. Anders als ein array stehen die Inhalte eines objects zwischen `{}`.

Zu beachten: Keys brauchen immer Anführungszeichen, Werte je nach ihrem Typ. Strings, also Texte, stehen unter Anführungszeichen, Zahlen aber nicht. Man kann auch Variable in einem Object speichern, Arrays, Formeln, Funktionen, weitere Objects usw.

Für unser Beispiel brauchen wir ein Object, dass das Ergebnis und die Farbe des/der KandidatIn enthält - und weils leicht geht, nehmen wir den Namen auch noch mit und schreiben die Variable ergebnis um:

```javascript
var ergebnis = {"name":"Griss", "wert":18.94, "farbe":"#91678A"};
```

Wir benötigen insgesamt sechs solcher Objects als Liste, um sie anschließend in unsere Schleife schicken zu können: Also stellen wir einen Array zusammen, der diese Objects enthält:

```javascript
var ergebnis = [
  {"name":"Griss", "wert":18.94, "farbe":"#91678A"},
  {"name":"Hofer", "wert":35.05, "farbe":"#356F7F"},
  {"name":"Hundstorfer", "wert":11.28, "farbe":"#B7615A"},
  {"name":"Khol", "wert":11.12, "farbe":"#000000"},
  {"name":"Lugner", "wert":2.26, "farbe":"#E2E062"},
  {"name":"Van der Bellen", "wert":21.34, "farbe":"#437C4F"}
  ];
```

Dann passen wir den Code in der for-Schleife an - da unser Array jetzt keine einfachen Zahlen mehr enthält, sondern eben mehrere Objects, müssen wir die jeweils benötigte Eigenschaft abrufen:

```javascript
svg.append("rect")
  .attr("x", balken_breite * i + abstand * i)
  .attr("y", max_hoehe - max_hoehe * ergebnis[i].wert / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * ergebnis[i].wert / 100)
  .style("fill", ergebnis[i].farbe);
```

Weiterhin wird die Balkenbreite und die Zahl der Balken automatisch berechnet, wenn man Einträge löscht oder ergänzt verändert sich die Darstellung. Bei den Farben verwenden wir testweise den [Hex-Wert](http://www.w3schools.com/html/html_colors.asp).

Zurück zu dem, warum wir überhaupt ein Object gebraucht haben: Wir wollen die Balken nach Größe sortieren. Das lässt sich einfach mit Javascript lösen, wir tragen nach dem Object (und vor der Schleife) folgenden Code ein:

```javascript
ergebnis.sort(function(a, b) { return a.wert < b.wert; });
```

Dieser Code geht durch den Array, vergleicht zwei Werte - a und b - und sortiert sie entsprechend um. Das `<` führt zu einer absteigenden Sortierung - was `>` macht sollte man ausprobieren. Wichtig: Wir können wieder nicht direkt die Werte im Array abrufen, sondern wir rufen jeweils einen Eintrag im Array auf, und dann die entsprechende Eigenschaft. So weit, so gut:

![Screenshot Balken 4]({{ site.staticurl }}pages/tutorial-balkendiagramm/balken_4.jpg)

## Daten mit d3.js abbilden...
Nachdem wir soweit gekommen sind, können wir uns kurz den Möglichkeiten von [d3.js](http://www.d3js.org) widmen. Diese library ist explizit dafür gedacht, Daten zu visualisieren und bietet sehr viel mehr als die bisher genutzten Befehle. Besonders praktisch ist, Daten mit erzeugten Elementen zu verbinden. Auf diesem Weg ist es z.B. möglich, Darstellungen dynamisch zu ändern, ohne alle Inhalte löschen und neu zeichnen zu müssen - die [Umsortierung der Balken der Wahlmotive](http://strategieanalysen.at/wahlen/bp2016/) basiert beispielsweise darauf. Für eine kurze Einführung sollte man die [Introduction](https://d3js.org/) lesen.

Um die Technik zu nutzen müssen wir ein paar Kleinigkeiten ändern. Unser Object bleibt gleich, wir löschen aber unsere gesamte for-Schleife und schreiben stattdessen:

```javascript
svg.selectAll("rect")
  .data(ergebnis)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return balken_breite * i + abstand * i; })
  .attr("y", function(d) { return max_hoehe - max_hoehe * d.wert / 100; })
  .attr("width", balken_breite)
  .attr("height", function(d) { return max_hoehe * d.wert / 100; })
  .style("fill", function(d) { return d.farbe; })
```

Das führt zu folgendem Ablauf: Mit `selectAll` schauen wir, ob schon Rechtecke vorhanden sind und wählen diese allenfalls aus. Dann laden wir mit `data(ergebnis)` unseren array, mit `enter()` werden die Daten dann quasi ausgerollt. Im Hintergrund wird eine Art for-Schleife ausgeführt und für jeden Eintrag im Array ein Rechteck erzeugt. Wenn schon Rechtecke vorhanden waren, dann werden zuerst diese mit den neuen Daten überschrieben. Merke: `data()` benötigt immer einen Array.

Bei der Festlegung der Attribute verwenden wir jetzt folgenden Code: `function(d, i) { }`. Das `d` steht für den jeweiligen Eintrag (gleichbedeutend mit dem `ergebnis[i]` von oben), `i` ist wieder eine Zählvariable. In dieser Logik werden alle Eigenschaften abgerufen und ausgegeben. Achtung: `d` kann immer ohne `i` stehen, `i` aber nie ohne `d`.

## ...und was das bringt
Das Ergebnis scheint gleich zu sein wie die Visualisierung via for-Schleife. Der Vorteil ist aber, dass die Daten aus unserem Object jetzt an die gezeichneten Elemente gebunden und leicht abrufbar sind. Als Beispiel dafür fügen wir etwas Interaktivität hinzu, und zwar soll das Ergebnis über dem Balken angezeigt werden, wenn man mit der Maus darüber fährt. Dazu erweitern wir den obigen Code um folgende Zeilen:

```javascript
svg.selectAll("rect")
  .data(ergebnis)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return balken_breite * i + abstand * i; })
  .attr("y", function(d) { return max_hoehe - max_hoehe * d.wert / 100; })
  .attr("width", balken_breite)
  .attr("height", function(d) { return max_hoehe * d.wert / 100; })
  .style("fill", function(d) { return d.farbe; })
  .on("mouseover", function(d, i) {
     svg.append("text")
       .attr("x", balken_breite * i + abstand * i)
       .attr("y", max_hoehe - max_hoehe * d.wert / 100)
       .style("fill", d.farbe)
       .text(d.wert);
     });
```

Der Ausdruck `.on("mouseover", function() {})` bedeutet genau das, was er sagt - nämlich, dass etwas passieren soll, wenn der Mauszeiger über das Element fährt. In unserem Fall erzeugen wir nach dem bereits bekannten Muster einen Text. Ein Text in SVG benötigt nur zwei Attribute, nämlich

* x - die Anfangsposition horizontal
* y - die Anfangspoition vertikal

Nachdem wir den Text beim Balken erscheinen lassen wollen, können wir zur Positionierung die gleichen Formeln verwenden wie für den Balken selbst. Der Text erhält noch eine Farbe, bevor mit `.text(d.wert)` der eigentliche Inhalt geschrieben wird.

Praktisch ist nun eben, dass wir den Wert nicht erst aus unserem array heraussuchen und zum passenden Balken zuordnen müssen. Wir können  stattdessen die Daten mit dem Ausdruck `function(d, i)` abrufen, wobei `d` wieder für den jeweiligen Eintrag im array steht, und `i` die Zählvariable ist.

Ein kurzer Test sollte zeigen, dass der `mouseover`-Effekt funktioniert (hoffentlich), das Ergebnis aber noch unbefriedigend ist. Erstens ist die Position des Textes am Anfang des Balkens unpassend, zweitens kann man nur einmal über den Balken fahren, da der Text dann permanent angezeigt wird.

Feilen wir zunächst am Aussehen des Textes. Um ihn über dem Balken zu zentrieren, müssen wir zur x-Position die halbe Balkenbreite addieren:

```javascript
svg.append("text")
  .attr("x", balken_breite * i + abstand * i + balken_breite / 2)
  .attr("y", max_hoehe - max_hoehe * d.wert / 100)
  .style("fill", d.farbe)
  .text(d.wert);
```

Das Ergebnis sieht etwas besser aus, allerdings ist der Text leicht nach rechts verschoben. Das liegt daran, dass wir mit `x` die Anfangsposition festlegen, also den Punkt, an dem der Text beginnt. Wir möchten aber, dass dieser Punkt in der Mitte des Textes liegt, was mit der Eigenschaft [`text-anchor`](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/text-anchor) erreicht werden kann:

```javascript
  .style("text-anchor", "middle")
```

Wenn wir schon dabei sind, dann machen wir den Text noch etwas größer und lassen einen kleinen Abstand zum Balken selbst:

```javascript
  .attr("y", max_hoehe - max_hoehe * d.wert / 100 - 5)

  .style("font-size", "36px")
```

Da das Koordinatensystem oben links beginnt, müssen wir ein paar Pixel abziehen, um den Text nach oben zu schieben.

Um den `mouseover`-Effekt zu komplettieren, müssen wir abschließend festlegen, was mit dem Text passieren soll, sobald die Maus den Balken wieder verlässt. In unserem Fall soll der Text gelöscht werden.

```javascript
svg.selectAll("rect")
  .data(ergebnis)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return balken_breite * i + abstand * i; })
  .attr("y", function(d) { return max_hoehe - max_hoehe * d.wert / 100; })
  .attr("width", balken_breite)
  .attr("height", function(d) { return max_hoehe * d.wert / 100; })
  .style("fill", function(d) { return d.farbe; })
  .on("mouseover", function(d, i) {
     svg.append("text")
       .attr("x", balken_breite * i + abstand * i + balken_breite / 2)
       .attr("y", max_hoehe - max_hoehe * d.wert / 100 - 5)
       .style("fill", d.farbe)
       .style("text-anchor", "middle")
       .style("font-size", "36px")
       .text(d.wert);
     })
   .on("mouseout", function() {
     svg.selectAll("text").remove();
   });
```

Mit `mouseout` rufen wir analog zu `mouseover` wieder eine Funktion auf. Diese benötigt diesmal weder `d` noch `i`, sie wählt einfach alle Text-Elemente auf dem SVG an und entfernt sie mittels `remove()`. Das `selectAll` ist dabei konsequent: Sollten noch andere Textteile irgendwo im SVG stehen, dann werden auch diese gelöscht.

![Screenshot Balken 5]({{ site.staticurl }}pages/tutorial-balkendiagramm/balken_5.jpg)

## Animation
Als letzten Punkt in diesem Tutorial basteln wir noch eine Einstiegsanimation, die die Balken wachsen lässt. Dazu kann man sich zunächst theoretisch überlegen, was passieren soll, damit es diesen Effekt gibt: Als Ausgangspunkt bräuchten wir einen Balken, der die Höhe `0` hat, als Endpunkt dann den Balken mit der Höhe `d.wert`. Das können wir probieren (der `mouseover`-Effekt ist zur Übersichtlichkeit gelöscht):

```javascript
svg.selectAll("rect")
  .data(ergebnis)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return balken_breite * i + abstand * i; })
  .attr("y", function(d) { return max_hoehe - max_hoehe * d.wert / 100; })
  .attr("width", balken_breite)
  .attr("height", 0)
  .style("fill", function(d) { return d.farbe; });  
```

Dieser Code führt zu einem leeren Bildschirm, da alle Balken die Höhe `0` haben. Im Inspector sollten sie aber sichtbar sein. Nach dem Ausgangspunkt brauchen wir den Endpunkt, den wir mit folgendem Code erzeugen:

```javascript
svg.selectAll("rect")  
  .attr("height", function(d) { return max_hoehe * d.wert / 100; });
```

Das Ergebnis sieht gleich aus wie zuvor, der Weg dorthin war aber etwas anders: Wir haben zuerst die Balken wie schon bekannt gezeichnet, allerdings ohne Höhe. Dann haben wir mit `svg.selectAll("rect")` nochmals alle Balken ausgewählt und nur ihre Höhe geändert. Dass dabei jeder Balken die richtige Höhe erhält, ist den Daten zu verdanken, die wir mit `data()` daran gebunden haben.

Was jetzt noch fehlt ist die Animation, die wir mit zwei kurzen Zeilen erzeugen:

```javascript
svg.selectAll("rect")
  .transition()
  .duration(1000)
  .attr("height", function(d) { return max_hoehe * d.wert / 100; });
```

`transition()` sagt dem Programm, dass es einen Übergang zwischen zwei Positionen herstellen soll - bei uns zwischen Höhe `0` und Höhe `d.wert`. Der Ausdruck `duration(1000)` legt in Millisekunden fest, auf welchen Zeitraum dieser Prozess aufgeteilt werden soll. 1000 Millisekunden entsprechen 1 Sekunde.

Soweit sich keine Tipp- oder sonstigen Fehler eingeschlichen haben sollten die Balken erscheinen - allerdings in einer verdrehten Version, sie fließen von oben nach unten. Programmatisch ist das richtig, wenn wir uns erinnern, dass wir die Balken - aufgrund des Koordinatensystems - genau so gezeichnet haben: Die y-Position ist das obere Ende, von dem aus wir den Balken in der Höhe des Werts nach unten zeichnen.

Passend sieht es trotzdem nicht aus, also drehen wir den Spieß um. Neben der Höhe verändern wir jetzt auch `y`. Ausgangspunkt ist nun der Fuß des Balkens (`max_hoehe`), von dem aus wir `y` nach oben verschieben, während wir gleichzeitig die Höhe von `0` auf `d.wert` setzen:

```javascript
svg.selectAll("rect")
  .data(ergebnis)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return balken_breite * i + abstand * i; })
  .attr("y", max_hoehe)
  .attr("width", balken_breite)
  .attr("height", 0)
  .style("fill", function(d) { return d.farbe; });  

svg.selectAll("rect")
  .transition()
  .duration(1000)
  .attr("y", function(d) { return max_hoehe - max_hoehe * d.wert / 100; })
  .attr("height", function(d) { return max_hoehe * d.wert / 100; });
```

Damit sollte nun alles wie gewünscht funktionieren und der Weg zum Ausprobieren und Experimentieren ist frei. Eine Kleinigkeit ergänzen wir aber dennoch, und zwar geben wir der `transition()` mit dem Befehl `delay(function(d, i) { return 250 * i; })` eine Staffelung beim Zeichnen der Balken mit. Was dabei passiert, sollte mittlerweile nachvollziehbar sein: `delay()` an sich verzögert die Ausführung einer `transition()` um x Millisekunden. Indem wir wieder die Zählvariable `i` dazunehmen, verzögert sich jeder Balken um 250 Millisekunden länger. Hier nun der finale Code:

```javascript
<script>
var ergebnis = [
  {"name":"Griss", "wert":18.94, "farbe":"#91678A"},
  {"name":"Hofer", "wert":35.05, "farbe":"#356F7F"},
  {"name":"Hundstorfer", "wert":11.28, "farbe":"#B7615A"},
  {"name":"Khol", "wert":11.12, "farbe":"#000000"},
  {"name":"Lugner", "wert":2.26, "farbe":"#E2E062"},
  {"name":"Van der Bellen", "wert":21.34, "farbe":"#437C4F"},
  ];

ergebnis.sort(function(a, b) { return a.wert < b.wert; });

var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var abstand = 10;
var balken_breite = width / ergebnis.length - abstand;
var max_hoehe = height;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);

svg.selectAll("rect")
  .data(ergebnis)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return balken_breite * i + abstand * i; })
  .attr("y", max_hoehe)
  .attr("width", balken_breite)
  .attr("height", 0)
  .style("fill", function(d) { return d.farbe; })
  .on("mouseover", function(d, i) {
     svg.append("text")
       .attr("x", balken_breite * i + abstand * i + balken_breite / 2)
       .attr("y", max_hoehe - max_hoehe * d.wert / 100 - 5)
       .style("fill", d.farbe)
       .style("text-anchor", "middle")
       .style("font-size", "36px")
       .text(d.wert);
     })
  .on("mouseout", function() {
     svg.selectAll("text").remove();
   });

svg.selectAll("rect")
  .transition()
  .delay(function(d, i) { return 250 * i; })
  .duration(1000)
  .attr("y", function(d) { return max_hoehe - max_hoehe * d.wert / 100; })
  .attr("height", function(d) { return max_hoehe * d.wert / 100; });

</script>
```

Als nächste Schritte kann man sich die unzähligen Beispiele auf [d3js.org](http://www.d3js.org) ansehen, vor allem jene zu den basic charts wie [bar chart](http://bl.ocks.org/mbostock/3885304), [line chart](http://bl.ocks.org/mbostock/3883245) oder auch [stacked bars](http://bl.ocks.org/mbostock/3886208).

Die nächste logische Stufe in d3.js selber ist die Verwendung von [Skalen](https://www.dashingd3js.com/d3js-scales), um Daten besser ins Optische übertragen zu können.
