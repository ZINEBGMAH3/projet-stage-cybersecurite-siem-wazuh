# J2 — Installation et configuration du serveur SIEM

## 1. Objectif

Cette étape consiste à mettre en place le serveur central de supervision de sécurité destiné à héberger la plateforme SIEM basée sur Wazuh.

Les objectifs sont les suivants :

* installer et configurer Ubuntu Server ;
* préparer les ressources système nécessaires ;
* installer Wazuh ;
* vérifier les composants du serveur SIEM ;
* configurer l'accès au Dashboard ;
* vérifier la communication avec l'API Wazuh ;
* mettre en place un accès d'administration sécurisé via SSH.

---

## 2. Environnement de travail

Le serveur SIEM est déployé dans une machine virtuelle créée avec VirtualBox.

| Élément        | Configuration             |
| -------------- | ------------------------- |
| Système        | Ubuntu Server 24.04.5 LTS |
| Architecture   | x86-64                    |
| Processeur     | 2 vCPU                    |
| Mémoire        | 4 Go RAM                  |
| Stockage       | 40 Go                     |
| Virtualisation | VirtualBox                |
| Nom d'hôte     | wazuh-siem                |
| Adresse IP     | 10.0.2.15                 |
| Version Wazuh  | 4.14.8                    |

Le stockage a été configuré avec LVM afin de disposer d'un espace suffisant pour le fonctionnement de la plateforme et l'indexation des événements.

---

## 3. Installation de Wazuh

L'installation de Wazuh a été réalisée sur le serveur Ubuntu selon une architecture centralisée.

La plateforme comprend les principaux composants suivants :

* Wazuh Manager ;
* Wazuh Indexer ;
* Wazuh Dashboard ;
* Wazuh API.

Le serveur Wazuh constitue le point central de supervision. Il sera utilisé ultérieurement pour recevoir et analyser les événements provenant des agents.

---

## 4. Vérification des services

L'état des principaux services a été vérifié avec la commande :

```bash
sudo systemctl status wazuh-indexer wazuh-manager wazuh-dashboard --no-pager
```

Les trois services doivent apparaître avec l'état :

```text
Active: active (running)
```

Cette vérification permet de confirmer que les principaux composants de la plateforme SIEM sont opérationnels.

---

## 5. Vérification de la version de Wazuh

La version installée a été vérifiée avec :

```bash
sudo /var/ossec/bin/wazuh-control info
```

La plateforme utilisée dans ce projet est :

```text
Wazuh 4.14.8
```

---

## 6. Accès au Dashboard

Le Dashboard Wazuh est accessible depuis le navigateur à l'adresse :

```text
https://127.0.0.1:8443
```

Le port `8443` correspond à une redirection VirtualBox vers le port HTTPS `443` du serveur Ubuntu.

L'accès au Dashboard permet notamment de consulter les informations de supervision, les événements de sécurité et les alertes générées par la plateforme.

Une vérification de la connexion entre le Dashboard et l'API Wazuh a également été réalisée.

---

## 7. Configuration de l'API Wazuh

La communication entre le Dashboard et l'API Wazuh a été vérifiée.

L'API Wazuh écoute sur le port :

```text
55000
```

Une authentification fonctionnelle entre le Dashboard et l'API a été validée.

Cette étape est nécessaire pour permettre au Dashboard d'interagir avec les composants de la plateforme et d'afficher les informations de supervision.

---

## 8. Configuration de l'accès SSH

Un accès SSH a été configuré afin de permettre l'administration du serveur depuis Windows.

La connexion utilise la redirection de port VirtualBox suivante :

| Élément              | Valeur    |
| -------------------- | --------- |
| Adresse côté hôte    | 127.0.0.1 |
| Port côté hôte       | 2223      |
| Adresse côté serveur | 10.0.2.15 |
| Port SSH             | 22        |
| Protocole            | TCP       |

Depuis Windows, la connexion est réalisée avec :

```powershell
ssh zineb@127.0.0.1 -p 2223
```

La connexion a été testée avec succès.

La commande :

```bash
hostname
```

retourne :

```text
wazuh-siem
```

Cette vérification confirme que l'administration distante du serveur fonctionne correctement.

---

## 9. Captures d'écran

Les principales preuves de configuration sont conservées dans le dossier :

```text
screenshots/j2/
```

Les captures prévues sont :

| Fichier                  | Description                          |
| ------------------------ | ------------------------------------ |
| `j2-services-wazuh.png`  | État des services Wazuh              |
| `j2-ip-serveur.png`      | Configuration réseau du serveur      |
| `j2-version-wazuh.png`   | Version de Wazuh installée           |
| `j2-dashboard-wazuh.png` | Dashboard Wazuh et connexion à l'API |
| `j2-ssh.png`             | Connexion SSH depuis Windows         |

---

## 10. Résultat

À l'issue de cette étape, le serveur SIEM est opérationnel.

Les composants Wazuh Manager, Wazuh Indexer et Wazuh Dashboard sont installés et fonctionnels. L'accès au Dashboard ainsi que la communication avec l'API ont été validés.

L'accès SSH depuis Windows a également été vérifié.

Le serveur est donc prêt pour l'étape suivante du projet, consacrée au déploiement et à la configuration des agents de supervision.

---

## 11. Conclusion

Cette étape a permis de mettre en place l'infrastructure centrale du projet SIEM. Le serveur Ubuntu constitue désormais le point central de collecte, d'analyse et de visualisation qui sera utilisé pour superviser les systèmes intégrés à la plateforme.

La prochaine étape consistera à déployer les premiers agents et à établir la collecte des événements de sécurité.
