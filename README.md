# 🦁 DV Maker : pipeline n8n multi-agents pour des CV conformes ATS et vérifiables

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![Claude](https://img.shields.io/badge/Claude-D97757?style=flat-square&logo=anthropic&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Hackathon](https://img.shields.io/badge/Hackathon-Oreegami_×_Documentum_Vitae-0F766E?style=flat-square)

> **Le défi :** générer, pour une offre d'emploi précise, un CV qui passe les filtres ATS des recruteurs, ne ressemble pas à un texte « écrit par une IA », et porte un **badge de confiance vérifiable**.
> **Le contexte :** hackathon de 2 jours **Oreegami × Documentum Vitae**, en équipe de 4 à 5 personnes.

---

## 🧠 L'idée

Documentum Vitae structure un profil professionnel autour de 5 piliers : **parcours, motivations, preuves sociales, preuves de travail et formation, compétences**. Les compétences sont rattachées au référentiel européen **ESCO**, et chaque source est qualifiée : *certifiée*, *documentée* ou *déclarative*.

DV Maker part de ces données fiables et ne garde, pour chaque offre, que ce qui est pertinent et prouvé.

## ⚙️ Le pipeline : 3 agents IA

```
Offre d'emploi (URL) + profil + compétences ESCO
                │
                ▼
   ✨ Magic 1 : score d'adéquation profil ↔ offre
                │
        Profil adapté ? ── non ──► arrêt (non-matching)
                │ oui
                ▼
   ✨ Magic 2 : génération du CV (Markdown → HTML)
                │
                ▼
   ✨ Magic 3 : audit de conformité ATS
                │
        Conforme ? ── non ──► retour à Magic 2 avec le feedback
                │ oui            (3 tentatives maximum)
                ▼
   📄 CV final (PDF)  +  🔳 QR code ──► page publique de vérification
```

| Agent | Rôle | Sortie |
|---|---|---|
| **Magic 1** | Filtrer les compétences et mesurer l'adéquation à l'offre | `fit`, `reason`, `filtered_competences` |
| **Magic 2** | Rédiger le CV ciblé à partir des seules données retenues | CV en Markdown puis HTML |
| **Magic 3** | Vérifier la conformité ATS et l'alignement avec l'offre | `ats_compliant`, `feedback` |

## 🛡️ Choix de conception

- **Boucle de correction bornée** : un compteur arrête le cycle Magic 2 ↔ Magic 3 après 3 échecs. Pas de boucle infinie en production, ni devant un jury.
- **Traçabilité** : chaque CV est stocké dans Supabase Storage et lié à un QR code qui renvoie vers une page de vérification.
- **Secrets hors du code** : clés API et identifiants remplacés par des variables `REPLACE_WITH_*`.
- **Contournement d'un bug réel** : Supabase servait les fichiers HTML en `text/plain`. Corrigé en passant l'upload n8n en mode `raw` avec `Content-Type: text/html`, avec en filet de sécurité une page `viewer.html` qui récupère et affiche le contenu.

## 🗄️ Modèle de données (Supabase)

`candidates` · `competences` · `proof_of_work` · `social_proof` · `job_offers` · `dv_exports` · `matching_logs`

## 🚀 Lancer en local

```bash
cd docker
cp .env.example .env      # renseigner identifiants et fuseau horaire
docker compose up -d      # n8n sur http://localhost:5678
```

Ensuite, importer le workflow JSON dans n8n et brancher ses propres identifiants (Claude, Supabase, conversion PDF).

## 💡 Ce que ce projet montre

- Concevoir une **orchestration multi-agents** avec des sorties structurées et des garde-fous
- Mettre l'**éthique au cœur du produit** : rien d'inventé dans le CV, chaque compétence est sourcée et vérifiable
- Livrer sous contrainte (2 jours, en équipe) un pipeline démontrable, avec un pitch scénarisé

---

<div align="center">

**Fahima Behillil** · Cheffe de projet IA & Automatisation · *Business Tech Bridge*

[LinkedIn](https://www.linkedin.com/in/fahima-be/) · [Prendre RDV (Horizons Datas)](https://calendly.com/horizonsdatas/30min) · [Tous mes projets](https://github.com/Fahima94)

</div>
