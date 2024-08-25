# Documentation de réflection sur la Configuration Réseau avec VyOS et pfSense (avant installation)

## Introduction
Ce document présente les différentes options de configuration réseau que nous avons envisagées pour notre infrastructure virtuelle utilisant VyOS comme routeur et pfSense comme pare-feu. Après avoir évalué les avantages et les inconvénients de chaque option, nous avons choisi une configuration initiale qui est simple, évolutive, et qui nous prépare à intégrer des concepts plus avancés, comme les VLANs, dans un futur proche.

## Architecture Réseau

### Adresses IP et Interfaces
| Nœud                  | Interface VYOS | Adresse IP     | Réseau          | Masque de Sous-réseau |
|-----------------------|----------------|----------------|-----------------|-----------------------|
| pfSense (vers internet)| -              | 10.0.0.4       | WAN             | /29                   |
| pfSense (vers VYOS)    | -              | 172.168.1.1    | LAN             | /30                   |
| VYOS (vers pfSense)    | ETH0           | 172.168.1.2    | LAN             | /30                   |
| Direction Générale     | ETH1           | 192.168.10.1   | Service DG      | /24                   |
| Direction Financière   | ETH2           | 192.168.20.1   | Service DF      | /24                   |

## Options de Configuration

### Option 1 : Deux Interfaces LAN (avec VLANs à venir)

#### Architecture Actuelle

Pour cette option, nous avons configuré VyOS avec deux interfaces LAN :
- **Interface LAN 1** : Cette interface est utilisée pour tous les services internes, regroupant l'ensemble des départements dans un même sous-réseau.
- **Interface LAN 2** : Cette interface est réservée pour des segments spéciaux ou un second groupe de services (par exemple, pour un réseau invité ou pour la gestion du réseau).

#### Planification Future pour les VLANs

Nous envisageons de segmenter logiquement le réseau en utilisant des VLANs une fois que nous aurons couvert cette notion en cours. Cela permettrait de diviser les services internes en plusieurs sous-réseaux virtuels sur une seule interface physique, ce qui offre plusieurs avantages :

- **Économie de Ressources** : Réduit le nombre d'interfaces nécessaires sur VyOS.
- **Flexibilité et Scalabilité** : Ajoute de nouveaux services ou segments sans besoin de nouvelles interfaces physiques.
- **Pratiques Courantes** : L'utilisation de VLANs est courante dans les réseaux d'entreprise.

#### Inconvénients à Prendre en Compte

- **Complexité de la Configuration des VLANs** : La configuration des VLANs demande une bonne maîtrise des concepts de réseau et des équipements.
- **Dépendance aux Switches VLAN** : Les switches doivent supporter les VLANs et être correctement configurés.

### Option 2 : Multiples Interfaces LAN (sans VLAN)

#### Architecture

Dans cette configuration, chaque service ou département a sa propre interface LAN dédiée sur VyOS. Par exemple :
- **Interface ETH1** : Connectée au sous-réseau de la Direction Générale.
- **Interface ETH2** : Connectée au sous-réseau de la Direction Financière.
- **(etc.)**

#### Avantages

- **Isolation Physique** : Chaque service est isolé physiquement, ce qui augmente la sécurité.
- **Simplicité de Segmentation** : La segmentation est simple à mettre en œuvre car chaque service a son propre sous-réseau physique.

#### Inconvénients

- **Coût Élevé** : Nécessite plus d'interfaces sur VyOS, ce qui peut consommer plus de ressources matérielles.
- **Complexité de Gestion** : La gestion de nombreuses interfaces physiques peut devenir complexe, surtout si le réseau doit évoluer.

## Choix de la Configuration : Option 1

Après avoir évalué les deux options, nous avons choisi l'**Option 1 : Deux Interfaces LAN avec Planification pour les VLANs** pour les raisons suivantes :

1. **Simplicité et Efficacité** : L'utilisation de deux interfaces LAN nous permet de démarrer avec une configuration simple, tout en laissant la possibilité d'évoluer vers une segmentation plus fine avec des VLANs.
2. **Planification Future** : En intégrant les VLANs plus tard, nous pourrons segmenter logiquement notre réseau de manière professionnelle, tout en restant économique en termes de ressources.
3. **Vision à Long Terme** : Cette approche nous prépare à l'avenir, en anticipant les besoins futurs de l'infrastructure réseau, tout en restant alignée avec notre progression dans l'apprentissage des concepts de réseau.

## Installation de VyOS et Configuration Initiale

