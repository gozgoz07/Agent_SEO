# 📋 Blackboard Stratégique Partagé (Shared Blackboard)
<!-- Espace de travail collaboratif inter-agents sous la gouvernance du Master Orchestrateur -->

> **Statut du Workspace :** Prêt & Opérationnel  
> **Master Agent Actif :** 👑 `@orchestrateur`  
> **Sous-Agents Disponibles :** 🔍 `@agent-seo-geo` | 🎨 `@agent-ux-ui` | 🚀 `@agent-marketing` | 🧭 `@agent-code-navigator` | 📡 `@agent-market-intelligence`  
> **Garde-Fous :** Strict Read-Only sur `football-dataviz-api` | Veto Anti-Dark Patterns Actif | Validation Humaine Primordiale

---

## 1. Registre des Ordres de Mission en Cours

| ID Mission | Donneur d'Ordre | Agent(s) Assigné(s) | Objectif Stratégique | Statut |
| :--- | :--- | :--- | :--- | :--- |
| `M-INIT` | 👤 Stratège Humain | 👑 `@orchestrateur` | Déploiement de l'architecture Master + Subagents sous Antigravity | ✅ Terminé |
| `M-001` | 👤 Stratège Humain | 🔍 `@agent-seo-geo` | Audit Nginx : Détection du blocage ClaudeBot / PerplexityBot pour RAG | ✅ Terminé |

---

## 2. Données Techniques Collectées par `@agent-code-navigator`

*Cet espace recueille les extractions condensées du code volumineux `football-dataviz-api` afin d'alimenter les réflexions sans saturer les fenêtres de contexte.*

### 🧭 Synthèse Topologique Actuelle
* **Front-End :** Stack React + `@tanstack/react-router` + Tailwind v4 OKLCH.
  - Route d'exploration rapide identifiée : `/explore`.
  - Contexte typographique adaptatif : `TypographyContext.tsx` (échelles de 0.95 à 1.18).
* **Back-End & Ingestion :** Fastify + Docker Nginx.
  - Fichier analysé : `docker/nginx/conf.d/00-maps.conf` (lignes 41 et 66).
  - Règle identifiée :
    ```nginx
    "~*(GPTBot|ChatGPT-User|ClaudeBot|PerplexityBot|AnthropicAI|Amazonbot)" "ai_crawler";
    map $bot_category $custom_robots_header {
        "ai_crawler"  "noindex, noai, noimageai, noarchive";
    }
    ```
  - `robots.txt` (`10-app-frontend.conf` L51) : `ClaudeBot` et `PerplexityBot` ne sont pas bloqués par Disallow, mais subissent le header HTTP `X-Robots-Tag: noindex, noai, noimageai, noarchive` sur toute la SPA.

---

## 3. Débats Croisés & Journal des Vetos (Peer-Review Loop)

*Cet espace consigne les vérifications contradictoires entre spécialistes avant soumission au fondateur.*

```text
[2026-10-21] 🔍 @agent-seo-geo : ALERTE GEO CRITIQUE
La directive "X-Robots-Tag: noindex, noai" injectée aux user-agents ClaudeBot et PerplexityBot 
neutralise à 100 % les citations RAG dans Perplexity AI et Claude Search. 
Recommandation : Découpler "ai_search" (autorisé en lecture index, follow) et "ai_training" (bloqué en noai).
```

---

## 4. Livrables Consolidés pour Décision Humaine (HITL)

* **Arbitrage M-001 (Configuration Nginx AI-Search) :**
  - Validation requise du fondateur pour séparer `ai_search` (`PerplexityBot`, `Claude-Search`, `ChatGPT-User`) et `ai_training` (`CCBot`, scrapers bruts) dans `00-maps.conf`.
