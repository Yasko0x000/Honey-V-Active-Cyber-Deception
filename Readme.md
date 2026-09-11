# 🍯 PROJET HONEY-V - ARCHIVE COMPLÈTE DE DÉPLOIEMENT

**Date d'archivage :** Juillet 2026  
**Contexte :** Projet de fin d'études - Master 2 Sécurité Informatique (M2 SI) à l'ESGI.  
**Objectif :** Conception et déploiement d'une infrastructure de Cyber-Déception Active (Honeynet L2 & SOAR).

---

## 👥 CRÉDITS & ÉQUIPE PROJET
Ce projet est le fruit du travail collaboratif de quatre ingénieurs :
*   **Yassine EL HAMIOUI** (*Lead Architecte Infrastructure & Sécurité*)
*   **Daliani ALI** (*Analyste SOC & Threat Intelligence*)
*   **Yanis KHASSER** (*Ingénieur Cyber-Déception - Active Directory & Windows*)
*   **Ilyas TUNAY** (*Ingénieur Cyber-Déception - Linux & API*)

---

## 📚 DOCUMENTATION THÉORIQUE ET TECHNIQUE
L'architecture logicielle, les scénarios d'attaques (Red Team), et la matrice des règles de détection (Wazuh) sont documentés en profondeur.
👉 **Avant tout déploiement, veuillez consulter le fichier `Document_Technique_Honey-V.pdf` inclus dans ce dépôt.**

---

## 🔐 GESTION DES SECRETS (VAULT)
Afin de permettre le déploiement et la prise en main rapide des machines virtuelles par la communauté, l'ensemble des identifiants et accès par défaut (pfSense, Linux, Active Directory, API) sont documentés dans le fichier : Honey-V_Vault_Export.html.
(⚠️ Ces identifiants sont des accès factices, strictement restreints à cet environnement de simulation).

---

## 📂 ACCÈS AUX MACHINES VIRTUELLES (80 Go)
⚠️ **Note sur la volumétrie :** L'infrastructure complète a été exportée sous forme de disques universels (`.vmdk`) et de dumps compressés Proxmox (`.vma.zst`). En raison des limites de taille de GitHub, ces fichiers lourds sont hébergés sur un cloud externe.

🔗 **[CLIQUEZ ICI POUR ACCÉDER AU GOOGLE DRIVE CONTENANT LES VMS https://drive.google.com/drive/folders/1ysrXD0FbHN5zq8J0KUBcR2J5veNlGg78?usp=sharing]**

L'archive Cloud contient la structure suivante :

### 1. Dossier `VMware/` (.vmdk)
Contient les disques bruts universels. Idéal pour remonter l'infra sur VMware ESXi, Workstation ou VirtualBox.
*   `HV-GATEWAY.vmdk` : Le pare-feu pfSense (10.0.1.1)
*   `HV-HONEYWALL.vmdk` : La VM Debian furtive (Pont L2 - Suricata)
*   `HV-AD-MIRROR.vmdk` : Le contrôleur de domaine factice Windows Server 2022
*   `HV-WEB-DECOY.vmdk` : L'Écrin Doré (Nginx) & Honey-API (Flask)
*   `HV-SSH-DECOY.vmdk` : Le Tarpit Cowrie
*   `HV-WAZUH-OS.vmdk` : L'OS du SIEM Wazuh
*   `HV-SOAR.vmdk` : L'orchestrateur TheHive

### 2. Dossier `Proxmox/` (.vma.zst)
Contient les dumps compressés natifs de Proxmox. Utilisables via la commande `qmrestore` pour un redéploiement à l'identique sur un nœud PVE.

### 3. Dossier `Configs_VMs/` (.conf)
*(Aussi inclus dans ce dépôt GitHub).* Contient les "Plans de construction" de chaque machine (Extrait de `/etc/pve/qemu-server/`).
**Indispensable pour connaître les allocations RAM/CPU et les interfaces réseaux (Bridges) à recréer.**

---

## 🏗️ INSTRUCTIONS DE REMONTAGE (DISASTER RECOVERY)
Pour recréer l'infrastructure fonctionnelle, respectez cet ordre de dépendance réseau :

### Étape 1 : Le Réseau
1. Dans l'hyperviseur cible, créez deux commutateurs virtuels (V-Switches) :
   *   `hvtrunk` : Switch principal.
   *   `hvwall` : Switch **isolé** (sans interface physique vers l'extérieur).
2. Connectez le `HV-GATEWAY` (pfSense) sur `hvtrunk`.

### Étape 2 : Le Mur Invisible (Honeywall)
1. Montez la VM `HV-HONEYWALL`.
2. Connectez `net0` sur `hvtrunk`.
3. Connectez `net1` sur `hvwall`.
4. Connectez `net2` sur `hvtrunk` avec le **Tag VLAN 30**.
*Note : Au démarrage, la VM fusionnera net0 et net1 en un pont `br0` furtif.*

### Étape 3 : Les Cibles (Leurres)
1. Montez les leurres (`HV-WEB-DECOY`, `HV-SSH-DECOY`, `HV-AD-MIRROR`).
2. Branchez toutes leurs interfaces sur le switch isolé **`hvwall`**.
*(Le trafic devra alors physiquement traverser le Honeywall pour atteindre le routeur).*

### Étape 4 : Le SOC
1. Montez `HV-WAZUH` et `HV-SOAR` sur le switch `hvtrunk` (Tag VLAN 30).
*Note pour Wazuh : Le disque exporté ne contient que l'OS. Vous devrez recréer et formater un second disque de 100Go (/dev/sdb) sur la nouvelle machine pour accueillir le point de montage `/var/lib/wazuh-indexer`.*

---

## 🎯 PHILOSOPHIE DU PROJET
Ce projet a été conçu avec une ligne directrice stricte : **Zéro Faux Positif**. 
En déployant des leurres à haute interaction dans une zone réseau totalement stérile et isolée (Layer 2), chaque trame capturée est par nature hostile. L'infrastructure Honey-V ne se contente pas d'alerter, elle riposte : la chaîne de réponse automatisée (Kill Chain) bannit les attaquants à la milliseconde près, transformant la sécurité passive en Légitime Défense Active.
