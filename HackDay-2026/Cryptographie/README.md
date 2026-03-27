# Write-Up : The Roughness of Vigenere

**Plateforme :** HackDay  
**Catégorie :** Cryptographie    
**Difficulté :** Moyen / Difficile  

## Objectif

L'objectif est de casser un message chiffré avec l'algorithme de **Vigenère** utilisant une clé très longue (>15 caractères). Le flag final est au format HACKDAY{SHA256(clé)}.

## 1. Analyse Initiale : Test de Kasiski

La première étape consiste à estimer la longueur de la clé en cherchant des répétitions de motifs (trigrammes) dans le texte chiffré.

**Observations :**

* Distances entre répétitions : 272, 1842, 6224, 14036...

* Longueurs candidates identifiées : 17, 19, 23, 29, 31.

* Cependant, les tests sur ces longueurs n'ont pas permis de retrouver un texte clair cohérent.

## 2. Calcul de l'Indice de Coïncidence (IC)

L'IC mesure la probabilité que deux lettres choisies au hasard dans un texte soient identiques. C'est l'outil ultime pour casser Vigenère.

* **Texte anglais naturel :** IC ≈ 0.065 - 0.070

* **Texte aléatoire (chiffré) :** IC ≈ 0.038

**Méthodologie :**
On divise le texte chiffré en N sous-textes (où N est la longueur de clé testée). Si N est la bonne longueur, chaque sous-texte devient un simple chiffre de César, et son IC doit remonter vers 0.065.

## 3. Extension de la Recherche

Les premières tentatives (longueurs 1 à 50) donnaient toutes un IC proche de 0.038, indiquant que la clé était bien plus longue que prévu. J'ai donc automatisé un script Python pour tester jusqu'à 500 caractères.

**Découverte :**
Les longueurs **374** et **744** ont montré une explosion de l'IC (> 0.060), confirmant une clé de 374 caractères.

## 4. Extraction de la Clé

Une fois la longueur de 374 confirmée, j'ai utilisé une analyse de fréquences sur chaque colonne pour identifier le décalage (la lettre de la clé).

**La clé identifiée :**
FGWUIREUYSTZQGHCMILKIPJUYVMEHLRQDKXBOXUDPCTUGAYDDWCDLTKRVJXCQZVVCXGEXIKUORLSGAOHBDSWYYPWGXFGEFTUHJBLWZJDUOSLXERJOBOFJRACQTPEJBQLDFJPJDJTRZMYDRTTMXJOPYLVJYTJKYDJTMYNYMOJAHMQFLILUOFRNWRPCVXHAUEGJNCHNBFPYHGNRLBISOQPUBUEBLPTFSBUOTTEJWWGIWJORTTUOZXOHAJDNPSFUFRESFYVFMXUTPUNYZNSFSUHADZUTIRIGHBCPLQMIDCJNNXFFCXPKGGNGWKYIFOXVZRFYZKDAYGNKSGHACNLERHSFRYXUZQISJJFUPYPKRZADOLNZOTJBXNM

## 5. Génération du Flag

Le challenge demandait le SHA-256 de cette clé.

**Commande de hachage :**

``` Bash
echo -n "FGWUIREUYST..." | sha256sum
```

**Résultat : a3ee1f2b7797cc2aa80a610155868523f6c0202eae82d7e048281891b88d8ff4**

**Flag final :** `HACKDAY{a3ee1f2b7797cc2aa80a610155868523f6c0202eae82d7e048281891b88d8ff4}`

## Leçon Apprise

Même avec une clé de 374 caractères, le chiffre de Vigenère reste vulnérable. Si le texte chiffré est suffisamment long, les propriétés statistiques de la langue d'origine (fréquence du 'E', 'A', etc.) transparaissent toujours à travers l'Indice de Coïncidence. La longueur de la clé complique l'attaque mais ne rend pas l'algorithme incassable.

### Outils utilisés

* **Python 3 :** Scripts personnalisés (collections.Counter, hashlib).

* **CyberChef :** Pour les vérifications rapides d'encodage.
