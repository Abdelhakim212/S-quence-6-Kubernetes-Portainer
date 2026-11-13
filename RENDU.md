# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :**
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >Control plane : k3d-lab-server-0 (rôle control-plane,master).
Workers : k3d-lab-agent-0 et k3d-lab-agent-1 (rôle <none>).
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >Chaque nœud est un conteneur Docker qui exécute k3s, plus un conteneur k3d-lab-serverlb qui sert de load balancer.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >Dans le namespace kube-system : coredns, local-path-provisioner, metrics-server, traefik et svclb-traefik.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >le Deployment (via son ReplicaSet) maintient le nombre de réplicas voulu (auto-réparation).

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >Non, le message a disparu : le conteneur est éphémère, le nouveau pod repart de l’image d’origine et la modification faite avec exec était perdue avec l’ancien pod.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Le port-forward est attaché à un pod précis ; ce pod ayant été supprimé, la connexion est coupée.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >kubectl create crée une ressource et échoue si elle existe déjà (impératif), alors que kubectl apply -f crée ou met à jour selon le YAML (déclaratif).
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le selector désigne les pods (par leurs labels) vers lesquels le Service envoie le trafic ; s’il ne correspond à aucun pod, le Service n’a aucun endpoint et le trafic n’aboutit pas

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
Lance kubectl get pods -l app=site -o wide et colle la sortie (colonne NODE) : les 3 réplicas sont répartis entre k3d-lab-agent-0 et k3d-lab-agent-1 (parfois server-0), au choix du scheduler.

   
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >Statut ErrImagePull puis ImagePullBackOff (image introuvable) ; la cause apparaît avec kubectl describe pod <nom> (section Events).
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >Oui, le site restait accessible : le rolling update garde les anciens pods fonctionnels tant que les nouveaux ne sont pas Ready.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://potential-yodel-4qrr75g99p7pf7w9j-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >Je retrouve le Deployment avec kubectl get deploy mario. Portainer et kubectl parlent au même composant : l’API Server du cluster (le control plane, k3d-lab-server-0). Le Deployment créé par Portainer est un Deployment Kubernetes ordinaire, comme ceux créés en ligne de commande

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >A : l'Ingress (Traefik, port 8081) garde le site accessible après suppression des pods, car il passe par le Service, qui envoie le trafic vers les pods disponibles, alors que le port-forward est attaché à un seul pod.
B : readinessProbe et resources ajoutées. Avec un chemin /nope, nginx répond 404, la probe échoue, le pod reste Running mais 0/1 Ready et ne reçoit aucun trafic ; lors d'un rolling update, les anciens pods restent en service.
C : après le drain, les pods de agent-0 sont recréés sur les autres nœuds. Après uncordon, ils ne reviennent pas tout seuls : le scheduler ne place que les nouveaux pods et ne déplace pas ceux qui tournent déjà (un rollout restart les redistribue).
D : le namespace prod contient une seconde copie du site, isolée de default ; les mêmes noms coexistent sans conflit.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
