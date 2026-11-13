# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : MAPAKOU Charlestone-Junior**
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > La commande kubectl get nodes affiche 3 nœuds, k3d-lab-server-0 est le nœud de contrôle (control-plane), k3d-lab-agent-0 et k3d-lab-agent-1 sont les nœuds de travail (workers), qui exécutent les applications dans des Pods.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` affiche 4 conteneurs : `k3d-lab-server-0` pour le contrôle du cluster, `k3d-lab-agent-0` et `k3d-lab-agent-1` pour exécuter les applications, et `k3d-lab-serverlb` comme répartiteur de charge. Dans k3d, un nœud est un conteneur Docker exécutant K3s : on a donc 3 nœuds Kubernetes et 1 conteneur supplémentaire pour le répartiteur de charge.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > La commande kubectl create deployment web --image=nginx a créé le Deployment web, qui a automatiquement entraîné la création du ReplicaSet web-68d995574f, puis du Pod web-68d995574f-rgkrd exécutant Nginx
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Après la suppression du Pod web-68d995574f-rgkrd, un nouveau Pod nommé web-68d995574f-8s9bx a été créé automatiquement, puis est passé à l’état Running. Le ReplicaSet géré par le Deployment remplace le Pod supprimé pour maintenir le nombre de réplicas souhaité, ici une instance : c’est le mécanisme d’auto-réparation de Kubernetes.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non le message « Bienvenue sur mon site Web » n’a pas survécu à la suppression du pod car la modification était stockée uniquement dans le système de fichiers du conteneur, qui est éphémère. Le nouveau Pod repart de l’image Nginx d’origine et affiche donc la page par défaut
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le port-forward a cessé de fonctionner parce que le tunnel est relié à un pod précis. La suppression de ce pod coupe la connexion, donc il faut relancer un nouveau port-forward pour rétablir une connexion.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > kubectl create crée une nouvelle ressource et renvoie une erreur si elle existe déjà. kubectl apply applique l’état souhaité décrit dans un fichier YAML : il crée la ressource si elle n’existe pas ou la met à jour si elle existe déjà
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector d’un Service sert à identifier les Pods auxquels envoyer le trafic grâce à leurs labels, par exemple app: site. S’il ne correspond à aucun Pod, le Service existe toujours mais ne dispose d’aucun Pod cible : il ne peut donc pas transmettre le trafic à l’application.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
Les 3 réplicas sont répartis sur les 3 nœuds du cluster : site-bbb5b6c9f-gfcqn sur k3d-lab-agent-0, site-bbb5b6c9f-9fw2h sur k3d-lab-agent-1 et site-bbb5b6c9f-hn26g sur k3d-lab-server-0. Ils sont tous dans l’état Running avec 1/1 conteneur prêt.
   ```
   site-bbb5b6c9f-9fw2h   1/1     Running   0          2m10s   10.42.3.5    k3d-lab-agent-1    <none>           <none>
   site-bbb5b6c9f-gfcqn   1/1     Running   0          5m47s   10.42.0.10   k3d-lab-agent-0    <none>           <none>
   site-bbb5b6c9f-hn26g   1/1     Running   0          2m10s   10.42.1.6    k3d-lab-server-0   <none>   
      ```
11. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le Pod est passé par le statut ErrImagePull, puis ImagePullBackOff, car Kubernetes ne parvenait pas à télécharger l’image nginx:9.99, kubectl describe pod site-cf5c77748-brfhs permet d’identifier la cause exacte
12. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Non, dans mon cas, le site n’était plus accessible depuis le navigateur : le port transféré avait disparu dans Codespaces.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://stunning-invention-7vq7ppqjpw572rv5w-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > La commande kubectl get deployment mario -n default permet de retrouver le Deployment mario créé depuis Portainer. Oui, Portainer et kubectl communiquent avec le même composant : l’API Server de Kubernetes (kube-apiserver), qui expose l’API du cluster.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** 15 / 15
