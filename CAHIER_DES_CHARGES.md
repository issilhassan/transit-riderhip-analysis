# CAHIER DES CHARGES
# Transit Ridership Intelligence – Chicago & Philadelphia
# Projet INT-Maroc DATA Analyst
# Date : 06 février 2026
# Version : 1.0

## 1. CONTEXTE MÉTIER

### 1.1 Client
Agences de transport urbain :
- Chicago Transit Authority (CTA)
- Southeastern Pennsylvania Transportation Authority (SEPTA)

### 1.2 Problématique
Les agences collectent des données éclatées dans 4 formats hétérogènes :
- RDF/XML (données historiques Chicago)
- Excel (CTA Daily Boarding Totals)
- CSV (données Philadelphia)
- APIs REST (temps réel : CTA Bus Tracker, SEPTA TransitView)

→ Reporting manuel inefficace → décisions non data-driven → surcapacité/sous-capacité opérationnelle

### 1.3 Enjeu stratégique
Optimiser la fréquence des lignes selon la demande réelle pour générer des économies opérationnelles :
- Cible : **50 000 USD/mois** d'économies via réduction de fréquence sur lignes sous-utilisées
- Amélioration de l'expérience passager via augmentation de fréquence sur lignes surchargées

### 1.4 Utilisateurs finaux
- Directeurs opérationnels (niveau stratégique)
- Planificateurs de services (niveau tactique)
- Analystes métier (niveau opérationnel)

## 2. OBJECTIFS DU PROJET

### 2.1 Objectifs métier (priorisés)

| Priorité | Objectif | Indicateur de succès |
|----------|----------|----------------------|
| P0 | Suivre l'évolution du ridership en temps réel | Dashboard mis à jour quotidiennement avec données historiques + temps réel |
| P0 | Comparer performances Chicago ↔ Philadelphie | Matrice comparative avec seuils d'alerte visuels (rouge → vert) |
| P1 | Détecter anomalies (pics/baisse >30%) | Alertes visuelles sur graphiques + annotations automatiques |
| P1 | Corréler fréquence (headway) et ridership | Scatter plot avec ligne de régression (R² visible) |
| P2 | Prédire ridership 30 jours | Modèle ARIMA avec intervalles de confiance (±12%) |
| P2 | Générer recommandations actionnables | 3+ recommandations quantifiées en USD |

### 2.2 Objectifs techniques

| Critère | Spécification |
|---------|---------------|
| Intégration données | Consolidation RDF → CSV + Excel + CSV + APIs mockées |
| Modèle de données | Schéma en étoile (1 table faits + 3 dimensions actives) |
| Statistiques | ≥4 tests validés (Shapiro-Wilk, T-test, ANOVA, Corrélation Pearson) |
| Prédictions | Modèle ARIMA 30 jours avec fallback linéaire |
| UI/UX | Dashboard 2-3 pages, slicers synchronisés, thème professionnel |
| Reproductibilité | Notebook Python exécutable en <2 min sans erreurs |

## 3. PÉRIMÈTRE FONCTIONNEL

### 3.1 User Stories validées

| ID | User Story | Critère d'acceptation |
|----|------------|------------------------|
| US-01 | En tant que directeur opérationnel, je veux voir le ridership total en KPI pour évaluer la santé du système | Affichage de 4 KPIs : Total, MoM %, Volatilité hebdo, Ratio Chicago/Philly |
| US-02 | En tant qu'analyste, je veux comparer Chicago vs Philadelphie par ligne pour identifier les bonnes pratiques | Matrice avec conditional formatting (rouge = faible, vert = élevé) |
| US-03 | En tant que planificateur, je veux détecter les pics anormaux (>30%) pour investiguer les causes | Points rouges sur graphique + annotation texte "⚠️ +XX%" |
| US-04 | En tant que décideur, je veux corréler fréquence (headway) et ridership pour optimiser les ressources | Scatter plot avec ligne régression (R² visible) + insight "Réduire headway de 2 min → +15% ridership" |
| US-05 | En tant que stratège, je veux prédire le ridership des 30 prochains jours pour ajuster les budgets | Graphique prévisions ARIMA avec bandes d'erreur (ci_lower/ci_upper) |
| US-06 | En tant qu'utilisateur, je veux filtrer par ville/mode/jour pour explorer les données librement | 3 slicers synchronisés sur les 3 pages (Ville, Mode, Période) |

### 3.2 Hors périmètre (explicitement exclus)
- Intégration APIs réelles en production (mocking uniquement)
- Alertes email/SMS automatiques
- Optimisation algorithmique des horaires (recommandations manuelles seulement)
- Données météo externes

## 4. SPÉCIFICATIONS TECHNIQUES

### 4.1 Sources de données

| Source | Format | Volume | Fréquence | Statut |
|--------|--------|--------|-----------|--------|
| Chicago RDF | XML/Turtle | ~500 Mo | Historique 2001-2025 | ✅ Parsé via xml.etree.ElementTree |
| CTA Excel | .xlsx | 10 Mo | Journalier | ✅ Chargé via pandas.read_excel() |
| Philadelphia CSV | .csv | 15 Mo | Mensuel | ✅ Chargé via pandas.read_csv() |
| APIs temps réel | JSON mocké | 200 lignes | Simulé (dernières 2h) | ✅ Généré par generate_api_mock() |

### 4.2 Stack technique

| Couche | Technologie | Version | Rôle |
|--------|-------------|---------|------|
| ETL | Python 3.10+ | 3.10+ | Transformation RDF → CSV, consolidation |
| Librairies | pandas, numpy | 2.0+ | Manipulation données tabulaires |
| Statistiques | scipy.stats | 1.10+ | Tests d'hypothèses (Shapiro, T-test, ANOVA) |
| Prédictions | statsmodels | 0.14+ | Modèle ARIMA + intervalles de confiance |
| Visualisation | Power BI Desktop | 2.120+ | Dashboard interactif 2-3 pages |
| DevOps | Git, GitHub | - | Versioning code + documentation |
| Planification | Jira, Confluence | - | Suivi backlog + documentation projet |

### 4.3 Modèle de données (schéma en étoile)
┌──────────────┐       ┌──────────────────┐       ┌──────────────┐
│ Dim_Date     │◄──────│ Fact_Ridership   │──────►│ Dim_Line     │
│ • date_key   │ 1:*   │ • date_key       │ *:1   │ • line_key   │
│ • year       │       │ • line_key       │       │ • line_name  │
│ • month      │       │ • city_key       │       │ • mode       │
│ • day_type   │       │ • ridership      │       │ • city       │
└──────────────┘       │ • target         │       └──────────────┘
                       └──────────────────┘                ▲
                              ▲ │
                                │                          │
                       ┌────────┴────────┐         ┌───────┴──────┐
                       │ Dim_City        │         │ Dim_DayType  │
                       │ • city_key      │         │ • day_type   │
                       │ • city_name     │         └──────────────┘
                       │ • timezone      │
                       └─────────────────┘

### 4.4 Mesures DAX requises

| Mesure | Formule (version française) | Objectif |
|--------|-----------------------------|----------|
| Total Ridership | `SUM(fact_ridership[ridership])` | KPI principal |
| Ridership MoM % | `VAR Current = SUM(...); VAR Previous = CALCULATE(SUM(...); DATEADD(...; -1; MONTH)); RETURN DIVIDE(Current-Previous; Previous; 0)` | Croissance mensuelle |
| Weekly Volatility | `STDEVX.P(fact_ridership; fact_ridership[ridership])` | Mesure risque opérationnel |
| Chicago vs Philly Ratio | `DIVIDE(CALCULATE(SUM(...); Dim_City[city_name]="Chicago"); CALCULATE(SUM(...); Dim_City[city_name]="Philadelphia"))` | Benchmarking |
| Days Above Target % | `DIVIDE(CALCULATE(COUNTROWS(...); fact_ridership[ridership] > fact_ridership[target]); COUNTROWS(...))` | Performance vs objectif |
| Dynamic Recommendation | `IF([Ridership Forecast] < [Total Ridership]*0.95; "⚠️ Réduire fréquence"; "✅ Maintenir")` | Recommandation actionnable |

## 5. LIVRABLES ATTENDUS

| Livrable | Format | Chemin | Critère qualité |
|----------|--------|--------|-----------------|
| Notebook Python | .ipynb | `/notebooks/data_preparation.ipynb` | Génère 7 fichiers CSV dans `/output/` sans erreur |
| Fichier Power BI | .pbix | `/powerbi/Transit_Dashboard.pbix` | Modèle étoile actif + 6 mesures DAX + 3 pages dashboard |
| Documentation | README.md | `/README.md` | Explication pipeline + instructions reproduction |
| Présentation | .pptx | `/docs/presentation.pptx` | 8 slides max, storytelling métier orienté |
| Planification | .csv | `/docs/jira_export.csv` | 5+ tickets avec statut "Done" |
| Code source | GitHub repo | https://github.com/[votre-user]/transit-ridership | Structure propre + .gitignore |

## 6. PLANNING (5 jours ouvrés)

| Jour | Date | Tâche | Durée | Statut |
|------|------|-------|-------|--------|
| J1 | 02/02/2026 | Parsing RDF Chicago → CSV + consolidation sources | 8h | ✅ Done |
| J2 | 03/02/2026 | Création tables dimensions + modèle étoile Power BI | 6h | ✅ Done |
| J3 | 04/02/2026 | Implémentation mesures DAX + 4 tests statistiques | 8h | ✅ Done |
| J4 | 05/02/2026 | Dashboard UI + prédictions ARIMA + tests utilisabilité | 10h | ✅ Done |
| J5 | 06/02/2026 | Documentation finale + répétition présentation | 4h | ✅ Done |
| **Présentation** | **06/02/2026 16h40** | **Démo technique + questions jury** | **45 min** | ⏳ À venir |

## 7. CRITÈRES D'ACCEPTATION (Checklist Jury)

| Critère | Validation | Poids |
|---------|------------|-------|
| Transformation RDF → CSV | ✅ Données propres, parsing sans erreur | 15% |
| Modèle en étoile | ✅ Relations actives (bleues) dans Power BI | 20% |
| Mesures DAX | ✅ ≥6 mesures fonctionnelles avec ; (version française) | 20% |
| Tests statistiques | ✅ ≥4 tests avec interprétation métier (p<0.05) | 15% |
| Dashboard | ✅ 2-3 pages max, slicers fonctionnels, UI professionnelle | 15% |
| Insights | ✅ ≥3 recommandations quantifiées en USD | 10% |
| Documentation | ✅ README complet + schéma modèle | 5% |

## 8. MAQUETTES ÉCRANS (Wireframes)

### Page 1 : Vue d'ensemble opérationnelle
┌─────────────────────────────────────────────────────────────┐
│ TRANSIT RIDERSHIP INTELLIGENCE – CHICAGO & PHILADELPHIA     │
├───────────┬───────────┬───────────┬───────────┐             │
│ 2 450 000 │ +8,2%     │ 2 150     │ 1,23x     │ [Slicers]   │
│ Total     │ MoM       │ Volatilité│ Ratio     │             │
└───────────┴───────────┴───────────┴───────────┘             │
│                                                             │
│ ┌──────────────────────────────────────────────────────┐    │
│ │ Ridership par mois (2023-2026)                       │    │
│ │                                    Chicago ────────╮ │    │
│ │                                    Philadelphia ───╯ │    │
│ └──────────────────────────────────────────────────────┘    │
│                                                             │
│ ┌──────────────────┐ ┌──────────────────┐                   │
│ │ Répartition Bus  │ │ Répartition Rail │                   │
│ │ vs Rail          │ │ par ville        │                   │
│ └──────────────────┘ └──────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
### Page 2 : Analyse comparative
┌─────────────────────────────────────────────────────────────┐
│ COMPARAISON CHICAGO ↔ PHILADELPHIA                          │
├─────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────────┐    │
│ │ Matrice comparative (lignes × villes)                │    │
│ │ 🔴 Faible 🟡 Moyen 🟢 Élevé                        │    │
│ └──────────────────────────────────────────────────────┘    │
│ 💡 Insight : Red Line surperforme de 28% vs Market-Frankford│
├─────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────────┐    │
│ │ Corrélation headway ↔ ridership (R²=0,66)            │    │
│ │ • Réduire headway de 2 min → +15% ridership          │    │
│ └──────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
### Page 3 : Prédictions & Recommandations
┌─────────────────────────────────────────────────────────────┐
│ PRÉVISIONS & RECOMMANDATIONS                                │
├─────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────────┐    │
│ │ Résultats tests statistiques                         │    │
│ │ ✅ Shapiro-Wilk : Non-normal (p<0.05)               │     │
│ │ ✅ T-test : Différence significative (p=0.003)      │     │
│ │ ✅ ANOVA : Weekday ≠ Weekend (p<0.001)              │     │
│ └──────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────────┐    │
│ │ Prévisions ARIMA 30 jours                            │    │
│ │ 🔻 Baisse prévue -12% en février                     │    │
│ │ 💰 Économie potentielle : $48 200/mois               │    │
│ └──────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

## 9. GLOSSAIRE DES TERMES MÉTIER

| Terme | Définition | Exemple |
|-------|------------|---------|
| Ridership | Nombre de passagers transportés | 2,45M passagers/jour à Chicago |
| Headway | Temps entre 2 véhicules consécutifs | Headway moyen = 10 min sur la Red Line |
| MoM (Month-over-Month) | Croissance mensuelle | +8,2% en janvier vs décembre |
| Volatilité | Écart-type du ridership | Volatilité weekend = 2,3× weekday |
| ARIMA | Modèle prévision séries temporelles | Prévision février 2026 avec ±12% erreur |
| Slicer | Filtre interactif dans Power BI | Slicer "Ville" → Chicago/Philly |
| RDF | Resource Description Framework | Format XML pour données sémantiques Chicago |

## 10. HYPOTHÈSES & CONTRAINTES

| Type | Description | Impact |
|------|-------------|--------|
| Hypothèse 1 | Données RDF Chicago simulées si parsing échoue | ✅ Fallback réaliste avec saisonnalité |
| Hypothèse 2 | APIs temps réel mockées (pas de clé API réelle) | ✅ Reproductibilité garantie |
| Contrainte 1 | Deadline serrée : 5 jours ouvrés | ⚠️ Priorisation features P0/P1 uniquement |
| Contrainte 2 | Présentation limitée à 45 min | ⚠️ Scénario démo préparé et chronométré |
| Contrainte 3 | Dashboard limité à 2-3 pages | ⚠️ Synthèse visuelle privilégiée à exhaustivité |

## 11. SIGNATURES DE VALIDATION

| Rôle | Nom | Validation | Date |
|------|-----|------------|------|
| Chef de projet | [Votre nom] | ✅ Cahier des charges approuvé | 06/02/2026 |
| Client métier | Jury INT-Maroc | ✅ Conformité exigences | 06/02/2026 |
| Tech Lead | [Votre nom] | ✅ Livrables réalisables dans délai | 06/02/2026 |

---
