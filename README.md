# GSD — Get Shit Done · Cheatsheet

> Système de planification hiérarchique pour le développement agentique solo avec Claude Code.

---

## Démarrage rapide

```
/gsd:new-project       # 1. Initialiser le projet
/gsd:plan-phase 1      # 2. Planifier la première phase
/gsd:execute-phase 1   # 3. Exécuter la phase
```

**Mise à jour :** `npx get-shit-done-cc@latest`

---

## Flux principal

```
/gsd:new-project → /gsd:plan-phase → /gsd:execute-phase → répéter
```

---

## Initialisation du projet

### `/gsd:new-project`
Initialise un nouveau projet via un flux unifié.

- Questions approfondies sur ce que vous construisez
- Recherche domaine optionnelle (4 agents parallèles)
- Définition des exigences v1/v2/hors-scope
- Création de la roadmap avec phases et critères de succès

**Artefacts créés :**
| Fichier | Contenu |
|---|---|
| `PROJECT.md` | Vision et exigences |
| `config.json` | Mode workflow (interactif/yolo) |
| `REQUIREMENTS.md` | Exigences scopées avec REQ-IDs |
| `ROADMAP.md` | Phases mappées aux exigences |
| `STATE.md` | Mémoire projet |
| `research/` | Recherche domaine (si sélectionnée) |

```
/gsd:new-project
```

---

### `/gsd:map-codebase`
Cartographie une codebase existante (projets brownfield).

- Analyse avec des agents Explore en parallèle
- Crée `.planning/codebase/` avec 7 documents (stack, architecture, structure, conventions, testing, integrations, concerns)
- À utiliser **avant** `/gsd:new-project` sur une codebase existante

```
/gsd:map-codebase
```

---

## Planification des phases

### `/gsd:discuss-phase <numéro>`
Articule votre vision d'une phase avant de planifier.

- Capture comment vous imaginez la phase
- Crée `CONTEXT.md` avec votre vision, essentiels et limites
- `--batch` pose 2-5 questions à la fois au lieu d'une par une

```
/gsd:discuss-phase 2
/gsd:discuss-phase 2 --batch
/gsd:discuss-phase 2 --batch=3
```

---

### `/gsd:research-phase <numéro>`
Recherche complète de l'écosystème pour les domaines complexes.

- Découvre le stack standard, les patterns d'architecture, les pièges
- Crée `RESEARCH.md` avec la connaissance "comment les experts construisent ça"
- Utile pour : 3D, jeux, audio, shaders, ML, domaines spécialisés

```
/gsd:research-phase 3
```

---

### `/gsd:list-phase-assumptions <numéro>`
Voir ce que Claude prévoit de faire avant de commencer.

- Montre l'approche envisagée par Claude pour une phase
- Permet de corriger si Claude a mal compris votre vision
- Aucun fichier créé — sortie conversationnelle uniquement

```
/gsd:list-phase-assumptions 3
```

---

### `/gsd:plan-phase <numéro>`
Crée un plan d'exécution détaillé pour une phase spécifique.

- Génère `.planning/phases/XX-phase-name/XX-YY-PLAN.md`
- Décompose la phase en tâches concrètes et actionnables
- Inclut critères de vérification et mesures de succès
- Plusieurs plans par phase supportés (XX-01, XX-02, etc.)

> **PRD Express :** Passez `--prd path/to/requirements.md` pour ignorer discuss-phase. Votre PRD devient des décisions verrouillées dans `CONTEXT.md`.

```
/gsd:plan-phase 1
# → Crée .planning/phases/01-foundation/01-01-PLAN.md
```

---

## Exécution

### `/gsd:execute-phase <numéro>`
Exécute tous les plans d'une phase, ou une vague spécifique.

- Regroupe les plans par vague (depuis le frontmatter), exécute les vagues séquentiellement
- Les plans dans chaque vague s'exécutent en parallèle via l'outil Task
- `--wave N` exécute uniquement la Vague N
- Vérifie l'objectif de la phase après complétion
- Met à jour `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`

```
/gsd:execute-phase 5
/gsd:execute-phase 5 --wave 2
```

---

## Routeur intelligent

### `/gsd:do <description>`
Route du texte libre vers la bonne commande GSD automatiquement.

- Analyse le langage naturel pour trouver la commande GSD correspondante
- Agit comme un dispatcher — ne fait jamais le travail lui-même
- Résout l'ambiguïté en demandant de choisir parmi les meilleures correspondances

```
/gsd:do fix the login button
/gsd:do refactor the auth system
/gsd:do I want to start a new milestone
```

---

## Mode rapide

### `/gsd:quick [--full] [--discuss] [--research]`
Exécute de petites tâches ad-hoc avec les garanties GSD mais sans les agents optionnels.

- Spawne planner + executor (ignore researcher, checker, verifier par défaut)
- Les tâches rapides vivent dans `.planning/quick/` séparément des phases planifiées
- Met à jour `STATE.md` (pas `ROADMAP.md`)

**Flags :**
| Flag | Effet |
|---|---|
| `--discuss` | Discussion légère pour identifier les zones grises avant de planifier |
| `--research` | Agent de recherche ciblée avant planification |
| `--full` | Ajoute la vérification du plan (max 2 itérations) et vérification post-exécution |

> Les flags sont composables : `--discuss --research --full` donne le pipeline qualité complet.

```
/gsd:quick
/gsd:quick --research --full
# → Crée .planning/quick/NNN-slug/PLAN.md et SUMMARY.md
```

---

### `/gsd:fast [description]`
Exécute une tâche triviale inline — sans sous-agents, sans fichiers de planning.

- Pour les tâches trop petites pour justifier une planification (typos, configs, commits oubliés)
- Aucun `PLAN.md` ni `SUMMARY.md` créé
- ≤ 3 fichiers modifiés — redirige vers `/gsd:quick` si la tâche est non-triviale
- Commit atomique avec message conventionnel

```
/gsd:fast "fix the typo in README"
/gsd:fast "add .env to gitignore"
```

---

## Gestion de la roadmap

### `/gsd:add-phase <description>`
Ajoute une nouvelle phase à la fin du milestone actuel.

```
/gsd:add-phase "Add admin dashboard"
```

---

### `/gsd:insert-phase <après> <description>`
Insère un travail urgent comme phase décimale entre des phases existantes.

- Crée une phase intermédiaire (ex: 7.1 entre 7 et 8)
- Maintient l'ordre des phases

```
/gsd:insert-phase 7 "Fix critical auth bug"
# → Crée la Phase 7.1
```

---

### `/gsd:remove-phase <numéro>`
Supprime une phase future et renumérote les phases suivantes.

- Supprime le répertoire de phase et toutes les références
- Fonctionne uniquement sur les phases futures (non démarrées)
- Commit git préserve l'historique

```
/gsd:remove-phase 17
# → Phase 17 supprimée, phases 18-20 deviennent 17-19
```

---

## Gestion des milestones

### `/gsd:new-milestone <nom>`
Démarre un nouveau milestone via un flux unifié.

- Questions approfondies sur ce que vous construisez ensuite
- Recherche domaine optionnelle (4 agents parallèles)
- `--reset-phase-numbers` redémarre la numérotation à la Phase 1

```
/gsd:new-milestone "v2.0 Features"
/gsd:new-milestone --reset-phase-numbers "v2.0 Features"
```

---

### `/gsd:complete-milestone <version>`
Archive le milestone complété et prépare pour la prochaine version.

- Crée une entrée `MILESTONES.md` avec les stats
- Archive les détails complets dans `milestones/`
- Crée un tag git pour la release

```
/gsd:complete-milestone 1.0.0
```

---

## Suivi de progression

### `/gsd:progress`
Vérifie le statut du projet et route intelligemment vers la prochaine action.

- Affiche une barre de progression visuelle et le pourcentage de complétion
- Résume les travaux récents depuis les fichiers SUMMARY
- Affiche la position actuelle et ce qui vient ensuite
- Propose d'exécuter le prochain plan ou de le créer s'il manque

```
/gsd:progress
```

---

## Gestion de session

### `/gsd:resume-work`
Reprend le travail d'une session précédente avec une restauration complète du contexte.

```
/gsd:resume-work
```

---

### `/gsd:pause-work`
Crée un handoff de contexte lors d'une pause en milieu de phase.

- Crée un fichier `.continue-here` avec l'état actuel
- Met à jour la section de continuité de session de `STATE.md`

```
/gsd:pause-work
```

---

## Débogage

### `/gsd:debug [description du problème]`
Débogage systématique avec état persistant entre les resets de contexte.

- Collecte les symptômes via des questions adaptatives
- Crée `.planning/debug/[slug].md` pour suivre l'investigation
- Méthode scientifique : preuves → hypothèse → test
- **Survit au `/clear`** — relancer `/gsd:debug` sans args pour reprendre
- Archive les problèmes résolus dans `.planning/debug/resolved/`

```
/gsd:debug "login button doesn't work"
/gsd:debug   # reprend la session active
```

---

## Notes rapides

### `/gsd:note <texte>`
Capture d'idée zéro-friction — une commande, sauvegarde instantanée.

- Sauvegarde une note horodatée dans `.planning/notes/`
- `--global` sauvegarde dans `$HOME/.claude/notes/` (hors projet)
- Fonctionne sans projet actif

| Sous-commande | Action |
|---|---|
| `append` (défaut) | Ajoute une note |
| `list` | Liste toutes les notes |
| `promote <n>` | Convertit la note n en todo structuré |

```
/gsd:note refactor the hook system
/gsd:note list
/gsd:note promote 3
/gsd:note --global cross-project idea
```

---

## Gestion des todos

### `/gsd:add-todo [description]`
Capture une idée ou tâche comme todo depuis la conversation courante.

- Extrait le contexte de la conversation (ou utilise la description fournie)
- Crée un fichier todo structuré dans `.planning/todos/pending/`
- Vérifie les doublons avant de créer

```
/gsd:add-todo                        # déduit depuis la conversation
/gsd:add-todo Add auth token refresh
```

---

### `/gsd:check-todos [area]`
Liste les todos en attente et en sélectionne un pour travailler.

- Liste tous les todos avec titre, zone, âge
- Filtre optionnel par zone
- Route vers l'action appropriée (travailler maintenant, ajouter à une phase, brainstorm)

```
/gsd:check-todos
/gsd:check-todos api
```

---

## Tests d'acceptation (UAT)

### `/gsd:verify-work [phase]`
Valide les fonctionnalités construites via UAT conversationnel.

- Extrait les livrables testables depuis les fichiers `SUMMARY.md`
- Présente les tests un par un (réponses oui/non)
- Diagnostique automatiquement les échecs et crée des plans de correction

```
/gsd:verify-work 3
```

---

## Livraison

### `/gsd:ship [phase]`
Crée une PR depuis le travail d'une phase complétée.

- Push la branche vers le remote
- Crée une PR avec le résumé depuis `SUMMARY.md`, `VERIFICATION.md`, `REQUIREMENTS.md`
- Demande optionnellement une code review
- Met à jour `STATE.md` avec le statut de livraison

> **Prérequis :** Phase vérifiée, `gh` CLI installé et authentifié.

```
/gsd:ship 4
/gsd:ship 4 --draft
```

---

### `/gsd:review --phase N [--gemini] [--claude] [--codex] [--all]`
Peer review cross-IA — invoque des CLIs externes pour revoir les plans de phase indépendamment.

- Détecte les CLIs disponibles (gemini, claude, codex)
- Chaque CLI revoit les plans indépendamment avec le même prompt structuré
- Produit `REVIEWS.md` avec les retours par reviewer et un résumé consensus

```
/gsd:review --phase 3 --all
# Réinjecter : /gsd:plan-phase N --reviews
```

---

### `/gsd:pr-branch [target]`
Crée une branche propre pour les PRs en filtrant les commits `.planning/`.

- Classifie les commits : code-only (inclus), planning-only (exclu), mixte (inclus sans `.planning/`)
- Les reviewers ne voient que les changements de code

```
/gsd:pr-branch
/gsd:pr-branch main
```

---

### `/gsd:plant-seed [idée]`
Capture une idée prospective avec des conditions de déclenchement.

- Les seeds préservent POURQUOI, QUAND les remonter, et des fils vers le code lié
- Se remonte automatiquement lors de `/gsd:new-milestone` quand les conditions correspondent

```
/gsd:plant-seed "add real-time notifications when we build the events system"
```

---

## Audit

### `/gsd:audit-uat`
Audit cross-phases de tous les items UAT et de vérification en attente.

- Scanne chaque phase pour les items en attente, ignorés, bloqués, et nécessitant une intervention humaine
- Croise avec la codebase pour détecter la documentation périmée
- Produit un plan de test humain priorisé
- À utiliser avant de démarrer un nouveau milestone

```
/gsd:audit-uat
```

---

### `/gsd:audit-milestone [version]`
Audite la complétion du milestone par rapport à l'intention originale.

- Lit tous les fichiers `VERIFICATION.md` des phases
- Vérifie la couverture des exigences
- Spawne un checker d'intégration pour le câblage cross-phases
- Crée `MILESTONE-AUDIT.md` avec les gaps et la dette technique

```
/gsd:audit-milestone
```

---

### `/gsd:plan-milestone-gaps`
Crée des phases pour combler les gaps identifiés par l'audit.

- Lit `MILESTONE-AUDIT.md` et regroupe les gaps en phases
- Priorise par priorité d'exigence (must/should/nice)
- Ajoute les phases de comblement à `ROADMAP.md`

```
/gsd:plan-milestone-gaps
```

---

## Configuration

### `/gsd:settings`
Configure les toggles du workflow et le profil de modèle de manière interactive.

- Toggle les agents researcher, plan checker, verifier
- Sélectionne le profil de modèle
- Met à jour `.planning/config.json`

```
/gsd:settings
```

---

### `/gsd:set-profile <profil>`
Change rapidement le profil de modèle pour les agents GSD.

| Profil | Description |
|---|---|
| `quality` | Opus partout sauf vérification |
| `balanced` | Opus pour planning, Sonnet pour exécution (défaut) |
| `budget` | Sonnet pour l'écriture, Haiku pour recherche/vérification |
| `inherit` | Utilise le modèle de session actuel pour tous les agents |

```
/gsd:set-profile budget
```

---

## Utilitaires

| Commande | Description |
|---|---|
| `/gsd:cleanup` | Archive les répertoires de phases des milestones complétés |
| `/gsd:help` | Affiche cette référence des commandes |
| `/gsd:update` | Met à jour GSD avec preview du changelog |
| `/gsd:join-discord` | Rejoindre la communauté Discord GSD |

---

## Structure des fichiers

```
.planning/
├── PROJECT.md            # Vision du projet
├── ROADMAP.md            # Breakdown des phases actuel
├── STATE.md              # Mémoire et contexte projet
├── RETROSPECTIVE.md      # Rétrospective vivante (mise à jour par milestone)
├── config.json           # Mode workflow & gates
├── todos/
│   ├── pending/          # Todos en attente
│   └── done/             # Todos complétés
├── debug/
│   └── resolved/         # Issues résolues archivées
├── milestones/
│   ├── v1.0-ROADMAP.md
│   ├── v1.0-REQUIREMENTS.md
│   └── v1.0-phases/
│       ├── 01-foundation/
│       └── 02-core-features/
├── codebase/             # Carte codebase (projets brownfield)
│   ├── STACK.md
│   ├── ARCHITECTURE.md
│   ├── STRUCTURE.md
│   ├── CONVENTIONS.md
│   ├── TESTING.md
│   ├── INTEGRATIONS.md
│   └── CONCERNS.md
└── phases/
    ├── 01-foundation/
    │   ├── 01-01-PLAN.md
    │   └── 01-01-SUMMARY.md
    └── 02-core-features/
        ├── 02-01-PLAN.md
        └── 02-01-SUMMARY.md
```

---

## Modes de workflow

### Mode Interactif
- Confirme chaque décision majeure
- Pause aux checkpoints pour approbation
- Plus de guidage tout au long du processus

### Mode YOLO
- Auto-approuve la plupart des décisions
- Exécute les plans sans confirmation
- S'arrête uniquement aux checkpoints critiques

> Configurable à tout moment dans `.planning/config.json`

---

## Configuration avancée (`.planning/config.json`)

| Clé | Défaut | Description |
|---|---|---|
| `planning.commit_docs` | `true` | `false` = artefacts gardés en local, non committés |
| `planning.search_gitignored` | `false` | `true` = inclut `.planning/` dans les recherches ripgrep |

```json
{
  "planning": {
    "commit_docs": false,
    "search_gitignored": true
  }
}
```

> `commit_docs: false` est utile pour les contributions OSS, projets clients, ou garder le planning privé. Pensez à ajouter `.planning/` au `.gitignore`.

---

## Workflows courants

**Démarrer un nouveau projet :**
```
/gsd:new-project      # questioning → research → requirements → roadmap
/clear
/gsd:plan-phase 1
/clear
/gsd:execute-phase 1
```

**Reprendre après une pause :**
```
/gsd:progress         # voir où vous en étiez et continuer
```

**Travail urgent en milieu de milestone :**
```
/gsd:insert-phase 5 "Critical security fix"
/gsd:plan-phase 5.1
/gsd:execute-phase 5.1
```

**Compléter un milestone :**
```
/gsd:complete-milestone 1.0.0
/clear
/gsd:new-milestone    # questioning → research → requirements → roadmap
```

**Capturer des idées pendant le travail :**
```
/gsd:add-todo                    # déduit depuis la conversation
/gsd:add-todo Fix modal z-index  # description explicite
/gsd:check-todos                 # revoir et travailler sur les todos
/gsd:check-todos api             # filtrer par zone
```

**Déboguer un problème :**
```
/gsd:debug "form submission fails silently"
# ... investigation, contexte se remplit ...
/clear
/gsd:debug   # reprendre là où vous vous êtes arrêtés
```

---

## Où trouver de l'aide

- `.planning/PROJECT.md` — vision du projet
- `.planning/STATE.md` — contexte actuel
- `.planning/ROADMAP.md` — statut des phases
- `/gsd:progress` — voir où vous en êtes
