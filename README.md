# Déploiement de l'application GD4H eau

Ce dépot permet le déploiement l'application [GD4H eau](https://github.com/blenzi/GD4H_eau).

#### Chart Helm

Le déploiement de l'application se fait via un chart Helm, basé sur le [template de l'application shiny-app](https://github.com/InseeFrLab/helm-charts-shiny-apps).

Le fichier `Chart.yaml` contient les métadonnées du chart (nom, version) ainsi que ses dépendances, i.e. les potentiels autres charts Helm dont il hérite. Dans notre cas, on voit que le chart hérite du [chart Shiny](https://github.com/InseeFrLab/helm-charts/tree/master/charts/shiny) d'InseeFrLab. Ce chart spécifie généralement les ressources Kubernetes nécessaires au déploiement d'une application Shiny, de sorte à ce que l'on ait qu'à modifier les valeurs d'instanciation pour déployer notre application.

Le fichier `values.yaml` contient précisément les valeurs que l'on modifie par rapport au chart général. Les modifications à apporter dépendent naturellement de ce que réalise en pratique l'application, car cela conditionne les ressources dont elle a besoin. Dans un premier temps, il nous faut modifier : 
- le chemin et nom de l'image (paramètre `shiny.image.repository`)
- le tag de l'image, i.e. sa version (paramètre `shiny.image.tag`)
- l'hostname de l'Ingress l'URL à laquelle l'application sera accessible une fois déployée (paramètre `shiny.ingress.hostname`)

### Token GEOAPIFY

La recherche d'adresse utilise le service [GEOAPIFY](https://www.geoapify.com/geocoding-api). 
Cela nécessite un _token_, stocké comme secret kubernetes, déclaré dans `values.yaml`, et défini [via](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-a-container-environment-variable-with-data-from-a-single-secret):

```shell
kubectl create secret generic geoapi-key --from-literal=geoapi-token=$GEOAPI_KEY
```

### Déploiement avec helm

Depuis la racine du _package_ :

```shell
helm dependency build helm/
helm install gd4h-eau helm/
```

Pour mettre à jour l'image utilisée (faut avoir `imagePullPolicy: Always` dans `values.yaml`):

```shell
kubectl rollout restart deployment gd4h-eau-shiny
```
