# Transit Ridership Analysis - Chicago & Philadelphia

## 🎯 Objectif
Dashboard Power BI comparant le ridership historique et temps réel des systèmes de transport de Chicago (CTA) et Philadelphie (SEPTA), avec analyses statistiques avancées et prédictions.

## 📦 Sources de données
- Chicago RDF (transformé via Python)
- CTA Daily Boarding Totals (Excel)
- Philadelphia Ridership (CSV)
- APIs temps réel mockées (CTA Bus Tracker / SEPTA)

## ⚙️ Stack technique
- **Python** : pandas, scipy, statsmodels (ETL + tests stats + ARIMA)
- **Power BI** : Power Query, Modèle étoile, DAX avancé
- **Statistiques** : Shapiro-Wilk, T-test, ANOVA, Corrélation Pearson, ARIMA
- **DevOps** : Git, GitHub, Jira (planification), Confluence (documentation)

## 📊 Modèle de données
![Star Schema](docs/star_schema.png)
- **Fact_Ridership** : ridership journalier par ligne/ville
- **Dim_Date** : granularité jour → mois
- **Dim_Line** : lignes + mode (Bus/Rail) + ville

## 📈 Tests statistiques réalisés
| Test | Résultat | Interprétation |
|------|----------|----------------|
| Shapiro-Wilk | p < 0.05 | Distribution non normale → utilisation tests non paramétriques |
| T-test Chicago vs Philly | p = 0.003 | Différence significative (Chicago > Philly) |
| ANOVA Weekday/Weekend | p < 0.001 | Comportement ridership diffère significativement |
| Corrélation fréquence/ridership | r = 0.78 | Forte corrélation positive |

## ▶️ Comment reproduire
1. Exécuter `notebooks/data_preparation.ipynb` → génère `/output/`
2. Ouvrir `powerbi/Transit_Dashboard.pbix`
3. Refresh data (données mockées incluses)

## 🎓 Apprentissages clés
- Intégration RDF dans pipeline analytique moderne
- Combinaison Python (stats) + DAX (business logic) dans Power BI
- Storytelling data-driven pour prise de décision opérationnelle# transit-riderhip-analysis
