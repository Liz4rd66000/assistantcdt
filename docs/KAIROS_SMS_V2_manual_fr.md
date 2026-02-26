# Manuel d'utilisation — KAIROS Smart Money Suite V2.2

Ce guide explique **comment utiliser**, **configurer** et **interpréter** l'indicateur `KAIROS SMS V2.2`.

---

## 1) Objectif de l'indicateur

KAIROS SMS V2.2 combine des concepts SMC (structure, liquidité, déséquilibres) avec une logique de fiabilité:

- anti-repaint (mode strict),
- filtres ATR (cassures/FVG),
- confluence pondérée,
- sortie exécutable (PE/SL/TP/RR),
- dashboard + table signal.

L'objectif n'est pas de donner un "signal magique", mais un **cadre décisionnel discipliné**.

---

## 2) Installation rapide

1. Ouvrir TradingView > Pine Editor.
2. Coller le contenu de `docs/KAIROS_SMS_V2.pine`.
3. Sauvegarder puis `Add to chart`.
4. Ouvrir les paramètres de l'indicateur.

---

## 3) Comprendre les blocs de paramètres

## 3.1 Display

- **Preset**: `Custom`, `Scalping`, `Intraday`, `Swing`, `Full`.
- **Graph view mode**: `Full` ou `Clean`.
- **Auto preset profile**: profil auto de réglage (`Manual`, `Auto-Scalp`, `Auto-Intraday`, `Auto-Swing`).
- **Execution-only view**: masque l'analytique visuelle pour garder une lecture exécution.
- **Show dashboard / Show signal table**: affichage des tableaux.
- Couleurs dashboard personnalisables.

## 3.2 Reliability

- **strictNonRepaint**: valide sur bougies confirmées.
- **useCloseOnlyForBreaks**: cassure au close (plus strict).
- **bosBufferAtr**: marge ATR pour valider BOS/CHoCH.
- **breakCooldownBars**: évite les déclenchements répétés trop proches.
- **minFvgAtr**: taille mini d'un FVG (en ATR).
- **requireImpulseForFvg**: impose une impulsion/displacement pour valider FVG.

## 3.3 Structure / Liquidity / Context

- Longueurs pivots internes/swing.
- Nombre d'OB/FVG conservés.
- Mitigation OB par close ou extrêmes.
- Zones Premium / Discount / Equilibrium.
- Niveaux previous D/W/M.

## 3.4 Specialist

- **confluenceThreshold**: seuil pour setup.
- **useHtfBiasFilter**: coupe les setups contre biais HTF.
- **sessionBoost**: bonus en sessions London/NY.
- **useVolumeFilter**: validation volume.
- **SL cap**: `maxSlPips`, `useMaxSlCap`, `pipSize` / `useAutoPipSize`.
- **narrativeDetail**: `Court`, `Standard`, `Expert`.
- **useDynamicAlertsFR** + templates alertes FR.

---

## 4) Lecture des sorties

## 4.1 Dashboard (haut droite)

- **Bias**: biais swing + HTF.
- **Confluence + / -**: score bull/bear.
- **Grade**:
  - A = setup premium,
  - B = setup bon,
  - C = setup acceptable,
  - `-` = insuffisant.
- **Narrative (1/2) / (2/2)**: résumé contextuel.
- **No-Trade**: si actif, éviter l'entrée.
- **Journal**: dernier setup enregistré.

## 4.2 Signal table (bas droite)

- **Signal**: LONG / SHORT / WAIT.
- **PE**: point d'entrée.
- **SL**: stop loss (capé si activé).
- **TP1 / TP2**: objectifs.
- **RR**: ratio risque/rendement.
- **Grade / Filter / Score**: qualité et filtrage du setup.

---

## 5) Méthode d'utilisation (process simple)

1. Vérifier **Bias** + **HTF**.
2. Vérifier **No-Trade = OFF**.
3. Attendre `Signal = LONG/SHORT` avec **Grade A/B**.
4. Contrôler **RR** (idéalement >= 1.5 ou ton seuil).
5. Entrer uniquement si **SL/TP** sont cohérents avec ta gestion du risque.
6. Sortie progressive: TP1 puis TP2, ou gestion active.

---

## 6) Réglages recommandés par style

## Scalping (M1-M5)

- strictNonRepaint: ON
- useCloseOnlyForBreaks: ON
- minFvgAtr: 0.20 à 0.30
- breakCooldownBars: 4 à 6
- useVolumeFilter: ON
- confluenceThreshold: 68+
- Graph mode: Clean

## Intraday (M15-H1)

- strictNonRepaint: ON
- minFvgAtr: ~0.20
- breakCooldownBars: ~5
- confluenceThreshold: 65+
- HTF bias filter: ON

## Swing (H4-D1)

- strictNonRepaint: ON
- bosBufferAtr: 0.12 à 0.20
- minSignalRR: 2+
- confluenceThreshold: 70+
- sessionBoost: optionnel

---

## 7) Gestion du SL cap (important)

Si le SL semble trop petit/grand:

1. Tester `useAutoPipSize = OFF`.
2. Régler `pipSize` manuellement selon l'actif.
3. Ajuster `maxSlPips` (ex: 100 pips).

Exemple:
- sur XAUUSD, utiliser souvent `pipSize = 0.01`.
- sur Forex 5 décimales, `pipSize = 0.0001`.

---

## 8) Alertes

Tu as deux modes:

1. **alertcondition()** standard (A+, conflict, execution ready, grade A).
2. **Alertes FR dynamiques** (`useDynamicAlertsFR`) avec détails setup.

Conseil: utiliser les alertes dynamiques pour l'exécution, et les alertes standard pour le monitoring large.

---

## 9) Bonnes pratiques

- Ne pas trader contre ton plan de risque.
- Éviter les entrées en No-Trade.
- Prioriser les setups Grade A/B.
- Backtester visuellement par actif/TF avant usage réel.
- Conserver un journal externe des trades (capture + résultat).

---

## 10) Limites

- Un indicateur ne remplace pas la discipline de risk management.
- Les concepts SMC restent sensibles au contexte marché.
- Toujours valider en conditions réelles (spread, volatilité, session).

---

## 11) Checklist d'entrée (pratique)

- [ ] Bias HTF aligné
- [ ] No-Trade OFF
- [ ] Signal LONG/SHORT confirmé
- [ ] Grade A ou B
- [ ] RR acceptable
- [ ] SL cap et taille position cohérents
- [ ] Session favorable (si requis)

Si un point manque: **attendre**.
