Projet de stage — Plateforme SIEM de supervision et détection des incidents
Conception et mise en œuvre d'une plateforme de supervision et de détection des incidents de cybersécurité basée sur Wazuh








Présentation

Ce projet est réalisé dans le cadre d'un stage en cybersécurité et porte sur la conception et la mise en œuvre d'une plateforme centralisée de Security Information and Event Management (SIEM) basée sur Wazuh.

L'objectif est de construire une plateforme capable de centraliser les événements de sécurité provenant de différents systèmes, de les analyser, d'identifier des comportements suspects et de faciliter la supervision ainsi que l'investigation des incidents.

Le projet suit une démarche progressive allant de la préparation de l'infrastructure jusqu'aux tests de détection, à l'analyse des alertes et au durcissement de la plateforme.

Objectifs du projet

Les principaux objectifs sont :

Mettre en place une infrastructure SIEM fonctionnelle.
Déployer et configurer un serveur Wazuh.
Centraliser les événements de sécurité provenant de plusieurs systèmes.
Déployer et configurer des agents de supervision.
Surveiller les activités système et les événements de sécurité.
Configurer des mécanismes de détection adaptés à différents scénarios.
Analyser les alertes générées par la plateforme.
Construire des tableaux de bord de supervision.
Réaliser des scénarios contrôlés d'incidents de sécurité.
Évaluer et renforcer la sécurité de la plateforme SIEM.
Documenter l'ensemble du processus de mise en œuvre.
Architecture

L'architecture repose sur un serveur central Wazuh chargé de recevoir, analyser et indexer les événements de sécurité provenant des systèmes supervisés.

                         ┌──────────────────────────┐
                         │       Administrateur     │
                         │    Dashboard Wazuh       │
                         └────────────┬─────────────┘
                                      │ HTTPS
                                      │
                         ┌────────────▼─────────────┐
                         │      Wazuh Dashboard     │
                         │   Visualisation / SIEM   │
                         └────────────┬─────────────┘
                                      │
                         ┌────────────▼─────────────┐
                         │      Wazuh Manager       │
                         │  Analyse / Détection     │
                         │  Règles / Alertes        │
                         └────────────┬─────────────┘
                                      │
                         ┌────────────▼─────────────┐
                         │      Wazuh Indexer       │
                         │ Stockage / Indexation    │
                         └────────────┬─────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
           ┌────────▼────────┐                ┌────────▼────────┐
           │   Agent Linux   │                │  Agent Windows  │
           │ Logs / Sécurité │                │ Logs / Sécurité │
           └─────────────────┘                └─────────────────┘
Composants principaux
Composant	Rôle
Wazuh Manager	Analyse des événements, gestion des agents et détection
Wazuh Indexer	Indexation et stockage des données de sécurité
Wazuh Dashboard	Supervision, visualisation et analyse
Wazuh Agent	Collecte des événements sur les systèmes supervisés
Ubuntu Server	Système hôte du serveur SIEM
VirtualBox	Infrastructure de virtualisation
Environnement technique
Serveur SIEM
OS : Ubuntu Server 24.04.5 LTS
Architecture : x86-64
CPU : 2 vCPU
RAM : 4 Go
Stockage : 40 Go
Virtualisation : Oracle VirtualBox
Adresse IP du serveur : 10.0.2.15
Wazuh : 4.14.8
Services Wazuh
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
Wazuh API
Administration
SSH
Systemd
Linux CLI
VirtualBox NAT

Compétences mises en œuvre
Cybersécurité
SIEM
Security Monitoring
Détection d'incidents
Analyse d'événements
Analyse d'alertes
Investigation de sécurité
Security Operations
Gestion des logs
Détection basée sur les règles
Systèmes
Linux
Ubuntu Server
Windows
Administration système
SSH
Systemd
Gestion des services
Outils
Wazuh
Wazuh Dashboard
Wazuh Manager
Wazuh Indexer
VirtualBox
Git
GitHub
Méthodologie
Conception d'architecture
Déploiement d'infrastructure
Tests fonctionnels
Analyse des résultats
Documentation technique
Gestion de versions avec Git
Organisation du dépôt
projet-stage-cybersecurite-siem-wazuh/
│
├── README.md
│
├── documentation/
│   ├── architecture/
│   ├── installation/
│   ├── detection/
│   ├── analyse/
│   └── rapport/
│
├── configurations/
│   ├── wazuh/
│   └── agents/
│
├── scripts/
│
└── screenshots/
    ├── j1/
    ├── j2/
    ├── j3/
    └── ...
Méthode de travail

Le projet est réalisé selon une progression structurée :

Conception
    ↓
Préparation de l'infrastructure
    ↓
Déploiement du SIEM
    ↓
Déploiement des agents
    ↓
Collecte des événements
    ↓
Détection
    ↓
Analyse des alertes
    ↓
Scénarios d'incidents
    ↓
Visualisation
    ↓
Durcissement
    ↓
Tests et validation

Chaque étape est documentée avec :

les objectifs ;
les configurations réalisées ;
les commandes utilisées ;
les résultats obtenus ;
les captures d'écran ;
les éventuelles difficultés rencontrées ;
les mesures correctives appliquées.
Résultats attendus

À l'issue du projet, la plateforme devra permettre de :

centraliser les événements de sécurité ;
superviser plusieurs systèmes ;
détecter différents comportements suspects ;
générer et analyser des alertes ;
visualiser les événements à travers des tableaux de bord ;
réaliser une première analyse d'incidents ;
disposer d'une infrastructure SIEM documentée et reproductible.
Documentation

La documentation technique et les résultats des différentes étapes sont disponibles dans le dossier :

documentation/

Les preuves de configuration et les résultats des tests sont disponibles dans :

screenshots/

Les configurations et éléments techniques utilisés dans le projet sont organisés dans :

configurations/


Auteur

Zineb Gmah

Master 2 — Ingénierie Informatique et Sécurité des Systèmes (2I2S)

Université Mohammed V — Faculté des Sciences de Rabat

Domaine : Cybersécurité / Sécurité des systèmes d'information

GitHub : ZINEBGMAH3

Note

Ce dépôt présente les travaux techniques, expérimentations, configurations et résultats réalisés dans le cadre du projet de stage. Les scénarios de sécurité sont réalisés dans un environnement de laboratoire contrôlé à des fins d'apprentissage, de validation et de recherche en cybersécurité.