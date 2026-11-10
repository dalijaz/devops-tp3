# Portail DevOps - TP 3 : Git, Branches, SSH & CI/CD

Ce dépôt sert de support de démonstration en direct pour le **TP 3 (3 heures)** destiné aux élèves-ingénieurs : gestion de versions avec **Git**, stratégie de branches, sécurisation des accès **SSH** et automatisation **CI/CD avec GitHub Actions**.

> **Périmètre technique :** Projet 100 % statique (**HTML5 / CSS3 / JavaScript léger** + **YAML**). Aucune base de données (pas de MySQL, pas de PHP, pas de `.env`).

---

## 1. Architecture pédagogique du TP

```text
+-----------------------------------+
|   Poste Étudiant (Windows)        |
|   - Git Bash (CLI Git & SSH)      |
|   - SmarTTY (Administration VPS)  |
+-----------------+-----------------+
                  |
                  | git push (HTTPS / SSH)
                  v
+-----------------------------------+
|   Dépôt Distant GitHub            |
|   - Branches & Tags (v1.0, v2.0)  |
|   - GitHub Secrets (SSH_KEY...)   |
+-----------------+-----------------+
                  |
                  | Déclenchement automatique (push sur main)
                  v
+-----------------------------------+
|   GitHub Actions (CI/CD)          |
|   1. ci-verification (HTML/CSS)   |
|   2. cd-deploiement (SSH)         |
+-----------------+-----------------+
                  |
                  | ssh <user>@<VPS_HOST> -> git pull origin main
                  v
+-----------------------------------+
|   Serveur de Production           |
|   - VPS Contabo                   |
|   - OS : Ubuntu 22.04 LTS         |
|   - Serveur Web : Apache 2        |
|   - Cible : /var/www/html/<user>  |
+-----------------------------------+
```

---

## 2. Structure des branches et historique du dépôt

Le dépôt illustre un workflow Git structuré avec des fusions explicites (`--no-ff`) et des tags annotés :

- **`main`** : Branche principale de production. Contient les versions stables taguées (`v1.0`, `v2.0`).
- **`feature/ui-dashboard`** *(fusionnée puis supprimée)* : Ajout de la feuille de style `style.css` et refonte du tableau de bord HTML5.
- **`ci/github-actions`** *(fusionnée puis supprimée)* : Ajout du pipeline `.github/workflows/deploy.yml` et passage de l'application en `v2.0`.
- **`feature/monitoring-widget`** *(ouverte et non fusionnée)* : Branche de démonstration ajoutant un widget JavaScript léger d'état serveur (Apache 2 / Ubuntu 22.04 LTS). Elle est volontairement laissée ouverte pour permettre aux étudiants d'observer une branche parallèle et de s'exercer à la fusion.

---

## 3. Commandes Git à tester en classe

### Observer l'historique, les branches et les tags
```bash
# Afficher le graphe complet des commits, branches et tags
git log --oneline --graph --all --decorate

# Afficher l'historique détaillé avec auteurs et dates
git log --all --format="%h | %ad | %an <%ae> | %s" --date=short

# Lister toutes les branches locales (et distantes avec -a)
git branch -a

# Lister et inspecter les tags annotés
git tag -n
git show v2.0
```

### Comparer et naviguer entre les branches
```bash
# Comparer main avec la branche ouverte feature/monitoring-widget
git diff main..feature/monitoring-widget

# Basculer sur la branche de fonctionnalité pour inspecter le code
git switch feature/monitoring-widget

# Revenir sur la branche principale
git switch main
```

### Exercice pratique : fusionner la branche ouverte
```bash
# Se placer sur main puis fusionner avec un commit de merge explicite
git switch main
git merge --no-ff feature/monitoring-widget -m "merge: intégration du widget de monitoring"
```

---

## 4. Pipeline CI/CD (`.github/workflows/deploy.yml`)

Le workflow s'exécute automatiquement à chaque `push` sur `main` en deux étapes séquentielles :

1. **`ci-verification`** : Récupère le code (`actions/checkout@v4`) et contrôle la présence ainsi que la validité syntaxique de base de `index.html` et `style.css`.
2. **`cd-deploiement`** *(dépend de `ci-verification`)* : Se connecte en SSH au VPS Contabo via les secrets GitHub (`VPS_SSH_KEY`, `VPS_HOST`, `VPS_USER`, `DEPLOY_PATH`) et exécute `git pull origin main` dans `/var/www/html/<user>`.
