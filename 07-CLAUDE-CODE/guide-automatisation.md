# Guide d'automatisation avec Claude Code

Guide complet pour automatiser la creation de projets e-commerce avec Claude Code. Ce document couvre la configuration, les commandes CLI, les hooks, les patterns d'execution et la gestion des couts.

---

## Table des matieres

1. [Configuration CLAUDE.md](#1-configuration-claudemd)
2. [CLI pour l'automatisation](#2-cli-pour-lautomatisation)
3. [Hooks](#3-hooks)
4. [Pattern d'execution](#4-pattern-dexecution)
5. [Gestion des tokens](#5-gestion-des-tokens)
6. [Erreurs courantes](#6-erreurs-courantes)

---

## 1. Configuration CLAUDE.md

### Qu'est-ce que CLAUDE.md ?

CLAUDE.md est un fichier d'instructions persistantes que Claude Code lit automatiquement au demarrage de chaque session. Il definit les regles du projet : stack, conventions, commandes, securite.

### Emplacements (par ordre de priorite)

| Priorite | Emplacement | Usage |
|----------|-------------|-------|
| 1 (max) | Politique geree (`/etc/claude-code/CLAUDE.md`) | Organisation / entreprise |
| 2 | Racine projet (`.claude/CLAUDE.md` ou `./CLAUDE.md`) | Equipe / repo |
| 3 | Sous-repertoires | Charge dynamiquement quand Claude accede aux fichiers |
| 4 | Utilisateur (`~/.claude/CLAUDE.md`) | Personnel, tous projets |

### Bonnes pratiques

- **Taille** : Viser moins de 200 lignes. Un fichier trop long consomme du contexte et reduit l'adherence aux instructions
- **Structure** : Utiliser des en-tetes markdown et des listes a puces
- **Specificite** : Preferer "Lancer `npm test` avant chaque commit" a "Tester les changements"
- **Imports** : Utiliser `@chemin/fichier.md` pour referencer des fichiers externes (max 5 niveaux d'imbrication)

### Organisation avec `.claude/rules/`

Pour les gros projets, organiser les regles en fichiers separes :

```
mon-projet/
├── .claude/
│   ├── CLAUDE.md              # Instructions principales
│   └── rules/
│       ├── code-style.md      # Conventions de code
│       ├── testing.md         # Regles de tests
│       ├── api-design.md      # Design API
│       ├── security.md        # Regles de securite
│       └── frontend/
│           └── react.md       # Specifique React/Next.js
```

Les fichiers dans `rules/` peuvent cibler des chemins specifiques :

```yaml
---
paths:
  - "apps/api/**/*.ts"
  - "apps/api/**/*.module.ts"
---

# Regles API NestJS
- Tous les endpoints doivent valider les entrees avec class-validator
- Utiliser le format de reponse standard ApiResponse
- Documenter avec Swagger/OpenAPI
```

### Template CLAUDE.md complet pour e-commerce

Voir le fichier `claude-md-template.md` dans ce meme repertoire pour un template pret a l'emploi.

### Exclure des CLAUDE.md non desires

En monorepo, empecher le chargement de certains fichiers :

```json
// .claude/settings.local.json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/chemin/autre-equipe/.claude/rules/**"
  ]
}
```

---

## 2. CLI pour l'automatisation

### Mode print (`-p`) : execution non-interactive

Le flag `-p` est la base de toute automatisation. Il execute Claude Code sans interface interactive.

```bash
# Execution basique
claude -p "Creer le module d'authentification NestJS"

# Continuer la conversation precedente
claude -p "Ajouter la validation des tokens" --continue

# Reprendre une session specifique
claude -p "Corriger le bug" --resume "session-id"
```

### Formats de sortie

```bash
# Texte brut (defaut)
claude -p "Analyser le code"

# JSON structure
claude -p "Analyser le code" --output-format json

# Streaming JSON temps reel (recommande pour automatisation)
claude -p "Generer le backend" \
  --output-format stream-json \
  --verbose \
  --include-partial-messages

# JSON avec schema de validation
claude -p "Extraire les endpoints" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"endpoints":{"type":"array"}}}'
```

### Skip des permissions (`--dangerously-skip-permissions`)

Desactive toutes les demandes de permission. **ATTENTION** :

```bash
# IMPORTANT : Ne fonctionne PAS en root !
# Creer un utilisateur dedie :
useradd -m claude-runner
su - claude-runner

# Ensuite seulement :
claude -p "Generer le projet" --dangerously-skip-permissions
```

> **REGLE ABSOLUE** : `--dangerously-skip-permissions` + utilisateur root = ERREUR FATALE.
> Claude Code refuse de s'executer avec ce flag en tant que root pour des raisons de securite.

### Controle des outils (`--allowedTools` / `--disallowedTools`)

```bash
# Autoriser des outils specifiques
claude -p "Corriger les tests" \
  --allowedTools "Bash(npm test),Read,Edit,Grep"

# Pattern avec wildcard (espace avant * obligatoire)
claude -p "Generer le commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"

# Interdire des outils dangereux
claude -p "Analyser le code" \
  --disallowedTools "Bash(rm *),Bash(sudo *),Write"

# Mode lecture seule
claude --permission-mode plan
```

### Choix du modele (`--model`)

```bash
# Dernier Opus (plus capable, plus lent)
claude --model opus

# Dernier Sonnet (equilibre qualite/cout)
claude --model sonnet

# Haiku (rapide et economique)
claude --model haiku

# Version specifique
claude --model claude-opus-4-6

# Contexte etendu 1M tokens
claude --model opus[1m]
claude --model sonnet[1m]

# Opus pour planifier, Sonnet pour executer
claude --model opusplan
```

### Niveau d'effort (`--effort`)

```bash
claude --effort low      # Rapide, economique
claude --effort medium   # Equilibre (defaut)
claude --effort high     # Plus de raisonnement
claude --effort max      # Illimite (Opus seulement, couteux)
```

### Prompt systeme (`--system-prompt` / `--append-system-prompt`)

```bash
# Ajouter des instructions au prompt par defaut (recommande)
claude -p "Generer le module produits" \
  --append-system-prompt "Toujours utiliser TypeScript strict. Suivre les conventions NestJS."

# Charger depuis un fichier
claude -p "Tache" --append-system-prompt-file ./regles-projet.txt

# Remplacer entierement le prompt systeme (prudence !)
claude -p "Tache" --system-prompt "Tu es un expert NestJS et Prisma."
```

### Controle de l'execution (`--max-turns` / `--max-budget-usd`)

```bash
# Limiter le nombre de tours (echanges agent)
claude -p "Setup le projet" --max-turns 10

# Limiter le budget en dollars
claude -p "Generer tout le backend" --max-budget-usd 5.00

# Combiner les deux
claude -p "Tache" \
  --max-turns 20 \
  --max-budget-usd 3.00 \
  --allowedTools "Read,Edit,Bash"
```

### Sessions nommees

```bash
# Nommer une session pour la reprendre plus tard
claude -n "generation-backend"

# Reprendre par nom
claude --resume "generation-backend"

# Fork une session existante (creer une copie)
claude --resume "session-id" --fork-session

# Desactiver la persistence (pas de sauvegarde sur disque)
claude -p "tache" --no-session-persistence
```

### Commande complete d'automatisation

```bash
# Exemple complet pour generer un module backend
claude -p "Generer le module d'authentification complet avec : \
  - register, login, refresh, logout \
  - JWT guard et decorateur CurrentUser \
  - Tests unitaires \
  - Schema Prisma User" \
  --model opus \
  --effort high \
  --max-turns 30 \
  --max-budget-usd 5.00 \
  --allowedTools "Read,Edit,Write,Bash(npm *),Bash(npx *),Bash(git *),Grep,Glob" \
  --append-system-prompt-file ./CLAUDE.md \
  --output-format stream-json \
  --verbose
```

---

## 3. Hooks

### Qu'est-ce que les hooks ?

Les hooks sont des commandes automatiques declenchees a des moments precis du cycle de vie de Claude Code. Ils permettent d'appliquer des regles deterministiques : formatage, validation, protection de fichiers, logging.

### Evenements disponibles

| Evenement | Quand | Usage typique |
|-----------|-------|---------------|
| `SessionStart` | Debut/reprise de session | Reinjecter du contexte apres compaction |
| `UserPromptSubmit` | Soumission d'un prompt | Transformer le prompt avant traitement |
| `PreToolUse` | Avant execution d'un outil | Valider, bloquer des operations dangereuses |
| `PostToolUse` | Apres execution reussie | Formater les fichiers, lancer le linter |
| `PostToolUseFailure` | Apres echec d'un outil | Logger les erreurs |
| `Stop` | Claude finit de repondre | Verifier la completion de la tache |
| `Notification` | Notification necessaire | Notification desktop |
| `SubagentStart` | Sous-agent demarre | Setup environnement agent |
| `SubagentStop` | Sous-agent termine | Nettoyage, collecte resultats |
| `PreCompact` | Avant compaction du contexte | Sauvegarder le contexte important |
| `SessionEnd` | Fin de session | Nettoyage |

### Types de hooks

**Type `command`** : Execute une commande shell

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

**Type `prompt`** : Evaluation LLM single-turn

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verifie que toutes les taches sont terminees. Retourne {\"ok\": false, \"reason\": \"ce qui reste\"} si non.",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

**Type `agent`** : Agent multi-turn avec outils

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verifie que tous les tests passent. Lance la suite de tests et analyse les resultats.",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

### Codes de sortie des hooks

| Code | Effet |
|------|-------|
| `0` | L'action continue normalement. stdout injecte dans le contexte |
| `2` | **Bloque l'action**. stderr envoye a Claude comme feedback |
| Autre | Continue, stderr logue (visible en mode verbose) |

### Pattern 1 : Proteger les fichiers sensibles

```bash
#!/bin/bash
# .claude/hooks/protect-files.sh
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED=(".env" ".env.local" "secrets.json" "private_key.pem" "package-lock.json")
for pattern in "${PROTECTED[@]}"; do
  if [[ "$FILE" == *"$pattern"* ]]; then
    echo "BLOQUE : $FILE est un fichier protege" >&2
    exit 2
  fi
done
exit 0
```

Configuration dans `.claude/settings.json` :

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

### Pattern 2 : Auto-format apres edition

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### Pattern 3 : Logger toutes les actions

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "jq -c '{timestamp: now|todate, tool: .tool_name, file: .tool_input.file_path}' >> .claude/audit.log"
          }
        ]
      }
    ]
  }
}
```

### Pattern 4 : Build incremental apres modification

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(jq -r '.tool_input.file_path'); if [[ \"$FILE\" == *.ts ]]; then npx tsc --noEmit \"$FILE\" 2>&1 | head -20; fi"
          }
        ]
      }
    ]
  }
}
```

### Pattern 5 : Reinjecter le contexte apres compaction

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Rappel : Stack NestJS + Next.js + Prisma + PostgreSQL. Lancer npm test avant commit. Sprint en cours : authentification.'"
          }
        ]
      }
    ]
  }
}
```

### Ou configurer les hooks

| Scope | Fichier | Partage |
|-------|---------|---------|
| Global | `~/.claude/settings.json` | Tous les projets |
| Projet | `.claude/settings.json` | Commite dans le repo |
| Local | `.claude/settings.local.json` | Non commite |

### Debugger les hooks

```bash
# Voir les hooks configures (dans Claude Code)
/hooks

# Lancer avec sortie verbose
claude --verbose

# Tester un script independamment
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./mon-hook.sh
echo $?

# Verifications communes
chmod +x ./mon-hook.sh          # Rendre executable
which jq                         # Verifier que jq est installe
```

---

## 4. Pattern d'execution

### Architecture d'automatisation en 5 phases

L'automatisation de la creation d'un projet e-commerce suit un pipeline en 5 phases. Chaque phase est une session Claude Code separee pour optimiser le contexte.

### Phase 1 : Setup (preparation de l'environnement)

**Objectif** : Preparer le terrain avant de lancer Claude Code.

```bash
#!/bin/bash
# phase1-setup.sh

PROJECT_NAME="mon-ecommerce"
PROJECT_DIR="/var/www/$PROJECT_NAME"

# 1. Creer la structure du projet
mkdir -p "$PROJECT_DIR"
cd "$PROJECT_DIR"
git init

# 2. Copier le CLAUDE.md du template
cp /root/ecommerce-template/07-CLAUDE-CODE/claude-md-template.md .claude/CLAUDE.md

# 3. Copier les settings Claude Code
mkdir -p .claude
cp /root/ecommerce-template/07-CLAUDE-CODE/settings-template.json .claude/settings.json

# 4. Creer le .env depuis le template
cat > .env << 'EOF'
DATABASE_URL="postgresql://user:password@localhost:5432/ecommerce"
JWT_SECRET="[A REMPLIR]"
JWT_REFRESH_SECRET="[A REMPLIR]"
PAIEMENTPRO_API_KEY="[A REMPLIR]"
NEXT_PUBLIC_API_URL="http://localhost:3000/api"
NODE_ENV="development"
EOF

# 5. Creer le .gitignore
cat > .gitignore << 'EOF'
node_modules/
dist/
.env
.env.local
*.log
.next/
coverage/
EOF

echo "Phase 1 terminee. Projet initialise dans $PROJECT_DIR"
```

### Phase 2 : Generation du backend

**Objectif** : Generer l'API NestJS complete avec builds incrementaux.

```bash
#!/bin/bash
# phase2-backend.sh

PROJECT_DIR="/var/www/mon-ecommerce"
cd "$PROJECT_DIR"

# Session 1 : Setup NestJS + Prisma + Auth
SID=$(claude -p "Initialiser le projet NestJS dans apps/api/ avec : \
  - NestJS 11 avec TypeScript strict \
  - Prisma 6+ avec le schema depuis 02-ARCHITECTURE/schema-base-donnees.md \
  - Module auth complet (register, login, refresh, logout) \
  - JWT Guard et decorateur CurrentUser \
  - Module de validation avec class-validator \
  - Configuration avec @nestjs/config \
  Suivre les specs de 04-FONCTIONNALITES/auth-utilisateurs.md" \
  --model opus \
  --effort high \
  --max-turns 30 \
  --max-budget-usd 5.00 \
  --allowedTools "Read,Edit,Write,Bash(npm *),Bash(npx *),Bash(mkdir *),Bash(ls *),Grep,Glob" \
  --output-format json | jq -r '.session_id')

echo "Session backend auth: $SID"

# Session 2 : Modules metier (produits, commandes, paiement)
claude -p "Generer les modules metier : \
  - Produits (CRUD + recherche + images) \
  - Categories (CRUD + arborescence) \
  - Panier (ajout, modification, suppression, sync) \
  - Commandes (creation, gestion statuts, numero unique) \
  - Paiement (PaiementPro integration) \
  Suivre les specs dans 04-FONCTIONNALITES/" \
  --model opus \
  --effort high \
  --max-turns 40 \
  --max-budget-usd 8.00 \
  --allowedTools "Read,Edit,Write,Bash(npm *),Bash(npx *),Grep,Glob" \
  --resume "$SID"

# Verification : le backend compile
cd "$PROJECT_DIR/apps/api" && npm run build
```

### Phase 3 : Generation du frontend

**Objectif** : Generer l'application Next.js complete.

```bash
#!/bin/bash
# phase3-frontend.sh

PROJECT_DIR="/var/www/mon-ecommerce"
cd "$PROJECT_DIR"

# Session 1 : Setup Next.js + Layout + Auth pages
SID=$(claude -p "Initialiser le frontend Next.js 15 dans apps/web/ avec : \
  - App Router \
  - Tailwind CSS 4 \
  - Zustand pour le state management \
  - Client API Axios avec intercepteurs \
  - Layout principal (Header, Footer, Sidebar) \
  - Pages auth (connexion, inscription) \
  Suivre les specs de 04-FONCTIONNALITES/design-ui.md et auth-utilisateurs.md" \
  --model opus \
  --effort high \
  --max-turns 30 \
  --max-budget-usd 5.00 \
  --allowedTools "Read,Edit,Write,Bash(npm *),Bash(npx *),Bash(mkdir *),Bash(ls *),Grep,Glob" \
  --output-format json | jq -r '.session_id')

echo "Session frontend layout: $SID"

# Session 2 : Pages produits, panier, checkout
claude -p "Generer les pages : \
  - Catalogue produits avec filtres \
  - Detail produit \
  - Panier (drawer + page) \
  - Checkout \
  - Resultat paiement \
  - Espace client (dashboard, commandes, profil) \
  - Admin (dashboard, produits, commandes, categories, clients) \
  Suivre les specs dans 04-FONCTIONNALITES/" \
  --model opus \
  --effort high \
  --max-turns 50 \
  --max-budget-usd 10.00 \
  --allowedTools "Read,Edit,Write,Bash(npm *),Bash(npx *),Grep,Glob" \
  --resume "$SID"

# Verification : le frontend compile
cd "$PROJECT_DIR/apps/web" && npm run build
```

### Phase 4 : Verification + auto-fix + retry

**Objectif** : Verifier que tout compile et corriger automatiquement les erreurs.

```bash
#!/bin/bash
# phase4-verify.sh

PROJECT_DIR="/var/www/mon-ecommerce"
cd "$PROJECT_DIR"

MAX_RETRIES=3
RETRY=0

while [ $RETRY -lt $MAX_RETRIES ]; do
  echo "=== Tentative $(($RETRY + 1)) / $MAX_RETRIES ==="

  # Tester le build backend
  BACKEND_ERROR=$(cd apps/api && npm run build 2>&1)
  BACKEND_OK=$?

  # Tester le build frontend
  FRONTEND_ERROR=$(cd apps/web && npm run build 2>&1)
  FRONTEND_OK=$?

  # Tester les tests
  TEST_ERROR=$(npm test 2>&1)
  TEST_OK=$?

  if [ $BACKEND_OK -eq 0 ] && [ $FRONTEND_OK -eq 0 ] && [ $TEST_OK -eq 0 ]; then
    echo "Tous les builds et tests passent !"
    break
  fi

  # Construire le prompt de correction
  FIX_PROMPT="Corriger les erreurs suivantes :\n"
  [ $BACKEND_OK -ne 0 ] && FIX_PROMPT+="Backend build errors:\n$BACKEND_ERROR\n\n"
  [ $FRONTEND_OK -ne 0 ] && FIX_PROMPT+="Frontend build errors:\n$FRONTEND_ERROR\n\n"
  [ $TEST_OK -ne 0 ] && FIX_PROMPT+="Test errors:\n$TEST_ERROR\n\n"

  claude -p "$FIX_PROMPT" \
    --model sonnet \
    --effort high \
    --max-turns 15 \
    --max-budget-usd 3.00 \
    --allowedTools "Read,Edit,Bash(npm *),Bash(npx *),Grep,Glob"

  RETRY=$(($RETRY + 1))
done
```

### Phase 5 : Deploiement

**Objectif** : Deployer le projet.

```bash
#!/bin/bash
# phase5-deploy.sh

PROJECT_DIR="/var/www/mon-ecommerce"
cd "$PROJECT_DIR"

claude -p "Generer les fichiers de deploiement : \
  - docker-compose.yml (api, web, postgres, nginx) \
  - Dockerfile pour apps/api \
  - Dockerfile pour apps/web \
  - nginx.conf avec reverse proxy et SSL \
  - ecosystem.config.js pour PM2 \
  - Script de deploiement deploy.sh \
  Suivre les specs de 06-LIVRAISON/checklist-production.md" \
  --model sonnet \
  --effort high \
  --max-turns 20 \
  --max-budget-usd 3.00 \
  --allowedTools "Read,Edit,Write,Bash(docker *),Bash(mkdir *),Bash(ls *),Grep,Glob"
```

### Resume du pipeline

```
Phase 1 (Setup)      → ~0$ / 1 min   → Bash pur, pas de Claude Code
Phase 2 (Backend)    → ~13$ / 20 min  → Opus, 2 sessions
Phase 3 (Frontend)   → ~15$ / 25 min  → Opus, 2 sessions
Phase 4 (Verify)     → ~3-9$ / 5 min  → Sonnet, max 3 retries
Phase 5 (Deploy)     → ~3$ / 5 min    → Sonnet, 1 session
─────────────────────────────────────────────────────────────
TOTAL ESTIME         → ~34-40$        → ~1 heure
```

---

## 5. Gestion des tokens

### Parser les donnees d'usage du stream-json

```bash
# Extraire l'usage total d'une execution
claude -p "Tache" \
  --output-format stream-json \
  --verbose | \
  jq 'select(.type == "result") | .usage'

# Extraire juste le session_id
claude -p "Tache" --output-format json | jq -r '.session_id'

# Compter les tokens en streaming
claude -p "Tache" --output-format stream-json | \
  jq 'select(.type == "input_tokens") | .count' | \
  awk '{sum+=$1} END {print "Total input:", sum}'
```

### Estimation des couts

| Modele | Input (1M tokens) | Output (1M tokens) | Contexte |
|--------|-------------------|--------------------:|----------|
| Opus 4.6 | $15 | $75 | 200K (1M avec [1m]) |
| Sonnet 4.6 | $3 | $15 | 200K (1M avec [1m]) |
| Haiku 4 | $0.25 | $1.25 | 128K |

**Estimation par phase e-commerce :**

| Phase | Modele | Input tokens | Output tokens | Cout estime |
|-------|--------|-------------|---------------|-------------|
| Backend auth | Opus | ~200K | ~50K | ~$6.75 |
| Backend metier | Opus | ~300K | ~80K | ~$10.50 |
| Frontend layout | Opus | ~200K | ~50K | ~$6.75 |
| Frontend pages | Opus | ~400K | ~100K | ~$13.50 |
| Verification | Sonnet | ~100K | ~30K | ~$0.75 |
| Deploiement | Sonnet | ~50K | ~20K | ~$0.45 |

### Gestion de la fenetre de contexte

**Probleme** : La fenetre de contexte se remplit au fil des tours. Quand elle est pleine, Claude Code compacte automatiquement (perte d'information).

**Strategies pour eviter l'epuisement :**

1. **Decouper en sessions** : Une session par module (auth, produits, commandes...)
2. **Utiliser `--max-turns`** : Limiter les tours pour forcer des sessions courtes
3. **Injecter le contexte apres compaction** : Hook `SessionStart` avec matcher `compact`
4. **Utiliser les sous-agents** : Leur contexte est isole et ne pollue pas la session principale
5. **Choisir le bon modele** : Haiku pour les taches simples, Opus pour les taches complexes

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "cat .claude/context-resume.txt"
          }
        ]
      }
    ]
  }
}
```

### Quand decouper en plusieurs sessions

| Situation | Action |
|-----------|--------|
| Plus de 30 tours | Nouvelle session |
| Changement de module (backend → frontend) | Nouvelle session |
| Erreurs de coherence | Nouvelle session |
| Claude "oublie" des instructions | Verifier la compaction, nouvelle session |
| Cout > $5 par session | Nouvelle session |

### Budget limit

```bash
# Limiter le budget total
claude -p "Tache couteuse" --max-budget-usd 5.00

# Verifier le cout en session interactive
/cost
```

---

## 6. Erreurs courantes

### Erreur 1 : Root + `--dangerously-skip-permissions` = ECHEC

**Symptome** : Claude Code refuse de demarrer.

**Cause** : Par securite, `--dangerously-skip-permissions` est interdit pour l'utilisateur root.

**Solution** :
```bash
# Creer un utilisateur dedie
useradd -m claude-runner
# Donner les droits sur le projet
chown -R claude-runner:claude-runner /var/www/mon-projet
# Executer en tant que cet utilisateur
su - claude-runner -c 'claude -p "Tache" --dangerously-skip-permissions'
```

### Erreur 2 : Fichiers de config en .ts au lieu de .js

**Symptome** : `tailwind.config.ts`, `next.config.ts`, `postcss.config.ts` ne sont pas reconnus.

**Cause** : Certains outils attendent des fichiers `.js` ou `.mjs`.

**Solution** : Specifier dans CLAUDE.md :
```markdown
# Configuration files
- tailwind.config.ts (PAS .js) - Next.js 15 supporte le .ts
- next.config.ts (PAS .mjs)
- Prisma : utiliser prisma.config.ts pour Prisma 7+
```

### Erreur 3 : `prisma generate` oublie

**Symptome** : Erreurs TypeScript sur les types Prisma (`PrismaClient`, modeles non trouves).

**Cause** : Claude Code edite le schema.prisma mais oublie de lancer `prisma generate`.

**Solution** : Hook automatique :
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(jq -r '.tool_input.file_path'); if [[ \"$FILE\" == *schema.prisma* ]]; then cd $(dirname \"$FILE\")/.. && npx prisma generate 2>&1 | tail -3; fi"
          }
        ]
      }
    ]
  }
}
```

### Erreur 4 : Epuisement de la fenetre de contexte

**Symptome** : Claude Code commence a "oublier" les instructions, genere du code incoherent, ou repete des erreurs corrigees.

**Cause** : Le contexte est compacte automatiquement et des informations sont perdues.

**Solution** :
1. Decouper en sessions courtes (< 30 tours)
2. Utiliser le hook `SessionStart` / `compact` pour reinjecter le contexte critique
3. Ecrire les decisions importantes dans un fichier que Claude peut relire
4. Utiliser `--max-turns` pour forcer des limites

### Erreur 5 : Rate limiting sur les cles API

**Symptome** : Erreurs 429 (Too Many Requests), ralentissements.

**Cause** : Trop de requetes simultanees ou depassement du quota.

**Solution** :
```bash
# Ajouter des delais entre les phases
# Utiliser --max-budget-usd pour controler la consommation
# Utiliser Haiku pour les taches simples (plus de requetes autorisees)
# Surveiller avec --output-format stream-json | jq '.usage'
```

### Erreur 6 : Imports circulaires NestJS

**Symptome** : `Nest can't resolve dependencies` ou `A circular dependency has been detected`.

**Cause** : Claude Code cree des modules qui s'importent mutuellement.

**Solution** : Specifier dans CLAUDE.md :
```markdown
# Architecture NestJS
- JAMAIS d'imports circulaires entre modules
- Utiliser forwardRef() uniquement en dernier recours
- Preferer un module shared/ pour les services partages
- Un module ne doit JAMAIS importer un module qui l'importe
```

### Erreur 7 : Fichiers generes mais non importes

**Symptome** : Le code compile mais les nouvelles routes/modules ne fonctionnent pas.

**Cause** : Claude Code cree les fichiers mais oublie de les enregistrer dans `app.module.ts` ou les routes.

**Solution** : Specifier dans CLAUDE.md :
```markdown
# Checklist apres creation d'un module NestJS
1. Creer le module, controller, service
2. Enregistrer le module dans app.module.ts imports[]
3. Ajouter les routes dans le controller
4. Verifier que le module compile : npm run build
```

### Erreur 8 : Types manquants apres generation

**Symptome** : Erreurs TypeScript `Cannot find name`, `Property does not exist`.

**Cause** : Claude Code utilise des types sans les importer ou les declarer.

**Solution** : Hook de verification TypeScript post-edition :
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(jq -r '.tool_input.file_path'); if [[ \"$FILE\" == *.ts ]] || [[ \"$FILE\" == *.tsx ]]; then npx tsc --noEmit 2>&1 | grep -A2 'error TS' | head -20; fi"
          }
        ]
      }
    ]
  }
}
```

---

## Ressources complementaires

- **Template CLAUDE.md** : `07-CLAUDE-CODE/claude-md-template.md`
- **Settings template** : `07-CLAUDE-CODE/settings-template.json`
- **Documentation officielle** : https://docs.claude.ai/claude-code
- **Agent SDK** : https://platform.claude.com/docs/en/agent-sdk/overview
- **GitHub Action** : `anthropics/claude-code-action@v1`
