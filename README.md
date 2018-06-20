# Progetto ASW

Questa repo contiene il progetto del corso di Architetture e Sistemi Software su Kubernetes fatto dagli studenti:

 - Christian Micocci
 - Andrea Valentini
 - Giammarco Sommaini



## Descrizione

Il progetto riguarda l'orchestrazione di tre applicazioni usando [Kubernetes](https://kubernetes.io/).

 - `hello` [link guida](#orchestrazione-di-hello) [link progetto](https://github.com/aswroma3/asw/tree/master/projects/asw-895-docker-orchestrazione/c-hello-service)
 - `lucky-word` [link guida](#orchestrazione-di-lucky-word) [link progetto](https://github.com/aswroma3/asw/tree/master/projects/asw-875-spring-cloud/a2-lucky-word-cloud-config-client)
 - `sentence` [link guida](#orchestrazione-di-sentence) [link progetto](https://github.com/aswroma3/asw/tree/master/projects/asw-895-docker-orchestrazione/e-sentence-stack-zuul)
 

Tutti i progetti sono stati già compilati e sono state create tutte le immagini 
Docker dei vari servizi e rese disponibili in 
[un Docker Hub pubblico](https://hub.docker.com/r/chrimic/).

In ogni cartella di questa repo sono disponibili le configurazioni YAML necessarie 
per orchestrare le tre applicazioni.


## Requisiti

È necessario che sia installato il software [Minikube](https://kubernetes.io/docs/setup/minikube/).

 
## Orchestrazione di "Hello"

Creare il Deployment e il Service:

```bash
$ kubectl create -f hello-word-kubernetes/hello-depl.yaml
$ kubectl create -f hello-word-kubernetes/hello-service.yaml
```

---

Attendere il rollout del deployment:

```bash
$ kubectl rollout status deployment/hello
```

```
Waiting for rollout to finish: 0 of 1 updated replicas are available...
deployment "hello" successfully rolled out
```


### Accedere al servizio

Leggere l'ip esterno di un nodo (campo NAME):

```bash
$ kubectl get nodes
```

```
NAME           STATUS                     ROLES     AGE       VERSION
172.17.8.101   Ready,SchedulingDisabled   <none>    2d        v1.10.2
172.17.8.102   Ready                      <none>    2d        v1.10.2
```

Trovare la porta su cui il servizio è stato esportato (è sempre casuale):

```bash
$ kubectl get svc
```

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello        NodePort    10.100.121.47   <none>        8080:30501/TCP   8m
kubernetes   ClusterIP   10.100.0.1      <none>        443/TCP          2d
```

Aprire un browser e collegarsi al nodo (*172.17.8.101*) tramite la porta trovata (*30501*): http://172.17.8.101:30501

*(potrebbe passare qualche minuto prima che l'applicazione sia disponibile)*


### Scalare "Hello"

```bash
$ kubectl edit deployment/hello
```

Si aprirà un editor di testo. Modificare il valore di `spec/replicas` da `1` a `3` e salvare il file.

Attendere il rollout come fatto prima:

```bash
$ kubectl rollout status deployment/hello
```

Aggiornando la pagina web numerose volte si dovrebbe vedere il nome della macchina (o meglio, del **pod**) cambiare. Ad esempio:

```bash
$ for i in {1..10}; do; curl http://172.17.8.101:30501/; echo; done
```

```
Hello from hello-68dcdd9545-b7466!
Hello from hello-68dcdd9545-b7466!
Hello from hello-68dcdd9545-b7466!
Hello from hello-68dcdd9545-b6pf5!
Hello from hello-68dcdd9545-b6pf5!
Hello from hello-68dcdd9545-b7466!
Hello from hello-68dcdd9545-b7466!
Hello from hello-68dcdd9545-b6pf5!
Hello from hello-68dcdd9545-7x6kx!
Hello from hello-68dcdd9545-b6pf5!
```


### Rimuovere "Hello"

Nel caso in cui si voglia rimuovere l'applicazione "Hello" dal nostro cluster Kubernetes si possono digitare i seguenti comandi:

```bash
$ kubectl delete svc hello
$ kubectl delete deployment hello
```
 
 

## Orchestrazione di "Lucky word"

Per la natura di "Lucky word" l'unico modo per avviarlo correttamente è quello di avviare per prima il *Deployment* e il *Service* del **config-server**, e solo dopo che questo sia correttamente avviato si può avviare anche il *Deployment* e il *Service* di **lucky-word**.

```bash
$ kubectl create -f lucky-word-kubernetes/common-config-depl.yaml
$ kubectl create -f lucky-word-kubernetes/common-config-service.yaml
```

Attendere che il deployment vada a buon fine:

```bash
$ kubectl rollout status deployment/common-config
```

Avviare il client:

```bash
$ kubectl create -f lucky-word-kubernetes/lucky-word-depl.yaml
$ kubectl create -f lucky-word-kubernetes/lucky-word-service.yaml
```

Attendere che il deployment vada a buon fine:

```bash
$ kubectl rollout status deployment/lucky-word
```

Ora è possibile [accedere al servizio come fatto con hello](#accedere-al-servizio).



## Orchestrazione di "Sentence"

```bash
for f in sentence-kubernetes/* 
do
kubectl create -f $f
done
```

Attendere che i deployment vadano a buon fine:

```bash
$ watch kubectl get pods
```

```
NAME                            READY     STATUS              RESTARTS   AGE
pod/object-66f6979cfd-xzxk7     0/1       ContainerCreating   0          28s
pod/sentence-5c6fccf988-lj9xt   0/1       ContainerCreating   0          27s
pod/subject-86986999bb-69dkn    0/1       ContainerCreating   0          27s
pod/verb-6bc9758d98-xpkpq       0/1       ContainerCreating   0          26s

```

Ci vorrà qualche minuto prima che lo **STATUS** dei pod cambi in **Running**.


---

Leggere l'ip esterno di un nodo (campo NAME):

```bash
$ kubectl get nodes
```

```
NAME           STATUS                     ROLES     AGE       VERSION
172.17.8.101   Ready,SchedulingDisabled   <none>    2d        v1.10.2
172.17.8.102   Ready                      <none>    2d        v1.10.2
```

Trovare la porta su cui il servizio è stato esportato (è sempre casuale):

```bash
$ kubectl get svc sentence
```

```
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
sentence   NodePort   10.100.49.57   <none>        8080:30807/TCP   5m
```

Aprire un browser e collegarsi al nodo (*172.17.8.101*) tramite la porta trovata (*30807*): http://172.17.8.101:30807

*(potrebbe passare qualche minuto prima che l'applicazione sia disponibile)*
