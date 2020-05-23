# Typescript
Dieses Dokument soll die Grundlagen von Typescript darstellen und ihre Anwendungsfälle.

Kurz was zu meiner Person: Ich bin seit 5 Jahren (2015) PHP Entwickler. Eine Programmiersprache die sich auch lange Zeit mit Typisierung schwer getan hat. Aber ich habe auch Erfahrungen mit typisierten Sprachen wie C++ und Java und kann sagen, typensichere Programmierung hat mehr Vorteile als Nachteile.

Vorteile sind Typensicherheit und damit einhergehend bessere Auto-Vervollständigung. Was genau das heißt darauf kommen wir später noch mal zu sprechen in Kapitel 3. Offensichtliche Nachteile sind zum einen die Tatsache dass es nicht von Javascript nativ unterstützt wird und deswegen mehr Aufwand in der Entwicklung ist das Projekt korrekt aufzusetzen. Und zum anderen kann es manchmal ziemlich anstrengend sein Typen-korrekt zu arbeiten. Aber auch dafür hat Typescript eine Lösung `any`. Mehr dazu in Kapitel 2.

## Kapitel 0: Die Motivation
Javascript ist eine sehr populäre Sprache für den Bau von Webseiten geworden. Ein treibender Faktor sind sicherlich die reiche Auswahl an Frameworks wie Angular, React, View und noch viele mehr. Aber auch eine Website die ohne Framework auskommen kann braucht für zum Beispiel interaktives Formular validieren Javascript.

Das Problem daran ist, dass Javascript eine ziemlich anstrengende Sprache sein kann wenn man mit ihr versucht Sachen zu debuggen. Hier mal ein Beispiel:

```javascript
// Angenommen `element` ist ein Element auf der Website
element.style.height = document.height / 2 + "px";
```

Das Element wird nicht auf die Hälfte der Seite vergrößert. Bei genauerem hinschauen wird man feststellen dass die Höhe des Elements `NaNpx` ist aber warum? Weil `document.height` nicht definiert ist. Schaut man sich an was `document.height` für einen Wert hat sieht man `undefined`. Was ist das? Das heißt an dem Objekt `window` ist keine Eigenschaft `height` definiert. Aber damit weiter zu arbeiten ist in Javascript kein Problem. `undefined / 2` ist dann `NaN` (also keine Zahl). 

Also um das noch einmal zusammen zu fassen: Der Ausdruck `document.height / 2 + "px"` macht aus einer `undefined` Eigenschaft wenn man durch eine Zahl teilt ein Wert der keine Zahl ist und schlussendlich kommt doch wieder ein String raus weil alles + einen String ist ein String. Aber was wir eigentlich wollten war die Höhe des Dokuments durch zwei als Pixelwert. Mit Typescript wäre das nicht passiert.

## Kapitel 1: Von Javascript zu Typescript
Schauen wir uns mal an wie wir solche Probleme bei unseren eigenen Funktionen verhindern können:
Angenommen wir haben eine Funktion `add` die zwei Zahlen addieren soll:
```javascript
function add(x, y) {
    return x + y;
}

add(10, 20);
add(1, '1');
```

Wie können wir sicher stellen, dass das nur das tut was wir wollen? Der zweite Aufruf ist nicht, gültig. Man kann das in dem Falle ziemlich einfach verhindern in dem man sagt: Konvertiere in der Funktion alles was keine Zahl ist in eine Zahl. Das macht aber das Problem nur noch komplizierter, denn dann müssen wir uns auch noch mit dem Fall beschäftigen, was passiert wenn eine oder beide der Parameter nicht in eine Zahl umwandelbar sind. Eine einfache Antwort dazu wäre dann gibt die Funktion `NaN` zurück. Aber das wollen wir nicht. Was wir wollen ist das:
```javascript
function add(x, y) {
    if (typeof(x) === 'number' && typeof(y) === 'number) {
        return x + y;
    }
}
```
Wir wollen in der Funktion nur eine Addition durchführen, wenn sowohl `x` und `y` Zahlen sind. Zufrieden? 

**Nein**, damit haben wir zwei neue Probleme erzeugt:
1. Wenn `x` oder `y` keine Zahl ist, gibt diese Funktion `undefined` zurück. (Damit kann man auch nicht weiter rechnen.)
2. Von außen können wir nicht sehen wie Funktion benutzt werden soll. Wir sehen nur zwei Parameter. Erst in der Funktion wird klar, dass es Zahlen sein müssen.

> Kleiner mathematischer Einschub: Viele Konzepte aus der Programmierung sind aus der Mathematik entlehnt bzw. sehr ähnlich. In der Mathematik ist eine Funktion so definiert: Eine Funktion f macht aus Werten in der Definitionsmenge Werte in der Wertemenge (auch Zielmenge). Also ein Beispiel die Funktion f: x + y würde aus zwei Natürlichen Zahlen x und y eine neue Natürliche Zahl machen. Und genau dieses Typensystem: Zahl + Zahl = Zahl wollen wir jetzt auch in Javascript haben. 
> Einschub Ende.

Um unsere Funktion also nach dieser Definition anzupassen sieht sie so aus.
```javascript
function add(x, y) {
    if (typeof(x) === 'number' && typeof(y) === 'number) {
        return x + y;
    }
    throw new Error('Parameters need to be numbers');
}
```
Aber allgemein beleibt dann immer noch Problem Nummer 2: Wir können nicht von außen sehen wie die Funktion genutzt werden soll. Und mit mehr Argumenten die auch unterschiedliche Typen haben, wird dieser Ansatz schnell unübersichtlich und unpraktisch.

## Kapitel 2: Erste Schritte mit Typescript
Für die Leute, die ein bisschen mit Typescript experimentieren wollen: [Hier ist die Sandbox](https://www.typescriptlang.org/play/)

Wir werden in diesem Dokument nicht darauf eingehen wie man Typescript in jeweilige Projekte integriert. In vielen Frameworks ist es schon drin und in selbstgebauten Projekten eignet sich ein Compiler wie Babel oder ähnliches dafür.

Um die Funktion von dem vorherigen Kapitel wieder aufzugreifen, so würde sie in Typescript aussehen:
```typescript
function add(x: number, y: number): number {
    return x + y;
}
```
Problem gelöst. Selbst wenn man nur die Signatur der Funktion sieht weiß man genau dass diese Funktion nur `number` als Typ hat und man kann sich drauf verlassen, dass da immer eine Zahl raus kommt, denn der Typescript-Compiler würde einen Fehler werfen, wenn die Funktion keine `number` zurück gibt. Hier ein Beispiel:
```typescript
function div(x: number, y: number): number {
    if (y !== 0) {
        return x / y;
    }
}
```
Was ist der Fehler an dieser Funktion? Richtig, wenn `y` gleich null ist, dann gibt diese Funktion `undefined` zurück und verletzt damit den Typ `number`. Der Compiler gibt dann eine Warnung aus: `Function lacks ending return statement and return type does not include 'undefined'.` 

Also der Fehler der mit Javascript erst durch ausführen der Funktion mit `y` gleich null aufgefallen ist, fällt jetzt sofort beim Schreiben der Funktion auf. Ein ähnlicher Fehler tritt auf wenn man versucht eine Funktion aufzurufen, wenn die Parameter nicht übereinstimmen.

Zum Beispiel wenn wir die Funktion `add` mit einem String aufrufen kommt folgender Fehler: `Argument of type '"1"' is not assignable to parameter of type 'number'.` Das macht entwickeln ziemlich einfach.

Aber was für Typen gibt es denn eigentlich?
* `number` (Zahlen)
* `string` (mit einzelnen und doppelte Anführungsstriche)
* `object` (Damit meint man nur das leere Objekt)
* `boolean` (`true` oder `false`)
* `enum` (Im Prinzip sind das benannte Konstanten)
* `void` (als Rückgabewert von Funktion heißt das "Diese Funktion gibt nichts zurück" das heißt auch dass man den Wert den die Funktion zurück gibt nicht verwenden kann.)
* `Array` und `tuple` (Mehr dazu im Kapitel über Container Typen)
* Konstante Literale sind auch erlaubt: `null`, `undefined` (Mehr dazu in Kapitel 4

Besondere Typen:
* `never`: Diese Funktion gibt nie irgendwas zurück, sondern schmeißt nur `Exception`s)
* `any`: Wenn man den Type angeben will aber in explizit wage lässt.

## Kapitel 3: Wie funktioniert das Typsystem
Das Typsystem von Typescript bzw. Javascript damit indirekt auch ist strukturelle Typisierung oder auch [Duck-Typing](https://de.wikipedia.org/wiki/Duck-Typing) genannt. Hier mal ein Beispiel: Wir haben zwei Objekte, die beide die gleichen Methoden: Beide Objekte haben eine Methode `quack(): string`. Für Typescript sind diese typen-mäßig das gleiche Objekt. An einem praktischen Beispiel:
```typescript
function getSize(sizable): number {
    return sizable.length;
}
```
Was für ein Typ muss der Parameter `sizable` haben? Eigentlich ist das egal solange `sizable` eine Eigenschaft `length` besitzt die vom Typ `number` ist. Ein Beispiel für gültige Typen in dem Fall wäre `string` und `Array`. Wie wir dann unseren eigenen Typen definieren, der diese Information enthält sehen wir im nächsten Kapitel.

Eine weitere Sache die noch wichtig zu verstehen ist, ist wo überall man Typen angeben kann:
1. Beim definieren von Funktionen
```typescript
function name(parameter: any): any {}
```
2. Beim definieren von Variablen (sinnvoll ist das meistens nur wenn die Variable später erst initialisiert wird).
```typescript
let a: number = 0;
// und weiter unten im Code passiert.
if (a === 0) {
    a = 10;
}
```
3. Beim expliziten Anpassen von Typen, denn Typescript ist nicht immer perfekt im erahnen was für ein Typ eine Variable hat. Besonders wenn es Daten sind die über eine API kommen. Deswegen kann man in Typescript Variablen Typen zuweisen, die sie eigentlich gar nicht haben.
```typescript
let f = someExternalFunction() as string;
```
Das setzt den Typ für die Variable `f` auf `string`. Sollte das nicht mit dem eigentlichen Rückgabewert übereinstimmen, ist das egal.

## Kapitel 4: Typen selber erstellen
