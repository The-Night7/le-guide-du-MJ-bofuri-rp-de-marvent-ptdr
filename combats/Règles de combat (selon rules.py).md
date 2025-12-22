# ⚔️ Règles de combat — Spécification exacte (selon `rules.py`)

Ce document décrit **fidèlement** les règles implémentées dans `rules.py` (résolution d’attaque, dégâts, réduction VIT, contrecoup, et structure du résultat).

---  

## 1) Types d’attaque (`AttackType`)

Les attaques sont typées :

- **`phys`** : physique (stat de dégâts = STR)
- **`magic`** : magique (stat de dégâts = INT)
- **`ranged`** : distance (stat de dégâts = DEX)

---  

## 2) Stat utilisée pour les dégâts : `_attack_stat(attacker, attack_type)`

La stat appelée `atk` dans les calculs vaut :

\[
atk =
\begin{cases}
STR & \text{si } attack\_type = phys\\
INT & \text{si } attack\_type = magic\\
DEX & \text{si } attack\_type = ranged
\end{cases}
\]

> Important : `atk` sert à **calculer les dégâts**, pas la précision.

---  

## 3) Étape 1 — Test de toucher / esquive (AGI)

On calcule deux valeurs :

\[
hit\_a = roll\_a + \frac{AGI(attacker)}{10}
\]
\[
hit\_b = roll\_b + \frac{AGI(defender)}{10}
\]

Condition de toucher :

\[
\text{hit} \iff hit\_a > hit\_b
\]

- Si \(hit\_a = hit\_b\) : **ça ne touche pas** (strictement supérieur requis).

---  

## 4) Étape 2 — Réduction par VIT (armure/résistance)

La défense du défenseur est modélisée par un terme :

\[
vit\_term = \frac{VIT(defender)}{vit\_div}
\]

où :

- si `perce_armure = False` → \(vit\_div = 10\)
- si `perce_armure = True` → \(vit\_div = 100\)

Donc :

- **Normal** : \[vit\_term = \frac{VIT}{10}\]
- **Perce-armure** : \[vit\_term = \frac{VIT}{100}\] (réduction **10× plus faible**, donc bien plus facile de faire des dégâts)

---  

## 5) Cas A — L’attaque touche (`hit_a > hit_b`)

### 5.1 Dégâts bruts avant modificateurs
\[
dmg = (hit\_a - hit\_b) + atk
\]

Interprétation :
- plus tu “gagnes” le jet de toucher, plus tu ajoutes de dégâts
- la stat offensive `atk` est ajoutée directement

### 5.2 Modificateurs selon le type d’attaque

#### Attaque magique (`magic`)
\[
dmg \leftarrow dmg \times
\begin{cases}
1.2 & \text{si } roll\_a > 15\\
0.9 & \text{sinon}
\end{cases}
\]

> Le seuil se base sur **`roll_a`** (le jet brut), pas sur \(hit\_a\).

#### Attaque à distance (`ranged`)
\[
dmg \leftarrow dmg \times 0.95
\]

#### Attaque physique (`phys`)
- pas de multiplicateur (reste tel quel)

### 5.3 Application de la réduction VIT
\[
dmg \leftarrow dmg - vit\_term
\]

Puis on borne :

\[
dmg \leftarrow \max(0, dmg)
\]

### 5.4 Application aux PV du défenseur

\[
defender.hp \leftarrow \max(0, defender.hp - dmg)
\]

---  

## 6) Cas B — L’attaque ne touche pas (`hit_a \le hit_b`)

Le code calcule une “puissance de défense” :

\[
defense\_power = (hit\_b - hit\_a) + vit\_term - atk
\]

- si `defense_power > 0` : il y a **contrecoup / réflexion / riposte** sur l’attaquant
- sinon : **aucun dégât** (attaque juste bloquée/dissipée/évité)

---  

## 7) Contrecoup sur l’attaquant (si `defense_power > 0`)

Le montant du contrecoup dépend du type d’attaque initial :

### 7.1 Physique (`phys`)
\[
contrecoup = defense\_power \times 1.0
\]

Message : *contre l'attaque… dégâts de contrecoup.*

### 7.2 Magique (`magic`)
\[
contrecoup = defense\_power \times 0.7
\]

Message : *résiste au sort… dégâts de réflexion magique.*

### 7.3 Distance (`ranged`)
\[
contrecoup = defense\_power \times 0.5
\]

Message : *esquive et riposte… dégâts.*

Puis on applique aux PV de l’attaquant :

\[
attacker.hp \leftarrow \max(0, attacker.hp - contrecoup)
\]

---  

## 8) Échec sans contrecoup (si `defense_power \le 0`)

Aucun dégât n’est infligé à l’attaquant.

Texte généré selon le type :
- `phys` : *bloque l'attaque… Aucun dégât en retour.*
- `magic` : *dissipe le sort… Aucun dégât en retour.*
- `ranged` : *évite le tir… Aucun dégât en retour.*

---  

## 9) Structure exacte de la sortie (`out: Dict[str, Any]`)

La fonction retourne toujours un dictionnaire contenant au minimum :

- `hit`: bool
- `roll_a`: float
- `roll_b`: float
- `hit_a`: float
- `hit_b`: float
- `attack_type`: `"phys" | "magic" | "ranged"`
- `perce_armure`: bool
- `vit_scale_div`: float (10.0 ou 100.0)
- `atk_stat`: float
- `vit_term`: float
- `raw`: dict (détails numériques)
  - si hit : `raw["damage"] = dmg`
  - sinon : `raw["defense_value"] = defense_power`
- `effects`: liste de strings (log RP lisible)

---  

## 10) Résumé ultra-court (MJ)

1. Calculer \[hit\_a = roll\_a + AGI_a/10\], \[hit\_b = roll\_b + AGI_b/10\]
2. \[vit\_term = VIT/10\] (ou \[VIT/100\] si perce-armure)
3. Si \[hit\_a > hit\_b\] :
   - \[dmg = (hit\_a - hit\_b) + atk\]
   - magique : ×1.2 si \[roll\_a > 15\] sinon ×0.9
   - ranged : ×0.95
   - \[dmg \leftarrow \max(0, dmg - vit\_term)\]
   - retirer `dmg` aux PV du défenseur
4. Sinon :
   - \[defense\_power = (hit\_b - hit\_a) + vit\_term - atk\]
   - si > 0 : contrecoup (×1.0 phys, ×0.7 magic, ×0.5 ranged) sur l’attaquant
   - sinon : rien

---  

## 11) Remarques d’équilibrage (constats “purement code”)

- La différence de jets \((hit\_a - hit\_b)\) impacte **directement** les dégâts : gagner le toucher “fortement” augmente beaucoup le DPS.
- La magie a un comportement spécial : elle est **buffée** si le jet brut est haut (\[roll\_a > 15\]), sinon **nerfée**.
- `perce_armure` ne supprime pas la VIT : il la rend **10× moins efficace**.
- Même quand tu rates, tu peux te prendre du contrecoup si la défense “dépasse” ton attaque : \[defense\_power > 0\].

---  

*Fin — conforme au code fourni.*