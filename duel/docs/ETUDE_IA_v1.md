# Étude de l'intelligence artificielle de « Duel des Rois »
### Version 1 — étude préalable, aucun code d'IA modifié

## 0. Résumé exécutif

**Objet.** Avant de toucher au code, savoir ce que vaut chaque choix de jeu (Roi, bans, picks, placement, avancée, immobilisation, combos, anticipation), puis en déduire dix niveaux d'IA (Fer → Challenger), un système d'Elo et huit personnalités pour le mode non classé.

**Ce que l'étude établit (étiquettes : [M] mesuré, [R] déduit des règles, [H] hypothèse à tester).**

1. **Le jeu est un jeu de tempo.** La zone de mort (5×5 autour du Roi), le déplacement + attaque dans le même tour et la règle « deux pièces + 2 mana = létal » dominent tout. Un style purement défensif perd 72,7 % contre un style équilibré de même niveau (−170 Elo) [M].
2. **La draft vaut cher.** Une draft guidée par un modèle de composition (régression logistique sur Roi, unités, paires, contres) gagne **59–60 %** contre une draft aléatoire à niveau égal, soit environ **+65 Elo** [M]. L'IA actuelle drafte de façon statique : c'est son plus gros point faible.
3. **Le placement vaut moins, mais compte.** Placement guidé contre aléatoire : 53,4 % (+24 Elo) ; recherche de placement approfondie : 52,1 % de plus (+14 Elo, non significatif) [M]. Pour J2, voir le placement de J1 n'apporte presque rien aujourd'hui, faute de l'exploiter.
4. **La profondeur est le levier principal.** Profondeur 3 contre 2 : 59,8 % (+70 Elo par niveau de profondeur) [M]. Le budget de calcul permet 10 à 50 fois plus qu'aujourd'hui, mais exige un Web Worker et une recherche à budget de temps (≈ 1,5–2 s pour Challenger).
5. **Les valeurs d'unités dépendent du niveau de l'IA** (la Messagère passe de 40 % à 54 % de victoires, le Stratège de 33 % à 48 % entre une IA profondeur 2 et 3) : les tables de draft doivent être recalculées par niveau et validées avec des parties humaines.
6. **Au total, 400+ points de connaissance** sont consignés (G1–G30 anatomie, D1–D55 draft, P1–P29 placement, T1–T63 plateau, U1–U132 unités, C1–C40 combos, A1–A28 anticipation), chacun relié à un niveau d'IA qui le maîtrise.

**Décisions recommandées (à valider par vous).**

| Sujet | Recommandation |
|---|---|
| Échelle | 10 paliers de 100 Elo affichés (Fer 0–99 … Challenger 900+), IA ancrées au centre du palier (50, 150 … 950) |
| Force réelle des IA | Écarts réels **croissants** (≈ 64 % de victoire entre paliers voisins jusqu'à Émeraude, 70–85 % au-dessus), calibrés par tournoi |
| Gains/pertes | Formule Elo classique, K = 40 (10 premières parties), 24 ensuite, 16 au-dessus de 600 ; **gain réduit contre une IA bien plus faible, perte accrue** (voir §12) |
| Séries | Bo3 comptée comme **une** série : score attendu 3p² − 2p³ |
| Anti-farm | Gains plafonnés contre une IA de plus de 200 Elo sous le joueur |
| Non classé | Niveau + personnalité (8), chaque personnalité à ±30 Elo de la Complète |
| Mise en œuvre | Ne rien coder avant votre validation ; plan en 6 étapes au §13 |

## 1. Méthode, sources et limites

**Sources.**
- Les règles et les données du jeu (`SPEC.md`, `units.json`, `EFFETS.md`).
- Les études d'équilibrage antérieures du projet (`RESULTATS_v2`, `EQUILIBRAGE_v2`) : étiquette [A].
- Des **simulations nouvelles** : IA contre IA, 300 à 500 parties par duel, alternance des côtés (J1/J2), marge d'erreur ≈ ±3 points à 300 parties ; régression logistique pénalisée (ridge, C = 0,02) sur la composition des équipes, erreurs-types par bootstrap.
- Le raisonnement à partir des règles : étiquette [R].

**Convention.** [M] = mesuré dans cette étude ; [A] = mesuré dans une étude antérieure ; [R] = déduit des règles ; [H] = hypothèse non encore testée.

**Limites à garder en tête.**
1. **Toutes les mesures viennent d'IA qui jouent contre des IA.** Les valeurs d'unités et de combos reflètent donc ce que les bots savent exploiter. Un humain expert pourra jouer autrement : d'où le protocole de calibrage humain (§12.7).
2. Les écarts de moins de 3 points ne sont **pas** significatifs à 300 parties.
3. Les Elo en « points » sont des conversions de pourcentages de victoire (formule logistique, base 400) ; ils valent pour un même niveau de profondeur.
4. Le niveau réel d'un joueur humain ne se déduit pas de ces chiffres : seul le test de vos propres parties donne la cible (§12.7).

## 2. État des lieux des IA actuelles

**Les bots** (`src/ai/bots.ts`) : dix rangs R0–R9 ; bots L0–L3 (gloutons à bruit), S2/S3 (recherche alpha-bêta sur les tours, profondeur 2 ou 3), T2 (style tortue), P2 (prudent).

**Faiblesses relevées, dans l'ordre d'impact.**

| # | Faiblesse | Effet mesuré ou attendu |
|---|---|---|
| 1 | **Draft statique** : l'IA ne pondère ni contres ni synergies et ne s'adapte pas aux picks adverses | −65 Elo [M] |
| 2 | **Profondeur ≤ 3** à tous les niveaux élevés | chaque ply manquant ≈ −70 Elo [M] |
| 3 | **Placement glouton** sur 8 candidats seulement parmi 720 | −24 à −38 Elo [M] |
| 4 | **Information de J2 non utilisée** (placement de J1 vu mais ignoré) | −14 Elo, n.s. [M] |
| 5 | **Évaluation unique réglée à la main**, mêmes poids pour toutes les compositions | [H] +100 à +250 si apprise |
| 6 | **Bas niveaux = bruit sur le même moteur** : une IA « Fer » joue des coups forts puis un coup aberrant | joueur humain perçoit une IA incohérente |
| 7 | **Aucune anticipation de l'adversaire** (pas de modèle de ses coups probables, pas de profil de session) | [H] |
| 8 | **Écarts d'Elo irréguliers** entre rangs (80, 70, 135, 160, 80, 160, 125, 55, 85) | paliers non comparables ; Challenger trop proche de Grand Maître |
| 9 | **Roi choisi à l'aveugle sans analyse** | Roi ↔ unités, voir A.5–A.7 |

**Conséquence.** Un joueur qui a compris la zone de mort bat « Maître » assez facilement (votre constat) : la recherche est courte, la draft ignorante et l'écart avec les paliers inférieurs mal réglé.

## 3. Anatomie du jeu : les faits qui structurent tout

Étiquettes utilisées dans tout le document : **[M]** mesuré dans cette étude (simulations), **[A]** mesuré dans une étude précédente (RESULTATS_v2, ANALYSE_UNITES_v1), **[R]** déduit des règles (certain tant que les règles restent figées), **[H]** hypothèse à vérifier (par simulation ou avec de vrais joueurs).

### 3.1 La structure du jeu

- **G1 [R]** Plateau 8 rangées × 6 colonnes. Chaque camp place ses 5 pièces (Roi + 4 unités) sur sa rangée de fond, donc 5 colonnes sur 6 : une colonne reste vide. Pour une équipe donnée il existe **720 placements** (6 × 5 × 4 × 3 × 2).
- **G2 [R]** Seul le Roi a des points de vie (2). Une attaque de base sur une unité ne la tue pas : elle l'immobilise. Le jeu est donc une **course vers le Roi adverse** ; toutes les unités sont des outils de tempo, de contrôle ou de protection.
- **G3 [R]** Déplacement : 1 case **orthogonale** par pièce et par tour. Attaque de base : sur les 8 cases adjacentes. Fond à fond, il faut 6 déplacements pour qu'une unité soit adjacente au Roi adverse (7 rangées d'écart, on s'arrête à la rangée voisine du Roi). Sans bonus de vitesse, aucune attaque sur le Roi avant le tour 7 (c'est cohérent avec la moyenne mesurée : premier coup au Roi au tour 8,7 [M]).
- **G4 [R]** Il faut **deux coups** sur le Roi pour gagner (2 PV). Deux attaques de base coûtent 2 mana ; il faut deux pièces différentes adjacentes (chaque pièce n'attaque qu'une fois par tour), ou deux tours de suite.
- **G5 [R]** Le seul moyen de gagner sans toucher deux fois le Roi est la limite de tour (30 par joueur, puis mort subite +10) : c'est rare (7 parties sur 3 600 [M]). L'IA n'a donc pas besoin d'une stratégie de fin de partie longue, mais le joueur humain prudent peut vouloir la jouer.
- **G6 [M]** Une partie dure en moyenne **10,4 tours par joueur** (médiane 10, 10 % des parties sous 7 tours, 10 % au-dessus de 15). C'est une partie courte : une erreur d'ouverture se paie rarement à la fin, car il n'y a pas de fin.
- **G7 [M]** Le joueur qui commence (J1) gagne **56,9 %** des parties (3 592 parties S2 contre S2, placement guidé). Sur les parties courtes (moins de 7 tours) l'avantage disparaît (49 %) ; il monte à 58–60 % dès que la partie dépasse 14 tours. L'avantage de J1 est donc un **avantage de tempo qui se cumule** : le premier à jouer arrive au contact le premier.
- **G8 [M]** Le vainqueur finit avec 2 PV dans 77 % des cas, donc dans la grande majorité des parties le perdant n'a jamais touché le Roi adverse. Le jeu est **asymétrique dans le temps** : celui qui est en retard de tempo ne reçoit pas de seconde chance.

### 3.2 Le mana

- **G9 [R]** Régénération : +3 mana au début de chaque tour de son propriétaire. Le plafond (6, ou 7 avec le Roi Marchand) s'applique **à la fin du tour**, pas pendant : avec 6 de réserve on dispose de 9 mana dans un tour, puis on retombe à 6. Conséquence : **dépenser jusqu'à 3 mana par tour ne coûte rien** (c'est la régénération qui aurait été perdue), et tout mana non dépensé au-delà du plafond est gaspillé.
- **G10 [R]** Une attaque de base coûte 1, les compétences coûtent 0 à 3. Avant le contact (tours 1 à 5 environ), le mana s'accumule jusqu'au plafond : **dépenser du mana pendant la marche d'approche est gratuit** tant qu'on garde de quoi attaquer au contact. L'IA qui garde son mana « pour plus tard » perd de la valeur ; celle qui le dépense avant le contact fait un coup gratuit.
- **G11 [R]** J2 possède un **jeton de mana (+1 une fois)**. C'est une compensation mineure de l'avantage de J1.
- **G12 [R]** Le coût en mana limite le nombre de compétences coûteuses (3 mana) par tour : avec un budget de 6 en réserve + 3 de régénération, un tour « explosif » possible est de l'ordre de 9 mana (par exemple Liche à 3 + Arlequine à 3 + deux attaques à 1 = 8).

### 3.3 L'immobilisation

- **G13 [R]** Une unité immobilisée par l'adversaire à son tour N ne peut pas **se déplacer** pendant son propre tour suivant. Elle peut toujours **attaquer** (y compris le Roi) et, pour la plupart, utiliser ses compétences (seules quelques compétences de mobilité sont bloquées). Une unité immobilisée adjacente au Roi adverse reste donc dangereuse.
- **G14 [R]** Juste après son tour d'immobilisation, l'unité reçoit une **immunité d'un tour adverse** : elle ne peut pas être immobilisée deux fois de suite. La chaîne maximale de contrôle est donc « immobilisé un tour sur deux » : le contrôle ne fait jamais perdre plus de la moitié des déplacements d'une unité. [R]
- **G15 [R]** Une unité immobilisée **bloque la case qu'elle occupe** pour toutes les pièces, alliées comme ennemies (on ne traverse pas une pièce). Avancer en première ligne et se faire immobiliser dans un couloir étroit est donc **un blocage de passage gratuit**. C'est la raison pour laquelle l'avance reste rentable même quand on est immobilisé après (vérifié par simulation : terrain valorisé = parties plus courtes, mais pas de gain net sur J1 [A]).
- **G16 [R]** Le Roi Bastion annule la première immobilisation de chaque unité : l'intérêt est maximal contre les équipes de contrôle (Colosse, Geôlier, Trappeur, Liche) ; contre le Roi Liche, le bouclier absorbe la paralysie générale. [A] Bastion gagne 74 % contre Stratège et 47 % contre Liche.

### 3.4 La règle « Roi atteignable »

- **G17 [R]** On ne peut pas jouer un coup (déplacement, mur, piège) qui **fermerait tout chemin** vers votre Roi en vous retirant toutes les cases d'approche. Il est donc impossible de se barricader complètement ; un Roi en coin avec deux alliés collés et un mur ne peut pas être scellé.
- **G18 [R]** Conséquence tactique : un défenseur ne peut pas « gagner » en bloquant. Il doit contrôler le **tempo** : immobiliser, repousser (Minotaure), désarmer (Orque), déplacer (Mentaliste), protéger (Sanctuaire).
- **G19 [R]** Le moteur applique déjà cette règle dans la liste des coups légaux : aucune IA n'a à la « connaître », mais elle doit **anticiper que l'adversaire est tenu par elle** (par exemple, un Roi adverse déjà entouré ne peut pas être scellé : on peut compter sur ses cases d'approche).

### 3.5 Le terrain créé par les unités

- **G20 [R]** Le mur (Boréale) dure jusqu'au début du 2e tour suivant de son propriétaire et bloque passage et ligne de vue. Il sert à couper une ligne d'attaque à distance (Geôlier, Mentaliste, Automate) ou à ralentir un Félin.
- **G21 [R]** Les deux pièges (Trappeur) immobilisent toute pièce qui entre dessus, **alliés compris**. Un nouveau lancer retire les anciens. Ils arrêtent le Félin et le Minotaure en cours de course.
- **G22 [R]** Un mur ou un piège posé dans le couloir central est une taxe de tempo sur **toutes** les pièces qui doivent passer par là, y compris les siennes : le poseur doit connaître son propre chemin.

### 3.6 Information et ordre de jeu

- **G23 [R]** Les Rois sont choisis **à l'aveugle** (chacun ignore le Roi de l'autre au moment de son choix, y compris l'IA : `chooseKing` ne reçoit pas le Roi adverse), puis les deux Rois sont révélés avant la draft. L'information est ensuite complète pendant toute la partie, sauf l'équipe adverse non encore choisie pendant la draft. **Le choix du Roi est donc le premier pari** : il se fait sur la force moyenne du Roi (et sur ce qu'on attend de l'adversaire), pas sur un contre.
- **G24 [R]** Ordre de draft : bans A, B, A, B ; picks A, B, B, A ; bans A, B ; picks B, A, A, B. Le **dernier choix est pour J2** : J2 complète sa composition en voyant tout. J1 prend le **premier choix**. Entre la 1re et la 2e phase de bans, l'information augmente fortement (3 unités adverses sur 4 sont connues quand J1 fait le dernier ban avant ses deux derniers picks).
- **G25 [R]** Placement : J1 place d'abord, **J2 place en connaissant le placement de J1**. J2 peut donc choisir ses colonnes en fonction des colonnes d'attaque de J1 (Félin, Minotaure, Voltigeuse) et de la position de son Roi.
- **G26 [R]** Tout ce qui se joue avant le premier tour de plateau (Roi, 4 bans, 8 picks, placement) est une suite de choix qui fixe l'équilibre des forces. Une IA forte doit donc « jouer » ces phases avec autant de soin que les tours de plateau : c'est là que les joueurs humains réfléchis font la différence (§4 et §5).

### 3.7 Ce que la sim dit de la part de la draft dans le résultat

- **G27 [M]** Un modèle qui ne connaît **que les deux Rois et les 8 unités draftées** (aucune information de placement ni de jeu) prédit le vainqueur dans **64,8 % des parties** (validation croisée 5 plis, 3 592 parties), contre 56,9 % en devinant « J1 gagne ». Le résultat d'une partie est donc **en bonne partie scellé à la fin de la draft**.
- **G28 [M]** Avec ce modèle, la probabilité de victoire de J1 selon la composition va de **25 %** (5e percentile) à **85 %** (95e percentile). Les compositions du premier décile prédisent plus de 80 % ; les tests donnent bien 77 % réels pour les prédictions au-dessus de 70 %, et 33 % pour celles sous 40 %.
- **G29 [M]** L'essentiel de l'information vient de **la force individuelle** (le Roi et chaque unité) : ajouter les paires alliées, les contres et les croisements Roi × unité n'améliore pas la prédiction hors échantillon (64,8 % → 64,6 %). Les synergies et contres existent (certains à z > 3, voir §4) mais pèsent peu face à la force individuelle avec 3 600 parties ; leur importance réelle est **probablement plus élevée entre bons joueurs**, qui les exploitent volontairement. [H]
- **G30 [M]** À niveau de jeu égal, une draft pilotée par ce modèle gagne **59,6 %** contre une draft aléatoire (500 parties), soit environ **+70 Elo** apportés par la draft seule.

## 4. La draft (bans et picks)

### 4.1 Ce que mesure la simulation

Mesures : 3 592 parties S2 contre S2, Rois tirés au hasard, bans et picks au hasard, placement guidé pour les deux camps. Un modèle logistique régularisé (475 paramètres, validation croisée) sépare la part de chaque effet. Le détail chiffré est en annexe A ; les messages à retenir :

- **D1 [M]** La **force individuelle du Roi** est le premier facteur. Écart de victoires entre le meilleur et le pire Roi : Liche 59,4 % contre Stratège 33,4 %. Un Roi Stratège en J2 ne gagne que **24 %** des parties (S2) : tant que l'IA ne sait pas le jouer, ce Roi est un « handicap volontaire ». **Mais ce chiffre dépend du niveau de l'IA** (de 33 % à 48 % selon la valeur donnée au terrain [A]) : un humain qui sait faire avancer toute l'armée avec la marche forcée peut en faire un Roi fort. [H]
- **D2 [M]** Les unités se classent nettement : **Félin 57 %, Automate 56 %, Arlequine 55 %, Trappeur 54 %, Voltigeuse 53 %** en haut ; **Messagère 40 %, Catalyseur 44 %, Colosse 46 %, Furtif 46 %, Boréale 47 %** en bas. L'écart entre première et dernière est de 17 points : c'est considérable pour un jeu supposé équilibré. La cause probable : les unités faibles de l'IA sont celles dont l'intérêt est **tactique et indirect** (Messagère, Catalyseur, Furtif), que l'IA S2 exploite mal. [H]
- **D3 [M]** Pour un humain, ces écarts seront **différents** : [A] la Messagère passe de 40 % (S2) à 54 % (S3) et le Trappeur de 53 % à 34–44 % selon la façon dont l'IA valorise le terrain. **La table de valeurs de la draft doit donc être recalculée pour chaque niveau d'IA** (une IA de rang Diamant ne doit pas drafter comme une IA de rang Argent), et idéalement vérifiée sur de vraies parties humaines (§12).
- **D4 [M]** Composition par classes : les équipes avec **au moins un Assassin** gagnent davantage (1 Assassin + 1 Mage + 2 Tanks : 57 % ; 2 Assassins + 1 Mage + 1 Tank : 54 %), les équipes avec **3 Supports** ou **2 Mages + 2 Supports** s'effondrent (37 % et 39 %). Une équipe sans pression (aucun Assassin ni Tank à contact) perd le tempo. [M]
- **D5 [M]** Les **bonnes paires** observées (z > 3) : Automate + Catalyseur (+0,21), Arlequine + Félin (+0,20). Les autres effets de paires apparaissent entre 2 et 2,7 écarts-types : avec 475 paramètres testés, plusieurs sont du bruit. **Seules les paires z > 3 sont à considérer comme établies** ; les autres restent des hypothèses [H].
- **D6 [M]** Les **contres** les plus nets : Sanctuaire contre Trappeur (+0,18), Messagère contre Mentaliste (−0,17), Minotaure contre Guérisseur (+0,17). L'ordre de grandeur d'un contre est de **+0,15 à +0,25 log-odds, soit 4 à 6 points de victoire**. Un contre n'est donc pas une victoire garantie, mais plusieurs contres cumulés (4 unités × 4 unités = 16 duels) pèsent.
- **D7 [M]** Une **draft intelligente apporte +9,6 points** de victoire (59,6 % contre une draft aléatoire, à jeu égal). C'est l'équivalent d'environ **70 Elo** : à elle seule, la draft sépare deux paliers de classement (100 Elo). Pour les paliers Or et plus, la draft est un critère de classement à part entière.

### 4.2 Structure : qui décide quoi

- **D8 [R]** Pool de 17 unités ; 6 bans + 8 picks = 14 unités utilisées ; **3 unités ne sont jamais jouées**.
- **D9 [R]** **Bans 1 (A, B, A, B)** : on ne sait encore presque rien de l'adversaire (seulement son Roi). Ces bans sont « préventifs » : on retire une unité dont on a peur dans l'absolu ou face à **son Roi**.
- **D10 [R]** **Picks 1 (A, B, B, A)** : J1 prend le meilleur choix libre, J2 prend deux unités d'affilée (la meilleure réponse + la meilleure unité restante), puis J1 en prend un. J2 sort donc de cette phase avec 2 picks contre 2.
- **D11 [R]** **Bans 2 (A, B)** : on connaît 4 unités sur 8 ; c'est le moment des bans **précis** : retirer ce qui complète une synergie adverse ou contre sa composition.
- **D12 [R]** **Picks 2 (B, A, A, B)** : J2 ouvre et ferme, donc **le dernier choix est pour J2** : il complète en connaissant les 7 autres unités et les 6 bans. C'est la compensation principale du tempo de J1.
- **D13 [R]** L'information augmente à chaque étape : les décisions du début sont les plus risquées, celles de la fin sont presque une optimisation. Une IA de bon niveau doit **réserver de la flexibilité** pour les derniers picks (ne pas s'enfermer dans une identité avant d'avoir vu ce que l'adversaire a pris).
- **D14 [R]** J1 a un double avantage (tempo + premier pick). Il doit **prendre la valeur la plus sûre** (indépendante de l'équipe adverse) et laisser les contres pour la fin. J2 doit **prendre les réponses**.
- **D15 [R]** Chaque ban de J1 est un ban **sans information** sur la suite : ban de ce qui est fort pour tous les Rois. Chaque ban de J2 peut déjà s'appuyer sur 1 à 2 picks de J1.

### 4.3 Principes de ban

- **D16 [R]** Un ban n'a de valeur que si l'unité retirée **aurait été prise par l'adversaire** : bannir une unité que l'adversaire n'aime pas est un ban gâché. L'IA doit estimer ce que l'adversaire **prendrait** (son modèle) et non ce qu'elle-même prendrait.
- **D17 [M]** En l'absence d'information, les bans prioritaires sont les unités les plus fortes en absolu : **Félin, Automate, Arlequine, Trappeur, Voltigeuse**.
- **D18 [M]** Les rois ont des **points faibles par unité** : le Roi **Liche** gagne +0,25 de plus quand l'adversaire a l'**Automate** (+0,19 pour le Marchand) ; le Roi **Stratège** perd 0,20 de plus contre la **Voltigeuse**. Conséquences : **ne pas prendre** l'Automate face à un Liche ou à un Marchand ; **prendre** la Voltigeuse face à un Stratège ; et pour celui qui joue ces Rois, **bannir** l'unité qui le contre.
- **D19 [R]** Bannir l'unité qui **complète une synergie** adverse plutôt que l'unité la plus forte : si l'adversaire a Félin, bannir l'Arlequine (synergie) vaut mieux que bannir un 5e « top ».
- **D20 [R]** Bannir une unité dont **on a besoin pour un contre** que l'adversaire a déjà : si on pense à Sanctuaire contre son Trappeur, ne pas bannir Sanctuaire soi-même ; au contraire, **bannir Sanctuaire quand c'est l'adversaire qui compte dessus** pour contrer notre Trappeur.
- **D21 [R]** Les bans de J1 doivent éviter de révéler sa stratégie : bannir une unité de contrôle trahit une équipe fragile au contrôle.
- **D22 [R]** Les **bans de la phase 2** servent surtout à empêcher le « dernier pick parfait » de J2 : J1 doit bannir **l'unité qui ferme** la composition de J2 (la pièce manquante de son plan), et J2 doit bannir **l'unité que J1 voulait prendre à son dernier pick**.
- **D23 [R]** Il est souvent rentable de **bannir une unité moyenne qui est le contre unique** de sa propre unité clé (ex. Sanctuaire pour protéger son Trappeur) plutôt qu'une unité forte qui ne gêne pas.
- **D24 [R]** Un ban peut être un **piège** : bannir une unité que l'adversaire croit forte pour le pousser vers une alternative qu'on sait contrée. Réservé aux niveaux élevés.
- **D25 [H]** Face à un joueur dont on connaît les habitudes (ses unités favorites), bannir **sa signature** vaut un ban fort. Pertinent pour une IA « adaptative » (profil du joueur local).

### 4.4 Principes de pick

- **D26 [R]** Premier pick (J1) : unité **indépendante du Roi adverse**, forte quel que soit l'adversaire (Félin, Automate, Arlequine). Ne pas prendre un contre avant d'avoir vu les cibles.
- **D27 [R]** Les **deux picks d'affilée de J2** : (1) la meilleure réponse au premier pick de J1 ; (2) une unité qui forme une synergie avec (1) ou avec le Roi. C'est l'unique moment du jeu où l'on place deux unités avec la même information.
- **D28 [R]** Le **pick de transition** de J1 (4e pick de la phase 1) doit rester flexible : il sera suivi de deux bans puis de picks où J2 aura l'avantage de la dernière réponse.
- **D29 [R]** Pour J2, le **dernier pick** est celui qui vaut le plus : on n'a plus rien à craindre, on choisit la **pièce manquante** (couverture d'un risque : immobilisation de masse, vitesse, contre direct).
- **D30 [R]** Le pick doit viser **au moins une source de pression sur le Roi** (Félin, Voltigeuse, Minotaure, Arlequine, Automate). Une équipe sans pression ne gagne pas par le tempo.
- **D31 [M]** Éviter **trois Supports** ou deux Supports avec deux Mages (37–39 % de victoires). Un seul Support est le meilleur compromis (Sanctuaire, Prêtresse ou Guérisseur).
- **D32 [R]** Compléter les **trous de défense** : si l'équipe n'a aucune réponse à l'immobilisation (ni Sanctuaire, ni Prêtresse, ni Roi Bastion), un Roi Liche ou un Colosse adverse devient décisif.
- **D33 [R]** Compléter les **trous d'attaque** : si l'équipe est entièrement défensive, elle perd le tempo ; elle doit contenir deux unités capables d'arriver au contact vers le tour 6.
- **D34 [R]** Les **unités à compétence unique** (Guérisseur, Roi Liche, Roi Stratège) sont des cartes de timing : elles se choisissent quand on est capable de les placer au bon moment (compétence des niveaux élevés).
- **D35 [R]** Les unités à **recharge** (Orque, Automate, Mentaliste, Sanctuaire…) demandent une planification (un tour sur deux) : elles sont meilleures dans les équipes dont le plan est lent et régulier.
- **D36 [R]** Prendre une unité **pour la retirer à l'adversaire** (pick défensif) est légitime en fin de draft quand cette unité est un contre direct de sa propre équipe.
- **D37 [R]** Éviter les unités qui se **gênent entre elles** : Boréale + Geôlier (−0,11), Boréale + Automate (−0,15) (lignes de vue coupées par le mur), Sanctuaire + Orque (−0,15, deux unités à recharge qui occupent le même rôle défensif). [M, z ~2]

### 4.5 Roi et équipe

- **D38 [R]** Le **Roi** est choisi **à l'aveugle avant la draft** (choix simultané) puis conditionne tous les picks et les bans. Le choix du Roi est donc un jeu à information cachée : il faut le traiter comme une **stratégie mixte** (jouer plusieurs Rois avec des probabilités), pas comme un choix déterministe. Les 5 Rois sont des « classes » : Bastion (anti-contrôle), Liche (tempo offensif), Diplomate (flexibilité), Marchand (économie), Stratège (marche forcée).
- **D39 [M]** **Roi Liche** : fort en J1 (68 %), neutre en J2 (51 %) : son effet est un tempo (paralyser l'armée) ; l'équipe doit être capable de **finir** dans le même tour ou le suivant. Associer à des unités rapides (Félin, Minotaure, Voltigeuse).
- **D40 [M]** **Roi Bastion** : meilleur sur des équipes **qui avancent** (le bouclier annule une immobilisation, donc il neutralise les pièges, le Colosse, le Geôlier). Excellente synergie avec la Prêtresse (+0,15).
- **D41 [M]** **Roi Diplomate** : s'accorde bien avec le Catalyseur (+0,20) et l'Arlequine (+0,11), mal avec la Prêtresse (−0,16) : son échange de places libère déjà les unités coincées.
- **D42 [M]** **Roi Marchand** : 48 %, **−13 points en J2** (42 % contre 55 % en J1) : la réserve de mana est utile surtout pour un jeu de tempo où le contact est précoce ; en J2 il ne rattrape pas le retard.
- **D43 [M]** **Roi Stratège** : il faut qu'il **déclenche la marche forcée au tour de l'assaut** (tous les alliés, Roi compris, ont +1 déplacement), sinon il est trop lent. Évite Messagère (−0,14), Catalyseur (−0,13) : toutes les unités qui dépensent du mana pour de petits gains.
- **D44 [M]** **Roi contre Roi** : Bastion bat Stratège (+0,16), Liche bat Diplomate (+0,16), Diplomate bat Bastion (+0,15), Liche perd contre Bastion (−0,12). Triangle : Bastion → Liche → Diplomate → Bastion, avec le Stratège dominé par tous sauf le Marchand.
- **D45 [R]** Quand l'adversaire a le Roi **Liche**, le plan est : (1) avoir une réponse à la paralysie générale (Sanctuaire adjacent à plusieurs unités, Roi Bastion) ; (2) garder des unités **hors de portée du contact** à l'ouverture pour que la paralysie ne tombe pas sur un gain de tempo ; (3) ne pas avancer en bloc.
- **D46 [R]** Contre **Bastion** : ne pas compter sur le contrôle ; privilégier les **pièges** (qui comptent comme une immobilisation mais sont absorbés une fois) et les déplacements forcés (Automate, Minotaure, Mentaliste), qui ne sont pas des immobilisations.
- **D47 [R]** Contre **Marchand** : le temps joue pour lui (réserve plus grande) ; il faut un **tempo court** : unités rapides et deux attaques le plus tôt possible.
- **D48 [R]** Contre **Stratège** : le Stratège a un tour de marche forcée unique ; **défendre le tour d'après** (retarder, bloquer le couloir central, repousser) suffit.
- **D49 [R]** Contre **Diplomate** : ne pas se fier à la distance (il téléporte une unité par échange) ; garder une unité **proche de son Roi** capable de gêner l'échange.

### 4.6 Anti-patterns (erreurs fréquentes d'un bot ou d'un joueur)

- **D50 [R]** Tout prendre « pour la valeur » : une équipe de cinq bonnes unités sans cohérence (pas de défense contre l'immobilisation, pas de pression) perd contre une équipe moins forte mais complète.
- **D51 [R]** Bannir les mêmes unités quel que soit le Roi adverse.
- **D52 [R]** Réserver un pick pour une unité qui peut être bannie : si on **joue autour d'une unité unique** (Trappeur), un ban adverse casse le plan. Garder **deux plans**.
- **D53 [R]** Ignorer le **dernier ban de J2** : J1 qui prend son dernier pick sans réfléchir au ban final de J2 donne souvent sa pièce clé.
- **D54 [R]** Ne pas prendre en compte **le placement futur** : certaines unités exigent un placement précis (Félin en ligne ouverte, Minotaure avec une case derrière l'ennemi). La draft doit tenir compte des 720 placements.
- **D55 [R]** Négliger le **mana** : une équipe avec trois compétences à 3 manque de mana en phase de contact ; une équipe avec peu de compétences gaspille.

### 4.7 Algorithme de draft par niveau (cible de conception)

| Niveau | Roi | Bans | Picks |
|---|---|---|---|
| Fer | tiré au hasard (à l'aveugle) | au hasard | au hasard |
| Bronze | au hasard, évite le Stratège | 1 ban sur 2 au hasard, 1 sur la valeur de base | valeur de base bruitée |
| Argent | pondéré par la force du Roi | les 4 unités fortes de base | valeur de base, bruit modéré |
| Or | pondéré, connaît les contres de Roi | unités fortes, évite de bannir ce qui l'aide | valeur de base + 1 règle de contre |
| Platine | pondéré par la force des Rois | modèle additif (force + Roi) | modèle additif (Roi × unité) |
| Émeraude | pondéré + triangle Bastion → Liche → Diplomate | modèle complet (paires, contres) | modèle complet, anticipe un pas |
| Diamant | stratégie mixte (tirage pondéré, jamais prévisible) | ban du « meilleur pick adverse » via le modèle | modèle complet + flexibilité (ne ferme pas trop tôt) |
| Maître | stratégie mixte calculée sur la matrice Roi contre Roi | lecture de l'adversaire (ce qu'il prendrait) | recherche à 2 coups sur la draft |
| Grand Maître | stratégie mixte + habitudes du joueur (si profil connu) | Monte-Carlo des fins de draft | recherche + simulation de parties (placements inclus) |
| Challenger | équilibre de Nash approché sur la matrice Roi × draft, adapté au profil | arbre de draft complet, valeur apprise par auto-jeu | idem, valeur apprise par auto-jeu avec le moteur Challenger |

## 5. Le placement

### 5.1 Ce que mesure la simulation

Mesures : 1 996 parties S2 contre S2 avec **placement aléatoire des deux côtés** (pour voir l'effet du placement sans biais de sélection), plus deux duels : placement guidé contre aléatoire (400 parties) et J2 informé contre non informé (400 parties).

- **P1 [R]** Toutes les pièces démarrent sur leur rangée de fond, donc **le placement se réduit à l'ordre des colonnes** (720 possibilités par équipe, 1 colonne vide). Il n'y a ni profondeur ni formation : seul compte *qui est à côté de qui* et *face à qui*.
- **P2 [M]** **Le Roi au centre gagne nettement plus** : Roi sur une colonne de bord **45,6 %**, sur la 2e colonne **49,7 %**, sur une colonne centrale **54,7 %** (1 996 parties, placement aléatoire). L'écart bord–centre est de **+9 points**, c'est l'effet le plus important du placement mesuré. Cause probable : au bord, un Roi n'a que 5 cases voisines (l'adversaire concentre l'attaque, la règle « Roi atteignable » laisse moins de marge) ; au centre, il a plus de voies de dégagement et de défenseurs possibles. [H]
- **P3 [M]** **Le Roi isolé perd** : avec **aucun voisin** sur sa rangée (deux colonnes vides autour de lui ou bord + trou), 34,6 % de victoires (260 parties) ; un voisin 49,2 % ; deux voisins **53,4 %**. Un Roi entouré de deux alliés est mieux protégé et mieux relayé.
- **P4 [M]** **La colonne vide** (le trou) ne change rien : de 47,7 % à 53,5 % selon la colonne, sans tendance (bruit de ±2,5 points).
- **P5 [M]** **Placement par classe** : les Tanks sont à peine meilleurs sur le bord (51,3 % contre 48,1 % au centre), les Mages et les Assassins sont indifférents. Les écarts **unité par unité** (Colosse −7,8 au centre, Automate −6,2, Sanctuaire +4,9, Mentaliste +4,7) restent dans le bruit (ordre de ±4 points avec 300 parties par case) et **ne doivent pas être utilisés comme règles** [H].
- **P6 [M]** **Placement guidé contre aléatoire** : le placement actuel (8 candidats évalués par 3 mini-parties chacun) gagne **53,4 %** (400 parties), soit environ **+24 Elo**. C'est utile mais modeste : le placement guidé actuel est limité par la qualité de ses mini-parties (IA gloutonne) et par le petit nombre de candidats (8 sur 720).
- **P7 [M]** **J2 informé contre non informé** : voir le placement de J1 donne **52,1 %** (400 parties, écart non significatif). Cela veut dire que **l'IA actuelle n'exploite pas l'information de placement** (elle évalue 8 candidats au hasard contre un seul adversaire fixe). Or c'est un avantage structurel de J2 (§3, G25) : un humain bon en tire profit.
- **P8 [H]** Avec les Rois sur la **même colonne**, J1 ne gagne que 50,9 % (318 parties), contre 59 % quand l'écart de colonnes est de 2 ou 3. Un placement en miroir annule peut-être le tempo de J1. À confirmer.

### 5.2 Principes de placement

#### Le Roi

- **P9 [M]** Roi au centre (colonnes 2 ou 3 ; la 2e colonne est acceptable). Éviter les colonnes de bord sauf plan défensif précis (Roi Bastion avec Sanctuaire, par exemple).
- **P10 [M]** Garder **au moins un voisin** sur la rangée, idéalement deux (un Tank et un Support).
- **P11 [R]** Le Roi central est exposé par les deux côtés : il faut une **unité de contrôle à côté** (Colosse, Orque, Sanctuaire) pour gêner l'approche des unités adverses.
- **P12 [R]** Le Roi **en coin** (colonne 0 ou 5) n'a qu'une approche latérale : il est plus facile à protéger d'un côté, mais l'adversaire peut y concentrer toutes ses attaques ; mauvais si l'adversaire possède le Roi Liche (paralysie puis assaut en bloc) ou une unité de saut (Voltigeuse, Arlequine).
- **P13 [R]** Face à un adversaire dont le Roi est au centre, une attaque directe en trois colonnes (Félin, Voltigeuse, Minotaure) **fait face** à ce Roi : placer ses attaquants **dans la colonne du Roi adverse ou les deux voisines** pour limiter le déplacement latéral (chaque déplacement latéral = un tour).

#### Les attaquants

- **P14 [R]** **Félin** : placer dans une colonne **sans pièce adverse en face** et sans piège possible ; il aura deux cases d'avance tous les 3 tours. Le plus rentable est le Félin en colonne **centrale** (il tape directement le Roi adverse).
- **P15 [R]** **Minotaure** : placer face à l'unité adverse la plus avancée prévisible (Félin, Sprinteur) : il peut la repousser et prendre sa place, ce qui gagne un tempo.
- **P16 [R]** **Voltigeuse** : saute orthogonalement ; elle a besoin d'une **pièce à enjamber** et d'une case libre derrière. En début de partie, elle saute par-dessus sa **propre voisine** (l'effet est faible) ; elle est surtout utile quand les pièces adverses arrivent au contact. Placer **derrière le front** (colonne de bord proche de la zone de combat) pour entrer par le flanc.
- **P17 [R]** **Arlequine** : échange avec une unité adjacente (8 cases) ; le meilleur placement est **à côté de l'unité qu'elle doit propulser** (Félin, Minotaure) : un échange vaut un déplacement de 2 cases sans coût de mouvement.
- **P18 [R]** **Automate** : portée 5×5 ; une fois lancé, il attire les ennemis situés à 2 cases. Le placer **au centre** pour couvrir les deux côtés ; son Roi derrière protège le dispositif.

#### Les contrôleurs et les supports

- **P19 [R]** **Colosse** : immobilise les ennemis adjacents ; à placer **à côté du Roi** : il joue le rôle d'un blindage dès que l'ennemi arrive.
- **P20 [R]** **Orque** : désarme les adjacents ; excellent en deuxième voisin du Roi (tour d'arrivée de l'attaquant).
- **P21 [R]** **Sanctuaire** : protège lui-même et ses **alliés adjacents** ; il rapporte d'autant plus qu'il a deux voisins (placer entre le Roi et un attaquant clé : le Roi protégé + un attaquant protégé).
- **P22 [R]** **Trappeur, Boréale, Geôlier, Mentaliste** (portée 2) : à l'arrière du front (colonne centrale) pour atteindre les deux flancs avec une portée carrée 5×5.
- **P23 [R]** **Guérisseur, Prêtresse, Messagère, Catalyseur** : leur compétence est à portée **globale** (Guérisseur, Prêtresse, Messagère, Catalyseur) : leur placement importe peu pour la compétence, mais comme ils n'attaquent pas bien, on les met **sur le bord** pour libérer le centre aux unités de combat.
- **P24 [R]** **Furtif** : placer face à une **colonne de départ de l'ennemi** : l'intangibilité d'un tour lui sert à passer par un couloir gardé.

#### Face à l'adversaire (J2 surtout)

- **P25 [R]** J2 voit le placement de J1 : il doit **choisir sa colonne de Roi** en fonction de la colonne du Félin, du Minotaure et de la Voltigeuse adverses (et les placer **en face**).
- **P26 [R]** Si J1 place son Roi sur un bord, J2 peut **concentrer** ses attaquants sur cette colonne.
- **P27 [R]** Si J1 a un Colosse à côté de son Roi, J2 évite de faire arriver plusieurs unités dans la même case voisine au même tour (le Colosse les immobiliserait toutes).
- **P28 [R]** J1, qui ne peut pas répondre, doit placer **robuste** : Roi central avec deux voisins, attaquants face au centre, aucun point faible exploitable par un seul placement de J2.
- **P29 [R]** Pour J1, **l'incertitude coûte** : il vaut mieux deux placements équivalents que le placement « optimal contre un adversaire aléatoire ».

### 5.3 Placement par niveau d'IA

| Niveau | Méthode |
|---|---|
| Fer | totalement aléatoire (Roi possible sur le bord, isolé) |
| Bronze | aléatoire, mais jamais de Roi isolé (règle P10) |
| Argent | Roi central + un voisin (règles P9, P10), reste aléatoire |
| Or | placement guidé actuel (8 candidats) + règle du Roi central |
| Platine | 24 candidats filtrés par règles (P9–P24), 6 mini-parties chacun |
| Émeraude | idem + **J2 exploite le placement de J1** (candidats évalués contre le placement réel) |
| Diamant | 60 candidats filtrés, mini-parties jouées avec l'IA Argent (et non gloutonne) |
| Maître | recherche de placement par évaluation à 2 coups (positions de départ) |
| Grand Maître | recherche d'équilibre : J1 choisit le placement qui résiste à la meilleure réponse de J2 |
| Challenger | recherche complète (tous les placements filtrés, valeur apprise) + J2 optimise explicitement la réponse |

## 6. Le jeu sur le plateau

### 6.1 La « zone de mort » : le concept central

- **T1 [R]** Une pièce adjacente au Roi adverse peut le frapper. Une pièce qui peut **se déplacer d'une case orthogonale** pour devenir adjacente peut le frapper **dans le même tour** (se déplacer, puis attaquer : rien n'interdit les deux). La menace sur un Roi couvre donc **toutes les cases à distance 2 (carré 5×5 autour du Roi)** qu'une pièce peut rejoindre en un pas.
- **T2 [R]** **Deux pièces dans la zone de mort au début du tour adverse, plus 2 mana, = Roi mort** (2 PV, 1 mana par attaque). C'est la condition de victoire en un tour. Toute la défense se résume à : *à la fin de mon tour, il ne doit pas y avoir deux pièces adverses capables d'entrer dans le carré 5×5 de mon Roi avec 2 mana disponibles*.
- **T3 [R]** Le **Roi peut aussi frapper** (il se déplace et attaque comme les autres pièces) : le Roi adverse peut compter comme l'une des deux pièces. Un Roi qui s'avance devient une arme (la règle « Roi atteignable » encadre les coups qui le scelleraient).
- **T4 [R]** Une pièce **immobilisée** qui est déjà adjacente au Roi peut encore le frapper (l'immobilisation retire le déplacement, pas l'attaque). Immobiliser une pièce **à distance 2** l'empêche d'entrer dans la zone : c'est une défense. Immobiliser une pièce **déjà adjacente** ne la neutralise pas : il faut la **désarmer** (Orque), la **déplacer** (Mentaliste, Minotaure, Automate) ou la rendre **non attaquante** autrement.
- **T5 [R]** Conséquences pour les unités de contrôle : **Geôlier, Trappeur, Boréale** agissent à portée 2 : elles interviennent **avant** l'entrée dans la zone de mort (défense à distance) ; **Colosse** n'agit qu'au contact : il ne neutralise pas une attaque déjà au contact. **Orque** désarme les adjacents, ce qui est la parade au contact.
- **T6 [R]** Offensivement, la zone de mort est aussi l'objectif : faire entrer **deux pièces dans la zone à un tour où l'adversaire ne peut plus les neutraliser**. Le Roi Liche (paralysie générale) donne un tour sans mouvement adverse, mais **pas un tour sans attaques** : il vaut surtout pour empêcher les défenseurs d'accourir.
- **T7 [R]** Si l'adversaire a **deux unités de contrôle à portée** et que je n'amène qu'une pièce, la pièce est immobilisée et l'attaque s'effondre. L'assaut doit amener **plus de pièces que l'adversaire ne peut neutraliser en un tour** (3 pièces pour 2 contrôles).

### 6.2 Tempo et course

- **T8 [R]** Distance nécessaire : 6 déplacements pour une unité qui part de sa rangée de fond jusqu'à la zone adjacente du Roi adverse en ligne droite (5 pour entrer dans la zone de mort). Avec le Félin (2 cases tous les 3 tours) : 4 à 5 tours. Premier coup possible au tour 6–7 ; en pratique 8,7 [M].
- **T9 [R]** J1 arrive un tour avant J2 : **s'il n'est pas retardé, J1 tue le premier**. J2 doit donc **gagner du temps** (retarder, immobiliser, repousser) ou **gagner de la distance** (le Roi de J2 ne doit pas être le plus facile à atteindre).
- **T10 [M]** Les parties longues favorisent J1 (58–60 % au-delà de 14 tours) : quand rien ne se passe, c'est J1 qui a le tempo.
- **T11 [R]** Le Minotaure gagne **un tempo net** (il avance de 1-2 cases et repousse l'ennemi d'une case : il recule l'ennemi de 1 et avance de 1 = écart de 2 cases).
- **T12 [R]** L'Automate fait gagner **1 case** (attire un ennemi vers lui), l'Arlequine **jusqu'à 2 cases** (échange avec une unité adjacente en diagonale ou ligne), la Voltigeuse **2 cases** (saute une pièce) mais ne peut pas attaquer ce tour-là.
- **T13 [R]** La Messagère donne **+1 déplacement** à n'importe quel allié, où qu'il soit : c'est un tempo **ciblé**. Le meilleur usage est le tour d'entrée dans la zone de mort (le dernier pas).
- **T14 [R]** Le Roi Stratège donne +1 déplacement à **tous** (Roi compris), une fois : c'est le tour de l'assaut. Avant ce tour, l'armée doit être **à 2 rangées de la zone** (le pas supplémentaire permet d'entrer et d'attaquer dans le même tour).
- **T15 [R]** Un **recul** (se mettre à distance) coûte du tempo : il n'a de valeur que s'il force l'adversaire à entrer dans votre zone de contrôle.

### 6.3 Avancer, se faire immobiliser, bloquer

- **T16 [R]** Avancer est **presque toujours rentable** : chaque case gagnée rapproche de la zone de mort adverse et réduit la profondeur de la défense. Une pièce avancée et immobilisée **garde** sa case : elle bloque, menace et reste dans la zone de mort si elle l'a atteinte.
- **T17 [R]** Une pièce immobilisée est un **obstacle** pour les deux camps : elle bouche une case. Dans un couloir de 6 colonnes avec 4 à 5 pièces par camp, une pièce immobilisée au milieu change le chemin de toutes les autres.
- **T18 [R]** **Prendre une immobilisation pour bloquer** : c'est rentable quand (a) la case occupée coupe un couloir de passage de l'adversaire, (b) la pièce n'a plus de mouvement utile de toute façon, (c) elle est protégée par une immunité qui suit (G14), qui l'empêche d'être immobilisée de nouveau tout de suite.
- **T19 [R]** L'immobilisation **ne coûte donc pas un tour** quand la pièce serait de toute façon arrêtée par un obstacle ou par un coup de l'adversaire. Elle coûte un tour de déplacement quand la pièce était en train de courir.
- **T20 [A]** Une valorisation du terrain dans l'IA (bonus par rangée avancée et malus d'immobilisation réduit) **raccourcit les parties** (de 10,2 à 8,2–9,0 tours) mais **ne change pas le taux de victoire de J1** (56 → 57–58 %). Avancer est bon pour les deux camps : la partie devient une course.
- **T21 [R]** Le premier à **entrer dans le territoire adverse** (rangées 5–8 pour J1) transforme le terrain en contrainte : la règle « Roi atteignable » force le défenseur à garder une approche ouverte.
- **T22 [R]** **Ne pas faire le premier pas** n'est pas forcément mauvais : si l'adversaire entre dans votre portée de contrôle (distance ≤ 2 d'un Geôlier, d'un Trappeur, d'un Colosse), c'est lui qui est immobilisé en premier. Il y a une **zone de contact** (rangées 3–4) où celui qui entre d'abord est contrôlé d'abord.
- **T23 [R]** Entrer dans la zone de contrôle adverse est correct si on y entre avec **plus de pièces que l'adversaire ne peut en contrôler** ou avec une pièce qui est **déjà protégée** (Sanctuaire adjacent, immunité post-immobilisation, Furtif, Bastion).
- **T24 [R]** Le **bouclier du Roi Bastion** transforme une pièce qui entre en première ligne en pièce **immune au premier contrôle** : le premier à entrer est toujours Bastion.

### 6.4 Défense

- **T25 [R]** Surveiller en priorité **la distance de l'ennemi le plus avancé à la zone de mort** (carré 5×5 du Roi) et **le nombre de pièces à distance ≤ 3**.
- **T26 [R]** À la fin de mon tour : au plus **une** pièce adverse capable d'entrer dans la zone. Si deux : les neutraliser (immobiliser à distance 2, désarmer au contact, repousser, déplacer).
- **T27 [R]** Les neutralisations se classent : **déplacement** (Mentaliste, Minotaure, Automate) > **désarmement** (Orque) > **immobilisation à distance 2** > **immobilisation au contact** (inefficace pour empêcher l'attaque).
- **T28 [R]** Le **Guérisseur** (+1 PV du Roi, usage unique) transforme « 2 coups » en « 3 coups » : il se joue **dès qu'un Roi a été touché** (pas avant) pour ne pas perdre l'avantage de tempo, mais jamais **après un coup mortel** (impossible).
- **T29 [R]** Défendre c'est aussi **déplacer le Roi** : un pas latéral peut sortir une pièce de portée ; mais le Roi qui bouge quitte son entourage (P3 : le Roi isolé perd).
- **T30 [R]** **Anti-Liche** : avant la paralysie générale, éviter d'avoir deux pièces alignées avec le Sanctuaire adjacent à plusieurs (le Sanctuaire protège de l'immobilisation, mais il a une recharge d'un tour sur deux : il doit être **prêt** au tour du Liche).
- **T31 [R]** **Anti-Félin** : bloquer la colonne par une pièce quelconque (même immobilisée) ou poser un piège/mur : le Félin ne traverse pas les pièces.
- **T32 [R]** **Anti-Arlequine/Voltigeuse** : ne pas laisser de pièce adjacente à l'unité qui saute ; l'Arlequine échange avec une pièce **adjacente** (la sienne ou la vôtre), la Voltigeuse saute sur une pièce adjacente en ligne droite vers une case libre : une case **occupée** derrière elle l'arrête.
- **T33 [R]** **Anti-Automate** : garder ses unités **à distance 3** de l'Automate quand son attraction est prête (recharge 1 tour sur 2 : il est prêt un tour sur deux, donc on peut prévoir).
- **T34 [R]** **Anti-Mentaliste** : l'effet déplace une cible à portée 2 en ligne vers une case adjacente à elle, puis l'immobilise (peut tirer un Roi dans la zone de combat) ; protéger les pièces clés par Sanctuaire/Furtif ou rester hors ligne droite.
- **T35 [R]** **Anti-Trappeur** : un piège immobilise toute pièce qui entre dessus, même une pièce alliée ; **repérer les deux cases piégées** avant d'avancer, passer par une autre case ou y envoyer une pièce « sacrifiable » (une pièce déjà inutile).

### 6.5 Attaque

- **T36 [R]** Un assaut = **trois pièces** (deux attaquantes, une réserve) arrivant **le même tour** dans la zone de mort, avec 2 mana de côté.
- **T37 [R]** **Ordre des actions dans le tour** : (1) contrôler les défenseurs (compétences), (2) déplacer, (3) attaquer le Roi, (4) dépenser le reste du mana. Le contrôle se fait **avant** de bouger quand il peut empêcher une réponse ; il se fait **après** quand il peut être annulé par un déplacement forcé de la cible.
- **T38 [R]** Attaquer le Roi **en premier** (avant d'attaquer une unité de défense) quand le mana est limité : chaque coup de Roi sert, chaque immobilisation d'unité ne sert que si elle empêche une riposte.
- **T39 [R]** **Tuer en un tour** n'est pas toujours nécessaire : un premier coup (1 PV) donne la pression : le Roi adverse (1 PV) doit tout sacrifier pour ne pas être touché une seconde fois, ce qui fige sa défense (le Guérisseur peut l'annuler).
- **T40 [R]** **Double menace** : deux attaquants dans la zone, un seul contrôle adverse disponible = victoire ; l'IA doit rechercher **les doubles menaces** plutôt que « la plus grosse menace ».
- **T41 [R]** **Menaces simultanées sur deux fronts** (Félin sur la colonne centrale + Voltigeuse sur le flanc) : un défenseur qui n'a qu'un contrôle par tour ne peut pas tout parer.
- **T42 [R]** **Sacrifice de tempo** : envoyer une pièce **au piège** ou dans une zone de contrôle pour que l'adversaire dépense du mana et un tour à la neutraliser peut être rentable : **chaque pièce de contrôle dépensée est un contrôle de moins** pour la pièce suivante.
- **T43 [R]** Le **mana de l'adversaire** est une ressource attaquable : quand il a 6 et que ses compétences coûtent 3, on peut **le forcer à dépenser** en menaçant deux choses. S'il a dépensé, il ne peut pas riposter (« menace létale » du moteur d'évaluation).

### 6.6 Mana

- **T44 [R]** **Budget standard** : 3 mana par tour sont gratuits (G9). Un plan qui exige plus de 3 mana par tour doit s'appuyer sur une réserve.
- **T45 [R]** Avant le contact, **dépenser** : poser des pièges/murs/lancer une compétence à 2 coûte zéro tant que la réserve reste à 6 en fin de tour (les compétences d'« installation » : Trappeur, Boréale, Sanctuaire, Catalyseur).
- **T46 [R]** Au contact, **chaque mana compte** : 2 mana pour tuer (G4). Une compétence à 3 avant le coup mortel n'est correcte que s'il reste 2 mana après.
- **T47 [R]** Le Catalyseur (+2 mana pour 1, soit +1 net, recharge 1 tour sur 2) ne vaut que quand le plafond n'est pas atteint : avant le contact, c'est du mana perdu ; **au contact**, c'est le coup supplémentaire qui fait la différence.
- **T48 [R]** Le Roi Marchand (+4 un tour sur deux, réserve 7) n'est utile que pour des **tours explosifs** (Liche + Arlequine + deux attaques = 8 mana).
- **T49 [R]** Jeton de J2 : 1 mana de plus **une fois** ; à garder pour le tour décisif (jamais le gaspiller).

### 6.7 Terrain : murs et pièges

- **T50 [R]** Un mur dure **jusqu'au début du 2e tour suivant** : il couvre donc le tour adverse en entier, et disparaît au début de mon deuxième tour : **un mur a une demi-vie de défense**.
- **T51 [R]** Un mur **près de son propre Roi** réduit les cases d'approche (et la règle « Roi atteignable » le laisse faire tant qu'un chemin reste).
- **T52 [R]** Un mur peut couper la **ligne de vue** du Geôlier/Mentaliste/Automate (portée en ligne) : un mur bien placé neutralise une pièce de contrôle.
- **T53 [R]** **Deux pièges** coupent un couloir : un piège est immobilisant **une fois** par pièce (par la suite, la pièce est immunisée un tour). Les piéger plusieurs fois exige de relancer.
- **T54 [R]** Un piège sur **une case que mes propres pièces doivent traverser** est une erreur : le poseur doit tenir compte de ses propres chemins.
- **T55 [R]** Poser un piège **adjacent au Roi adverse** n'a pas de sens (la portée du Trappeur couvre 2) ; poser en **couloir central** à distance 2–3 devant ses unités est plus efficace.

### 6.8 Roi

- **T56 [R]** Le Roi n'est pas passif : il **avance** (rarement), **frappe** (en cas de besoin) et **attire** la défense. Un Roi avancé sans soutien meurt.
- **T57 [R]** Le **Roi adverse peut être déplacé** (Mentaliste, Automate, Minotaure, Voltigeuse peut le sauter). Les compétences qui déplacent le Roi sont des **armes de victoire** : un Roi tiré dans votre zone est touché le tour même.
- **T58 [R]** Le **Roi peut être immobilisé** (Geôlier, Colosse, Trappeur, Mentaliste, attaque) : un Roi immobilisé ne peut pas fuir (mais peut frapper). Un Roi immobilisé **ne peut pas défendre en s'écartant**.
- **T59 [R]** Le Sanctuaire adjacent au Roi le protège **de l'immobilisation et du déplacement** pendant le tour adverse : c'est la défense contre l'Automate/Mentaliste/Minotaure.
- **T60 [R]** L'Insaisissable (Furtif) ne peut pas être ciblé : c'est un **pion invulnérable** un tour sur deux ; il peut entrer dans une zone de contrôle sans en subir les effets.

### 6.9 Fin de partie

- **T61 [R]** La limite de 30 tours est un **tie-break sur les PV du Roi** : avec un coup d'avance au compteur de PV, un joueur peut chercher le blocage total (mais la règle d'atteignabilité l'interdit, §3).
- **T62 [R]** Une position de **blocage mutuel** (aucun Roi touché) se résout par la mort subite (le premier coup tue) : **le joueur qui contrôle le tempo gagne**. Une IA qui joue la mort subite doit **calculer exactement** (la profondeur de recherche suffit).
- **T63 [R]** À 1 PV, le Roi touché doit **tout** sacrifier à la défense : l'IA qui a touché doit « enchaîner » (maintenir deux pièces dans la zone).

## 7. Fiches d'unités : ce que chaque IA doit savoir

Pour chaque unité : **rôle**, **bon usage**, **mauvais usage**, **combos**, **craint**, **draft**. « Victoires » = taux mesuré en S2 contre S2 (3 592 parties, drafts aléatoires) ; il dépend du niveau de l'IA (§4, D3). Numérotation : U + numéro.

### 7.1 Les cinq Rois

**Roi Bastion** (victoires 55,7 %, 61,7 % en J1) — passif : les unités ignorent leur première immobilisation.
- **U1 Rôle** : roi d'avance : ses unités entrent en première ligne sans craindre le premier contrôle.
- **U2 Bon usage** : avancer en bloc ; passer les pièges et les zones de contrôle avec une unité « éclaireur » (le bouclier est consommé, la pièce continue).
- **U3 Mauvais usage** : rester passif (le passif ne sert pas) ; croire le bouclier permanent (une seule fois par unité).
- **U4 Combos** : Prêtresse (+0,15) ; unités à contact (Minotaure, Félin).
- **U5 Craint** : Roi Diplomate (−0,15), unités de déplacement (Automate, Mentaliste, Minotaure) qui **ne sont pas des immobilisations**.
- **U6 Draft** : bannir les unités de déplacement adverses ; prendre des unités à contact.

**Roi Liche** (59,4 % ; 67,9 % en J1, 50,6 % en J2) — usage unique, coût 3 : immobilise tous les ennemis (hors Roi).
- **U7 Rôle** : tempo décisif : un tour où l'armée adverse ne peut pas bouger.
- **U8 Bon usage** : **au tour où deux de mes pièces entrent dans la zone de mort** et où l'adversaire ne peut plus répondre en déplaçant ; ou au tour où l'adversaire s'apprête à entrer dans mon carré 5×5 (paralysie défensive).
- **U9 Mauvais usage** : trop tôt (les pièces adverses encore loin n'avaient pas besoin d'être arrêtées), ou sans avoir **les 2 mana** pour finir ensuite (il coûte 3).
- **U10 Combos** : Félin, Minotaure, Trappeur (+0,09 [M], z = 1,8), tout ce qui frappe dans le même tour que la paralysie.
- **U11 Craint** : Sanctuaire adjacent (annule l'immobilisation), Roi Bastion (−0,12 : boucliers), Prêtresse (libère une pièce).
- **U12 Draft** : prendre des unités rapides ; bannir Sanctuaire adverse si on joue Liche.

**Roi Diplomate** (53,3 %) — coût 3, répétable : échange la place de deux alliés (hors Roi ; pas une unité qui a déjà attaqué).
- **U13 Rôle** : roi de flexibilité : téléporte une unité à la place d'une autre (jusqu'à 7 cases de saut).
- **U14 Bon usage** : amener un attaquant **directement dans la zone de mort** en l'échangeant avec une pièce avancée ; sauver une pièce coincée ; défendre en ramenant une pièce près du Roi.
- **U15 Mauvais usage** : l'utiliser à chaque tour (3 mana par échange) ; échanger sans plan.
- **U16 Combos** : Catalyseur (+0,20), Arlequine (+0,11), pièces avancées immobilisées (l'échange les libère).
- **U17 Craint** : Roi Liche (−0,16 : la paralysie gèle les échanges de l'adversaire). **Bat** le Roi Bastion (+0,15 [M]).
- **U18 Draft** : unités à attaque en une fois (Félin, Voltigeuse) pour profiter de la téléportation.

**Roi Marchand** (48,4 % ; 55 % en J1, 42 % en J2) — passif : réserve 7, +4 mana un tour sur deux.
- **U19 Rôle** : économie : permet des tours à 8-9 mana.
- **U20 Bon usage** : accumuler (la réserve de 7 se remplit) pour **un tour explosif** : deux compétences à 3 + deux attaques.
- **U21 Mauvais usage** : jouer comme un Roi normal (les 4 mana du tour pair sont perdus).
- **U22 Combos** : équipes à compétences coûteuses (Minotaure, Arlequine, Voltigeuse, Automate : toutes coût 3).
- **U23 Craint** : Automate (−0,19 : Marchand contre Automate), tempo court.
- **U24 Draft** : à éviter en J2 face à un bon J1 ; à choisir (Roi) quand on connaît ses unités à 3 coûts.

**Roi Stratège** (33,4 % ; 24 % en J2) — usage unique, coût 2 : tous les alliés, Roi compris, +1 déplacement ce tour.
- **U25 Rôle** : marche forcée : tour d'assaut coordonné. Mesuré faible contre l'IA actuelle [M], **très dépendant du jeu** [A, 33 % → 48 %].
- **U26 Bon usage** : l'armée avance **en bloc à 2 rangées** de la zone de mort ; le tour de la marche forcée, tous entrent en même temps et **frappent** (Roi compris).
- **U27 Mauvais usage** : le jouer tôt (les pièces avancent trop vite et sont immobilisées) ou tard (l'avance est perdue).
- **U28 Combos** : Boréale (écran), Orque ; unités qui **ne** dépensent **pas** de mana avant l'assaut (le Stratège coûte 2).
- **U29 Craint** : Voltigeuse (−0,20), Guérisseur (−0,13), Geôlier (−0,13).
- **U30 Draft** : **le prendre seulement avec un plan précis** ; l'IA ne doit le choisir que si son niveau sait coordonner un assaut (à partir de Diamant).

### 7.2 Les Tanks

**Colosse** (45,9 %) — coût 2 : immobilise **tous** les ennemis adjacents.
- **U31 Rôle** : verrou de mêlée ; **protège** le Roi quand plusieurs ennemis sont adjacents.
- **U32 Bon usage** : quand **au moins 2 ennemis** sont adjacents (rentable : 2 mana contre 2 attaques de base = 2 mana pour le même résultat sur 2 unités, mais une seule action).
- **U33 Mauvais usage** : l'utiliser quand **les ennemis adjacents peuvent encore attaquer le Roi** (immobilisation n'empêche pas l'attaque : T4).
- **U34 Combos** : Automate (attire les ennemis dans la case adjacente du Colosse) ; Boréale (les murs obligent les ennemis à passer par son voisinage).
- **U35 Craint** : Minotaure (+0,16 contre lui), Sanctuaire (annule l'immobilisation), Furtif, Bastion adverse.
- **U36 Draft** : unité plutôt faible en S2 (46 %) ; utile contre les équipes **sans** Sanctuaire ni Bastion.

**Minotaure** (52,5 %) — coût 3 : fonce sur un ennemi adjacent (ou à 2 en ligne), le repousse d'une case et prend sa place.
- **U37 Rôle** : tempo et défense : avance et recule l'ennemi de 2 cases d'écart ; peut **repousser un Roi** hors de sa case.
- **U38 Bon usage** : repousser une pièce adjacente au Roi ; prendre la place d'une pièce en ligne pour entrer dans la zone de mort ; repousser une pièce sur un **piège** (une pièce repoussée sur un piège est immobilisée).
- **U39 Mauvais usage** : charger quand la case derrière la cible est occupée (impossible) ou quand la case de départ ne vaut rien.
- **U40 Combos** : Trappeur (pousser l'ennemi sur un piège), Colosse (même zone), Guérisseur (+0,17 contre lui : le Minotaure **contre** le Guérisseur).
- **U41 Craint** : Arlequine (−0,13), Sanctuaire (ne peut pas être déplacé), Furtif.
- **U42 Draft** : bonne unité pour J2 (défense active).

**Orque** (47,7 %) — coût 2, recharge 1/2 : désarme les ennemis adjacents (ils ne peuvent attaquer que lui).
- **U43 Rôle** : parade **au contact** : empêche les attaquants adjacents de frapper le Roi ce tour-là.
- **U44 Bon usage** : au tour où **deux ennemis sont adjacents au Roi** ou entrent dans sa zone, l'Orque adjacent à eux les empêche de frapper (ils ne peuvent attaquer que l'Orque). C'est **la seule parade directe** à la menace au contact.
- **U45 Mauvais usage** : utilisé trop tôt (recharge d'un tour sur deux) ; lancé quand aucun ennemi adjacent n'attaquerait le Roi.
- **U46 Combos** : Automate (+0,14 : attire, puis désarme) ; Sanctuaire (−0,15 : même rôle, redondance).
- **U47 Craint** : Guérisseur (−0,16 [M]), Automate (−0,15 [M]) ; par les règles : Furtif (non ciblable, donc pas désarmé) et Mentaliste (déplace l'Orque) [R].
- **U48 Draft** : sous-estimé par l'IA ; **au niveau humain**, c'est l'unité défensive la plus précieuse contre l'assaut.

**Automate** (55,8 %) — coût 3, recharge 1/2 : attire d'une case les ennemis situés à 2 cases (carré 5×5, diagonales et « L » compris).
- **U49 Rôle** : contrôle de position : **ramène** les ennemis dans la zone de contrôle de l'équipe (et sort de leur formation).
- **U50 Bon usage** : attirer des ennemis **dans la zone du Colosse, sur un piège, devant l'Orque** ; **attirer le Roi adverse** à portée de mes attaquants (T57) ; défaire un assaut en rapprochant les pièces de ma défense.
- **U51 Mauvais usage** : lancer quand l'attraction rapproche les ennemis de **mon Roi** ; lancer sans suite (le coût est de 3 et le Roi adverse a le temps de répondre).
- **U52 Combos** : Catalyseur (+0,21 : mana), Orque (+0,14), Provocation + immobilisation ; mal avec Boréale (−0,15).
- **U53 Craint** : Guérisseur (−0,17 [M]), Roi Marchand (−0,19 [M]) ; **bat** le Sanctuaire (+0,17) et l'Orque (+0,15) [M].
- **U54 Draft** : **l'unité la plus rentable après le Félin** (55,8 %) ; **à éviter** contre le Roi Liche et contre un Roi Marchand adverse (leurs victoires augmentent de 0,25 et 0,19 face à l'Automate [M]).

### 7.3 Les Mages

**Boréale** (47,1 %) — coût 2 : pose un mur sur une case libre du carré 5×5 (bloque passage et vue ; dure jusqu'au 2e tour suivant).
- **U55 Rôle** : contrôle de l'espace : ferme une ligne, protège un couloir.
- **U56 Bon usage** : couper une **ligne de vue** (Geôlier, Mentaliste, Automate adverses) ; fermer la colonne du Félin ; réduire les cases d'approche du Roi **sans le sceller** (règle d'atteignabilité).
- **U57 Mauvais usage** : poser un mur qui bloque **ses propres** pièces ; lancer à chaque tour (puits à mana : l'IA en lance 7,6 par partie [A], sans bénéfice).
- **U58 Combos** : Colosse (le mur rabat l'ennemi vers lui) ; Roi Bastion.
- **U59 Craint** : Automate adverse et Mentaliste (le mur ne bloque pas le déplacement par attraction).
- **U60 Draft** : faible en S2 (47 %) ; **meilleure aux mains d'un humain** qui sait couper une ligne au bon moment.

**Trappeur** (53,8 %) — coût 2 : pose 2 pièges (carré 5×5) ; une pièce qui entre dessus est immobilisée ; un nouveau lancer retire les anciens.
- **U61 Rôle** : zone de déni permanent : protège un couloir sans rien dépenser tant qu'il n'est pas relancé.
- **U62 Bon usage** : piéger **deux cases d'un même couloir** par lequel l'adversaire doit passer ; piéger la case où le Minotaure/l'Automate/la Mentaliste amèneront l'ennemi.
- **U63 Mauvais usage** : piéger un chemin que ses propres pièces doivent prendre ; relancer inutilement (les anciens pièges disparaissent).
- **U64 Combos** : Minotaure (pousse sur un piège), Automate (attire sur un piège), Mentaliste.
- **U65 Craint** : Sanctuaire (+0,18 [M]), Voltigeuse (+0,13 : elle saute par-dessus un piège adjacent), Roi Bastion [R]. **Bat** le Félin (+0,17 [M]).
- **U66 Draft** : solide (53,8 %) ; **contre-draft** : Sanctuaire ou Bastion.

**Mentaliste** (52,4 %) — coût 3, recharge 1/2 : déplace un ennemi à portée 2 en ligne (ligne de vue) sur une case libre adjacente à lui, puis l'immobilise.
- **U67 Rôle** : contrôle précis : sort un attaquant de la zone ou **tire une pièce (même le Roi) dans la zone de mes attaquants**.
- **U68 Bon usage** : **tirer le Roi adverse** à côté de mes pièces (puis attaque) ; sortir un attaquant adjacent à mon Roi ; mettre une pièce sur un piège.
- **U69 Mauvais usage** : déplacer une pièce **vers** son Roi ; lancer quand une autre action suffirait.
- **U70 Combos** : Guérisseur (+0,17 : jeu lent), Minotaure, Trappeur, Colosse.
- **U71 Craint** : Félin (+0,12 pour lui [M]) ; **bat** la Messagère (+0,17 [M]). Par les règles : Sanctuaire adjacent à la cible et Furtif la rendent inopérante [R].
- **U72 Draft** : bonne pour J2 (réponse précise) ; **cible n°1 des bans** quand l'adversaire joue un Roi fragile.

**Geôlier** (47,8 %) — coût 2 : immobilise un ennemi à portée 2 en ligne (ligne de vue) et un second ennemi adjacent au premier.
- **U73 Rôle** : contrôle à distance de **deux pièces à la fois**.
- **U74 Bon usage** : immobiliser **deux pièces qui entrent dans la zone de mort** à distance 2 (elles ne peuvent plus entrer) ; contrôle de colonne.
- **U75 Mauvais usage** : viser des pièces déjà adjacentes au Roi (inutile : T4) ; lancer si le second ennemi est inutile.
- **U76 Combos** : Boréale (−0,11 : mur gêne sa ligne), Colosse (même zone), Trappeur.
- **U77 Craint** : Sanctuaire, Furtif, Bastion [R]. **Bat** le Catalyseur (+0,13 [M]).
- **U78 Draft** : moyen (47,8 %) ; devient fort **avec un Roi qui profite du tempo** (Liche + Geôlier = tempo et contrôle).

### 7.4 Les Supports

**Sanctuaire** (49,2 %) — coût 2, recharge 1/2 : lui-même et ses alliés adjacents ne peuvent être ni immobilisés ni déplacés pendant le prochain tour adverse.
- **U79 Rôle** : bouclier collectif : annule le contrôle et le déplacement forcé.
- **U80 Bon usage** : **juste avant un tour de contrôle adverse** (Liche, Automate, Mentaliste, Trappeur) ; en défense autour du Roi (il protège le Roi du déplacement) ; en attaque, avec ses attaquants adjacents.
- **U81 Mauvais usage** : le lancer tous les tours (5 lancers par partie [A]) ; l'oublier quand la recharge est prête et que l'adversaire a des contrôles.
- **U82 Combos** : Prêtresse (+0,14), Messagère (+0,13) ; mal avec Orque (−0,15), Mentaliste (−0,13).
- **U83 Craint** : Prêtresse adverse (−0,14 [M]), Arlequine (−0,14 [M]), Automate (−0,17 [M]) ; la cause probable est la **recharge** : un tour sur deux, le Sanctuaire n'est pas actif [H].
- **U84 Draft** : **contre-draft du Trappeur** (+0,18) ; en revanche il perd contre l'Automate, l'Arlequine et la Prêtresse. À bannir quand on joue Trappeur.

**Guérisseur** (51,7 %) — coût 2, usage unique : le Roi regagne 1 PV.
- **U85 Rôle** : assurance : transforme « 2 coups » en « 3 coups ».
- **U86 Bon usage** : après un premier coup sur le Roi **et** quand une seconde frappe est imminente ; avec un Roi qui a déjà été touché.
- **U87 Mauvais usage** : lancer avant d'avoir été touché (le PV maximal est 2 : le soin est perdu).
- **U88 Combos** : Mentaliste (+0,17), jeu lent ; avec Orque (gagner le tour d'après).
- **U89 Craint** : Minotaure (+0,17 [M]). **Bat** l'Orque (+0,16) et l'Automate (+0,17) [M].
- **U90 Draft** : cible utile mais pas indispensable ; **à ne prendre que si l'équipe défend**.

**Prêtresse** (51,4 %) — coût 2, recharge 1/2 : libère un allié immobilisé.
- **U91 Rôle** : contre-contrôle : rend le mouvement à une pièce (et surtout l'attaque ou le saut).
- **U92 Bon usage** : libérer **la pièce clé** juste avant son entrée dans la zone ; contre un Liche, la libérer **avant** le tour d'assaut.
- **U93 Mauvais usage** : libérer une pièce qui n'a pas de mouvement utile ; l'oublier après la paralysie générale.
- **U94 Combos** : Bastion (+0,15), Sanctuaire (+0,14), Félin (+0,12) ; **mal avec Diplomate** (−0,16 : l'échange fait déjà ce travail).
- **U95 Craint** : Colosse (immobilise les pièces qu'elle devrait libérer) [R]. **Bat** le Sanctuaire (+0,14 [M]).
- **U96 Draft** : bon support « ceinture et bretelles » contre les équipes de contrôle.

**Catalyseur** (44,4 %) — coût 1, recharge 1/2 : +2 mana (gain net +1).
- **U97 Rôle** : économie ; **utile uniquement au contact**.
- **U98 Bon usage** : au tour de l'assaut quand le mana est à 5 et que l'action complète exige 7+ ; dans une équipe de compétences à 3 (Automate : +0,21 ; Arlequine, Voltigeuse, Minotaure).
- **U99 Mauvais usage** : l'utiliser avant le contact (le plafond de 6 absorbe le gain).
- **U100 Combos** : Automate (+0,21), Diplomate (+0,20) ; **mal** avec Furtif (−0,13), Stratège (−0,13).
- **U101 Craint** : Geôlier (−0,13).
- **U102 Draft** : unité **la plus faible mesurée après la Messagère** ; à prendre seulement dans une équipe à trois compétences à 3.

**Messagère** (39,9 %) — coût 2 : un allié de son choix (où qu'il soit) gagne +1 déplacement ce tour.
- **U103 Rôle** : tempo ciblé ; **dernier pas** de l'assaut.
- **U104 Bon usage** : donner +1 à la pièce qui, ce tour-là, **entre dans la zone de mort et frappe** (permet d'entrer un tour plus tôt) ; à l'unité qui sort d'une immobilisation.
- **U105 Mauvais usage** : donner le déplacement à une pièce qui n'a pas de case utile ; l'utiliser avant le contact.
- **U106 Combos** : Sanctuaire (+0,13) ; mal avec Trappeur (−0,13), Furtif (−0,13), Catalyseur (−0,14, deux unités de mana sans pression).
- **U107 Craint** : Mentaliste adverse (−0,17 [M]), Roi Marchand adverse (−0,19 [M]).
- **U108 Draft** : **la plus faible en S2 (39,9 %)** mais la plus **dépendante du niveau** (54 % en S3 [A]) ; à réserver aux niveaux Diamant+ qui savent la jouer.

### 7.5 Les Assassins

**Félin** (57,0 %) — passif : 2 cases en ligne droite un tour sur 3, 1 case sinon.
- **U109 Rôle** : tempo gratuit : arrive 2 cases plus tôt à la zone de mort.
- **U110 Bon usage** : colonne libre (aucune pièce en face) ; sprint au tour où il fait entrer la pièce dans la zone ; avec Arlequine (+0,20).
- **U111 Mauvais usage** : sprinter dans une case piégée (le Félin s'arrête sur la première case piégée) ; sprinter vers un mur.
- **U112 Combos** : Arlequine (+0,20), Prêtresse (+0,12). **Craint** le Roi Diplomate (−0,14 [M]).
- **U113 Craint** : Trappeur (−0,17 [M] : un piège sur la première case d'une course l'arrête), Boréale (colonne fermée) [R]. **Bat** la Mentaliste (+0,12).
- **U114 Draft** : **meilleur pick mesuré** (57 %) : premier ban/pick.

**Voltigeuse** (53,4 %) — coût 3, recharge 1/2 : saute par-dessus une pièce adjacente en ligne droite vers la case libre derrière ; pas d'attaque de base ce tour.
- **U115 Rôle** : franchit un front et un blocage.
- **U116 Bon usage** : **contourner une pièce immobilisée** qui ferme un couloir ; passer le front adverse et **entrer dans la zone de mort** (saut = 2 cases) ; sauter par-dessus le Roi adverse (sans l'attaquer) pour se mettre dans son dos.
- **U117 Mauvais usage** : sauter sans suite (elle ne peut pas attaquer ensuite : elle doit être soutenue).
- **U118 Combos** : Trappeur (+0,13 contre lui : elle saute par-dessus un piège adjacent), Messagère (le déplacement + saut).
- **U119 Craint** : les pièces derrière la cible (case occupée = saut impossible) et le mur [R] ; **anti-synergie** avec le Colosse allié (−0,12 [M]).
- **U120 Draft** : bon choix (53,4 %) ; **le contre de Stratège** (−0,20) : bannir quand le Stratège est le Roi adverse.

**Furtif** (46,1 %) — coût 1, recharge 1/2 : ne peut pas être ciblé par l'adversaire pendant son prochain tour.
- **U121 Rôle** : pion invulnérable ponctuel.
- **U122 Bon usage** : traverser une zone de contrôle (Geôlier, Trappeur, Colosse) ; couvrir **une pièce clé** quand elle y est seule ; passer en couloir quand il est la seule pièce qui peut le faire.
- **U123 Mauvais usage** : le lancer quand il n'y a pas d'adversaire à portée de contrôle ; le lancer tous les tours (recharge).
- **U124 Combos** : Colosse (+0,13), Orque (anti-ciblage rendu inutile) ; mal avec Minotaure (−0,14), Messagère (−0,13), Catalyseur (−0,13).
- **U125 Craint** : les effets globaux (Liche : immobilise tous les ennemis, y compris un Furtif non protégé) [R]. **Bat** l'Arlequine (+0,13 [M] : elle ne peut pas le cibler).
- **U126 Draft** : faible (46 %) mais peu risqué ; devient fort **dans les mains de l'IA qui sait où se placer**.

**Arlequine** (54,6 %) — coût 3 : échange sa place avec une unité adjacente (8 cases, hors Roi).
- **U127 Rôle** : téléportation courte : jusqu'à 2 cases d'un coup, et **échange avec une pièce adverse** (ses alliés comme ses ennemis).
- **U128 Bon usage** : s'échanger avec **un ennemi avancé** pour entrer dans la zone (l'ennemi se retrouve en arrière) ; échanger avec **mon propre attaquant** pour lui faire gagner 2 cases ; **échanger avec un ennemi adjacent au Roi** pour l'éloigner et prendre sa place.
- **U129 Mauvais usage** : se placer **au contact** de plusieurs ennemis (sans protection) ; échanger sans gain de tempo.
- **U130 Combos** : Félin (+0,20), Roi Diplomate (+0,11).
- **U131 Craint** : Furtif (−0,13 [M]), Roi Liche (−0,11 [M]). **Bat** le Sanctuaire (+0,14) et le Minotaure (+0,13) [M].
- **U132 Draft** : bonne unité (54,6 %), polyvalente ; **atout de contre-draft** face à des équipes à unités fixes.

## 8. Combos et séquences

Un combo = une suite d'actions du même tour (ou de deux tours consécutifs) qui produit un résultat qu'aucune action seule n'obtient. Les combos « mesurés » sont ceux dont la paire apparaît dans les synergies (annexe A.3) ; les autres sont des **séquences de règles** [R].

### 8.1 Combos de tempo (arriver plus vite)

- **C1 [M/R]** **Arlequine + Félin** (+0,20, z = 3,2) : l'Arlequine s'échange avec le Félin avancé ; le Félin gagne 2 cases de plus et se retrouve déjà dans la zone de mort. Meilleur synergie mesurée.
- **C2 [R]** **Messagère + n'importe quelle pièce à 1 pas de la zone** : +1 déplacement = entrée et attaque dans le même tour.
- **C3 [R]** **Minotaure + case libre derrière l'ennemi** : le Minotaure avance de 2 cases (charge à distance 2 en ligne) et repousse l'ennemi.
- **C4 [R]** **Diplomate + pièce avancée** : échange une pièce arrière avec une pièce déjà proche de la zone (saut de 5 à 6 cases) ; le tour suivant, deux pièces sont dans la zone.
- **C5 [R]** **Voltigeuse + pièce adverse qui bloque** : saute par-dessus un blocage qui coûterait deux tours à contourner.
- **C6 [R]** **Roi Stratège + armée à 2 rangées de la zone** : +1 pas pour tous : toutes les pièces entrent et frappent le même tour.
- **C7 [R]** **Félin + colonne libre** : 2 cases gratuites tous les 3 tours : avant le tour du sprint, **ne pas bloquer sa propre colonne**.
- **C8 [R]** **Arlequine + pièce adverse avancée** : échanger avec un ennemi avancé le renvoie en arrière et amène l'Arlequine à sa place : un tempo net de 2 cases en un coup.

### 8.2 Combos de contrôle (empêcher l'adversaire d'agir)

- **C9 [R]** **Trappeur + Minotaure** : le Minotaure repousse un ennemi **sur un piège** : l'ennemi est immobilisé sans que le Minotaure ait besoin de dépenser un coup d'immobilisation.
- **C10 [R]** **Trappeur + Automate** : attirer un ennemi sur la case piégée adjacente à l'Automate.
- **C11 [R]** **Automate + Colosse** : l'Automate attire les ennemis devant le Colosse ; le Colosse les immobilise **tous** d'un coup (2 compétences, 5 mana : un tour de transition, le suivant est décisif).
- **C12 [R]** **Automate + Orque** (+0,14) : attirer, puis désarmer les ennemis adjacents : ils ne peuvent plus attaquer que l'Orque.
- **C13 [R]** **Mentaliste + attaquants** : tirer une pièce (ou le **Roi adverse**) à côté de mes pièces, puis la frapper ; **le Roi tiré** est le combo le plus fort du jeu (immobilisé et adjacent à mes pièces : 2 attaques = victoire).
- **C14 [R]** **Geôlier + Boréale** : la ligne de vue est coupée par le mur : à éviter (−0,11) ; **inversement**, utiliser le mur adverse **comme écran** pour le Geôlier adverse.
- **C15 [R]** **Liche + attaque en bloc** : une seule fois par partie : paralysie, puis entrée dans la zone avec deux pièces ; l'adversaire ne peut ni bouger ni libérer (hors Prêtresse).
- **C16 [R]** **Colosse + Orque** : zone de verrou complète : les ennemis adjacents sont immobilisés **et** désarmés (si l'Orque est prête).
- **C17 [R]** **Sanctuaire + attaquants adjacents** : protège le groupe d'un contrôle pendant le tour adverse : à lancer **avant** le tour de Liche/Automate/Mentaliste.
- **C18 [M/R]** **Sanctuaire + Prêtresse** (+0,14) : protection + libération : rend une équipe presque insensible au contrôle.
- **C19 [R]** **Boréale + Colosse** : le mur oblige l'ennemi à passer par une case adjacente au Colosse.
- **C20 [R]** **Furtif + Colosse** (+0,13) : le Furtif va au contact **sans pouvoir être ciblé** ; le Colosse immobilise le reste.

### 8.3 Combos de défense

- **C21 [R]** **Orque au contact** : parade directe d'un assaut ; à jouer au tour où deux attaquants sont adjacents au Roi.
- **C22 [R]** **Guérisseur après une première touche** : gagne un tour supplémentaire ; combiné à l'Orque, l'attaque est stoppée deux fois.
- **C23 [R]** **Mentaliste sur un attaquant** : le sort de la zone de mort (adjacent à ma Mentaliste) et l'immobilise.
- **C24 [R]** **Minotaure sur un attaquant adjacent au Roi** : repousse d'une case : l'attaquant sort de la zone d'attaque (ne peut plus frapper s'il n'est plus adjacent, mais peut revenir : toujours à vérifier).
- **C25 [R]** **Sanctuaire adjacent au Roi + Colosse** : le Roi ne peut ni être déplacé ni immobilisé ; les ennemis qui arrivent sont immobilisés.
- **C26 [R]** **Boréale devant le Roi** : réduit les cases d'approche à 2 ; le Geôlier et le Trappeur couvrent les deux cases restantes.
- **C27 [R]** **Trappeur devant le Roi (distance 2)** : deux pièges à la bonne distance empêchent d'entrer dans la zone sans s'arrêter.

### 8.4 Combos économiques

- **C28 [M/R]** **Automate + Catalyseur** (+0,21, z = 3,9) : le Catalyseur finance les compétences à 3 de l'Automate (le meilleur couple mesuré).
- **C29 [R]** **Diplomate + Catalyseur** (+0,20) : l'échange coûte 3 mana ; le Catalyseur le finance.
- **C30 [R]** **Marchand + trois compétences à 3** : le tour explosif à 8-9 mana.
- **C31 [M/R]** **Guérisseur + Mentaliste** (+0,17) : jeu lent, défensif ; la Mentaliste est une unité à recharge.
- **C32 [M/R]** **Prêtresse + Bastion** (+0,15) : double protection contre le contrôle.

### 8.5 Combos de Roi (Roi + unité)

- **C33 [M]** **Roi Diplomate + Catalyseur** (+0,20) ; **Roi Diplomate + Arlequine** (+0,11) : deux sources de téléportation.
- **C34 [M]** **Roi Bastion + Prêtresse** (+0,15) ; **Roi Bastion + Automate** (+0,11).
- **C35 [M]** **Roi Liche + Trappeur** (+0,09) ; **Roi Liche** fort contre Automate et Prêtresse adverses.
- **C36 [M]** **Roi Stratège** mal accompagné de Messagère, Catalyseur, Furtif, Colosse : pas d'unité qui dépense du mana pour un gain mineur.

### 8.6 Anti-combos (à reconnaître et empêcher)

- **C37 [R]** La **Mentaliste adverse** qui tire mon Roi : toujours garder un Sanctuaire adjacent ou rester hors de sa ligne.
- **C38 [R]** Le **Liche adverse + Félin** : paralysie puis entrée : ne pas avoir deux pièces dans le carré 5×5 au tour du Liche.
- **C39 [R]** L'**Arlequine adverse** qui échange avec ma pièce avancée : ne pas laisser une pièce clé **adjacente** à l'Arlequine.
- **C40 [R]** L'**Automate adverse** : attirer mes pièces sur ses pièges : ne pas s'aligner à distance 2 de lui quand la recharge est prête.

## 9. Anticiper : lecture de l'adversaire

### 9.1 Niveaux de profondeur

- **A1 [R]** **Profondeur 1** (« glouton ») : choisit le meilleur tour pour soi, sans regarder la réponse. Perd contre toute menace à deux tours.
- **A2 [R]** **Profondeur 2** : regarde la **meilleure réponse adverse** après son tour : voit les menaces immédiates (un Roi touchable au tour suivant) et évite de se mettre en prise.
- **A3 [R]** **Profondeur 3** : voit **sa propre suite** : ses menaces à deux tours (préparer une double menace) ; c'est le premier niveau qui joue des **plans**.
- **A4 [R]** **Profondeur 4** : voit la **réponse à la double menace** : repère les contre-plans de l'adversaire (le défenseur qui prépare un Sanctuaire/une immobilisation à distance 2).
- **A5 [R]** **Profondeur 5-6** : au-delà des plans à deux tours : voit des **séquences de piégeage** (sacrifice de tempo) ; suffisant en pratique pour une partie qui dure 10 tours.
- **A6 [M]** Les parties courtes (10 tours) rendent la profondeur très rentable : une IA à profondeur 3 bat une IA à profondeur 2 dans **59,8 %** des parties (300 parties, soit environ +70 Elo) : voir §10.

### 9.2 Modèle de l'adversaire

- **A7 [R]** **Hypothèse du meilleur adversaire** (minimax) : correcte contre un bon joueur ; contre un joueur faible, on gagne **plus** en cherchant les erreurs probables.
- **A8 [R]** **Modèle des habitudes** : l'IA retient (pour la session) quelles unités l'adversaire prend, quels bans il fait, quels placements il joue (colonne du Roi) ; elle adapte ses bans (bannir sa signature) et son placement.
- **A9 [R]** **Modèle de défense** : si le joueur ne protège jamais son Roi à distance 2, l'IA prépare **un rush simple** ; s'il garde toujours deux pièces à côté du Roi, l'IA joue **le contrôle**.
- **A10 [R]** **Modèle de compétence** : l'IA estime le niveau du joueur (ses erreurs de tempo, ses oublis de recharge) pour **choisir ses plans** : le Challenger ne se contente pas de calculer, il choisit les plans que ce joueur **sait** parer le moins bien.
- **A11 [R]** Une **IA prévisible** se fait exploiter par un humain : prévoir un **bruit contrôlé** sur les coups quasi-équivalents (hasard parmi les coups à moins de X points de la meilleure valeur).

### 9.3 Anticipation de ses propres plans (préparation)

- **A12 [R]** **Préparer le contact** : placer ses pièces pour que les deux entrent dans la zone de mort **au même tour** (arrivée simultanée), plutôt que d'envoyer la première seule.
- **A13 [R]** **Préparer la recharge** : savoir à chaque tour quelles compétences seront disponibles au tour N+1 et N+2, et **retarder** un assaut d'un tour pour attendre une recharge.
- **A14 [R]** **Préparer le mana** : avoir 2 mana d'avance après les compétences ; planifier le Catalyseur le tour d'avant.
- **A15 [R]** **Préparer un piège de tempo** : placer une pièce **hors de portée mais à distance 2** de la zone de contrôle adverse pour forcer l'adversaire à l'immobiliser (ou à gaspiller une compétence).
- **A16 [R]** **Réserver son unique usage** (Liche, Stratège, Guérisseur) : ne pas tirer au premier tour où c'est utile, mais **au tour où cela est décisif** : demande une recherche à profondeur 3+.

### 9.4 Anticipation des actions de l'adversaire

- **A17 [R]** **Ce qu'il peut faire à son prochain tour** : calculer **toutes** les menaces (deux pièces dans la zone, mana ≥ 2) ; **la plus dangereuse n'est pas la plus évidente** (souvent un échange Arlequine/Diplomate qui téléporte une pièce).
- **A18 [R]** **Ce qu'il ne peut pas faire** : compter les compétences en recharge, le mana, les « usages uniques » consommés : la défense réelle est souvent **moins** chargée qu'elle ne paraît.
- **A19 [R]** **Anticiper le Liche** : si l'adversaire a le Roi Liche et son usage n'est pas consommé, **chaque tour est un tour de paralysie possible** : ne jamais entrer à deux dans la zone sans prévoir qu'il peut répondre (paralysie), et surtout ne pas se mettre sur sa case de tempo.
- **A20 [R]** **Anticiper l'Automate/Mentaliste** : tout ce qui est à distance 2 de ces unités est déplaçable : le Roi aussi.
- **A21 [R]** **Anticiper la pose de piège** : le Trappeur qui pose au tour N pose sur **le couloir où je dois passer** au tour N+1 : passer par un autre couloir ou envoyer d'abord une pièce sacrifiable.
- **A22 [R]** **Anticiper le dernier pick de J2** (draft) et **le placement de J2** (placement) : J1 doit choisir ce qui reste bon quelle que soit la réponse (§4 et §5).

### 9.5 Pièges d'anticipation (erreurs d'une IA)

- **A23 [R]** **Horizon** : l'IA à profondeur 2 repousse un mauvais coup d'un tour sans le voir ; la profondeur 3 sait que la menace revient.
- **A24 [R]** **Ignorer la recharge adverse** : l'IA croit qu'une unité peut rejouer alors qu'elle est en recharge.
- **A25 [R]** **Sur-défendre** : un bot trop prudent (style tortue) perd le tempo : 72,7 % de victoires pour un joueur d'attaque contre un joueur tortue de même niveau [M] (300 parties).
- **A26 [R]** **Ignorer le gain de territoire** : une IA qui ne valorise pas l'avance (valeur de terrain nulle) donne la zone de contrôle sans combattre.
- **A27 [R]** **Gaspiller le mana** : une IA qui dépense à chaque tour sans garder 2 mana pour le coup final ne peut pas conclure.
- **A28 [R]** **Éviter les mêmes erreurs de type humain** : l'IA de bas niveau doit avoir des **erreurs plausibles** (oublier une recharge, ne pas voir une menace à distance 2, ne pas utiliser le Guérisseur), et non un « bruit de coups » qui les rend illisibles.

## 10. Les dix niveaux d'IA, de Fer à Challenger

### 10.1 Principe : un niveau = un ensemble de **connaissances** et de **moyens de calcul**

Les IA actuelles (§2) se distinguent surtout par un **bruit aléatoire** et une profondeur de recherche : le bas du classement joue « au hasard », le haut « calcule plus ». Ce n'est pas ce que vit un joueur : un Fer humain ne joue pas au hasard, il **ignore des concepts**. La proposition : chaque niveau est défini par **les concepts qu'il maîtrise** (matrice 10.3) et **un budget de calcul**. Un joueur qui progresse découvre alors réellement les idées du jeu (zone de mort, désarmement avant l'assaut, ban anticipé, placement face à J2…) en passant de palier en palier.

Trois leviers mesurés (§3–§5) donnent l'ordre de grandeur de ce qu'on peut ajouter :

| Levier | Mesure | Équivalent en Elo |
|---|---|---|
| Draft pilotée par un modèle contre draft aléatoire | 59,6 % [M] | **≈ +70** |
| Une profondeur de recherche en plus (S3 contre S2) | 59,8 % [M] | **≈ +70** |
| Placement guidé contre aléatoire | 53,4 % [M] | **≈ +24** |
| J2 qui exploite le placement de J1 (avec l'implémentation actuelle) | 52,1 % [M], non significatif | ≈ +14 |
| Style tortue contre style attaque (même profondeur) | 27,3 % pour la tortue [M] | ≈ −170 |
| Évaluation apprise par auto-jeu, profondeur 4-6, modèle de l'adversaire | non mesuré | à mesurer (hypothèse : +100 à +250 au total) [H] |

(Conversion : 59,6 % ≈ +69 Elo ; 53,4 % ≈ +24 ; 52,1 % ≈ +15.)

### 10.2 Moyens de calcul (par tour, sur ordinateur ; ×2 à ×4 de marge pour mobile)

Mesure actuelle (avec 2 simulations en parallèle) : Or 33 ms par tour, Émeraude 20 ms, Maître 33 ms, Grand Maître 53 ms, **Challenger 119 ms**. Il reste donc **10 à 50 fois** de marge avant de gêner le joueur (seuil confortable : environ 1 s par tour, 2 s maximum au plus haut niveau). Le calcul doit tourner dans un **Web Worker** (pour ne pas figer l'écran) et utiliser une **recherche à budget de temps** (approfondissement itératif : la profondeur s'adapte à la machine).

| Niveau | Budget cible par tour |
|---|---|
| Fer à Argent | moins de 30 ms |
| Or | 40–80 ms |
| Platine | 100 ms |
| Émeraude | 200 ms |
| Diamant | 350 ms |
| Maître | 600 ms |
| Grand Maître | 1 s |
| Challenger | 1,5–2 s (plafonné) |

### 10.3 Matrice concepts × niveaux

Légende : ○ ignoré · ◐ partiel (le concept est vu, mais avec erreurs ou seulement dans les cas simples) · ● maîtrisé.

| Concept | Fer | Bronze | Argent | Or | Platine | Émeraude | Diamant | Maître | GM | Chall. |
|---|---|---|---|---|---|---|---|---|---|---|
| Jouer des coups légaux, avancer | ◐ | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Frapper le Roi quand une pièce lui est adjacente | ◐ | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Voir la menace immédiate sur son Roi (touché au tour suivant) | ○ | ◐ | ◐ | ● | ● | ● | ● | ● | ● | ● |
| Immobiliser à distance 2 avant l'entrée en zone | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● | ● | ● |
| Parer le contact (désarmer, déplacer, repousser) | ○ | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● | ● |
| Zone de mort : deux pièces + 2 mana | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● | ● | ● |
| Double menace (deux fronts) | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● | ● |
| Gérer le mana (garder 2 pour le coup final, dépenser avant le contact) | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● | ● | ● |
| Recharges et usages uniques (timing Liche, Stratège, Guérisseur) | ○ | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● | ● |
| Terrain : murs et pièges utiles (sans gêner soi-même) | ○ | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● | ● |
| Avancer pour le territoire / bloquer par immobilisation | ○ | ○ | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● |
| Combos de paires (Arlequine + Félin, Automate + Orque…) | ○ | ○ | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● |
| Tirer le Roi adverse (Mentaliste, Automate, Minotaure) | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● |
| Anticiper à 2 tours (profondeur 3) | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● | ● |
| Anticiper le contre-plan adverse (profondeur 4) | ○ | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● |
| Séquences de piégeage et sacrifices de tempo (profondeur 5-6) | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● |
| Lire l'adversaire (habitudes, modèle de compétence) | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● |
| **Choix du Roi** : force de base | ○ | ◐ | ● | ● | ● | ● | ● | ● | ● | ● |
| **Draft** : valeur de base des unités | ○ | ◐ | ● | ● | ● | ● | ● | ● | ● | ● |
| **Draft** : Roi × unité, composition par classes (pas 3 Supports) | ○ | ○ | ○ | ◐ | ● | ● | ● | ● | ● | ● |
| **Draft** : synergies et contres | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● | ● |
| **Ban** : retirer l'unité qui complète l'adversaire | ○ | ○ | ○ | ○ | ◐ | ◐ | ● | ● | ● | ● |
| **Draft** : dernier pick de J2, flexibilité de J1 | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● |
| **Draft** : recherche et simulation (placement compris) | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● |
| **Placement** : Roi central avec un voisin | ○ | ◐ | ● | ● | ● | ● | ● | ● | ● | ● |
| **Placement** : guidé (candidats évalués) | ○ | ○ | ○ | ● | ● | ● | ● | ● | ● | ● |
| **Placement** : J2 répond au placement de J1 | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● | ● |
| **Placement** : J1 robuste à la meilleure réponse | ○ | ○ | ○ | ○ | ○ | ◐ | ● | ● | ● | ● |

### 10.4 Fiche de chaque niveau

**Fer** (Elo cible : 0–99, ancre 50) — *il joue vraiment mal, mais de façon lisible.*
- Roi, draft, bans, placement : au hasard. Il avance et frappe le Roi s'il est adjacent (1 fois sur 2).
- Ne voit pas les menaces sur son Roi ; oublie les recharges ; utilise ses compétences presque au hasard.
- Rôle : accueillir un nouveau joueur, qui doit pouvoir le battre en 3–4 parties en apprenant les règles.
- Calcul : < 5 ms.

**Bronze** (100–199, ancre 150)
- Roi au hasard (évite parfois le Stratège), draft par valeur de base avec bruit fort, bans au hasard.
- Roi jamais isolé au placement (règle P10).
- Coup glouton : maximise un tour sans regarder la réponse (profondeur 1) ; ne voit pas les menaces immédiates la moitié du temps.

**Argent** (200–299, ancre 250)
- Roi pondéré par la force (Liche, Bastion, Diplomate), draft par valeur de base, bans des unités fortes.
- Roi central, voisin dans la rangée ; reste aléatoire.
- Voit la menace immédiate sur son Roi (80 %) ; immobilise parfois à distance 2.

**Or** (300–399, ancre 350) — *« déjà corsé » : un joueur qui n'a jamais réfléchi à la draft perd contre lui.*
- Profondeur 2 (il voit la meilleure réponse) avec faisceau étroit ; voit toutes les menaces immédiates.
- Draft : valeur de base + règles de composition (pas 3 Supports, au moins un Assassin ou un Tank à contact, un contre simple).
- Placement guidé (8 à 12 candidats) avec Roi central.
- Gère le mana de façon simple (garde 2 mana si l'adversaire est à distance ≤ 3).
- Défauts volontaires : ne connaît pas les combos, oublie le timing des usages uniques, ne tire jamais le Roi adverse.

**Platine** (400–499, ancre 450)
- Profondeur 2 large ; connaît l'immobilisation à distance 2, la parade (Orque, Minotaure), le terrain utile, la zone de mort.
- Draft : modèle additif (force + Roi × unité), évite les anti-synergies ; bans : l'unité forte pour son Roi adverse.
- Placement : 24 candidats filtrés par règles (Roi central, voisins, contrôleurs à côté), évalués par mini-parties.

**Émeraude** (500–599, ancre 550)
- Profondeur 3 : prépare des doubles menaces et des plans à deux tours ; timing des usages uniques ; avance de territoire.
- Draft : modèle complet (paires, contres), anticipe un pas ; J2 exploite le placement de J1.

**Diamant** (600–699, ancre 650)
- Profondeur 3 large + évaluation affinée (poids appris) ; combos de paires ; tire le Roi adverse quand c'est décisif ; bans prédictifs.
- Placement : candidats évalués par parties avec l'IA Argent (et non gloutonne).

**Maître** (700–799, ancre 750)
- Profondeur 4 (voit le contre-plan) avec budget de temps ; draft par recherche à 2 coups ; placement par évaluation à 2 coups.
- Reconnaît les pièges tactiques à 4 coups ; lit les habitudes de la session en cours (pour ses bans).

**Grand Maître** (800–899, ancre 850)
- Profondeur 5 avec table de transposition ; évaluation apprise par auto-jeu ; draft par simulation Monte-Carlo (placement compris) ; modèle de l'adversaire.
- Cible : **un joueur très réfléchi ne le bat que rarement**.

**Challenger** (900+, ancre 950) — *« imbattable »*
- Profondeur 6 avec budget 1,5–2 s ; évaluation apprise ; arbre de draft complet avec valeur apprise ; placement par recherche d'équilibre ; modèle du joueur adverse ; bruit nul.
- Cible : **moins de 3 % de victoires** pour un joueur humain même très entraîné.

### 10.5 Étalonnage : comment fixer les Elo des niveaux

1. **Tournoi à la ronde** : chaque niveau contre ses deux voisins et deux niveaux plus bas, 400 parties par paire (précision ±2,5 points), côtés alternés. On calcule l'Elo de chaque niveau par maximum de vraisemblance (Bradley–Terry), pas de proche en proche.
2. **Cible entre voisins** : 64 % de victoires pour le niveau supérieur (écart de 100 Elo) jusqu'à Émeraude ; **70 % à 85 %** entre voisins dans le haut (écart de 150 à 300 Elo) : c'est ce qui rend les sommets « extrêmement difficiles ».
3. **Étalonnage humain** : une session d'essai du joueur (vous) : 20 parties contre chaque niveau de Argent à Challenger ; cibles pour un joueur expert : Or ≈ 80 %, Platine ≈ 65 %, Émeraude ≈ 50 %, Diamant ≈ 30 %, Maître < 15 %, Grand Maître < 5 %, Challenger ≈ 0–1 %.
4. **Étalonnage par la population** : exporter les rapports de partie (déjà produits par le jeu) de quelques joueurs et recaler les ancres.
5. **Révision à chaque changement de règle ou d'IA** : refaire le tournoi complet (environ 2 h de calcul).

## 11. Les personnalités (mode non classé)

### 11.1 Principe

En mode non classé, le joueur choisit **le niveau** (Fer à Challenger) et **la personnalité** de l'IA. La personnalité n'est pas un déguisement : elle agit à **quatre endroits** (la liste est celle que vous avez demandée : pick and ban, placement, jeu sur le plateau) :

1. **Choix du Roi** : préférences de Roi.
2. **Draft** : poids par classe d'unité, unités fétiches, habitudes de bans.
3. **Placement** : forme du dispositif (où est le Roi, qui est à côté).
4. **Jeu sur le plateau** : poids d'évaluation (attaque, danger, garde, terrain, immobilisation, mana), prise de risque, manière de choisir parmi des coups équivalents.

Une personnalité est donc un **vecteur de paramètres** ajouté à un niveau. Le niveau fixe *ce que l'IA sait faire*, la personnalité fixe *ce qu'elle préfère faire*.

### 11.2 Ce que la mesure dit des styles

| Duel (même niveau, profondeur 2, 300 parties) | Résultat |
|---|---|
| Équilibrée contre **Tortue pure** (Roi en coin, gardes collés) | **72,7 %** pour l'équilibrée |
| Équilibrée contre **Prudente** (tortue seulement quand elle mène aux PV) | 53,5 % pour l'équilibrée |
| **Prudente** contre **Tortue pure** | **72,5 %** pour la prudente |

- **S1 [M]** Un style **purement défensif** perd lourdement contre un style d'attaque de même niveau (−170 Elo). Le jeu récompense le tempo : s'enfermer ne protège pas, car l'adversaire n'a pas besoin de forcer un blocage (impossible, G17), il joue la zone de mort et les doubles menaces.
- **S2 [M]** La prudence **conditionnelle** (défendre quand on mène, attaquer sinon) est à peu près **aussi forte** que l'équilibre (46,5 %, dans la marge d'erreur de ±3 points).
- **S3 [R]** Une personnalité **défensive** jouable doit donc être une **défense active** : *contrôler la zone de mort adverse, retarder, contre-attaquer* ; pas « rester au fond ».
- **S4 [R]** Une personnalité ne doit pas changer la force de l'IA de plus de **±30 Elo** (sinon, choisir « Défensive » revient à choisir un niveau plus bas, ce qui trompe le joueur). Chaque personnalité est donc **calibrée** par tournoi (cible : 45–55 % entre deux personnalités de même niveau, avec un « pierre–feuille–ciseaux » léger : l'Offensive bat un peu la Prudente, la Contrôleuse bat un peu l'Offensive, etc.).

### 11.3 Les huit personnalités

| # | Nom | En une phrase |
|---|---|---|
| 1 | **Complète** | Équilibre attaque et défense, adapte son plan à la draft : le comportement « de référence » d'un niveau. |
| 2 | **Offensive** | Cherche la zone de mort au plus vite : tempo, rush, double menace. |
| 3 | **Défensive** | Défense active : contrôle la zone adverse, ne s'expose pas, punit celui qui entre. |
| 4 | **Prudente** | N'attaque que quand elle est sûre de ne pas se faire contrer ; mène aux PV puis verrouille. |
| 5 | **Anticipatrice** | Joue autour de ce que l'adversaire peut faire : bans prédictifs, plans à contre-plan, pièges de tempo. |
| 6 | **Contrôleuse** | Immobilisation, murs, pièges : fait perdre du temps, gagne par l'usure de tempo. |
| 7 | **Rusée** | Déplacements forcés et téléportations : Mentaliste, Arlequine, Diplomate, Voltigeuse, tirer le Roi. |
| 8 | **Imprévisible** | Joue des lignes variées parmi les coups quasi équivalents ; draft et placement tirés d'un large éventail. |

#### Paramètres par personnalité

Les valeurs sont des **points de départ** à ajuster par tournoi (§11.4). Le poids d'évaluation « attaque », « danger », « garde » reprend les réglages existants de l'IA (`Style`) ; « terrain » et « immobilisation » sont les réglages `EV.terr` et `EV.immoW` (§2).

| Personnalité | Attaque | Danger | Garde | Terrain | Poids immobilisation | Prise de risque |
|---|---|---|---|---|---|---|
| Complète | 1,0 | 1,1 | 0 | 2 | 0,5 | moyenne |
| Offensive | 1,4 | 0,8 | 0 | 4 | 0,25 | élevée |
| Défensive | 0,7 | 1,3 | 8 | 1 | 1,0 | faible |
| Prudente | 1,0 (0,5 si en tête) | 1,1 (1,5 si en tête) | 0 (12 si en tête) | 2 | 0,7 | faible si en tête |
| Anticipatrice | 1,0 | 1,2 | 0 | 2 | 0,6 | moyenne (profondeur +1 de plus que le niveau) |
| Contrôleuse | 0,8 | 1,1 | 4 | 2 | **1,4** (valorise ses immobilisations) | moyenne |
| Rusée | 1,1 | 1,0 | 0 | 3 | 0,4 | élevée sur les échanges |
| Imprévisible | 1,0 | 1,0 | 0 | 2 | 0,5 | variable (bruit de choix élargi) |

#### Roi, draft, bans, placement

| Personnalité | Rois préférés | Unités et classes favorites (draft) | Bans habituels | Placement |
|---|---|---|---|---|
| Complète | selon la force mesurée | meilleure valeur de composition (modèle) | l'unité la plus dangereuse pour son équipe | Roi central, 2 voisins |
| Offensive | Liche, Stratège (si niveau Diamant+) | Félin, Voltigeuse, Minotaure, Arlequine, Automate | contrôles qui arrêtent l'assaut : Geôlier, Trappeur, Colosse | Attaquants en face du Roi adverse, Roi central |
| Défensive | Bastion, Diplomate | Orque, Sanctuaire, Mentaliste, Geôlier, Guérisseur | unités de rush (Félin, Voltigeuse, Arlequine) | Roi central entouré d'Orque/Sanctuaire |
| Prudente | Bastion, Liche | équilibrée, évite les unités de timing risqué | unités qui ne se contrent pas facilement | Roi central, contrôleur à côté, un attaquant de chaque côté |
| Anticipatrice | selon l'adversaire (contre-Roi si possible) | contres de l'équipe adverse, flexibilité au dernier pick | **la signature** de l'adversaire (ses unités fréquentes) | J2 : répond au placement de J1 ; J1 : robuste |
| Contrôleuse | Bastion (pour rester libre), Diplomate | Trappeur, Geôlier, Colosse, Boréale, Prêtresse | Sanctuaire, Prêtresse adverses (qui annulent le contrôle) | Contrôleurs au centre, mur d'unités |
| Rusée | Diplomate, Liche | Arlequine, Mentaliste, Voltigeuse, Automate, Minotaure | Sanctuaire (annule les déplacements), Furtif | Arlequine à côté du Félin ou du Minotaure |
| Imprévisible | tirage large | tirage parmi les 8 meilleures options | tirage | choisit parmi les 5 meilleurs placements |

#### Comportement sur le plateau

- **Complète** : applique l'évaluation de base ; choisit **le plan** (rush/contrôle/défense) selon la draft.
- **Offensive** : lance le Liche/le Stratège **tôt** si cela donne la zone de mort ; accepte de perdre des pièces au contrôle pour garder le tempo.
- **Défensive** : garde toujours **au plus une** pièce adverse capable d'entrer dans la zone de mort ; place Orque et Sanctuaire avant d'avancer ; passe à l'offensive quand l'adversaire a engagé.
- **Prudente** : calcule l'avance de PV : quand elle a touché le Roi, elle verrouille (Guérisseur prêt, Orque, Sanctuaire) ; sinon attaque normalement.
- **Anticipatrice** : cherche systématiquement **le coup adverse le plus dangereux** et prépare la réponse (par exemple un Sanctuaire prêt le tour du Liche) ; son **profil adverse** (habitudes de la session) alimente ses bans et son placement.
- **Contrôleuse** : priorise la pose de pièges/murs dès le tour 1–2, tire les pièces adverses sur ses pièges, ne lance l'assaut que quand l'adversaire a ≥ 2 pièces immobilisées.
- **Rusée** : cherche les **échanges** et les **déplacements forcés** (tirer le Roi adverse avec la Mentaliste) ; coûteuse en mana, donc utilise le Catalyseur si elle l'a.
- **Imprévisible** : parmi les coups à moins de 15 % de la meilleure valeur, tire au hasard pondéré ; peut changer de plan à mi-partie.

### 11.4 Ce que le joueur voit et ce qu'il faut calibrer

- **Sélecteur** : « Niveau » (10 rangs) + « Personnalité » (8) ; une ligne de description (« Offensive : cherche la zone de mort au plus vite »). Pas d'Elo en mode non classé (déjà le cas).
- **Calibrage** (par tournoi, 300 parties par paire) : chaque personnalité contre la Complète au même niveau : écart toléré 45–55 %. Si trop faible : renforcer l'attaque/le contrôle ou réduire le danger pondéré ; si trop forte : l'inverse.
- **Test de personnalité** : sur 200 parties, vérifier que la personnalité **se reconnaît** (statistiques : colonne du Roi, classes draftées, tour du premier contact, taux de compétences lancées) ; sinon le paramètre n'a aucun effet visible.
- **Compatibilité avec le niveau** : une personnalité ne fait pas franchir au bot un concept qu'il ne connaît pas (la Rusée de niveau Fer ne tire pas le Roi) ; elle choisit parmi ce qu'il sait.

## 12. Le système d'Elo

### 12.1 Échelle et ancrage
- 10 paliers de **100 Elo** : Fer 0–99, Bronze 100–199, Argent 200–299, Or 300–399, Platine 400–499, Émeraude 500–599, Diamant 600–699, Maître 700–799, Grand Maître 800–899, Challenger 900+.
- Chaque IA est **ancrée au centre** de son palier (50, 150 … 950) : une IA Or vaut 350.
- Le joueur démarre à 0 (ou 100 si vous préférez un accueil plus doux) ; classé : **le joueur choisit le niveau de l'IA** affronté ; le gain dépend de l'écart.

### 12.2 Formule
Attendu E = 1 / (1 + 10^((Elo IA − Elo joueur)/400)) ; variation = K × (résultat − E), résultat 1 / 0,5 / 0.
K = **40** pendant les 10 premières parties, **24** ensuite, **16** au-dessus de 600 Elo.

Cette formule donne déjà ce que vous demandez : gagner contre une IA faible rapporte peu, perdre contre elle coûte cher, et inversement. Avec K = 24 :

**Joueur à 230 Elo**

| IA affrontée | Victoire attendue | Gain si victoire | Perte si défaite |
|---|---|---|---|
| Fer (50) | 74 % | +6,3 | −17,7 |
| Bronze (150) | 61 % | +9,3 | −14,7 |
| Argent (250) | 47 % | +12,7 | −11,3 |
| Or (350) | 33 % | +16,0 | −8,0 |
| Platine (450) | 22 % | +18,7 | −5,3 |
| Émeraude (550) | 14 % | +20,7 | −3,3 |
| Diamant (650) | 8 % | +22,0 | −2,0 |
| Maître (750) | 5 % | +22,9 | −1,1 |
| Grand Maître (850) | 3 % | +23,3 | −0,7 |
| Challenger (950) | 2 % | +23,6 | −0,4 |

**Joueur à 450 Elo** : contre Fer +2,2 / −21,8 ; Argent +5,8 / −18,2 ; Platine +12,0 / −12,0 ; Diamant +18,2 / −5,8 ; Challenger +22,7 / −1,3.
**Joueur à 700 Elo** : contre Fer +0,6 / −23,4 ; Or +2,8 / −21,2 ; Diamant +10,3 / −13,7 ; Grand Maître +16,9 / −7,1 ; Challenger +19,4 / −4,6.

### 12.3 Deux faiblesses de la formule pure, et leurs corrections
1. **Défaite contre un bien plus fort quasi gratuite** (−0,4 contre Challenger pour un joueur à 230) : on peut tenter les IA les plus fortes sans risque, mais le gain (+23) rend le « farm » de hauts niveaux rentable dès qu'une victoire arrive par chance. Correction : **gain contre une IA > 300 Elo au-dessus plafonné à +16** et **perte plancher de −3**.
2. **Victoires faciles contre des IA faibles** : correction **anti-farm** : contre une IA de plus de 200 Elo **sous** le joueur, le gain est nul (la perte reste complète).

### 12.4 Séries (Bo3)
Une Bo3 compte comme **une** partie Elo, avec un score attendu E_s = 3E² − 2E³. Exemple, joueur à 230 contre Argent (E = 0,47 → E_s = 0,457) : victoire +13,0 / défaite −11,0 ; contre Fer (E_s = 0,83) : +4,1 / −19,9 ; contre Or (E_s = 0,26) : +17,9 / −6,1.

### 12.5 « Challenger en deux semaines » : ce que la simulation montre
Simulation Monte-Carlo de 300 parties (appariement ±1 palier 60/20/20, K = 40/24/16) :

| Option | Description | Résultat |
|---|---|---|
| **A** | Force réelle des IA = ancrage (écarts uniformes de 100) | L'Elo du joueur converge vers sa **force réelle** en ≈ 100 parties. Un expert de force réelle 850 atteint Challenger dans 46 % des cas en 300 parties. |
| **B** | Force réelle des IA croissante (écarts 50, 150, 260, 380, 520, 680, 870, 1100, 1380, 1750) | **Plafond effectif** : un expert de force 850 reste Diamant (0 % Challenger) ; il faut une force de ≈ 1500. |

**Lecture.** Le temps pour atteindre un rang ne dépend pas de la formule mais de l'écart de **force réelle** entre paliers. Pour que Challenger soit quasi inatteignable (objectif LoL), il faut des écarts réels croissants **et** des IA hautes très fortes ; l'affichage reste à 100 points par palier.

### 12.6 Recommandation : option A+ (écarts réels croissants, affichage uniforme)
- Affichage : 100 points par palier, uniforme (lisible).
- Force réelle : chaque IA bat sa voisine inférieure dans **≈ 64 %** des parties jusqu'à Émeraude, **70–85 %** au-dessus (calibrage par tournoi round-robin et ajustement Bradley–Terry).
- Elo du joueur = Elo **affiché**, mis à jour contre l'Elo d'ancrage de l'IA, donc la montée devient naturellement plus lente en haut sans trucage.
- Alternative **B** (MMR caché plus un rang affiché par seuils) : plus flexible, mais moins transparent ; à n'envisager que si le test humain montre trop de progression.

**Cibles de population** (inspirées de LoL, après des mois de jeu) : sous Or ≥ 50 % ; Or–Platine ≈ 30 % ; Émeraude ≈ 10 % ; Diamant ≈ 4 % ; Maître et plus < 1 % ; Challenger ≈ 0,05 %.

### 12.7 Calibrage humain (protocole)
Vous jouez 20 parties par niveau ; cibles de victoires pour un joueur expert : Or ≈ 80 %, Platine ≈ 65 %, Émeraude ≈ 50 %, Diamant ≈ 30 %, Maître < 15 %, Grand Maître < 5 %, Challenger ≈ 0–1 %. Les niveaux sont ensuite ajustés (profondeur, budget, bruit) jusqu'à atteindre ces cibles.

### 12.8 Décisions ouvertes
1. Elo par partie ou par série (Bo3) ?
2. Valeurs de K (40/24/16) et plafonds (+16 / −3) ?
3. Départ à 0 ou à 100 ?
4. Option A+ ou B ?

## 13. Plan de réalisation (après votre validation)

| Étape | Contenu | Validation |
|---|---|---|
| 1 | **Moteur de recherche à budget de temps** (itératif, table de transposition, Web Worker) | Profondeur et temps par niveau conformes à §10.2 |
| 2 | **Modèle de draft** : tables Roi/unités/contres/synergies par niveau, bans prédictifs | Draft modèle ≥ 59 % contre draft aléatoire à niveau égal |
| 3 | **Placement guidé** (top-K sur 720) et utilisation du placement de J1 par J2 | ≥ 53 % contre placement aléatoire |
| 4 | **Dix niveaux** : connaissances (matrice §10.3) et bruit de bas niveau cohérent | Tournoi round-robin : écarts ≈ 64 % puis 70–85 % |
| 5 | **Huit personnalités** et sélecteur non classé | Chaque personnalité à 45–55 % de la Complète, reconnaissable |
| 6 | **Elo** (formule, K, anti-farm, Bo3) et calibrage humain | Vos 20 parties par niveau (§12.7) |

Aucune de ces étapes n'est lancée : j'attends vos décisions (§12.8) et vos retours sur la liste des personnalités.


## Annexe A — Données mesurées

### A.1 Rois (3 592 parties S2 contre S2, Rois et drafts aléatoires)

| Roi | Victoires | En J1 | En J2 | Coef. (log-odds) |
|---|---|---|---|---|
| Roi Liche | 59.4 % | 67.9 % | 50.6 % | +0.13 ± 0.02 |
| Roi Bastion | 55.7 % | 61.7 % | 49.6 % | +0.08 ± 0.02 |
| Roi Diplomate | 53.3 % | 57.0 % | 49.6 % | +0.04 ± 0.01 |
| Roi Marchand | 48.4 % | 55.0 % | 42.2 % | -0.02 ± 0.02 |
| Roi Stratège | 33.4 % | 42.8 % | 24.1 % | -0.24 ± 0.01 |

### A.2 Unités (force individuelle)

| Unité | Parties | Victoires | Coef. (log-odds) |
|---|---|---|---|
| Félin | 1741 | 57.0 % | +0.10 ± 0.02 |
| Automate | 1646 | 55.8 % | +0.09 ± 0.02 |
| Arlequine | 1662 | 54.6 % | +0.07 ± 0.02 |
| Trappeur | 1656 | 53.8 % | +0.05 ± 0.02 |
| Voltigeuse | 1659 | 53.4 % | +0.05 ± 0.02 |
| Minotaure | 1691 | 52.5 % | +0.04 ± 0.02 |
| Mentaliste | 1674 | 52.4 % | +0.03 ± 0.02 |
| Guérisseur | 1670 | 51.7 % | +0.03 ± 0.02 |
| Prêtresse | 1705 | 51.4 % | +0.02 ± 0.02 |
| Sanctuaire | 1678 | 49.2 % | -0.01 ± 0.02 |
| Geôlier | 1725 | 47.8 % | -0.03 ± 0.02 |
| Orque | 1693 | 47.7 % | -0.03 ± 0.02 |
| Boréale | 1710 | 47.1 % | -0.05 ± 0.01 |
| Furtif | 1688 | 46.1 % | -0.05 ± 0.02 |
| Colosse | 1681 | 45.9 % | -0.06 ± 0.02 |
| Catalyseur | 1688 | 44.4 % | -0.09 ± 0.02 |
| Messagère | 1769 | 39.9 % | -0.15 ± 0.02 |

### A.3 Synergies et anti-synergies entre alliés (z ≥ 2)

| Relation | Effet (log-odds) | z |
|---|---|---|
| Automate + Catalyseur | +0.21 ± 0.06 | +3.9 |
| Arlequine + Félin | +0.20 ± 0.06 | +3.2 |
| Guérisseur + Mentaliste | +0.17 ± 0.06 | +2.7 |
| Furtif + Catalyseur | -0.13 ± 0.05 | -2.5 |
| Sanctuaire + Orque | -0.15 ± 0.06 | -2.5 |
| Sanctuaire + Messagère | +0.13 ± 0.05 | +2.5 |
| Messagère + Trappeur | -0.13 ± 0.05 | -2.5 |
| Sanctuaire + Prêtresse | +0.14 ± 0.06 | +2.5 |
| Furtif + Colosse | +0.13 ± 0.05 | +2.4 |
| Minotaure + Furtif | -0.14 ± 0.06 | -2.3 |
| Automate + Boréale | -0.15 ± 0.07 | -2.2 |
| Sanctuaire + Mentaliste | -0.13 ± 0.06 | -2.2 |
| Automate + Orque | +0.14 ± 0.07 | +2.1 |
| Geôlier + Boréale | -0.11 ± 0.05 | -2.1 |
| Voltigeuse + Colosse | -0.12 ± 0.06 | -2.0 |
| Prêtresse + Félin | +0.12 ± 0.06 | +2.0 |

### A.4 Contres entre unités (positif = la 1re unité bat la 2e)

| Relation | Effet (log-odds) | z |
|---|---|---|
| Sanctuaire contre Trappeur | +0.18 ± 0.06 | +3.1 |
| Messagère contre Mentaliste | -0.17 ± 0.05 | -3.1 |
| Minotaure contre Guérisseur | +0.17 ± 0.06 | +2.9 |
| Automate contre Guérisseur | -0.17 ± 0.06 | -2.7 |
| Guérisseur contre Orque | +0.16 ± 0.06 | +2.7 |
| Minotaure contre Colosse | +0.16 ± 0.06 | +2.6 |
| Félin contre Trappeur | -0.17 ± 0.06 | -2.6 |
| Sanctuaire contre Prêtresse | -0.14 ± 0.05 | -2.6 |
| Voltigeuse contre Trappeur | +0.13 ± 0.05 | +2.5 |
| Automate contre Sanctuaire | +0.17 ± 0.07 | +2.5 |
| Automate contre Orque | +0.15 ± 0.06 | +2.4 |
| Catalyseur contre Geôlier | -0.13 ± 0.06 | -2.3 |
| Minotaure contre Arlequine | -0.13 ± 0.06 | -2.2 |
| Arlequine contre Furtif | -0.13 ± 0.06 | -2.1 |
| Félin contre Mentaliste | +0.12 ± 0.06 | +2.1 |
| Sanctuaire contre Arlequine | -0.14 ± 0.07 | -2.0 |

### A.5 Roi × unité alliée (positif = l’unité est meilleure avec ce Roi)

| Relation | Effet (log-odds) | z |
|---|---|---|
| Catalyseur avec Roi Diplomate | +0.20 ± 0.06 | +3.4 |
| Prêtresse avec Roi Diplomate | -0.16 ± 0.06 | -2.8 |
| Arlequine avec Roi Diplomate | +0.11 ± 0.04 | +2.7 |
| Catalyseur avec Roi Stratège | -0.13 ± 0.05 | -2.7 |
| Prêtresse avec Roi Bastion | +0.15 ± 0.06 | +2.6 |
| Messagère avec Roi Stratège | -0.14 ± 0.06 | -2.4 |
| Automate avec Roi Bastion | +0.11 ± 0.05 | +2.3 |
| Colosse avec Roi Diplomate | +0.11 ± 0.05 | +2.3 |
| Boréale avec Roi Diplomate | -0.11 ± 0.05 | -2.1 |
| Furtif avec Roi Stratège | -0.12 ± 0.06 | -2.0 |

### A.6 Roi contre unité adverse (positif = ce Roi gagne mieux contre cette unité)

| Relation | Effet (log-odds) | z |
|---|---|---|
| Roi Liche contre Automate | +0.25 ± 0.05 | +4.9 |
| Roi Stratège contre Voltigeuse | -0.20 ± 0.05 | -3.7 |
| Roi Marchand contre Automate | -0.18 ± 0.06 | -3.1 |
| Roi Marchand contre Messagère | +0.19 ± 0.07 | +2.8 |
| Roi Diplomate contre Félin | +0.14 ± 0.05 | +2.5 |
| Roi Stratège contre Geôlier | -0.12 ± 0.05 | -2.4 |
| Roi Stratège contre Guérisseur | -0.13 ± 0.06 | -2.3 |
| Roi Liche contre Prêtresse | +0.13 ± 0.06 | +2.1 |
| Roi Marchand contre Félin | -0.11 ± 0.06 | -2.1 |

### A.7 Roi contre Roi (positif = le 1er Roi bat le 2e)

| Relation | Effet (log-odds) | z |
|---|---|---|
| Roi Liche contre Roi Diplomate | +0.16 ± 0.05 | +3.2 |
| Roi Diplomate contre Roi Bastion | +0.15 ± 0.05 | +3.1 |
| Roi Bastion contre Roi Stratège | +0.16 ± 0.06 | +2.8 |
| Roi Liche contre Roi Bastion | -0.12 ± 0.05 | -2.4 |
| Roi Diplomate contre Roi Marchand | +0.09 ± 0.04 | +2.1 |
| Roi Liche contre Roi Stratège | +0.11 ± 0.06 | +1.9 |
| Roi Marchand contre Roi Bastion | +0.05 ± 0.05 | +1.0 |
