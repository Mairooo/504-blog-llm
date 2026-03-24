# 504 Blog LLM — Blog automatisé par IA 

Plateforme de blog automatisée qui agrège des articles d'actualité depuis des flux RSS francophones, les réécrit à l'aide de l'IA (Google Gemini), génère du contenu de fact-checking (debunk), et publie le tout sur un blog stylisé.

## Fonctionnalités

- **Agrégation RSS** — Collecte d'articles depuis FranceInfo, Le Monde, BFMTV, Le Parisien et CNEWS
- **Sélection intelligente** — L'IA analyse les articles et choisit les plus pertinents
- **Réécriture automatique** — Génération de titres accrocheurs et réécriture du contenu en Markdown
- **Évaluation qualité** — Scoring automatique (ragebait, clarté, cohérence, impact) avec seuil minimum de 0.6
- **Génération d'images** — Création d'images via Gemini Imagen 4.0
- **Fact-checking (Debunk)** — Génération automatique de contenu de vérification des faits
- **Publication Twitter** — Partage automatique des articles via l'API Twitter v2
- **Interface blog** — Affichage des articles avec rendu Markdown et typographie soignée

## Stack technique

| Composant       | Technologie                                  |
|-----------------|----------------------------------------------|
| Backend         | Laravel 12, PHP                              |
| IA              | Google Gemini 2.5 Flash, Mistral (fallback)  |
| Frontend        | Blade, Tailwind CSS 4, Vite 7               |
| Base de données | MySQL                                        |
| Tests           | Pest 4                                       |
| Markdown        | League CommonMark                            |

## Prérequis

- PHP 8.2+
- Composer
- Node.js & npm
- MySQL
- Clé API Google Gemini
- (Optionnel) Clés API Twitter

## Installation

```bash
# Cloner le dépôt
git clone <url-du-repo>
cd site

# Installation automatique (dépendances, clé app, migrations, build frontend)
composer setup
```

Ou manuellement :

```bash
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate
npm run build
```

## Configuration

Copier `.env.example` en `.env` et renseigner les variables suivantes :

```env
# Base de données
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nom_de_la_base
DB_USERNAME=utilisateur
DB_PASSWORD=mot_de_passe

# IA (obligatoire)
GEMINI_API_KEY=votre_clé_gemini

# API interne
API_KEY=votre_clé_api

# Twitter (optionnel)
TWITTER_API_KEY=...
TWITTER_API_SECRET=...
TWITTER_ACCESS_TOKEN=...
TWITTER_ACCESS_TOKEN_SECRET=...
TWITTER_BEARER_TOKEN=...
```

## Utilisation

### Lancer le serveur de développement

```bash
composer dev
```

Lance simultanément le serveur Laravel, le worker de queue, le monitoring des logs (Pail) et Vite en mode dev.

### Traitement automatique des articles

```bash
php artisan articles:auto
```

Cette commande exécute le workflow complet :

1. **Sélection** — L'IA choisit le meilleur article parmi les flux RSS
2. **Réécriture & évaluation** — Réécriture avec titre accrocheur, évaluation qualité (jusqu'à 3 tentatives)
3. **Debunk** — Génération du contenu de fact-checking

Les logs sont écrits dans `storage/logs/log.txt`.

## Routes

| Méthode | Route                        | Description                              |
|---------|------------------------------|------------------------------------------|
| GET     | `/`                          | Page d'accueil (liste des articles)      |
| GET     | `/article/{id}`              | Page détail d'un article                 |
| GET     | `/articles/{id}/debunk`      | Page de fact-checking d'un article       |
| GET     | `/articles/latest`           | Liste des articles (AJAX)                |
| GET     | `/api/articles/json`         | Tous les articles en JSON                |
| GET     | `/api/articles/one`          | Sélection IA du prochain article         |
| GET     | `/api/articles/rewrite`      | Réécriture et évaluation d'un article    |
| GET     | `/api/articles/debunk`       | Génération du debunk                     |

## Architecture

```
Flux RSS → RssFeedService → Sélection IA (Gemini)
                                  ↓
                          ChosenArticles (BDD)
                                  ↓
                     Réécriture IA + Évaluation
                                  ↓
                     Score ≥ 0.6 → Articles (BDD)
                                  ↓
                     Génération Debunk → Debunks (BDD)
                                  ↓
                     Affichage Blog (Blade + Markdown)
                                  ↓
                     Publication Twitter (API v2)
```

## Structure du projet

```
app/
├── Console/Commands/       # Commande artisan (AutoProcessArticles)
├── Http/
│   ├── Controllers/        # AIController, ArticleController, DebunkController, NewsController
│   └── Middleware/          # ApiKeyMiddleware (authentification X-API-KEY)
├── Models/                 # Article, ChosenArticle
└── Services/               # GeminiService, MistralService, RssFeedService, TwitterService
database/migrations/        # Tables : articles, chosen_articles, debunks
resources/views/            # Blade : home, articles, debunks, layouts
routes/web.php              # Définition des routes
```

## Tests

```bash
composer test
```

Les tests utilisent Pest et couvrent les contrôleurs AI et News ainsi que les endpoints API.

## Scripts Composer

| Commande         | Description                                            |
|------------------|--------------------------------------------------------|
| `composer setup` | Installation complète (dépendances, migrations, build) |
| `composer dev`   | Serveur de dev + queue + logs + Vite                   |
| `composer test`  | Exécution des tests Pest                               |
