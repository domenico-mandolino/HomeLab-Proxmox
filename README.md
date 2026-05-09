# Infrastructure Proxmox Sécurisée

## 1. Présentation du projet

### 1.1 Contexte
Ce projet consiste à concevoir et déployer une infrastructure virtualisée sécurisée basée sur Proxmox VE afin de répondre aux besoins d’administration distante, de segmentation réseau et de supervision d’une PME multi-sites.

L’objectif principal est de construire une architecture fiable, évolutive et sécurisée permettant :

- l’administration distante via VPN ;
- la centralisation des accès d’administration ;
- l’isolation des services internes ;
- la supervision des systèmes ;
- la préparation à l’automatisation via Ansible.

## 2. Objectifs du projet

### 2.1 Objectifs principaux

Les objectifs principaux sont les suivants :

- Déployer un hyperviseur Proxmox VE.
- Sécuriser l’accès distant via VPN WireGuard.
- Mettre en place un bastion d’administration.
- Segmenter les réseaux internes.
- Préparer une infrastructure évolutive.
- Centraliser l’administration système.
- Limiter l’exposition Internet.
- Implémenter une supervision centralisée.

### 2.2 Contraintes du projet

Contraintes techniques:
 
- Infrastructure basée sur un serveur unique.
- Utilisation de matériel recyclé.
- Limitation des ressources matérielles.
- Nécessité de maintenir un haut niveau de sécurité.
- Contraintes organisationnelles
- Administration multi-sites.
- Accès distants sécurisés.
- Simplicité d’utilisation.
- Limitation des coûts.

## 3. Infrastructure matérielle

### 3.1 Serveur principal

| Élément           | Configuration        |
|-------------------|----------------------|
| Modèle            | Lenovo Mini PC       |
| CPU               | Intel i5-10600       |
| Cœurs / Threads   | 6 cœurs / 12 threads |
| RAM               | 16 Go DDR4           |
| Stockage 1        | NVMe 256 Go          |
| Stockage 2        | NVMe 500 Go          |
| Hyperviseur       | Proxmox VE           |


### 3.2 Dimensionnement des machines virtuelles

| Rôle VM       | RAM recommandée |
|---------------|-----------------|
| Bastion SSH   | 512 Mo – 1 Go   |
| WireGuard     | 512 Mo          |
| DNS/DHCP      | 512 Mo          |
| Ansible       | 1 Go            |
| Prometheus    | 2–4 Go          |
| Grafana       | 1 Go            |
| Nextcloud     | 2–4 Go          |

### important :
- éviter le surprovisionnement permanent ;
- garder de la RAM libre pour le cache disque Proxmox/ZFS ;
- éviter le swap.


## . Architecture logique

### 4.1 Vue d’ensemble
L’architecture repose sur une séparation stricte entre :
- le réseau de management ;
- le réseau VPN ;
- les services internes ;
- les postes utilisateurs.
L’ensemble des accès d’administration doit transiter exclusivement via le VPN puis via le bastion.

### 4.2 Architecture réseau cible

| Réseau       | Fonction                |  Adresse        |
|--------------|-------------------------|-----------------|
| vmbr0        | Réseau LAN / management | `<BASTION_IP>`  |
| vmbr100      | Réseau interne isolé    | `172.16.0.0/24` |
| WireGuard    | Réseau VPN              | `10.10.0.0/24`  |


## 5. Machines virtuelles prévues
 
| VM         | Fonction                 |
|------------|--------------------------|
| Debian VPN | Serveur WireGuard        |
| Bastion    | Administration sécurisée |
| DNS/DHCP 1 | Services réseau          |
| DNS/DHCP 2 | Redondance               |
| Monitoring | Prometheus + Grafana     |
| Ansible    | Automatisation           |
| Nextcloud  | Signature PDF / IT Drive |


## 6. Sécurité

### 6.1 Principes de sécurité

Le projet repose sur les principes suivants :
- principe du moindre privilège ;
- segmentation réseau ;
- absence d’exposition directe des services ;
- accès centralisé ;
- authentification par clés ;
- journalisation des accès.

### 6.2 Politique d’exposition Internet

Services autorisés

| Service           | Exposition Internet |
|-------------------|---------------------|
| WireGuard         | Oui                 |
| SSH               | Non                 |
| Proxmox WebUI     | Non                 |
| Bastion           | Non                 |
| Services internes | Non                 |

## Roadmap prévisionnelle

### Phase 1 — Fondations
- Installation Proxmox
- Configuration réseau
- Déploiement VPN

### Phase 2 — Sécurisation

- Bastion d’administration
- Firewall
- Segmentation réseau

### Phase 3 — Services internes
- DNS / DHCP
- Supervision
- Ansible

### Phase 4 — Services complémentaires
- Nextcloud
- Outils internes

## Perspectives d’évolution

Le projet est conçu pour évoluer vers :
- une infrastructure multi-sites ;
- une haute disponibilité partielle ;
- une automatisation avancée ;
- une supervision complète ;
- une centralisation des logs.
