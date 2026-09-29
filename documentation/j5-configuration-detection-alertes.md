# J5 — Configuration de la détection et analyse des alertes

## 1. Objectif

L'objectif de cette journée est de vérifier les capacités de détection de la plateforme SIEM Wazuh à partir d'événements générés sur le poste Windows supervisé.

Les tests réalisés permettent de vérifier plusieurs types de détection :

* échec d'authentification Windows ;
* activité PowerShell ;
* détection de vulnérabilités sur le système Windows.

Les alertes générées sont collectées par l'agent Wazuh installé sur le poste Windows puis analysées par le serveur Wazuh.

---

## 2. Environnement de test

| Élément           | Valeur          |
| ----------------- | --------------- |
| SIEM              | Wazuh           |
| Version Wazuh     | 4.14.8          |
| Serveur SIEM      | `wazuh-siem`    |
| Agent Windows     | `Windows-Host`  |
| ID agent          | `001`           |
| Adresse IP agent  | `192.168.56.1`  |
| Serveur Wazuh     | `192.168.56.10` |
| Système supervisé | Windows 10 Pro  |
| Branche GitHub    | `main`          |

L'agent Windows est enregistré auprès du serveur Wazuh et son état est `active`.

---

## 3. Principe de détection

La chaîne de supervision utilisée pendant les tests est la suivante :

```text
Activité sur Windows
        |
        v
Journalisation Windows
        |
        v
Agent Wazuh Windows
        |
        v
Serveur Wazuh
        |
        v
Décodage de l'événement
        |
        v
Règle de détection
        |
        v
Alerte Wazuh
        |
        v
Wazuh Dashboard
```

Cette architecture permet de centraliser les événements provenant du poste Windows et de les analyser à l'aide des règles de détection de Wazuh.

---

## 4. Scénario 1 — Échec d'authentification Windows

### 4.1 Description

Le premier scénario consiste à vérifier la détection d'une tentative d'authentification Windows échouée.

L'événement Windows observé correspond à l'Event ID `4625`.

Dans le Dashboard Wazuh, l'événement a été associé à la règle `60122`.

### 4.2 Résultat observé

| Champ             | Valeur                                         |
| ----------------- | ---------------------------------------------- |
| Agent             | `Windows-Host`                                 |
| Event ID          | `4625`                                         |
| Rule ID           | `60122`                                        |
| Niveau            | `5`                                            |
| Description       | `Logon Failure - Unknown user or bad password` |
| Utilisateur testé | `FAKE_USER`                                    |
| Résultat          | `AUDIT_FAILURE`                                |

L'événement indique une tentative d'ouverture de session avec un utilisateur inexistant ou un mot de passe incorrect.

Cette détection montre que les événements de sécurité Windows relatifs aux authentifications échouées sont correctement collectés et analysés par Wazuh.

###

---

## 5. Scénario 2 — Détection d'une activité PowerShell

### 5.1 Description

Le deuxième scénario concerne la journalisation des scripts PowerShell.

Le mécanisme Script Block Logging de Windows permet de journaliser les blocs de scripts PowerShell exécutés. L'agent Wazuh collecte le journal :

```text
Microsoft-Windows-PowerShell/Operational
```

Un événement PowerShell `4104` a été observé dans le Dashboard Wazuh.

### 5.2 Résultat observé

| Champ           | Valeur                                                    |
| --------------- | --------------------------------------------------------- |
| Agent           | `Windows-Host`                                            |
| Event ID        | `4104`                                                    |
| Rule ID         | `91816`                                                   |
| Niveau          | `4`                                                       |
| Description     | `Powershell script querying system environment variables` |
| Canal           | `Microsoft-Windows-PowerShell/Operational`                |
| Decoder         | `windows_eventchannel`                                    |
| MITRE ID        | `T1082`                                                   |
| MITRE Tactic    | `Discovery`                                               |
| MITRE Technique | `System Information Discovery`                            |

Le champ `scriptBlockText` permet également de visualiser le contenu du script analysé.

L'événement observé contenait notamment une commande utilisant `secedit`, une lecture du fichier de configuration généré et une recherche sur `ResetLockoutCount`.

Cette observation confirme que Wazuh reçoit et analyse les événements Script Block Logging provenant de Windows.

###

---

## 6. Scénario 3 — Détection d'une vulnérabilité Windows

### 6.1 Description

Le troisième scénario concerne la capacité de Wazuh à identifier des vulnérabilités présentes sur le système supervisé.

Dans les événements du poste `Windows-Host`, une alerte associée à la règle `23505` a été observée.

L'alerte affichait :

```text
CVE-2026-68876 affects Microsoft Windows 10 Pro
```

### 6.2 Résultat observé

Les informations affichées dans le Dashboard sont les suivantes :

| Champ                  | Valeur observée  |
| ---------------------- | ---------------- |
| Agent                  | `Windows-Host`   |
| Rule ID                | `23505`          |
| Niveau                 | `10`             |
| CVE                    | `CVE-2026-68876` |
| Assignateur            | `microsoft`      |
| Classification         | `CVSS`           |
| CVSS 3 Base Score      | `8`              |
| Attack Vector          | `NETWORK`        |
| Confidentiality Impact | `HIGH`           |
| Integrity Impact       | `HIGH`           |
| Availability Impact    | `HIGH`           |
| Privileges Required    | `LOW`            |

Ces informations correspondent aux données présentées par l'instance Wazuh lors de l'analyse de l'alerte.

Il s'agit ici d'une détection de vulnérabilité et non d'un événement comportemental comme les scénarios d'authentification ou de PowerShell.

###

---

## 7. Synthèse des détections

Les trois scénarios réalisés pendant J5 permettent d'observer différentes capacités de la plateforme Wazuh.

| Scénario | Événement / détection            |   Règle | Niveau | Type                                  |
| -------- | -------------------------------- | ------: | -----: | ------------------------------------- |
| 1        | Échec d'authentification Windows | `60122` |    `5` | Événement de sécurité                 |
| 2        | Activité PowerShell              | `91816` |    `4` | Journalisation / détection PowerShell |
| 3        | Vulnérabilité Windows            | `23505` |   `10` | Détection de vulnérabilité            |

Ces résultats montrent que Wazuh peut centraliser différents types d'informations de sécurité provenant d'un même endpoint.

---

## 8. Analyse
## 8. Analyse détaillée des scénarios

Cette section présente l’analyse des trois scénarios de détection réalisés avec Wazuh. L’objectif est de comprendre la chaîne complète de détection :

**Action sur le poste Windows → événement Windows → collecte par l’agent Wazuh → transmission au serveur Wazuh → décodage → application d’une règle → génération d’une alerte → analyse par l’administrateur.**

### 8.1 Scénario 1 — Détection d’un échec d’authentification Windows

Le premier scénario consiste à provoquer volontairement une tentative de connexion avec un utilisateur inexistant ou un mot de passe incorrect afin de vérifier que Wazuh détecte cet événement de sécurité.

#### a) Authentification

L’authentification est le processus permettant à Windows de vérifier l’identité d’un utilisateur qui tente d’accéder au système.

Dans ce scénario, une tentative de connexion incorrecte a été effectuée avec l’utilisateur `FAKE_USER`.

Windows a alors enregistré l'échec dans son journal de sécurité.

#### b) Event ID 4625

L’**Event ID 4625** correspond à un événement Windows indiquant un échec de connexion.

Un événement Windows possède généralement un identifiant numérique appelé **Event ID**. Cet identifiant permet de déterminer le type d’événement enregistré.

Dans notre cas :

* **Event ID :** `4625`
* **Signification :** échec de connexion
* **Utilisateur :** `FAKE_USER`
* **Résultat :** `AUDIT_FAILURE`

L’événement constitue donc la donnée initiale utilisée par Wazuh pour effectuer la détection.

#### c) AUDIT_FAILURE

`AUDIT_FAILURE` signifie que Windows a enregistré une opération de sécurité qui a échoué.

Le terme **audit** désigne l'enregistrement des activités importantes pour la sécurité du système.

Ici, Windows conserve la trace de la tentative de connexion échouée afin qu'elle puisse être analysée ultérieurement.

#### d) Règle Wazuh 60122

Après réception et analyse de l’événement, Wazuh applique ses règles de détection.

La **Rule ID 60122** a été déclenchée.

Une **Rule ID** est l’identifiant unique d’une règle Wazuh. Une règle décrit une condition permettant d’identifier un événement particulier.

Dans ce scénario, la règle est associée à la description :

`Logon Failure - Unknown user or bad password`

Cela signifie que Wazuh a identifié une tentative de connexion qui correspond à un utilisateur inconnu ou à un mauvais mot de passe.

Le niveau de l’alerte observé est **5**.

Le niveau Wazuh indique la sévérité attribuée par le moteur de détection à l’événement. Il ne doit pas être confondu avec un score CVSS.

#### e) Chaîne de détection

Le fonctionnement observé peut être résumé ainsi :

**Tentative de connexion incorrecte**

→ Windows détecte l’échec

→ Windows génère l’Event ID `4625`

→ l’agent Wazuh installé sur Windows collecte l’événement

→ l’agent transmet l’événement au serveur Wazuh

→ Wazuh décode les informations reçues

→ la règle `60122` correspond à l’événement

→ Wazuh génère une alerte

→ l’alerte apparaît dans le dashboard Wazuh.

Ce scénario montre que Wazuh peut transformer un événement Windows brut en une alerte de sécurité exploitable.

---

### 8.2 Scénario 2 — Détection d’une activité PowerShell

Le deuxième scénario concerne la surveillance de l’utilisation de PowerShell.

PowerShell est un environnement de ligne de commande et de scripting fourni par Microsoft. Il permet notamment d’administrer Windows et d’exécuter des commandes et des scripts.

Comme PowerShell peut être utilisé aussi bien pour l’administration légitime que dans certaines techniques d’attaque, sa surveillance constitue un élément important de la supervision d’un poste Windows.

#### a) Script Block Logging

Pour permettre à Wazuh d’analyser les scripts PowerShell exécutés, le mécanisme **Script Block Logging** de Windows a été activé.

Le **Script Block Logging** est une fonctionnalité Windows qui permet d’enregistrer les blocs de scripts PowerShell exécutés dans le journal :

`Microsoft-Windows-PowerShell/Operational`

Cette journalisation fournit donc davantage d’informations sur les commandes exécutées avec PowerShell.

#### b) Event ID 4104

Le **Event ID 4104** correspond à un événement de **PowerShell Script Block Logging**.

Il contient notamment le contenu du bloc de script enregistré.

Dans l’alerte observée, le champ `scriptBlockText` contient notamment :

`secedit /export /cfg $env:TEMP\secpol.cfg`

puis :

`Get-Content $env:TEMP\secpol.cfg | Select-String ResetLockoutCount`

et enfin :

`Remove-Item $env:TEMP\secpol.cfg`

Ces commandes ont été utilisées dans le cadre du test de détection.

#### c) Explication des commandes

`secedit`

`secedit` est un utilitaire Windows permettant notamment d’exporter ou de configurer des paramètres de sécurité.

Dans le test, il est utilisé avec l’option `/export` afin d’exporter une configuration de sécurité vers un fichier.

`$env:TEMP`

Cette expression PowerShell permet d’accéder à la variable d’environnement `TEMP`.

Elle représente le répertoire temporaire utilisé par Windows pour stocker certains fichiers temporaires.

`Get-Content`

La commande `Get-Content` permet de lire le contenu d’un fichier.

Dans notre scénario, elle permet de lire le fichier de configuration précédemment exporté.

`Select-String`

`Select-String` permet de rechercher une chaîne de caractères dans du texte.

Dans le scénario, la chaîne recherchée est :

`ResetLockoutCount`

Cela permet de rechercher un paramètre particulier dans le fichier de configuration.

`Remove-Item`

La commande `Remove-Item` permet de supprimer un fichier ou un élément.

Elle est utilisée ici pour supprimer le fichier temporaire créé pendant le test.

#### d) Règle Wazuh 91816

Wazuh a identifié l’événement PowerShell et a déclenché la **Rule ID 91816**.

La description observée est :

`Powershell script querying system environment variables`

Cette règle appartient notamment aux groupes :

`windows`

et

`powershell`

Le niveau observé est **4**.

L’alerte permet donc à l’administrateur de savoir qu’une activité PowerShell correspondant à une règle de détection a été observée sur le poste Windows.

#### e) MITRE ATT&CK

L’alerte est également associée à **MITRE ATT&CK**.

MITRE ATT&CK est une base de connaissances utilisée en cybersécurité pour décrire les comportements et techniques pouvant être utilisés par des attaquants.

Une **tactique** représente l’objectif général recherché par l’attaquant.

Une **technique** représente une méthode utilisée pour atteindre cet objectif.

Dans l’alerte observée :

* **Tactique :** Discovery
* **Technique :** T1082
* **Technique :** System Information Discovery

**Discovery** correspond à la phase pendant laquelle un acteur cherche à obtenir des informations sur l’environnement informatique.

**T1082 System Information Discovery** correspond à la découverte d’informations concernant le système.

Il est important de préciser que la présence d’une correspondance MITRE ATT&CK ne signifie pas automatiquement qu’une attaque a eu lieu. Elle indique que l’activité observée correspond à un comportement décrit dans la base MITRE ATT&CK.

#### f) Chaîne de détection

Le scénario peut être résumé ainsi :

**Exécution d’une commande PowerShell**

→ PowerShell exécute le script

→ Windows enregistre l’activité avec le Script Block Logging

→ Windows génère l’Event ID `4104`

→ l’agent Wazuh collecte l’événement

→ l’événement est transmis au serveur Wazuh

→ Wazuh décode les données

→ une règle PowerShell est déclenchée

→ l’alerte est enrichie avec des informations MITRE ATT&CK

→ l’alerte est affichée dans le dashboard.

---

### 8.3 Scénario 3 — Détection d’une vulnérabilité Windows

Le troisième scénario concerne la détection des vulnérabilités présentes sur le système Windows supervisé.

Une **vulnérabilité** est une faiblesse dans un logiciel, un système ou une configuration pouvant être exploitée pour compromettre la sécurité.

Wazuh peut identifier certaines vulnérabilités associées aux logiciels présents sur les endpoints supervisés.

#### a) CVE

Les vulnérabilités sont notamment identifiées avec un identifiant **CVE**.

**CVE** signifie **Common Vulnerabilities and Exposures**.

Un identifiant CVE permet de référencer une vulnérabilité de manière standardisée.

Dans notre test, Wazuh a détecté notamment :

`CVE-2026-68876`

L’alerte indique que cette vulnérabilité affecte :

`Microsoft Windows 10 Pro`

#### b) Rule ID 23505

La détection a déclenché la **Rule ID 23505**.

Cette règle est utilisée par Wazuh pour générer une alerte lorsqu’une vulnérabilité correspondant aux informations collectées est identifiée.

Le niveau Wazuh observé est :

`10`

Cela indique une alerte considérée comme importante par le moteur de règles Wazuh.

#### c) CVSS

L’alerte contient également des informations provenant du système **CVSS**.

**CVSS** signifie **Common Vulnerability Scoring System**.

Il s’agit d’un système standardisé permettant de décrire la gravité technique d’une vulnérabilité.

Dans notre événement, le **CVSS Base Score** observé est :

`8`

Le score de base représente les caractéristiques techniques de la vulnérabilité indépendamment du contexte particulier d’une organisation.

#### d) Attack Vector : NETWORK

L’**Attack Vector** indique comment une vulnérabilité peut être exploitée.

Dans notre événement :

`Attack Vector = NETWORK`

Cela signifie que le vecteur d’attaque associé à la vulnérabilité est lié au réseau.

Il ne faut pas interpréter ce champ comme la preuve qu’une attaque réseau a réellement été effectuée sur notre machine. Il décrit une caractéristique de la vulnérabilité référencée.

#### e) Confidentiality : HIGH

La **confidentialité** concerne la protection des informations contre une consultation non autorisée.

Dans l’événement :

`Confidentiality Impact = HIGH`

Cela indique que, selon l’évaluation CVSS associée à cette vulnérabilité, son exploitation peut avoir un impact important sur la confidentialité.

#### f) Integrity : HIGH

L’**intégrité** concerne la protection des données et du système contre les modifications non autorisées.

Dans l’événement :

`Integrity Impact = HIGH`

Cela indique que la vulnérabilité possède un impact important sur l’intégrité selon les caractéristiques CVSS enregistrées.

#### g) Availability : HIGH

La **disponibilité** signifie que les systèmes et les ressources doivent rester accessibles aux utilisateurs autorisés.

Dans l’événement :

`Availability Impact = HIGH`

Cela indique que l’exploitation de la vulnérabilité peut avoir un impact important sur la disponibilité selon l’évaluation CVSS.

#### h) Privileges Required : LOW

Le champ **Privileges Required** indique le niveau de privilèges nécessaire pour exploiter la vulnérabilité.

Dans notre événement :

`Privileges Required = LOW`

Cela signifie que l’évaluation CVSS considère qu’un niveau faible de privilèges est requis pour l’exploitation.

Là encore, ce champ décrit les caractéristiques de la vulnérabilité. Il ne constitue pas la preuve qu’un attaquant possède actuellement ces privilèges sur notre poste.

#### i) Chaîne de détection

Le fonctionnement peut être résumé ainsi :

**Identification d’une vulnérabilité sur Windows**

→ l’agent Wazuh collecte les informations nécessaires

→ les informations sont transmises au serveur Wazuh

→ Wazuh analyse les données de vulnérabilités

→ la vulnérabilité est associée à un identifiant CVE

→ la règle `23505` génère une alerte

→ les informations CVSS sont associées à l’alerte

→ l’alerte apparaît dans le dashboard Wazuh.

---

### 8.4 Comparaison des trois scénarios

| Élément           | Scénario 1                | Scénario 2             | Scénario 3                     |
| ----------------- | ------------------------- | ---------------------- | ------------------------------ |
| Type              | Authentification          | PowerShell             | Vulnérabilité                  |
| Source            | Windows Security          | PowerShell Operational | Informations de vulnérabilités |
| Event ID          | 4625                      | 4104                   | —                              |
| Rule ID           | 60122                     | 91816                  | 23505                          |
| Niveau Wazuh      | 5                         | 4                      | 10                             |
| Élément principal | Échec de connexion        | Script PowerShell      | CVE                            |
| MITRE ATT&CK      | Non indiqué dans ce test  | T1082                  | —                              |
| CVSS              | —                         | —                      | Score de base 8                |
| Résultat          | Alerte d’authentification | Alerte PowerShell      | Alerte de vulnérabilité        |

Dans les trois scénarios, la chaîne de détection repose sur le même principe : les événements ou informations de sécurité sont collectés au niveau de l’endpoint Windows, transmis au serveur Wazuh, décodés et analysés par le moteur de règles, puis transformés en alertes exploitables dans le dashboard. Cette approche permet de centraliser la supervision, de contextualiser les événements détectés et de faciliter leur analyse par l’administrateur de sécurité.

## 9. Résultats obtenus

À la fin de J5, les résultats suivants ont été vérifiés :

* les alertes Windows sont visibles dans Wazuh Dashboard ;
* les événements d'échec d'authentification `4625` sont détectés ;
* les événements PowerShell `4104` sont collectés ;
* les règles Wazuh sont appliquées aux événements reçus ;
* les informations MITRE ATT&CK sont associées à certaines règles ;
* les informations de vulnérabilité Windows sont visibles dans le Dashboard ;
* les captures des trois scénarios sont archivées dans le dépôt GitHub.

---

## 10. Captures réalisées

Les preuves visuelles de J5 sont enregistrées dans le répertoire :

```text
screenshots/j5/
```

avec les fichiers :

```text
j5-01-detection-echec-authentification-4625.png
j5-02-detection-powershell-4104.png
j5-03-detection-vulnerabilite-windows-cve.png
```

---

## 11. Compétences mises en œuvre

Cette journée a permis de mettre en pratique :

* la supervision d'un endpoint Windows avec Wazuh ;
* l'analyse des événements Windows ;
* l'identification des Event IDs de sécurité ;
* l'analyse des règles Wazuh ;
* l'analyse des niveaux d'alerte ;
* la collecte des événements PowerShell ;
* l'interprétation des informations MITRE ATT&CK ;
* l'analyse des informations de vulnérabilité ;
* l'utilisation du Dashboard Wazuh pour l'investigation ;
* la documentation des résultats dans un dépôt Git.

---

## 12. Conclusion

La journée J5 a permis de vérifier concrètement le fonctionnement du mécanisme de détection de Wazuh sur un endpoint Windows.

Trois catégories d'événements ont été étudiées : les échecs d'authentification Windows, les activités PowerShell et les vulnérabilités détectées sur le système.

Les résultats obtenus constituent la base des travaux suivants consacrés à l'analyse des alertes, aux scénarios d'incidents et à la supervision de la plateforme SIEM.
