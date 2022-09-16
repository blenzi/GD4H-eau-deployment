# Déploiement de l'application Prelshiny Golem

Ce dépot permet le déploiement l'application [Prelshiny Golem](https://gitlab-forge.din.developpement-durable.gouv.fr/dreal-normandie/pole-renovation-energetique/prelshiny_golem) sur un cluster Kubernetes. La configuration présentée est spécifiquement adaptée au [SSP Cloud](https://datalab.sspcloud.fr/home).

#### Chart Helm

Le déploiement de l'application se fait via un chart Helm, basé sur le [template de l'application shiny-app](https://github.com/InseeFrLab/helm-charts-shiny-apps).

Le fichier `Chart.yaml` contient les métadonnées du chart (nom, version) ainsi que ses dépendances, i.e. les potentiels autres charts Helm dont il hérite. Dans notre cas, on voit que le chart hérite du [chart Shiny](https://github.com/InseeFrLab/helm-charts/tree/master/charts/shiny) d'InseeFrLab. Ce chart spécifie généralement les ressources Kubernetes nécessaires au déploiement d'une application Shiny, de sorte à ce que l'on ait qu'à modifier les valeurs d'instanciation pour déployer notre application.

Le fichier `values.yaml` contient précisément les valeurs que l'on modifie par rapport au chart général. Les modifications à apporter dépendent naturellement de ce que réalise en pratique l'application, car cela conditionne les ressources dont elle a besoin. Dans un premier temps, il nous faut modifier : 
- le chemin et nom de l'image (paramètre `shiny.image.repository`)
- le tag de l'image, i.e. sa version (paramètre `shiny.image.tag`)
- l'hostname de l'Ingress l'URL à laquelle l'application sera accessible une fois déployée (paramètre `shiny.ingress.hostname`)

La branche `main` de ce package correspond au déploiement de la branche `main` de [Prelshiny Golem](https://gitlab-forge.din.developpement-durable.gouv.fr/dreal-normandie/pole-renovation-energetique/prelshiny_golem), sur https://prel.lab.sspcloud.fr/ et de manière analogue la branche `dev` est déployée sur https://prel-dev.lab.sspcloud.fr/.

### Récupération de l'image docker

L'image docker de [Prelshiny Golem](https://gitlab-forge.din.developpement-durable.gouv.fr/dreal-normandie/pole-renovation-energetique/prelshiny_golem) est générée automatiquement par gitlab et stockée sur le registre du ministère (registry.gitlab-forge.din.developpement-durable.gouv.fr). Pour la récupérée il faut une autentification via un token de groupe, crée sur le dépôt gitlab et gardé en tant que secret kubernetes en utilisant [cette recette](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line). Le nom d'utilisateur correspond au nom du token et le mot de passe à sa valeur.

Ensuite la configuration de déploiement (`values.yaml`) contient la référence au secret:

```yaml
imagePullSecrets:
  - name: regcred
```

### Déploiement avec helm

Depuis la racine du _package_ :

```shell
helm install prel-dev helm/  # ou prel
```

Pour mettre à jour l'image utilisée (faut avoir `imagePullPolicy: Always` dans `values.yaml`):

```shell
kubectl rollout restart deployment prel-dev
```
