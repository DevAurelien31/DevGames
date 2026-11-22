# Effets et statuts — Duel des Rois

Chaque effet a une icône (carte d'information de l'unité, légende dans Règles) et un champ précis dans le moteur (`src/engine/types.ts`, `src/engine/rules.ts`).

| Icône (id) | Effet | Règle | Modélisation moteur |
|---|---|---|---|
| <img src="icons/immo.svg" width="40" alt="immo"><br>`immo` | **Immobilisé** | Ne peut pas se déplacer. Peut encore attaquer et lancer sa compétence. Infligé par une attaque de base ou certaines compétences. | Piece.immo : 1 = immobilisé à son prochain tour, 2 = immobilisé ce tour. Passe de 1 à 2 au début du tour de son propriétaire, puis à 0 en fin de tour (avec immunité). |
| <img src="icons/antiImmo.svg" width="40" alt="antiImmo"><br>`antiImmo` | **Immunité post-immobilisation** | Quand l’immobilisation se termine (fin de son tour immobilisé), l’unité ne peut pas être immobilisée à nouveau pendant le tour adverse qui suit. Ne protège pas des déplacements forcés. | Piece.antiImmo : posé quand immo 2 expire (ou par le Purificateur selon règle), remis à false au début du tour de son propriétaire. |
| <img src="icons/shield.svg" width="40" alt="shield"><br>`shield` | **Bouclier** | Annule la première immobilisation subie (Roi Indomptable : bouclier de départ de ses unités). Le bouclier disparaît alors. | Piece.indShield : immoOutcome → "absorb", puis indShield = false. |
| <img src="icons/antiCC.svg" width="40" alt="antiCC"><br>`antiCC` | **Anti-contrôle** | Insensible aux immobilisations et déplacements forcés jusqu’au début de son prochain tour (Anti-CC, Messager). | Piece.antiCC, remis à false au début du tour de son propriétaire. |
| <img src="icons/ghost.svg" width="40" alt="ghost"><br>`ghost` | **Insaisissable** | Ne peut pas être ciblé par les actions adverses. Les alliés peuvent toujours le cibler (Messager, Purificateur…). | Piece.untargetable ; testé uniquement quand l’acteur est ennemi (enemy(pc, t)). |
| <img src="icons/taunt.svg" width="40" alt="taunt"><br>`taunt` | **Désarmé** | Désarmé par le Provocateur : ne peut attaquer (attaque de base) que lui ce tour ; ses compétences ciblées doivent aussi le viser si elles le peuvent. | Piece.tauntedBy = id du Provocateur ; filtre les actions legalActions ; remis à -1 en fin de tour. |
| <img src="icons/free.svg" width="40" alt="free"><br>`free` | **Attaque gratuite** | L’attaque de base ne coûte pas de mana ce tour (aura du Messager / Stratège). | Piece.freeAtk ; ignoré par pay(s, p, 1) ; remis à false au début du tour. |
| <img src="icons/cooldown.svg" width="40" alt="cooldown"><br>`cooldown` | **Recharge** | Compétence utilisable 1 tour sur 2 : indisponible le tour suivant son lancer. | Piece.lastSkillTurn ; test turnNo - lastSkillTurn < 2 pour les compétences cooldown. |
| <img src="icons/once.svg" width="40" alt="once"><br>`once` | **Usage unique** | Compétence utilisable une seule fois par partie ; une fois lancée elle est épuisée. | Piece.usedOnce pour les compétences once. |
| <img src="icons/sprint.svg" width="40" alt="sprint"><br>`sprint` | **Sprint en recharge** | Le Sprinteur ne peut refaire un déplacement de 2 cases qu’après la recharge prévue. | Piece.sprintTurn + RULES.sprintCd. |
| <img src="icons/wall.svg" width="40" alt="wall"><br>`wall` | **Mur de glace** | Case bloquée temporairement ; disparaît au début du tour du créateur indiqué. | State.walls[] { idx, owner, expires }. |
| <img src="icons/trap.svg" width="40" alt="trap"><br>`trap` | **Piège** | Case piégée par le Trappeur ; se déclenche quand un ennemi y entre. | State.traps[] { idx, by }. |
| <img src="icons/token.svg" width="40" alt="token"><br>`token` | **Jeton bonus** | Le second joueur dispose d’un jeton : +1 mana, une fois, au moment choisi. | State.token[joueur] (bonus de cristal clignotant bleu sur le plateau). |

## Principes
- Les effets sont soit **sur une pièce** (champs de `Piece`), soit **sur une case** (`State.walls`, `State.traps`), soit **sur un joueur** (`State.token`, mana).
- Un effet « insaisissable » ne bloque que les actions **adverses** ; les compétences alliées peuvent toujours cibler l'unité.
- Une unité immobilisée peut toujours utiliser sa compétence et attaquer ; seul le déplacement est impossible.
- Les remises à zéro ont lieu au début du tour du propriétaire (`startTurn`) ou en fin de tour (`endTurn`).
