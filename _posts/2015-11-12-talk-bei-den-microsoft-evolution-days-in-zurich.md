---
id: 807
title: Talk bei den Microsoft Evolution Days in Zürich
date: 2015-11-12T21:42:44+00:00
author: Fabian Gosebrink
layout: post
tags: angular angularjs rest restapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hallo zusammen,

am 29. Oktober war ich zu Gast bei den Microsoft Evolution Days, die Digicomp ausgerichtet hat. Ich habe einen <span id="snippet_meta" class="desc" contenteditable="true">Talk bei den Microsoft Evolution Days in Zürich</span> über AngularJs & REST mit einer ASP.NET WebAPI gehalten. Die Folien gibts direkt bei [Digicomp](http://news.digicomp.ch/de/2015/11/10/microsoft-evolution-day-2015-rueckblick-und-slides-zum-download/). Oder direkt [hier](http://static.news.digicomp.ch/1447166270/track-development.zip). Alle Beispiele gibts auch auf [Github](https://github.com/FabianGosebrink).

<span style="color: #999999;">Mein erster Vortrag als MVP 🙂</span>

Früher hatten die Evolution Days keinen Developer-Track. Dieses Jahr war aber einer dabei und ich bin froh, dass ich eingeladen wurde. Zusammen mit [Mark Allibone](https://twitter.com/mallibone) und Hanspeter Bornhauser habe ich einen von insgesamt 5 Vorträgen im Developertrack gehalten.

Das komplette Programm gibts [hier](https://www.digicomp.ch/media/misc/MicrosoftEvolutionDay-2015-DigicompZuerich-29.10.pdf).

<blockquote class="twitter-tweet" lang="de">
  <p dir="ltr" lang="en">
    .<a href="https://twitter.com/FabianGosebrink">@FabianGosebrink</a> about <a href="https://twitter.com/hashtag/angular?src=hash">#angular</a> and <a href="https://twitter.com/hashtag/webapi?src=hash">#webapi</a> at the <a href="https://twitter.com/hashtag/msevolutionday?src=hash">#msevolutionday</a> from <a href="https://twitter.com/DigicompCH">@digicompch</a> /cc <a href="https://twitter.com/Noser_Eng">@Noser_Eng</a> <a href="https://t.co/T73OdWRuiB">pic.twitter.com/T73OdWRuiB</a>
  </p>
  
  <p>
    — Mark Allibone (@mallibone) <a href="https://twitter.com/mallibone/status/659736379954495488">29. Oktober 2015</a>
  </p>
</blockquote>



Natürlich bedanke ich mich bei Digicomp und Microsoft, dass ich auf dem Event reden durfte. Es hat Spass gemacht. Gerne wieder 🙂

Hier ein Auszug aus den Folien:

![Talk bei den Microsoft Evolution Days in Zürich]({{site.baseurl}}assets/articles/2015-11-12/f502e543-418a-4a9f-9d0d-14b2188a554b.png)

Ich denke es ist wichtig klarzustellen, dass "REST" nicht gleich "WebAPI" ist. REST ist eine Form, die ein Web-Service erfüllen kann. ASP.NET WebAPI ist eine Technologie, mit der man solche Services umsetzen kann. Das ist ein fundamentaler Unterschied.

![Talk bei den Microsoft Evolution Days in Zürich]({{site.baseurl}}assets/articles/2015-11-12/1db18db9-1a0b-444f-9b8e-45aa287c1d26.png)

Hier beschreibe ich eine klassische REST-API. Das PATCH-Attribut habe ich auf der nächsten Folie. Es ging also nicht vergessen. Jeder Link stellt eine eindeutige Aktion dar. Alle CRUDOperationen sind durch diese Links für eine Entität abgedeckt.

![Talk bei den Microsoft Evolution Days in Zürich]({{site.baseurl}}assets/articles/2015-11-12/94354840-eecb-4a60-a304-20466e95def1.png)

Diese Folie beschreibt wohl das, was AngularJS so mächtig macht. Das Framework bietet einen clientseitige Architektur mit Javascript (oder Typescript), die es ermöglicht komplexe Web-Apllikationen zu bauen.

![Talk bei den Microsoft Evolution Days in Zürich]({{site.baseurl}}assets/articles/2015-11-12/397b01a3-83d9-4cc1-95bf-506ca0979654.png)

"$scope" ist wohl das Wort, was bei vielen ein Erleichtern auslöst. Es entspricht etwa dem ViewModel, welches man schon von anderen Applikationsarten kennt. Hierauf kann man im AngularJS Funktionen und Werte definieren, die man dann von der View aus via DataBinding konsumieren kann.

![Talk bei den Microsoft Evolution Days in Zürich]({{site.baseurl}}assets/articles/2015-11-12/6397f920-336f-4f84-ba3b-8812a7be5c3e.png)

Bis dahin

Fabian
