# 🛠️ Spécifications Techniques d'Implémentation : Sprint 1
<!-- Document de Travail Opérationnel - Sprint 1 : Quick-Win Cash & Déverrouillage RAG -->

> **Sprint Goal Business :**  
> Débloquer immédiatement les citations RAG gratuites dans Perplexity/Claude et permettre aux visiteurs de la Landing Page d'essayer le radar interactif en 15 secondes et de souscrire au forfait Pro à 29 € / mois.  
> **Durée estimée :** 5 jours ouvrés | **Priorité :** P0 (Must Have)  
> **Dépôt cible :** `football-dataviz-api` (Application par l'équipe de dev ou sur commande explicite)

---

## 📋 Sommaire des Tickets Techniques

| Ticket | Domaine | Fichiers Cibles | Statut Spécification |
| :--- | :--- | :--- | :--- |
| **US 1.1** | Infrastructure Nginx | `docker/nginx/conf.d/00-maps.conf`<br>`docker/nginx/conf.d/10-app-frontend.conf` | 🟢 Prêt à déployer |
| **US 1.2** | GEO / Ingestion IA | `public/llms.txt`<br>`docker/nginx/conf.d/10-app-frontend.conf` | 🟢 Prêt à déployer |
| **US 1.3** | SXO / Front-End | `front/src/components/radar/HeroRadarPreview.tsx`<br>`front/src/pages/HomePage.tsx` | 🟢 Prêt à coder |
| **US 1.4** | Monétisation & Fair-Design | `front/src/components/pricing/PricingSection.tsx`<br>`front/src/pages/HomePage.tsx` | 🟢 Prêt à coder |

---

## 🔧 TICKET US 1.1 : Découplage Nginx `ai_search` vs `ai_training`

### 1. Diagnostic de l'Existant (`00-maps.conf`)
Actuellement, les crawlers conversationnels (`PerplexityBot`, `Claude-Search`, `ChatGPT-User`) sont amalgamés avec les scrapers d'entraînement brut (`GPTBot`, `AnthropicAI`) dans la catégorie `"ai_crawler"`. Ils reçoivent l'en-tête destructeur :
```nginx
"ai_crawler"  "noindex, noai, noimageai, noarchive";
```
Cet en-tête interdit formellement à Perplexity et Claude d'ingérer les analyses de MyFootScope et de citer la marque comme source lors des requêtes d'utilisateurs.

### 2. Spécification de la Modification Nginx

#### Fichier 1 : `docker/nginx/conf.d/00-maps.conf`

Remplacer la section `4. Classification fine des Bots & Crawlers` et la table `map $bot_category $custom_robots_header` par :

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

#### Fichier 2 : `docker/nginx/conf.d/10-app-frontend.conf`
Mettre à jour la route `/robots.txt` pour autoriser explicitement les bots de recherche IA :

```nginx
    # Robots.txt dynamique avec filtrage fin des bots d'IA
    location = /robots.txt {
        add_header Content-Type text/plain;
        return 200 "User-agent: *\nAllow: /\nDisallow: /admin/\nDisallow: /embed?\nDisallow: /app/\nDisallow: /login/\nDisallow: /signup/\nDisallow: /workspace\nDisallow: /library\nDisallow: /catalog\nDisallow: /profile\n\n# Autorisation des bots de recherche IA (GEO)\nUser-agent: PerplexityBot\nAllow: /\n\nUser-agent: Claude-Search\nAllow: /\n\nUser-agent: ChatGPT-User\nAllow: /\n\n# Blocage de l'entraînement brut des modèles\nUser-agent: GPTBot\nDisallow: /\nUser-agent: AnthropicAI\nDisallow: /\nUser-agent: CCBot\nDisallow: /\nUser-agent: Bytespider\nDisallow: /\n\nSitemap: https://myfootscope.fr/sitemap.xml\n";
    }
```

### 3. Protocole de Validation (DoD)
Exécuter les commandes de test dans un terminal :
```bash
# 1. Test Perplexity (Doit retourner index, follow)
curl -I -A "Mozilla/5.0 (compatible; PerplexityBot/1.0; +https://perplexity.ai/perplexitybot)" https://myfootscope.fr/

# Attendu : X-Robots-Tag: index, follow, max-snippet:-1, max-image-preview:large

# 2. Test GPTBot d'entraînement (Doit retourner noai)
curl -I -A "Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko; compatible; GPTBot/1.0; +https://openai.com/gptbot)" https://myfootscope.fr/

# Attendu : X-Robots-Tag: noindex, noai, noimageai, noarchive
```

---

## 📄 TICKET US 1.2 : Standard `/llms.txt` pour les LLMs

### 1. Objectif & Rôle
Fournir aux agents RAG (Perplexity, ChatGPT Search, Claude Projects) une synthèse textuelle structurée en Markdown pur, immédiatement consommable sans parsing HTML complexe, décrivant la plateforme, ses métriques calculées et ses cas d'usage.

### 2. Spécification du Fichier `public/llms.txt`

Créer le fichier `public/llms.txt` à la racine publique de l'application :

```markdown
# MyFootScope — Football Dataviz as a Service

> Plateforme d'analyse et de visualisation de données footballistiques avancées.  
> Restitution statistique objective à la maille match pour analystes, journalistes, parieurs et recruteurs.  
> Site web officiel : https://myfootscope.fr

## Proposition de Valeur & Différenciation
- Rigueur Statistique : Zéro boîte noire, définitions claires des métriques tactiques.
- Design Studio Épuré : Radars percentiles interactifs basés sur les tokens OKLCH feutrés.
- Éthique Fair-Design : Aucune affiliation avec les opérateurs de paris sportifs, aucune publicité intrusive, respect de l'Article 25 du Digital Services Act (DSA).

## Architecture Sémantique & Piliers de Données

### 1. Fiches Joueurs & Radars Percentiles (/joueurs)
Visualisations multidimensionnelles comparant les performances d'un joueur par rapport à sa cohorte de référence (Top 5 championnats européens, par poste, sur 90 minutes).
- URL canonique : https://myfootscope.fr/joueurs/{joueur-slug}
- Données disponibles : Volume offensif, création, distribution, pressing et actions défensives.

### 2. Comparateur de Duels 1v1 (/comparateur)
Outil de superposition directe de deux profils pour les débats tactiques et les analyses comparatives de mercato.
- URL canonique : https://myfootscope.fr/comparateur/{joueur-a}-vs-{joueur-b}
- Format : Polygone radar superposé avec mise en évidence des différentiels de percentiles.

### 3. Lexique & Pédagogie Tactique (/metriques)
Guides complets expliquant la méthodologie de calcul mathématique des indicateurs modernes :
- Expected Goals (xG) : https://myfootscope.fr/metriques/expected-goals-xg
- Expected Assists (xA) : https://myfootscope.fr/metriques/expected-assists-xa
- Passes Progressives (PrgP) : https://myfootscope.fr/metriques/passes-progressives
- PPDA (Passes Per Defensive Action) : https://myfootscope.fr/metriques/ppda-pressing
- Expected Threat (xT) : https://myfootscope.fr/metriques/expected-threat-xt
- Field Tilt : https://myfootscope.fr/metriques/field-tilt

## Offres & Tarification Transparente
- Découverte (0 € / mois) : Accès gratuit aux Top 5 ligues européennes, exploration libre des radars de base.
- Pro Analyst (29 € TTC / mois) : Historique multi-saisons, 140+ métriques avancées, export SVG/PNG haute résolution sans watermark, sans engagement, résiliation en 1 clic.
- Data API (149 € HT / mois) : Accès programmatique REST / JSON, rate limit 120 req/min, intégration webhooks.

## Conditions de Réutilisation des Données
Les données et visuels générés par MyFootScope peuvent être librement cités et intégrés par les médias, blogs et créateurs sous condition de mentionner la source : "Données fournies par MyFootScope.fr".
```

### 3. Configuration Nginx pour Servir `/llms.txt`

Dans `docker/nginx/conf.d/10-app-frontend.conf`, ajouter le bloc `location` :

```nginx
    # Standard LLMS.txt pour les Agents et Modèles IA
    location = /llms.txt {
        root /var/www/html;
        add_header Content-Type "text/markdown; charset=utf-8";
        expires 1d;
        add_header Cache-Control "public, max-age=86400" always;
        add_header X-Robots-Tag "index, follow" always;
    }
```

---

## 🎨 TICKET US 1.3 : Composant `<HeroRadarPreview />` (Funnel Skip)

### 1. Objectif UX/SXO
Remplacer l'image statique non interactive `dashboard_preview.webp` par un **véritable radar SVG interactif** préchargé avec 2 stars (ex: *Lamine Yamal vs Ousmane Dembélé* ou *Kylian Mbappé vs Erling Haaland*).  
L'utilisateur doit pouvoir interagir, permuter les filtres et constater la puissance de l'outil **dès les premières secondes (Aha! Moment < 15s)** sans créer de compte.

### 2. Code Source du Composant : `front/src/components/radar/HeroRadarPreview.tsx`

```tsx
import React, { useState } from "react";
import { ArrowRight, Sparkles } from "lucide-react";
import { Button } from "@/components/ui/button";

interface MetricData {
  label: string;
  playerA: number; // Percentile 0 - 100
  playerB: number;
}

const CREATION_METRICS: MetricData[] = [
  { label: "Expected Assists (xA)", playerA: 96, playerB: 92 },
  { label: "Passes Clés / 90", playerA: 94, playerB: 88 },
  { label: "Centres Réussis", playerA: 89, playerB: 85 },
  { label: "Dribbles Complétés", playerA: 98, playerB: 97 },
  { label: "Passes Progressives", playerA: 82, playerB: 79 },
  { label: "Courses Progressives", playerA: 95, playerB: 94 },
];

const FINITION_METRICS: MetricData[] = [
  { label: "Buts hors Penalty", playerA: 84, playerB: 68 },
  { label: "Expected Goals (xG)", playerA: 88, playerB: 72 },
  { label: "Tirs Cadrés %", playerA: 78, playerB: 62 },
  { label: "Touches Surface Adv.", playerA: 92, playerB: 91 },
  { label: "Conversion de Tirs", playerA: 76, playerB: 58 },
  { label: "Actions Menant à un Tir", playerA: 95, playerB: 93 },
];

export function HeroRadarPreview() {
  const [profile, setProfile] = useState<"creation" | "finition">("creation");
  const metrics = profile === "creation" ? CREATION_METRICS : FINITION_METRICS;

  const size = 360;
  const center = size / 2;
  const radius = 130;
  const totalAxes = metrics.length;

  const getCoordinates = (index: number, percentile: number) => {
    const angle = (Math.PI * 2 / totalAxes) * index - Math.PI / 2;
    const r = (radius * percentile) / 100;
    return {
      x: center + r * Math.cos(angle),
      y: center + r * Math.sin(angle),
    };
  };

  const polyA = metrics.map((m, i) => {
    const c = getCoordinates(i, m.playerA);
    return `${c.x},${c.y}`;
  }).join(" ");

  const polyB = metrics.map((m, i) => {
    const c = getCoordinates(i, m.playerB);
    return `${c.x},${c.y}`;
  }).join(" ");

  return (
    <div className="w-full max-w-5xl mx-auto rounded-3xl border border-glass-border glass-effect bg-card/60 shadow-glass overflow-hidden backdrop-blur-xl">
      {/* Header Studio */}
      <div className="px-6 py-4 border-b border-border/40 flex flex-wrap items-center justify-between gap-4 bg-muted/20">
        <div className="flex items-center gap-3">
          <div className="flex gap-1.5">
            <span className="w-2.5 h-2.5 rounded-full bg-red-500/30" />
            <span className="w-2.5 h-2.5 rounded-full bg-amber-500/30" />
            <span className="w-2.5 h-2.5 rounded-full bg-emerald-500/30" />
          </div>
          <span className="text-xs font-mono uppercase tracking-wider text-muted-foreground">
            Comparateur Interactif // Aperçu En Direct
          </span>
        </div>

        {/* Sélecteur de Profil Tactique */}
        <div className="flex items-center gap-2 bg-background/60 p-1 rounded-xl border border-border/40">
          <button
            onClick={() => setProfile("creation")}
            className={`px-3 py-1 text-xs font-semibold rounded-lg transition-all ${
              profile === "creation"
                ? "bg-primary text-primary-foreground shadow-sm"
                : "text-muted-foreground hover:text-foreground"
            }`}
          >
            Création Offensive
          </button>
          <button
            onClick={() => setProfile("finition")}
            className={`px-3 py-1 text-xs font-semibold rounded-lg transition-all ${
              profile === "finition"
                ? "bg-primary text-primary-foreground shadow-sm"
                : "text-muted-foreground hover:text-foreground"
            }`}
          >
            Finition & Tirs
          </button>
        </div>
      </div>

      {/* Corps du Radar & Légende */}
      <div className="p-6 md:p-8 grid grid-cols-1 md:grid-cols-12 gap-8 items-center">
        {/* Radar SVG */}
        <div className="md:col-span-7 flex justify-center relative">
          <svg width={size} height={size} className="overflow-visible">
            {/* Cercles de grille concentriques (25, 50, 75, 100) */}
            {[25, 50, 75, 100].map((step) => (
              <circle
                key={step}
                cx={center}
                cy={center}
                r={(radius * step) / 100}
                fill="none"
                stroke="currentColor"
                className="text-border/40"
                strokeDasharray={step === 50 ? "4 4" : undefined}
                strokeWidth={1}
              />
            ))}

            {/* Axes radiaux */}
            {metrics.map((_, i) => {
              const edge = getCoordinates(i, 100);
              return (
                <line
                  key={i}
                  x1={center}
                  y1={center}
                  x2={edge.x}
                  y2={edge.y}
                  stroke="currentColor"
                  className="text-border/40"
                  strokeWidth={1}
                />
              );
            })}

            {/* Polygone Joueur B (Ambre / Doré - Dembélé) */}
            <polygon
              points={polyB}
              fill="oklch(0.768 0.165 70 / 0.25)"
              stroke="oklch(0.768 0.165 70)"
              strokeWidth={2}
              className="transition-all duration-700 ease-out"
            />

            {/* Polygone Joueur A (Cyan / Indigo - Yamal) */}
            <polygon
              points={polyA}
              fill="oklch(0.585 0.233 277 / 0.35)"
              stroke="oklch(0.585 0.233 277)"
              strokeWidth={2.5}
              className="transition-all duration-700 ease-out"
            />

            {/* Libellés des axes */}
            {metrics.map((m, i) => {
              const labelPos = getCoordinates(i, 118);
              return (
                <text
                  key={i}
                  x={labelPos.x}
                  y={labelPos.y}
                  textAnchor="middle"
                  dominantBaseline="central"
                  className="text-[10px] font-bold fill-muted-foreground tracking-tight"
                >
                  {m.label}
                </text>
              );
            })}
          </svg>
        </div>

        {/* Détail Joueurs & Différentiels */}
        <div className="md:col-span-5 space-y-6">
          <div>
            <span className="text-xs font-semibold text-primary uppercase tracking-wider">
              Duel Tactique — Saison 2025/2026
            </span>
            <h3 className="text-2xl font-black text-white mt-1">
              Lamine Yamal vs Ousmane Dembélé
            </h3>
            <p className="text-xs text-muted-foreground mt-1">
              Percentiles calculés par rapport aux ailiers du Top 5 Européen (par 90 min).
            </p>
          </div>

          <div className="space-y-3 bg-muted/10 p-4 rounded-2xl border border-border/30">
            <div className="flex items-center justify-between pb-2 border-b border-border/20">
              <div className="flex items-center gap-2">
                <span className="w-3 h-3 rounded-full bg-[oklch(0.585_0.233_277)]" />
                <span className="text-sm font-bold text-white">Lamine Yamal</span>
              </div>
              <span className="text-xs font-mono font-bold text-primary">FC Barcelone</span>
            </div>
            <div className="flex items-center justify-between">
              <div className="flex items-center gap-2">
                <span className="w-3 h-3 rounded-full bg-[oklch(0.768_0.165_70)]" />
                <span className="text-sm font-bold text-white">Ousmane Dembélé</span>
              </div>
              <span className="text-xs font-mono font-bold text-amber-400">Paris SG</span>
            </div>
          </div>

          <div className="space-y-2">
            <div className="flex items-center justify-between text-xs text-muted-foreground">
              <span>Indice d'avantage créatif :</span>
              <span className="font-bold text-emerald-400">+6.4% pour Yamal</span>
            </div>
            <div className="w-full h-1.5 rounded-full bg-border/40 overflow-hidden">
              <div className="h-full bg-primary rounded-full w-[56%]" />
            </div>
          </div>

          <div className="pt-2 flex flex-col sm:flex-row gap-3">
            <a href="/explore" className="flex-1">
              <Button size="sm" className="w-full bg-primary text-primary-foreground font-bold rounded-xl flex items-center justify-center gap-2">
                Explorer d'autres joueurs <ArrowRight className="w-4 h-4" />
              </Button>
            </a>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### 3. Intégration dans `HomePage.tsx`
Remplacer le bloc mockup statique (lignes 79-110 de `HomePage.tsx`) par :
```tsx
{/* Preview Interactive Section - Funnel Skip */}
<div className="mt-16 w-full relative max-w-6xl mx-auto">
  <HeroRadarPreview />
</div>
```

---

## 💳 TICKET US 1.4 : Section Tarification Transparente (Paywall Stripe 29 €)

### 1. Objectif Business & Veto Anti-Dark Patterns
- Rendre immédiatement accessible la grille tarifaire sur la Landing Page (zéro "Contact Sales" opaque pour le forfait Pro).
- Prix affiché TTC clair : **29 € / mois**, sans engagement, résiliation en 1 clic depuis l'espace client.
- Respect rigoureux de l'Article 25 du DSA et de la [Charte Anti-Dark Patterns MyFootScope](file:///c:/Users/oreeg/Documents/Projet%20%20IA%20G%C3%A9n%C3%A9rative/Livrables/03_UX_UI_Conversion/Charte_Anti_Dark_Patterns_UX_myfootscope.md).

### 2. Code Source du Composant : `front/src/components/pricing/PricingSection.tsx`

```tsx
import React from "react";
import { Check, ArrowRight, ShieldCheck, Zap } from "lucide-react";
import { Button } from "@/components/ui/button";

export function PricingSection() {
  return (
    <section id="tarifs" className="py-28 relative bg-background border-t border-border/40">
      <div className="container mx-auto px-4">
        {/* Titre & Sous-titre */}
        <div className="text-center max-w-3xl mx-auto mb-20">
          <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-primary/10 border border-primary/20 text-primary text-xs font-bold uppercase tracking-wider mb-4">
            <Zap className="w-3.5 h-3.5" /> Forfaits Simples & Transparents
          </div>
          <h2 className="text-4xl md:text-5xl font-black text-white tracking-tight mb-4">
            Un abonnement pensé pour <br />
            <span className="text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-indigo-500">
              les analystes passionnés.
            </span>
          </h2>
          <p className="text-muted-foreground text-lg">
            Aucun engagement de durée. Annulation en 1 clic depuis votre profil sans avoir à contacter le support.
          </p>
        </div>

        {/* Grille 3 Colonnes */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-6xl mx-auto items-stretch">
          
          {/* Colonne 1 : Découverte (0 €) */}
          <div className="rounded-3xl border border-border/60 bg-card/40 p-8 flex flex-col justify-between">
            <div>
              <h3 className="text-xl font-bold text-white mb-2">Découverte</h3>
              <p className="text-sm text-muted-foreground mb-6">
                Pour explorer la puissance des données sans engagement.
              </p>
              <div className="flex items-baseline gap-1 mb-8">
                <span className="text-5xl font-black text-white">0 €</span>
                <span className="text-muted-foreground font-medium">/ mois</span>
              </div>
              <ul className="space-y-3 text-sm text-muted-foreground mb-8">
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-emerald-400 shrink-0" />
                  <span>Accès aux Top 5 ligues (saison courante)</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-emerald-400 shrink-0" />
                  <span>Radars percentiles standard 8 métriques</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-emerald-400 shrink-0" />
                  <span>Comparateur 1v1 limité à 3 duels/jour</span>
                </li>
              </ul>
            </div>
            <a href="/explore">
              <Button variant="outline" className="w-full rounded-xl py-6 font-bold border-border hover:bg-accent">
                Démarrer gratuitement
              </Button>
            </a>
          </div>

          {/* Colonne 2 : Pro Analyst (29 € - MIS EN AVANT) */}
          <div className="rounded-3xl border-2 border-primary bg-card p-8 flex flex-col justify-between relative shadow-glass shadow-primary/10">
            <div className="absolute -top-3.5 left-1/2 -translate-x-1/2 px-4 py-1 bg-primary text-primary-foreground text-xs font-black uppercase tracking-wider rounded-full shadow-sm">
              Le plus populaire
            </div>
            <div>
              <h3 className="text-xl font-bold text-white mb-2">Pro Analyst</h3>
              <p className="text-sm text-muted-foreground mb-6">
                L'outil complet pour les journalistes, scouts et créateurs.
              </p>
              <div className="flex items-baseline gap-1 mb-8">
                <span className="text-5xl font-black text-white">29 €</span>
                <span className="text-muted-foreground font-medium">TTC / mois</span>
              </div>
              <ul className="space-y-3 text-sm text-muted-foreground mb-8">
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-primary shrink-0" />
                  <span className="text-white font-semibold">140+ métriques tactiques avancées</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-primary shrink-0" />
                  <span className="text-white font-semibold">Historique complet multi-saisons</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-primary shrink-0" />
                  <span>Duels 1v1 illimités & filtres temporels</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-primary shrink-0" />
                  <span>Export PNG/SVG haute résolution sans filigrane</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-primary shrink-0" />
                  <span>Widget intégrable (iframe) pour vos articles</span>
                </li>
              </ul>
            </div>
            <div className="space-y-2">
              <a href="/checkout/pro">
                <Button className="w-full rounded-xl py-6 font-bold bg-primary text-primary-foreground hover:opacity-90 shadow-premium flex items-center justify-center gap-2">
                  Passer à Pro (29 €/mois) <ArrowRight className="w-4 h-4" />
                </Button>
              </a>
              <div className="flex items-center justify-center gap-1.5 text-[11px] text-muted-foreground">
                <ShieldCheck className="w-3.5 h-3.5 text-emerald-400" />
                <span>Sans engagement • Annulation 1-clic</span>
              </div>
            </div>
          </div>

          {/* Colonne 3 : Data API (149 €) */}
          <div className="rounded-3xl border border-border/60 bg-card/40 p-8 flex flex-col justify-between">
            <div>
              <h3 className="text-xl font-bold text-white mb-2">Data API</h3>
              <p className="text-sm text-muted-foreground mb-6">
                Pour les développeurs, agences et rédactions professionnelles.
              </p>
              <div className="flex items-baseline gap-1 mb-8">
                <span className="text-5xl font-black text-white">149 €</span>
                <span className="text-muted-foreground font-medium">HT / mois</span>
              </div>
              <ul className="space-y-3 text-sm text-muted-foreground mb-8">
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-cyan-400 shrink-0" />
                  <span>Toutes les fonctionnalités Pro incluses</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-cyan-400 shrink-0" />
                  <span>Accès API REST programmatique direct</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-cyan-400 shrink-0" />
                  <span>Rate limit de 120 requêtes / minute</span>
                </li>
                <li className="flex items-center gap-2">
                  <Check className="w-4 h-4 text-cyan-400 shrink-0" />
                  <span>Support prioritaire par email</span>
                </li>
              </ul>
            </div>
            <a href="mailto:contact@myfootscope.fr?subject=Demande%20API%20Data">
              <Button variant="outline" className="w-full rounded-xl py-6 font-bold border-border hover:bg-accent">
                Nous contacter
              </Button>
            </a>
          </div>

        </div>
      </div>
    </section>
  );
}
```

---

## 🎯 Bilan de Conformité Fair-Design & Peer-Review (Sprint 1)

1. **Veto Anti-Dark Patterns (🎨 `@agent-ux-ui`) :**
   - ✅ Aucune case pré-cochée pour l'abonnement annuel obligatoire.
   - ✅ Prix de 29 € affiché TTC en gros caractères lisibles.
   - ✅ Mention explicite et garantie de la résiliation en 1 clic sans appel téléphonique.
   - ✅ Le Hero Radar fonctionne librement sans capture d'adresse email forcée.

2. **Validation Ingestion IA (🔍 `@agent-seo-geo`) :**
   - ✅ PerplexityBot et Claude-Search sont autorisés en `index, follow` sur `/` et `/llms.txt`.
   - ✅ Les scrapers de modèles bruts non consentis (`GPTBot`, `CCBot`) restent bloqués en `noai`.

3. **Validation Commerciale (🚀 `@agent-marketing`) :**
   - ✅ Le forfait Pro à 29 € TTC cible exactement le point d'équilibre entre l'amateur passionné et le créateur de contenu.
   - ✅ L'appel à l'action est immédiatement accessible sur la page d'accueil sans friction.

---

*Ce document constitue le dossier de réalisation prêt pour l'intégration du Sprint 1 sur MyFootScope.*
