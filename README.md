# Infrastructure Proxmox Sécurisée

# 1. Présentation du projet

## 1.1 Contexte

Ce projet consiste à concevoir et déployer une infrastructure virtualisée sécurisée basée sur Proxmox VE afin de répondre aux besoins d’administration distante, de supervision et de gestion centralisée d’une flotte de machines réparties sur plusieurs sites.

L’objectif principal n’est pas uniquement l’hébergement de machines virtuelles, mais la création d’une plateforme centralisée permettant :

* l’administration sécurisée de postes distants ;
* la supervision d’une flotte de machines via VPN ;
* la segmentation réseau ;
* la centralisation des accès d’administration ;
* l’automatisation des tâches système ;
* l’hébergement de services internes complémentaires.

L’infrastructure repose sur une approche orientée sécurité et administration centralisée inspirée des pratiques DevOps et Zero Trust.

---

# 2. Objectifs du projet

## 2.1 Objectifs principaux

Les objectifs principaux sont les suivants :

* Déployer un hyperviseur Proxmox VE.
* Sécuriser les accès distants via VPN WireGuard.
* Créer une plateforme d’administration centralisée.
* Superviser une flotte de machines distantes.
* Mettre en place une segmentation réseau stricte.
* Préparer l’automatisation via Ansible.
* Réduire la surface d’exposition Internet.
* Héberger des services internes sécurisés.
* Centraliser les accès d’administration.
* Préparer une architecture multi-sites.

---

## 2.2 Contraintes du projet

### Contraintes techniques

* Infrastructure initialement basée sur un serveur unique.
* Utilisation de matériel recyclé.
* Ressources matérielles limitées.
* Nécessité d’un haut niveau de sécurité.
* Administration distante multi-sites.
* Services accessibles uniquement via VPN.

### Contraintes organisationnelles

* Simplicité d’administration.
* Limitation des coûts.
* Administration centralisée.
* Réduction des interventions physiques.
* Évolutivité progressive de l’infrastructure.

---

# 3. Infrastructure matérielle

## 3.1 Serveur principal

| Élément         | Configuration        |
| --------------- | -------------------- |
| Modèle          | Lenovo Mini PC       |
| CPU             | Intel i5-10600       |
| Cœurs / Threads | 6 cœurs / 12 threads |
| RAM             | 16 Go DDR4           |
| Stockage 1      | NVMe 256 Go          |
| Stockage 2      | NVMe 500 Go          |
| Hyperviseur     | Proxmox VE           |

---

## 3.2 Dimensionnement des machines virtuelles

| VM            | vCPU | RAM recommandée | Stockage | Rôle                         |
| ------------- | ---- | --------------- | -------- | ---------------------------- |
| WireGuard VPN | 1    | 512 Mo          | 8 Go     | Réseau VPN                   |
| Bastion       | 2    | 1 Go            | 16 Go    | Point d’accès administrateur |
| Ansible       | 2    | 1 Go            | 16 Go    | Administration distante      |
| Monitoring    | 2    | 2–4 Go          | 32 Go    | Prometheus + Grafana         |
| DNS/DHCP 1    | 1    | 512 Mo          | 8 Go     | Services réseau              |
| DNS/DHCP 2    | 1    | 512 Mo          | 8 Go     | Redondance réseau            |
| Nextcloud     | 2    | 2–4 Go          | 32–64 Go | Signature PDF / stockage IT  |

### Recommandations

* éviter le surprovisionnement permanent ;
* conserver de la RAM libre pour Proxmox ;
* limiter l’utilisation du swap ;
* séparer les rôles critiques ;
* privilégier des VM spécialisées.

---

# 4. Architecture logique

## 4.1 Vue d’ensemble

L’architecture repose sur une séparation stricte entre :

* le réseau VPN ;
* les accès administrateurs ;
* les services internes ;
* les services de supervision ;
* les postes distants administrés.

L’ensemble des accès administratifs doit obligatoirement transiter via le VPN.

Le bastion devient le point d’entrée humain unique de l’infrastructure.

Ansible et la supervision sont isolés dans une zone d’administration interne distincte.

---

## 4.2 Architecture réseau cible

| Réseau    | Fonction                | Adresse          |
| --------- | ----------------------- | ---------------- |
| vmbr0     | Réseau management / LAN | `192.168.X.X/24` |
| vmbr100   | Réseau interne isolé    | `172.16.0.0/24`  |
| WireGuard | Réseau VPN              | `10.10.0.0/24`   |

---

## 4.3 Flux d’administration

```text
Administrateur
        ↓
VPN WireGuard
        ↓
Bastion
        ↓
VM Ansible
        ↓
Machines distantes
```

---

## 4.4 Flux de supervision

```text
Machines distantes
        ↓
VPN WireGuard
        ↓
Prometheus
        ↓
Grafana
```

---

# 5. Machines virtuelles prévues

| VM         | Fonction                      | Niveau d’exposition     |
| ---------- | ----------------------------- | ----------------------- |
| Debian VPN | Serveur WireGuard             | Internet UDP uniquement |
| Bastion    | Point d’entrée administrateur | VPN uniquement          |
| Ansible    | Administration centralisée    | Interne uniquement      |
| Monitoring | Supervision centralisée       | Interne uniquement      |
| DNS/DHCP 1 | Services réseau               | Interne                 |
| DNS/DHCP 2 | Redondance réseau             | Interne                 |
| Nextcloud  | Signature PDF / stockage IT   | VPN uniquement          |

---

# 6. Sécurité

## 6.1 Principes de sécurité

Le projet repose sur les principes suivants :

* principe du moindre privilège ;
* segmentation réseau ;
* réduction de la surface d’attaque ;
* absence d’exposition directe des services internes ;
* authentification par clés SSH ;
* administration centralisée ;
* journalisation des accès ;
* séparation des rôles critiques ;
* accès VPN obligatoire ;
* approche Zero Trust simplifiée.

---

## 6.2 Politique d’exposition Internet

### Services autorisés

| Service           | Exposition Internet |
| ----------------- | ------------------- |
| WireGuard         | Oui                 |
| SSH               | Non                 |
| Proxmox WebUI     | Non                 |
| Bastion           | Non                 |
| Ansible           | Non                 |
| Monitoring        | Non                 |
| Services internes | Non                 |

---

## 6.3 Principes d’administration

| Élément                 | Politique                               |
| ----------------------- | --------------------------------------- |
| Administration distante | VPN obligatoire                         |
| SSH                     | Clés uniquement                         |
| Root SSH                | Interdit                                |
| Bastion                 | Point d’entrée unique                   |
| Ansible                 | Accessible uniquement depuis le bastion |
| Monitoring              | Interne uniquement                      |
| Services internes       | Non exposés                             |
| Journalisation          | Centralisée progressivement             |

---

# 7. Roadmap prévisionnelle

## Phase 1 — Fondations

* Installation Proxmox VE
* Configuration réseau
* Mise en place WireGuard
* Validation des flux VPN

---

## Phase 2 — Administration sécurisée

* Déploiement bastion
* Hardening SSH
* Firewall
* Segmentation réseau
* Validation des accès administratifs

---

## Phase 3 — Administration centralisée

* Déploiement VM Ansible
* Gestion des clés SSH
* Inventaires
* Premiers playbooks
* Administration distante des postes

---

## Phase 4 — Supervision

* Déploiement Prometheus
* Déploiement Grafana
* Monitoring de la flotte distante
* Alerting

---

## Phase 5 — Services internes

* DNS / DHCP
* Nextcloud
* Services internes complémentaires
* Sauvegardes

---

# 8. Perspectives d’évolution

Le projet est conçu pour évoluer vers :

* une infrastructure multi-sites ;
* une automatisation avancée ;
* une supervision centralisée complète ;
* une centralisation des logs ;
* une gestion avancée des accès ;
* une architecture orientée Zero Trust ;
* un environnement DevOps/SRE complet.
