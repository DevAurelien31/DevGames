# Analyse des unités et des Rois — v1 (22 nov. 2026)

Source : 2 200 parties IA S2 contre S2 (règles actuelles). Version tableur : `duel/docs/Analyse_unites_v1.xlsx`.

## Tableau par unité

### Roi Cataclysmique (Roi)
- **Compétence** : 1 fois par partie : immobilise tous les ennemis (sauf le Roi).
- **Coût** 2 · portée Globale · cooldown Non · passif Non · usage unique Oui
- **Rendement** : 56.7 % de victoires (898 parties), 1 compétence(s) par partie → **Fort – à surveiller**
- **Remarque** : Le plus fort (56,7 %). Immobiliser toute l'armée adverse un tour (usage unique, coût 2) est un coup de tempo décisif pour un rush de 2 attaques sur le Roi. Sa matrice est favorable : 53 % contre Indomptable, 67 % contre Stratège, 60 % contre Économiste, 54 % contre Diplomate. Aucun Roi ne le contre nettement. Intéressant : décision de timing, il se joue une seule fois.
- **Idée de modification** : Surveiller. Si >57 % chez des humains : coût 3, ou ne pas affecter les ennemis déjà adjacents à votre Roi (effet plus défensif, moins décisif en attaque).
- **Utilité** : Roi « all-in » : un tour de paralysie pour terminer la partie. Très fort en 2e joueur après avoir vu le placement.
- **5 noms** : Cataclysme · Glaciation · Fléau · Apocalypse · Liche

### Roi Diplomate (Roi)
- **Compétence** : Échange la place de deux de vos alliés (hors Roi).
- **Coût** 3 · portée Globale · cooldown Non · passif Non · usage unique Non
- **Rendement** : 53.6 % de victoires (880 parties), 1.36 compétence(s) par partie → **Équilibré**
- **Remarque** : 53,6 % : bien équilibré. Seul Roi à compétence répétable (coût 3). Permet de téléporter une unité devant (rush), de sauver une unité immobilisée ou de réorganiser la défense. Très riche pour un joueur expérimenté.
- **Idée de modification** : Rien à changer. Si >55 % : recharge 1 tour sur 2 ou impossibilité d'échanger une unité qui vient d'être déplacée.
- **Utilité** : Roi « mobilité / ruse » : contourne murs, pièges et lignes de vue en échangeant deux alliés.
- **5 noms** : Émissaire · Chronos · Horloger · Sablier · Médiateur

### Roi Économiste (Roi)
- **Compétence** : Passif : réserve de mana de 7 ; +4 mana un tour sur deux, +3 les autres tours.
- **Coût** 0 · portée Globale · cooldown Non · passif Oui · usage unique Non
- **Rendement** : 46.3 % de victoires (861 parties), 0 compétence(s) par partie → **Faible**
- **Remarque** : 46 % : le mana n'est pas contraignant (plafond 6, +3/tour, une attaque par pièce), donc 7 de réserve et +4 un tour sur deux apportent peu. Aucune décision. Il donne pourtant une vraie identité (« Banquier »).
- **Idée de modification** : Transformer l'excédent de mana en action : 3 mana = une attaque ou un déplacement supplémentaire pour une pièce (1 fois par tour). Ou plafond 8 et compétences des unités à −1 coût (il faut alors vérifier Séisme et Chargeur).
- **Utilité** : Roi d'attrition : permet de lancer plusieurs compétences coûteuses dans le même tour (Sauteur + Illusionniste + Chargeur).
- **5 noms** : Marchand · Banquier · Trésorier · Intendant · Fortuné

### Roi Indomptable (Roi)
- **Compétence** : Passif : vos unités (hors Roi) ignorent leur première immobilisation.
- **Coût** 0 · portée Globale · cooldown Non · passif Oui · usage unique Non
- **Rendement** : 56.1 % de victoires (841 parties), 0 compétence(s) par partie → **Fort – à surveiller**
- **Remarque** : Passif sans décision : les 4 unités ignorent leur 1re immobilisation. Très solide (56 %), surtout contre le Roi Stratège (74 % de victoires pour lui) et contre les équipes basées sur l'immobilisation (Séisme, Mage de chaîne, pièges). Contre Cataclysmique c'est quasi égal (47 % pour lui) : les boucliers annulent le gel, mais Cataclysmique reste fort. Défaut compétitif : il ne demande aucun choix, donc son niveau dépend uniquement de la composition adverse (bans).
- **Idée de modification** : Garder tel quel mais à surveiller (>55 %). Si trop fort : le bouclier ne couvre que 3 unités sur 4 (le joueur désigne celle qu'il ne protège pas au placement) ; ou il ne protège pas des pièges (le Trappeur reste utile).
- **Utilité** : Roi « de stabilité » : permet une approche agressive sans craindre le contrôle. À prendre contre les compositions de contrôle.
- **5 noms** : Imprenable · Inflexible · Bastion · Invaincu · Rempart

### Roi Stratège (Roi)
- **Compétence** : Passif : +4 mana par tour. Compétence (1 fois par partie, coût 1) : tous vos alliés gagnent +1 déplacement ce tour.
- **Coût** 1 · portée Globale · cooldown Non · passif Non · usage unique Oui
- **Rendement** : 37.5 % de victoires (920 parties), 1 compétence(s) par partie → **Très faible**
- **Remarque** : Le plus faible (37,5 %). Double défaut : (1) +4 mana/tour est quasi inutile car le mana semble peu limitant (en moyenne ≈ 4 mana restent inutilisés en fin de tour ; hypothèse à confirmer) ; (2) « +1 déplacement » pénalise celui qui approche : il est frappé le premier. La compétence usage unique à coût 1 est correcte mais insuffisante.
- **Idée de modification** : Remplacer le passif : « Marche forcée » = +1 déplacement ET immunité aux immobilisations jusqu'au tour adverse suivant (tour de rush sûr). Variante : à la place du mana, les unités alliées entrent en jeu avec 1 déplacement supplémentaire au 1er tour. À tester par simulation (cible 47-52 %).
- **Utilité** : Roi « tempo » : prépare un tour d'assaut coordonné. Aujourd'hui trop dépendant du bon timing et du mana superflu.
- **5 noms** : Tacticien · Général · Maréchal · Capitaine · Conquérant

### Aimant (Tank)
- **Compétence** : Attire d'une case vers lui tous les ennemis situés à 2 cases.
- **Coût** 2 · portée 2 cases · cooldown Non · passif Non · usage unique Non
- **Rendement** : 51.2 % de victoires (1051 parties), 0.69 compétence(s) par partie → **Équilibré**
- **Remarque** : 51,2 %. Attire d'une case tous les ennemis situés EXACTEMENT à 2 cases en ligne droite (vue dégagée). Il ne fonctionne donc pas sur les « cavaliers » (1 de côté, 2 de haut) ni à travers une pièce. Peu lancé (0,7/partie) mais bon quand il sert : il déstructure la formation adverse et amène les cibles dans la zone du Séisme, d'un piège ou du Chargeur.
- **Idée de modification** : Option si <48 % chez des humains : attire aussi les ennemis à 3 cases en ligne droite. Sinon inchangé.
- **Utilité** : Outil de synergie : amène les ennemis là où vos autres unités veulent les frapper (Séisme, piège, Chargeur) et les sort de leur formation.
- **5 noms** : Magnétron · Polaris · Attireur · Gravitor · Électro

### Chargeur (Tank)
- **Compétence** : Fonce sur un ennemi adjacent (diagonales comprises), le repousse d'une case dans la même direction et prend sa place.
- **Coût** 3 · portée 1 case · cooldown Non · passif Non · usage unique Non
- **Rendement** : 50.3 % de victoires (1051 parties), 0.55 compétence(s) par partie → **Équilibré**
- **Remarque** : 50,3 % : équilibré. Avance de 1-2 cases, repousse l'ennemi et prend sa place : il défait un rush (repousse l'attaquant du Roi) et crée de l'espace. Coût 3 élevé donc choix de timing. Limite : échoue si la case derrière est occupée, donc moins efficace en fond de plateau.
- **Idée de modification** : Aucune pour l'instant. Surveiller la synergie avec le Trappeur (un ennemi repoussé sur un piège est immobilisé).
- **Utilité** : Outil de tempo et d'anti-rush : gagne une case ET en fait perdre une à l'adversaire.
- **5 noms** : Minotaure · Bélier · Taureau · Fonceur · Cogneur

### Provocateur (Tank)
- **Compétence** : Les ennemis adjacents doivent le viser s'ils attaquent ce tour-là.
- **Coût** 2 · portée 1 case · cooldown Non · passif Non · usage unique Non
- **Rendement** : 45.7 % de victoires (1012 parties), 0.27 compétence(s) par partie → **Faible**
- **Remarque** : 45,7 % mais probablement sous-estimé : l'IA le lance 0,27 fois/partie. Entre humains il sert de défense de clutch : les ennemis adjacents qui veulent attaquer doivent viser le Provocateur, donc peuvent ne pas frapper le Roi. Il ne restreint pas les compétences de zone ni les unités non adjacentes.
- **Idée de modification** : À valider avec des humains. Si réellement faible : coût 1 ; ou la provocation dure 2 tours ; ou elle s'étend aux ennemis à distance 2.
- **Utilité** : Défense du Roi : les ennemis adjacents sont forcés de frapper le Provocateur plutôt que le Roi ; base d'un jeu défensif.
- **5 noms** : Railleur · Bravache · Hurleur · Défi · Gladiateur

### Séisme (Tank)
- **Compétence** : Immobilise tous les ennemis adjacents.
- **Coût** 2 · portée 1 case · cooldown Non · passif Non · usage unique Non
- **Rendement** : 46.8 % de victoires (1051 parties), 0.73 compétence(s) par partie → **Faible**
- **Remarque** : 46,8 %. Pour 2 mana, immobilise tous les ennemis adjacents : mais deux attaques de base coûtent aussi 2 mana et font la même chose avec deux pièces. Il n'est supérieur qu'à partir de 3 adjacents ou quand les autres pièces sont loin. Il doit être au contact des ennemis pour servir.
- **Idée de modification** : Coût 1 pour une zone vraiment rentable ; ou immobilise ET empêche l'attaque de base des ennemis adjacents ce tour (vrai contrôle). À tester les deux.
- **Utilité** : Verrou local : transforme une mêlée en piège ; très utile pour geler un groupe devant le Roi.
- **5 noms** : Secousse · Colosse · Golem · Basalte · Titan

### Mage de chaîne (Mage)
- **Compétence** : Immobilise un ennemi à portée 2, et un second ennemi adjacent au premier.
- **Coût** 2 · portée 2 cases (ligne de vue) · cooldown Non · passif Non · usage unique Non
- **Rendement** : 46.6 % de victoires (1050 parties), 2.26 compétence(s) par partie → **Faible**
- **Remarque** : 46,6 %. Pour 2 mana immobilise un ennemi à portée 2 plus un ennemi adjacent : similaire à deux attaques de base (2 mana) mais depuis 2 cases et sans que l'unité soit au contact. Utile contre des unités groupées. Le rebond obligatoire peut toucher un ennemi inutile.
- **Idée de modification** : Coût 1 si l'impact réel reste <46 % ; ou deux cibles de choix à portée 2 (plus libre). À tester.
- **Utilité** : Contrôle de groupe à distance, sûr car hors de portée de la mêlée.
- **5 noms** : Foudre · Tonnerre · Orageux · Électron · Voltaïque

### Mur de Glace (Mage)
- **Compétence** : Pose un mur sur une case libre à portée 2. Il bloque le passage et la vue, et disparaît au début de votre 2e tour suivant.
- **Coût** 2 · portée 2 cases · cooldown Non · passif Non · usage unique Non
- **Rendement** : 47.3 % de victoires (1035 parties), 7.63 compétence(s) par partie → **Faible**
- **Remarque** : 47,3 %, mais l'IA le lance 7,6 fois/partie (puits à mana). Humain : très fort pour bloquer une ligne de sprint, de saut, de vue (Mage de chaîne, Aimant, Télékinésiste) ou protéger un couloir. Le mur ne peut jamais fermer le dernier chemin vers le Roi (règle d'atteignabilité).
- **Idée de modification** : Inchangé. Si les tests humains montrent du camping défensif : durée réduite à 1 tour. Pas de buff : le mur deviendrait trop présent.
- **Utilité** : Contrôle de l'espace : ouvre/ferme des lignes ; excellent contre les unités à ligne droite (Sprinteur, Chargeur, Aimant).
- **5 noms** : Glacier · Givre · Banquise · Muraille · Frimas

### Télékinésiste (Mage)
- **Compétence** : Déplace un ennemi à portée 2 sur une case libre adjacente à lui, puis l'immobilise. Recharge : 1 tour sur 2.
- **Coût** 3 · portée 2 cases (ligne de vue) · cooldown Oui · passif Non · usage unique Non
- **Rendement** : 49.8 % de victoires (1035 parties), 1.93 compétence(s) par partie → **Équilibré**
- **Remarque** : 49,8 % : équilibré (autrefois 56 %). Pour 3 mana : déplace un ennemi à 2 cases de 1 case (sur case libre) et l'immobilise. Peut sortir un attaquant de la case adjacente au Roi, ou le jeter dans un piège. Recharge d'un tour sur deux : il faut planifier.
- **Idée de modification** : Aucune pour le moment. Surveiller si le cooldown est retiré.
- **Utilité** : Contrôle précis à distance : réponse flexible à un attaquant du Roi.
- **5 noms** : Télékine · Psychique · Mentaliste · Lévitant · Esprit

### Trappeur (Mage)
- **Compétence** : Pose 2 pièges sur des cases libres à portée 2. Une pièce qui entre sur un piège est immobilisée. Un nouveau lancer retire les anciens pièges.
- **Coût** 2 · portée 2 cases · cooldown Non · passif Non · usage unique Non
- **Rendement** : 55.2 % de victoires (1025 parties), 3.9 compétence(s) par partie → **Fort – à surveiller**
- **Remarque** : 55,2 % : fort et très utilisé (3,9 lancers/partie). Deux pièges persistants (jusqu'au prochain lancer) immobilisent TOUTE pièce qui entre, alliée comprise, y compris par déplacement forcé ; ils arrêtent sprint et charge. Zone de déni permanente pour 2 mana, très difficile à contourner pour un joueur qui ne connaît pas le terrain.
- **Idée de modification** : Tester d'abord une durée de vie de 3 tours pour les pièges (usure), sinon un seul piège actif à la fois.
- **Utilité** : Contrôle de zone permanent : protège un couloir vers le Roi et menace de gels à distance.
- **5 noms** : Piégeur · Chasseur · Rôdeur · Braconnier · Ranger

### Anti-CC (Support)
- **Compétence** : Lui-même et ses alliés adjacents ne peuvent être ni immobilisés ni déplacés pendant le prochain tour adverse.
- **Coût** 2 · portée 1 case · cooldown Non · passif Non · usage unique Non
- **Rendement** : 54.2 % de victoires (1023 parties), 5.14 compétence(s) par partie → **Fort – à surveiller**
- **Remarque** : 54,2 % (5,1 lancers/partie). Réponse à tout le contrôle : l'unité et ses alliés adjacents ne peuvent être ni immobilisés ni déplacés. Très fort en réponse, y compris contre Cataclysmique (si l'unité est adjacente aux autres). À surveiller : l'IA le lance quasi chaque tour, ce qui est sûrement moins rentable pour un humain.
- **Idée de modification** : Surveiller. Si trop fort : recharge 1 tour sur 2 ou ne protège que lui-même + 1 allié.
- **Utilité** : Bouclier de zone : permet à un groupe d'avancer sans craindre le contrôle ; bon choix contre les équipes de contrôle (Séisme, Mage de chaîne, Télékinésiste).
- **5 noms** : Égide · Barrière · Immuable · Sanctuaire · Garde

### Gardien du Roi (Support)
- **Compétence** : 1 fois par partie : votre Roi regagne 1 PV.
- **Coût** 2 · portée Globale · cooldown Non · passif Non · usage unique Oui
- **Rendement** : 52.4 % de victoires (1041 parties), 0.54 compétence(s) par partie → **Équilibré**
- **Remarque** : 52,4 % : équilibré. Rend 1 PV au Roi (usage unique, coût 2). Avec 2 PV, le soin vaut la moitié de la vie du Roi (3 touches nécessaires au lieu de 2). Il n'est utilisable qu'après une touche : décision de timing intéressante. Il ne protège pas d'une double touche dans le même tour.
- **Idée de modification** : Garder tel quel. Ne pas autoriser le soin en réaction à une touche mortelle : trop puissant.
- **Utilité** : Résilience du Roi : rend le jeu moins punissant ; gagne un tour supplémentaire pour la défense.
- **5 noms** : Veilleur · Valkyrie · Soigneur · Sentinelle · Protecteur

### Invocateur d'Énergie (Support)
- **Compétence** : Gagne 2 mana (coût 1, donc +1 net). Recharge : 1 tour sur 2.
- **Coût** 1 · portée Globale · cooldown Oui · passif Non · usage unique Non
- **Rendement** : 47.2 % de victoires (992 parties), 4.36 compétence(s) par partie → **Faible**
- **Remarque** : 47,2 %, trop lancé par l'IA (4,4/partie) pour un gain net de +1 mana. Si le mana est peu limitant (hypothèse à confirmer), +1 mana est faible. Il manque une identité compétitive.
- **Idée de modification** : Transformer en soutien d'action : coût 1, un allié choisi attaque gratuitement ce tour, ou gagne +1 déplacement au prochain tour. À tester.
- **Utilité** : Support d'économie : permet des tours à 3 compétences si le Roi Économiste est là.
- **5 noms** : Invocateur · Catalyseur · Mystique · Source · Médium

### Messager (Support)
- **Compétence** : Un allié de votre choix (où qu'il soit) gagne +1 déplacement ce tour.
- **Coût** 2 · portée Globale · cooldown Non · passif Non · usage unique Non
- **Rendement** : 40.1 % de victoires (1045 parties), 3.79 compétence(s) par partie → **Très faible**
- **Remarque** : 40,1 % : le plus faible des unités. +1 déplacement pour un seul allié est faible : le joueur qui avance trop tôt est frappé le premier. Tests antérieurs (±3,6 pts) : variante « Alerte » (alliés adjacents : anti-contrôle + attaque gratuite) +9 points ; variante « Relais » (échange avec un allié ≤3 cases, +1 déplacement) +7 points.
- **Idée de modification** : Appliquer la variante Alerte (ou Relais) : c'est le meilleur gain mesuré, déjà codé (RULES.x.msgAlert / msgRelay). Re-simuler avant validation.
- **Utilité** : Soutien de coordination : prépare l'assaut ou protège un groupe.
- **5 noms** : Héraut · Coursier · Estafette · Éclaireur · Relais

### Purificateur (Support)
- **Compétence** : Libère un de vos alliés immobilisés. Recharge : 1 tour sur 2.
- **Coût** 2 · portée Globale · cooldown Oui · passif Non · usage unique Non
- **Rendement** : 50.6 % de victoires (992 parties), 1.82 compétence(s) par partie → **Équilibré**
- **Remarque** : 50,6 % : équilibré après les ajustements. Libère 1 allié immobilisé (coût 2, recharge). Réponse directe au contrôle. L'immobilisation ne bloquant que le déplacement, sa valeur est surtout de remettre une unité en route.
- **Idée de modification** : Garder. Pas d'immunité automatique (déjà décidé).
- **Utilité** : Support anti-contrôle ; rend l'équipe moins vulnérable à Mage de chaîne et Séisme.
- **5 noms** : Purifieur · Prêtresse · Libérateur · Oracle · Exorciste

### Illusionniste (Assassin)
- **Compétence** : Échange sa place avec une unité adjacente (hors Roi).
- **Coût** 3 · portée 1 case · cooldown Non · passif Non · usage unique Non
- **Rendement** : 52.6 % de victoires (1043 parties), 0.65 compétence(s) par partie → **Équilibré**
- **Remarque** : 52,6 % : équilibré (autrefois 57-59 %). Échange sa place avec une unité adjacente (alliée ou ennemie, hors Roi). Peut téléporter un ennemi dans un piège ou sortir un allié d'un mauvais contact. Coût 3 élevé donc choix de timing.
- **Idée de modification** : Aucune. Surveiller : sa combinaison avec Trappeur (piège).
- **Utilité** : Ruse et repositionnement : échanger un attaquant dangereux avec un allié protégé.
- **5 noms** : Faussaire · Mirage · Trompeur · Illusion · Joker

### Insaisissable (Assassin)
- **Compétence** : Ne peut pas être ciblé par l'adversaire pendant son prochain tour. Recharge : 1 tour sur 2.
- **Coût** 1 · portée Soi-même · cooldown Oui · passif Non · usage unique Non
- **Rendement** : 48.5 % de victoires (1058 parties), 2.91 compétence(s) par partie → **Équilibré**
- **Remarque** : 48,5 %. Coût 1, intargetable jusqu'à son prochain tour, recharge. Il se protège lui-même seulement : il avance donc sans craindre aucun contrôle un tour sur deux. Utile en approche du Roi ; peu d'impact sinon.
- **Idée de modification** : Renforcer l'identité : pendant son tour intargetable, il peut traverser les autres pièces (fantôme). À tester.
- **Utilité** : Infiltrateur : approche du Roi sous protection ; peut recevoir les soutiens alliés (Messager, Purificateur), car seul l'adversaire ne peut pas le cibler.
- **5 noms** : Furtif · Spectre · Fantôme · Ombre · Évanescent

### Sauteur (Assassin)
- **Compétence** : Saute par-dessus une pièce adjacente (diagonales comprises) vers la case libre derrière. Il est immobilisé à son prochain tour.
- **Coût** 3 · portée 1 case · cooldown Non · passif Non · usage unique Non
- **Rendement** : 55.3 % de victoires (1051 parties), 0.35 compétence(s) par partie → **Fort – à surveiller**
- **Remarque** : 55,3 % (autrefois 62-64 %, maintenant acceptable après coût 3 et immobilisation à l'atterrissage). Saute par-dessus une pièce adjacente : contourne murs et pièges. Bonne unité de percée, mais exposée : immobilisée au tour suivant.
- **Idée de modification** : Rien. Surveiller : si le taux remonte, recharge 1 tour sur 2.
- **Utilité** : Percée et contournement : fait passer les défenses ; difficile à contrer sans anticipation.
- **5 noms** : Acrobate · Voltigeur · Funambule · Bondisseur · Cabriole

### Sprinteur (Assassin)
- **Compétence** : Passif : avance de 2 cases en ligne droite (sans traverser de pièce) un tour sur 3 ; les autres tours, 1 case.
- **Coût** 0 · portée 2 cases en ligne droite · cooldown Non · passif Oui · usage unique Non
- **Rendement** : 55.1 % de victoires (1045 parties), 0 compétence(s) par partie → **Fort – à surveiller**
- **Remarque** : 55,1 % : fort pour un passif gratuit. Sprint de 2 cases en ligne droite obligatoire un tour sur 3. La portée de menace est de 3 cases le tour du sprint (2 + attaque). Très gênant : il force la défense à réagir tôt. Souvent banni dans la méta.
- **Idée de modification** : Option si nécessaire : pas d'attaque de base le tour d'un sprint de 2 cases (testé : 55 → 52 % à l'époque) ; ou un tour sur 4.
- **Utilité** : Menace de rush : oblige l'adversaire à garder une unité en retrait ou à poser un mur/piège sur sa ligne.
- **5 noms** : Ninja · Coureur · Éclair · Fulgurant · Filant

## Avis global

- **Verdict en une phrase** : Le concept est solide et mérite d'être figé SI quatre points sont levés : l'avantage du premier joueur (56 % en simulation), le mana qui ne limite pas vraiment le jeu, les 4 unités/Rois nettement faibles (Stratège 37,5 %, Messager 40 %, Économiste 46 %, Invocateur 47 %) et le risque de blocage défensif. Le jeu n'est ni trop simple ni lassant sur le papier, mais deux inconnues décisives restent à mesurer avec des humains : la méta de draft et le jeu défensif.

- **Ce que sont les chiffres** : 2 200 parties IA contre IA (IA de recherche S2, draft aléatoire, placement guidé, règles actuelles). Chaque unité apparaît dans ≈1 000 parties : incertitude ±1,6 point (95 %). Les chiffres mesurent la force dans le jeu d'une IA qui joue à profondeur limitée : ils sont utiles pour classer, pas pour trancher des cas proches (±3 points).

- **Ce qui fait un bon jeu compétitif : grille d'évaluation (note sur 5)** : 1) Information parfaite et zéro aléa : 5/5. Tout se joue sur la décision (comme aux échecs), donc le classement Elo est légitime.
2) Écart de niveau (skill gap) : 4/5. Mesuré : L1 bat L0 100 %, L2 bat L1 76 %, S2 bat L2 86 %, S3 bat S2 62 %. Chaque niveau de réflexion supplémentaire gagne clairement : le jeu récompense la profondeur, signe de bonne profondeur.
3) Équité des côtés : 3/5. J1 gagne 56,1 % (J2 43,8 %, nuls 0,1 %). Les échecs ont ≈ 55 % pour les Blancs : c'est tolérable, mais on peut faire mieux (voir « À vérifier »).
4) Diversité de la méta : 3/5. Unités de 40 % à 55 %, Rois de 37 % à 57 % : un Roi (Stratège) est quasi injouable, trois unités sont dominées. Cible : 45-55 % partout.
5) Lisibilité : 3,5/5. 22 personnages, 7 statuts, plusieurs règles fines (atteignabilité du Roi, immunité). Bonne avec les icônes et la carte d'info, mais la courbe d'apprentissage est réelle.
6) Rythme : 4/5. Partie médiane de 9 tours par joueur (4 à 40), ≈ 106 actions ; 60 s par tour. Court, mobile-friendly.
7) Rejouabilité : 4/5 en théorie (5 Rois × 17 unités, 2 380 équipes de 4 possibles, draft avec bans, placement asymétrique) mais à confirmer par une méta réelle.
8) Risque de blocage (camping) : 3/5. Voir « À vérifier ».

- **Compétitivité** : Points forts : pas d'aléa ; draft/bans à 2 phases (A-B-B-A puis B-A-A-B) qui donne un vrai jeu avant le jeu ; placement non simultané (J2 voit la ligne de J1) qui compense en partie l'avantage de J1 ; écart de niveau mesuré. Points faibles : le premier joueur garde 56 % ; aucune unité n'est « éliminée » (pas de capture) donc pas de compensation matérielle : une erreur se paie en tempo, et un tempo perdu est dur à récupérer (les parties durent 9 tours et la 1re touche sur un Roi arrive en moyenne au tour 8,5). Un joueur qui perd l'initiative peut n'avoir aucun retour.

- **Rejouabilité** : Les 5 Rois ont des identités claires (bouclier, mana, tempo, gel de masse, échange). Le nombre de compositions est très grand. Mais une méta se forme vite : dans les simulations précédentes Purificateur et Sprinteur étaient presque toujours bannis et Messager jamais pris. Si 5 à 6 unités dominent les bans, la rejouabilité réelle tombe à ≈ 12 unités. Priorité : amener Messager, Stratège, Économiste, Invocateur à un niveau jouable.

- **Trop simple ?** : Non. La règle de base est simple (1 déplacement, 1 attaque, 1 compétence), mais l'espace de décisions est riche : chaque pièce a 3 types d'action, la règle d'atteignabilité interdit l'emmurement, les compétences se combinent (Aimant + Séisme, Trappeur + Chargeur, Diplomate + Sauteur). Le risque est plutôt le contraire : trop de statuts à retenir pour un débutant. Un tutoriel progressif (3 à 5 unités d'abord) est conseillé.

- **Trop lassant ?** : Risque réel dans la 1re moitié : la 1re touche sur un Roi arrive en moyenne au tour 8,5 pour une partie médiane de 9 tours. Pendant ≈ 7 tours, rien n'est « touché » : les joueurs se positionnent. Pour garder la tension : (1) rendre visibles les menaces (cases attaquables au tour suivant) dans l'interface ; (2) envisager un rythme plus rapide (placement initial plus avancé, ou un déplacement supplémentaire le 1er tour) à tester ; (3) garder la limite de tour pour éviter le blocage.

- **Axes d'amélioration prioritaires** : 1) Ramener J1 à ≈ 53-54 % : jeton de J2 à 3 mana (54,5 % mesuré avant), ou parties classées par paires avec côtés inversés (supprime le biais en classé).
2) Vérifier si le mana est assez contraignant : en moyenne ≈ 4 mana restent inutilisés en fin de tour (64 % des tours ≥ 3), mais 28 % des tours ont une action bloquée par le mana. Image mitigée : tester +2/tour et plafond 5 avant de conclure.
3) Refaire Stratège, Messager, Économiste, Invocateur (voir feuille Unités). Le défaut commun : leur effet est du « mana » ou « +1 déplacement », deux effets faibles dans ce jeu.
4) Valider avec des humains les unités défensives sous-utilisées par l'IA (Provocateur, Mur de Glace, Anti-CC).
5) Ajouter de la télémétrie anonyme (draft, placement, résultat) pour calibrer sans dépendre de l'IA.
6) Mettre en place des « saisons » : retirer/remplacer une unité, ou ajuster les coûts, une fois la méta stabilisée.

- **Ce que je ferais avant de figer** : Appliquer (sur le moteur, en une passe) : Messager = variante Alerte ; Stratège = marche forcée + immunité ; Économiste = conversion de mana ; Invocateur = tempo ; puis re-simuler 4 000 parties, puis un test avec 20-30 humains sur 200 parties. Figer seulement quand toutes les unités sont entre 45 et 55 % et que J1 est ≤ 54 %.

## À vérifier

- **Avantage du premier joueur** : 56,1 % en simulation (2 200 parties, ±2,1). Équité faible pour un jeu « ultra compétitif ». — Jeton J2 à 3 mana ; classé en 2 manches côtés inversés ; mesurer avec humains (200 parties).

- **Mana : contrainte réelle ?** : mesuré (120 parties IA) ≈ 4 mana inutilisés en fin de tour ; 64 % des tours finissent avec ≥ 3 mana ; 28 % des tours ont une action bloquée par le mana. Les unités « mana » sous-performent, mais la cause (mana peu utile ou jeu de l'IA) n'est pas prouvée. — Faire varier plafond/régénération et comparer victoires et mana inutilisé ; confirmer avec des humains.

- **Camping / blocage défensif** : Le jeu n'a pas de capture : on peut seulement gêner. Les tests anciens avec un bot « tortue » montraient des fins par limite de tours (36 % entre tortues). — Simuler S2/S3 contre une IA tortue forte ; durée médiane, nuls, taux de limite de tours (actuellement 0,2 % en miroir).

- **Déséquilibre entre Rois (pas de contre-pick possible : choix simultané)** : les Rois sont choisis en même temps (doublons possibles), donc c'est un problème d'équilibre global. Matrice mesurée : Indomptable bat Stratège 74 % ; Cataclysmique bat Stratège 67 %, Économiste 60 % ; Stratège perd contre tous (25-41 %). — Après refonte de Stratège/Économiste, refaire la matrice 5×5 (≥ 150 parties par case) : objectif toutes les cases entre 40 et 60 %.

- **Biais de l'IA sur certaines unités** : Mur de Glace 7,6 lancers/partie, Anti-CC 5,1, Invocateur 4,4 mais Provocateur 0,27 : l'IA ne joue pas comme un humain. — Télémétrie en ligne + parties de test avec joueurs ; comparer taux de pick, de bans et de victoire.

- **Méta de draft** : Avec 17 unités mais seulement 4 choisies, 5 à 6 unités risquent de dominer les bans (Sprinteur, Purificateur, Sauteur, Trappeur, Anti-CC). — Simuler la draft avec une IA qui joue bans/picks réalistes ; mesurer la diversité des équipes sur 1 000 parties.

- **Rythme et première touche** : La 1re touche sur un Roi arrive en moyenne au tour 8,5. Risque de longue phase d'approche sans tension. — Tester un 1er tour plus actif ; ajouter une vue « menaces au prochain tour » (cases attaquables).

- **Règle d'atteignabilité du Roi** : Règle essentielle mais complexe : un couloir étroit reste possible. Action illégale = joueur bloqué. — Sur échantillon de parties humaines : fréquence des actions refusées ; message d'explication clair.

- **Mort subite et limite de tours** : Seules 13 parties sur 2 200 vont à la limite ou en mort subite : bon, mais à revérifier si on ajoute des unités défensives. — Garder en indicateur automatique de chaque simulation.

- **Lisibilité mobile** : 22 personnages, 7 statuts, plusieurs règles fines. — Test d'onboarding : un nouveau joueur doit gagner une partie contre IA facile en <10 min sans lire les règles.

## Méthode et limites

- **Simulation** : Outil : src/sim/balance.ts, IA S2 (recherche 2 plis) contre S2, SMART=2 (J1 place en aveugle, J2 place après avoir vu J1), draft aléatoire, règles actuelles du moteur (RULES). 2 200 parties (2 × 1 100), graines 1000-2099 et 3000-4099.

- **Rendement** : « Victoires (sim.) % » = part des parties gagnées par l'équipe qui a cette unité (ou ce Roi). « Compétences/partie » = lancers par partie où l'unité joue. Verdict : ≥ 54 % fort, 48-54 % équilibré, 44-48 % faible, < 44 % très faible. Incertitude ±1,6 pt (unités) / ±3,3 pts (Roi vs Roi, ≈ 170 parties par case).

- **Résultats globaux** : J1 56,1 % / J2 43,8 % / nuls 0,1 % ; 10,2 tours par joueur en moyenne (médiane ≈ 9) ; 106 actions par partie ; 2,45 touches de Roi par partie ; ≈ 7,1 mana perdus par partie ; fin par Roi vaincu 99,4 %.

- **Rois** : Cataclysmique 56,7 % ; Indomptable 56,1 % ; Diplomate 53,6 % ; Économiste 46,3 % ; Stratège 37,5 %.

- **Limites** : IA de profondeur limitée : une unité d'anticipation (Provocateur, Mur de Glace, Aimant) est sous-estimée ; une unité « spam » (Mur de Glace, Anti-CC) peut être sur-utilisée. Le draft est aléatoire (pas de méta). Les variantes (Messager Alerte/Relais, etc.) viennent des essais antérieurs sur d'anciennes règles : à re-simuler.
