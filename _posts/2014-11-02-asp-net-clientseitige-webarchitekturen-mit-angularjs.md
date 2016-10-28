---
id: 557
title: 'ASP.NET - Clientseitige Webarchitekturen mit AngularJS'
date: 2014-11-02T07:05:39+00:00
author: Fabian Gosebrink
layout: post
tags: angular angularjs entityframework sample solution webarchitecture asp.net
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

ASP.NET MVC hat, wie der Name schon sagt, die MVC-Struktur auf dem Server etabliert und es ist somit möglich saubere Architekturen auf dem Server zu erstellen.

Mehr und mehr geht die Richtung jedoch hin zu Mobile-First-Implementierungen. Hierbei spielt der Client, dabei meine ich die Tatsache, _dass_ es ein Client ist, nicht welche Art von Client (Handy, Tablet etc.), eine immer grössere Rolle.

Wir tragen heute Rechner in unseren Hosentaschen, die stärker sind als die Desktop-Rechner vor 5 Jahren und man findet Rechenpower sehr viel schneller als zu damaliger Zeit. Tablets, Handys, Phablets und und und sind internetfähig und die Seiten müssen mobile Ansichten einfach in den Fokus stellen. MediaQueries tun ihre Arbeit, aber moderne WebApplikationen müssen auch fähig sein, dem Benutzer die Usability einer mobilen App zu geben. Die Arbeit findet also auf dem Client, statt auf dem Server, statt. Aufgrund der Leistung ist dies ohne Probleme möglich. Aber je mehr Arbeit auf dem Client getan werden muss, desto mehr muss auch bei dem Erstellen der Appltikation clientseitig implementiert werden. Dies geht nur mit einer testbaren und wartbaren Struktur und Architektur.

AngularJS bietet neben der Implementierung mit Javascript auch noch die Möglichkeit eine Clientseitige geordnete Architektur, im MV*-Stil aufzuziehen, um auch grössere Anforderungen an Webseiten geordnet abzubilden.

Durch die Tatsache, dass Angular uns Dependency-Injection out-of-the-box mitliefert gewinnen wir schon eine sehr lose Kopplung der einzelnen Module, die sich sehr schön und passend zusammensetzen lassen.

Im Folgenden möchte ich eine beispielhafte Architektur aufzeigen und erläutern. Diese ist nicht in Stein gemeisselt, ist aber sicher ein guter Anfang für Projekte.

Die Struktur bzw Architektur der Client-Applikation liegt in einem Verzeichnis „app“ im root-Verzeichnis der Anwendung.

![ASP.NET - Clientseitige Webarchitekturen mit AngularJS]({{site.baseurl}}assets/articles/2014-11-02/1a7892fc-a371-4a95-a227-00ad58930f3e.png)

In ihr enthalten ist ein Ordner für Bilder und Scripts, einer für Styles (css-Dateien) und für die Views. Der Skript-Ordner beinhaltet die eigentliche Applikation:

![ASP.NET - Clientseitige Webarchitekturen mit AngularJS]({{site.baseurl}}assets/articles/2014-11-02/df98c920-8500-43dd-9a78-f1fbf9205475.jpg)

Hier kann man auch schon die clientseitige Architektur bzw. deren Ansatz erkennen: der controllers-Folder bildet den „Namespace“ für Controller ab, der die gleiche Rolle spielt wie in ASP.NET-Anwendungen auch: Er nimmt die Anfragen vom UI entgegen und verarbeitet diese. Dazu arbeitet er mit dem Viewmodel, dass in Angular „$scope“ getauft wurde.

Die Services bieten eine Abstrahierung von etwaigen Aufgaben. Hier können Business-Services weggekapselt werden, die ihre eigenständigen Aufgaben haben. Auch Repositories sind denkbar. Auch in einem eigenen Namespace, wenn dies gewünscht ist. Durch die Dependency-Injection wäre das Aufteilen in verschiedene Klassen und Namespaces kein Problem.

App.js bietet uns den Start unserer Anwendung. Hier wird die App erstellt und einer Variable zugewiesen, auf der die Controller, Services etc. in Zukunft registriert werden.

<pre><code class="javascript">var firstApp = angular.module('firstApp', ['ngRoute', 'ngResource', 'ui.bootstrap']);</code></pre>



Welche Module hier noch hinzugefügt wurden ist im Moment völlig irrelevant. Wichtig ist die Variable „firstApp“, die uns noch weiter begegnen wird im Laufe dieses Blogposts.

Die Controller bieten nun die Möglichkeit, direkt mit dem Viewmodel zu arbeiten. Er setzt alle Properties auf dem Viewmodel und bietet der View genau, und nur genau das, was sie zum Anzeigen braucht. Man kann (und sollte) beispielsweise auch Methoden auf dem Scope registrieren, die der Benutzer mit einem Klick ausführen kann. Der Scope bietet der View alles, was sie zum Funktionieren braucht.

Der Controller versorgt den Scope und empfängt seinerseits Daten aus einem Repository, einem Service etc. Die Datenquelle kann beliebig sein, eben auch eine REST-Schnittstelle. Hierbei würde wiederum ASP.NET WebAPI zum Zuge kommen können.

![ASP.NET - Clientseitige Webarchitekturen mit AngularJS]({{site.baseurl}}assets/articles/2014-11-02/6f9c209d-9714-417a-a602-eb96715b6f2d.png)

Die Controller bei der Beispiel-Todo-App im Anhang befinden sich in einem seperaten Namespace „controllers“, die Services dazu in einem Namespace „services“.

![ASP.NET - Clientseitige Webarchitekturen mit AngularJS]({{site.baseurl}}assets/articles/2014-11-02/d6159b2a-f5fd-4dd5-98f7-03c153b31b4e.png)

Hierbei kommen die oben genannten Aufgaben zum Tragen.

Der Todo-Service beispielsweise bietet die Funktionen zum Abrufen, Löschen und Hinzufügen an:

<pre><code class="javascript">'use strict';
firstApp.factory('todoService', function ($http) {
 
    var todoService = {};
 
    var urlPrefix = '/api/Todo/';
 
    var _addTodo = function (todoName) {
        var data = { Name: todoName };
        var promise = $http.post(urlPrefix + 'AddTodoItem', data);
        return promise;
    };
 
    var _deleteTodo = function (item) {
        var promise = $http.post(urlPrefix + 'RemoveTodoItem', item);
        return promise;
    };
   
    var _getTodoItems = function () {
        var promise = $http.get(urlPrefix + 'GetAllTodoItems').then(function (results) {
            //console.log(results);
            return results.data;
        });
        return promise;
    };
 
    todoService.getTodoItems = _getTodoItems;
    todoService.addTodo = _addTodo;
    todoService.deleteTodo = _deleteTodo;
 
    return todoService;
});
</code></pre>



Durch die Registrierung auf der App-Variable „firstApp“ steht nun die Dependency Injection zur Verfügung, die es möglich macht, den Service im Controller zu injecten. Der Controller kann nun die Methoden in Anspruch nehmen.

<pre><code class="javascript">firstApp.controller('todoController', function ($scope, todoService) {
 
    var _addTodo = function () {
        todoService.addTodo($scope.TodoItem).then(
            function () {
                _getTodoItemAndSetOnScope();
            },
            function () {
                alert("Error occured");
            });
    };
 
    var _deleteTodo = function (item) {
        todoService.deleteTodo(item).then(function () {
                _getTodoItemAndSetOnScope();
            });
    };
 
    var _getTodoItemAndSetOnScope = function () {
        todoService.getTodoItems().then(function (result) {
            $scope.todoItems = result;
        });
    };
 
    _getTodoItemAndSetOnScope();
 
    $scope.TodoItem = "";
    $scope.AddTodo = _addTodo;
    $scope.DeleteTodo = _deleteTodo;
});
</code></pre>



Er setzt die Informationen auf dem scope und gibt diese zurück an die View. Diese bekommt per

<pre><code class="javascript">ng-controller="todoController"</code></pre>



den Controller mitgeteilt und kann so das konsumieren, was der Controller auf dem Scope setzt. Beispielsweise:

<pre><code class="javascript">&lt;tbody ng-repeat="item in todoItems"&gt;</code></pre>



Hierbei ist „todoItems“ eine Liste aller vorhandenen Items.

Die Serverseitige WebAPI lässt sich kurz beschreiben. Hierbei gebe ich gern die Action direkt mit. Das ist aber nur eine Vorliebe meinerseits. Dies muss nicht so gelöst werden. Es kann auch mit Attributen der ActionName verändert werden, so dass dies „on-the-fly“ passiert.

<pre><code class="cs">public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services
            config.Formatters.Clear();
            config.Formatters.Add(new JsonMediaTypeFormatter());

            // Web API routes
            config.MapHttpAttributeRoutes();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{action}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }</code></pre>



Gearbeitet wird in der Beispiel-Solution mit dem Code-First-Ansatz, der eine richtige Konfiguration des MSSQL-Servers voraussetzt.

<pre><code class="cs">public class TodoController : ApiController
    {
        private readonly ITodoRepository _todoRepository;

        public TodoController()
        {
            _todoRepository = new TodoRepositoryImpl(new DatabaseContext());
        }

        [HttpPost]
        public void AddTodoItem(TodoItem todoItem)
        {
            using (_todoRepository)
            {
                todoItem.Added = DateTime.Now;
                _todoRepository.Add(todoItem);
                _todoRepository.Save();
            }
        }

        [HttpGet]
        public List&lt;TodoItem&gt; GetAllTodoItems()
        {
            using (_todoRepository)
            {
                return _todoRepository.GetAll();
            }
        }

        [HttpPost]
        public void RemoveTodoItem(TodoItem todoItem)
        {
            using (_todoRepository)
            {
                TodoItem findSingle = _todoRepository.FindSingle(todoItem.Id);

                if (findSingle != null)
                {
                    _todoRepository.Delete(todoItem.Id);
                    _todoRepository.Save();
                }
            }
        }
    }</code></pre>

Und das dazugehörige Repository:

<pre><code class="cs">namespace AngularJsTemplate.Repositories.TodoRepository
{
    public interface ITodoRepository : IRepositoryContext&lt;TodoItem&gt;
    {
         
    }
}</code></pre>



<pre><code class="cs">namespace AngularJsTemplate.Repositories.TodoRepository.Impl
{
    public class TodoRepositoryImpl : RepositoryContextImpl&lt;TodoItem&gt;, ITodoRepository
    {
        public TodoRepositoryImpl(DbContext databaseContext) 
            : base(databaseContext)
        {
        }
    }
}</code></pre>

Benutzt wurde hier mein Repository und UnitOfWork-Paket, was auch auf <a href="http://www.nuget.org/packages/OfferingSolutions.UnitOfWork.Structure/" target="_blank">NuGet </a>zu finden ist.



Gruss



Fabian

[AngularJsTemplate](http://offering.solutions/wp-content/uploads/2014/10/AngularJsTemplate.zip)
