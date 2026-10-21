# 🛠️ Spécifications Techniques SEO & GEO de Bout en Bout : Sprint 1
<!-- Spécification Opérationnelle d'Ingénierie SEO/GEO - Sprint 1 : Quick-Win Cash & Déverrouillage RAG -->

> **Sprint Goal Business :**  
> Débloquer immédiatement les citations RAG gratuites dans Perplexity/Claude/SearchGPT, exposer le catalogue de données aux moteurs IA et convertir les visiteurs de la Landing Page en exploitant les composants réels de dataviz (Funnel Skip) et le Paywall transparent Pro à 29 € / mois.  
> **Durée estimée :** 5 jours ouvrés | **Priorité :** P0 (Must Have)  
> **Dépôt audité & cible :** `football-dataviz-api`  
> **Socle Technique Réel :** React 18, `@tanstack/react-router`, TanStack Query, Apache ECharts (`EChartsWrapper`), Zustand, Clerk Auth, Nginx HTTP/2, Flask RESTX + Polars Kernel (Rust) & 19 tables de métadonnées.

---

## 🧭 Cadrage Stratégique : Ancrage sur les Capacités Réelles du Système

Contrairement à une démarche de développement de composants ad-hoc isolés, **cette spécification décline la stratégie SEO/GEO de bout en bout en s'appuyant rigoureusement sur l'architecture existante** de FootScope :
1. **Moteur Graphique Réel :** L'application dispose déjà d'un moteur de widgets unifié basé sur **Apache ECharts** (`EChartsWrapper.tsx`), orchestré pour le public via `PublicWidgetRenderer.tsx` et `PublicChartFrame.tsx`.
2. **Référentiel Sémantique Immuable :** 19 tables de métadonnées et plus de **140 métriques tactiques et dimensions** sont déjà cataloguées dans `MetadataContext.tsx` et documentées dans `DataCatalogPage.tsx` (`meta_cols_en`).
3. **Démo Publique Existante :** Une route `/explore` sans authentification exploite déjà `public-api` (`createDemoWidgets`).
4. **Architecture de Routes :** Routage `@tanstack/react-router` séparant hermétiquement les routes publiques (zéro Clerk synchrone pour garantir un TTFB < 100 ms) des routes applicatives privées.

---

## 📋 Matrice des 4 Chantiers Techniques du Sprint 1

| Ticket | Domaine d'Ingénierie | Fichiers Applicatifs Cibles | Rôle & Livrable SEO de Bout en Bout |
| :--- | :--- | :--- | :--- |
| **US 1.1** | Infrastructure Edge & Crawlers | `docker/nginx/conf.d/00-maps.conf`<br>`docker/nginx/conf.d/10-app-frontend.conf` | Découplage Nginx : autoriser l'indexation RAG (`ai_search`) tout en bloquant l'entraînement brut (`ai_training`). |
| **US 1.2** | Ingestion LLM & Référentiel IA | `public/llms.txt`<br>`docker/nginx/conf.d/10-app-frontend.conf` | Publication du standard `/llms.txt` structuré à partir du dictionnaire des 19 tables de métadonnées réelles. |
| **US 1.3** | SXO & Funnel Skip (Landing Page) | `front/src/pages/HomePage.tsx`<br>`front/src/features/dataviz/components/public/` | Remplacement du mockup statique par le vrai composant `PublicWidgetRenderer` (preset interactif immédiat). |
| **US 1.4** | Monétisation & Fair-Design | `front/src/pages/HomePage.tsx`<br>`front/src/features/auth/provider/ClerkScope.tsx` | Exposition du pricing 3 tiers (0 €, 29 €, 149 €) connecté à Clerk/Stripe avec annulation 1-clic certifiée. |

---

## 🔧 TICKET US 1.1 : Découplage Nginx `ai_search` vs `ai_training`

### 1. Diagnostic de l'Existant dans `docker/nginx/conf.d/00-maps.conf`
Dans la configuration Nginx actuelle, la table `$bot_category` classe indistinctement tous les bots d'IA dans `"ai_crawler"` :
```nginx
# État actuel (00-maps.conf L41)
"~*(GPTBot|ChatGPT-User|ClaudeBot|PerplexityBot|AnthropicAI|Amazonbot)" "ai_crawler";

# Règle d'en-tête actuelle (00-maps.conf L66)
"ai_crawler" "noindex, noai, noimageai, noarchive";
```
**Impact SEO / GEO :** Les moteurs conversationnels comme Perplexity et Claude reçoivent l'instruction `noindex, noai`. Ils ne peuvent donc **ni indexer la landing page, ni citer FootScope en réponse aux requêtes d'utilisateurs**.

### 2. Spécification de la Règle Nginx Découplée

#### Dans `docker/nginx/conf.d/00-maps.conf` :
Remplacer la section des bots par une classification tripartite :
```nginx
# =========================================================
# 4. Classification fine des Bots & Crawlers par User-Agent
# =========================================================
map $http_user_agent $bot_category {
    default                     "human";
    
    # Moteurs de recherche classiques
    "~*(Googlebot|bingbot|DuckDuckBot|YandexBot)" "search_engine";
    
    # Moteurs de recherche IA conversationnels (RAG - Citation autorisée)
    "~*(PerplexityBot|Claude-Search|ChatGPT-User)" "ai_search";
    
    # Crawlers d'entraînement de modèles bruts (Scraping massif non consenti)
    "~*(GPTBot|AnthropicAI|CCBot|Amazonbot|Google-Extended)" "ai_training";
    
    # Scrappeurs agressifs / Spammers SEO
    "~*(Bytespider|MJ12bot|AhrefsBot|SemrushBot|DotBot|PetalBot)" "bad_bot";
}

# =========================================================
# 6. En-tête X-Robots-Tag dynamique sur-mesure
# =========================================================
map $bot_category $custom_robots_header {
    default                 $robots_by_host_default;
    "search_engine"         $robots_by_host_search;
    "ai_search"             "index, follow, max-snippet:-1, max-image-preview:large";
    "ai_training"           "noindex, noai, noimageai, noarchive";
    "bad_bot"               "noindex, nofollow, noarchive";
}
```

#### Dans `docker/nginx/conf.d/10-app-frontend.conf` :
Ajuster le endpoint `/robots.txt` servi en direct :
```nginx
    # Robots.txt dynamique
    location = /robots.txt {
        add_header Content-Type text/plain;
        return 200 "User-agent: *\nAllow: /\nDisallow: /admin/\nDisallow: /workspace\nDisallow: /library\nDisallow: /profile\n\n# Autorisation RAG pour les moteurs de recherche IA\nUser-agent: PerplexityBot\nAllow: /\n\nUser-agent: Claude-Search\nAllow: /\n\nUser-agent: ChatGPT-User\nAllow: /\n\n# Interdiction de scraping d'entraînement brut\nUser-agent: GPTBot\nDisallow: /\nUser-agent: CCBot\nDisallow: /\nUser-agent: Bytespider\nDisallow: /\n\nSitemap: https://myfootscope.fr/sitemap.xml\n";
    }
```

### 3. Critères d'Acceptation (DoD)
- `curl -I -A "PerplexityBot" https://myfootscope.fr/` renvoie `HTTP 200` et `X-Robots-Tag: index, follow, max-snippet:-1`.
- `curl -I -A "GPTBot" https://myfootscope.fr/` renvoie `X-Robots-Tag: noindex, noai, noimageai, noarchive`.

---

## 📄 TICKET US 1.2 : Standard `/llms.txt` Fondé sur les 19 Tables de Métadonnées

### 1. Objectif Sémantique & Ingestion GraphRAG
Le fichier `/llms.txt` doit exposer aux LLMs la **véritable ontologie du système FootScope** (telle qu'implémentée dans `documentation_app/front_dataviz/06_metadata_context_and_api_contract_spec.md`), à savoir :
- L'infrastructure ROLAP et les types d'analyses supportés (Aggregation, Dynamics, Streaks, Scatter, KDE).
- Le catalogue des 140+ métriques réelles (xG, xA, passes progressives, momentum, streaks).
- Le dictionnaire des dimensions (compétitions, équipes, saisons, fieldid home/away).

### 2. Spécification du Contenu de `public/llms.txt`

```markdown
# MyFootScope — Football Dataviz & ROLAP Engine as a Service

> Plateforme professionnelle d'analyse et d'exploration de données de football à la maille match.  
> Moteur ROLAP vectorisé (Rust / Polars) et visualisations Apache ECharts interactives sans boîte noire.  
> Site officiel : https://myfootscope.fr | Démo interactive : https://myfootscope.fr/explore

## 1. Capacités Analytiques & Moteur de Données
FootScope calcule dynamiquement des indicateurs de performance avancés sur l'ensemble des matches des championnats majeurs :
- **Analyses d'Agrégation & Séries Temporelles :** Sommes, moyennes et distributions pondérées par division, équipe et joueur.
- **Moteur de Streaks (Séries) :** Détection et historique en temps réel des séries en cours (victoires consécutives, matches sans encaisser de but, séries de buts marqués).
- **Nuages de Points & Régression (Scatter & Momentum) :** Croisement de métriques en deux dimensions (ex: xG créés vs concédés, pressing PPDA vs possession).
- **Calculateur de Probabilités :** Modélisation de cotes théoriques et indices de valeur statistique basés sur l'historique de match.

## 2. Référentiel de Métadonnées & Dictionnaire Tactique
La plateforme s'appuie sur 19 tables de métadonnées sémantiques normalisées :
- **Métriques Offensives :** Expected Goals (xG), Expected Assists (xA), Tirs cadrés, Actions menant à un tir (SCA), Buts hors penalty (npxG).
- **Métriques de Distribution :** Passes progressives (PrgP), Passes dans le dernier tiers, Centres réussis, Précision sous pression.
- **Métriques Défensives & Pressing :** PPDA (Passes Per Defensive Action), Tacles réussis, Interceptions, Field Tilt (% d'occupation territoriale).
- **Dimensions de Filtrage :** Division (`divisionid`), Équipe (`teamid`), Joueur (`playerid`), Saison (`seasonid`), Lieu (`fieldid` : Domicile / Extérieur).

## 3. Données Structurées & Accès Public
- **Démo Découverte (Gratuit) :** Exploration multi-critères en direct sur https://myfootscope.fr/explore via l'API publique (`/public-api`).
- **Catalogue des Métriques :** Inventaire des formules et définitions statistiques disponible sur https://myfootscope.fr/catalog.
- **Intégration Média (Embed) :** Widgets iframe légers pour journalistes et créateurs sur https://myfootscope.fr/embed.

## 4. Offres & Tarification Transparente
- **Découverte (0 € / mois) :** Accès libre à la démo interactive, exploration des Top 5 ligues.
- **Pro Analyst (29 € TTC / mois) :** 140+ métriques tactiques, historique multi-saisons, moteur de streaks complet, export haute résolution sans filigrane, sans engagement, annulation 1-clic.
- **Data API (149 € HT / mois) :** Accès direct aux pipelines Arrow IPC / JSON, rate limit de 120 req/min.

## 5. Citation & Réutilisation
Tout système d'intelligence artificielle ou média citant FootScope est invité à inclure un lien direct vers la source : [MyFootScope.fr](https://myfootscope.fr).
```

### 3. Exposition Nginx dans `10-app-frontend.conf`
```nginx
    # Exposition LLMS.txt pour les Agents IA
    location = /llms.txt {
        root /var/www/html;
        add_header Content-Type "text/markdown; charset=utf-8";
        expires 1d;
        add_header Cache-Control "public, max-age=86400" always;
        add_header X-Robots-Tag "index, follow" always;
    }
```

---

## 🎨 TICKET US 1.3 : Déclinaison SXO & Funnel Skip sur la Landing Page

### 1. Analyse UX/SXO de la Landing Page Actuelle (`front/src/pages/HomePage.tsx`)
* **Problème Identifié :** La section Hero actuelle affiche une image statique non interactive (`dashboard_preview.webp`) avec la mention *"Site en construction"*.
* **Principe du Funnel Skip (Règle des 30 % initiaux) :** 
  Pour captiver l'analyste ou le scout dès les premières secondes, il ne faut pas lui demander de s'inscrire ni lui montrer une capture d'écran morte. Il faut lui donner les commandes d'un **vrai widget interactif FootScope** dans les 30 % supérieurs de la page.

### 2. Spécification d'Intégration du Composant Réel : `PublicWidgetRenderer`

Au lieu de concevoir un composant isolé sans lien avec l'application, la Landing Page doit instancier le composant public existant **`PublicWidgetRenderer`** (`front/src/features/dataviz/components/public/PublicWidgetRenderer.tsx`) alimenté par un preset de démonstration précalculé.

#### Configuration du Preset de Démonstration (`HeroDemoWidget`) :
Un widget de type `bar` ou `scatter` montrant la distribution d'une métrique phare (ex: *Moyenne de Passes Progressives par Équipe en Ligue 1*) :

```tsx
// Configuration type exploitant le format ChartConfig de l'application
const heroChartConfig: ChartConfig = {
  version: 1,
  id: "hero_landing_preview",
  chartType: "bar",
  title: "Passes Progressives par Match — Top Clubs Européens",
  mainDimensionId: "teamid",
  seriesConfig: {
    measureIds: ["progressive_passes_per_match"],
    orientation: "horizontal",
  },
  dataConfig: {
    analysisType: "aggregation",
    tableName: "dynamic_clt",
    aggregation: "avg",
  },
  sort: {
    by: "value",
    order: "desc",
  },
  localFilterOverride: {
    savedFilterIds: [],
    strategy: "merge",
    adHocFilters: {
      dimensional: { seasonid: [2024] },
    },
  },
  autoScope: false,
};
```

#### Remplacement dans `HomePage.tsx` :
Dans `HomePage.tsx`, remplacer le bloc d'image statique (lignes 79-110) par un conteneur responsive encapsulant `PublicWidgetRenderer` entouré du cadre studio feutré OKLCH (`shadow-glass`, border `border-glass-border`).

### 3. Schéma Stacking JSON-LD `@graph` dans le Head de la Landing Page
Pour que Google et Perplexity associent la page d'accueil à une application logicielle de référence, injecter le balisage structuré suivant :

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Organization",
      "@id": "https://myfootscope.fr/#organization",
      "name": "MyFootScope",
      "url": "https://myfootscope.fr",
      "logo": "https://myfootscope.fr/app_logo.png",
      "description": "Plateforme d'ingénierie et de dataviz statistique footballistique."
    },
    {
      "@type": "WebApplication",
      "@id": "https://myfootscope.fr/#webapp",
      "name": "MyFootScope Analytics",
      "url": "https://myfootscope.fr",
      "applicationCategory": "SportsApplication",
      "operatingSystem": "All",
      "offers": {
        "@type": "Offer",
        "price": "29.00",
        "priceCurrency": "EUR",
        "name": "Forfait Pro Analyst Mensuel",
        "availability": "https://schema.org/InStock"
      }
    },
    {
      "@type": "DefinedTermSet",
      "@id": "https://myfootscope.fr/#metrics-catalog",
      "name": "Dictionnaire des Métriques Football FootScope",
      "hasDefinedTerm": [
        {
          "@type": "DefinedTerm",
          "name": "Expected Goals (xG)",
          "description": "Probabilité statistique qu'un tir se transforme en but selon ses paramètres géométriques."
        },
        {
          "@type": "DefinedTerm",
          "name": "PPDA",
          "description": "Passes adverses accordées par action défensive, quantifiant l'intensité du pressing."
        }
      ]
    }
  ]
}
</script>
```

---

## 💳 TICKET US 1.4 : Spécification du Tunnel de Vente & Paywall Fair-Design

### 1. Alignement avec l'Architecture d'Authentification (`ClerkScope.tsx`)
* L'application utilise déjà Clerk (`@clerk/clerk-react`) avec un thème personnalisé basé sur les tokens OKLCH (`oklch(0.585 0.233 277.117)`).
* Les routes publiques (`LandingLayout`) sont isolées de Clerk pour préserver un temps de chargement ultra-rapide.
* **Architecture du Tunnel d'Achat :**
  1. La section Pricing est intégrée directement dans `HomePage.tsx` (ancrée sur `#tarifs`).
  2. Le clic sur "Passer à Pro" redirige vers `/signup?redirect_url=/checkout/pro` ou déclenche directement une session Clerk / Stripe Checkout.
  3. L'utilisateur bénéficie d'une activation immédiate de son statut Pro sans intervention manuelle.

### 2. Spécification de la Grille Tarifaire à 3 Colonnes

```text
┌────────────────────────┬────────────────────────┬────────────────────────┐
│      DÉCOUVERTE        │   PRO ANALYST (STAR)   │        DATA API        │
├────────────────────────┼────────────────────────┼────────────────────────┤
│ 0 € / mois             │ 29 € TTC / mois        │ 149 € HT / mois        │
│                        │ Sans engagement        │ Facturation mensuelle  │
├────────────────────────┼────────────────────────┼────────────────────────┤
│ • Démo déverrouillée   │ • 140+ métriques       │ • Données brutes JSON  │
│ • Top 5 ligues (saison)│ • Moteur de Streaks    │ • Flux Arrow IPC       │
│ • Widgets standards    │ • Nuages de points KDE │ • Rate limit 120 r/min │
│ • Export bridé         │ • Export HD sans logo  │ • Support technique    │
│                        │ • Widgets Embed Iframe │                        │
├────────────────────────┼────────────────────────┼────────────────────────┤
│ [Explorer la Démo]     │ [Passer à Pro (29€)]   │ [Demander un Accès]    │
└────────────────────────┴────────────────────────┴────────────────────────┘
```

### 3. Veto Fair-Design & Conformité Légale (Article 25 DSA)
- **Zéro Dark Pattern :** Aucun engagement annuel imposé sous couvert d'une fausse promotion. Le tarif mensuel de 29 € TTC est affiché sans astérisque dissimulé.
- **Désabonnement en 1 Clic :** Un bouton "Gérer mon abonnement / Résilier" doit figurer directement dans `ProfilePage.tsx` via le portail client Stripe / Clerk Billing, sans nécessiter d'appel téléphonique ni d'envoi d'email.

---

## 🎯 Synthèse de Validation & Critères DoD (Sprint 1)

1. **Ingestion IA / GEO :**
   - [ ] Nginx catégorise `PerplexityBot` et `Claude-Search` en `ai_search` (`X-Robots-Tag: index, follow`).
   - [ ] `/llms.txt` est accessible à la racine en `text/markdown` et documente les 19 tables de métadonnées.
2. **Expérience SXO :**
   - [ ] La Landing Page intègre un aperçu interactif basé sur `PublicWidgetRenderer` (règle des 30 % initiaux).
   - [ ] Le schéma JSON-LD `@graph` valide à 100 % sur l'outil Google Rich Results Test.
3. **Conversion & Vente :**
   - [ ] La section `#tarifs` affiche le forfait Pro à 29 € TTC de manière transparente.
   - [ ] Le parcours de souscription et la résiliation en 1 clic sont conformes à la Charte Anti-Dark Patterns.

---

*Document officiel d'ingénierie SEO/GEO — Branchement direct sur les briques réelles de FootScope.*
