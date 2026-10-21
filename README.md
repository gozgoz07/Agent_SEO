# Agent_SEO : Cadre Stratégique & Agentique MyFootScope

Workspace de pair-strategy multi-agents dédié à la conception, l'évaluation et l'alignement de la chaîne de valeur du produit **MyFootScope** (Football Dataviz as a Service).

---

## 📚 Livrables Stratégiques Produit

Le dossier [`Livrables/`](Livrables/) est organisé en 4 domaines thématiques pour accueillir les livrables actuels et futurs :

### 01. Pilotage, Roadmap & Epics (`Livrables/01_Roadmap_Epics/`)
| Document | Rôle & Thématique Clé |
| :--- | :--- |
| 📋 [**Roadmap SEO Globale & 5 Epics**](Livrables/01_Roadmap_Epics/Strategie_SEO_Globale_Roadmap_Epics_myfootscope.md) | Synthèse vulgarisée pour le fondateur, plan d'action MoSCoW et découpage en 5 Epics prêtes pour la vente |
| 🏁 [**Plan d'Exécution des 5 Sprints**](Livrables/01_Roadmap_Epics/Plan_Deploiement_Sprints_Execution_MyFootScope.md) | Découpage opérationnel en 5 Sprints (35 j), User Stories, fichiers cibles, critères DoD et diagramme Gantt |
| 🛠️ [**Spécifications Techniques Sprint 1**](Livrables/01_Roadmap_Epics/Sprint_01_QuickWin_Cash_RAG_Specs_Techniques.md) | Dossier de réalisation prêt au dev : Nginx ai_search, /llms.txt, PublicWidgetRenderer ECharts et Paywall Stripe Pro 29€ |
| 🧠 [**Spécifications Techniques Sprint 2**](Livrables/01_Roadmap_Epics/Sprint_02_Autorite_GEO_Cocon_Educatif_Specs_Techniques.md) | Dossier de réalisation prêt au dev : 6 Fiches Piliers (xG, xA, PPDA...), Schema Stacking JSON-LD @graph et maillage Reasonable Surfer |

### 02. Acquisition, SEO & GEO (`Livrables/02_Acquisition_SEO_GEO/`)
| Document | Rôle & Thématique Clé |
| :--- | :--- |
| 🎯 [**Cocon Sémantique & Landing Page**](Livrables/02_Acquisition_SEO_GEO/Strategie_Cocon_Semantique_Landing_Page_myfootscope.md) | Topic Authority à 3 piliers, benchmark (DataMB/FBref), wireframe SXO & Funnel Skip |
| 🔍 [**Stratégie SEO & GEO 2026**](Livrables/02_Acquisition_SEO_GEO/Strategie_SEO_GEO_myfootscope_fr.md) | Ingestion IA, programmatic SEO, standards `/llms.txt`, Schema Stacking et IndexNow |

### 03. Expérience Utilisateur, SXO & Éthique (`Livrables/03_UX_UI_Conversion/`)
| Document | Rôle & Thématique Clé |
| :--- | :--- |
| 🎨 [**Stratégie UX/UI & Conversion**](Livrables/03_UX_UI_Conversion/Strategie_UX_UI_Conversion_2026_myfootscope.md) | SXO, Funnel Skip (30 % initiaux), Design System OKLCH feutré et scaling contextuel |
| 🛡️ [**Charte Anti-Dark Patterns**](Livrables/03_UX_UI_Conversion/Charte_Anti_Dark_Patterns_UX_myfootscope.md) | Décalogue d'interdictions éthiques (DSA art. 25, RGPD, annulation 1-clic) et pouvoir de veto |

### 04. Go-To-Market & Croissance (`Livrables/04_Marketing_GTM/`)
| Document | Rôle & Thématique Clé |
| :--- | :--- |
| 🚀 [**Stratégie Marketing & GTM**](Livrables/04_Marketing_GTM/Strategie_Marketing_GTM_myfootscope_fr.md) | Product-Led Growth (PLG), widget embed viral ($K$-factor), personas et packaging transparent |

---

## 🛠️ Configuration & Architecture Agentique

La documentation d'ingénierie et de configuration des agents est centralisée dans [`Docs/config/`](Docs/config/) :

| Document | Rôle & Usage |
| :--- | :--- |
| 📘 [**Guide Opératoire Utilisateur**](Docs/config/Guide_Operatoire_Exploitation_Agents.md) | Mode d'emploi, cheat sheet de prompts et procédures pour exploiter les agents au quotidien |
| 👑 [**Macro-Architecture Agentique**](Docs/config/Macro_Architecture_Agentique_MyFootScope.md) | Architecture multi-agents, boucle récursive de peer-review et diagramme de séquence Mermaid |
| 📋 [**Blackboard Stratégique Partagé**](Livrables/Blackboard_Strategique.md) | Espace de délibération inter-agents, logs de veto éthique et synthèses consolidées |

---

## 🤖 Gouvernance Multi-Agents & Skills Antigravity

Ce workspace implémente un modèle **Master Orchestrator + Subagents spécialisés** configuré dans `.agents/` :

* 👑 **Master Orchestrateur :** Gouverné par [`.agents/AGENTS.md`](.agents/AGENTS.md)
* 🔍 **Sous-Agent SEO/GEO :** [`.agents/skills/subagent-seo-geo/SKILL.md`](.agents/skills/subagent-seo-geo/SKILL.md)
* 🎨 **Sous-Agent UX/UI Fair-Design :** [`.agents/skills/subagent-ux-ui/SKILL.md`](.agents/skills/subagent-ux-ui/SKILL.md)
* 🚀 **Sous-Agent Marketing GTM :** [`.agents/skills/subagent-marketing/SKILL.md`](.agents/skills/subagent-marketing/SKILL.md)
* 🧭 **Sous-Agent Code Navigator :** [`.agents/skills/subagent-code-navigator/SKILL.md`](.agents/skills/subagent-code-navigator/SKILL.md) (Inspection chirurgicale du code volumineux sans saturation de contexte)
* 📡 **Sous-Agent Market Intelligence :** [`.agents/skills/subagent-market-intelligence/SKILL.md`](.agents/skills/subagent-market-intelligence/SKILL.md) (Veille concurrentielle active, surveillance SearchGPT/Perplexity et cadre réglementaire)

---

## 📚 Corpus Documentaire & Guides Méthodologiques
* 🏗️ [**Guide d'Ingénierie Agentique Antigravity (Cas Marketing)**](Docs/agents/Guide_Creation_Agents_Antigravity_Domaine_Marketing.md) : Méthode générique pas à pas, architecture en 4 piliers, boilerplates et bonnes pratiques.
* 📖 Corpus technique de référence SEO et UX/UI : [`Docs/`](Docs/)
