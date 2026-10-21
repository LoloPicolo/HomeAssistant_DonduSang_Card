# EFS – Carte des collectes de don du sang

Carte Lovelace pour Home Assistant qui présente les collectes de l’Établissement français du sang remontées par l’intégration [HomeAssistant_DonduSang](https://github.com/Laurent14760/HomeAssistant_DonduSang) : prochaine collecte, agenda des semaines à venir, plan des lieux et remplissage des créneaux.

![Aperçu – thème clair](images/apercu-clair.png)

## Prérequis

- Home Assistant 2024.11 ou plus récent.
- L’intégration **HomeAssistant_DonduSang**, installée et configurée : version **1.3.0 ou plus** recommandée (adresse et heure de fin dans la liste des collectes, plusieurs zones). La carte fonctionne à partir de la 1.1.0, avec moins de détails quand un filtre est actif.

## Installation

### Via HACS (recommandé)

1. HACS → ⋮ → **Dépôts personnalisés**.
2. URL : `https://github.com/Laurent14760/HomeAssistant_DonduSang_Card`, catégorie **Tableau de bord** (*Dashboard*), puis **Ajouter**.
3. Rechercher « EFS – Carte des collectes de don du sang » dans HACS, **Télécharger**.
4. HACS déclare lui-même la ressource du tableau de bord. Rechargez la page du navigateur (Ctrl + Maj + R).
   Dans l’application mobile : Paramètres → Application compagnon → Dépannage (ou Débogage) → **Réinitialiser le cache du frontend**, puis fermez complètement l’application et rouvrez-la.

### Manuelle

1. Copier `dist/efs-collectes-card.js` dans `/config/www/efs-collectes-card.js`.
2. Paramètres → Tableaux de bord → ⋮ → **Ressources** → **Ajouter une ressource** : URL `/local/efs-collectes-card.js?v=1.2.1`, type **Module JavaScript**.
3. Recharger la page (et vider le cache de l’application mobile). À chaque mise à jour manuelle, incrémenter le `?v=` pour forcer le rechargement.

## Utilisation

Ajoutez la carte depuis le sélecteur (« EFS – Collectes de don du sang ») : **tous les réglages se font dans l’éditeur visuel**, sans YAML. Les options affichées s’adaptent à la vue choisie.

| Réglage de l’éditeur | Clé YAML | Défaut | Effet |
|---|---|---|---|
| Instance (zone surveillée) | `entity` | 1re instance trouvée | N’importe quelle entité de l’instance à afficher (ex. `sensor.efs_caen_collectes_a_venir`) |
| Vue | `vue` | `complet` | `synthese`, `agenda`, `carte` ou `complet` (les trois) |
| Collecte présentée | `commune` | `auto` | Vue synthèse : `auto` = prochaine collecte du calendrier ; un nom de commune (`CAEN`, `Bretteville-l’Orgueilleuse`…) = prochaine collecte dans cette commune. Liste proposée dans l’éditeur avec les dates ; casse, accents et tirets indifférents |
| Titre de la carte | `titre` | aucun | Titre affiché en tête de carte |
| Collectes affichées | `filtre` | `toutes` | `rdv_ouverts` : seulement les collectes où l’on peut prendre rendez-vous |
| Limiter aux prochains jours | `jours` | horizon de l’intégration | Ne montre que les collectes des N prochains jours |
| Limiter à une distance | `distance_max` | rayon de l’intégration | Resserre la zone (le plan s’adapte) ; ne peut pas élargir le rayon de l’intégration |
| Nombre de lignes | `max` | `6` | Lignes de l’agenda et de la liste de remplissage |
| Couleur d’accent | `couleur` | rouge `#B42318` | Couleur des barres, boutons et repères (`[r, g, b]` ou `"#RRGGBB"`) |
| En-tête « Don du sang » | `afficher_entete` | oui | Vue synthèse |
| Lieu et adresse | `afficher_adresse` | oui | Synthèse et agenda |
| Boutons et liens de rendez-vous | `afficher_boutons` | oui | Synthèse (boutons) et agenda (liens) |
| Pied de carte | `afficher_pied` | oui | Heure de mise à jour, nombre d’autres collectes |
| Plan des lieux | `afficher_plan` | oui | Vue carte |
| Jauge de remplissage | `afficher_jauge` | oui | Vue carte |
| Places réservées par collecte | `afficher_remplissage` | oui | Vue carte |

L’éditeur n’écrit dans la configuration que ce qui diffère des valeurs par défaut.

### Exemples

Tablette murale, uniquement l’essentiel :

```yaml
type: custom:efs-collectes-card
vue: synthese
afficher_entete: false
afficher_pied: false
```

Présenter la collecte de Caen plutôt que la plus proche dans le temps :

```yaml
type: custom:efs-collectes-card
vue: synthese
commune: CAEN
```

Si la commune n’a plus de collecte à venir (ou si les filtres de la carte l’écartent), la carte affiche la prochaine collecte du calendrier et le signale. Seules les collectes reçues de l’intégration sont proposées : pour choisir une collecte lointaine, augmentez son réglage « Nombre maximum de collectes listées » (jusqu’à 30).

Agenda des rendez-vous disponibles à moins de 5 km dans le mois :

```yaml
type: custom:efs-collectes-card
vue: agenda
titre: Où donner ce mois-ci
filtre: rdv_ouverts
distance_max: 5
jours: 30
```

### Plusieurs zones

L’intégration peut être ajoutée plusieurs fois (domicile, travail, résidence secondaire…). Chaque instance a ses propres entités, nommées d’après la zone (`sensor.efs_caen_…`, `sensor.efs_bayeux_…`). Ajoutez une carte par zone et choisissez l’instance dans l’éditeur :

```yaml
type: vertical-stack
cards:
  - type: custom:efs-collectes-card
    vue: synthese
    titre: Domicile
    entity: sensor.efs_prochaine_collecte
  - type: custom:efs-collectes-card
    vue: synthese
    titre: Travail
    entity: sensor.efs_caen_prochaine_collecte
```

![Aperçu – thème sombre](images/apercu-sombre.png)

## Comportement

- La carte suit le thème de Home Assistant (clair ou sombre) et ne charge aucune ressource externe ; les seuls liens sortants mènent au site de l’EFS (rendez-vous) et à OpenStreetMap (itinéraire, lieux).
- Elle se redessine uniquement quand l’une des deux entités change, ce qui la garde légère même sur une tablette murale.
- Les liens de rendez-vous fournis sans `https://` par l’API de l’EFS sont complétés ; tout autre schéma est ignoré.

## Dépannage

| Symptôme | Cause et solution |
|---|---|
| « Custom element doesn’t exist: efs-collectes-card » | La ressource n’est pas chargée, presque toujours à cause du cache. Navigateur : Ctrl + Maj + R. Application mobile : réinitialiser le cache du frontend **et** forcer l’arrêt de l’application. |
| « Aucune instance de l’intégration EFS trouvée » | L’intégration n’est pas installée, ou l’entité indiquée n’existe plus (renommée, instance supprimée) : choisir l’instance dans l’éditeur. |
| « Plan indisponible : mettez l’intégration à jour » | L’intégration est antérieure à la 1.1.0 ; la mettre à jour via HACS puis redémarrer Home Assistant. |
| Deux versions de la carte coexistent | L’intégration 1.1.0 embarquait elle aussi la carte. Mettre l’intégration en 1.2.0 ou plus, qui ne l’embarque plus, et supprimer toute ressource pointant vers `/efs_dondesang/efs-collectes-card.js`. |

## Licence

Code fourni tel quel, pour un usage personnel. Données : API Carto v3 de l’Établissement français du sang ; cette carte n’est pas affiliée à l’EFS.
