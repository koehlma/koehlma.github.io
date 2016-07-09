---
layout: post
title: "German: Jaspy – Einführung"
---
Dies ist der erste Artikel einer ganzen Reihe, die sich mit dem Schreiben eines Python Interpreters in JavaScript beschäftigt. Ziel des Ganzen ist es, nicht mehr länger auf JavaScript angewiesen zu sein, sondern stattdessen auch im Browser Python verwenden zu können. Außerdem erhoffe ich mir so, vieles über den internen Aufbau von Python und Interpretern im Allgemeinen zu lernen – ein sehr interessantes Thema, wie ich finde.

Auf die Idee dazu brachte mich [PyPy.js](http://pypyjs.org/), eine mit [Emscripten](http://kripken.github.io/emscripten-site/) von C nach JavaScript übersetzte Version des beliebten PyPy Interpreters. Leider ist dessen VM jedoch bereits über 6MB groß. Damit scheidet es als Ersatz für JS praktisch schon aus. Es gibt allerdings noch eine Reihe ähnlicher Projekte, die versuchen Python in den Browser zu bringen. Besonders hervorgehoben sei an dieser Stelle [Brython](http://www.brython.info/). Brython ist eine Python 3 kompatible Python Implementierung, die den Python Code zunächst nach JavaScript übersetzt und das Resultat dann ausführt.

Alle diese Implementierungen haben jedoch eines gemeinsam, sie unterstützen kein kooperatives Multitasking in Form von Koroutinen oder [Greenlets](https://greenlet.readthedocs.org/en/latest/). Beides Features, die ich gerne im Browser zur Verfügung hätte, deren Implementierung in JavaScript allerdings nicht trivial ist und die auch schwierig auf bestehende Implementierungen aufzusetzen sind. Daher beginne ich noch einmal komplett von neuem mit Jaspy, ein Python 3 kompatibler Python Interpreter in JavaScript, der kooperatives Multitasking unterstützen und zudem einfach mit nativen JavaScript Modulen erweiterbar sein soll.

Zum [zweiten Artikel]({% post_url 2016-03-09-german-jaspy-code-objekte %}).