# ⚔️ Règles de combat — Spécification exacte (selon `rules.py`) *(format GitHub)*

> GitHub n’affiche pas toujours le LaTeX dans les fichiers `.md` d’un dépôt (README/docs).  
> Cette version est donc **100% compatible GitHub**, sans formules LaTeX, mais strictement conforme au code.

---  

## 1) Types d’attaque (`AttackType`)

Les attaques possibles :

- `phys` : physique
- `magic` : magique
- `ranged` : distance

---  

## 2) Stat utilisée pour les dégâts : `atk_stat`

La stat d’attaque **utilisée pour calculer les dégâts** (et non la précision) dépend du type :

| `attack_type` | Stat `atk_stat` utilisée |
|---|---|
| `phys` | `STR` |
| `magic` | `INT` |
| `ranged` | `DEX` |

> Important : `atk_stat` sert à calculer les dégâts, pas la précision.

---  

## 3) Étape 1 — Test de toucher / esquive (AGI)

Deux valeurs sont calculées :

- `hit_a = roll_a + attacker.AGI / 10`
- `hit_b = roll_b + defender.AGI / 10`

Condition de toucher :
- L’attaque **touche** si `hit_a > hit_b`
- Si `hit_a == hit_b`, l’attaque **ne touche pas** (strictement supérieur requis)

---  

## 4) Étape 2 — Réduction via VIT (armure/résistance)

Le défenseur fournit un terme de réduction `vit_term` :

- Si `perce_armure = False` :
    - `vit_term = defender.VIT / 10`
- Si `perce_armure = True` :
    - `vit_term = defender.VIT / 100` *(VIT 10× moins efficace)*

Le code expose aussi :
- `vit_scale_div = 10.0` (normal) ou `100.0` (perce-armure)

---  

## 5) Cas A — L’attaque touche (`hit_a > hit_b`)

### 5.1 Dégâts de base (avant modifs)
- `dmg = (hit_a - hit_b) + atk_stat`

> Interprétation : plus tu “gagnes” le jet de toucher, plus tu ajoutes de dégâts.

### 5.2 Modificateurs selon le type

- Si `attack_type == "magic"` :
    - si `roll_a > 15` : `dmg *= 1.2`
    - sinon : `dmg *= 0.9`
    - ⚠️ Le seuil se base sur `roll_a` (jet brut), pas `hit_a`.

- Si `attack_type == "ranged"` :
    - `dmg *= 0.95`

- Si `attack_type == "phys"` :
    - pas de multiplicateur

### 5.3 Application de la VIT + bornage
- `dmg -= vit_term`
- `dmg = max(0, dmg)`

### 5.4 Retrait des PV du défenseur
- `defender.hp = max(0, defender.hp - dmg)`

### 5.5 Texte généré (log)
Le texte utilise une description selon le type :
- `phys` → "frappe"
- `magic` → "lance un sort sur"
- `ranged` → "tire sur"

Et ajoute :
- `"{attacker} ... {defender} et inflige XX.XX dégâts."`
- `"PV {defender}: current/max"`

---  

## 6) Cas B — L’attaque ne touche pas (`hit_a <= hit_b`)

On calcule une “puissance de défense” :

- `defense_power = (hit_b - hit_a) + vit_term - atk_stat`

Le code met :
- `raw["defense_value"] = defense_power`

Puis :
- si `defense_power > 0` : contrecoup sur l’attaquant
- sinon : pas de contrecoup (simple blocage/évitement)

---  

## 7) Contrecoup (si `defense_power > 0`)

Montant du contrecoup selon le type d’attaque initial :

- `phys` : `contrecoup = defense_power * 1.0`
    - texte : "contre l'attaque… dégâts de contrecoup."
- `magic` : `contrecoup = defense_power * 0.7`
    - texte : "résiste au sort… dégâts de réflexion magique."
- `ranged` : `contrecoup = defense_power * 0.5`
    - texte : "esquive et riposte… dégâts."

Application :
- `attacker.hp = max(0, attacker.hp - contrecoup)`
- puis log `"PV {attacker}: current/max"`

---  

## 8) Échec sans contrecoup (si `defense_power <= 0`)

Aucun dégât n’est infligé à l’attaquant.

Texte généré :
- `phys` : "{defender} bloque l'attaque de {attacker}. Aucun dégât en retour."
- `magic` : "{defender} dissipe le sort de {attacker}. Aucun dégât en retour."
- `ranged` : "{defender} évite le tir de {attacker}. Aucun dégât en retour."

---  

## 9) Structure exacte du retour (`Dict[str, Any]`)

La fonction retourne un dictionnaire `out` avec :

- `hit` : bool
- `roll_a`, `roll_b` : float
- `hit_a`, `hit_b` : float
- `attack_type` : `"phys" | "magic" | "ranged"`
- `perce_armure` : bool
- `vit_scale_div` : float (10.0 ou 100.0)
- `atk_stat` : float
- `vit_term` : float
- `raw` : dict
    - si hit : `raw["damage"] = dmg`
    - sinon : `raw["defense_value"] = defense_power`
- `effects` : list[str] (messages RP)

---  

## 10) Résumé MJ (procédure courte)

1. `hit_a = roll_a + AGI(attacker)/10`
2. `hit_b = roll_b + AGI(defender)/10`
3. `vit_term = VIT(defender)/10` (ou `/100` si perce-armure)
4. Si `hit_a > hit_b` :
    - `dmg = (hit_a - hit_b) + atk_stat`
    - magic : `dmg *= 1.2` si `roll_a > 15` sinon `dmg *= 0.9`
    - ranged : `dmg *= 0.95`
    - `dmg = max(0, dmg - vit_term)`
    - enlever `dmg` aux PV du défenseur
5. Sinon :
    - `defense_power = (hit_b - hit_a) + vit_term - atk_stat`
    - si `defense_power > 0` : contrecoup (phys×1.0, magic×0.7, ranged×0.5) sur l’attaquant
    - sinon : aucun dégât en retour