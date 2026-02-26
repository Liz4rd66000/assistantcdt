# KAIROS Smart Money Suite — Plan V2 fiabilité

Ce document propose une V2 orientée **fiabilité des signaux** (moins de repaint, moins de faux positifs, meilleure reproductibilité en live).

## 1) Anti-repaint (priorité absolue)

1. Remplacer les appels critiques `request.security(..., lookahead = barmerge.lookahead_on)` par `lookahead_off`.
2. Ajouter un mode `strictNonRepaint` qui:
   - valide les signaux uniquement sur bougie confirmée (`barstate.isconfirmed`),
   - interdit les décisions sur des extrêmes "live" non confirmés,
   - fige les zones PD sur swings confirmés.
3. Séparer clairement:
   - **objets visuels décoratifs** (tolérance repaint),
   - **logique de signal** (tolérance repaint = 0).

## 2) Filtrage structurel des cassures (BOS/CHoCH)

Ajouter ces filtres pour confirmer une cassure:

- Buffer ATR: `close > pivot + atr * bosBufferAtr` (ou inverse en bear).
- Option `confirmBreakOnCloseOnly` pour empêcher déclenchement intra-bar.
- Cooldown minimal entre deux cassures (`breakCooldownBars`) pour éviter le sur-signaling.

Paramètres conseillés:

- `bosBufferAtr = 0.10`
- `breakCooldownBars = 3` à `5`

## 3) Filtrage FVG plus robuste

Le filtre actuel basé sur la taille du corps relatif au prix peut être conservé, mais complété par:

- taille minimale du gap en ATR: `gapSize >= atr * minFvgAtr`,
- option de validation conjointe avec impulsion (`displacement`) ou cassure récente.

Paramètres conseillés:

- `minFvgAtr = 0.20`
- `requireImpulseForFvg = true` (sur unités rapides)

## 4) Score de confluence sans double comptage

Le score V1 additionne parfois des signaux corrélés (sweep + displacement + BOS sur la même impulsion).

Approche recommandée:

- créer des **familles de score**:
  - Structure (0–35)
  - Liquidité (0–35)
  - Contexte HTF/PD (0–20)
  - Timing session (0–10)
- plafonner chaque famille puis sommer.

Cela évite les scores artificiellement élevés dus à un seul événement de marché.

## 5) Sortie exploitable (signal actionnable)

Ajouter une sortie `setupLongReady` / `setupShortReady` basée sur règles explicites, par exemple:

- Long:
  - biais HTF bull,
  - sweep sell-side récent,
  - déplacement haussier,
  - prix en discount/équilibre bas,
  - OB bullish non mitigé.
- Invalidation: clôture sous OB / dernier swing low.

## 6) Alertes standardisées

Créer des `alertcondition()` dédiées:

- `A+ Long`
- `A+ Short`
- `HTF Conflict`
- `Sweep + Displacement`

Message recommandé:

`{{ticker}} | {{interval}} | Setup={{setup}} | Score={{score}} | Bias={{bias}} | Invalidation={{invalidLevel}}`

## 7) Validation quantitative

Pour prouver la fiabilité:

1. Maintenir l’indicateur visuel (`indicator`).
2. Créer une version `strategy` simplifiée pour backtest.
3. Mesurer par actif/timeframe/session:
   - winrate,
   - expectancy,
   - profit factor,
   - max drawdown,
   - stabilité OOS (out-of-sample).

## 8) Paramètres V2 recommandés (départ)

- `strictNonRepaint = true`
- `useHtfBiasFilter = true`
- `bosBufferAtr = 0.10`
- `minFvgAtr = 0.20`
- `breakCooldownBars = 4`
- `confluenceThreshold = 65`
- `sessionBoost = true`

## 9) Feuille de route technique (ordre d’implémentation)

1. Sécuriser anti-repaint (security/lookahead + validation close).
2. Renforcer BOS/CHoCH.
3. Renforcer FVG ATR.
4. Refonte scoring par familles.
5. Alertes + setup actionnable.
6. Version `strategy` de contrôle.

---

## Snippet Pine utile (exemple de patterns V2)

```pine
// Inputs V2
strictNonRepaint = input.bool(true, 'Strict non-repaint')
bosBufferAtr = input.float(0.10, 'BOS buffer (ATR x)', minval = 0.0, step = 0.01)
minFvgAtr = input.float(0.20, 'FVG min size (ATR x)', minval = 0.0, step = 0.01)
breakCooldownBars = input.int(4, 'Break cooldown bars', minval = 0)

// Helpers
signalBarConfirmed = not strictNonRepaint or barstate.isconfirmed

validBullBreak(float level, float atrv) =>
    signalBarConfirmed and close > level + atrv * bosBufferAtr

validBearBreak(float level, float atrv) =>
    signalBarConfirmed and close < level - atrv * bosBufferAtr

validBullFvg(float gapTop, float gapBottom, float atrv) =>
    signalBarConfirmed and (gapTop - gapBottom) >= atrv * minFvgAtr

validBearFvg(float gapTop, float gapBottom, float atrv) =>
    signalBarConfirmed and (gapTop - gapBottom) >= atrv * minFvgAtr
```

Ce snippet n’est pas un remplacement complet de la V1: il illustre la couche "fiabilité" à intégrer.
