# 🦁 DV Maker : pipeline n8n multi-agents pour des CV conformes ATS et vérifiables

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/GPT--5_mini-412991?style=flat-square&logo=openai&logoColor=white)
![S3](https://img.shields.io/badge/Stockage-S3-569A31?style=flat-square&logo=amazons3&logoColor=white)
![Hackathon](https://img.shields.io/badge/Hackathon-Oreegami_×_Documentum_Vitae-0F766E?style=flat-square)

> **Le défi :** générer, pour une offre d'emploi précise, un CV qui passe les filtres ATS des recruteurs, ne ressemble pas à un texte « écrit par une IA », et porte un **badge de confiance vérifiable**.
> **Le contexte :** hackathon de 2 jours **Oreegami × Documentum Vitae**, en équipe de 4 à 5 personnes.

---

## 🧠 L'idée

Documentum Vitae structure un profil professionnel autour de 5 piliers : **parcours, motivations, preuves sociales, preuves de travail et formation, compétences**. Les compétences sont rattachées au référentiel européen **ESCO**, et chaque source est qualifiée : *certifiée*, *documentée* ou *déclarative*.

DV Maker part de ces données fiables et ne garde, pour chaque offre, que ce qui est pertinent et prouvé.

## ⚙️ Le pipeline

```
📝 Formulaire : URL de l'offre + CV (PDF) + GitHub + LinkedIn
                │
   ┌────────────┼──────────────────┐
   ▼            ▼                  ▼
 Page offre   Texte du CV     Profil + 10 derniers dépôts GitHub (API)
   └────────────┼──────────────────┘
                ▼
        Assemblage du profil (JSON)
                │
                ▼
   ✨ Magic 1 : adéquation profil ↔ offre (mapping ESCO, proximité sémantique)
                │
        Profil adapté ? ── non ──► arrêt « non_matching » + raison
                │ oui
                ▼
   ✨ Magic 2 : génération du CV ciblé en HTML
                │
                ▼
   ✨ Magic 3 : audit ATS + cohérence avec l'offre + score de confiance
                │
        Conforme ? ── non ──► nouvelle génération avec le feedback (3 tentatives max)
                │ oui                     └──► sinon arrêt « echec_ats_max_tentatives »
                ▼
   📄 CV final  +  🔳 QR code de vérification ──► stockage S3
```

| Agent | Rôle | Sortie structurée |
|---|---|---|
| **Magic 1** | Mapper les compétences sur ESCO et juger leur transférabilité vers le poste (raisonnement sémantique, pas de simple mot-clé) | `fit`, `reason`, `filtered_competences` |
| **Magic 2** | Rédiger un CV une colonne, sections standards, factuel, sans formules creuses (« orienté résultats »…) | CV en HTML |
| **Magic 3** | Vérifier les règles ATS et la cohérence avec l'offre ; pondérer la confiance selon la source (*certifiée* > *documentée* > *déclarative*) | `ats_compliant`, `feedback` |

## 🛡️ Choix de conception

- **Boucle de correction bornée** : un compteur arrête le cycle Magic 2 ↔ Magic 3 après 3 échecs, avec un statut d'échec explicite plutôt qu'une boucle infinie.
- **Sources multiples recoupées** : le CV déclaré est confronté aux traces publiques (dépôts GitHub), ce qui fonde le score de confiance.
- **Entrées tronquées** (6 000 caractères pour l'offre, 8 000 pour le CV) pour maîtriser le coût et la fenêtre de contexte.
- **Aucun secret dans le dépôt** : identifiants et domaine de vérification à renseigner (`REPLACE_WITH_*`).

## 🚀 Utiliser le workflow

1. Dans n8n : **Import from file** → `n8n/dv_maker_workflow.json`
2. Créer et brancher ses identifiants : **OpenAI** (3 nœuds « OpenAI Model ») et **S3** (bucket de stockage)
3. Remplacer `REPLACE_WITH_VERIFY_DOMAIN` par le domaine de la page de vérification
4. Activer le workflow, puis ouvrir l'URL du **formulaire d'entrée**

## 🧭 Prochaine étape

Clarifier **ce que le QR code certifie** : le profil (diplômes, expériences vérifiés à la source) ou le document lui-même. C'est la différence entre un CV joli et un CV **digne de confiance**, et c'est tout l'enjeu contre les faux diplômes.

## 💡 Ce que ce projet montre

- Concevoir une **orchestration multi-agents** avec des sorties structurées et des garde-fous
- Mettre l'**éthique au cœur du produit** : rien d'inventé dans le CV, chaque compétence est sourcée et vérifiable
- Livrer sous contrainte (2 jours, en équipe) un pipeline démontrable, avec un pitch scénarisé

---

<div align="center">

**Fahima Behillil** · Cheffe de projet IA & Automatisation · *Business Tech Bridge*

[LinkedIn](https://www.linkedin.com/in/fahima-be/) · [Prendre RDV (Horizons Datas)](https://calendly.com/horizonsdatas/30min) · [Tous mes projets](https://github.com/Fahima94)

</div>
