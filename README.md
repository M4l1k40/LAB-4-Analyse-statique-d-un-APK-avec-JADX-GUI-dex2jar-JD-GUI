# 🔐 LAB 4 : Analyse statique d'un APK avec JADX GUI + dex2jar + JD-GUI

## 📌 Description
Ce projet présente une analyse statique d'une application Android (APK) dans un contexte pédagogique.

APK analysé : UnCrackable Level 1 (OWASP)

## 🎯 Objectifs
- Comprendre la structure d’un APK
- Analyser AndroidManifest.xml
- Identifier des vulnérabilités
- Proposer des remédiations

---

## 🛠️ Outils utilisés
- JADX GUI
- dex2jar
- JD-GUI

---

## 🔍 Résultats de l’analyse

### 🔐 Permissions
Aucune permission sensible n’est demandée.

---

### 📤 Composants exportés
- MainActivity (exported implicitement)

---

### ⚠️ Vulnérabilités identifiées

#### 1. Debug activé
- Sévérité : Élevée
- Risque : Attaques dynamiques

#### 2. Secret côté client
- Sévérité : Élevée
- Risque : Reverse engineering

#### 3. Protection root faible
- Sévérité : Moyenne

#### 4. allowBackup activé
- Sévérité : Moyenne

---

## 🛡️ Remédiations
- Désactiver debug
- Déplacer logique vers serveur
- Renforcer détection root
- Désactiver backup

---

## 📁 Structure du projet
├── README.md
├── rapport.md
├── results/
└── screenshots/

---

## ⚠️ Disclaimer
Ce projet est réalisé dans un cadre pédagogique uniquement.
