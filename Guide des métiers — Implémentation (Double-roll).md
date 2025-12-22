# Guide des métiers — Implémentation **Option 2** (*double-roll*)

Cette implémentation sert à éviter les ambiguïtés quand plusieurs loots partagent une même plage (ex: **80–100**), en séparant :
1) **le roll de réussite** (as-tu réussi à trouver quelque chose ?)
2) **le roll de qualité / type** (qu’est-ce que tu trouves exactement ?)

> Convention : tous les rolls sont sur **1d100** (1 à 100), sauf mention contraire.  
> Les quantités sont ensuite déterminées par une table ou un jet dédié.

---

## 1) Ramassage — Coquillages (*Lac de Crystal*)

### Étape A — Roll de réussite (Trouvaille)
Lance **1d100** :
- **1–79** : Rien (ou “débris/algues” si tu veux narrer)
- **80–100** : **Trouvaille : Coquillages** → passe à l’Étape B

### Étape B — Roll de type (Commun vs Rare)
Lance **1d100** :
- **1–20** : **Coquillage rare**
- **21–100** : **Coquillage (commun)**

### Étape C — Quantité
Lance **1d100** (ou choisis une valeur si tu veux accélérer) :
- Si **Commun** : quantité = **entre 80 et 100**, avec un **maximum absolu de 200**  
  - Méthode simple : quantité = `roll_quantité` mais **minimum 80** et **maximum 100**
- Si **Rare** : quantité = **1 à 20**  
  - Méthode simple : quantité = `ceil(roll_quantité / 5)` (donne 1–20)

> Variante (si tu veux 0 calcul) :  
> - Commun : **1d21 + 79** (donne 80–100)  
> - Rare : **1d20** (donne 1–20)

---

## 2) Pêche — Poisson-chat (*Lac de Crystal*)

### Étape A — Roll de capture
Lance **1d100** :
- **1–30** : +**1 Poisson-chat**
- **31–100** : rien

> Option “temps de pêche” : si le joueur pêche longtemps, autorise **N tentatives** (ex: 3 tentatives par scène).

---

## 3) Drop rare — Pendentif de la sirène amoureuse (*Lac de Crystal*)

Deux façons de l’intégrer en **Option 2** (choisis une seule) :

### Version 2A — Le pendentif est un “bonus” en plus du loot
Après une scène de pêche/ramassage réussie (ou après avoir obtenu un poisson rare), lance **1d100** :
- **99–100** : drop **Pendentif** (*2%*)
- sinon : rien

**Modif DEX (simple) :**
- DEX élevée : aucun changement
- DEX faible : la plage devient **100 uniquement** (*1%*)

### Version 2B — Le pendentif est un résultat de “qualité”
Étape A : roll réussite (ex: 1d100 ; 60–100 = réussite).  
Étape B : roll qualité 1d100 :
- **1–97** : loot normal (poisson/coquillage/table habituelle)
- **98–100** : check pendentif (re-roll 1d100 → 99–100 = pendentif, sinon loot normal)

---

## Notes MJ (recommandations rapides)
- L’Option 2 est idéale quand tu veux garder **les mêmes ranges** sans les “casser” en sous-plages.
- Tu peux ajouter des bonus d’équipement/DEX comme **+X au roll de réussite** (Étape A), sans toucher aux tables de loot (Étape B/C).
- Pour accélérer : fais seulement **Étape A + Étape B**, et fixe une quantité moyenne (ex: commun = 90, rare = 10).