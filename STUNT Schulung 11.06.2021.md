# Projekt 'Todo-Liste' der Schulung 'Einführung in die Webentwicklung'

## Scope des Inhaltes

Dieses Skript enthält die wichtigsten Informationen und Code-Snippets, um die Basisfunktionen der in der Schulung vorgestellten Todo - App zu gewährleisten.  

## Verwendete Tools

1. Hosting durch [Digital Ocean](https://m.do.co/c/ec1b24286026) 1GB VM Droplet

2. Datenbank und API mit Entwicklungsversion von [Strapi](https://strapi.io/)

3. [Materializecss](https://materializecss.com/) für die Darstellung der Todo - Liste

4. [Visual Studio Code](https://code.visualstudio.com/) und das Plugin [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)

5. [Miro](https://miro.com/) zur Visualisierung der Datenstruktur & Nutzeroberfläche

## Annahmen des Benutzers

1. Ist Teilnehmer der Schulung oder

2. hat Basiskenntnisse in der Webentwicklung mit Javascript

> Du musst nicht jede Zeile Code verstehen, die hier verwendet wird. Es handelt sich um ein Projekt zur Einführung. Trotzdem werden einige Techniken angewendet, die nicht auf Einsteigerniveau sind. 
> 
> Manchmal musst du rennen, bevor du krabbeln lernst. Genauso ist es hier.

## Das Projekt : Setup des Servers

Alternativ zu der hier vorgestellten Lösung kannst du auch eine Umgebung mit Docker erstellen. Dazu ist allerdings ein umfänglicheres Setup notwendig. In dieser Schulung nutzen wir Strapi auf DO, damit jeder Teilnehmer auf dieselbe Datenquelle zugreifen kann.

Zur Installation von Strapi mit Docker: [Offizielle Dokumentation](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/installation/docker.html) 

### Schritt 1: Erstelle ein neues Droplet für 5$

- Öffne deine Projektseite von Digitalocean

- Klicke auf 'Create', um ein neues Droplet zu erstellen

- Wähle den Plan: 
  
  - **Distribution**: Ubuntu 20.04 LTS
  
  - **Shared CPU - Basic**
  
  - **CPU Options**: Regular Intel with SSD, 1GB & 1 CPU
  
  - **Datacenter Region**: Frankfurt
  
  - **Authentication**: Password oder, falls schon mit SSH vertraut, SSH keys

- Bestätigung - warte, bis das Droplet verfügbar ist.

### Schritt 2: Login über OpenSSH

- Erhalte die IP - Adresse des Droplets

- Dann kannst du dich per [OpenSSH]([Raspberry Pi: SSH einrichten - so geht&#39;s](https://www.heise.de/tipps-tricks/Raspberry-Pi-SSH-einrichten-so-geht-s-4190645.html)) mit dem Benutzernamen 'root' und deinem gewählten Passwort anmelden

```shell
$ ssh root@<ip-adresse>
```

- Es ist dann geläufige Praxis, die folgenden Dinge zu tun:
  
  - Einen neuen Nutzer anzulegen und diesen für alle weiteren Schritte zu verwenden
  
  - Ein Systemupdate zu machen

- Das Skript in Schritt 3 übernimmt diese beiden Aktionen

```shell
# Lege neuen Nutzer an und füge ihn zur Admingruppe hinzu
$ adduser node_todoapp_stunt_admin 
$ usermod -aG sudo node_todoapp_stunt_admin

# Führe ein Systemupdate aus
$ apt update && apt upgrade
```

### Schritt 3: Kickstarte ein neues Strapi Projekt

Das Folgende Skript installiert Nginx und Strapi. 

- Strapi ist ein [Headless Content Management System]([Headless content management system - Wikipedia](https://en.wikipedia.org/wiki/Headless_content_management_system)). Hier werden wir die Daten für unsere Todos speichern

- Nginx ist ein hochperformanter Webserver und wird häufig als [Reverse-Proxy](https://www.cloudflare.com/en-gb/learning/cdn/glossary/reverse-proxy/) verwendet. Er verhält sich wie ein 'Gateway' zu unserer App

```shell
#!/bin/bash
HOST="<ip-adresse>"
ROOT_USERNAME="root"

USER_CREATE_SCRIPT="adduser node_todoapp_stunt_admin && usermod -aG sudo node_todoapp_stunt_admin"
NGINX_INSTALL_SCRIPT=""
NODE_INSTALL_SCRIPT="sudo apt update -y && sudo apt upgrade -y && sudo apt install nodejs -y && sudo apt install npm -y && sudo apt install nginx && node -v && nginx -v"
STRAPI_QUICKSTART_SCRIPT="npx create-strapi-app stunt-schulung --quickstart"

# Exec user creation
echo "Creatinging user node_todoapp_stunt_admin ..." 
ssh -l ${ROOT_USERNAME} ${HOST} "${USER_CREATE_SCRIPT}"

# Exec node install
echo "Installing Nodejs ..."
ssh -l ${ROOT_USERNAME} ${HOST} "${NODE_INSTALL_SCRIPT}"

# Exec strapi quickstart
echo "Setting up strapi quickstart project ... "
ssh -l ${ROOT_USERNAME} ${HOST} "${STRAPI_QUICKSTART_SCRIPT}"

echo "Restarting droplet. Server available under /root/stunt-schulung"
echo "Start script is 'npm run develop'"
echo "Remember to remove authentication under 'Settings > Roles > Todo > Select All'"
ssh -l ${ROOT_USERNAME} ${HOST} "reboot"
```

### Schritt 4: (Optional) Nginx Konfiguration

Einen Webserver zu konfigurieren ist nicht Teil der Schulung. Es gibt viele gute Kurse auf Udemy, z.B. [NGINX Fundamentals: High Performance Servers from Scratch](https://www.udemy.com/share/101X7qBEEdcF9RRHg=/)

Der folgende Code wird die Standardkonfiguration von Nginx überschreiben und uns erlauben, über eine Schnittstelle auf Strapi zuzugreifen.

```shell
# Login per SSH auf dem Remote Rechner
$ ssh root@<ip-adresse>
$ cd /etc/nginx/sites-available
$ nano default
```

In der sich dann öffnenden Text-file musst du 

```shell
location / {
    try_files $uri $uri/ = 404; 
}
```

durch folgendes ersetzen: 

```shell
location / {
    proxy_pass "http://localhost:1337"; 
}
```

Auf diese Weise werden eingehende Anfragen auf die IP Adresse intern auf die Strapi App umgeleitet. Die App selbst läuft auf [Port]([Port (Protokoll) – Wikipedia](https://de.wikipedia.org/wiki/Port_(Protokoll))) 1337.

> Nginx ist ein mächtiges Tool in der modernen Webentwicklung. Allerdings gibt es auch sogenannte Serverless - setups. Hierbei übernimmt ein Dienstleister die Bereitstellung von Serverarchitektur und allem, was dazu gehört. So kann sich ein Entwicklungsteam auf das Wesentliche konzentrieren - Webapps zu entwickeln.

### Schritt 5: Zugriff auf Strapi

- Nach dem Server Neustart ist ein neues Verzeichnis im root - Ordner vorhanden ODER, falls ein neuer Nutzer angelegt wurde, unter /home/`<nutzername>`/stunt-schulung

- Mit dem folgenden Befehl kannst du jetzt Strapi starten

```shell
# Wechsele in das Arbeitsverzeichnis
$ cd /root/stunt-schulung
# cd /home/<username>/stunt-schulung

# Starte Strapi im Entwicklungsmodus
$ npm run develop
```

Strapi wird dann unter der IP-Adresse des Droplets erreichbar sein, entweder

- Mit Nginx unter `<ip-adresse>/admin`

- Ohne Nginx unter `<ip-adresse>:1337/admin`

Sollte hier die Meldung `502 - Bad Gateway` angezeigt werden, musst du noch folgendes tun: 

```shell
# Auf dem Cloud Rechner per ssh: 
$ sudo chown -R www-data:www-data /var/lib/nginx
```

Durch diesen Befehl erhält der www-data - Nutzer, den Nginx verwendet, Zugriff auf die relevanten Verzeichnisse, die er zum Ausführen des Reverse-Proxy benötigt.

**Damit ist die Konfiguration des Backends abgeschlossen. Als nächstes müssen wir noch Strapi so konfigurieren, dass wir Todos anlegen, abrufen und ändern können.**

### Schritt 6: Todo - Content-Typ anlegen

- Content Typen beschreiben und klassifizieren Inhalte

- In unserem Fall möchten wir als Inhalt eine Todo - Liste anlegen

- Ähnlich wie Blogs oder Websites bei Wordpress müssen wir diese erst beschreiben, bevor wir sie anlegen

- Was ist also Teil einer Todo - Liste? Klar, Todos.
  
  Ein einzelnes Todo lässt sich beispielhaft so beschreiben:
  
  - **Titel** - Ziel oder Inhalt des Todos
  
  - **Beschreibung** - Zusätzliche Informationen
  
  - **Erledigt** - Ja oder nein?

- In Strapi können wir diese Beschreibung nun in einen Content Typ überführen

- Klicke dazu auf 'Create  your first Content Type'

- Gib dann 'todos' als Display Name ein

- Nun kannst du diesem mehrere benannte Felder zuordnen, die ihn beschreiben

- Wir nehmen die Beschreibung von oben: 
  
  > - TItel = Text
  > 
  > - Beschreibung = Rich Text
  > 
  > - Erledigt = Boolean

- Bestätige dann die Erstellung, indem du auf 'Save' klickst.

- Nach dem Neustart wird der Content Typ von Strapi verwaltet und ist somit von außen ansprechbar

### Schritt 7: Auf die todo Resource zugreifen

- Da Strapi ein Headless - CMS ist, können Inhalte von überall über die jeweilige URL abgerufen werden

- Im Fall unserer Todo - Liste ist dies `http://<ip-adresse>/todos`

- Dieser Endpunkt ist standardmäßig nur für angemeldete Nutzer sichtbar. 

- Wir wollen allerdings keinen separaten Login, sondern nur auf die Daten zugreifen

- Dafür müssen wir eine Nutzerrolle ändern, die von Strapi verwaltet wird, nämlich public

- Dazu kannst du unter `'Settings' > 'Roles' > 'Public'` unter dem Reiter `'Permissions' > 'Applications'` alle Interaktionen für die Öffentlichkeit freischalten, indem du auf die Checkbox 'Select all' klickst und dann speicherst.

- Wenn du erneut versuchst, über die IP - Adresse auf die Resource zuzugreifen, wirst du eine andere Antwort erhalten

- Damit ist die Konfiguration von Strapi abgeschlossen.

## Das Projekt: Setup der Webapp

Bevor du anfängst, die Webapp zu entwickeln, nimm dir einen Moment, sie grafisch darzustellen. Einfach drauf loszuprogrammieren macht Spaß, du läufst aber damit Gefahr, dich während des Programmierens in Kleinigkeiten und Feinheiten zu verlieren. 

> Miro bietet einige praktische Features, um sogenannte Wireframes zu erstellen. Damit kannst du das Layout der App - und auch, in einem gewissen Rahmen, ihr Aussehen - skizzieren

Beispiel: [Miro | Online Whiteboard for Visual Collaboration](https://miro.com/app/board/o9J_kr4JAG8=/) (nur temporär verfügbar)

![Todo-Wireframing.jpg](https://github.com/tq-bit/stunt-schulung-todo-app/blob/main/Todo-Wireframing.jpg?raw=true)

### Start Template in HTML

Um die Nutzeroberfläche abzubilden brauchst du eine HTML Struktur. In unserem Fall wird diese recht einfach. Da wir ein CSS Framework nutzen, können wir uns auf die wesentlichen Elemente beschränken und alles weitere dynamisch mit Javascript hinzufügen

> Startseite Materialize CSS: [Getting Started - Materialize](https://materializecss.com/getting-started.html)

Such dir zunächst einen Ordner aus, in dem du dein Projekt speichern möchtest. Lege dann 3 Dateien an mit den folgenden Namen: 

- index.html -> **H**yper**T**ext **M**arkup **L**anguage beschreibt die Struktur deiner  Website

- styles.css -> **C**ascading **S**tyle**S**heets beschreiben das Aussehen deiner Website

- main.js -> **J**ava**S**cript erweckt deine Website zum Leben und macht sie dynamisch

Schreibe dann folgendes in die index.html Datei: 

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STUNT Schulung Todo liste</title>

    <!-- Importiere Materialize CSS in deine Website -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">

</head>

<body>

    <nav>
        <div class="nav-wrapper green darken-4">
            <a href="/" class="brand-logo center">Todo - Projekt</a>
        </div>
    </nav>

    <!-- Platzhalter für 'Todo-Übersicht' -->

    <!-- Platzhalter für 'Neues Todo anlegen' -->

    <!-- Platzhalter für 'Bestehendes Todo ändern' -->

    <!-- Importiere Materialize Javascript in deine Website -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
</body>

</html>
```

### Todo Übersicht

Bevor wir neue Todos anlegen, sollten wir eine Möglichkeit haben, diese anzuzeigen. Fangen wir also mit der Todo - Übersicht an. 

Für die abgebildeten Todos nutzen wir zunächst nur einen Platzhalter, um einen Eindruck vom Aussehen zu bekommen. 

Füge folgendes in deine `index.html` Datei bei 'Todo-Übersicht' ein

```html
    <main class="container mt-2">
        <div class="row">
            <div class="col m2 lg3"></div>
            <div class="col s12 m8 lg6">
                <!-- Suchleiste und Button für neues Todo -->
                <div class="row">
                    <div class="input-field col s9">
                        <i class="material-icons prefix">search</i>
                        <input id="input-search" type="text">
                        <label for="input-search">Suche nach Todos ...</label>
                    </div>
                    <div class="input-field col s3">
                        <button data-target="modal-new-todo" id="button-new-todo" class="btn waves-effect waves-light green darken-4 w-full">Add new Todo
                            <i class="material-icons left">add</i>
                        </button>
                    </div>
                </div>
                <!-- /Suchleiste und Button für neues Todo -->

                <!-- Todo Übersicht -->
                <div class="row">
                    <ul class="collection">
                        <li class="collection-item">
                            <p>
                                <label>
                                    <input type="checkbox" />
                                    <span>Wäsche machen</span>
                                </label>
                            </p>
                        </li>
                        <li class="collection-item">
                            <p>
                                <label>
                                    <input type="checkbox" />
                                    <span>Das Haus aufräumen</span>
                                </label>
                            </p>
                        </li>
                        <li class="collection-item">
                            <p>
                                <label>
                                    <input type="checkbox" checked="checked" />
                                    <span>Einkaufen und kochen</span>
                                </label>
                            </p>
                        </li>
                    </ul>
                </div>
                <!-- /Todo Übersicht -->
            </div>
            <div class="col m2 lg3"></div>
        </div>
    </main>
```

Füge außerdem folgendes zu deiner styles.css Datei hinzu: 

```css
.mt-2 {
  margin-top: 2rem;
}

.w-full {
  width: 100%;
}
```

### Neues Todo anlegen: Modal und Formular

Bevor wir eine Liste abbilden können, müssen neue Todos anlegen. Das funktioniert traditionell über HTML - Formulare. Materialize bietet uns einige vorgefertigte Elemente, die wir nutzen können, unter anderem Modals - Dialogelemente, die dynamisch Ein - und ausblendbar sind.

Füge folgendes in deine `index.html` Datei bei 'Neues Todo anlegen' ein 

```html
    <!-- Modal für neues Todo -->
    <div id="modal-new-todo" class="modal">
        <div class="modal-content">
            <h4>Neues Todo anlegen</h4>
            <form id="form-new-todo">
                <div class="row">
                    <div class="input-field col s12">
                        <input id="input-new-todo-title" type="text" class="validate">
                        <label for="input-new-todo-title">Titel</label>
                    </div>

                    <div class="input-field col s12">
                        <textarea id="input-new-todo-description" class="materialize-textarea"></textarea>
                        <label for="input-new-todo-description">Beschreibung</label>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" id="button-new-todo-close" class="btn waves-effect waves-light red darken-4">Cancel
                        <i class="material-icons left">cancel</i>
                    </button>
                    <button type="submit" id="button-new-todo-confirm" class="btn waves-effect waves-light green darken-4">Create
                        <i class="material-icons left">add</i>
                    </button>
                </div>
            </form>
        </div>
    </div>
    <!-- /Modal für neues Todo -->
```

### Bestehendes Todo ändern

Da sich beide Felder von oben im Änderungsdialog wiederholen können wir die Struktur von oben wiederverwenden

```html
    <div id="modal-edit-todo" class="modal">
        <div class="modal-content">
            <h4>Todo ändern</h4> <span>Todo ID: </span> <span id="input-edit-todo-id"></span>
            <form id="form-edit-todo">
                <div class="row">
                    <div class="input-field col s12">
                        <input id="input-edit-todo-title" type="text" class="validate">
                        <label for="input-edit-todo-title">Titel</label>
                    </div>

                    <div class="input-field col s12">
                        <textarea id="input-edit-todo-description" class="materialize-textarea"></textarea>
                        <label for="input-edit-todo-description">Beschreibung</label>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" id="button-edit-todo-close" class="btn waves-effect waves-light red darken-4">Cancel
                        <i class="material-icons left">cancel</i>
                    </button>
                    <button type="submit" id="button-edit-todo-confirm" class="btn waves-effect waves-light green darken-4">Create
                        <i class="material-icons left">add</i>
                    </button>
                </div>
            </form>
        </div>
    </div>
```

### Javascript - Dynamische Todo - Liste

Da nun alle wichtigen HTML Elemente bereit sind, können wir anfangen, die Website dynamisch zu machen.

> Sauberer Code ist wichtig! Es ist eine Konvention, in einer einzelnen Datei erst alle Variablen zu deklarieren, dann die Funktionen, und erst dann die Logik zu beschreiben, in der sie ausgeführt werden

Der folgende Code wird nun folgendes tun: 

- Wähle alle HTML Elemente aus und mache sie im Code ansteuerbar

- Definiere Funktionen zur initialisierung vom Materialize Javascript

- Definiere Funktionen, die Werte aus den HTML Elementen lesen können

- Definiere Funktionen, die Funktionen an HTML Elemente binden

- Binde alle Funktionen an die Webapp, sobald die Inhalte fertig geladen sind

Das folgende Stück Code ist das Herzstück der App, daher ist es ein wenig länger als die folgenden Codeschnipsel

Füge also die folgenden Zeilen in die `main.js` Datei ein:

```javascript
// Halte die URL des Strapi CMS fest
var strapiUrl = 'http://104.248.25.140/todos';

// Wähle alle Elemente der App aus
var buttonNewTodo = document.querySelector('#button-new-todo');
var buttonCloseModalNewTodo = document.querySelector('#button-new-todo-close');
var buttonCloseModalEditTodo = document.querySelector('#button-edit-todo-close');

// Modal Elemente
var modalNewTodo = document.querySelector('#modal-new-todo');
var modalEditTodo = document.querySelector('#modal-edit-todo');

// Form new Elemente
var formNewTodo = document.querySelector('#form-new-todo');
var formNewTodoTitle = document.querySelector('#input-new-todo-title')
var formNewTodoDescription = document.querySelector('#input-new-todo-description')

// Form Edit Elemente
var formEditTodo = document.querySelector('#form-edit-todo');
var formEditTodoId = document.querySelector('#input-edit-todo-id')
var formEditTodoTitle = document.querySelector('#input-edit-todo-title')
var formEditTodoDescription = document.querySelector('#input-edit-todo-description')

// Definiere eine Funktion, um die Materialize Bibliothek zu initialisieren
function initMaterialize() {
  var modalElements = document.querySelectorAll('.modal');

  M.Modal.init(modalElements);
}

// Definiere eine Funktion, um die Modals zu initialisieren
function initModals() {
  var instanceNewTodo = M.Modal.getInstance(modalNewTodo);
  var instanceEditTodo = M.Modal.getInstance(modalEditTodo);

  buttonNewTodo.addEventListener('click', function () {
    instanceNewTodo.open();
  });
  buttonCloseModalNewTodo.addEventListener('click', function (event) {
    event.preventDefault();
    instanceNewTodo.close();
  })
  buttonCloseModalEditTodo.addEventListener('click', function(event) {
    event.preventDefault();
    instanceEditTodo.close();
  })
}

// Definiere eine Funktion, um Inhalte des Formulars an Strapi zu senden
async function createNewTodo() {
  var title = formNewTodoTitle.value;
  var description = formNewTodoDescription.value;
  var payload = { title, description, done: false };
  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'post',
    body: JSON.stringify(payload)
  };

  var response = await fetch(strapiUrl, options);

  return response.status;
}


// Binde alle Funktionen an die App
function bindFormNewTodo() {
  var instanceNewTodo = M.Modal.getInstance(modalNewTodo);

  formNewTodo.addEventListener('submit', async function (event) {
    event.preventDefault();
    const status = await createNewTodo();
    if (status >= 200 && status < 300) {
      M.toast({html: 'Todo erfolgreich erstellt'})
      instanceNewTodo.close();
    }
  })
}

// Führe Funktionen erst aus, nachdem der Browser die Seite fertig geladen hat
document.addEventListener('DOMContentLoaded', function () {
  initMaterialize();
  initModals()
  bindFormNewTodo();
});
```

Sofern alles geklappt hat, können wir einen Schritt zurücktreten und uns das erste Ergebnis ansehen. Die HTML - Seite kann nun per Live-Server geöffnet - und getestet werden.

### Dynamischer Aufbau der Liste

Da wir nun Todos anlegen können sollten wir als nächstes die Todo Liste von Strapi in unsere App einbinden. Dafür müssen wir folgendes tun: 

- Die Daten abrufen

- Für jeden Eintrag ein einzelnes Todo - Element erzeugen

- Dieses Todo - Element in unsere Seite einbinden

Füge also das folgende in die `main.js` Datei ein:

```shell
function renderTodos(todos) {
  var todoList = document.querySelector('#app-todo-list')

  todos.forEach(function (todo) {
    console.log(todo.done)
    var listElement = document.createElement('li');
    var labelElement = document.createElement('label');
    var inputElement = document.createElement('input');
    var spanElement = document.createElement('span');

    listElement.classList.add('collection-item');
    listElement.id = 'todoItem-' + todo.id;
    inputElement.type = 'checkbox';
    if(todo.done) inputElement.checked = 'checked';
    spanElement.innerHTML = todo.title;

    labelElement.append(inputElement, spanElement);
    listElement.append(labelElement);

    todoList.append(listElement);
  })
}

async function getTodos() {
  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'get',
  };
  var response = await fetch(strapiUrl, options);
  var data = await response.json();

  return renderTodos(data);
}
```

### Bestehende Todos updaten: Checkbox

Bevor wir mit der Modal beginnen sollten wir uns die Checkboxen anschauen. Wenn ein Nutzer auf diese Klickt soll das entsprechende Todo item auf 'erledigt' gesetzt werden. Im selben Zug können wir auch ein paar Verbesserungen an der Darstellung vornehmen

- Beim Klick auf ein Todo soll dieses auf Strapi gefunden werden. 

- Je nachdem ob es erledigt ist oder nicht soll es auf den jeweils anderen Wert gesetzt werden

- Der Nutzer soll dann darauf hingewiesen werden, dass seine Aufgabe abgeschlossen wurde

- Das Klick - Event muss dynamisch an jedes Checkbox Element gebunden werden, das heißt, während es in der renderTodos - Funktion erzeugt wird.

- Dort können wir auch die ID übergeben, um das Todo Element auf Strapi zu identifizieren

Ändere zunächst deine `renderTodos` - Funktion ein wenig ab, sodass sie aussieht wie folgt: 

```javascript
function renderTodos(todos) {
  var todoList = document.querySelector('#app-todo-list')

  todos.forEach(function (todo) {
    var listElement = document.createElement('li');
    var labelElement = document.createElement('label');
    var inputElement = document.createElement('input');
    var spanElement = document.createElement('span');

    listElement.classList.add('collection-item');
    listElement.id = 'todoItem-' + todo.id;
    inputElement.type = 'checkbox';
    if (todo.done) inputElement.checked = 'checked';
    bindCheckboxUpdateTodo(inputElement, todo.id);
    spanElement.innerHTML = todo.title;

    labelElement.append(inputElement, spanElement);
    listElement.append(labelElement);

    todoList.append(listElement);
  })
}
```

Füge dann die Funktion `bindCheckboxUpdateTodo` zu deiner `main.js` Datei hinzu: 

```javascript
async function updateTodoDoneById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId
  var optionsGet = {
    headers: { 'content-type': 'application/json' },
    method: 'get',
  };
  var responseGet = await fetch(todoUrl, optionsGet);
  var dataGet = await responseGet.json();

  var isDone = dataGet.done;

  var payloadUpdate = {
    done: !isDone
  }

  var optionsUpdate = {
    headers: { 'content-type': 'application/json' },
    method: 'put',
    body: JSON.stringify(payloadUpdate)
  };

  var responseUpdate = await fetch(todoUrl, optionsUpdate);
  var dataUpdate = await responseUpdate.json();
  return {
    status: responseUpdate.status,
    todo: dataUpdate
  };
}
```

### Bestehende Todos Updaten: Modal

Wie nähern uns der Zielgeraden. 

Was noch fehlt sind zwei wichtige Features, nämlich Update des Titles & der Beschreibung, sowie das Löschen des ganzen Todos. 

Das Update von Titel und Beschreibung ist etwas komplizierter als der Vorgang mit der Checkbox. Wie müssen, da mehr als eine Interaktion im Spiel ist, für jede davon eine eigene Funktion definieren

Die Logik bleibt aber identisch - die Suche nach dem Todo anhand seiner ID, Präsentation und dann Update

Fangen wir mit der Suche an. Wir können hierbei wieder das Listenelement nutzen, und die Suchfunktion dynamisch an einen weiteren Button binden.

Ändere erneut deine `renderTodos` - Funktion so ab: 

```javascript
function renderTodos(todos) {
  var todoList = document.querySelector('#app-todo-list')

  todos.forEach(function (todo) {
    var listElement = document.createElement('li');
    var labelElement = document.createElement('label');
    var inputElement = document.createElement('input');
    var spanElement = document.createElement('span');
    var buttonSectionElement = document.createElement('div')
    var updateButtonElement = document.createElement('button')

    listElement.classList.add('collection-item', 'flex');
    listElement.id = 'todoItem-' + todo.id;

    inputElement.type = 'checkbox';
    if (todo.done) inputElement.checked = 'checked';
    spanElement.innerHTML = todo.title;
    bindCheckboxUpdateTodo(inputElement, todo.id);

    updateButtonElement.classList.add('waves-effect', 'waves-light', 'btn', 'green', 'darken-2');
    updateButtonElement.innerHTML = '<i class="material-icons">edit</i>'
    bindButtonOpenModal(updateButtonElement, todo.id)

    buttonSectionElement.append(updateButtonElement);
    labelElement.append(inputElement, spanElement);
    listElement.append(labelElement, buttonSectionElement);
    todoList.append(listElement);
  })
}
```

Füge dann die folgende Funktion hinzu: 

```javascript
async function getTodoById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId
  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'get',
  };
  var response = await fetch(todoUrl, options);
  var data = await response.json()
  return {
    status: response.status,
    todo: data
  }
}
```

Dann können wir das Modal Event an den Button neben dem Todo - Element binden und die Strapi-Werte des Todos in der Modal anzeigen:

```javascript
function bindButtonOpenModal(buttonElement, todoId) {
  var instanceEditTodo = M.Modal.getInstance(modalEditTodo);

  buttonElement.addEventListener('click', async function () {
    var { status, todo } = await getTodoById(todoId);
    if (status >= 200 && status < 300) {
      instanceEditTodo.open();
      formEditTodoId.innerHTML = todo.id
      formEditTodoTitle.value = todo.title;
      formEditTodoDescription.value = todo.description;
    }
  })
}
```

Füge außerdem die folgenden Zeilen in deine `styles.css` - Datei ein: 

```css
.flex {
  display: flex;
  justify-content: space-between;
}
```

Jetzt sollte sich jedes Mal, wenn du auf den 'Edit' - Button klickst, eine Modal mit den Daten aus dem Todo erscheinen, auf das du geklickt hast. Jetzt müssen wir noch in einem letzten Schritt die geänderten Werte wieder zurück an Strapi geben

### Bestehendes Todo Updaten: Der letzte Zug

Zu guter letzt müssen wir noch, ähnlich wie bei der ersten Aktion, die abgeänderten Daten wieder an Strapi übergeben. 

Anders als bislang ist die Todo ID nicht aus einer anderen Funktion übergeben worden, sondern ist an ein HTML Element gebunden. In einem größeren Projekt werden solche Probleme mit [State Management]([State management - Wikipedia](https://en.wikipedia.org/wiki/State_management)) gelöst

Füge die folgenden Zeilen zu deiner `main.js` hinzu: 

Zunächst die eigentliche Update Funktion:

```javascript
async function updateTodoById() {
  var todoId = formEditTodoId.innerHTML;
  var title = formEditTodoTitle.value;
  var description = formEditTodoDescription.value;

  var todoUrl = strapiUrl + '/' + todoId
  var payload = { title, description }

  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'put',
    body: JSON.stringify(payload)
  };

  var response = await fetch(todoUrl, options);
  return response.status;
}
```

Dann die Funktion, die sie durch Bestätigung des Formulars auslöst

```javascript
function bindFormUpdateTodo() {
  var instanceEditTodo = M.Modal.getInstance(modalEditTodo);

  formEditTodo.addEventListener('submit', async function (event) {
    event.preventDefault();
    var status = await updateTodoById();
    if (status >= 200 && status < 300) {
      M.toast({ html: 'Todo erfolgreich geändert' })
      instanceEditTodo.close();
    }
  })
}
```

Und schließlich noch die Initialisierung

```javascript
document.addEventListener('DOMContentLoaded', function () {
  // ...
  bindFormUpdateTodo();
  // ...
});
```

### Todos löschen

Die letzte Interaktion ist nun noch, die Todos von Strapi wieder löschen zu können. Hierfür können wir uns wieder der renderTodos-Funktion bedienen

Ändere zunächst wieder die `renderTodos` Funktion wie folgt ab:

```javascript
function renderTodos(todos) {
  var todoList = document.querySelector('#app-todo-list')

  todos.forEach(function (todo) {
    var listElement = document.createElement('li');
    var labelElement = document.createElement('label');
    var inputElement = document.createElement('input');
    var spanElement = document.createElement('span');
    var buttonSectionElement = document.createElement('div')
    var updateButtonElement = document.createElement('button')
    var deleteButtonElement = document.createElement('button')

    listElement.classList.add('collection-item', 'flex');
    listElement.id = 'todoItem-' + todo.id;

    inputElement.type = 'checkbox';
    if (todo.done) inputElement.checked = 'checked';
    spanElement.innerHTML = todo.title;
    bindCheckboxUpdateTodo(inputElement, todo.id);

    updateButtonElement.classList.add('waves-effect', 'waves-light', 'btn', 'green', 'darken-2');
    updateButtonElement.innerHTML = '<i class="material-icons">edit</i>'
    bindButtonOpenModal(updateButtonElement, todo.id)

    deleteButtonElement.classList.add('waves-effect', 'waves-light', 'btn', 'red', 'darken-2');
    deleteButtonElement.innerHTML = '<i class="material-icons">delete_sweep</i>'
    bindButtonDeleteTodo(deleteButtonElement, todo.id)

    buttonSectionElement.append(deleteButtonElement, updateButtonElement);
    labelElement.append(inputElement, spanElement);
    listElement.append(labelElement, buttonSectionElement);
    todoList.append(listElement);
  })
}
```

Füge dann die folgenden zwei Funktionen hinzu, um das Projekt abzuschließend

```javascript
async function deleteTodoById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId;
  var options = {
    method: 'delete'
  }

  var response = await fetch(todoUrl, options);
  return response.status;
}
```

```javascript
function bindButtonDeleteTodo(buttonElement, todoId) {
  buttonElement.addEventListener('click', async function (event) {
    var status  = await deleteTodoById(todoId);
    if (status >= 200 && status < 300) {
      M.toast({ html: 'Todo wurde gelöscht' })
    }
  })
}
```

Damit sind alle Grundfunktionen für das Projekt gegeben. Wenn du es bis hierher durchgehalten - & vorher keine Erfahrungen in der Webentwicklung hattest - god damn dann hast du Nerven aus Stahl. Du kannst aber auch verdammt stolz auf dich sein, da du gerade deine erste, voll funktionstüchtige Webapp geschrieben hast. Klar gibt es eine Menge an Potenziel, was bis zu einer Veröffentlichung noch implementiert werden kann. Aber für den Anfang hast du eine solide Basis.

### Bonus: Update bei Änderungen

Wer beim obrigen Code aufgepasst hat wird feststellen, dass die `getTodos()` - Funktion nur einmal beim Laden der Seite ausgeführt wird. Das sollten wir ändern. 

Wir haben im Projekt bereits sogenannte Event Listener verwendet. Solche Events können wir auch selbst definieren und wiederverwenden. 

Angenommen, wir wollten jedes mal, wenn wir mit Strapi interagieren, ein 'todoRefresh' - Event ausgeben, wären wir an einer anderen Stelle in der Lage, dieses abzufangen und die getTodos - Funktion erneut aufzurufen. 

Füge dazu bei den folgenden Funktionen kurz vor dem `return` - statement folgende Zeile ein: 

```javascript
window.dispatchEvent(new Event('todoRefresh'));
```

z.B. bei `deleteTodoById()`

```javascript
async function deleteTodoById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId;
  var options = {
    method: 'delete'
  }

  var response = await fetch(todoUrl, options);
  window.dispatchEvent(new Event('todoRefresh'));
  return response.status;
}
```

Spätestens jetzt sollte auch jedem eine weitere Kleinigkeit auffallen: Wenn getTodos() ausgelöst wird, werden alte Todo - Elemente im Browser nicht überschrieben. Auch dies lässt sich leicht beheben durch einfügen der folgenden Zeile in `renderTodos`: 

```javascript
  // ...
  var todoList = document.querySelector('#app-todo-list') 
  todoList.innerHTML = "";
  // ...
```

### Bonusfeature: Filterfunktion

Zu Anfang des Projektes haben wir ein Inputfeld hinzugefügt, das bislang unbenutzt blieb. Als Sahnehäubchen wollen wir dies nun Nutzen, um Todos direkt im Browser zu filtern.

Füge zunächst den Selektor oben in der App ein: 

```javascript
var inputFilterTodos = document.querySelector('#input-search');
```

Dann brauchen wir wieder einen event listener, der jedes Mal ausgelöst wird, wenn sich der Inhalt des Feldes ändert

```javascript
function bindFilterInput() {
  inputFilterTodos.addEventListener('keyup', function (event) {
    var filterValue = event.target.value;
    handleFilterTodos(filterValue);
  })
}

// ... füge diese Funktion auch wieder zum DOMContentLoaded - Event hinzu
```

Zu guter letzt brauchen wir noch die Filterfunktion selbst: 

```javascript

```

### Der vollständige Code

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STUNT Schulung Todo liste</title>

    <!-- Importiere Materialize CSS in deine Website -->
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">

    <!-- Importieren eigenes CSS -->
    <link rel="stylesheet" href="/styles.css">

</head>

<body>

    <nav>
        <div class="nav-wrapper green darken-4">
            <a href="/" class="brand-logo center">Todo - Projekt</a>
        </div>
    </nav>

    <main class="container mt-2">
        <div class="row">
            <div class="col m2 lg3"></div>
            <div class="col s12 m8 lg6">
                <!-- Suchleiste und Button für neues Todo -->
                <div class="row">
                    <div class="input-field col s9">
                        <i class="material-icons prefix">search</i>
                        <input id="input-search" type="text">
                        <label for="input-search">Suche nach Todos ...</label>
                    </div>
                    <div class="input-field col s3">
                        <button data-target="modal-new-todo" id="button-new-todo" class="btn waves-effect waves-light green darken-4 w-full">Add new Todo
                            <i class="material-icons left">add</i>
                        </button>
                    </div>
                </div>
                <!-- /Suchleiste und Button für neues Todo -->

                <!-- Todo Übersicht -->
                <div class="row">
                    <ul id="app-todo-list" class="collection">
                    </ul>
                </div>
                <!-- /Todo Übersicht -->
            </div>
            <div class="col m2 lg3"></div>
        </div>
    </main>

    <!-- Modal für neues Todo -->
    <div id="modal-new-todo" class="modal">
        <div class="modal-content">
            <h4>Neues Todo anlegen</h4>
            <form id="form-new-todo">
                <div class="row">
                    <div class="input-field col s12">
                        <input id="input-new-todo-title" type="text" class="validate">
                        <label for="input-new-todo-title">Titel</label>
                    </div>

                    <div class="input-field col s12">
                        <textarea id="input-new-todo-description" class="materialize-textarea"></textarea>
                        <label for="input-new-todo-description">Beschreibung</label>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" id="button-new-todo-close" class="btn waves-effect waves-light red darken-4">Cancel
                        <i class="material-icons left">cancel</i>
                    </button>
                    <button type="submit" id="button-new-todo-confirm" class="btn waves-effect waves-light green darken-4">Create
                        <i class="material-icons left">add</i>
                    </button>
                </div>
            </form>
        </div>
    </div>
    <!-- /Modal für neues Todo -->

    <!-- Modal für bestehendes Todo -->
    <div id="modal-edit-todo" class="modal">
        <div class="modal-content">
            <h4>Todo ändern</h4> <span>Todo ID: </span> <span id="input-edit-todo-id"></span>
            <form id="form-edit-todo">
                <div class="row">
                    <div class="input-field col s12">
                        <input id="input-edit-todo-title" type="text" class="validate">
                        <label for="input-edit-todo-title">Titel</label>
                    </div>

                    <div class="input-field col s12">
                        <textarea id="input-edit-todo-description" class="materialize-textarea"></textarea>
                        <label for="input-edit-todo-description">Beschreibung</label>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" id="button-edit-todo-close" class="btn waves-effect waves-light red darken-4">Cancel
                        <i class="material-icons left">cancel</i>
                    </button>
                    <button type="submit" id="button-edit-todo-confirm" class="btn waves-effect waves-light green darken-4">Update
                        <i class="material-icons left">save</i>
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Importiere Materialize Javascript in deine Website -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
    <script src="/main.js"></script>
</body>

</html>
```

```css
.mt-2 {
  margin-top: 2rem;
}

.w-full {
  width: 100%;
}

.flex {
  display: flex;
  justify-content: space-between;
}
```

```javascript
// Halte die URL des Strapi CMS fest
var strapiUrl = 'http://104.248.25.140/todos';

// Wähle alle Elemente der App aus
var buttonNewTodo = document.querySelector('#button-new-todo');
var buttonCloseModalNewTodo = document.querySelector('#button-new-todo-close');
var buttonCloseModalEditTodo = document.querySelector('#button-edit-todo-close');

// Modal Elemente
var modalNewTodo = document.querySelector('#modal-new-todo');
var modalEditTodo = document.querySelector('#modal-edit-todo');

// Form new Elemente
var formNewTodo = document.querySelector('#form-new-todo');
var formNewTodoTitle = document.querySelector('#input-new-todo-title')
var formNewTodoDescription = document.querySelector('#input-new-todo-description')

// Form Edit Elemente
var formEditTodo = document.querySelector('#form-edit-todo');
var formEditTodoId = document.querySelector('#input-edit-todo-id')
var formEditTodoTitle = document.querySelector('#input-edit-todo-title')
var formEditTodoDescription = document.querySelector('#input-edit-todo-description')

// Definiere eine Funktion, um die Materialize Bibliothek zu initialisieren
function initMaterialize() {
  var modalElements = document.querySelectorAll('.modal');

  M.Modal.init(modalElements);
}

// Definiere eine Funktion, um die Modals zu initialisieren
function initModals() {
  var instanceNewTodo = M.Modal.getInstance(modalNewTodo);
  var instanceEditTodo = M.Modal.getInstance(modalEditTodo);

  buttonNewTodo.addEventListener('click', function () {
    instanceNewTodo.open();
  });
  buttonCloseModalNewTodo.addEventListener('click', function (event) {
    event.preventDefault();
    instanceNewTodo.close();
  })
  buttonCloseModalEditTodo.addEventListener('click', function (event) {
    event.preventDefault();
    instanceEditTodo.close();
  })
}

/**
 * Funktionen für View Rendering
 */
function renderTodos(todos) {
  var todoList = document.querySelector('#app-todo-list')
  todoList.innerHTML = "";

  todos.forEach(function (todo) {
    var listElement = document.createElement('li');
    var labelElement = document.createElement('label');
    var inputElement = document.createElement('input');
    var spanElement = document.createElement('span');
    var buttonSectionElement = document.createElement('div')
    var updateButtonElement = document.createElement('button')
    var deleteButtonElement = document.createElement('button')

    listElement.classList.add('collection-item', 'flex');
    listElement.id = 'todoItem-' + todo.id;

    inputElement.type = 'checkbox';
    if (todo.done) inputElement.checked = 'checked';
    spanElement.innerHTML = todo.title;
    bindCheckboxUpdateTodo(inputElement, todo.id);

    updateButtonElement.classList.add('waves-effect', 'waves-light', 'btn', 'green', 'darken-2');
    updateButtonElement.innerHTML = '<i class="material-icons">edit</i>'
    bindButtonOpenModal(updateButtonElement, todo.id)

    deleteButtonElement.classList.add('waves-effect', 'waves-light', 'btn', 'red', 'darken-2');
    deleteButtonElement.innerHTML = '<i class="material-icons">delete_sweep</i>'
    bindButtonDeleteTodo(deleteButtonElement, todo.id)

    buttonSectionElement.append(deleteButtonElement, updateButtonElement);
    labelElement.append(inputElement, spanElement);
    listElement.append(labelElement, buttonSectionElement);
    todoList.append(listElement);
  })
}

/**
 * Funktionen für Strapi Kommunikation
 */
async function getTodos() {
  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'get',
  };
  var response = await fetch(strapiUrl, options);
  var data = await response.json();

  return renderTodos(data);
}

async function getTodoById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId
  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'get',
  };
  var response = await fetch(todoUrl, options);
  var data = await response.json()
  return {
    status: response.status,
    todo: data
  }
}

async function createNewTodo() {
  var title = formNewTodoTitle.value;
  var description = formNewTodoDescription.value;
  var payload = { title, description, done: false };
  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'post',
    body: JSON.stringify(payload)
  };

  var response = await fetch(strapiUrl, options);

  return response.status;
}

async function updateTodoDoneById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId
  var optionsGet = {
    headers: { 'content-type': 'application/json' },
    method: 'get',
  };
  var responseGet = await fetch(todoUrl, optionsGet);
  var dataGet = await responseGet.json();

  var isDone = dataGet.done;

  var payloadUpdate = {
    done: !isDone
  }

  var optionsUpdate = {
    headers: { 'content-type': 'application/json' },
    method: 'put',
    body: JSON.stringify(payloadUpdate)
  };

  var responseUpdate = await fetch(todoUrl, optionsUpdate);
  var dataUpdate = await responseUpdate.json();
  return {
    status: responseUpdate.status,
    todo: dataUpdate
  };
}

async function updateTodoById() {
  var todoId = formEditTodoId.innerHTML;
  var title = formEditTodoTitle.value;
  var description = formEditTodoDescription.value;

  var todoUrl = strapiUrl + '/' + todoId
  var payload = { title, description }

  var options = {
    headers: { 'content-type': 'application/json' },
    method: 'put',
    body: JSON.stringify(payload)
  };

  var response = await fetch(todoUrl, options);
  return response.status;
}

async function deleteTodoById(todoId) {
  var todoUrl = strapiUrl + '/' + todoId;
  var options = {
    method: 'delete'
  }

  var response = await fetch(todoUrl, options);
  return response.status;
}

/**
 * Binde Funktionen an Elemente
 */
function bindFormNewTodo() {
  var instanceNewTodo = M.Modal.getInstance(modalNewTodo);

  formNewTodo.addEventListener('submit', async function (event) {
    event.preventDefault();
    var status = await createNewTodo();
    if (status >= 200 && status < 300) {
      M.toast({ html: 'Todo erfolgreich erstellt' })
      instanceNewTodo.close();
    }
  })
}

function bindFormUpdateTodo() {
  var instanceEditTodo = M.Modal.getInstance(modalEditTodo);

  formEditTodo.addEventListener('submit', async function (event) {
    event.preventDefault();
    var status = await updateTodoById();
    if (status >= 200 && status < 300) {
      M.toast({ html: 'Todo erfolgreich geändert' })
      instanceEditTodo.close();
    }
  })
}

function bindCheckboxUpdateTodo(checkboxElement, todoId) {
  checkboxElement.addEventListener('click', async function (event) {
    var { status, todo } = await updateTodoDoneById(todoId);
    if (status >= 200 && status < 300) {
      if (todo.done === true) {
        M.toast({ html: 'Todo ' + todo.title + ' als erledigt markiert' })
      }

      if (todo.done === false) {
        M.toast({ html: 'Todo ' + todo.title + ' wieder in Liste aufgenommen' })
      }
    }
  })
}

function bindButtonOpenModal(buttonElement, todoId) {
  var instanceEditTodo = M.Modal.getInstance(modalEditTodo);

  buttonElement.addEventListener('click', async function () {
    var { status, todo } = await getTodoById(todoId);
    if (status >= 200 && status < 300) {
      instanceEditTodo.open();
      formEditTodoId.innerHTML = todo.id
      formEditTodoTitle.value = todo.title;
      formEditTodoDescription.value = todo.description;
    }
  })
}

function bindButtonDeleteTodo(buttonElement, todoId) {
  buttonElement.addEventListener('click', async function (event) {
    var status  = await deleteTodoById(todoId);
    if (status >= 200 && status < 300) {
      M.toast({ html: 'Todo wurde gelöscht' })
    }
  })
}

// Führe Funktionen erst aus, nachdem der Browser die Seite fertig geladen hat
document.addEventListener('DOMContentLoaded', function () {
  initMaterialize();
  initModals()
  bindFormNewTodo();
  bindFormUpdateTodo();
  getTodos();
});
```
