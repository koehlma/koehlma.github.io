---
layout: post
title: "German: Jaspy – Bytecode und VM"
---
In diesem Artikel zur Entstehung von Jaspy, einem Python Interpreter in JavaScript, geht es um den Python Bytecode und die virtuelle Maschine, die diesen ausführt. Zum [vorherigen Artikel]({% post_url 2016-03-09-german-jaspy-code-objekte %}).

Geschriebener Python-Quellcode wird nicht direkt vom Interpreter ausgeführt, sondern zunächst, in einem Zwischenschritt, zu Bytecode übersetzt. Dies ist vergleichbar mit dem Kompilierungsvorgang für einen Prozessor, der eine bestimmte Menge an Instruktionen versteht. Der resultierende Bytecode wird anschließend innerhalb einer virtuellen Maschine (VM), die eben jenen Befehlssatz versteht, ausgeführt.

### Theoretisches Modell
Allgemein gibt es verschiedene Möglichkeiten eine Maschine, wie zum Beispiel einen herkömmlichen Computer, zu konstruieren. Bei x86- oder MIPS-Prozessoren handelt es sich beispielsweise um Registermaschinen. Es gibt also eine Menge an Registern und eine Menge an Instruktionen, die bestimmte Operationen auf den Registern durchführen. Zusätzlich dazu existiert ebenfalls noch ein Speicher, in den Werte geschrieben oder aus dem heraus Werte gelesen werden können.

Ein Programm ist dann eine Folge von Instruktionen für eine gegebene Maschine. Wird ein Programm ausgeführt, wird es zunächst Instruktion für Instruktion abgearbeitet. Um auch Sprünge zu ermöglichen gibt es einen Zähler (Program Counter, PC) der die Position im Programm angibt, an der sich die Maschine aktuell befindet. Ein Sprungbefehl verändert nun einfach den Program Counter entsprechend.

Bei der virtuellen Maschine von Python handelt es sich um eine Stapel-Maschine mit Speicher. Es gibt also einen Stapel (im Folgenden Stack) auf dem Objekte abgelegt werden können, eine Menge an Instruktionen, die dann Operationen auf diesen Objekten durchführen, sowie einen Speicher in dem Objekte unter einem bestimmten Bezeichner abgelegt werden können. Betrachten wir dazu einmal folgendes Beispiel:

{% highlight python %}
3 + x
{% endhighlight %}

Übersetzt in Instruktionen für eine entsprechende Stapel-Maschine sieht das dann folgendermaßen aus:

{% highlight python %}
LOAD_CONST       3
LOAD_NAME        x
BINARY_ADD
{% endhighlight %}

Bei der Ausführung dieses Programms wird also zunächst die Konstante „3“ geladen und danach das an den Bezeichner „x“ gebundene Objekt, beispielsweise die Zahl Vier. Nach der Ausführung der ersten beiden Instruktionen sieht der Stack dann so aus:

{% highlight python %}
[3, 4]
{% endhighlight %}

Die Instruktion *BINARY_ADD* nimmt nun die beiden oberen Objekte vom Stack herunter, addiert sie und legt das Ergebnis anschließend wieder auf dem Stack ab, der dann also abschließend so aussieht:

{% highlight python %}
[7]
{% endhighlight %}

Eine Instruktion für die Stapel-Maschine kann ein Argument besitzen, wie bei *LOAD_CONST* und *LOAD_NAME*, oder nicht, wie bei *BINARY_ADD*.

Soweit zum theoretischen Modell einer Stapel-Maschine.

### Bytecode
Jedes Python-Programm wird vom CPython Interpreter also zunächst in eine entsprechende Folge an Instruktionen übersetzt. Die dann in eine binäre Form, den Bytecode, kodiert wird. Jedes Code-Objekt hat ein Feld *co_code*, das den Bytecode enthält:

{% highlight python %}
>>> compile('3 + x', '<string>', 'eval').co_code
b'd\x00\x00e\x00\x00\x17S'
{% endhighlight %}

Mithilfe des Moduls [dis](https://docs.python.org/3/library/dis.html) lässt sich der Bytecode dekodieren:

{% highlight python %}
>>> dis.dis(compile('3 + x', '<string>', 'eval'))
  1           0 LOAD_CONST               0 (3)
              3 LOAD_NAME                0 (x)
              6 BINARY_ADD
              7 RETURN_VALUE
{% endhighlight %}

In unserem JavaScript Python-Code-Objekt steht uns der Bytecode durch das Feld *bytecode* als Zeichenkette zur Verfügung. Um diesen später auch ausführen zu können, müssen wir allerdings zunächst verstehen, wie die Folge an Instruktionen kodiert wird.

Wie bereits festgestellt kann eine Instruktion ein Argument haben. Sie besteht also maximal aus zwei Teilen, dem Opcode (*LOAD_CONST*, …), der angibt was getan werden soll und dem optionalen Argument. Die Opcodes sind alle durchnummeriert (siehe [opcode.py](https://hg.python.org/cpython/file/tip/Lib/opcode.py)), wobei alle Opcodes ab der Nummer *dis.HAVE_ARGUMENT* ein Argument besitzen. Diese Konstanten werden in Jaspy in der Datei [jaspy/constants.js](https://github.com/koehlma/jaspy/blob/master/src/constants.js) vorgehalten.

Eine Instruktion wird dann wie folgt kodiert. Das erste Byte gibt den Opcode der Instruktion an. Hat eine Instruktion ein Argument, so geben die beiden nachfolgenden Bytes den Wert des Arguments an (Kodierung erfolgt in Little-Endian Reihenfolge). Ein Argument ist also erst einmal nur eine Zahl. Hier kommen dann die zusätzlichen Informationen aus dem Code-Objekt ins Spiel. Beispielsweise gibt im Fall von *LOAD_NAME* das Argument den Index an unter welchem der eigentliche Bezeichner im Feld *names* (bzw. *co_names* in CPython) des Code-Objekts zu finden ist.

Der Bytecode ergibt sich schließlich durch die Aneinanderreihung der Kodierungen der einzelnen Instruktion.

In Jaspy kümmert sich die Funktion *disassemble* in der Datei [jaspy/dis.js](https://github.com/koehlma/jaspy/blob/master/src/dis.js) darum den Bytecode in einzelne Instruktionen zu zerlegen. Die Ziele von Sprungbefehlen werden dabei ebenfalls entsprechend angepasst, sodass sie den Index der Ziel Instruktion angeben anstelle der Position in der binären Kodierung.

Wir erweitern außerdem die Python-Code-Objekte noch um ein Feld um die zerlegten Instruktionen zu speichern:

{% highlight javascript %}
this.instructions = disassemble(this);
{% endhighlight %}

Bevor wir nun die virtuelle Maschine bauen können, die die Instruktionen letztendlich ausführt, müssen wir uns im nächsten Schritt zuerst noch überlegen, wie Python-Objekte in JavaScript repräsentiert werden sollen.