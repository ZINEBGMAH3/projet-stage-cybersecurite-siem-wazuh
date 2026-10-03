# J7 — Surveillance et détection d’un événement de connexion Windows

## 1. Objectif

L’objectif de cette étape est de vérifier la capacité de Wazuh à collecter et analyser un événement de sécurité généré sur le système Windows surveillé.

L’événement étudié correspond à une authentification Windows réussie associée à l’Event ID 4624.

## 2. Environnement

* SIEM : Wazuh 4.14.8
* Agent : Windows-Host
* Agent ID : 001
* Système surveillé : Windows 10 Pro
* Serveur Wazuh : wazuh-siem
* Type d’événement : authentification réussie
* Event ID : 4624

## 3. Détection de l’événement

L’événement Windows a été collecté par l’agent Wazuh et analysé par le moteur de règles.

Les informations principales sont :

| Élément     | Valeur                           |
| ----------- | -------------------------------- |
| Agent       | Windows-Host                     |
| Agent ID    | 001                              |
| Event ID    | 4624                             |
| Rule ID     | 60106                            |
| Description | Windows Logon Success            |
| Niveau      | 3                                |
| Logon Type  | 5                                |
| Target User | Système                          |
| Processus   | C:\Windows\System32\services.exe |

La règle Wazuh `60106` correspond à la détection d'une connexion Windows réussie.

## 4. Interprétation

L’Event ID 4624 indique qu’une authentification Windows a réussi.

Dans cet événement, le `Logon Type 5` correspond à une ouverture de session effectuée par un service Windows.

Le compte indiqué est `Système` et le processus associé est :

`C:\Windows\System32\services.exe`

L’événement observé est donc cohérent avec l’activité normale d’un service Windows. Il ne doit pas être présenté comme une attaque sur la seule base de cet événement.

La règle `60106` contient également un mapping MITRE ATT&CK vers `T1078 - Valid Accounts`. Ce mapping décrit la technique associée à la règle ; il ne signifie pas que l’événement observé constitue automatiquement une attaque.

## 5. Vérification dans Wazuh

L’événement a été vérifié dans :

`Threat Hunting → Events`

Les informations affichées permettent de confirmer :

* la présence de l’agent `Windows-Host` ;
* l’Event ID `4624` ;
* la règle Wazuh `60106` ;
* la description `Windows Logon Success` ;
* le niveau de règle `3` ;
* le type de connexion `5`.

## 6. Captures réalisées

Deux captures ont été réalisées afin de documenter la détection.

### Capture 1 — Vue synthétique

Fichier :

`screenshots/j7/j7-01-detection-windows-logon-4624.png`

Cette capture présente les principales informations de l’événement :

* timestamp ;
* agent.name ;
* rule.description ;
* rule.level ;
* rule.id.

### Capture 2 — Détails de l’événement

Fichier :

`screenshots/j7/j7-02-details-windows-logon-4624.png`

Cette capture présente les détails permettant de caractériser précisément l’événement :

* agent.name = Windows-Host ;
* agent.id = 001 ;
* eventID = 4624 ;
* rule.id = 60106 ;
* rule.description = Windows Logon Success ;
* rule.level = 3 ;
* logonType = 5 ;
* targetUserName = Système ;
* processName = C:\Windows\System32\services.exe.

## 7. Résultat

La collecte et l’analyse de l’événement Windows ont été correctement vérifiées dans Wazuh.

L’agent `Windows-Host` transmet l’événement `4624` au serveur Wazuh, qui l’identifie avec la règle `60106` et le niveau 3.

Cette étape confirme le fonctionnement de la surveillance des événements d’authentification Windows et constitue une base pour l’analyse d’événements de sécurité plus significatifs.
