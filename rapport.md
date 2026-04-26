**RAPPORT D\'ANALYSE STATIQUE**

Application Android Mobile

**UnCrackable Level 1 --- OWASP MASTG**

  ----------------------- ------------------------------------------------------------------
  **Date d\'analyse**     26 avril 2026

  **Analyste**            Malika ELAANTRI

  **APK analysé**         UnCrackable-Level1.apk

  **Version**             1.0 (versionCode: 1)

  **Provenance**          OWASP MASTG --- GitHub officiel

  **SHA-256**             1da8bf57d266109f9a07c01bf7111a1975ce01f190b9d914bcd3ae3dbef96f21

  **JADX GUI**            v1.5.5

  **dex2jar**             v2.x (Kali Linux)

  **JD-GUI**              v1.6.x
  ----------------------- ------------------------------------------------------------------

*CONFIDENTIEL --- Usage pédagogique uniquement*

**A. Résumé exécutif**

Cette analyse statique a révélé 5 vulnérabilités potentielles dans
l\'application UnCrackable Level 1 (owasp.mstg.uncrackable1).
L\'application est intentionnellement conçue à des fins pédagogiques par
l\'OWASP MASTG et présente plusieurs failles de sécurité classiques du
développement Android.

**Les principales préoccupations concernent :**

- L\'attribut android:debuggable=\"true\" actif en production,
  permettant l\'attachement d\'un débogueur externe.

- L\'absence de protection efficace contre le root malgré la présence de
  vérifications contournables.

- Le stockage d\'un secret chiffré dans le code source (logique de
  validation côté client).

**Le niveau de risque global est évalué comme : ÉLEVÉ dans un contexte
de production réel.**

**Actions prioritaires recommandées :**

1.  Retirer android:debuggable=\"true\" du manifeste avant tout
    déploiement en production.

2.  Ne jamais stocker de logique de validation de secret côté client ---
    déléguer au serveur.

3.  Renforcer les protections anti-root avec des solutions éprouvées
    (SafetyNet/Play Integrity API).

**B. Informations générales de l\'APK**

**B.1 Manifeste Android**

  ----------------------------- -------------------------------------------------
  **Package principal**         owasp.mstg.uncrackable1

  **versionName / versionCode** 1.0 / 1

  **minSdkVersion**             19 (Android 4.4 KitKat)

  **targetSdkVersion**          28 (Android 9 Pie)

  **android:debuggable**        **true ⚠ CRITIQUE**

  **android:allowBackup**       **true ⚠ À corriger**

  **usesCleartextTraffic**      non défini (absent --- OK)

  **network_security_config**   absent
  ----------------------------- -------------------------------------------------

**B.2 Structure du code source**

  ------------------------------------------- ------------------------------------------
  **Classe**                                  **Rôle identifié**

  sg.vantagepoint.uncrackable1.MainActivity   Activité principale --- vérification du
                                              secret saisi

  sg.vantagepoint.uncrackable1.a              Classe de validation --- déchiffrement
                                              AES + comparaison

  sg.vantagepoint.a.a                         Implémentation du déchiffrement AES

  sg.vantagepoint.a.b                         Détection du mode debuggable

  sg.vantagepoint.a.c                         Détection de root (3 méthodes)

  owasp.mstg.uncrackable1.R                   Références aux ressources (auto-généré)
  ------------------------------------------- ------------------------------------------

**C. Constats détaillés**

+-------------------------------------+-------------------------------------------------+
| **Constat #1 --- android:debuggable=\"true\" en production**                          |
+-------------------------------------+-------------------------------------------------+
| **Sévérité**                        | **● Élevée**                                    |
+-------------------------------------+-------------------------------------------------+
| **Description**                     | L\'attribut android:debuggable est défini à     |
|                                     | true dans le manifeste. L\'application peut     |
|                                     | être déboguée sur n\'importe quel appareil      |
|                                     | Android via adb sans restriction.               |
+-------------------------------------+-------------------------------------------------+
| **Localisation**                    | AndroidManifest.xml → \<application             |
|                                     | android:debuggable=\"true\"\> (ligne 11)        |
+-------------------------------------+-------------------------------------------------+
| **Impact potentiel**                | Un attaquant peut attacher un débogueur (jdb,   |
|                                     | Frida, GDB) au processus en cours d\'exécution, |
|                                     | inspecter la mémoire, modifier les variables en |
|                                     | runtime et contourner toutes les vérifications  |
|                                     | de sécurité de l\'application.                  |
+-------------------------------------+-------------------------------------------------+
| **Remédiation**                     | Retirer android:debuggable=\"true\" du          |
|                                     | manifeste. En production, ne jamais inclure cet |
|                                     | attribut --- il est false par défaut. Utiliser  |
|                                     | des variantes de build (debug/release) dans     |
|                                     | Gradle.                                         |
+-------------------------------------+-------------------------------------------------+

+-------------------------------------+-------------------------------------------------+
| **Constat #2 --- Validation du secret côté client**                                   |
+-------------------------------------+-------------------------------------------------+
| **Sévérité**                        | **● Élevée**                                    |
+-------------------------------------+-------------------------------------------------+
| **Description**                     | La méthode verify() de MainActivity appelle     |
|                                     | a.a(str) pour valider le secret saisi.          |
|                                     | L\'ensemble de la logique de validation         |
|                                     | (déchiffrement AES + comparaison) est embarqué  |
|                                     | dans l\'APK et extractible par décompilation.   |
+-------------------------------------+-------------------------------------------------+
| **Localisation**                    | sg/vantagepoint/uncrackable1/MainActivity.java  |
|                                     | → méthode verify(View) + classe a.java          |
+-------------------------------------+-------------------------------------------------+
| **Impact potentiel**                | Tout utilisateur disposant de l\'APK peut       |
|                                     | décompiler l\'application, analyser la logique  |
|                                     | de validation et retrouver le secret sans       |
|                                     | jamais interagir avec l\'application. Aucune    |
|                                     | protection réelle du secret n\'est possible     |
|                                     | côté client.                                    |
+-------------------------------------+-------------------------------------------------+
| **Remédiation**                     | Déléguer la validation du secret à un serveur   |
|                                     | backend via une API sécurisée (HTTPS). Ne       |
|                                     | jamais stocker de secrets ou de logique de      |
|                                     | validation critique dans le code client.        |
+-------------------------------------+-------------------------------------------------+

+-------------------------------------+-------------------------------------------------+
| **Constat #3 --- Protections anti-root contournables**                                |
+-------------------------------------+-------------------------------------------------+
| **Sévérité**                        | **● Moyenne**                                   |
+-------------------------------------+-------------------------------------------------+
| **Description**                     | L\'application vérifie la présence de root via  |
|                                     | trois méthodes dans c.a(), c.b(), c.c() avant   |
|                                     | de démarrer. Ces vérifications reposent sur des |
|                                     | techniques détectables et contournables         |
|                                     | (présence de binaires su, tags de build,        |
|                                     | chemins système).                               |
+-------------------------------------+-------------------------------------------------+
| **Localisation**                    | sg/vantagepoint/a/c.java → méthodes a(), b(),   |
|                                     | c() appelées depuis MainActivity.onCreate()     |
+-------------------------------------+-------------------------------------------------+
| **Impact potentiel**                | Sur un appareil rooté avec des outils de        |
|                                     | masquage de root (Magisk Hide, Zygisk), ces     |
|                                     | vérifications peuvent être contournées. Un      |
|                                     | attaquant peut alors exécuter l\'application    |
|                                     | dans un environnement root pour l\'analyser     |
|                                     | dynamiquement.                                  |
+-------------------------------------+-------------------------------------------------+
| **Remédiation**                     | Intégrer une API d\'attestation d\'intégrité    |
|                                     | robuste (Google Play Integrity API,             |
|                                     | ex-SafetyNet). Effectuer les vérifications côté |
|                                     | serveur. Utiliser plusieurs couches de          |
|                                     | détection indépendantes et régulièrement mises  |
|                                     | à jour.                                         |
+-------------------------------------+-------------------------------------------------+

+-------------------------------------+-------------------------------------------------+
| **Constat #4 --- android:allowBackup=\"true\"**                                       |
+-------------------------------------+-------------------------------------------------+
| **Sévérité**                        | **● Moyenne**                                   |
+-------------------------------------+-------------------------------------------------+
| **Description**                     | L\'attribut allowBackup n\'est pas défini à     |
|                                     | false, activant la sauvegarde ADB par défaut.   |
|                                     | Les données de l\'application sont extractibles |
|                                     | sans root sur Android \< 9.                     |
+-------------------------------------+-------------------------------------------------+
| **Localisation**                    | AndroidManifest.xml → \<application             |
|                                     | android:allowBackup=\"true\"\> (ligne 11)       |
+-------------------------------------+-------------------------------------------------+
| **Impact potentiel**                | Via la commande adb backup, un attaquant ayant  |
|                                     | accès physique à l\'appareil peut extraire les  |
|                                     | données stockées par l\'application             |
|                                     | (SharedPreferences, bases de données, fichiers  |
|                                     | internes) sans avoir besoin de root.            |
+-------------------------------------+-------------------------------------------------+
| **Remédiation**                     | Ajouter android:allowBackup=\"false\" dans le   |
|                                     | manifeste. Si une fonctionnalité de sauvegarde  |
|                                     | est requise, implémenter BackupAgent avec un    |
|                                     | contrôle précis des données sauvegardées.       |
+-------------------------------------+-------------------------------------------------+

+-------------------------------------+-------------------------------------------------+
| **Constat #5 --- Absence d\'obfuscation du code**                                     |
+-------------------------------------+-------------------------------------------------+
| **Sévérité**                        | **● Faible**                                    |
+-------------------------------------+-------------------------------------------------+
| **Description**                     | Le code source est décompilable directement     |
|                                     | avec JADX ou JD-GUI avec une lisibilité         |
|                                     | quasi-parfaite. Aucune obfuscation ProGuard/R8  |
|                                     | n\'est appliquée. Les noms de classes (ex:      |
|                                     | MainActivity, verify) sont conservés en clair.  |
+-------------------------------------+-------------------------------------------------+
| **Localisation**                    | Sources entières --- visible dans JADX GUI et   |
|                                     | JD-GUI après décompilation du JAR (app.jar)     |
+-------------------------------------+-------------------------------------------------+
| **Impact potentiel**                | La logique interne de l\'application, y compris |
|                                     | les mécanismes de protection, est entièrement   |
|                                     | lisible. Cela facilite l\'analyse statique et   |
|                                     | réduit le temps nécessaire à un attaquant pour  |
|                                     | comprendre le fonctionnement de l\'application. |
+-------------------------------------+-------------------------------------------------+
| **Remédiation**                     | Activer ProGuard/R8 en mode release dans        |
|                                     | build.gradle avec des règles d\'obfuscation     |
|                                     | agressives. L\'obfuscation ne remplace pas une  |
|                                     | architecture sécurisée mais augmente le coût de |
|                                     | l\'analyse pour un attaquant.                   |
+-------------------------------------+-------------------------------------------------+

**D. Annexes**

**D.1 Permissions demandées (uses-permission)**

Aucune permission uses-permission n\'est déclarée dans le manifeste de
cette application. La surface d\'attaque liée aux permissions système
est donc nulle --- l\'application n\'accède pas au réseau, au stockage
externe, à la caméra ou à d\'autres ressources sensibles.

**D.2 Composants exportés**

  ------------- ------------------------------------------- --------------- -------------------
  **Type**      **Composant**                               **Exported**    **Intent-filter**

  Activity      sg.vantagepoint.uncrackable1.MainActivity   **true          MAIN / LAUNCHER
                                                            (implicite)**   
  ------------- ------------------------------------------- --------------- -------------------

*Note : Depuis Android 12 (API 31), l\'attribut android:exported doit
être déclaré explicitement pour tout composant possédant un
intent-filter. Son absence constitue une mauvaise pratique
indépendamment de la valeur implicite appliquée.*

**D.3 Ressources analysées**

  ----------------------------- --------------------------------------------
  **Ressource**                 **Observation**

  AndroidManifest.xml           debuggable=true, allowBackup=true, 1
                                activity avec intent-filter

  res/values/strings.xml        5 chaînes --- aucun secret en clair. Chaîne
                                edit_text=\'Enter the Secret String\'

  network_security_config.xml   Absent --- pas de configuration réseau
                                personnalisée

  classes.dex                   1 seul fichier DEX (pas de multi-dex) ---
                                5528 bytes

  app.jar (généré)              5.4 KB --- converti avec dex2jar, analysé
                                avec JD-GUI
  ----------------------------- --------------------------------------------

**D.4 Comparaison outils --- JADX vs JD-GUI**

  ----------------- --------------------------- ---------------------------
  **Aspect**        **JADX GUI v1.5.5**         **JD-GUI**

  **Navigation**    Structure Android complète  Hiérarchie Java uniquement
                    (manifeste, ressources,     (packages + classes)
                    code)                       

  **Ressources      Accès direct XML, assets,   Absent --- JAR uniquement
  Android**         drawables                   

  **R.id            Résolues automatiquement    Valeurs brutes (ex:
  (références)**    (R.id.edit_text)            2130837505)

  **Vue Smali**     Onglet Smali intégré par    Absent
                    classe                      

  **Recherche       Ctrl+F sur toutes les       Recherche fichier courant
  globale**         sources et ressources       uniquement

  **Obfuscation**   Tentative de reconstruction Noms obfusqués bruts
                    des noms                    conservés

  **Rapidité**      Indexation complète (plus   Ouverture JAR instantanée
                    lent)                       
  ----------------- --------------------------- ---------------------------

**E. Conclusion**

L\'application UnCrackable Level 1 est une application
intentionnellement vulnérable conçue par l\'OWASP MASTG à des fins
pédagogiques. Cette analyse statique a permis d\'identifier 5 constats
de sécurité, dont 2 de sévérité élevée.

Le constat le plus critique reste l\'activation de
android:debuggable=\"true\", qui permet à un attaquant de contourner
dynamiquement l\'ensemble des protections embarquées. Le second constat
majeur est la validation du secret côté client : indépendamment de
l\'obfuscation ou du chiffrement utilisé, toute logique de validation
embarquée dans un APK peut être analysée et contournée.

Ces vulnérabilités illustrent des erreurs de conception fréquentes dans
le développement mobile Android et soulignent l\'importance d\'une
architecture sécurisée dès la conception (Security by Design), où la
logique critique est déportée côté serveur et les configurations de
build sont strictement contrôlées selon l\'environnement cible.
