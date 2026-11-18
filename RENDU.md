# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** NDTOUNGOU SCHOUAME Fabrice
**Groupe :** RS2  

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > `kubectl get nodes` affiche **3 nœuds** en état `Ready` :
   > - `k3d-lab-server-0` : rôle **`control-plane`** (le nœud maître / master qui héberge l'API Server, le scheduler, le controller-manager et le stockage de l'état du cluster) ;
   > - `k3d-lab-agent-0` et `k3d-lab-agent-1` : rôles **workers** (`<none>` dans la colonne `ROLES`), chargés d'exécuter les pods applicatifs.

2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` affiche **4 conteneurs Docker** actifs : `k3d-lab-server-0`, `k3d-lab-agent-0` et `k3d-lab-agent-1` (basés sur l'image `rancher/k3s`), ainsi que `k3d-lab-serverlb` (le load balancer/proxy qui expose les ports du cluster vers l'hôte).
   > Concrètement, dans **k3d**, un nœud Kubernetes est un **conteneur Docker** exécutant une instance légère et certifiée de Kubernetes (**k3s**).

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > La commande `kubectl create deployment web --image=nginx` a créé trois ressources liées :
   > - un **`Deployment`** (`deployment.apps/web`), qui définit l'état désiré et pilote les mises à jour ;
   > - un **`ReplicaSet`** (`replicaset.apps/web-...`), créé automatiquement par le Deployment pour maintenir le nombre de réplicas demandé ;
   > - un **`Pod`** (`pod/web-...`), instancié par le ReplicaSet et contenant le conteneur `nginx`.
   > *(Le `Service` `service/web` a ensuite été créé par la commande `kubectl expose`.)*

4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Dès la suppression du pod (`Terminating`), Kubernetes a immédiatement recréé un **nouveau pod avec un suffixe différent**, qui est passé en état `Running`.
   > **Pourquoi :** Le `Deployment` (via son `ReplicaSet`) exécute en continu la **boucle de réconciliation** de Kubernetes : il compare l'état réel (`0` pod actif) à l'état désiré (`1` réplica) et recrée automatiquement un pod pour combler l'écart (auto-réparation / *self-healing*).

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > **Non**, le message a disparu et la page par défaut `Welcome to nginx!` est revenue.
   > **Pourquoi :** Le système de fichiers d'un conteneur est **éphémère**. Lorsqu'un pod est supprimé et recréé, Kubernetes démarre un conteneur neuf à partir de l'image d'origine (`nginx`), ce qui efface toute modification effectuée manuellement à l'intérieur.

6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Bien qu'il cible le Service (`svc/web`), `kubectl port-forward` s'attache en réalité à un **pod précis** derrière ce Service. Lorsque ce pod a été supprimé, la connexion TCP sous-jacente a été rompue (`lost connection to pod`) et le tunnel s'est fermé.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `kubectl create` est une commande **impérative** qui crée une ressource une seule fois et échoue si elle existe déjà.
   > `kubectl apply` est une approche **déclarative** (et idempotente) : on décrit l'état cible dans des manifestes YAML versionnés dans Git, et Kubernetes crée les ressources manquantes ou applique uniquement les modifications nécessaires sur les ressources existantes.

8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le `selector` permet au Service d'identifier dynamiquement, via leurs **labels** (`app: site`), les pods prêts (`Ready`) vers lesquels il doit router et répartir le trafic réseau.
   > S'il ne correspond à aucun pod, le Service existe toujours mais sa liste d'**`Endpoints`** est vide (`<none>`) : les requêtes envoyées au Service n'aboutissent pas.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? 
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE               NOMINATED NODE   READINESS GATES
site-54dbc4f754-drl5f   1/1     Running   0          5m33s   10.42.2.7    k3d-lab-server-0   <none>           <none>
site-54dbc4f754-vppvn   1/1     Running   0          5m58s   10.42.0.8    k3d-lab-agent-0    <none>           <none>
site-54dbc4f754-xs2zs   1/1     Running   0          5m44s   10.42.1.12   k3d-lab-agent-1    <none>           <none>

**Score obtenu avec `./scripts/check.sh` :** 18 / 18