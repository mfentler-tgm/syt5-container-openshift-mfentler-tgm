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

```
oc login
username
password
```

### Add Docker images to project
Auf der Webseite einfach auch Add to Project klicken und dann bei "Name" den richtigen Namen vom Image angeben.  

Um Anwendungen zu skalieren erhöht man die "Pod-Anzahl" über die Pfeile daneben (im Overview Fenster).

"Application scaling can happen extremely quickly because OpenShift is just launching new instances of an existing image, especially if that image is already cached on the node."

### Self Healing
Wenn man zwei Pods eingestellt hat und eine killt, dann kümmert sich OpenShift automatisch darum wieder eine zweite Instanz zu kreieren.

### Routes
Um auf diese Applikationen von außen zuzugreifen muss man eine Route hinzufügen. Das macht man einfach über den Button "Add Route".  
Anschließend ist die Applikation über den angegeben Link aufrufbar.
