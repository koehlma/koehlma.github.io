---
layout: post
title: "German: Jaspy – Code-Objekte"
---

Dies ist der zweite Artikel einer Reihe, die sich mit dem Schreiben von Jaspy, einem Python Interpreter in JavaScript, befasst. Zur [Einführung]({% post_url 2016-03-08-german-jaspy-einfuehrung %}).

Um einen Interpreter schreiben zu können, muss man sich zunächst überlegen, wie die ausführbaren Einheiten eines Programms aussehen.

Python ist nach der Von-Neumann-Architektur aufgebaut. Das bedeutet Code und Daten liegen in einem gemeinsamen Speicher. In diesem Sinne ist in Python insbesondere der Code selbst ein Objekt, das zur Laufzeit zur Verfügung steht. Daher unterscheide ich im Folgenden zwischen dem eigentlichen Quellcode, der in Form einer Textdatei vorliegt und dem Code-Objekt, wie es der Interpreter sieht bzw. wie es innerhalb von Python selbst zur Verfügung steht. Allgemein gilt, das jeglicher Quellcode vom Python-Interpreter zunächst in ein entsprechendes Code-Objekt übersetzt wird. Auf dieses kann beispielsweise bei einer Funktion über das Attribut **__code__** zugegriffen werden:

{% highlight python %}
>>> def hello(name='World'):
...     print('Hello,', name + '!')
...
>>> hello.__code__
<code object hello at 0x7f26c26b8930, file "<stdin>", line 1>
{% endhighlight %}

Ein Code-Objekt enthält sowohl eine binäre Repräsentation (Bytecode) des eigentlichen Programmcodes, also die Anweisungen, die später vom Interpreter ausgeführt werden, als auch zusätzliche Informationen, wie verwendete Bezeichner, Konstanten und die Parameter-Signatur.

Die Parameter-Signatur gibt an, welche Argumente das Stück Bytecode erwartet. Sie ist eindeutig bestimmt durch die Anzahl an positionellen sowie keyword-only Argumenten, deren Namen und eventuell, ob eine variable Anzahl an Argumenten bzw. Keyword-Argumenten akzeptiert wird. Diese Informationen werden innerhalb eines Code-Objekts in den Attributen *co_argcount*, *co_kwonlyargcount*, *co_varnames* und *co_flags* gespeichert. Auch Module, deren Code ebenfalls durch ein Code-Objekt repräsentiert wird, weisen eine Signatur auf, allerdings sieht diese keine Parameter vor. Werden Standardwerte wie im Beispiel oben angegeben, so werden diese erst zur Laufzeit an das Funktions-Objekt gebunden. Ein Code-Objekt ist also ein konstantes Konstrukt, das vom Interpreter beim Einlesen des Quellcodes erzeugt werden kann.

Um Jaspy möglichst klein zu halten, was der Ladezeit zu Gute kommt, erhält es erst einmal keinen eigenen Compiler, um aus Quellcode ein entsprechendes Code-Objekt zu erzeugen. Dafür kommt stattdessen der herkömmliche CPython Interpreter zum Einsatz, der die Code-Objekte erzeugt, welche dann in einem Zwischenschritt in ein entsprechendes JavaScript Objekt übertragen werden sollen.

Dazu muss das Konzept eines Code-Objekts nun also entsprechend in JavaScript nachgebaut werden. Zunächst machen wir uns dabei keine Gedanken darüber, wie später aus Python-Code darauf zugegriffen werden kann (im Zweifelsfall erledigt dies ein einfacher Wrapper).

Da Jaspy einfach mit nativen JavaScript Modulen erweiterbar sein soll, ist es außerdem wichtig neben dem klassischen Python-Code-Objekt, dessen Bytecode interpretiert werden muss, noch eine zweite Kategorie von Code einzuführen – den nativen Code. Native Code-Objekte enthalten anstelle des Bytecodes eine JavaScript Funktion, die die Anweisungen enthält.

Die folgenden beiden Abschnitte beziehen sich auf konkreten Quellcode von Jaspy, der in der Datei [src/code.js](https://github.com/koehlma/jaspy/blob/master/src/code.js) zu finden ist.

### Signatur
Die Parameter-Signatur ist sowohl für Python-Code als auch für nativen Code nach dem selben Schema, dem von Python, aufgebaut. Anders als in CPython gibt es ein eigenständiges Signatur-Objekt, dessen Konstruktor in JavaScript folgendermaßen aussieht:

{% highlight javascript %}
function Signature(argnames, poscount, var_args, var_kwargs) {
    this.argnames = argnames || [];
    this.poscount = poscount || 0;
    this.var_args = var_args || false;
    this.var_kwargs = var_kwargs || false;
}
{% endhighlight %}

Eine Signatur, wie `(a, b, *c, x, **y)` wird dann entsprechend in

{% highlight javascript %}
new Signature(['a', 'b', 'c', 'x', 'y'], 2, true, true);
{% endhighlight %}

übersetzt. Anders als in CPython wird dabei, die Anzahl an keyword-only Argumenten (in diesem Fall „Eins“) nicht explizit gespeichert. Sie lässt sich berechnen indem man von der Gesamtzahl der Argumente (ohne die variablen Argumente zu berücksichtigen) die Anzahl der positionellen Argumente subtrahiert.

Ein Signatur-Objekt stellt die Methode *parse_args* bereit, mit der sich Argumente bei einem Aufruf entsprechend verarbeiten lassen:

{% highlight javascript %}
>>> var sig = new Signature(['a', 'b', 'c', 'x', 'y'], 2, true, true);
>>> sig.parse_args([123, 'hello', 1, 2], {x: 'world', d: 4, e: 5})
[123, 'hello', [1, 2], 'world', {d: 4, e:5}]
{% endhighlight %}

Die Argumente werden entweder in Form eines Arrays zurückgegeben oder direkt in einen gegeben Namensraum (zum Beispiel den der lokalen Variablen innerhalb einer Funktion) geschrieben. Außerdem lässt sich ein Objekt mit Standard-Werten für bestimmte Parameter übergeben und bei Fehlern gibt es eine entsprechende Exception in JavaScript.

Des Weiteren gibt es zwei statische Methoden, die eine neue Signatur anhand einer gegeben Spezifikation (*from_spec*) oder aus den Daten eines CPython Code-Objekts (*from_python*) erzeugen:

{% highlight javascript %}
>>> Signature.from_spec(['a', 'b', '*c', 'x', '**y']);
Signature(['a', 'b', 'c', 'x', 'y'], 2, true, true);
>>> Signature.form_python(['a', 'b', 'x', 'c', 'y'], 2, 1, VARARGS | VARKWARGS);
Signature(['a', 'b', 'c', 'x', 'y'], 2, true, true);
{% endhighlight %}

Damit erhält man eine Abstraktion der Parameter-Signaturen, die für beide Kategorien von Code-Objekten einsetzbar ist.

### Code
Mithilfe der allgemeinen Parameter-Signaturen lässt sich ein generisches Code-Objekt nun folgendermaßen erzeugen:

{% highlight javascript %}
function Code(signature, options) {
    this.signature = signature;

    options = options || {};

    this.name = options.name || '<unknown>';
    this.filename = options.filename || '<unknown>';

    this.flags = options.flags || 0;
}
{% endhighlight %}

Ein Code-Objekt hat also immer eine Parameter-Signatur sowie einen optionalen Namen, außerdem wird auch noch der Dateiname gespeichert, aus der das Code-Objekt heraus erzeugt wurde, sofern er bekannt ist. Im<em> flags</em> Attribut lassen sich verschiedene Bits setzen, zum Beispiel ob es sich bei dem Code-Objekt um einen Generator handelt.

### Python Code
Ein Python-Code Objekt enthält zunächst einmal den Bytecode, der aus den eigentlichen Anweisungen erzeugt wurde. Mit dem Bytecode selbst werden wir uns im [nächsten Artikel]({% post_url 2016-03-10-german-jaspy-bytecode-und-vm %}) genauer auseinandersetzen, für den Moment bekommt unser Python-Code-Objekt einfach ein Feld, in dem der Bytecode als Zeichenkette abgespeichert wird. In CPython gibt es hierfür das Attribut *co_code* innerhalb eines Code-Objekts. In Jaspy werde ich die Attribute jedoch anders benennen, da ich denke, dass ein Präfix wie *co_*, der sich aus dem Typ ableitet, im Allgemeinen redundant ist.

Zusätzlich zu dem Bytecode werden noch einige andere Informationen abgelegt. Damit ergibt sich dann folgender Python-Code-Konstruktor:

{% highlight javascript %}
function PythonCode(bytecode, options) {
    this.bytecode = bytecode;

    options = options || {};
    options.flags = (options.flags || 0) | CODE_FLAGS.PYTHON;
    options.name = options.name || '&lt;module&gt;';
    options.filename = options.filename || '&lt;string&gt;';

    this.names = options.names || [];
    this.varnames = options.varnames || [];
    this.freevars = options.freevars || [];
    this.cellvars = options.cellvars || [];

    this.argcount = options.argcount || 0;
    this.kwargcount = options.kwargcount || 0;

    Code.call(this, Signature.from_python(this.varnames, this.argcount, this.kwargcount, options.flags), options);

    this.constants = options.constants || [];

    this.firstline = options.firstline || 1;
    this.lnotab = options.lnotab || '';
}
{% endhighlight %}

Die Felder *names* und *varnames* enthalten im Programm vorkommende lokale Bezeichner, wobei *varnames* auch die Namen der Parameter für die Signatur enthält (mit dem genauen Unterschied befassen wir uns zu einem späteren Zeitpunkt). Das Feld *freevars* enthält Bezeichner, die im Quellcode frei vorkommen, die also benutzt werden ohne, dass sie vorher einen Wert zugewiesen bekommen haben. Diese benötigen später eine extra Behandlung, genau wie die Bezeichner im Feld *cellvars*. Das Attribut *constants* enthält im Quellcode vorkommende Konstanten. Mithilfe der Felder *firstline* und *lnotab* lässt sich jeder Position innerhalb des Bytecodes eine Zeilennummer in der Quelldatei zuordnen. Mehr dazu später.

### Native Code
Nativer Code besteht im wesentlichen aus einer Parameter-Spezifikation (siehe oben) und einer JavaScript Funktion (*func*):

{% highlight javascript %}
function NativeCode(func, options, spec) {
    this.func = func;

    options = options || {};
    options.flags = (options.flags || 0) | CODE_FLAGS.NATIVE;
    options.name = options.name || func.name;
    options.filename = options.filename || '&lt;native&gt;';

    Code.call(this, Signature.from_spec(spec), options);
}
{% endhighlight %}

Zum [nächsten Artikel]({% post_url 2016-03-10-german-jaspy-bytecode-und-vm %}).