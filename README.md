# Typescript für Einsteiger
Dieses Dokument soll die Grundlagen von Typescript darstellen und ihre Anwendungsfälle.

Ich bin seit 5 Jahren PHP Entwickler und daher sind die Anwendungsbeispiele ein bisschen theoretisch hoffe aber trotzdem dass sie veranschaulichen, wie Typescript funktioniert und was für Vorteile es hat.

## Kapitel 0: Die Motivation
Javascript ist eine sehr populäre Sprache für den Bau von Webseiten geworden. Sollte man jedoch keine eigenen Lösungen bauen sondern ein Framework benutzen wird die Sprache meistens vorgegeben. Die meisten Frameworks wie Angular, React und View heutzutage benutzen schon Typescript. Jedoch bei kleineren Projekten wird meist immer noch auf Javascript gesetzt weil das integrieren von Typescript in den workflow nicht ganz so einfach ist wie es scheint.

Jedoch eins der Probleme die an Javascript selber sehr schnell klar werden ist, dass dies nicht sehr Entwickler-freundlich ist und mache Sachen sind schwer zu debuggen. Hier mal ein Beispiel:

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
    if (typeof(x) === 'number' && typeof(y) === 'number') {
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
    if (typeof(x) === 'number' && typeof(y) === 'number') {
        return x + y;
    }
    throw new Error('Parameters need to be numbers');
}
```
Aber allgemein beleibt dann immer noch Problem Nummer 2: Wir können nicht von außen sehen wie die Funktion genutzt werden soll. Und mit mehr Argumenten die auch unterschiedliche Typen haben, wird dieser Ansatz schnell unübersichtlich und unpraktisch.

## Kapitel 2: Erste Schritte mit Typescript
Für die Leute, die ein bisschen mit Typescript experimentieren wollen: [Hier ist die Sandbox](https://www.typescriptlang.org/play/)

Wir werden in diesem Dokument nicht darauf eingehen wie man Typescript in jeweilige Projekte integriert. In vielen Frameworks ist es schon drin und in selbstgebauten Projekten eignet sich ein Compiler wie Babel oder ähnliches dafür. Aber da Typescript den Javascript Syntax erweitert ist es kein Problem den Typescript Compiler über ein Javascript Projekt laufen zu lassen. Damit hat man nur nicht viel erreicht, weil in Javascript nur sehr wenig Typisierungsinformation gesammelt werden kann. Das Typescript aber auch Javascript Dateien im Projekt erlaubt macht die Migration eines existierenden Projekts einfacher.

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
 * Konstante Literale sind auch erlaubt: `null`, `undefined`

```typescript
function howOldAmI(age: 'old'|'young'|'very old'): string {
    return 'You are ' + age;
}
```
Das heißt gültige Werte für diese Funktion sind der String "old", "young" oder "very old" und kein anderer `String`.

Besondere Typen:
* `never`: Diese Funktion gibt nie irgendwas zurück, sondern schmeißt nur `Exception`s)
* `any`: Wenn man den Type angeben will aber in explizit wage lässt.

Eine weitere Sache die noch wichtig zu verstehen ist, ist wo überall man Typen angeben kann:

1\. Beim definieren von Funktionen
```typescript
function name(parameter: any): any {}
```

2\. Beim definieren von Variablen (sinnvoll ist das meistens nur wenn die Variable später erst initialisiert wird).
```typescript
let a: number = 0;
// und weiter unten im Code passiert.
if (a === 0) {
    a = 10;
}
```
3\. Beim expliziten Anpassen von Typen, denn Typescript ist nicht immer perfekt im erahnen was für ein Typ eine Variable hat. Besonders wenn es Daten sind die über eine API kommen. Deswegen kann man in Typescript Variablen Typen zuweisen, die sie eigentlich gar nicht haben.
```typescript
let f = someExternalFunction() as string;
```
Das setzt den Typ für die Variable `f` auf `string`. Sollte das nicht mit dem eigentlichen Rückgabewert übereinstimmen, ist das egal. Da beim Überführen des Typescript Codes zu reinem Javascript die Typen Definitionen weg gelassen werden.


## Kapitel 3: Wie funktioniert das Typsystem
Das Typsystem von Typescript bzw. Javascript damit indirekt auch ist strukturelle Typisierung oder auch [Duck-Typing](https://de.wikipedia.org/wiki/Duck-Typing) genannt.
```typescript
let test1 = { test: true }
let test2 = { test: false }

```
Dann haben die beiden Objekte `test1` und `test2` den gleichen Typ, da beide die Eigenschaft `test` enthalten, die ein `boolean` ist. 

Fügen wir nun dem Objekt `test2` eine Eigenschaft `num: number` hinzu, so ist `test2` "größer" als `test1`. Was bedeutet, überall wo `test1` erlaubt ist, ist auch `test2` erlaubt. Anders herum aber nicht, weil `test1` die Eigenschaft `num` fehlt.

Wenn man sich das ganze in einer Klassenhierarchie vorstellt sieht das dann so aus:
```typescript
class Test1 {
    test: boolean;
}

class Test2 extends Test1 {
    num: number;
}
```
Mehr zum Klassensyntax in Kapitel 5.

Aber eine Sache die man bedenken muss ist, nicht überall ist es sinnvoll Typen anzugeben. Typescript hat, wie viele andere Programmiersprachen auch, die Möglichkeit Typen abzuleiten. Beispiel:
```typescript
function getNumber(): number {
    return 0;
}

let f = getNumber();
```
In diesem Falle brauchen wir bei der Definition `let f:number = getNumber()` keinen Typ dran schreiben. Der Grund dafür ist, es ist für den Compiler klar, dass es eine `number` sein muss. Und `let f: string = getNumber()` wäre einfach falsch.

Optionale Parameter und optionale Eigenschaften von Objekten können mit dem `:?` Syntax dargestellt werden. Das bedeutet eigentlich nur, dass dieser Parameter / Eigenschaft auch `undefined` sein kann.
```typescript
function 
```

## Kapitel 4: Typen selber erstellen
Typen kann man in Typescript überall definieren. Für zentrale Typen, die man überall in dem Projekt braucht eignet sich eine sogenannte [Definitionsdatei](https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html) mit der Endung `.d.ts`.

#### Aus Klassen und Objekten
Die einfachste Art einen Typ anzulegen ist eine Klasse zu definieren, denn jede Klasse ist ein Typ. Wenn man von einem bereits existierenden Objekt den Typ übernehmen will, kann man das machen mit dem `typeof` Schlüsselwort.
```typescript
let a = 10;
let b: typeof a = 20;
```
Dann haben `a` und `b` beide den Typ (in diesem Fall `number`).

### Union Types
Eine weitere Möglichkeit ist es zwei oder mehrere existierende Typen zu verbinden. Dies nennt sich ein Union-Type. Hier ein Beispiel:
```typescript
type numberLike = boolean|number;
```
Das heißt eine Funktion mit diesem Parameter kann entweder mit jeder beliebigen Zahl aufgerufen werden oder mit einem `boolean`. Sollte dies als Typ eines Parameters einer Funktion gelten. Muss die Funktion für beide Typen "funktionieren". Dazu prüft Typescript die unterschiedlichen Pfade die Variablen in der Funktion nehmen können und prüft alle. Ein Beipsiel:
```typescript
function someFunction(a: string|number): void {
    if (typeof(a) === 'string') {
        // a ist ein String
        console.log(a.length);
    } else {
        // a ist eine Zahl
        console.log(a.toFixed(2));
    }
}

someFunction("Hello");
someFunction(10.224334);
```
Der Output ist einmal `5` und "10.22". Würden wir aber jetzt die beiden `console.log` Aufrufe tauschen, gibt Typescript den Fehler aus, dass es kein Property `length` auf number gibt und keine Funktion `toFixed` auf einem String. Wenn man dann eine IDE mit Vervollständigung hat, werden einem in den unterschiedlichen Zweigen der If-Abfrage auch für `a.` unterscheidliche Vorschläge angezeigt.

### Interfaces
Ein Interface ist ähnlich wie eine Klasse nur ohne eine gegeben Implementation. In einem Interface definiert man die Eigenschaften und Methoden die ein Objekt oder eine Klasse haben soll.
```typescript
interface Countable {
    length: number;
}

function getLength(a: Countable): number {
    return a.length;
}

const someObject: Countable = {
    length: 10
}

console.log(getLength(someObject));
```
Dieses Programm schreibt `10` in die Konsole. Dies mag zwar auf den ersten Blick nicht sehr sinnvoll aussehen und man fragt sich warum man das machen sollte, das ist bei APIs ziemlich hilfreich. Mehr dazu in Kapitel 6.

## Kapitel 5: Erweiterung des Klassensyntax
Wie schon im ES6 Standard enthalten hat Javascript einen Klassen Syntax.
```typescript
class Duck {
    public quack(): void {
        console.log('Quack');
    }
}
```
Da hat sich mit Typescript aber auch noch etwas geändert: Wie in vielen anderen Programmiersprachen auch haben Methoden und Eigenschaften in einer Klasse jetzt Zugriffsmodifikatoren. `public`, `protected` und `private`:

* `public`: Jeder hat auf die Methode bzw. Eigenschaft Zugriff.
* `protected`: Wenn eine Klasse erbt, dann hat sie Zugriff auf die `protected` Eigenschaften und Methoden der Kind Klasse, aber kein anderer.
* `private`: Nur Funktionen und Methoden innerhalb der gleichen Klasse haben Zugriff auf die Eigenschaften und Methoden.

Eine weiter Änderung in Typescript erfordert das Definieren von Eigenschaften in einer Klasse ähnlich wie bei dem `Interface`:
```typescript
class Car {
    private brand: string;
    private horsePower: number;
}
```
`Private` Instanzvariablen müssen im `constructor` gesetzt werden. Das kann man auf zwei Arten machen:
```typescript
// Alle Eigenschaften einzeln
class Car {
    private brand: string;
    private horsePower: number;

    public constructor(brand: string, horsePower: number) {
        this.brand = brand;
        this.horsePower = horsePower;
    }
}

// Da das häufig passiert gibt es dafür eine kürzere Variante
class Car {
    public constructor(
        private brand: string,
        private horsePower: number
    ) { }
}
```
Das Konzept von "Eigenschaften müssen im Vorfeld bekannt sein" ändert ein Stück weit auch das Objektmodel. Nun ist es nicht mehr möglich ist neue Eigenschaften dynamisch hinzufügen, zumindest nicht direkt.
```typescript
class AllObjects{
    [key: string]: any;

    // Rest der Klasse hier
}
```
Was das bedeutet, ist außerhalb der Grundlagen und deswegen [hier](https://www.typescriptlang.org/docs/handbook/advanced-types.html#index-types) nur ein Link zu der Dokumentation von Typescript wie Index-Typen funktionieren.
> Die vereinfachte Variante ist: In Javascript erlaubt damit `object['property']` zu sagen solange das in den eckigen Klammern vom Typ `string` ist, der Typ von dem Ausdruck ist dann `any`.

Eine weiter ziemlich hilfreiche Funktion ist Eigenschaften `readonly` zu machen. Das bedeutet, dass man die ein Mal setzen kann und danach nur noch lesen (ähnlich wie eine Konstante).
```typescript
class SomeClass {
    constructor(
        public readonly test: boolean
    ) {}
}

let a = new SomeClass(false);
a.test = false;
```
Gibt uns folgenden Fehler: `Cannot assign to 'test' because it is a read-only property.`

## Kapitel 6: Anwendungsbeispiele
Hier sind zwei mögliche Anwendungsbeispiele:

### Typisierte API Antworten
Im folgenden ein Beispiel was verdeutlicht, wie Typescript beim verwenden von APIs behilflich ist.
```typescript
interface User {
    name: string,
    roles: string[], // Ein Array von Strings
    age: number,
    isAdmin: boolean
}

interface ApiResponse {
    users: User[] // Ein Array von Usern
}

fetch('https://some-url-that-does-return-stuff.com/users')
    .then(response => response.json())
    .then((data: ApiResponse) => {
        // Die Logik hier!
    })
```
In diesem Beispiel wird die [Fetch Funktion](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) verwendet. Aber nichts desto trotz ohne die eigentliche API je gesehen zu haben kann man jetzt Typen-sicher programmieren mit Autovervollständigung. Und wenn man das zentral in einer Typ-Datei speichert, hat man auf diese Typen im ganzen Projektzugriff.

### Typescript hat auch Types für DOM-Objekte
Ein weiteres Beispiel für Hilfe bei der DOM Manipulation:
```typescript
const element = document.getElementById('someId');
```
Diese Konstante hat jetzt den Typ `HTMLElement|null` weil es kann sein, dass das Element nicht existiert. Wenn wir die Möglichkeit ignorieren möchten schreiben wir folgendes:
```typescript
const element = document.getElementById('someId') as HTMLElement;
```

Damit geben wir den Typ des Ausdrucks auf der rechten Seite vor. Das ganze Beispiel ist folgendes:
```typescript
const element = document.getElementById('someId') as HTMLElement;
element.style = {position: "absolute", height: "100px", width: "10px"};
```

In Javascript wäre das kein Problem. Aber es macht nicht dass was es soll. Das Element hat kein neuen Style. Die Fehlermeldung von Typescript sagt warum: `Cannot assign to 'style' because it is a read-only property.`

### Anwendungsfall für `Enum`s
Ein guter Anwendungsfall für ein `Enum` sind unterschiedliche Zustände zum Beispiel für eine Ampel.
```typescript
enum TrafficLightState {
    RED, YELLOW, GREEN
}
```
Damit brauch man sich nicht auf magische Konstanten zu verlassen und auch nicht auf string Vergleiche. Ein Anwendungsfall wäre zum Beispiel folgendes:
```typescript
function canICrossTheStreet(traficLight: TrafficLightState): string
{
    switch(traficLight)
    {
        case TrafficLightState.GREEN:
            return "Jap, einfach drüber";
        case TrafficLightState.YELLOW:
            return "Eigentlich nicht aber macht sowieso jeder";
        case TrafficLightState.RED:
            return "Nein";
    }
}
console.log(canICrossTheStreet(TrafficState.GREEN));
```
Diese Funktion gibt nun zu jedem Zustand der Ampel einen String aus. Es gibt nun immer noch die Möglichkeit direkt die Zahl rein zu geben, aber davon wird abgeraten.

## Kapitel 7: Weiterführende Themen
Typescript kann natürlich noch so viel mehr als das was gerade beschrieben wurde. Ein paar der etwas komplizierteren Themen sollen hier nur mal kurz angeschnitten werden.

### Template Types aka. Generics
Einen Datentyp den wir uns noch nicht genauer angeschaut haben sind `Array`s. Eine Typendefinition von einem Array kann zwei unterschiedliche Erscheinungsbilder haben. Entweder schreibt man es mit `number[]` oder mit `Array<number>`. Die zweite Schreibweise verdeutlicht wie genau das gemeint ist. Denn der Typ `Array` hat einen Parameter, das was in den spitzen Klammern.

Das ist das Grundkonzept von Generics. Wir haben eine Generische Klasse zu Beispiel ein Array, die einfach nur das Konzept von einer Liste abbildet. Und dann mit einem weiteren Typen `number` zu einer Liste an Zahlen wird.

### Iterator und Generator
Iteratoren und Generatoren sind Konzepte die es auch schon in Javascript gibt. In Typescript werden die zusammen mit Generics zu einem sehr mächtigen Tool. Hier mal ein einfaches Beispiel
```typescript
function* getPaginatedUrls(): Generator<string> {
    for (let page = 1; page < 100; page++) {
        yield `http://someurl/some-resource?page=${page}`;
    }
}

for(let url of getPaginatedUrls()) {
    console.log(url);
}
```

### Decorators
Das ist ein experimentelles Feature von Typescript. Das erlaubt es einem Funktionen durch einfaches davor schreiben von einem Ausdruck zu ändern:
```
@logging
function doSomething(): void
{
}
```
Wenn die Funktion logging ein `console.log` vor und nach der Ausführung von `doSomething` schreibt kann man so ganz einfach jede Funktion mit der `@logging` Dekoration versehen und ganz einfach vieles loggen. Mehr dazu [in der Dokumentation](https://www.typescriptlang.org/docs/handbook/decorators.html).

## Kapitel 8: Schlusswort
Typescript ist ein mächtiges Tool und es kann manchmal auch nervenaufreibend sein es zufrieden zu stellen aber dafür kann man sicher sein, dass die einzigen Fehler im Code Logikfehler sind und nicht irgendwelche fehlenden Eigenschaften.

Egal ob man mit Typescript neu anfängt oder ein großes Projekt portieren darf: Es ist wichtig dass man sich die Compiler Flag anschaut die Typescript hat. Für neue Projekte alle Compiler Flags anmachen um striktere Checks zu bekommen und gar nicht erst die Fehler machen. Die großen legacy Projekte können von der `--allow-js` Funktion Gebrauch machen und Stück für Stück migrieren.

Alles in allem definitiv eine Bereicherung für jeden Frontend-Entwickler und ein Tool, das ich nicht mehr missen möchte.
