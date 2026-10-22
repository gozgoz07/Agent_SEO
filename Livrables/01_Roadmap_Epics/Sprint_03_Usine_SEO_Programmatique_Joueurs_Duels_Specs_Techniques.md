# 🛠️ Spécifications Techniques SEO & GEO de Bout en Bout : Sprint 3
<!-- Spécification Opérationnelle d'Ingénierie SEO/GEO - Sprint 3 : L'Usine SEO Programmatique (Joueurs & Duels 1v1 SSR) -->

> **Sprint Goal Business :**  
> Déployer une usine SEO programmatique de **50 000 points d'entrée organiques** indexables sans JavaScript (SSR / Prerendering) sur Google et les moteurs IA, et alimenter les débats viraux sur les réseaux sociaux (Twitter/X, Reddit) via un comparateur 1v1 dédupliqué et des cartes OpenGraph 1200x630 générées à la volée.  
> **Durée estimée :** 8 jours ouvrés | **Priorité :** P1 (Should Have)  
> **Dépôt audité & cible :** `football-dataviz-api`  
> **Socle Technique Réel :** Tables `ref_players`, `ref_teams`, `ref_leagues`, `@tanstack/react-router`, `PublicWidgetRenderer` (Apache ECharts), API Flask RESTX, Nginx Reverse Proxy.

---

## 🧭 Cadrage Stratégique : L'Usine Programmatique Fondée sur `ref_players`

Le back-end FootScope héberge en mémoire vive (RAM Tmpfs Arrow) la totalité des métadonnées des joueurs (`ref_players`), équipes (`ref_teams`) et compétitions (`ref_leagues`), synchronisées via `MetadataContext` et servies par les endpoints `/meta/players` et `/meta/teams`.

Le **Sprint 3** industrialise l'exposition de ces données sous la forme d'un **générateur programmatique à très grande échelle** :
1. **Zéro boîte noire :** Chaque page joueur expose les métriques objectives issues du moteur Polars/Arrow.
2. **Indexabilité sans JavaScript :** Génération de coquilles HTML pré-rendues (SSR / Dynamic Prerendering Nginx) pour que les crawlers IA (PerplexityBot, Claude-Search) et Googlebot indexent 100 % du texte et des statistiques au premier passage.
3. **Élimination mathématique du contenu dupliqué :** Normalisation stricte de l'ordre alphabétique des duels 1v1 pour diviser par deux le nombre d'URLs de comparaison et concentrer le PageRank.

---

## 📋 Matrice des 3 Chantiers Techniques du Sprint 3

| Ticket | Domaine d'Ingénierie | Fichiers Cibles (`football-dataviz-api`) | Rôle & Livrable SEO/GEO |
| :--- | :--- | :--- | :--- |
| **US 3.1** | SEO Programmatique & SSR | `front/src/pages/players/PlayerDetailPage.tsx`<br>`front/src/routes.tsx`<br>`docker/nginx/conf.d/10-app-frontend.conf` | Générateur dynamique de 50 000 fiches joueurs SSR (`/joueurs/{slug}`), paragraphe sémantique de percentiles (Gain d'Information) et Schema.org `Person`/`Athlete`. |
| **US 3.2** | Comparateur 1v1 & Canoniques | `front/src/pages/compare/CompareDuelPage.tsx`<br>`front/src/routes.tsx` | Moteur de duels `/comparateur/{a}-vs-{b}` avec normalisation alphabétique stricte (Redirection 301 automatique sur l'ordre inverse) et vue ECharts superposée. |
| **US 3.3** | Viralité Visuelle & OpenGraph | `app/api_app/api/resources/og/`<br>`docker/nginx/conf.d/20-app-api.conf` | Endpoint de génération d'images OG 1200x630 à la volée, balises `twitter:card` et cache Nginx 7 jours pour maximiser le taux de clic sur X et Reddit. |

---

## 🏭 TICKET US 3.1 : Générateur Programmatique de Fiches Joueurs (`/joueurs/{slug}`)

### 1. Structure d'URL Canonique & Normalisation des Slugs
Pour éviter tout risque de collision d'homonymes (ex: *Gabriel Silva*, *Luis Diaz*) et garantir la pérennité des URLs :
* **Format Canonique :** `/joueurs/{nom-normalise}-{p_player_id}`  
  *(Exemple : `/joueurs/lamine-yamal-84920`)*
* **Algorithme de Slugging (RFC 3986) :**
  1. Conversion en minuscules.
  2. Remplacement des caractères diacritiques (é -> e, ã -> a, etc.).
  3. Suppression des caractères spéciaux non alphanumériques (ponctuation, apostrophes).
  4. Remplacement des espaces par des tirets `-`.
  5. Suffixe obligatoire de l'identifiant immuable `p_player_id` issu de `ref_players`.

---

### 2. Stratégie de Rendu Côté Serveur (SSR / Dynamic Prerendering Nginx)
Pour que Googlebot et PerplexityBot consomment les données sans exécuter de JavaScript :

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              FLUX D'INGESTION PRERENDERING / SSR NGINX                      │
├─────────────────────────────────────────────────────────────────────────────┤
│ Requête HTTP GET /joueurs/lamine-yamal-84920                                │
│                                                                             │
│ [Nginx 10-app-frontend.conf]                                                │
│   ├── Si User-Agent = "human"                                               │
│   │     └── Serve SPA React (/index.html) -> Hydratation TanStack Router    │
│   │                                                                         │
│   └── Si User-Agent = "search_engine" OU "ai_search"                        │
│         └── Proxy_pass vers service Prerender interne (port 3000 / SSR)     │
│               └── Renvoie HTML statique hydraté (< 150 ms) avec :           │
│                     • Paragraphe de percentiles en clair                    │
│                     • Balisage JSON-LD emboîté complet                      │
│                     • Balises Meta OpenGraph dynamiques                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Configuration Nginx (`docker/nginx/conf.d/10-app-frontend.conf`) :
```nginx
    # Routage dynamique des fiches joueurs avec Prerendering pour les robots
    location ~* ^/joueurs/([a-z0-9-]+)$ {
        set $is_crawler 0;
        if ($bot_category ~* "^(search_engine|ai_search)$") {
            set $is_crawler 1;
        }

        if ($is_crawler = 1) {
            # Redirection transparente vers le moteur de rendu SSR ou cache statique
            proxy_pass http://frontend_ssr:3000;
            break;
        }

        # Comportement standard SPA pour les navigateurs humains
        include /etc/nginx/conf.d/common/security-headers.conf;
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }
```

---

### 3. Ingénierie Textuelle Automatisée (Gain d'Information — Brevet US11354342B2)
Pour que la page joueur ne soit pas considérée par Google comme une page blanche (SPA vide) ou du *Thin Content*, un module algorithmique génère côté serveur un texte de **120 à 150 mots à très haute densité factuelle** basé sur les 3 percentiles majeurs du joueur calculés par le moteur Polars :

```markdown
<!-- Exemple de bloc textuel généré côté serveur pour Lamine Yamal -->
### Profil Tactique & Analyse des Percentiles 2025/2026 : Lamine Yamal
Évoluant au poste d'Ailier Droit au FC Barcelone en Liga (saison 2025/2026), Lamine Yamal (18 ans) 
se positionne dans le **96e percentile** des joueurs de sa catégorie en création d'occasions, 
avec une moyenne de **0.42 Expected Assists (xA) par 90 minutes**. 
Sur le plan de la percussion offensive, il se classe dans le **98e percentile** en dribbles réussis 
(4.1 par match) et le **95e percentile** en courses progressives vers le dernier tiers adverse. 
En finition, sa production atteint **0.31 npxG (buts hors penalty attendus)** par tir, démontrant une 
sélection d'angle de frappe supérieure à la moyenne européenne.
```

---

### 4. Balisage Schema Stacking JSON-LD (`Person` + `Athlete` + `SportsTeam`)

```tsx
// Modèle de composant d'injection Schema.org pour la fiche joueur
export function JsonLdPlayer({ player, team, league, metrics }: PlayerJsonLdProps) {
  const playerUrl = `https://myfootscope.fr/joueurs/${player.slug}-${player.p_player_id}`;

  const schemaGraph = {
    "@context": "https://schema.org",
    "@graph": [
      {
        "@type": "Person",
        "@id": `${playerUrl}#athlete`,
        "name": player.name,
        "url": playerUrl,
        "image": player.hasimg ? `https://cdn.myfootscope.fr/players/${player.p_player_id}.png` : undefined,
        "jobTitle": "Footballeur Professionnel",
        "memberOf": {
          "@type": "SportsTeam",
          "name": team.name,
          "sport": "Football",
          "memberOf": {
            "@type": "SportsOrganization",
            "name": league.name
          }
        }
      },
      {
        "@type": "WebPage",
        "@id": `${playerUrl}#webpage`,
        "url": playerUrl,
        "name": `${player.name} : Statistiques, Percentiles & Radar Tactique 2026`,
        "description": `Analyse statistique détaillée de ${player.name} (${team.name}) : xG, xA, passes progressives et percentiles de performance.`,
        "mainEntity": {
          "@id": `${playerUrl}#athlete`
        },
        "breadcrumb": {
          "@type": "BreadcrumbList",
          "itemListElement": [
            { "@type": "ListItem", "position": 1, "name": "Accueil", "item": "https://myfootscope.fr" },
            { "@type": "ListItem", "position": 2, "name": "Joueurs", "item": "https://myfootscope.fr/joueurs" },
            { "@type": "ListItem", "position": 3, "name": player.name, "item": playerUrl }
          ]
        }
      }
    ]
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schemaGraph) }}
    />
  );
}
```

---

## ⚔️ TICKET US 3.2 : Comparateur de Duels 1v1 & Règle Canonique Absolue

### 1. Le Piège Algorithmique du Contenu Dupliqué sur les Duels
Si un utilisateur accède à `/comparateur/dembele-vs-yamal` et qu'un autre accède à `/comparateur/yamal-vs-dembele`, Google identifie deux pages au contenu identique avec des URLs distinctes. Cela dilue le PageRank par 2 et déclenche une pénalité pour contenu dupliqué.

### 2. Spécification de la Règle Canonique Alphabétique
* **Règle d'Or :** Les deux slugs de joueurs sont **toujours triés par ordre alphabétique strict** ($Slug_A < Slug_B$).
* **Comportement de Routage :**
  - Accès à `/comparateur/dembele-vs-yamal` (Correct car 'd' < 'y') -> Réponse HTTP 200 avec `<link rel="canonical" href="https://myfootscope.fr/comparateur/dembele-vs-yamal" />`.
  - Accès à `/comparateur/yamal-vs-dembele` (Ordre inversé) -> **Redirection HTTP 301 immédiate (Permanente)** vers `/comparateur/dembele-vs-yamal`.

#### Implémentation du Middleware de Résolution Canonique (TanStack Router) :
```tsx
// Route TanStack Router pour le comparateur
export const compareDuelRoute = createRoute({
  getParentRoute: () => publicLayout,
  path: "/comparateur/$duel",
  beforeLoad: ({ params }) => {
    const parts = params.duel.split("-vs-");
    if (parts.length === 2) {
      const [playerA, playerB] = parts;
      if (playerA > playerB) {
        // Redirection 301 automatique vers l'ordre alphabétique canonique
        throw redirect({
          to: `/comparateur/${playerB}-vs-${playerA}`,
          statusCode: 301,
        });
      }
    }
  },
  component: CompareDuelPage,
});
```

---

### 3. Visualisation Polygone Radar Superposé (ECharts Réel)
La page duel instancie `PublicWidgetRenderer` en mode comparaison :
- **Polygone 1 :** Joueur A représenté par la couleur primaire FootScope (`oklch(0.585 0.233 277)`).
- **Polygone 2 :** Joueur B représenté par la teinte complémentaire studio (`oklch(0.768 0.165 70)`).
- **Barre de différentiels :** Tableau comparatif instantané affichant les écarts de percentiles sur les 6 métriques phares.

---

## 🖼️ TICKET US 3.3 : Générateur Dynamique de Cartes OpenGraph (1200x630)

### 1. Objectif de Viralité & Croissance ($K$-Factor)
Pour dominer les débats sur Twitter/X et Reddit, chaque lien partagé d'un joueur ou d'un duel doit automatiquement afficher une **carte visuelle ultra-premium de 1200x630 pixels** intégrant le logo FootScope, les visages des joueurs et le polygone radar comparatif.

---

### 2. Architecture de l'Endpoint Fastify / Flask : `/api/v1/og/`

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 PIPELINE DE GÉNÉRATION D'IMAGES OPENGRAPH                   │
├─────────────────────────────────────────────────────────────────────────────┤
│ Requête HTTP GET /api/v1/og/compare/dembele-841/yamal-849.png              │
│                                                                             │
│ [Nginx Cache L1 (7 jours)]                                                  │
│   ├── Hit  ──> Réponse binaire PNG directe (< 15 ms)                       │
│   └── Miss ──> Appel Flask / Fastify (/api/v1/og/)                          │
│                  ├── Extraction des métadonnées Arrow en RAM (Tmpfs)        │
│                  ├── Rendu SVG vectoriel 1200x630 (Design System OKLCH)     │
│                  ├── Rendu Raster PNG via Resvg / Cairo (< 45 ms)           │
│                  └── Stockage cache Nginx & Réponse avec headers :          │
│                        Cache-Control: public, max-age=604800, immutable     │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### En-têtes HTML injectés dans le `<head>` de la page duel :
```html
<!-- OpenGraph Standard -->
<meta property="og:type" content="article" />
<meta property="og:title" content="Duel Tactique : Ousmane Dembélé vs Lamine Yamal (2025/2026)" />
<meta property="og:description" content="Comparaison radar des percentiles : qui domine en création d'occasions et dribbles ?" />
<meta property="og:image" content="https://api.myfootscope.fr/api/v1/og/compare/dembele-841/yamal-849.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />

<!-- Twitter / X Cards -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:site" content="@MyFootScope" />
<meta name="twitter:title" content="Ousmane Dembélé vs Lamine Yamal : Qui est le roi de la création ?" />
<meta name="twitter:image" content="https://api.myfootscope.fr/api/v1/og/compare/dembele-841/yamal-849.png" />
```

---

## 🎯 Critères d'Acceptation & Validation DoD (Sprint 3)

- [ ] **Capacité d'Indexation Programmatique :** Les URLs `/joueurs/{slug}` sont générées dynamiquement à partir de `ref_players` et servent un rendu HTML complet lisible avec JavaScript désactivé.
- [ ] **Gain d'Information Validé :** Chaque fiche joueur comporte un paragraphe textuel unique de $\ge 120$ mots intégrant au minimum 3 percentiles calculés en temps réel.
- [ ] **Déduplication Stricte du Comparateur :** Tout accès à `/comparateur/b-vs-a` où $b > a$ déclenche une redirection HTTP 301 vers `/comparateur/a-vs-b`.
- [ ] **Cartes OpenGraph Opérationnelles :** L'URL de partage d'un duel génère une image PNG 1200x630 conforme, validée sur le *Twitter Card Validator* et *Facebook Sharing Debugger*.
- [ ] **Temps de Réponse & Mise en Cache :** Les requêtes d'images OG en cache répondent en $< 20$ ms grâce à la configuration Nginx.

---

*Dossier de réalisation officiel Sprint 3 — Usine SEO Programmatique MyFootScope.*
