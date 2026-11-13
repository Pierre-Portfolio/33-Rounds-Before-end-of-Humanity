<h1 align="center">
  <img src="./Assets/images/github/header.png" alt="SIMULATION SCREEN" />
</h1>
<img src="./Assets/images/github/star.gif" alt="star" />

---

# Hunger Games Analytics Simulator

## Aperçu
Simulation de type **"Battle Royale"** inspirée d'une tendance Instagram. Le notebook génère une population mondiale réaliste, simule des combats anatomiques tour par tour (membre par membre, avec état mental), puis fait s'affronter tout le monde dans un tournoi à élimination directe jusqu'au dernier survivant.

Conçu pour **vérifier les calculs simplistes des influenceurs** et fournir une estimation statistique de jusqu'à quel round un individu personnalisé peut réellement survivre. Tout tourne **localement** dans un notebook Python — aucun serveur, aucune dépendance externe hormis Matplotlib.

## Fonctionnalités

### Génération de la population
- **Démographie pondérée** : 6 grandes régions du monde (Europe/Amérique du Nord, Asie de l'Est, Asie du Sud, Afrique, Amérique latine, Océanie) tirées selon des poids réalistes (somme normalisée à 1.0)
- **Pyramide des âges mondiale** : tirage par tranches de 5 ans (0-14 ans ≈ 25 %, 65 ans et plus ≈ 10 %), sans trou ni bosse entre tranches
- **Physique cohérent** : taille dérivée de la région + courbe de croissance pour les mineurs, poids calculé à partir d'un IMC cible régional et de la taille
- **Force = masse maigre × sexe × âge** : la masse grasse (formule de Deurenberg) ne donne pas de force. Mesuré sur 120 000 humains : une femme adulte a **61 %** de la force d'un homme adulte, un enfant de 8-10 ans **31 %**, un homme de 80-91 ans **58 %** de celle d'un 18-29 ans (pic à 30-39 ans)
- **Vitesse** : pic de 18 à 30 ans, déclin ensuite ; le surpoids ralentit (−2 % par point d'IMC au-delà de 25)
- **Robustesse** : à poids égal, un enfant ou un vieillard a moins de PV qu'un adulte
- **Santé** : probabilité de maladie croissante avec l'âge (`0.02 + (âge/100)² × 0.5` — prévalence réaliste : ~2 % chez les jeunes, ~50 % chez les très âgés), sévérité tirée sur une loi exponentielle
- **Résilience psychologique** : facteur individuel (loi bêta, renforcé par l'entraînement, atténué aux âges extrêmes) qui module l'encaissement du trauma
- **Richesse** : facteur économique régional × facteur d'âge × loi log-normale → classement en **10 classes sociales** (de *Indigent* à *Fortune*)
- **Identité localisée** : prénom tiré selon la région et le sexe

### Combat anatomique tour par tour
- **Corps en 6 membres** : tête, torse, bras G/D, jambes G/D — chacun avec ses propres PV proportionnels au poids
- **Système au temps réel** : chaque combattant possède un *timer*, celui dont le timer est le plus bas agit ; le temps d'action dépend de la **vitesse** (`75 / vitesse`) et **ralentit avec la fatigue**
- **Endurance (fatigue intra-combat)** : chaque action coûte de l'énergie (plus vite épuisé si lourd/âgé, plus lentement si entraîné) ; un combattant épuisé frappe moins fort, vise moins bien et agit plus lentement
- **Défense active** : le défenseur peut **esquiver/parer** selon ses réflexes (vitesse), son entraînement, son sang-froid et son souffle (esquive bornée à 75 %)
- **Ciblage piloté par l'intelligence** : un combattant lucide **et** intelligent vise les zones vitales de façon délibérée ; un mental bas fait paniquer et disperse les coups
- **Précision réaliste** : dépend de l'**entraînement + sang-froid + endurance + zone visée** (tête ×0,5, membres ×0,8, torse ×1), **bornée dans [0.05, 0.95]** ; la vitesse joue sur la cadence et l'esquive, l'intelligence sur le *ciblage*
- **Dégâts localisés** : échelle réduite (`DEGATS_SCALE`) pour des duels à **plusieurs échanges** plutôt qu'un *one-shot* ; tête ×1.5 (zone la plus rentable : elle n'a que 0,8 × les PV du torse, mais elle est deux fois plus dure à toucher), torse ×1.0, membres ×0.5 ; un membre abîmé **pénalise les stats** tout de suite (jambe → vitesse, bras → force), par la même règle pendant le combat et après le repos
- **Deux conditions de mort** : physique (tête ≤ 0 ou torse ≤ −20) ou **psychologique** (effondrement mental → abandon)

### État mental & psychologique
- **Santé mentale** de 0 (brisé) à 1 (stable), influencée par l'âge, la maladie et les coups reçus
- **Trauma de combat** : la douleur et les blessures graves font chuter le mental ; un combattant **tétanisé** (mental ≤ 0.1) passe ses tours
- **Endurcissement** : les combattants entraînés (et naturellement résilients) encaissent mieux le choc psychologique d'infliger des blessures ; tous les chocs sont modulés par la **résilience individuelle**
- **Cicatrice psychologique (PTSD)** : un trauma profond (blessure grave, mise à mort) **abaisse durablement le plafond mental** — la récupération plafonne ensuite sous le niveau initial
- **Apaisement post-combat** : le survivant récupère une partie de son mental après la victoire

### Récupération & soins
- **Remise en condition** entre les rounds (24 h de repos) : le repos restaure surtout la **condition** (fatigue, contusions, coupures superficielles, récupération proportionnelle au déficit) — pas une cicatrisation miracle
- **Blessures profondes lentes** : un membre détruit ne se « répare » **pas** en 24 h ; avec une équipe médicale on stabilise (anti-infection) et on amorce une consolidation lente (~2 PV/jour) → il reste hors d'usage plusieurs rounds
- **Effet médecin** : une équipe médicale accélère la remise en condition et la guérison de la maladie
- **Maladie réversible** : la sévérité décroît avec le temps (plus vite avec médecin) ; une fois guéri, le combattant **retrouve sa pleine force/vitesse**
- **Recalcul fonctionnel** : la force suit l'état **moyen** des deux bras, la vitesse l'état de la **pire** jambe ; le repos ne peut que rendre de la force, jamais en retirer

### Tournoi (Hunger Games)
- **33 rounds** : 2^33 = 8,6 milliards d'humains. La population simulée (8192) en est un échantillon : 20 rounds à taille constante (chacun affronte la copie d'un autre membre), puis 13 rounds de tableau exact. Limite : les adversaires des derniers rounds sont les meilleurs de l'échantillon, pas les meilleurs de 8,6 milliards
- **Graine** (`seed`) : même graine, même tournoi
- **Monte-Carlo** (`estimer_survie`) : le parcours d'un joueur est rejoué 1000 fois ; on lit une distribution (médiane, quartiles, probabilité par round), pas un tirage unique
- Combats par paires à chaque round, **exemption (bye)** pour la population impaire
- Mélange aléatoire des affrontements à chaque round
- Suivi de combattants ciblés (`tracked_ids`) : annonce de leur élimination et **profil du tueur** pour comparaison
- **Bilan final** : profil complet du champion et destin de chaque joueur suivi

### Intégration d'un joueur
- Insertion d'un combattant personnalisé (prénom, âge, sexe, taille, poids, mental, entraînement, intelligence)
- **Force et vitesse calculées sans aléa** depuis le physique imposé (`Human.stats_attendues`) : mêmes entrées, même joueur
- Validation stricte des entrées (taille en centimètres, sexe `Homme`/`Femme`, bornes sur chaque champ), avertissement si le joueur ne peut pas se battre, insertion dans un emplacement libre (jamais deux joueurs écrasés)

### Visualisations graphiques
- Parité H/F · pyramide des âges · niveau d'entraînement · état de santé (% PV) · santé mentale · profil force/vitesse · répartition par région · distribution des richesses

## Comment fonctionne le calcul

Le moteur repose sur trois étages enchaînés : **génération → combat → tournoi**.

**1. Chaque humain est un objet `Human`** construit à partir de tirages aléatoires corrélés. L'âge conditionne presque tout : un mineur a une taille, une force et une vitesse réduites, un senior perd en force et en vitesse. La force suit `masse maigre × coefficient de sexe × maturité × déclin de l'âge`, avec un aléa individuel log-normal de 15 % — **un poids lourd frappe fort, mais seulement s'il est musclé, pas gras**. La vitesse part de 50, baisse avec l'âge (avant 18 ans et après 30 ans) et avec le surpoids. La maladie applique un **malus multiplicatif réversible** sur la force et la vitesse, séparé du potentiel sain stocké dans `force_max` / `vitesse_max`.

**2. Un duel (`combat_a_mort`) se joue au temps continu.** Deux *timers* avancent : à chaque itération, le combattant le plus « en avance » (timer le plus bas) attaque, et son timer progresse de `75 / vitesse` (allongé par la **fatigue**) — donc **plus on est rapide et frais, plus on agit souvent**. À chaque action : on consomme de l'**endurance**, on tire une **cible** (instinctive, paniquée ou tactique selon le sang-froid et l'intelligence), on teste la **touche** (entraînement + mental + fatigue + zone visée, bornée), le défenseur tente une **esquive**, puis on calcule des **dégâts** = `force × uniform(0.8,1.2) × DEGATS_SCALE × multiplicateur de zone × fatigue` — volontairement faibles pour que le duel dure **plusieurs échanges**. Les PV du membre baissent ; à la rupture, les stats du défenseur chutent et son mental encaisse un choc (modulé par sa résilience). Le duel s'arrête à la **mort physique** (tête détruite ou torse à −20) ou à la **mort psychologique** (mental à 0). Des garde-fous (double tétanie, `MAX_TOURS`) empêchent toute boucle infinie et départagent aux points si nécessaire.

**3. Le tournoi (`hunger_games`)** mélange la population, l'apparie, fait combattre chaque paire, accorde **24 h de récupération** au vainqueur (et à l'exempté), puis recommence avec les survivants — round après round — jusqu'à ce qu'il n'en reste qu'un. La récupération réinjecte de la guérison physique/mentale et **soigne progressivement la maladie**, si bien qu'un combattant peut entrer diminué dans un round et en ressortir plus fort au suivant.

> **Lecture clé** : un joueur lourd frappe fort mais agit lentement ; un joueur léger et entraîné agit souvent, vise mieux (intelligence) et résiste au stress. Mesuré sur un tournoi de 33 rounds (172 031 duels) : **96,0 %** des éliminations sont physiques, **2,4 %** psychologiques, **1,4 %** des forfaits (combattant incapable). Le mental pèse donc sur la précision et l'esquive, rarement comme cause directe d'élimination.

## Technologies
- **Python 3** (Google Colab / Jupyter)
- **Matplotlib** — visualisations (camemberts, histogrammes, nuages de points)
- Aucun serveur, aucune base de données externe — tout s'exécute dans le notebook

## Installation

**Aucune installation nécessaire.** Le projet est un notebook autonome :

- Ouvrez `33Rounds.ipynb` dans **Google Colab** (ou Jupyter local).
- Exécutez les cellules d'initialisation (Imports → classe `Human` → fonctions de simulation).
- Dans la section **"Tournoi avec moi"**, renseignez votre profil dans `integrer_joueur` (taille, poids, âge, entraînement…).
- Lancez la cellule principale `hunger_games` et patientez pendant la simulation des combats (environ 20 secondes).
- Consultez le journal d'exécution : il indique **à quel round vous avez été éliminé** et le profil du champion.
- Lancez la cellule Monte-Carlo : elle donne votre round de sortie médian sur 1000 tournois.

## Structure du projet
```
33-Rounds-Before-end-of-Humanity/
  33Rounds.ipynb        → Notebook complet (génération + combats + tournoi + graphiques)
  README.md             → Ce fichier
  CLAUDE.md             → Consignes pour Claude Code
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
# Générer une population reproductible : échantillon de 8192 humains parmi 2^33
population = creer_population(8192, seed=33)

# Insérer un joueur personnalisé (force/vitesse calculées sans aléa)
MOI = dict(
    prenom="Pierre", age=26, sexe="Homme",
    taille=190, poids=94,            # taille en cm, poids en kg
    mental=0.7, niveau_entrainement=0.5, intelligence=110
)
mon_id = integrer_joueur(population, **MOI)

# Lancer les 33 rounds en suivant son joueur (avec équipe médicale)
niveaux = []
vainqueur = hunger_games(population, tracked_ids=[mon_id], equipe_medecin=True,
                         nb_rounds=33, niveaux=niveaux)

# Un tournoi = un tirage. Rejouer 1000 fois pour lire une distribution
resultat = estimer_survie(niveaux, nb_tournois=1000, seed=33, equipe_medecin=True, **MOI)
graph_survie(resultat)

# Visualiser l'état de la population à un round donné
graph_age(population, round_num=1)
graph_etat_mental(population, round_num=1)
graph_force_vitesse(population, round_num=1)
```

## Auteur
- [Pierre-Portfolio](https://github.com/Pierre-Portfolio/)

---

<p align="center">Projet réalisé en 2026.</p>
