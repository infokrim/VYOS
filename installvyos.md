# Réflection sur la Configuration Réseau avec VyOS et pfSense (avant installation)

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


# Mise en place et configuration d'un routeur VyOS avec une passerelle pfSense sous Proxmox

## Étape 1 : Création des interfaces sur Proxmox pour VyOS

1. **Accéder à l'interface Web de Proxmox** :
   - Ouvrez votre navigateur et connectez-vous à l'interface web de votre serveur Proxmox avec vos identifiants.

2. **Créer des interfaces réseau pour VyOS** :
   - **Sélectionnez la VM de VyOS** : Dans la vue principale de Proxmox, sélectionnez la machine virtuelle (VM) VyOS que vous avez déjà créée.
   - **Accédez à l'onglet Hardware** :
     - Dans le menu de gauche, cliquez sur "Hardware".
   - **Ajoutez une interface réseau** :
     - Cliquez sur "Add" puis sélectionnez "Network Device".
   - **Configuration de l'interface réseau** :
     - **Bridge** : Sélectionnez le bridge nommé `G2` qui connectera votre VM VyOS au réseau souhaité.
     - **Model** : Choisissez "Intel E1000" comme modèle d'interface réseau.
   - **Répétez la procédure** :
     - Recommencez cette procédure pour chaque sous-réseau que vous souhaitez connecter à VyOS.    

## Configuration de l'Interface `eth0` (vers pfSense)

L'interface `eth0` sur VyOS doit être configurée avec une adresse IP statique pour se connecter à pfSense. Voici les étapes pour configurer cette interface :

### Étape 1 : Configurer l'Adresse IP Statique pour `eth0`

Attribuez l'adresse IP `172.168.1.2/30` à l'interface `eth0` :

```bash
set interfaces ethernet eth0 address 172.168.1.2/30

3. **Appliquer les changements** :
   - Une fois toutes les interfaces configurées, appliquez les changements avec la commande suivante :
     ```bash
     commit
     ```
   - Enregistrez la configuration pour qu'elle persiste après un redémarrage :
     ```bash
     save
     ```

## Étape 3 : Configuration de la passerelle pfSense

1. **Créer une VM pour pfSense sur Proxmox** :
   - Suivez un processus similaire pour créer une VM pfSense sur Proxmox.
   - Configurez les interfaces réseau pour qu'elles correspondent aux sous-réseaux gérés par VyOS.

2. **Configurer pfSense** :
   - Une fois pfSense installé, accédez à son interface web et configurez les interfaces réseau avec les adresses IP qui correspondent à celles configurées sur VyOS.
   - Assurez-vous de configurer les routes nécessaires pour permettre la communication entre les sous-réseaux via VyOS et pfSense.

## Étape 4 : Vérification et tests

1. **Tester la connectivité** :
   - Assurez-vous que chaque sous-réseau configuré sur VyOS est accessible via pfSense.
   - Testez la connectivité entre les sous-réseaux et vérifiez que le routage fonctionne comme prévu.

2. **Monitorer les logs** :
   - Vérifiez les logs sur VyOS et pfSense pour détecter d'éventuelles erreurs ou problèmes de connectivité.
