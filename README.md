<h1 align="center">
  <img src="./assets/images/github/header.png" alt="SIMULATION SCREEN" />
</h1>
<img src="./assets/images/github/star.gif" alt="star" />

---

# Hunger Games Analytics Simulator

## Aperçu
Simulation de type **"Battle Royale"** inspirée d'une tendance Instagram. Le notebook génère une population mondiale réaliste, simule des combats anatomiques tour par tour (membre par membre, avec état mental), puis fait s'affronter tout le monde dans un tournoi à élimination directe jusqu'au dernier survivant.

Conçu pour **vérifier les calculs simplistes des influenceurs** et fournir une estimation statistique de jusqu'à quel round un individu personnalisé peut réellement survivre. Tout tourne **localement** dans un notebook Python — aucun serveur, aucune dépendance externe hormis NumPy et Matplotlib.

## Fonctionnalités

### Génération de la population
- **Démographie pondérée** : 6 grandes régions du monde (Europe/Amérique du Nord, Asie de l'Est, Asie du Sud, Afrique, Amérique latine, Océanie) tirées selon des poids réalistes (somme normalisée à 1.0)
- **Pyramide des âges** : tirage par tranches (enfant 30 % / adulte 35 % / mature 25 % / vieux 10 %)
- **Physique cohérent** : taille dérivée de la région + courbe de croissance pour les mineurs, poids calculé à partir d'un IMC cible régional et de la taille
- **Santé** : probabilité de maladie croissante avec l'âge (`0.05 + âge/100 × 0.75`), sévérité tirée sur une loi exponentielle
- **Richesse** : facteur économique régional × facteur d'âge × loi log-normale → classement en **10 classes sociales** (de *Indigent* à *Fortune*)
- **Identité localisée** : prénom tiré selon la région et le sexe

### Combat anatomique tour par tour
- **Corps en 6 membres** : tête, torse, bras G/D, jambes G/D — chacun avec ses propres PV proportionnels au poids
- **Système au temps réel** : chaque combattant possède un *timer*, celui dont le timer est le plus bas agit ; le temps d'action dépend de la **vitesse** (`75 / vitesse`)
- **Ciblage pondéré** : le torse et la tête sont visés en priorité ; un mental bas dégrade la visée (tirs désespérés → plus de coups à la tête)
- **Précision réaliste** : `0.7 + (intelligence − vitesse adverse)/300`, modulée par le mental, **bornée dans [0.05, 0.95]**
- **Dégâts localisés** : tête ×2.5, torse ×1.0, membres ×0.6 ; un membre rompu **pénalise les stats** (jambe → vitesse, bras → force)
- **Deux conditions de mort** : physique (tête ≤ 0 ou torse ≤ −20) ou **psychologique** (effondrement mental → abandon)

### État mental & psychologique
- **Santé mentale** de 0 (brisé) à 1 (stable), influencée par l'âge, la maladie et les coups reçus
- **Trauma de combat** : la douleur et les blessures graves font chuter le mental ; un combattant **tétanisé** (mental ≤ 0.1) passe ses tours
- **Endurcissement** : les combattants entraînés encaissent mieux le choc psychologique d'infliger des blessures
- **Apaisement post-combat** : le survivant récupère une partie de son mental après la victoire

### Récupération & soins
- **Guérison physique et mentale** entre les rounds (24 h de repos par round)
- **Effet médecin** : une équipe médicale multiplie par 5 la vitesse de guérison et permet de stabiliser/réparer les membres détruits (plâtre, sutures)
- **Maladie réversible** : la sévérité décroît avec le temps (plus vite avec médecin) ; une fois guéri, le combattant **retrouve sa pleine force/vitesse**
- **Recalcul fonctionnel** : la force et la vitesse sont reconstruites selon l'état réel des paires de membres (on prend le **minimum** de chaque paire)

### Tournoi (Hunger Games)
- Combats par paires à chaque round, **exemption (bye)** pour la population impaire
- Mélange aléatoire des affrontements à chaque round
- Suivi de combattants ciblés (`tracked_ids`) : annonce de leur élimination et **profil du tueur** pour comparaison
- **Bilan final** : profil complet du champion et destin de chaque joueur suivi

### Intégration d'un joueur
- Insertion d'un combattant personnalisé (prénom, âge, sexe, taille, poids, mental, entraînement, intelligence)
- **Force et vitesse calculées automatiquement** depuis le physique imposé, selon le même modèle que la population
- Validation stricte des entrées et insertion dans un emplacement libre (jamais deux joueurs écrasés)

### Visualisations graphiques
- Parité H/F · pyramide des âges · niveau d'entraînement · état de santé (% PV) · santé mentale · profil force/vitesse · répartition par région · distribution des richesses

## Comment fonctionne le calcul

Le moteur repose sur trois étages enchaînés : **génération → combat → tournoi**.

**1. Chaque humain est un objet `Human`** construit à partir de tirages aléatoires corrélés. L'âge conditionne presque tout : un mineur a une taille et une force réduites, un senior perd en vitesse mais gagne en intelligence et en richesse. La force suit `max(1, 𝒩(30,10) + poids × 0.4)` (un poids lourd frappe fort), la vitesse `𝒩(50,8)` pénalisée après 25 ans. La maladie applique un **malus multiplicatif réversible** sur la force et la vitesse, séparé du potentiel sain stocké dans `force_max` / `vitesse_max`.

**2. Un duel (`combat_a_mort`) se joue au temps continu.** Deux *timers* avancent : à chaque itération, le combattant le plus « en avance » (timer le plus bas) attaque, et son timer progresse de `75 / vitesse` — donc **plus on est rapide, plus on agit souvent**. À chaque coup, on tire d'abord une **cible** (pondérée), puis on teste la **touche** via une précision bornée, et enfin on calcule les **dégâts** = `force × uniform(0.8,1.2) × multiplicateur de zone`. Les PV du membre baissent ; à la rupture, les stats du défenseur chutent et son mental encaisse un choc. Le duel s'arrête à la **mort physique** (tête détruite ou torse à −20) ou à la **mort psychologique** (mental à 0). Des garde-fous (double tétanie, `MAX_TOURS`) empêchent toute boucle infinie et départagent aux points si nécessaire.

**3. Le tournoi (`hunger_games`)** mélange la population, l'apparie, fait combattre chaque paire, accorde **24 h de récupération** au vainqueur (et à l'exempté), puis recommence avec les survivants — round après round — jusqu'à ce qu'il n'en reste qu'un. La récupération réinjecte de la guérison physique/mentale et **soigne progressivement la maladie**, si bien qu'un combattant peut entrer diminué dans un round et en ressortir plus fort au suivant.

> **Lecture clé** : la survie n'est *pas* qu'une affaire de force brute. Un joueur lourd frappe fort mais agit lentement ; un joueur léger et entraîné agit souvent, vise mieux (intelligence) et résiste au stress. Le mental est souvent le vrai facteur d'élimination.

## Technologies
- **Python 3** (Google Colab / Jupyter)
- **NumPy** — quelques tirages statistiques
- **Matplotlib** — visualisations (camemberts, histogrammes, nuages de points)
- Aucun serveur, aucune base de données externe — tout s'exécute dans le notebook

## Installation

**Aucune installation nécessaire.** Le projet est un notebook autonome :

- Ouvrez `Code/33Rounds.ipynb` dans **Google Colab** (ou Jupyter local).
- Exécutez les cellules d'initialisation (Imports → classe `Human` → fonctions de simulation).
- Dans la section **"Tournoi avec moi"**, renseignez votre profil dans `integrer_joueur` (taille, poids, âge, entraînement…).
- Lancez la cellule principale `hunger_games` et patientez pendant la simulation des combats.
- Consultez le journal d'exécution : il indique **à quel round vous avez été éliminé** et le profil du champion.

## Structure du projet
```
33-Rounds-Before-end-of-Humanity/
  Code/
    33Rounds.ipynb      → Notebook complet (génération + combats + tournoi + graphiques)
  BannerLogo.ipynb      → Génération du logo / bannière du projet
  README.md             → Ce fichier
  Assets/
    images/github/      → Images README (header, star, UI)
```

## Modèle de données (objet `Human`)

```python
# Identité & démographie
id, prenom, sexe, age, region_key, classe_sociale, richesse

# Physique
taille, poids

# Santé & mental
est_malade, severite_maladie        # maladie réversible
mental, mental_max                  # 0 = brisé, 1 = stable

# Combat
force, vitesse, intelligence        # valeurs courantes (malus maladie/blessures)
force_max, vitesse_max              # potentiel sain de référence
niveau_entrainement                 # 0 = débutant, 1 = expert

# Anatomie : 6 membres, chacun { pv_max, pv_actuels }
corps = {
    'tete':  {...}, 'torse': {...},
    'bras_g':{...}, 'bras_d':{...},
    'jambe_g':{...}, 'jambe_d':{...}
}
peut_se_battre                      # recalculé après chaque blessure
```

## Extraits de code utiles

```python
# Générer une population
population = [Human() for _ in range(2048)]

# Insérer un joueur personnalisé (force/vitesse calculées automatiquement)
mon_id = integrer_joueur(
    population,
    prenom="Pierre", age=26, sexe="Homme",
    taille=190, poids=94,            # grand & lourd -> force énorme, mais lent
    mental=0.7, niveau_entrainement=0.5, intelligence=110
)

# Lancer le tournoi en suivant son joueur (avec équipe médicale)
vainqueur = hunger_games(population, tracked_ids=[mon_id], equipe_medecin=True)

# Visualiser l'état de la population à un round donné
graph_age(population, round_num=1)
graph_etat_mental(population, round_num=1)
graph_force_vitesse(population, round_num=1)
```

## Aperçu de l'interface
<img src="./assets/images/github/UI.png" alt="Aperçu de la simulation" />

## Auteur
- [Pierre-Portfolio](https://github.com/Pierre-Portfolio/)

---

<p align="center">Projet réalisé en 2026.</p>
