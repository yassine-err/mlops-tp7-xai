# Model Card - fraude (version servie par l'alias production)

## Modele
- Nom et version dans le registre MLflow : `fraude` version `1` (alias `production`)
- Famille et reglages : `RandomForestClassifier` (Forêt aléatoire de 500 arbres, profondeur maximale 16)
- Run d'origine (MLflow) : `5f9e4065a5394d4fa26b633bdc59a...`

## Objectif et usage prevu
Détecter en temps réel les transactions frauduleuses (paiements par carte) afin de bloquer automatiquement les opérations à haut risque et d'alerter les équipes de sécurité.

## Usages exclus
- Bloquer définitivement et de manière irréversible un compte client sans revue humaine.
- Utiliser le score de fraude pour évaluer la solvabilité d'un client ou pour une décision de crédit.
- Prendre des décisions sur des données non conformes ou hors du périmètre d'entraînement.

## Donnees
- Fichier, nombre de lignes, part de fraudes, empreinte : `data/transactions.csv`, 5000 lignes, 568 fraudes (environ 11,36%), empreinte MD5 : `cf9f6f86c141e7df32c61715c493483f`
- Origine des donnees : Synthétiques (générées avec graine fixe pour le TP).

## Performances (jeu de test, 1 250 transactions)
- F1 : 0.8669
- precision : 0.8837
- rappel : 0.8507
- AUC : 0.9872
*(Note : 114 fraudes détectées sur 134 réelles).*

## Methodes d'explicabilite
- Globale : Importance par permutation et SHAP TreeExplainer.
- Locale : SHAP waterfall et LIME.
- Variables les plus influentes : `montant` (fort impact au-delà de 1000 MAD), `anciennete` (effet protecteur), `nuit` (facteur aggravant), `freq_24h` (facteur aggravant).

## Comparaison avec un modele interpretable
- **Régression logistique (log10(montant))** : F1 = 0.8593 (113/134 fraudes détectées). Très bon compromis performance/lisibilité.
- **Arbre de décision (profondeur 3)** : F1 = 0.5737 (72/134 fraudes détectées). Très lisible mais forte perte de performance.
- **Conclusion** : La forêt aléatoire reste en production pour sa performance brute (114 fraudes), mais la logistique avec log10(montant) constitue une excellente alternative si une transparence totale est exigée.

## Limites et risques connus
- **Données synthétiques** : Le modèle est entraîné sur des données simulées, son comportement sur des données réelles pourrait différer.
- **Instabilité de LIME** : Les explications locales de LIME varient selon la graine aléatoire (random_state), contrairement à SHAP.
- **SHAP ≠ Causalité** : Une forte contribution de `nuit` indique une corrélation avec la fraude, pas une cause directe.
- **Cas rares** : Les fraudes atypiques (ex: petits montants de jour) sont moins bien capturées et expliquées.

## Recommandations d'utilisation et de surveillance
- **Revue humaine** : Toute transaction bloquée (comme celle de Mme Benali) doit pouvoir être révisée par un conseiller.
- **Surveillance des données** : Surveiller toute dérive de l'empreinte MD5 des données (`donnees_md5`).
- **Mise à jour** : Recalculer systématiquement les explications (SHAP/LIME) après chaque réentraînement du modèle.

## Traçabilité
- Run des explications (experience tp7-explicabilite) : `3acd522c8b40428f869151883aeef1b1`
- Auteurs : [Ton Nom et Prénom]
- Date : 12/11/2026
- Prochaine revue : 12/05/2027