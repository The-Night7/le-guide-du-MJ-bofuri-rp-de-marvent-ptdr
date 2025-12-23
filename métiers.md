# Guide des métiers — Tables de *roll* (récolte) + Qualité (craft/enchant)

Ce guide regroupe **tous les éléments “métiers” actuellement définis** (récolte/pêche/herboristerie/minage + forge/alchimie/renforcement/enchantement), avec une implémentation claire de l’**Option 2 (Double-roll)** quand c’est nécessaire (ex: coquillages).

---

## 0) Conventions (important)

- Quand un **range est donné sans précision**, c’est un **résultat de roll sur 1–100** (*d100*).
- **d100** = entier de **1 à 100**.
- Bonus de stats : on utilise `floor()` (arrondi à l’inférieur).
    - Exemple : DEX = 37 → `floor(DEX/10) = 3`
- Les “taux de chance” peuvent être joués soit en **% direct**, soit convertis en **plage de d100**.

---

## 1) Craft — Qualité (Forge / Alchimie / Renforcement / Enchantement)

### 1.1 Forge, Alchimie, Renforcement d’Item (DEX)
**Roll final :** `roll = d100 + floor(DEX/10)`

**Résultat :**
- Si `roll < 25` → **Échec**
- Si `25 ≤ roll ≤ 85` → **Qualité standard**
- Si `roll > 85` → **Bonne qualité**
- Si `roll > 95` → **Haute qualité**
- Si `roll ≥ 100` → **Qualité supérieure**

### 1.2 Enchantement d’Item (INT)
**Roll final :** `roll = d100 + floor(INT/10)`

**Résultat :** *(mêmes seuils que ci-dessus)*
- Si `roll < 25` → **Échec**
- Si `25 ≤ roll ≤ 85` → **Qualité standard**
- Si `roll > 85` → **Bonne qualité**
- Si `roll > 95` → **Haute qualité**
- Si `roll ≥ 100` → **Qualité supérieure**

> Règle : si plusieurs seuils sont atteints, on garde **le meilleur palier**.

---

## 2) Récolte — Implémentation **Option 2 (Double-roll)**

**But :** éviter les ambiguïtés quand plusieurs objets partagent la même plage (ex: “80–100”).

- **Étape A — Roll de réussite (d100)** : est-ce que tu trouves quelque chose ?
- **Étape B — Roll de résultat (d100)** : quoi/rare/quantité.

---

# 3) Métiers de récolte (tables complètes)

## 3.1 Ramassage aquatique — Coquillages
**Lieu :** *Lac de Crystal*

### Étape A — Réussite “Je trouve des coquillages ?” (d100)
- `1–79` → **Rien**
- `80–100` → **Réussite** (passer à l’étape B)

### Étape B — Type + Quantité (d100)
- `1–20` → **Coquillage rare**
    - **Quantité :** `1 à 20`
- `21–100` → **Coquillage (commun)**
    - **Quantité :** `80 à 100` (**max 200**)

> Le “max 200” sert de **cap** si vous ajoutez plus tard des bonus (outil, métier, talent, événement).

---

## 3.2 Pêche — Poisson-chat
**Lieu :** *Lac de Crystal*

**Taux de chance :** 30% pour 1

**Implémentation simple (1 roll) :**
- `d100` :
    - `1–30` → **+1 Poisson-chat**
    - `31–100` → **Rien**

---

## 3.3 Drop très rare — Pendentif de la sirène amoureuse
**Lieu :** *Lac de Crystal*

**Condition :** “DEX élevé”, sinon la chance est **réduite**.

**Base (tel qu’écrit) :**
- Drop si `d100 ∈ {98, 99, 100}`
- ⚠️ Note MJ : `{98,99,100}` correspond à **3%** sur un d100, même si c’est noté “2%”.

### Ajustement DEX (proposition claire, facile à appliquer)
Choisir une règle (une seule) :
- **Règle A (seuil DEX)** :
    - si **DEX ≥ 30** → drop sur `98–100`
    - si **DEX < 30** → drop sur `100` uniquement
- **Règle B (malus de roll)** :
    - si DEX faible → `roll = d100 - 5` (ou `-10`) avant de comparer à `98–100`

---

## 3.4 Herboristerie — Plantes
**Lieu :** *Forêts*

**Table de récolte (d100) :**
- `1–50` → **Plante médicinale de soin**
- `51–80` → **Plante anti-poison**
- `81–97` → **Plante anti paralysie / étourdissement**
- `98–100` → **Plante de la vie (anti-malédiction)**

---

## 3.5 Minage — Minerais & Cristaux
**Lieu :** *Grottes rocheuses*

**Table de minage (d100) :**
- `1–30` → **Minerai de roche**
- `31–50` → **Minerai de cuivre**
- `51–80` → **Pierre de mana**
- `81–97` → **Cristal random**
- `98–100` → **Cristal épique**

---

# 4) Résumé ultra-court (pour MJ pressé)

- **Coquillages (Lac de Crystal)** :
    - A-roll `80–100` = trouvé ; B-roll `1–20` rare (1–20), sinon commun (80–100, cap 200)
- **Poisson-chat (Lac de Crystal)** : `1–30` = +1
- **Pendentif (Lac de Crystal)** : `98–100` + condition DEX (réduction si DEX faible)
- **Herboristerie (Forêts)** : 1–50 / 51–80 / 81–97 / 98–100
- **Minage (Grottes rocheuses)** : 1–30 / 31–50 / 51–80 / 81–97 / 98–100
- **Forge/Alchimie/Renforcement** : `d100 + DEX/10` → seuils de qualité
- **Enchantement** : `d100 + INT/10` → seuils de qualité