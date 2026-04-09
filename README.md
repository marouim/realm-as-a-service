# Realm as a Service

ATTENTION: Le code contenu dans ce repo est pour des fins de tests et ne contient pas les bonnes pratiques en matière de sécurité de l'information. Un ajustement doit être fait afin de le rendre conforme aux standards de votre organisation. 


### Étapes à suivre pour tester le concept

1. Créer un projet (ex: keycloak)

2. Installer Operateur Red Hat build of Keycloak Operator dans le projet

3. Créer un secret pour la base de données PostgreSQL

Note: Une méthode plus sécuritaire avec clés externalisées est recommendée. 
```
apiVersion: v1
kind: Secret
metadata:
  name: keycloak-db-secret
type: Opaque
stringData:
  username: [nom d'usager]
  password: [mot de passe]
```

4. Créer une base de donnée PostgreSQL

```
oc apply -f db.yaml
```

5. Créer le secret contenant les certificats

Note: Une méthode plus sécuritaire avec clés externalisées est recommendée. 

```
kind: Secret
apiVersion: v1
metadata:
  name: my-tls-secret
data:
  tls.crt: 
  tls.key: 
type: kubernetes.io/tls
```

6. Créer une instance de Keycloak

Note: Ajuster les paramètres du manifest keycloak.yaml tels que le hostname et autres configurations spécifiques. 

```
oc apply -f keycloak.yaml
```

7. Vérifier le fonctionnement de keycloak

Par défaut, un secret est automatiquement créé pour l'accès administration temporaire. 

```
oc get secret | grep initial-admin
example-keycloak-initial-admin       kubernetes.io/basic-auth   2      29h
```

8. Créer un nouveau realm "as-a-service"

Pour créer un nouveau REALM, appliquer le CRD KeycloakRealmImport suivant: 

Ce CRD doit être dans le même namespace que l'instance keycloak. 

Ajuster le manifest KeycloakRealmImport avec le nom exact de l'unité d'affaires et 

```
oc apply -f realm-bu1.yaml
```

Une fois appliqué, un pod (job) sera crée ex: 

```
oc get pods | grep bu-1
realm-bu-1-sxc7m                1/1     Running     0          27s
```

Attendre que le pod soit complété

```
oc get pods | grep bu-1
realm-bu-1-sxc7m                0/1     Completed   0          68s
```

Vérifier l'accès au niveau realm (route de keycloak ajustée avec le nom du realm)

Exemple: 

```
https://keycloak-keycloak.apps.cy0ho9ko.eastus.aroapp.io/admin/realm-bu-1/console/
```

Connecter en utilisant le mot de passe temporaire contenu dans le manifest realm-bu1.yaml. 

ATTENTION: Le mot de passe dans le manifest n'est pas une bonne pratique. Il s'agit d'une configuration d'exemple pour laboratoire. 
