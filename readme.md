VIDEO: STRATEGIE DE DEPLOYMENT BLUE-GREEN

OUTILS:

- cluster k8s multi-noeud

- manifests yaml de drupal blue-green dans vscode

- manifests yaml de drupal blue-green dans la vm master

ETAPES

- Rappels sur pod, ReplicaSet, Deployment

- evoquer la notion de montée en version des deployments UPGRADE / DOWNGRADE

- Présentation sommaire de quelques stratégies de déploiements

- Présentation de la stratégie blue-green

- Présentation de l'architecture à mettre en oeuvre

- Pratique

- installation du nfs-server

  + NAMESPACE

kubectl create ns storage

  + DEPLOYMENT

kubectl apply -f https://raw.githubusercontent.com/appscode/third-party-tools/master/storage/nfs/artifacts/nfs-server.yaml

  + SERVICE

kubectl apply -f https://raw.githubusercontent.com/appscode/third-party-tools/master/storage/nfs/artifacts/nfs-service.yaml

  + AJOUT DES DOSSIERS DRUPAL & MYSQL

kubectl -n storage exec -t -i $(kubectl -n storage get pod -l app=nfs-server -o jsonpath="{.items[0].metadata.name}") -- mkdir /exports/{drupal,mysql}

  + IP DU NFS-SERVICE

kubectl -n storage get service/nfs-service -o jsonpath="{.spec.clusterIP}"

- Déploiement de drupal en version blue

  + NAMESPACE

kubectl create ns drupal

  + MODIFIER LES FICHIERS PV.YAML POUR METTRE LA BONNE ADRESSE DU NFS-SERVICE !!!

  + CREATION DES RESSOURCES

kubectl -n drupal apply -f .

  + RECUPERER LE NODE_PORT ET JOINDRE LE NAVIGATEUR

kubectl -n drupal get svc

http://<NODE-IP>:<NODE-PORT>

-- INSTALLATION DE DRUPAL 9.4 DANS LE NAVIGATEUR

	Problème d'écriture dans le dossier drupal !?!

-- NOM DU NOEUD >> NFS - ATTRIBUER LES DROITS AU DOSSIER DRUPAL

	vagrant ssh worker

	sudo su -

	cd /data/nfs

	ls -l drupal

	chmod 777 drupal

--REVENIR DANS LE NAVIGATEUR POUR TERMINER LA CONFIG

- Déploiement de drupal en version green

  + CREATION DU DEPLOYMENT & TEST

kubectl -n drupal apply -f green/drupal-green-deployment.yaml

"kubectl port-forward" pour s'assurer que la version green fonctionne

  + BASCULEMENT VERS LA VERSION GREEN

kubectl -n drupal apply -f green/drupal-service.yaml

  + VERIFIER LES SELECTIONS DU SERVICE

kubectl -n drupal describe svc drupal

JOINDRE LE NAVIGATEUR
http://<NODE-IP>:<NODE-PORT>

CONFIGURER LA VERSION 9.5, LE RESTE C'EST POUR LES ADMINISTRATEURS DRUPAL §§§