# J3 — Installation et configuration de l’agent Wazuh Windows

## 1. Objectif

L’objectif de cette étape est d’installer et de configurer un agent Wazuh sur un poste Windows afin de permettre la collecte et la transmission des événements de sécurité vers le serveur Wazuh.

Cette étape permet également de vérifier l’enregistrement de l’agent auprès du manager Wazuh, son état de connexion et sa visibilité depuis le tableau de bord.

## 2. Architecture

L’architecture mise en œuvre est composée de deux éléments principaux :

* **Serveur Wazuh** : `wazuh-siem`
* **Poste Windows supervisé** : `Windows-Host`

La communication entre les deux machines est réalisée sur le réseau Host-Only.

| Élément                     | Information     |
| --------------------------- | --------------- |
| Serveur Wazuh               | `wazuh-siem`    |
| Adresse IP du serveur       | `192.168.56.10` |
| Agent                       | `Windows-Host`  |
| Adresse IP du poste Windows | `192.168.56.1`  |
| ID de l’agent               | `001`           |
| Version de l’agent          | `4.14.8`        |
| Système supervisé           | Windows 10 Pro  |
| Groupe Wazuh                | `default`       |

## 3. Installation de l’agent Windows

L’agent Wazuh a été installé sur le poste Windows afin de permettre la collecte des événements système et de sécurité.

Le service Windows correspondant à l’agent est :

```text
WazuhSvc
```

Après l’installation, le service a été vérifié afin de confirmer qu’il était actif.

## 4. Vérification du service Wazuh

La commande suivante permet de vérifier l’état du service :

```powershell
Get-Service WazuhSvc
```

Le résultat indique que le service est en état `Running` et configuré pour démarrer automatiquement.

Cette vérification confirme que l’agent Wazuh fonctionne correctement sur le poste Windows.

## 5. Enregistrement de l’agent auprès du serveur

L’agent Windows a été enregistré auprès du manager Wazuh.

La commande suivante permet de vérifier les agents connus par le serveur :

```bash
sudo /var/ossec/bin/agent_control -l
```

L’agent `Windows-Host` apparaît avec l’identifiant `001`.

## 6. Vérification de la connexion

Après l’enregistrement, l’état de l’agent a été vérifié depuis le serveur Wazuh.

L’agent `Windows-Host` apparaît comme actif, ce qui confirme que la communication entre le poste Windows et le serveur Wazuh est opérationnelle.

## 7. Vérification depuis le Dashboard

L’agent Windows a également été vérifié depuis le tableau de bord Wazuh.

Le Dashboard permet notamment de visualiser l’agent, son état et les informations associées au poste supervisé.

## 8. Résultat

À l’issue de cette étape :

* l’agent Wazuh Windows est installé ;
* le service `WazuhSvc` est actif ;
* l’agent `Windows-Host` est enregistré auprès du manager ;
* l’agent possède l’identifiant `001` ;
* la communication avec le serveur Wazuh est opérationnelle ;
* l’agent est visible depuis le Dashboard Wazuh.

Cette configuration constitue la base nécessaire pour les étapes suivantes de collecte, de supervision et de détection des événements de sécurité Windows.

## 9. Compétences mobilisées

Cette étape a permis de mettre en pratique :

* Installation et configuration d’un agent SIEM ;
* Administration Windows ;
* Administration Linux ;
* Communication agent/manager Wazuh ;
* Supervision d’un poste Windows ;
* Vérification de l’état d’un agent ;
* Utilisation du Dashboard Wazuh ;
* Mise en place d’une architecture de supervision de sécurité.

## 10. Conclusion

L’agent Wazuh Windows a été correctement intégré à la plateforme SIEM. Le poste `Windows-Host` est désormais supervisé par le serveur `wazuh-siem`.

La plateforme est ainsi prête pour les étapes suivantes consacrées à la collecte et à l’analyse des événements de sécurité.
