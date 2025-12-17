# 🍖 Système de Dévoration & DPS Sans Arme

---

## 🦷 **DÉVORATION DE MOBS**

### Question: Quelle stat utiliser pour bouffer un mob?

**Réponse recommandée: STR + VIT**

### 📊 Justification:

| Stat | Rôle dans la dévoration | Pourquoi? |
|------|------------------------|-----------|
| **STR (Force)** | Puissance de morsure / Capacité à déchirer | Permet de **briser** la défense du mob |
| **VIT (Vitalité)** | Résistance digestive / Endurance | Permet de **digérer** sans subir d'effets négatifs (poison, toxines, etc.) |
| **STR + VIT** | ✅ **COMBINAISON OPTIMALE** | Force pour tuer/immobiliser + Résistance pour digérer |

### 🎲 Mécanique suggérée:

```
Roll de Dévoration = 1d20 + (STR/10) + (VIT/10)

Seuils de réussite:
- 10-14: Dévoration partielle (50% HP mob absorbés)
- 15-18: Dévoration réussie (100% HP mob absorbés)
- 19-20: Dévoration critique (100% HP + bonus temporaire de stats)
- Échec (<10): Mob résiste ou effets négatifs
```

### ⚠️ Conditions:
- Le mob doit être **affaibli** (moins de 30% HP) ou **immobilisé**
- Les boss/élites nécessitent des **rolls plus élevés** ou des **compétences spéciales**
- Certains mobs (poison, élémentaires) peuvent infliger des **malus** si VIT insuffisante

---

## 👊 **DPS SANS ARME (GRAIL)**

### 📌 Système de base:

**Dégâts de base sans arme: 2% des HP du mob**

### 🎲 Scaling avec le roll:

| Roll (d20) | Multiplicateur de dégâts | Exemple (mob 1000 HP) |
|------------|--------------------------|------------------------|
| 1-5 | x0.5 (échec partiel) | 10 HP (1%) |
| 6-10 | x1 (normal) | 20 HP (2%) |
| 11-15 | x1.5 | 30 HP (3%) |
| 16-19 | x2 | 40 HP (4%) |
| 20 (critique) | x3 | 60 HP (6%) |

### 📈 Scaling avec la différence de niveau:

**Contre mobs PLUS FAIBLES:**

```
Bonus de dégâts = +0.5% par niveau de différence

Exemple:
- Joueur Lvl 30 vs Mob Lvl 20 (-10 niveaux)
- Dégâts de base: 2% + (10 × 0.5%) = 7% des HP du mob
- Avec roll 20: 7% × 3 = 21% des HP du mob
```

**Contre mobs PLUS FORTS:**

```
Malus de dégâts = -0.3% par niveau de différence (minimum 0.5%)

Exemple:
- Joueur Lvl 20 vs Mob Lvl 30 (+10 niveaux)
- Dégâts de base: 2% - (10 × 0.3%) = max(0.5%, -1%) = 0.5%
- Avec roll 20: 0.5% × 3 = 1.5% des HP du mob
```

---

## 👑 **CONTRE LES BOSS**

### Même système, mais avec ajustements:

| Type de Boss | Modificateur | Raison |
|--------------|--------------|--------|
| **Boss Standard** | Dégâts × 0.75 | Résistance accrue |
| **Boss Élite** | Dégâts × 0.5 | Défense renforcée |
| **Boss Event** | Dégâts × 0.6 | Variable selon boss |

### 💡 Exemple complet:

**Joueur Lvl 50 vs Boss Élite Lvl 50 (5000 HP)**

```
1. Dégâts de base: 2% de 5000 = 100 HP
2. Modificateur Boss Élite: 100 × 0.5 = 50 HP
3. Roll 18 (x2): 50 × 2 = 100 HP
4. Avec STR 200: +20 bonus = 120 HP finaux
```

---

## 🔧 **FORMULE COMPLÈTE SUGGÉRÉE**

```python
# Dégâts sans arme
base_dmg_percent = 2.0

# Ajustement niveau
level_diff = player_level - mob_level
if level_diff > 0:  # Mob plus faible
    base_dmg_percent += level_diff * 0.5
else:  # Mob plus fort
    base_dmg_percent = max(0.5, base_dmg_percent + level_diff * 0.3)

# Calcul de base
base_damage = (mob_max_hp * base_dmg_percent / 100)

# Multiplicateur roll
roll_multiplier = {
    range(1, 6): 0.5,
    range(6, 11): 1.0,
    range(11, 16): 1.5,
    range(16, 20): 2.0,
    20: 3.0
}[roll]

# Modificateur boss
if is_boss:
    boss_modifier = 0.75 if standard else 0.5 if elite else 0.6
    base_damage *= boss_modifier

# Bonus STR
str_bonus = player_str / 10

# Dégâts finaux
final_damage = (base_damage * roll_multiplier) + str_bonus
```

---

## 📋 **RÉSUMÉ**

### 🍖 Dévoration:
- **Stat utilisée:** STR + VIT (combinaison)
- **STR** pour la puissance de morsure
- **VIT** pour résister aux effets du mob

### 👊 Combat sans arme:
- **Base:** 2% HP du mob
- **Scale avec roll:** x0.5 à x3
- **Scale avec niveau:** +0.5% par niveau de diff (contre plus faible)
- **Contre boss:** Réduction de 25% à 50% des dégâts
- **Bonus STR:** +STR/10 dégâts fixes

---

**Besoin de précisions ou d'ajustements sur ces mécaniques?** 🎮