Projet CloudShop - Guide de Déploiement CompletCe document permet de déployer l'intégralité de l'infrastructure CloudShop de manière autonome, de la création du cluster jusqu'aux tests de validation.

🏗️ 1. Création du Cluster KindLe cluster est composé de 1 Control-Plane et 3 Workers pour permettre une isolation réelle.

Commande : 
- kind create cluster --config kind-config.yaml --name cloudshop-cluster

🛡️ 2. Installation du Réseau et de la Sécurité (CNI Calico)Par défaut, Kind ne gère pas les NetworkPolicies. Il faut installer Calico pour que le "pare-feu" fonctionne.

COMMANDES (l'une après l'autre) :
- kubectl apply -f 
- kubectl delete pods -n kube-system -l k8s-app=kindnet

🚀 3. Déploiement de l'InfrastructureIl est crucial de respecter cet ordre pour que les ressources se lient correctement.

COMMANDES : 
- Création Namespace : kubectl create namespace cloudshopStockage
- Stocakge Persistant : kubectl apply -f storage/
- Droits d'accès (RBAC) : kubectl apply -f rbac/Pare-feu 
- Pare-feu (NetworkPolicies) : kubectl apply -f networkpolicy/
- Exposition (Services) : kubectl apply -f services/
- Applications (Deployments) : kubectl apply -f deployments/
- Routage Externe (Ingress) : kubectl apply -f ingress/

🧪 4. Tests de Validation 

A. Test de Sécurité (Doit afficher "Connection timed out")

Le Frontend ne peut pas attaquer la base de données en direct.

COMMANDE :
kubectl exec -it $(kubectl get pod -l tier=frontend -n cloudshop -o name | head -n 1) -n cloudshop -- curl --connect-timeout 2 db-service:3306

B. Test de Persistance (Survie des données)

Il faut supprimer le pod de la base de données et vérifier que les données reviennent.

COMMANDES :

- Supprimer le pod : kubectl delete pod -l app=database -n cloudshop
- Vérifier les bases après 10 secondes : kubectl exec -it $(kubectl get pod -l app=database -n cloudshop -o name) -n cloudshop -- bash -c 'mysql -u root -p$MYSQL_ROOT_PASSWORD -e "SHOW DATABASES;"'

C. Test de Droits (RBAC)

On vérifie que le backend a uniquement les droits nécessaires.

COMMANDES :
- Test de lecture (Doit dire YES) : kubectl auth can-i get pods --as=system:serviceaccount:cloudshop:backend-sa -n cloudshop
- Test de destruction (Doit dire NO) : kubectl auth can-i delete nodes --as=system:serviceaccount:cloudshop:backend-sa

🌐 5. Accès au Site WebPour ouvrir le site CloudShop sur ton navigateur Windows :
COMMANDE : 
- kubectl port-forward svc/frontend-service -n cloudshop 8080:80

URL À OUVRIR :http://localhost:8080

Auteur : Louis GAUTIER