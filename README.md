# syt5-container-openshift-mfentler-tgm
syt5-container-openshift-mfentler-tgm created by GitHub Classroom
## Getting started with OpenShift
OpenShift is often referred to as a container application 
platform in that it is a platform designed for the development and deployment 
of containers.

OpenShift kann entweder über die CLI oder über eine Webansicht bearbeitet werden.

Die CLI wird verwendet um 
- direkt mit dem Projekt Source Code zu arbeiten
- Scripting OpenShift Operations
- Wenn man die Webconsole nicht benutzen kann.

        oc login
        username
        password

### Add Docker images to project
Auf der Webseite einfach auch **"Add to Project"** klicken und dann bei "Name" den richtigen Namen vom Image angeben.  

Um Anwendungen zu skalieren erhöht man die "Pod-Anzahl" über die Pfeile daneben (im Overview Fenster).

"Application scaling can happen extremely quickly because OpenShift is just launching new instances of an existing image, especially if that image is already cached on the node."

### Self Healing
Wenn man zwei Pods eingestellt hat und eine killt, dann kümmert sich OpenShift automatisch darum wieder eine zweite Instanz zu kreieren.

### <a name="route">Routes</a>
Um auf diese Applikationen von außen zuzugreifen muss man eine Route hinzufügen. Das macht man einfach über den Button **"Add Route"**.  
Anschließend ist die Applikation über den angegeben Link aufrufbar, den man über **Application/** findet.

### Building Docker Image from source code
Um ein Dockerimage von einem vorhandenen Source Code zu erstellen, geht man wieder zu **"Add to project"** und dann drückt man statt **"Deploy Image"** auf **"Browse Catalogue"**.  
Dort wählt man die Sprache aus und fügt anschließend einen _Link zum GitHub Repo_ ein.

## Deploying applications from images
Es gibt drei Möglichkeiten um Applikationen zu deployen. Zwei davon wurden oben schon beschrieben.  

- Deploy an application from an existing Docker-formatted image.
- Build and deploy from source code contained in a Git repository using a Source-to-Image builder.
- Build and deploy from source code contained in a Git repository from a Dockerfile.

<a name="neuProjekt"></a>So erstellt man Projekte über die cli:

        oc new-project <projectname>
Um neue Applikationen zu erstellen:

        oc new-app

### Deploy from existing docker image
#### Webseite
Über die Webseite kann man wie im vorherigen Teil beschrieben, bestehende Dockerfiles hinzufügen.  
Dort kann man aus den zwei Optionen **"Image Stream Tag"** und **"Image Name"** auswählen.  

Die erste Option wählt man, wenn das Image schon einmal in das OpenShift Cluster importiert wurde oder man die Möglichkeit "Source-to-Image" verwendet hat, dann kann man dort den _Image-Stream-Tag_ eingeben.

Letzteres wählt man, wenn das Docker-Image auf einer externen Image-Registry liegt.

#### CLI
Um zu überprüfen, ob der Name vom Image (openshiftkatacoda/blog-django-py) auch valid ist kann man das in der cli so überprüfen:

        oc new-app --search openshiftkatacoda/blog-django-py // oc new-app --search <image-name>
Um das Image dann zu deployen:

        oc new-app openshiftkatacoda/blog-django-py 

Der Name für die Applikation wird von OpenShift automatisch ausgewählt, genauso wie in der Webversion.  
Will man ihn selber bestimmen, dann muss man den vorigen Befehl mit der Option **"--name"** ausführen.

Anschließend kann man der Applikation noch eine externe Route hinzufügen.
### Creating a external route
#### Webseite
Siehe [oben](#route)

#### CLI
        oc expose service/blog-django-py // service/<application-name>
Hostnamen anzeigen:

        oc get route/blog-django-py // route/<appliaction-name>
### <a name="loeschen"></a>Deleting the application
List all created ressources

        oc get all -o name
Der command listet mir alle Ressourcen auf. Sobald es aber mehrere Applikationen gibt muss unterschieden werden was zu was gehört.  
Das macht man, in dem man am besten nach dem Label sucht(vorher beim Erstellen gesetzt). Dazu wählt man sich eine der vorher aufgelisteten Ressourcen aus und schreibt:

        oc describe <route/blog-django-py>
Der Befehl oben gibt einem viele Informationen über die Ressource aus. Unteranderem auch das Label (app=???)  
Anschließend kann man danach filtern:

        oc get all --selector app=blog-django-py -o name
Um diese Ressourcen nun zu **löschen**:

        oc delete all --selector app=blog-django-py
### Import application images
Wenn man mehrere Applikationen deployen will, dann sollte man, bevor man das tut die Application Images herunterladen. Denn so kann man sicherstellen, dass das Label von diesem Image-Stream nicht gesetzt wurde.  
Es kann zu einem Problem führen, wenn man mehrere Applikationen deployed und dann die eine, auf die mit dem Label referenziert wird löscht. Dann werden nämlich alle mit diesem Label gelöscht.

Daher wird das Image zuerst erstelt:

        oc import-image openshiftkatacoda/blog-django-py
Anschließend kann man eine neue Applikation erstellen:

        oc new-app blog-django-py --name blog-1
Wenn man nun eine Zweite Applikation erstellt und danach folgenden Befehl schreibt kann man erkennen, dass es die Dinge wie "deploymentconfigs", "replicationcontrollers", etc. jeweils doppelt gibt. Nur den **Image-Stream**  gibt es einmal.  

Um die Applikationen anschließend wieder zu [löschen](#loeschen), muss das einzeln für jede gemacht werden.(app=block1, app=block2)

## Connecting to a database using port forwarding
### Wichtigste Commands
        oc rsh <pod-name>
        oc port-forward <pod-name> <local-port>:<remote-port>
        // bzw
        oc port-forward <pod-name> :<remote-port>
Ausgangslage ist ein [neues Projekt](#neuProjekt). 

Um die Datenbank zu erstellen reicht folgender langer Command:

        oc new-app postgresql-ephemeral --name database --param DATABASE_SERVICE_NAME=database  
        --param POSTGRESQL_DATABASE=sampledb --param POSTGRESQL_USER=username  
        --param POSTGRESQL_PASSWORD=password
Durch diese Methode wird allerdings nur eine lokale Datenbank erstellt -> Wenn die Datenbank neu gestartet wird ist alles weg.  
Sobald dieser Command exited ist die Datenbank fertig und bereit:

        oc rollout status dc/database
Wenn man die Datenbank mit der FrontEnd-Web-Applikation verbinden will, dann muss man sie auf der Web-Applikation manuell einrichten.

Um herauszufinden wo die Datenbank läuft um sich dann dort hin zu verbinden:

        oc get pods --selector app=database
Um leichter auf den Namen der Datenbank zu referenzieren kann man sich diesen in eine Umgebungsvariable speichern:

        POD=`oc get pods --selector app=database -o custom-columns=name:.metadata.name --no-headers`; echo $POD
Um eine Shell im selben Container wie die Datenbank zu öffen und das nachher zu validieren benutzt man die Befehle:

        oc rsh $POD
        ps x
Dort kann man nun mit dem **"psql"** command an der Datenbank herumspielen. Um aus der Shell wieder rauszukommen benuzt man **"\exit"**.

### Remote Connection
Um die Datenbank auch von einem Laptop in einer Remote Verbindung zu erreichen muss man Port Forwarding benutzen. Dazu muss auf der Applikation eine Route angelegt sein. 
Anschließend führt man diesen Befehl aus:

        oc port-forward <pod-name> <local-port>:<remote-port>
Lokal wird die Datenbank auf Port 15432 angesprochen, falls schon eine PostgreSQL Datenbank auf 5432 läuft.
        
        //Oder wie in unserem Beispiel mit der Postgre Datenbank (Port 5432);
        oc port-forward $POD 15432:5432
Wenn man nicht weis, welche Ports frei sind, dann kann man auch einen **Random-Port statt dem "local-port"** verwenden.  
Im **Output** von diesem Command kann man sehen **welcher Port** verwendet wird.

        oc port-forward <pod-name> :<remote-port>

Wenn man sich jetzt mit der Datenbank dann verbinden will muss man den Port explizit angeben. Anschließend wird die Prompt geöffnet, die normalerweise nach dem "psql-command" geöffnet wird.  
Das könnte dann so aussehen:
        
        psql sampledb username --host=127.0.0.1 --port=15432
Wenn man fertig ist muss noch der port forwarding Prozess gekillt werden der im Hintergrund aktiv ist:
        
        kill %1

## Transfering files in and out of container
### Downloading files/folder from a container
Für den Download aus einem Container benützt man folgende Befehle. Die Destination auf der lokalen Maschine muss existieren.  
Existierende Files mit dem selben Namen werden überschrieben.

        oc rsync <pod-name>:/remote/dir/filename ./local/dir
        // oder Folder
        oc rsync <pod-name>:/remote/dir/. ./local/dir
        
        
        
