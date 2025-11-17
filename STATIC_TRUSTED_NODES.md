# Static and Trusted Nodes Configuration

## Overview

Les fichiers `static-nodes.json` et `trusted-nodes.json` sont maintenant réactivés et fonctionnels dans cette version de go-ethereum.

## Fichiers de configuration

### static-nodes.json

Les **static nodes** sont des nœuds avec lesquels votre nœud maintient toujours une connexion. Si la connexion est perdue, elle sera automatiquement rétablie.

**Emplacement** : `<datadir>/<instance-name>/static-nodes.json`

Exemple pour geth : `~/.ethereum/geth/static-nodes.json`

### trusted-nodes.json

Les **trusted nodes** sont des nœuds qui sont toujours autorisés à se connecter, même si la limite maximale de pairs est atteinte.

**Emplacement** : `<datadir>/<instance-name>/trusted-nodes.json`

Exemple pour geth : `~/.ethereum/geth/trusted-nodes.json`

## Format des fichiers

Les deux fichiers utilisent le même format : un tableau JSON contenant des URLs enode.

```json
[
  "enode://<node-id>@<ip>:<port>",
  "enode://<node-id>@<ip>:<port>"
]
```

### Exemple complet

```json
[
  "enode://1dd9d65c4552b5eb43d5ad55a2ee3f56c6cbc1c64a5c8d659f51fcd51bace24351232b8d7821617d2b29b54b81cdefb9b3e9c37d7fd5f63270bcc9e1a6f6a439@192.168.1.100:30303",
  "enode://6f8a80d14311c39f35f516fa664deaaaa13e85b2f7493f37f6144d86991ec012937307647bd3b9a82abe2974e1407241d54947bbb39763a4cac9f77166ad92a0@10.3.58.6:30303"
]
```

## Format de l'URL enode

Format : `enode://<hex-node-id>@<ip-address>:<port>[?discport=<discovery-port>]`

Composants :
- **hex-node-id** : Identifiant unique du nœud (clé publique en hexadécimal)
- **ip-address** : Adresse IP ou nom de domaine
- **port** : Port TCP d'écoute (généralement 30303)
- **discport** : (optionnel) Port UDP pour la découverte si différent du port TCP

## Utilisation

1. Créez le fichier approprié dans le répertoire de données de votre instance
2. Ajoutez les URLs des nœuds au format JSON
3. Démarrez ou redémarrez votre nœud

Le nœud chargera automatiquement les fichiers au démarrage et affichera un message de log :

```
INFO Loaded nodes from file file=static-nodes.json count=2
INFO Loaded nodes from file file=trusted-nodes.json count=1
```

## Obtenir l'URL d'un nœud

Pour obtenir l'URL enode de votre propre nœud :

```bash
# Via console geth
> admin.nodeInfo.enode
"enode://..."

# Via RPC
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"admin_nodeInfo","params":[],"id":1}' http://localhost:8545
```

## Différences entre static et trusted nodes

| Caractéristique | Static Nodes | Trusted Nodes |
|-----------------|--------------|---------------|
| Reconnexion automatique | Oui | Non |
| Priorité de connexion | Normale | Haute |
| Ignore limite de pairs | Non | Oui |
| Usage recommandé | Nœuds de votre infrastructure | Nœuds de confiance absolue |

## Combinaison avec la configuration TOML

Les nœuds chargés depuis les fichiers JSON sont **fusionnés** avec ceux définis dans le fichier de configuration TOML :

```toml
[P2P]
StaticNodes = ["enode://..."]
TrustedNodes = ["enode://..."]
```

Tous les nœuds des deux sources seront utilisés.

## Validation

Le système valide automatiquement chaque URL enode lors du chargement. Les URLs invalides génèrent un avertissement dans les logs mais n'empêchent pas le démarrage du nœud.

```
WARN Invalid node URL in node list file=static-nodes.json url=enode://invalid error=...
```

## Migration depuis l'ancienne version

Si vous utilisiez précédemment `P2P.StaticNodes` et `P2P.TrustedNodes` dans config.toml, vous pouvez :

1. **Continuer à utiliser config.toml** : La configuration TOML fonctionne toujours
2. **Migrer vers JSON** : Déplacez vos URLs dans les fichiers JSON
3. **Utiliser les deux** : Les nœuds des deux sources sont fusionnés

## Exemples d'utilisation

### Réseau privé

Pour un réseau privé avec 3 nœuds :

**Nœud 1** (`static-nodes.json`) :
```json
[
  "enode://<node2-id>@192.168.1.101:30303",
  "enode://<node3-id>@192.168.1.102:30303"
]
```

**Nœud 2** (`static-nodes.json`) :
```json
[
  "enode://<node1-id>@192.168.1.100:30303",
  "enode://<node3-id>@192.168.1.102:30303"
]
```

### Nœud de confiance pour un validateur

```json
[
  "enode://<backup-validator>@backup.example.com:30303"
]
```

## Dépannage

### Les nœuds ne se chargent pas

1. Vérifiez l'emplacement du fichier (utilisez le répertoire de l'instance, pas le datadir racine)
2. Vérifiez la syntaxe JSON avec un validateur
3. Consultez les logs au démarrage

### Format de fichier invalide

```bash
# Valider le JSON
cat static-nodes.json | python -m json.tool
```

### Permissions de fichier

```bash
chmod 644 static-nodes.json
chmod 644 trusted-nodes.json
```

## Voir aussi

- Documentation officielle P2P : https://geth.ethereum.org/docs/interface/peer-to-peer
- Format enode : https://ethereum.org/en/developers/docs/networking-layer/network-addresses/

