# 🛠️ Spécifications Techniques SEO & GEO de Bout en Bout : Sprint 2
<!-- Spécification Opérationnelle d'Ingénierie SEO/GEO - Sprint 2 : Autorité GEO & Cocon Sémantique Éducatif -->

> **Sprint Goal Business :**  
> Établir FootScope comme la référence absolue citée par Google et les moteurs génératifs (ChatGPT Search, Perplexity, Claude) sur le lexique tactique du football moderne.  
> Valoriser les 140+ métriques du `DataCatalog` en un cocon sémantique éducatif public, optimisé pour l'ingestion GraphRAG, la pyramide inversée et le maillage contextuel *Reasonable Surfer*.  
> **Durée estimée :** 7 jours ouvrés | **Priorité :** P1 (Should Have)  
> **Dépôt cible :** `football-dataviz-api`  
> **Socle Technique Réel :** `@tanstack/react-router`, `PublicWidgetRenderer` (Apache ECharts), `MetadataContext` (`meta_cols_en`), Schema Stacking JSON-LD (`@graph`).

---

## 🧭 Cadrage Stratégique : Du Data Catalog au Cocon Sémantique Public

Le système FootScope dispose d'un capital statistique exceptionnel consigné dans son dictionnaire de métadonnées (`meta_cols_en`, `ref_leagues`, `ref_teams`) et exposé en interne dans `DataCatalogPage.tsx`.

Cependant, ces données étaient jusqu'alors confinées derrière une page privée.  
Le **Sprint 2** transforme ce référentiel propriétaire en **6 pages piliers d'autorité thématique (*Topical Authority*)**, publiquement indexables, rédigées selon les standards GEO 2026 (Princeton SIGKDD) et maillées vers l'environnement interactif de la démo `/explore`.

---

## 📋 Matrice des Chantiers Techniques du Sprint 2

| Ticket | Domaine d'Ingénierie | Fichiers Cibles (`football-dataviz-api`) | Rôle & Livrable SEO/GEO |
| :--- | :--- | :--- | :--- |
| **US 2.1** | Contenu GEO & Pyramide Inversée | `front/src/pages/metrics/MetricDetailPage.tsx`<br>`front/src/content/metrics/*.md` | Rédaction des 6 fiches tactiques piliers (définition mathématique < 50 mots, densité $\ge 12$ stats/100 mots). |
| **US 2.2** | GraphRAG & Schema Stacking | `front/src/components/seo/JsonLdMetric.tsx`<br>`front/src/routes.tsx` | Balisage JSON-LD `@graph` interconnectant `DefinedTerm`, `TechArticle`, `DefinedTermSet`, `WebApplication` et `FAQPage`. |
| **US 2.3** | Maillage & Intégration ECharts | `front/src/routes.tsx`<br>`front/src/features/dataviz/components/public/` | Déclaration des routes publiques TanStack Router, maillage *Reasonable Surfer* et preset ECharts interactif par fiche. |

---

## 📚 TICKET US 2.1 : Les 6 Fiches Piliers Tactiques (Rédaction Haute Densité Factuelle)

Chaque fiche répond strictement à la **règle de la pyramide inversée** : la réponse directe à l'intention de recherche est résolue dès les 30 % initiaux du document afin d'alimenter directement les snippets Google AI Overviews et la fenêtre de contexte des agents RAG (Gain d'Information - brevet US11354342B2).

---

### Fiche 1 : `/metriques/expected-goals-xg`

* **URL Canonique :** `https://myfootscope.fr/metriques/expected-goals-xg`
* **Title SEO :** `Expected Goals (xG) : Définition, Calcul Mathématique & Modèle Statistique | FootScope`
* **Meta Description :** `Comprenez le calcul des Expected Goals (xG) dans le football moderne : probabilité de tir, modèle statistique de régression logistique et analyse comparative par match.`

#### 1. Chapô & Définition Formelle (< 50 mots — Pyramide Inversée)
> Les **Expected Goals (xG)** quantifient la probabilité statistique (de 0.00 à 1.00) qu'un tir se transforme en but. Calculés par régression logistique ou algorithmes de Gradient Boosting, ils évaluent la distance à la cage, l'angle de frappe, la partie du corps utilisée et la pression défensive adverse.

#### 2. Spécification Mathématique & Formule
$$P(\text{But}) = \frac{1}{1 + e^{-(\beta_0 + \beta_1 \cdot \text{Distance} + \beta_2 \cdot \text{Angle} + \beta_3 \cdot \text{TypeAction} + \beta_4 \cdot \text{PartieCorps})}}$$
* **Variables discriminantes du modèle FootScope :**
  - **Distance axiale :** Distance Euclidienne mesurée en mètres entre l'impact du tir et le centre de la ligne de but.
  - **Angle de vue :** Angle solide sous-tendu par les deux poteaux depuis le point de tir ($\theta \in [0^\circ, 180^\circ]$).
  - **Type de passe préalable :** Balle en profondeur, centre aérien, passe en retrait, coup de pied arrêté ou transition rapide.
  - **État du jeu :** Tir en jeu ouvert (*open play*), contre-attaque éclair, coup franc direct ou penalty (valeur fixe normalisée à 0.78 xG).

#### 3. Cas d'Usage Tactique & Évaluation de Performance
* **Surperformance / Sous-performance clinique :** Le différentiel $\Delta = \text{Buts réels} - \text{xG cumulés}$ isole l'efficacité de finition du volume d'occasions créées.
* **Non-Penalty xG (npxG) :** Exclusion des penalties pour neutraliser le biais d'attribution individuelle des tireurs désignés.

#### 4. Preset Widget ECharts Associé (`PublicWidgetRenderer`)
* **Type de Graphique :** `ScatterChart` (xG par tir vs Fréquence de frappe) ou `BarChart` comparant les npxG cumulés des 5 meilleurs buteurs de Ligue 1.

---

### Fiche 2 : `/metriques/expected-assists-xa`

* **URL Canonique :** `https://myfootscope.fr/metriques/expected-assists-xa`
* **Title SEO :** `Expected Assists (xA) : Mesure de la Création d'Occasions en Football | FootScope`
* **Meta Description :** `Découvrez le fonctionnement des Expected Assists (xA) : comment mesurer la qualité réelle de la dernière passe indépendamment de l'efficacité du buteur.`

#### 1. Chapô & Définition Formelle (< 50 mots)
> Les **Expected Assists (xA)** mesurent la probabilité qu'une passe donnée devienne une passe décisive. Contrairement aux passes décisives traditionnelles tributaires de la finition du receveur, le xA attribue une valeur intrinsèque à la création en lui assignant la valeur xG du tir résultant.

#### 2. Différence Fondamentale : Passes Décisives vs xA
* **Le Biais du Buteur :** Une passe anodine suivie d'une frappe téléguidée de 30 mètres (0.02 xG) crédite historiquement une passe décisive, récompensant à tort le passeur.
* **L'Occasion Manquée :** Un caviar à 1 mètre de la cage vide (0.92 xG) vendangé par l'attaquant ne laissait aucune trace dans les statistiques traditionnelles. Le xA réhabilite l'apport créatif du passeur.

#### 3. Preset Widget ECharts Associé
* **Type de Graphique :** `BarChart` horizontal croisant xA/90 et passes clés par joueur en Top 5 européen.

---

### Fiche 3 : `/metriques/passes-progressives`

* **URL Canonique :** `https://myfootscope.fr/metriques/passes-progressives`
* **Title SEO :** `Passes Progressives (PrgP) : Indicateur de Verticalité et Gain de Terrain | FootScope`
* **Meta Description :** `Définition et règles de calcul des passes progressives : seuils de distance, élimination de lignes adverses et volume de projection vers le but adverse.`

#### 1. Chapô & Définition Formelle (< 50 mots)
> Une **Passe Progressive (PrgP)** est une transmission complétée qui fait progresser le ballon de manière significative vers la ligne de but adverse. Pour être comptabilisée, la trajectoire doit rapprocher le ballon d'au minimum 10 mètres de la ligne de but adverse (ou 15 mètres dans son propre camp).

#### 2. Règles Métriques Officielles
* **Exclusion du premier tiers :** Les passes initiées dans les 40 % défensifs nécessitent un déplacement minimal de 15 mètres pour être qualifiées de progressives.
* **Zone médiane :** Déplacement minimal de 10 mètres vers l'avant.
* **Dernier tiers :** Toute passe complétée pénétrant dans la surface de réparation adverse est automatiquement considérée comme progressive, indépendamment de la distance parcourue.

#### 3. Preset Widget ECharts Associé
* **Type de Graphique :** `ScatterChart` croisant le % de précision des passes et le volume de passes progressives par match pour les milieux axiaux.

---

### Fiche 4 : `/metriques/ppda-pressing`

* **URL Canonique :** `https://myfootscope.fr/metriques/ppda-pressing`
* **Title SEO :** `PPDA Football : Calcul du Pressing et Intensité Défensive | FootScope`
* **Meta Description :** `Guide technique du PPDA (Passes Per Defensive Action) : mesurez la hauteur du bloc équipe et l'intensité du contre-pressing dans le football moderne.`

#### 1. Chapô & Définition Formelle (< 50 mots)
> Le **PPDA (Passes Per Defensive Action)** quantifie l'intensité du pressing exercé par une équipe en zone haute. Il divise le nombre de passes concédées à l'adversaire dans ses 60 % de terrain par le total des interventions défensives actives (tacles, interceptions, fautes, duels engagés).

#### 2. Formule Mathématique
$$\text{PPDA} = \frac{\text{Passes adverses accordées dans les 60\% défensifs adverses}}{\text{Tacles} + \text{Interceptions} + \text{Fautes} + \text{Duels défensifs contestés}}$$

* **Grille d'interprétation tactique :**
  - **$\text{PPDA} \le 8.5$ :** Pressing d'élite ultra-agressif (ex: Manchester City, Arsenal, Bayern Munich).
  - **$8.5 < \text{PPDA} \le 11.5$ :** Pression médiane organisée et harcèlement par zones.
  - **$\text{PPDA} > 14.0$ :** Bloc bas passif privilégiant la couverture axiale au détriment de la récupération haute.

#### 3. Preset Widget ECharts Associé
* **Type de Graphique :** `LineChart` ou `ScatterChart` visualisant la corrélation entre PPDA et possession moyenne par équipe.

---

### Fiche 5 : `/metriques/expected-threat-xt`

* **URL Canonique :** `https://myfootscope.fr/metriques/expected-threat-xt`
* **Title SEO :** `Expected Threat (xT) : Modélisation Spatiale de la Dangerosité du Jeu | FootScope`
* **Meta Description :** `Comprendre l'Expected Threat (xT) : chaînes de Markov, discrétisation du terrain en 192 zones et valorisation des passes brise-lignes décisives.`

#### 1. Chapô & Définition Formelle (< 50 mots)
> L'**Expected Threat (xT)** évalue la probabilité qu'une action aboutisse à un but dans les phases de jeu ultérieures en divisant le terrain en une grille matricielle (16x12). Le xT récompense les transmissions et conduites de balle qui déplacent le ballon vers des zones à plus forte probabilité d'occasion.

#### 2. Mécanique Markovienne
Contrairement aux xG et xA focalisés sur la phase terminale d'un mouvement, le xT capture la valeur des actions intermédiaires (les passes avant la dernière passe, initiées par les milieux de terrain profonds ou les défenseurs centraux relanceurs).

#### 3. Preset Widget ECharts Associé
* **Type de Graphique :** `HeatmapChart` ou vue matricielle de la grille de dangerosité du terrain FootScope.

---

### Fiche 6 : `/metriques/field-tilt`

* **URL Canonique :** `https://myfootscope.fr/metriques/field-tilt`
* **Title SEO :** `Field Tilt Football : Possession Territoriale dans le Dernier Tiers | FootScope`
* **Meta Description :** `Qu'est-ce que le Field Tilt en football ? Découvrez la mesure de la domination territoriale réelle face à la possession stérile.`

#### 1. Chapô & Définition Formelle (< 50 mots)
> Le **Field Tilt** mesure la domination territoriale d'une équipe en calculant la part de ses passes complétées dans le dernier tiers par rapport au total des passes complétées par les deux formations dans leurs derniers tiers respectifs.

#### 2. Formule Mathématique
$$\text{Field Tilt (\%)} = \frac{\text{Passes réussies équipe A dans le dernier tiers adverse}}{\text{Passes réussies équipe A en zone 3} + \text{Passes réussies équipe B en zone 3}} \times 100$$
* **Détection de la possession stérile :** Une équipe affichant 65 % de possession globale mais seulement 42 % de Field Tilt fait circuler le ballon entre ses défenseurs sans jamais peser dans les 30 derniers mètres adverses.

---

## 🏷️ TICKET US 2.2 : Balisage Schema Stacking JSON-LD `@graph` (GraphRAG Ready)

Pour alimenter de façon déterministe les graphes de connaissances des moteurs (Google Knowledge Graph, Perplexity GraphRAG, Claude), chaque fiche métrique intègre un bloc JSON-LD emboîté unifié via le contexte `@graph`.

### Modèle de Balisage Structuré (`front/src/components/seo/JsonLdMetric.tsx`)

```tsx
import React from "react";

interface MetricJsonLdProps {
  slug: string;
  name: string;
  abbreviation: string;
  definition: string;
  formula: string;
  datePublished: string;
  dateModified: string;
}

export function JsonLdMetric({
  slug,
  name,
  abbreviation,
  definition,
  formula,
  datePublished,
  dateModified,
}: MetricJsonLdProps) {
  const url = `https://myfootscope.fr/metriques/${slug}`;

  const schemaGraph = {
    "@context": "https://schema.org",
    "@graph": [
      // 1. L'organisation émettrice
      {
        "@type": "Organization",
        "@id": "https://myfootscope.fr/#organization",
        "name": "MyFootScope",
        "url": "https://myfootscope.fr",
        "logo": {
          "@type": "ImageObject",
          "url": "https://myfootscope.fr/app_logo.png"
        }
      },
      // 2. L'application SaaS
      {
        "@type": "WebApplication",
        "@id": "https://myfootscope.fr/#webapp",
        "name": "MyFootScope Analytics",
        "url": "https://myfootscope.fr",
        "applicationCategory": "SportsApplication",
        "offers": {
          "@type": "Offer",
          "price": "29.00",
          "priceCurrency": "EUR"
        }
      },
      // 3. Le catalogue global de métriques (DefinedTermSet)
      {
        "@type": "DefinedTermSet",
        "@id": "https://myfootscope.fr/catalog#dataset",
        "name": "Catalogue Officiel des Métriques Tactiques FootScope",
        "url": "https://myfootscope.fr/catalog"
      },
      // 4. L'entité terminologique spécifique (DefinedTerm)
      {
        "@type": "DefinedTerm",
        "@id": `${url}#term`,
        "name": name,
        "alternateName": abbreviation,
        "description": definition,
        "inDefinedTermSet": "https://myfootscope.fr/catalog#dataset"
      },
      // 5. L'article technique d'explication (TechArticle)
      {
        "@type": "TechArticle",
        "@id": `${url}#article`,
        "headline": `${name} (${abbreviation}) : Définition, Calcul & Formule Tactique`,
        "url": url,
        "description": definition,
        "inLanguage": "fr-FR",
        "datePublished": datePublished,
        "dateModified": dateModified,
        "publisher": {
          "@id": "https://myfootscope.fr/#organization"
        },
        "about": {
          "@id": `${url}#term`
        },
        "mainEntity": {
          "@id": `${url}#term`
        }
      },
      // 6. Foire Aux Questions dédiée (FAQPage pour Rich Snippets Google)
      {
        "@type": "FAQPage",
        "@id": `${url}#faq`,
        "mainEntity": [
          {
            "@type": "Question",
            "name": `Comment calcule-t-on les ${name} (${abbreviation}) ?`,
            "acceptedAnswer": {
              "@type": "Answer",
              "text": definition
            }
          },
          {
            "@type": "Question",
            "name": `Quelle est la formule mathématique de ${abbreviation} ?`,
            "acceptedAnswer": {
              "@type": "Answer",
              "text": formula
            }
          }
        ]
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

## 🔗 TICKET US 2.3 : Routage TanStack Router & Maillage Contextuel (*Reasonable Surfer*)

### 1. Déclaration des Routes dans `front/src/routes.tsx`
Les fiches métriques sont rattachées au layout public (`publicLayout`), sans import synchrone de Clerk, garantissant une transmission instantanée du code HTML pré-rendu :

```tsx
// Déclaration de la route dynamique du cocon métrique
const metricDetailRoute = createRoute({
  getParentRoute: () => publicLayout,
  path: "/metriques/$slug",
  component: lazy(() =>
    import("@/pages/metrics/MetricDetailPage").then((m) => ({
      default: m.MetricDetailPage,
    })),
  ),
});
```

### 2. Maillage Interne Contextuel (*Reasonable Surfer Model* — Brevet US7716225B2)

Conformément à la théorie du surfeur raisonnable, la valeur de transmission de lien est maximisée lorsqu'elle se situe dans le **Main Content** et répond à une intention de navigation naturelle :

```
┌───────────────────────────────────────────────────────────────────────────┐
│                      BOUCLE DE MAILLAGE INTERNE CONTEXTUEL                │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│        [DataCatalogPage /catalog] (Inventaire des 140+ métriques)         │
│                        │                                                  │
│                        ▼ Liens d'ancres contextuelles                     │
│        [Fiche Métrique Pilier /metriques/$slug] (Contenu Haute Densité)   │
│                        │                                                  │
│                        ▼ CTA d'expérimentation en direct                  │
│        [Démo Publique Interactive /explore?measure=xg] (Aha! Moment)      │
│                        │                                                  │
│                        ▼ Funnel de monétisation                           │
│        [Tarification Pro /#tarifs] (Conversion 29 € / mois)               │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
```

1. **Depuis `DataCatalogPage.tsx` (`MetricsCatalogTable.tsx`) :**
   - Chaque ligne de tableau de métrique comporte un badge cliquable : `[Comprendre la formule]` pointant vers `/metriques/{slug}`.
2. **Sur chaque Fiche Métrique `/metriques/{slug}` :**
   - **Bandeau de Passage à l'Action :** En pied d'article, un bouton d'action directe :  
     `"Explorer cette métrique en direct sur la saison 2024/2025"` redirige vers `/explore` avec le filtre pré-sélectionné.
   - **Maillage Transversal :** Les concepts tactiques liés sont inter-reliés dans le texte (ex: *la fiche xG lie naturellement vers la fiche xA et la fiche npxG*).

---

## 🎯 Critères d'Acceptation & Validation DoD (Sprint 2)

- [ ] **Couverture Textuelle :** Les 6 pages piliers (`/metriques/*`) sont rédigées avec une définition explicite en $< 50$ mots dès l'ouverture et une densité factuelle $\ge 12$ statistiques ou entités par tranche de 100 mots.
- [ ] **Balisage Structuré :** Le composant `JsonLdMetric` imbrique 5 types Schema.org interconnectés via `@graph` et est validé à 100 % sur l'outil Google Rich Results Test (aucun avertissement).
- [ ] **Intégration Graphique Réelle :** Chaque fiche instancie `PublicWidgetRenderer` pour afficher un graphique ECharts interactif sans dépendance externe superflue.
- [ ] **Performance TTFB :** Le temps de premier octet (TTFB) des routes `/metriques/*` est mesuré sous les 120 ms grâce à l'isolation des bundles publics.
- [ ] **Veto Fair-Design Validé :** Aucun dark pattern, accès au contenu 100 % gratuit sans mur d'inscription obligatoire pour consulter les définitions mathématiques.

---

*Dossier de réalisation officiel Sprint 2 — Cocon Sémantique Éducatif & Autorité GEO MyFootScope.*
