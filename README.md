# Pronote dans Homeassistant
Ceci est un tutoriel pour intégrer Pronote dans Home assistant !  
C'est une intégration mais qui n'est pas plug and play comme les inétégrations standard de HA.  
Elle se base sur un script python que j'ai développé et qui est basé sur l'API wrapper [pronotepy](https://github.com/bain3/pronotepy).  
 
![Lovelace](screen-pronote1.png?raw=true "Screen Shot")

### Nouveautés - 16/10/2022
- Externalisation de la configuration dans un fichier de [config.ini] qui permet de configurer plusieurs comptes pour plusieurs enfants - merci à [pelrol](https://github.com/pelrol)

### Nouveautés - 27/03/2022
- Connexion via les ENT (si celle-ci est développée dans la lib pronotepy)
- Ajout de la gestion des comptes "parents" de PRONOTE pour se connecter (plus uniquement les comptes elèves dont on n'a pas toujours le mot de passe) 
- Ajout des absences : dans le script et dans le lovelace 
<img src="screen-absence.png?raw=true" alt="absence" width="300"/>


# Rappel : 
Pronote est une application en ligne déployée dans plusieurs milliers de collèges et lycées français.  
Elle permet aux élèves de voir leurs emploi du temps, leurs notes et leurs devoirs.  
Vous pouvez voir une démo de l'application à l'adresse suivante :  

https://demo.index-education.net/pronote/eleve.html?login=true 
utlisateur : demonstration 
mot de passe : pronotevs 

L'idée consiste donc à remonter ces informations (emploi du temps, note, devoirs, absence...) dans HA pour créer des automatisations comme :   
- Régler l'heure de son réveil avec l'heure de début des cours du lendemain
- Envoyer une notification (mobile, sms ou autre) si l'élève a une nouvelle note 
- Envoyer une notification si un cours est annulé (voir décaler l'heure du réveil)
- Envoyer une notification en cas d'absence


## Pré-requis :
- Connaitre un minimum HomeAssistant
- Avoir quelques bases en Python
- Savoir installer un script python sur la VM qui héberge HA

## Principe de base : 
- Un script python qui se connecte avec la lib pronotepy à Pronote et récupère les données dans un fichier JSON
- Plusieurs sensor REST dans la configuration de HA qui se connectent en local au fichier JSON récupéré 
- Des Card Markdown dans Lovelace pour afficher les données 

## 1. Installation de la lib pronotepy 

Le script python que j'ai développé dépend d'une librairie python développé en open source : [pronotepy](https://github.com/bain3/pronotepy)
(merci à [bain3](https://github.com/bain3).
C'est cette librairie qui permet par exemple que le script se connecter à Pronote via les ENT (s'ils ont été implémenté et suportés par bain3)

Pour installer cette librairie sur votre systeme, je vous conseille d'utiliser la commande pip ou pip3.  
Commande : pip install pronotepy  
Recommandation : il est important de la mettre à jour régulièrement cette librairie.  
Commande : pip install --upgrade pronotepy  

## 2. Installation du script python 

Je mets donc à disposition un script python [pronote.py](pronote.py).  
Ce script permet de se connecter à Pronote et récupère toutes les informations dans un JSON.  
Ce script utilise un fichier de configuration [config.ini] qui est initialisé avec le compte de demo de Pronote > reste à l'adapter à vos identifiants en changeant les variables dans la section defaut.
Et si vous avez plusieurs enfants, ajouter une nouvel section comme    
Il faut donc installer ce script ainsi que le fichier de configuration dans un dossier (nommé par exemple "python_script") dans le dossier /config de votre HA.  
Ce script quand il est lancé génère un fichier pronote_AAAA.json qu'il dépose dans /config/www/ de votre HA.
NB : AAAA est le nom de l'élève à paramétré dans le fichier de configuration.

## 3. Exécution du script 

Pour lancer le script il faut avoir accès en SSH et se placer dans le dossier créé précédement
    cd /usr/share/hassio/homeassistant/python_scripts/
Et lancer le script avec la commande 
    python3 pronote.py 
Et si vous avez créé une section pour un 2eme enfant dans le config.ini, alors : 
    python3 pronote.py enfant1

Il doit ensuite être lancé de façon régulière - toute les 5 ou 10 minutes - via la crontab par exemple.  
Exemple : 
    */10 * * * * /usr/bin/python3 /usr/share/hassio/homeassistant/python_scripts/pronote.py > /tmp/pronote.log 2>&1  

## 4. Configuration YAML pour récupérer l'emploi du temps dans un sensor

Je fourni donc un fichier configuration.yaml [configuration.yaml](configuration.yaml)  à copier à l'intérieur de celui de votre HA.  
Attention : N'oubliez pas de mettre la bonne adresse IP interne de votre HA (ne pas laisser 192.168.XX.XX). 
Et renommer pour changer "demo" avec le nom qu vous voulez (votre enfant).  


## 5. Test des sensors avec le Developper Tools de HA

Tester les sensors qui commencent par "pronote" avec  le Developer Tools de Home assistant

<img src="screen-devtools.png?raw=true" alt="devtools" width="300"/>



## 6. Affichage dans les cartes Lovelace

Il suffit de créer un nouvel onglet dans HA et ensuite de copier coller le contenu du fichier lovelace.yaml [lovelace.yaml](lovelace.yaml) 

ça doit donner ça : 

![Lovelace](screen-pronote2.png?raw=true "Screen Shot avec début des cours")

## 7. Notification

Vous trouverez dans le fichier automation.yaml des automatisations qui permettent d'être notifié (notif persistant pour l'exemple) en cas de : 

- cours annulé aujour'hui
- cours annulé dans du prochain jour de cours 
- Nouvelle note 

![Lovelace](screen-pronote-notif.png?raw=true "Screen Notif")

il vous suffit de créer une automatisation vide et de copier/coller chaque automatisation en mode d'edition yaml
