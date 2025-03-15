# K8s Intro

## Use cases
K8s este un manager de containere dezvoltat de google, cel mai important feature este cel de restart al containerelor pentru momentele in care din varii motive aplicatia crapa si containerul iese cu error code.

Un mare avantaj al unui cluster de k8s este scalabilitate usora pe orizontala (adica putem adauga instante ale acealeasi aplicatii rapid), alt lucru atractiv in a folosi k8s este principiul de high-availability in care aplicatia este accesibila constant in ciuda erorilor.

## Install and set up

Sunt cateva metode prin care putem instala k8s:

- Instalarea fiecari componente de mana
- utilitarul _kubeadm_
- prin docker folosind imagea de _minikube_

### Recomandari:

- daca e enviroment de testing/experimentare --> minikube
- daca trebuie sa fie rapid instalat si configuratia de baza e buna --> _kubeadm_
- daca sunt cerinte specifice --> instalarea fiecarei componente se face de mana

## Pentru demo putem folosi _minikube_

#### Alternativa KillerKonda --> [click me](https://killercoda.com/playgrounds/scenario/kubernetes)

#### Alternativa playWithK8s --> [click me](- [Play with Kubernetes](https://labs.play-with-k8s.com/))

Prerequisites --> Docker installed

Pasi:

- Install cli kubectl prin acest link --> [click me!](https://kubernetes.io/docs/tasks/tools/)
- Urmariti pasii de instalarea din documentatia oficiala pentru minikube --> [click me!](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download)

### Comenzi de baza pentru interactiune

```
kubectl get <resource>        # listeaza resursele
kubectl describe <resource>   # printeaza informatii despre resursa (debugging)
kubectl logs <pod-name>       # printeaza log-urile create de pod (debugging)
kubectl apply -f <file-name>  # in cazul in care exista IaC - good practice
kubectl delete <resource>     # sterge podul
kubectl run <name> --image=<image> # creaza dinamic un pod
kubectl edit <resource>       # editeaza definitia unei resurse
kubectl exec -it <pod> -- sh  # intram in pod si executam sh
```

### Cele mai uzuale resurse

K8s pune la dispozitie mai multe tipuri de resurse:

- pods/static-pods
- services
- replicasets/statefulsets
- deployments
- persistentVolumes / persistentVolumeClaims
- config-maps/secrets
- roles
- cronjobs

### Overview arhitectura
![image](https://github.com/user-attachments/assets/b30e1716-e20c-4095-8e8c-51791e593389)


Exista 2 tipuri de noduri:

 - Master node sau control-plane-node: - ETCD-cluster - kube-scheduler - kube-controller-manager - kube-apiserver
 - Worker node: - kubelet - kube-proxy 

Toate componentele de baza sunt instalate ca pods prin kubeadm intr-un namespace dedicat, adica in kube-system namespace.

### Pods

### Services

Pe scurt sunt 3 tipuri de services: - NodePort - externalizeaza aplicatia catre exterior i.e User

```
k create svc nodeport test --tcp=80:80 --dry-run=client -o yaml
# legam service-ul de pod prin selector, iar in acel selector specificam labelul podului dorit
```
![image](https://github.com/user-attachments/assets/fc23f55b-03e5-4842-b338-b02b96ba29a0)



- ClusterIP - default type - diferenta fata de nodeport este ca leaga poduri intern

![image](https://github.com/user-attachments/assets/8307de75-b08c-464b-b2ac-bcbaec70cdc3)

- LoadBalancer -- doar cloud providers e.g. Azure, AWS, GCP -- face fix acelasi lucru ca si nodeport --- pentru cazuri private se poate configura prin NGINX

### Logging

In k8s putem avea un pod cu mai multe containere ruland simultan, dar si init-containers care "mor" dupa terminarea executiei, in acest context cum putem verifica aplicatia din container?

```
k logs <pod> <container-name> # pentru mai multe containere cu running apps
k logs <pod> -c <init-container> # pentru cele care au init containers
```

### Aplication Lifecycle management

Deployment-urile au 2 stategii de update

- Rolling update -- default
- Recreate update

![image](https://github.com/user-attachments/assets/fc20e20b-6583-4e9c-b8b5-48f4083178a1)

Prerequisites -- diferenta intre ENTRYPOINT si CMD in docker?
In k8s avem optiunea sa le folosim pe ambele in interiorul unui container dupa urmatoare regula:

- command echivalent la ENTRYPOINT
- args echivalent la CMD
- putem specifica args si din comandline prin kubectl e.g.:

```
kubectl run nginx --image=nginx -- sh
```

Dar ce putem face in legatura cu env-variable?
In k8s putem specifica aceste env-variable in mai multe moduri:

- Plain
- Config-map
- Secret

Specificam in yaml dupa urmatorul format:

![image](https://github.com/user-attachments/assets/2fc03190-b3e1-4290-9aba-4e39872e723a)

```
#configmap
kubectl create cm config --from-literal=key=val --dry-run=client -o yaml
#secret
kubectl create secret generic secret --from-literal=key=val --dry-run=client -o yaml
```

### Exercitii

1. Pentru acest exercitiu vom avea nevoie sa facem deployment la un webapp.

    a) Cum putem rula _deploy.yaml_ ?

    b) Ce imagine ruleaza pe acest deployment?

    c) Cum putem crea un service care sa aibe urmatoarele cerinte:

        - Name: webapp-service
        - Type: NodePort
        - targetPort: 8080
        - port: 8080
        - nodePort: 30080
        - selector:
          - name: simple-webapp

    d) Cum putem scala deploymentul?

 2. In acest execitiu trebuie sa identificam probleme din cadrul unei aplicatii.

    a) Rulati un pod cu urmatoarele specificatii:

        - name: webapp-1
        - image: kodekloud/event-simulator
        - port: 8080
        - protocol: TCP

    b) Ce probleme are User5?

3. Rulati fisierul _envVar-pod.yaml_ si folositi comanda _kubectl_ pentru determinarea env variable ce sunt setate pe acest pod. Cum putem edita aceasta variabila de env in 'green'?

4. Creati un _config-map_ cu urmatoarele specificatii si injectati acest config-map in pod-ul _webapp-color_:

        - APP_COLOR: darkblue
        - APP_OTHER: disregard

5. Creati un secret generic cu urmatoarele specificatii si inlocuiti config-map-ul anterior cu acest secrt in pod-ul _webapp-color_:

        - APP_COLOR: pink
        - APP_OTHER: something

6. Rulati multi-container-pod.yaml si identificati cate containere ruleaza, care este fiecare numele desemnat fiecarui container.

7. Creati un multicontainer pod cu urmatoarele specificatii:

        - Name: yellow
        - Container 1 Name: lemon
        - Container 1 Image: busybox
        - Container 2 Name: gold
        - Container 2 Image: redis

8. Pentru acest exercitiu trebuie sa avem urmatoarele componente din ELK stack up and running: kibana-app + kibana-service, elastic-app + elastic-service, app. [elasticsearch-docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html), [kibana-docs](https://www.elastic.co/guide/en/kibana/current/development.html) ‼️ Log-urile aplicatiei sunt scrise in pod /var/log/app/webapp ‼️
 ```
  kubectl apply -f elk-stack.yaml && kubectl apply -f elk-app.yaml
 ```
  

Vrem sa adaugam un sidecar pentru app in care avem urmatoarele specificatii:

SideCAR-container:

    - Name: app
    - Container Name: sidecar
    - Container Image: kodekloud/filebeat-configured
    - Volume Mount: log-volume
    - Mount Path: /var/log/event-simulator/
    - Existing Container Name: app
    - Existing Container Image: kodekloud/event-simulator
  
---
Special Thanks KodeKloud for images provided

---

# Extra topics:

## Cluster Maintenance with backups and restores

## Storage how does it work

## Security in k8s

## Networking -- ingress/egress/network policies

## Helm -- versioning + managing k8s aplications
