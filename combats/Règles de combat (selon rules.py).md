# ⚔️ Règles de combat — Implémentation `rules.py` (référence MJ/Joueurs)

> Ce document décrit **les règles telles qu’elles sont codées** dans `rules.py`.
>
> ⚠️ **Note importante :** je ne vois ici qu’un **extrait partiel** de `rules.py` (le fichier est tronqué dans l’aperçu).  
> J’ai donc documenté **tout ce qui est certain** (signatures, formules visibles, conventions), et j’ai laissé des **encarts “À confirmer”** pour les morceaux manquants.  
> Si tu colles le contenu complet de `rules.py`, je te sors une version 100% exhaustive.

---  

## 1) Entités et stats utilisées

Les calculs se font entre deux entités runtime :

- **Attaquant** : `attacker: RuntimeEntity`
- **Défenseur** : `defender: RuntimeEntity`

Les stats utilisées (visibles dans l’extrait) :

- `STR` : Force (attaques physiques)
- `INT` : Intelligence (attaques magiques)
- `DEX` : Dextérité (attaques à distance)
- `AGI` : Agilité (sert au **toucher / esquive**)

---  

## 2) Types d’attaque (`AttackType`)

Le code définit :

- `phys` : attaque **physique**
- `magic` : attaque **magique**
- `ranged` : attaque **distance**

Cela influe sur **la stat d’attaque utilisée pour calculer les dégâts** (pas la précision).

---  

## 3) Stat d’attaque (dégâts) : `_attack_stat`

Fonction : `_attack_stat(attacker, attack_type) -> float`

Règle codée :

- Si `attack_type = "phys"` → stat utilisée = `STR`
- Si `attack_type = "magic"` → stat utilisée = `INT`
- Si `attack_type = "ranged"` → stat utilisée = `DEX`

En clair :

\[
\text{StatDégâts} =
\begin{cases}
STR & \text{si phys}\\
INT & \text{si magic}\\
DEX & \text{si ranged}
\end{cases}
\]

> ✅ Cette stat est explicitement décrite comme : **“Stat qui sert à calculer les dégâts (pas la précision).”**

---  

## 4) Résolution d’une attaque : `resolve_attack`

Signature (visible) :

- `resolve_attack(attacker, defender, roll_a, roll_b, attack_type="phys", perce_armure=False) -> Dict[str, Any]`

Paramètres importants :

- `roll_a` : jet (type d20 attendu côté attaquant, mais c’est passé en `float`)
- `roll_b` : jet (type d20 attendu côté défenseur, mais c’est passé en `float`)
- `attack_type` : `"phys" | "magic" | "ranged"`
- `perce_armure` : booléen (active un mode “perce-armure”)

---  

## 5) Jet de toucher / esquive (précision)

Le docstring (visible) décrit :

- Attaquant :
  \[
  A = d20 + \frac{AGI(\text{attacker})}{10}
  \]
- Défenseur :
  \[
  D = d20 + \frac{AGI(\text{defender})}{10}
  \]
- Condition de toucher :
  \[
  \text{hit si } A > D
  \]

**Interprétation MJ :**
- `AGI` ajoute un bonus “léger” (divisé par 10).
- En cas d’égalité \(A = D\), l’attaque **ne touche pas** (puisque la condition est strictement \(>\)).

---  

## 6) Dégâts, armure, perce-armure (À confirmer)

L’extrait s’arrête au milieu de la docstring (“hit si A > D...”), donc les règles suivantes sont **probables** mais **non vérifiables** sans la fin du fichier.

### 6.1 Dégâts bruts (probable)
On s’attend à une formule de type :

- dégâts bruts basés sur `StatDégâts` (cf. `_attack_stat`)
- éventuellement modulés par d’autres stats (VIT/DEF), par un coefficient, et/ou par un jet

**À confirmer dans le code :**
- Est-ce que `roll_a` influence aussi les dégâts (critique, variance) ?
- Y a-t-il une attaque de base “arme” côté `RuntimeEntity` ?
- Existe-t-il une réduction via `VIT(defender)` ?

### 6.2 Réduction / Armure (probable)
Le paramètre `perce_armure: bool` suggère une branche :

- `perce_armure = False` : dégâts réduits par une défense / armure (souvent `VIT`)
- `perce_armure = True` : ignore tout ou partie de cette défense

**À confirmer dans le code :**
- Quel pourcentage/quelle règle d’ignorance ?
- Perce-armure ignore-t-il totalement la réduction ou seulement une partie ?

---  

## 7) Sortie de la fonction (résultat)

`resolve_attack(...)` renvoie un dictionnaire `Dict[str, Any]`.

**Champs attendus (À confirmer) :**
- `hit` (bool)
- `A`, `D` (valeurs des jets comparés)
- `damage` (dégâts finaux)
- éventuellement `damage_raw`, `mitigated`, `crit`, etc.

Sans la fin du code, on ne peut pas lister les clés exactes.

---  

## 8) Exemples rapides (basés sur les règles certaines)

### Exemple 1 — toucher raté
- Attaquant : `AGI = 12`, jet `roll_a = 10`
- Défenseur : `AGI = 20`, jet `roll_b = 9`

\[
A = 10 + \frac{12}{10} = 11.2
\quad ; \quad
D = 9 + \frac{20}{10} = 11
\]

Ici \(A > D\) donc **ça touche** (de justesse).

### Exemple 2 — égalité = échec
Si \(A = D\), alors **pas de hit** (condition strictement \(>\)).

---  

## 9) Pour finaliser la doc à 100%

Colle-moi le `rules.py` complet (ou au moins la fin à partir de `hit si A > D...`), et je te mets à jour ce document avec :
- la formule exacte des **dégâts finaux**
- la gestion exacte de `perce_armure`
- les **clés exactes** du dict retourné
- les cas limites (crit, minimum de dégâts, etc.)