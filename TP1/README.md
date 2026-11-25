# TP 1 : Le modèle client-serveur

**École :** UCAD — École Supérieure Polytechnique de Dakar (ESP)
**Filière :** DUT 2 Télécommunications et Réseaux (2026-2027)
**Module :** Applications Client-Serveur et Web (ACSW)
**Enseignant :** Dr Keba Gueye
**Binôme :**
- Gueye Serigne Falilou
- Diop Cheikh Mamour Insa

---

## Présentation du TP

L'objectif de ce TP est de comprendre le fonctionnement du modèle client-serveur en mettant en place deux services sur une machine serveur (**MariaDB** et **Apache**) et en les interrogeant depuis deux machines clientes.

### Environnement réseau (`192.168.56.0/24`)

| Machine | Adresse IP | Rôle |
| :--- | :--- | :--- |
| Poste Windows | `192.168.56.1` | Client 1 (HeidiSQL, navigateur web) |
| VM1 (Ubuntu) | `192.168.56.11` | Serveur MariaDB (port `3306`) et Apache (port `80`) |
| VM2 (Ubuntu) | `192.168.56.12` | Client 2 (`mariadb`, `curl`) |

---

## Réponses aux questions

### Partie 1 — Préparer le serveur (VM1)

- **Q1.** La base `scolarite` contient **30 étudiants**, **6 EC** et **180 notes**, répartis dans 4 tables (`filiere`, `etudiant`, `ec`, `note`). La table `note` fait le lien entre `etudiant` (via `matricule`) et `ec` (via le code de l'EC).
- **Q2.** Non, ce ne sont pas le même programme. `mariadbd` est le démon serveur qui tourne en arrière-plan et gère les données, tandis que `mariadb` est le programme client lancé par l'utilisateur. Ils ont deux PID différents dans `ps -ef | grep -i mariadb`.

### Partie 2 — Ouvrir le service au réseau

- **Q3.** Le code `10061` signifie « connexion refusée ». VM1 était bien joignable (réponse au `ping`) et MariaDB fonctionnait, mais il n'écoutait que sur `127.0.0.1` et pas sur l'interface réseau.
- **Q4.** `127.0.0.1` limite l'écoute à la machine locale (loopback), alors que `0.0.0.0` permet d'écouter sur toutes les interfaces réseau IPv4. Ubuntu met `127.0.0.1` par défaut pour des raisons de sécurité.
- **Q5.** On obtient `USER() = etudiant@192.168.56.1` (le compte et l'IP avec lesquels le client se présente) et `CURRENT_USER() = etudiant@192.168.56.%` (le compte enregistré sur le serveur qui a été retenu pour appliquer les droits).
- **Q6.** La connexion en `root` depuis le réseau est refusée. C'est une bonne chose car `root` a tous les droits sur la base ; le bloquer à distance limite les risques d'attaque.

### Partie 3 — Plusieurs clients, un seul serveur

- **Q7.** Lors des deux `SELECT SLEEP(20)` simultanés, on relève :
  - Client Windows : `192.168.56.1:58866`
  - Client VM2 : `192.168.56.12:49284`
  Ces ports sources sont choisis temporairement par le système d'exploitation de chaque client. Ils sont différents de `3306` (port fixe du serveur) pour que le serveur puisse distinguer chaque connexion.
- **Q8.** Le compte `etudiant` reçoit une erreur `1142` (`UPDATE command denied`) et le compte `secretariat` reçoit aussi une erreur `1142` sur `DELETE`. Les droits sont stockés et vérifiés côté serveur (base système `mysql`), donc le client ne peut pas les contourner.
- **Q9.** Oui, Windows voit tout de suite la nouvelle note (`12`) modifiée par VM2. Comme les données sont centralisées sur le serveur unique, tous les clients lisent toujours la même information à jour.

### Partie 4 — Le serveur web (Apache)

- **Q10.** Dans `curl -v`, les lignes `>` représentent la requête du client (`GET / HTTP/1.1`, en-têtes `Host`, `User-Agent`) et les lignes `<` représentent la réponse du serveur (`HTTP/1.1 200 OK`, en-têtes `Server`, `Content-Type`).
- **Q11.** Dans `/var/log/apache2/access.log`, Apache enregistre l'IP du client, la date/heure, la requête (`GET / HTTP/1.1`), le code HTTP (`200`), la taille envoyée et le `User-Agent` (navigateur Windows ou `curl` sur VM2).
- **Q12.** Le système d'exploitation utilise le **port de destination** du segment TCP pour transmettre les données au bon programme (`3306` pour MariaDB, `80` pour Apache). Le client doit connaître l'adresse IP (la machine) et le port (le service) car c'est lui qui initie la connexion.
- **Q13.**
  - **Connexion persistante (MariaDB)** : garde le contexte (utilisateur, transaction), mais consomme des ressources serveur même au repos.
  - **Requêtes indépendantes (HTTP)** : serveur plus léger pouvant gérer beaucoup de clients, mais sans mémoire entre deux requêtes. Pour « se souvenir » d'un utilisateur, un site web utilise un identifiant de session envoyé dans un **cookie**.

### Partie 5 — Les limites du modèle

- **Q14.** Quand MariaDB est arrêté, les deux clients reçoivent une erreur de connexion refusée (`2002 / 10061`). Tous les clients sont touchés : c'est le **point unique de défaillance (SPOF)**.
- **Q15.** La capture `tcpdump` montre en clair le nom du compte (`etudiant`), la requête SQL et les résultats (nom et notes). Le mot de passe n'apparaît pas en clair (authentification par défi/empreinte). Pour sécuriser les échanges, il faut chiffrer la connexion avec **TLS**, un **tunnel SSH** ou un **VPN**.

---

## Synthèse

| Caractéristique | MariaDB | Apache (HTTP) |
| :--- | :--- | :--- |
| Serveur et port | VM1 (`192.168.56.11`), port `3306` | VM1 (`192.168.56.11`), port `80` |
| Clients utilisés | `mariadb`, HeidiSQL | Navigateur web, `curl` |
| Initiative de connexion | Le client | Le client |
| Durée de la connexion | Persistante (toute la session) | Courte (requête / réponse) |
| Vérification des droits | Côté serveur (par compte) | Côté serveur (configuration) |
| Arrêt du serveur | Tous les clients bloqués | Page inaccessible pour tous |

### Conclusion (Q16)

Dans le modèle client-serveur, un serveur écoute sur un port connu et répond aux requêtes envoyées par des clients. Les clients ne communiquent jamais entre eux et passent toujours par le serveur.

- **Avantages :** centralisation et cohérence des données pour tous les clients ; gestion centralisée des droits et de la sécurité.
- **Inconvénients :** point unique de défaillance (si le serveur tombe, tout s'arrête) ; trafic en clair par défaut sur le réseau (nécessite TLS/SSH/VPN).

---

## Contenu du dossier

- `README.md` : résumé du TP et réponses aux questions.
- `rapport_TP1.pdf` : compte rendu complet avec les captures d'écran.%
