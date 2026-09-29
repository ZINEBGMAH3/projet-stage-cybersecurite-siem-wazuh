# J6 — Surveillance et détection des événements de sécurité Linux

## 1. Objectif

L'objectif de cette journée est de mettre en œuvre et de valider la surveillance de sécurité du serveur Linux `wazuh-siem` à l'aide de Wazuh.

Cette étape consiste à vérifier que les événements liés aux authentifications SSH sont correctement :

* générés par le système Linux ;
* enregistrés dans les journaux système ;
* collectés par Wazuh ;
* décodés et analysés ;
* associés à des règles de détection ;
* transformés en alertes de sécurité.

Un scénario contrôlé d'échec d'authentification SSH a été utilisé afin de vérifier le fonctionnement de cette chaîne de détection.

---

## 2. Environnement de travail

Le serveur Linux utilisé pour cette étape est la machine virtuelle Wazuh :

* Nom d'hôte : `wazuh-siem`
* Utilisateur Linux : `zineb`
* Service SSH : OpenSSH Server
* Port SSH : `22`
* Serveur Wazuh : `wazuh-siem`
* Agent ID : `000`
* Adresse IP source observée lors du test : `10.0.2.2`

L'agent ID `000` correspond au serveur Wazuh lui-même. Dans ce scénario, le serveur Wazuh surveille donc ses propres événements Linux.

---

## 3. Vérification du service SSH

La première vérification a consisté à contrôler l'état du service SSH avec la commande :

```bash
sudo systemctl status ssh --no-pager
```

Le résultat montre :

```text
ssh.service - OpenBSD Secure Shell server
Loaded: loaded
Active: active (running)
```

Le service SSH est donc actif et fonctionne correctement.

Le système indique également :

```text
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
```

Cela signifie que le serveur SSH écoute sur le port 22 pour les connexions IPv4 et IPv6.

Le service est également indiqué comme `enabled`, ce qui signifie qu'il est configuré pour démarrer automatiquement avec le système.

---

## 4. Journalisation des événements d'authentification

Les événements d'authentification Linux sont disponibles dans le journal système.

Le fichier `/var/log/auth.log` a été vérifié avec :

```bash
ls -l /var/log/auth.log
```

Le fichier existe et contient les événements relatifs aux authentifications et aux opérations de sécurité.

Pour observer les nouveaux événements en temps réel, la commande suivante a été utilisée :

```bash
sudo tail -f /var/log/auth.log
```

Cette commande permet de suivre les nouvelles lignes ajoutées au journal sans avoir à ouvrir régulièrement le fichier.

---

## 5. Scénario contrôlé d'échec d'authentification SSH

Afin de tester la détection, une connexion SSH vers le serveur Linux a été réalisée avec le compte `zineb`.

La commande utilisée depuis Windows est :

```powershell
ssh zineb@127.0.0.1 -p 2223
```

Une fois la demande de mot de passe affichée, un mot de passe incorrect a volontairement été saisi.

L'objectif était uniquement de générer un événement contrôlé permettant de vérifier le fonctionnement de la supervision.

Le serveur Linux a immédiatement enregistré l'événement dans son journal.

Le message observé était :

```text
Failed password for zineb from 10.0.2.2 port 65151 ssh2
```

Un autre événement associé a également été enregistré :

```text
pam_unix(sshd:auth): authentication failure
```

Ces deux événements montrent qu'une tentative d'authentification SSH a échoué.

---

## 6. Vérification de l'événement dans le journal Linux

La présence de l'événement dans le journal a été vérifiée avec :

```bash
sudo grep "Failed password" /var/log/auth.log | tail -5
```

Le résultat a confirmé la présence de l'événement :

```text
2026-09-29T02:29:36.278833+00:00 wazuh-siem sshd[4948]: Failed password for zineb from 10.0.2.2 port 65151 ssh2
```

Cette ligne contient plusieurs informations importantes.

`sshd[4948]` indique que l'événement provient du processus SSH (`sshd`).

`zineb` correspond au compte utilisé pour la tentative de connexion.

`10.0.2.2` correspond à l'adresse IP source observée.

`65151` correspond au port source utilisé pour cette connexion.

`ssh2` indique que la connexion utilise SSH version 2.

Le numéro `4948` correspond au processus `sshd` ayant traité cette tentative.

---

## 7. Collecte et analyse par Wazuh

Après la génération de l'événement, celui-ci a été retrouvé dans le Dashboard Wazuh.

L'événement possède notamment les champs suivants :

```text
agent.id       = 000
agent.name     = wazuh-siem
data.dstuser   = zineb
data.srcip     = 10.0.2.2
data.srcport   = 65151
decoder.name   = sshd
decoder.parent = sshd
location       = journald
```

Le champ `agent.name` indique que l'événement provient du serveur `wazuh-siem`.

Le champ `agent.id` égal à `000` indique que le serveur Wazuh analyse ici ses propres journaux.

Le champ `data.dstuser` indique le compte ciblé par la tentative d'authentification.

Le champ `data.srcip` correspond à l'adresse IP source observée.

Le champ `decoder.name` indique que Wazuh a reconnu l'événement comme un événement SSH.

Le champ `location` indique que l'événement a été récupéré depuis le système de journalisation `journald`.

Le champ `full_log` contient le message original :

```text
Sep 29 02:29:36 wazuh-siem sshd[4948]: Failed password for zineb from 10.0.2.2 port 65151 ssh2
```

---

## 8. Détection par la règle Wazuh 5760

L'événement a été associé à la règle Wazuh :

```text
ID: 5760
Level: 5
Description: sshd: authentication failed
File: 0095-sshd_rules.xml
```

La règle appartient notamment aux groupes :

```text
authentication_failed
syslog
sshd
```

La règle contient la condition :

```text
if_sid: 5700,5716
```

et recherche notamment les motifs :

```text
Failed password
Failed keyboard
authentication error
```

Dans notre scénario, le message contient :

```text
Failed password
```

La règle 5760 correspond donc directement à l'événement observé.

Le niveau 5 indique le niveau d'alerte attribué par Wazuh à cet événement.

---

## 9. Détection par la règle PAM 5503

Le même scénario a également été associé à une règle PAM :

```text
ID: 5503
Level: 5
Description: PAM: User login failed
File: 0085-pam_rules.xml
```

Les groupes associés sont :

```text
authentication_failed
pam
syslog
```

La règle possède la condition :

```text
if_sid: 5500
```

et recherche le motif :

```text
authentication failure; logname=
```

Cette règle permet d'identifier les événements d'échec d'authentification provenant de PAM.

PAM signifie « Pluggable Authentication Modules ». Il s'agit d'un mécanisme utilisé par Linux pour gérer différentes opérations d'authentification.

---

## 10. Correspondance MITRE ATT&CK

La règle 5503 indique une correspondance avec la technique MITRE ATT&CK :

```text
Password Guessing
```

Cette association signifie que le type d'événement détecté peut être lié à des tentatives de deviner un mot de passe.

Cependant, le scénario réalisé dans ce projet ne constitue pas une attaque par force brute ou une campagne réelle de password guessing.

Une seule tentative incorrecte a volontairement été réalisée dans un environnement contrôlé afin de vérifier la capacité de détection de la plateforme SIEM.

La correspondance MITRE doit donc être interprétée comme une **classification technique du type d'événement**, et non comme la preuve qu'une attaque réelle par password guessing a eu lieu.

---

## 11. Chaîne complète de détection

Le scénario permet de démontrer la chaîne de supervision suivante :

```text
Tentative de connexion SSH
          ↓
Mot de passe incorrect
          ↓
Service sshd
          ↓
PAM / authentification Linux
          ↓
Journal système
          ↓
Collecte par Wazuh
          ↓
Decoder sshd / PAM
          ↓
Règle 5503
          ↓
Règle 5760
          ↓
Alerte Wazuh niveau 5
          ↓
Visualisation dans le Dashboard
```

Cette chaîne montre le fonctionnement d'un SIEM depuis la génération de l'événement jusqu'à sa transformation en alerte exploitable par un analyste.

---

## 12. Analyse de sécurité

L'intérêt de cette détection est de permettre à un analyste de surveiller les tentatives d'authentification échouées sur un serveur Linux.

Une tentative isolée peut correspondre à une erreur légitime de saisie du mot de passe. Elle ne permet donc pas, à elle seule, de conclure à une attaque.

En revanche, plusieurs échecs répétés provenant d'une même adresse IP, ciblant plusieurs comptes ou apparaissant sur plusieurs services pourraient constituer un signal nécessitant une analyse complémentaire.

Dans une infrastructure réelle, l'analyse pourrait notamment prendre en compte :

* le nombre d'échecs ;
* leur fréquence ;
* l'adresse IP source ;
* le compte ciblé ;
* les horaires ;
* les succès d'authentification précédant ou suivant les échecs ;
* d'autres événements de sécurité associés.

La corrélation de plusieurs événements permettrait ainsi de distinguer plus efficacement une erreur utilisateur d'un comportement potentiellement malveillant.

---

## 13. Captures réalisées

Les captures suivantes ont été réalisées et ajoutées au dépôt GitHub.

### Capture J6-1 — Événement SSH détecté

Fichier :

```text
screenshots/j6/j6-01-detection-echec-authentification-ssh.png
```

Cette capture montre les détails de l'événement SSH détecté par Wazuh, notamment le compte ciblé, l'adresse IP source, le decoder `sshd` et le message `Failed password`.

### Capture J6-2 — Règle SSH Wazuh

Fichier :

```text
screenshots/j6/j6-02-regle-wazuh-authentification-ssh.png
```

Cette capture présente la règle `5760`, son niveau 5, ses groupes et les motifs utilisés pour détecter les échecs d'authentification SSH.

### Capture J6-3 — Règle PAM Wazuh

Fichier :

```text
screenshots/j6/j6-03-regle-pam-authentification-echouee.png
```

Cette capture présente la règle `5503`, son niveau 5, ses groupes, sa condition de détection et la correspondance MITRE ATT&CK `Password Guessing`.

---

## 14. Résultats

Les tests réalisés permettent de confirmer les points suivants :

| Élément                                 | Résultat                           |
| --------------------------------------- | ---------------------------------- |
| Service SSH                             | Actif                              |
| Port SSH                                | 22                                 |
| Journal Linux                           | Disponible                         |
| Échec SSH généré                        | Oui                                |
| Événement présent dans le journal Linux | Oui                                |
| Événement reçu par Wazuh                | Oui                                |
| Decoder `sshd`                          | Identifié                          |
| Règle 5503                              | Déclenchée                         |
| Règle 5760                              | Déclenchée                         |
| Niveau des règles                       | 5                                  |
| Visualisation Dashboard                 | Oui                                |
| Classification MITRE                    | Password Guessing sur la règle PAM |
| Scénario contrôlé                       | Oui                                |

---

## 15. Compétences mobilisées

Cette étape a permis de mettre en pratique plusieurs compétences :

* Administration Linux ;
* Administration du service SSH ;
* Analyse des journaux système ;
* Utilisation de `systemctl` ;
* Utilisation de `grep` et `tail` ;
* Compréhension de PAM ;
* Configuration et supervision Wazuh ;
* Analyse des événements de sécurité ;
* Compréhension des règles Wazuh ;
* Lecture des decoders ;
* Utilisation de MITRE ATT&CK ;
* Analyse d'alertes SIEM ;
* Documentation technique ;
* Gestion des preuves et captures dans GitHub.

---

## 16. Conclusion

La journée J6 a permis de mettre en œuvre et de vérifier la surveillance des événements de sécurité Linux avec Wazuh.

Un scénario contrôlé d'échec d'authentification SSH a été généré sur le serveur `wazuh-siem`. L'événement a été enregistré par le système Linux, collecté par Wazuh, décodé comme un événement SSH et associé aux règles `5503` et `5760`, toutes deux de niveau 5.

Cette expérimentation confirme le fonctionnement de la chaîne de collecte et de détection sur le serveur Linux et constitue une base pour les scénarios de sécurité plus avancés qui seront réalisés dans les étapes suivantes du projet.
