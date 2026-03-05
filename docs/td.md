## **Les principaux binaires de PostgreSQL**

### **pg\_ctl (réalisé par les installateurs comme systemD ou celui de windows)**

* gestion de l'instance / cluster  
* start / stop / kill  
* init : création autre espace de datas  
* promote : promotion de standby

### **psql**

* le premier client de connexion en mode CLI  
* on se connecte avec un utilisateur à une base de donnée  
* on peux exécuter du SQL et des méta-commandes, ou des script sql (fichiers)

### ***Exemple Pg\_createcluster***

* création d'une instance PG  
* Création des répertoires (/etc et /var/lib)

Accompagné de pg\_dropcluster (suppression de cluster), pg\_lscluster (lister), pg\_ctlcluster (contrôleur du cluster)

### ***Définition*****Les binaires pour la sauvegarde**

**pg\_dump**

* sauvegarde d'une instance  
* différents formats : plain text (DDL schéma et SQL), binaire (lisible seulement par PG), compressé  
* différents niveaux d'objets (cluster, db, table, schéma)

**pg\_dumpall**

* sauvegarde intégrale en format binaire

**pg\_restore**

* restauration à partir d'une sauvegarde pg\_dumpall

### ***Complément Wrappers*** **(des raccourcis)**

* createdb  
* dropdb  
* createuser  
* dropuser

### ***Complément Maintenance***

* reindexdb : réindexation  
* vaccumdb : tâche de maintenance (ménage)

### ***Attention Spécifiques*** **système avancées**

* pg\_controldata : vérifie l'état du serveur et des infos critiques  
* pg\_resetwal : en cas de crash avec des pb de WAL (risques de pertes de données)  
* pg\_receive\_wal : récupération des WAL d'une autre DB (réplication)  
* pg\_basebackup : récupération de datas par une autre connexion à une autre DB (réplication)

## **Maintenance \- Vacuum et autre dashboard**

Récupère l'espace inutilisé et, optionnellement, analyse une base.

Lors des opérations normales de PostgreSQL, les lignes supprimées ou rendues obsolètes par une mise à jour ne sont pas physiquement supprimées de leur table.

Vacuum permet de récupérer l'espace occupé par les lignes supprimées.

Le **VACCUM** standard (sans FULL) récupère simplement l'espace et le rend disponible pour une réutilisation. Cette forme de la commande peut opérer en parallèle avec les opérations normales de lecture et d'écriture de la table, car elle n'utilise pas de verrou exclusif.

**VACCUM FULL** fait un traitement plus complet et, en particulier, déplace des lignes dans d'autres blocs pour compacter la table au maximum sur le disque. Cette forme est beaucoup plus lente et pose un verrou exclusif sur la table pour faire son traitement.

Des VACUUM standard et d'une fréquence modérée sont une meilleure approche que des VACUUM FULL, même non fréquents, pour maintenir des tables mises à jour fréquemment.

Après avoir ajouté ou supprimé un grand nombre de lignes, il peut être utile de faire un **VACUUM ANALYZE** sur la table affectée. Cela met les catalogues système à jour de tous les changements récents et permet à l'optimiseur de requêtes de PostgreSQL™ de faire de meilleurs choix lors de l'optimisation des requêtes.

Pour exécuter un VACUUM sur une table, vous devez habituellement être le propriétaire de la table ou un super utilisateur. Les propriétaires de la base de données sont autorisés à exécuter VACUUM sur toutes les tables de leurs bases de données, sauf sur les catalogues partagés. Cette restriction signifie qu'un vrai VACUUM sur une base complète ne peut se faire que par un super utilisateur.

Il est recommandé que les bases de données actives de production soient traitées par VACUUM fréquemment (au moins toutes les nuits), pour supprimer les lignes mortes.

### **Auto-vacuum**

Automatiser l'exécution des commandes VACUUM et ANALYZE .

Une fois activé, le démon autovacuum s'exécute périodiquement et vérifie les tables ayant un grand nombre de lignes insérées, mises à jour ou supprimées. Ces vérifications utilisent la fonctionnalité de récupération de statistiques au niveau ligne.

Dans la configuration par défaut, l'autovacuum est activé et les paramètres liés sont correctement configurés.

### **Dashboard \- Visuel sur les indicateurs de la base**

#### ***Exemple*****En SQL**

**Taille des fichiers**

Pour info :

La TOAST \=\> grand objet comme geom

CTRL+C pour copier, CTRL+V pour coller

1

SELECTrelname as "Table",pg\_size\_pretty(pg\_total\_relation\_size(relid)) As "Size",

2

pg\_size\_pretty(pg\_total\_relation\_size(relid) \- pg\_relation\_size(relid)) as "External Size"

3

FROM pg\_catalog.pg\_statio\_user\_tables ORDER BY pg\_total\_relation\_size(relid) DESC;

4

Pour info :

La TOAST \=\> grand objet comme geom

