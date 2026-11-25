# TP 2 — Architectures 2-tiers et 3-tiers comparées

Compte rendu et fichiers du TP 2 du module **Applications Client-Serveur et Web (ACSW)** à l'École Supérieure Polytechnique (ESP) de Dakar — DUT 2 Télécommunications et Réseaux (2026-2027).

## Binôme
* **Gueye Serigne Falilou**
* **Diop Cheikh Mamour Insa**

**Enseignant :** Dr Keba Gueye

---

## Objectif du TP

Comparer concrètement deux architectures sur la même application de gestion de scolarité (base MariaDB `scolarite` mise en place au TP 1) :
* **Architecture 2-tiers (client lourd)** : un script Python (`client_lourd.py`) exécuté sur le poste client qui se connecte directement à la base de données.
* **Architecture 3-tiers (application web)** : une application PHP hébergée sur un serveur Apache intermédiaire, consultée depuis un simple navigateur web.

La comparaison porte sur le déploiement, la mise à jour d'une règle métier, le volume de données échangé sur le réseau et la sécurité.

---

## Environnement utilisé

> **Note :** Notre poste hôte tourne sous **Ubuntu** (et non Windows). Les commandes côté client ont donc été lancées dans un terminal Linux (`python3`, `python3-pymysql`, `curl`).

Les machines communiquent sur le réseau `192.168.56.0/24` :

| Machine | Adresse IP | Rôle |
| :--- | :--- | :--- |
| **Hôte Ubuntu** | `192.168.56.1` | Poste client principal (`client_lourd.py` en 2-tiers, navigateur et `curl` en 3-tiers) |
| **VM1 (Linux)** | `192.168.56.11` | Serveur de base de données MariaDB (port `3306`) |
| **VM2 (Linux)** | `192.168.56.12` | 2e client en 2-tiers, puis serveur d'application Apache + PHP (port `80`) en 3-tiers |

---

## Étapes réalisées

### 1. Application en 2-tiers (client lourd Python)
* Installation du connecteur MariaDB sur l'hôte Ubuntu et VM2 :
  ```bash
  sudo apt install python3-pymysql
