# Gobuster: The Basics - TryHackMe

**Difficulté :** Easy
**OS :** Linux
**Date :** 30 mars 2026
**Temps :** ~1h30
**Lien room :** https://tryhackme.com/room/gobusteruf

---

## 🎯 Objectif

Introduction à Gobuster, un outil d'énumération offensive pour découvrir des répertoires web, fichiers, sous-domaines DNS et virtual hosts.

---

## 📚 Introduction

Gobuster est un outil open-source écrit en Go utilisé pour l'énumération de :
- Répertoires et fichiers web (mode `dir`)
- Sous-domaines DNS (mode `dns`)
- Virtual hosts (mode `vhost`)
- Buckets S3 Amazon

![Room Info](screenshots/room-info.png)

---

## 💥 Task 4 - Directory & File Enumeration

### Question 1 : Flag pour skip TLS verification ?

**Réponse :** `--no-tls-validation`

### Énumération des répertoires

```bash
gobuster dir -u "http://www.offensivetools.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**Découverte :**
- `/images` (Status: 301)
- `/home` (Status: 200)
- `/media` (Status: 301)
- `/secret` (Status: 301) ← Répertoire suspect

![Directory Enumeration](screenshots/dir-enum-1.png)

### Question 2 : Quel répertoire attire l'attention ?

**Réponse :** `secret`

### Énumération avec extension .js

```bash
gobuster dir -u "http://www.offensivetools.thm/secret" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .js
```

**Découverte :**
- `/flag.js` (Status: 200)

![JS File Discovery](screenshots/dir-enum-2.png)

### Question 3 : Flag dans le fichier .js ?

Navigation vers `http://www.offensivetools.thm/secret/flag.js`

![Flag Found](screenshots/flag.png)

**Flag :** `THM{ReconWasASuccess}`

---

## 🌐 Task 5 - DNS Subdomain Enumeration

### Question 1 : Flag shorthand obligatoire ?

**Réponse :** `-d`

### Énumération des sous-domaines

```bash
gobuster dns -d offensivetools.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Résultats :**
- `www.offensivetools.thm`
- `forum.offensivetools.thm`
- `store.offensivetools.thm`
- `primary.offensivetools.thm`

![DNS Enumeration](screenshots/dns-enum.png)

### Question 2 : Nombre de sous-domaines ?

**Réponse :** `4`

---

## 🖥️ Task 6 - Virtual Host Enumeration

### Énumération des vhosts

```bash
gobuster vhost -u "http://10.129.141.28" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```

**Résultats (Status 200) :**
- `shop.example.thm`
- `blog.example.thm`
- `academy.example.thm`
- `WWW.example.thm`

![Vhost Enumeration](screenshots/vhost-enum.png)

### Question 1 : Nombre de vhosts répondant 200 ?

**Réponse :** `4`

---

## 📚 Ce que j'ai appris

1. **Gobuster a 3 modes principaux** : `dir`, `dns`, `vhost` pour différents types d'énumération

2. **Extensions sont cruciales** : Le flag `-x` permet de découvrir des fichiers cachés (.js, .php, .txt, etc.)

3. **Wordlists adaptées** : Utiliser des wordlists spécifiques selon le contexte (directories, DNS, subdomains)

4. **Filtrage des faux positifs** : `--exclude-length` est essentiel pour éliminer les réponses similaires en vhost enumeration

5. **Méthode systématique** : L'énumération révèle souvent des vecteurs d'attaque cachés (subdomains oubliés, fichiers de debug, etc.)

---

## 🛠️ Outils utilisés

- Gobuster v3.6
- Wordlists : dirbuster, SecLists
- Burp Suite (analyse HTTP)

---

## 💡 Recommandations de sécurité

Pour se protéger de l'énumération :

1. **Désactiver directory listing** : Configuration serveur web
2. **Fichiers sensibles hors web root** : Ne jamais exposer de fichiers de debug/config
3. **Rate limiting** : Limiter les requêtes pour détecter les scans
4. **WAF** : Détecter et bloquer les patterns d'énumération
5. **Monitoring** : Alerter sur les accès 404 répétés
6. **Sous-domaines** : Vérifier que tous les subdomains sont patchés au même niveau

---

![Completion](screenshots/completion.png)

*Writeup by 0xMalt - TryHackMe Profile : https://tryhackme.com/p/0xMalt*
