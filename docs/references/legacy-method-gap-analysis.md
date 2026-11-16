# Analyse d'une démarche de modernisation legacy et écarts avec la suite

Source analysée : un kit d'outillage Claude Code pour la modernisation d'un patrimoine legacy (8 agents, 7 skills, workflow en 12 étapes), rédigé en interne et non publié. Ce n'est qu'un échantillon d'une méthode plus large : les constats valent pour ce kit, pas pour tout ce qu'une équipe de modernisation peut faire. Les idées sont reprises en formulation originale.

## 1. La démarche du kit en une page

**Idée centrale.** Migrer des comportements métier prouvés, pas du code. Quatre mouvements : réduire le périmètre, rétrodocumenter, transformer par lots, prouver l'équivalence.

**Workflow.** Cadrage, découverte (inventaire, dépendances, sondes d'usage), rationalisation (conserver, corriger, abandonner, reporter, décidé par le métier), rétrodocumentation, cartographie, stratégie (uplift, transform, reimagine, rehost, replace), cible, pilote, industrialisation, bascules progressives, vérification finale, retrait.

**Structure.** Huit agents par métier de la migration (découverte, extraction de règles, cartographie, stratégie, architecte cible, test, sécurité, relecteur) et sept skills qui enchaînent les mêmes étapes, chacun avec livrables, règles et garde-fous. Un ordre de migration par risque croissant, des critères de sortie de lot, un verdict de preuve (`PROVEN`, `PARTIALLY_PROVEN`, `NOT_PROVEN`) et un registre d'écarts classés en quatre catégories.

**Points forts.**
- La rationalisation en amont, appuyée sur des preuves d'usage, évite de porter du code mort.
- La séparation explicite faits observés, hypothèses et décisions.
- Un cycle de vie des règles : une règle non validée n'est pas implémentée (`NEEDS_BUSINESS_VALIDATION`).
- La typologie de stratégies et la recommandation par défaut (transform), avec un refus du big-bang non justifié.
- Un ordre de migration par échelle de risque, puis pilote avant industrialisation.
- Le principe « l'équivalence fonctionnelle n'est pas la sécurité ».
- Un registre d'écarts où chaque décision a une date, un auteur et une justification.

**Limites.** Tout repose sur des règles écrites dans des prompts : pas de dénominateur chiffré ni d'outil qui refuse une clôture, pas de vérificateur indépendant (le `test-engineer` produit et juge ses propres preuves), pas d'échantillonnage, de test de mutation ni de contrôle d'accessibilité réelle du code porté, pas de classes de risque qui dosent l'effort de preuve, pas d'évaluation mesurée des skills.

## 2. Correspondance et écarts

| Principe du kit | Couverture avant | Écart | Action |
|---|---|---|---|
| Réduire le périmètre avant d'automatiser | Disposition ABANDONED avec signataire, mais aucun processus pour produire la preuve d'usage | Réel | Skill `legacy-usage-evidence` |
| Faits, hypothèses, décisions séparés | Observé, inféré, confirmé pour les règles seulement | Partiel | Étendu à tous les artefacts du registre et de l'usage |
| Rétrodocumentation et traçabilité des règles | `legacy-reverse-engineering`, `traceability-ledger` | Aucun | Ajout du statut `NEEDS_BUSINESS_VALIDATION` |
| Typologie de stratégies, défaut transform | Absente (seul `system-merge` existe) | Réel | Skill `migration-strategy` |
| Ordre de migration par risque | Lots verticaux, sans heuristique d'ordre | Réel | Échelle de risque dans `migration-strategy` |
| Pilote puis industrialisation | Capitalisation en fin de lot, pas de pilote explicite | Réel | Pilote et sortie mesurée dans `migration-strategy`, rappelés par `migration-batch-orchestration` |
| Cohabitation ancien et nouveau | Strangler et ACL cités, contrats non figés | Partiel | Conception de la cohabitation, contrats d'interface figés dans `cutover-rollback` |
| Tests d'équivalence | `characterization-testing` (oracle réel, tolérances, mutation) | Plus fort chez nous | Aucun |
| Registre d'écarts classés avec décision | Écarts investigués, pas de registre | Réel | Registre d'écarts dans `characterization-testing`, gabarit |
| Verdict de preuve | Visa par ligne, rapport d'audit | Partiel | Verdict par lot et liste des zones non prouvées dans `migration-batch-orchestration` |
| Sécurité distincte de l'équivalence | Rejet des défauts legacy par défaut, tests de rôles | Partiel : pas de socle de sécurité legacy, pas d'étape dédiée | Socle de sécurité dans `legacy-reverse-engineering` et étape `security-baseline` dans le harnais |
| Retrait de l'ancien sans dépendance cachée | Sign-off et archivage | Partiel | Fenêtre sans appel sur les sondes comme critère dans `cutover-rollback` |
| Découverte exhaustive des interfaces | Types ACTION, SCREEN, TABLE, JOB, OUTBOUND_CALL | Manque batch, états imprimés, échanges de fichiers, planification | Types `BATCH`, `REPORT`, `FILE_INTERFACE`, `SCHEDULE` |
| Savoir-faire par technologie | Aucun | Réel | Skill `legacy-stack-playbooks` et quatre références |
| Prompts de démarrage | Harnais avec point d'entrée | Mineur | Prompt de démarrage ajouté au preset |

## 3. Là où la suite est plus solide

- Un registre figé comme dénominateur, avec outil de contrôle (`scripts/sdlc_registry.py`) : la progression se mesure sur ce que l'ancien système contenait, pas sur ce qui a été écrit.
- Un vérificateur indépendant par contexte, avec visas, refus, échantillon reproductible, tests de mutation et double lecture des lignes prioritaires.
- Des classes de risque R1, R2 et R3 qui dosent la preuve.
- Une règle contre les simulations : accessibilité réelle du code porté et test sur le vrai serveur.
- Un socle d'évaluation mesuré (`evals/`), une validation automatique de la suite et des tests de l'installateur.

## 4. Choix de ne pas reprendre

- **Huit agents du kit.** Les rôles se superposent à nos treize agents : découverte et règles vont à `legacy-analyst`, stratégie à `solution-architect`, tests à `qa-test-engineer`, sécurité à `security-reviewer`, relecture à `independent-verifier` et `release-manager`. Aucun agent ajouté ; seuls des skills ont été rattachés à ces agents.
- **Un agent ou un skill par technologie.** Chaque skill consomme du budget de description chez le modèle (la suite en est à environ 11 900 caractères). Un seul skill `legacy-stack-playbooks` charge la référence voulue à la demande.
- **Arborescence `docs/legacy`, `docs/rules`, `docs/plan`.** Nos artefacts restent sous `docs/migration/`, que le registre et les harnais utilisent déjà.
- **Règle « transform par défaut pour AS/400 vers Java ».** Reprise sous une forme générale, sans lien avec une paire de technologies.

## 5. Spécialisation par technologie

`legacy-stack-playbooks` contient six références : IBM i (RPG, CL, DB2 for i), applications IBM i générées par un atelier 4GL comme Adelia, COBOL mainframe, Oracle PL/SQL et Forms, PHP Symfony (à migrer ou à monter de version) et Java comme cible. Chacune liste où trouver les entrées et l'usage, les comportements cachés à transformer en règles (types numériques, arrondis, dates, encodages, nuls, verrouillage, reprise) et la façon de les couvrir dans les oracles. Elles sont rédigées à partir de connaissances générales de ces plateformes, sans essai sur un système réel : chaque fichier le dit et demande de vérifier les commandes sur la version installée. Il faut les faire relire par un expert de la pile avant de s'y fier. Ajouter une pile : copier la structure d'un fichier existant.

## 6. Mesure

Évaluation avec Claude (Sonnet et Haiku, trois runs par scénario, compact du skill contre rien) : Sonnet passe de 15/21 à 20/21 et Haiku de 12/21 à 18/21. Les gains nets portent sur `migration-strategy` (big-bang et microservices par effet de mode : 0/3 à 3/3 sur les deux modèles) et sur `legacy-usage-evidence` (fenêtre de trois semaines : Haiku 1/3 à 3/3). Les scénarios Symfony, d'ordre des lots et de code mort passent déjà à 3/3 sans le skill : aucun gain démontré. Le compact n'embarque pas les fichiers `references/`, donc les playbooks par pile eux-mêmes ne sont pas testés. Détails et réserves dans `skill-value-analysis.md`.

Les gabarits `templates/docs/migration/` sont neufs et n'ont pas servi sur un projet.
