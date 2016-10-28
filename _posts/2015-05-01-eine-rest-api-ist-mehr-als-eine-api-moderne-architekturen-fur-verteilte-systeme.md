---
id: 657
title: Eine REST API ist mehr als eine API – Moderne Architekturen für verteilte Systeme
date: 2015-05-01T17:35:37+00:00
author: Fabian Gosebrink
layout: post
tags: api restapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In diesem Blogpost möchte ich zeigen, wie wichtig eine API heutzutage ist. Frei nach dem Motto: Eine REST API ist mehr als eine API – Moderne Architekturen für verteilte Systeme.

In der heutigen Zeit ist es absolut unabdingbar eine Software, die man für internen oder externen Gebrauch schreibt, auf mehreren Wegen verfügbar bzw. konsumierbar zu machen. Wichtig sind hierbei sicherlich Desktop-PCs, Smartphones und Tablets.

Völlig unabhängig der eingesetzten Technologie (ASP.NET WebAPI, AngularJS, Xamarin, ASP.NET MVC, etc.) möchte ich mit diesem Blog einen Schritt zurückgehen und den Blick auf die Architektur lenken, die moderne Software-Lösungen voranbringen können. Hierbei werde ich zwar auch Technologien als Beispiel erwähnen, diese sind aber nur als Beispiel zu betrachten. Die Lösungen in dieser Hinsicht sind sicherlich vielfältiger als das hier geschriebene.

### _Die ganze Welt schreibt Apis_

Trotz dem gemeinsamen Fakt, dass vom Kunden maximale Verfügbarkeit gefordert wird, sind Lösungen in diesem Bereich sehr individuell anzusehen. Kunde A braucht eine Lösung in einer bestimmten Richtung, für Kunde B ist wiederum etwas anderes interessant, etc.

Was viele gemeinsam haben ist jedoch, dass sie Daten aus einer bestimmten Quelle beziehen und ablegen und auf dem anderen Ende anzeigen müssen. Hierbei verwendet man heutzutage REST-APIs. Auf einer Entwicklerkonferenz habe ich ein Zitat aufgeschnappt, was mich nicht mehr loslässt:

> „Die ganze Welt schreibt APIs“

Egal mit welcher Lösung man auf die Datensenke zugreift, also welche Anwendung der Kunde wünscht, ist es doch das Beste, man hat ein einheitliche Schnittstelle, gegen die man implementieren kann, die alle Operationen zur Verfügung stellt, die der Kunde braucht. Hierbei rede ich nicht nur von Daten, die abgerufen und abgelegt werden können. Hierbei geht es auch um Sortierungen, Paging, Datashaping, Autorisierung, Authentifizierung und und und. Moderne APIs stellen so etwas standardmässig zur Verfügung. Je nachdem ob der Client dies benutzen muss oder nicht: Die Funktionalität sollte auf jeden Fall implementiert sein. Wer weiss schon, welcher Client mit meiner Api kommuniziert? Die API sollte auf einem einheitlichen Standard basieren. Hier bietet sich der Standard an, der sich seit Jahren durchgesetzt hat: REST. Die API kann Http-Calls interpretieren und wie gewünscht auch mit einer entsprechenden Inhalt antworten.

Nehmen wir als Beispiel den Kurznachrichtendienst Twitter. Twitter ist ein Unternehmen, das moderne Architekturen implementiert, fördert und lebt. Es gibt unendlich viele Twitter Clients für nahezu jede (!) Plattform. Diese kommunizieren alle gegen die Twitter API, die ein und dieselbe Funktionalität zur Verfügung stellt. Es kann REST-Calls interpretieren und ist somit in jede Richtung offen.

Die Implementierung der API, auch im Twitter-Fall, erfolgt nach den Regeln von HTML. Das REST-Prinzip stellt hierbei alle Funktionen zur Verfügung, die ich brauche und ist ein Standard, den alle Endsysteme ausnahmslos sprechen. Ob Android, MVC, iOS, Windows: HTML bzw. die HTML-Verben sind Sprach, System und Plattformunabhängig.

![Eine REST API ist mehr als eine API – Moderne Architekturen für verteilte Systeme]({{site.baseurl}}assets/articles/2015-05-01/0f47f393-303c-492a-807d-3a973312a440.png)

Somit ist eine modern entwickelte API das A und O einer modernen Architektur. Cloud-Systeme wie Microsofts Azure machen es einfacher denn je solche Systeme zu hosten. Um eine standardisierte Sprache zwischen den Clients und der API einzusetzen hat sich JSON durchgesetzt und hält auch in modernen Technologien (ASP.NET 5, Bower, …) auch als Konfigurationsmodell Einzug.

Vor dem Hintergrund ist nun die einzige Anforderung an einen Client nur noch, dass er online http-Calls absetzen kann. Clientseitiges Caching sorgt hingegen dafür, dass die Applikation auch offline benutzt werden kann. Synchronisierungen finden natürlich nur im Online-Zustand statt. Aber wer ist denn heutzutage schon lange offline 😉

Die Clients könne vielfältig und in verschiedenen Sprachen entwickelt worden sein.

<p style="padding-left: 30px;">
  <em>Web: Asp.Net MVC, JavaScript (bspw. AngularJS)</em>
</p>

<p style="padding-left: 30px;">
  <em>Mobile: Xamarin, Native, AngularJS</em>
</p>

<p style="padding-left: 30px;">
  <em>Desktop: WPF, AngularJS (Node WebKit)</em>
</p>

![Eine REST API ist mehr als eine API – Moderne Architekturen für verteilte Systeme]({{site.baseurl}}assets/articles/2015-05-01/4cae52dd-fca1-414f-ade4-96ca54b84977.png)

Eine REST-Api kann also, unabhängig vom Client, Daten und Datenoperationen zur Verfügung stellen. Sie sollte heutzutage mehr bieten als nur GET und POST-Methoden, sondern auch Paging, Sorting, Datashaping etc. unterstützen um wirklich eine flexible und fähige API zu bieten. Mit solch einer REST-API im Hintergrund können für den Kunden individuelle Lösungen entwickelt werden, die alle letztendlich die gleiche Datensenke konsumieren.
