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

# Deploying applications from images
Es gibt drei Möglichkeiten um Applikationen zu deployen. Zwei davon wurden oben schon beschrieben.  

- Deploy an application from an existing Docker-formatted image.
- Build and deploy from source code contained in a Git repository using a Source-to-Image builder.
- Build and deploy from source code contained in a Git repository from a Dockerfile.

So erstellt man Projekte über die cli:

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
### Deleting the application
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
