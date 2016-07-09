---
layout: post
title: "German: GIMP – Animation verlangsamen mit Python"
---
Man sollte doch meinen, dass es, für eine solch triviale Aufgabe, wie eine Animation um einen gegeben Faktor zu verlangsamen, eigentlich eine eingebaute Funktion in GIMP geben sollte. Dies ist jedoch nicht der Fall.

Die Dauer jeder Frame wird im Name der jeweiligen Ebene in Klammern angegeben. Zu faul um alle Ebenen händisch umzubenennen, kam mir die Idee, dafür einfach den in GIMP eingebetteten Python Interpreter zu bemühen, dessen Konsole über *Filter → Python-Fu → Konsole* zu erreichen ist. Dieses kleine Skript erledigt dann die eigentliche Arbeit:

{% highlight python %}
import re

regex = re.compile('\(([0-9]+)(m?s)\)')
image = gimp.image_list()[0]

for layer in image.layers:
    match = regex.search(layer.name)
    if not match: continue
    duration = int(match.group(1))
    start = layer.name[:match.start() + 1]
    end = layer.name[match.end() - 1:]
    layer.name = start + str(int(duration * 1.5)) + match.group(2) + end
{% endhighlight %}

Die Animation wird dadurch um den Faktor 1,5 langsamer.