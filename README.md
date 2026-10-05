# Resilience

> Projet final de formation — plateforme full-stack de résilience climatique, conçue comme un cran au-dessus de Cellar et TechnoCovoit.

Resilience est un projet de fin de formation destiné à transformer des données climatiques et environnementales réelles en informations compréhensibles, comparables et exploitables.

Le projet est volontairement construit par étapes : d'abord une application full-stack robuste et déterministe, puis des capacités géospatiales et d'analyse, puis une intégration RAG / MCP / IA qui reste subordonnée au cœur métier.

## Vision

Resilience doit permettre d'explorer la résilience climatique d'un territoire ou d'un logement à partir de données sourcées : chaleur, sécheresse, précipitations, inondation, incendie, qualité de l'air, stress hydrique et autres indicateurs pertinents.

L'objectif n'est pas de produire un score opaque ni de demander à un LLM d'inventer une conclusion. L'application doit conserver une source de vérité explicite, traçable et testable.

## Principe fondamental

**L'application doit rester utile sans IA.**

Le moteur métier reste déterministe. Les données externes sont normalisées, validées et historisées avec leur provenance. Le RAG, MCP et le LLM n'interviennent qu'au-dessus de ce socle pour rechercher, orchestrer, contextualiser et expliquer.

## Architecture cible

Le backend sera construit comme un **monolithe modulaire avec Spring Boot et Spring Modulith**, en appliquant des principes **Clean / hexagonaux à l'intérieur de chaque module**.

```text
Angular
   │
   │ HTTPS / REST
   ▼
┌─────────────────────────────────────────────────────────────┐
│                 Resilience · Spring Modulith                │
│                                                             │
│  location   climate   risk   comparison   knowledge   ...  │
│     │          │       │         │            │             │
│     └──────────┴───────┴─────────┴────────────┘             │
│                                                             │
│  Chaque module :                                            │
│  domain ← application / ports ← adapters                    │
└───────────────┬───────────────────────┬─────────────────────┘
                │                       │
                ▼                       ▼
      PostgreSQL / PostGIS          External APIs
             / PGVector         (weather, climate, etc.)
                ▲
                │
         RAG / knowledge
                ▲
                │
          MCP server / tools
                ▲
                │
        Grounded AI assistant
```

### Découpage envisagé

```text
resilience
├── location
│   ├── api
│   └── internal
│       ├── domain
│       ├── application
│       └── adapter
├── climate
│   ├── api
│   └── internal
│       ├── domain
│       ├── application
│       └── adapter
├── risk
│   ├── api
│   └── internal
│       ├── domain
│       ├── application
│       └── adapter
├── comparison
├── knowledge
├── recommendation
└── identity
```

Le découpage exact pourra évoluer avec le domaine, mais les dépendances métier devront rester explicites et testables.

## Moteur de risque

Le **Risk Engine** restera indépendant du LLM et des fournisseurs de données externes.

```text
External data
     │
     ▼
climate adapters
     │
     ▼
normalized ClimateData
     │
     ▼
deterministic Risk Engine
     │
     ├── Heat risk
     ├── Flood risk
     ├── Fire risk
     ├── Drought risk
     └── other derived indicators
```

Le LLM pourra expliquer un résultat, mais il ne sera jamais autorisé à inventer ou modifier le résultat calculé par le domaine.

## Données et persistence

La cible actuelle est :

- **PostgreSQL** pour les données applicatives et historiques ;
- **PostGIS** pour les besoins géographiques ;
- **PGVector** pour la recherche vectorielle et le RAG ;
- migrations versionnées ;
- provenance, période et date de collecte visibles pour les données externes.

Les sources externes seront ajoutées progressivement. Une API météo fera partie du MVP ; d'autres jeux de données climatiques et environnementaux pourront ensuite enrichir l'analyse.

## RAG et MCP

Le RAG doit apporter de la **connaissance documentaire sourcée**, pas remplacer le moteur métier.

Le serveur MCP sera une frontière d'intégration séparée qui exposera des outils explicites réutilisant les mêmes services applicatifs que l'API REST.

Exemples d'outils envisagés :

- `get_location_profile`
- `get_temperature_history`
- `get_precipitation_history`
- `compare_locations`
- `get_heatwave_statistics`
- `get_indicator_metadata`
- `search_climate_documents`
- `assess_risk`
- `explain_risk`

L'API REST et MCP ne doivent pas dupliquer la logique métier :

```text
Angular ──REST──► Application services
MCP     ────────► Application services
```

## Parcours utilisateur envisagé

- rechercher et sélectionner un lieu ;
- afficher des indicateurs climatiques et environnementaux ;
- observer des historiques et tendances ;
- comparer plusieurs localisations ;
- distinguer données observées, indicateurs dérivés et projections ;
- visualiser la provenance des données ;
- enregistrer des lieux ou logements ;
- obtenir progressivement des recommandations d'adaptation ;
- interroger ensuite l'application en langage naturel à partir des mêmes données et documents sourcés.

## Stack cible

| Domaine | Technologies |
|---|---|
| Backend | Java, Spring Boot, Spring Modulith |
| Architecture | Clean / hexagonale par module |
| API | REST / JSON |
| Frontend | Angular, TypeScript |
| Persistence | PostgreSQL, PostGIS |
| RAG | PGVector |
| Testing | JUnit, Spring Boot Test, tests d'intégration |
| Runtime | Docker / Docker Compose |
| MCP | Serveur MCP dédié + intégration côté application |
| IA | LLM + tool use + RAG après stabilisation du cœur métier |
| CI/CD | GitHub Actions |

Les versions exactes seront choisies au moment de l'implémentation plutôt que figées trop tôt.

## Progression prévue

### Phase 1 — Full-stack déterministe

1. Spring Boot / Spring Modulith ;
2. PostgreSQL et migrations ;
3. domaine location / climate / risk ;
4. intégration d'une première API météo ;
5. endpoints REST, DTOs, validation et gestion d'erreurs ;
6. Angular ;
7. tests ;
8. Docker Compose ;
9. provenance des données.

### Phase 2 — Géospatial et analyse

1. PostGIS ;
2. comparaison de lieux ;
3. historique et tendances ;
4. Risk Engine déterministe ;
5. indicateurs complémentaires et projections selon les sources disponibles.

### Phase 3 — Knowledge / RAG / MCP

1. corpus documentaire sourcé ;
2. embeddings et PGVector ;
3. recherche RAG ;
4. serveur MCP ;
5. outils climatiques explicites et réutilisables.

### Phase 4 — Assistant IA ancré dans les données

Le modèle sera autorisé à répondre uniquement à partir :

- des données du projet ;
- des résultats des outils MCP ;
- des documents récupérés via le RAG.

Les réponses devront exposer les éléments de preuve utiles : période, indicateurs et provenance.

## Objectifs qualité

Le projet doit démontrer plus que la présence de fonctionnalités :

- frontières métier explicites ;
- règles et invariants testables ;
- évolution reproductible de la base ;
- tests unitaires et d'intégration significatifs ;
- gestion contrôlée des pannes des services externes ;
- distinction entre observations, projections et indicateurs dérivés ;
- traçabilité de la provenance ;
- absence de dépendance silencieuse à un LLM pour la logique déterministe ;
- documentation des décisions architecturales.

## Structure du repository

```text
Resilience/
├── backend/                  # Spring Boot / Spring Modulith
├── frontend/                 # Angular
├── mcp/                      # MCP server / protocol-specific integration
├── docs/
│   ├── ARCHITECTURE.md
│   ├── PRODUCT_SCOPE.md
│   ├── ROADMAP.md
│   └── DECISIONS.md
├── .env.example
├── .gitignore
├── docker-compose.yml
└── README.md
```

## Statut

**Conception / initialisation.** Le périmètre produit et l'architecture sont définis avant l'introduction progressive du code de l'application.

L'objectif est de pouvoir expliquer et défendre chaque choix technique, et de faire de Resilience la continuité logique de la progression :

```text
Cellar
  → architecture backend et métier

TechnoCovoit
  → backend + frontend Angular

Resilience
  → full-stack + données externes + géospatial + RAG + MCP + IA
```
