# Startup Notifier - GraphLinq Network Registry

## Overview

Le Startup Notifier est une fonctionnalité légère intégrée dans Geth qui remplace ethstats pour notifier automatiquement le registre réseau GraphLinq lorsqu'un nœud démarre et est prêt à accepter des connexions.

## Fonctionnement

Lorsqu'un nœud Geth démarre avec un moniker configuré, il:
1. Vérifie que la blockchain est prête en appelant `eth_blockNumber` sur le RPC local
2. Envoie une notification au registre réseau GraphLinq avec le nom du nœud et le port RPC
3. Réessaie automatiquement en cas d'échec (5 tentatives avec backoff exponentiel)

## Utilisation

### Flags disponibles

#### `--moniker`
- **Description**: Nom du nœud à afficher dans le registre réseau
- **Type**: String
- **Requis**: Oui (pour activer le notifier)
- **Exemple**: `--moniker "My GLQ Node"`

#### `--notifier.url`
- **Description**: URL du registre réseau GraphLinq
- **Type**: String
- **Valeur par défaut**: `https://network.graphlinq.io/`
- **Requis**: Non

### Exemples d'utilisation

#### Configuration basique
```bash
./geth --moniker "My GLQ Node"
```

#### Configuration avec URL personnalisée
```bash
./geth --moniker "My GLQ Node" --notifier.url "https://custom-registry.example.com/"
```

#### Configuration complète avec port RPC personnalisé
```bash
./geth \
  --moniker "My GLQ Node" \
  --notifier.url "https://network.graphlinq.io/" \
  --http \
  --http.port 8545
```

## Avantages par rapport à ethstats

- **Léger**: Pas de connexion WebSocket permanente
- **Automatique**: Notification unique au démarrage
- **Simple**: Pas besoin de serveur ethstats séparé
- **Fiable**: Retry automatique avec backoff exponentiel
- **Intégré**: Aucune dépendance externe nécessaire

## Logs

Le notifier produit des logs informatifs lors de son exécution:

```
INFO Successfully notified network registry of blockchain startup moniker="My GLQ Node" endpoint="https://network.graphlinq.io/" rpc_port=8545
```

En cas d'échec temporaire:
```
INFO Startup notification attempt failed attempt=1 maxRetries=5 error="..." retryIn=10s
```

## Désactivation

Pour désactiver le startup notifier, il suffit de ne pas fournir le flag `--moniker`.

## Code source

Le code source du notifier se trouve dans:
- `/cmd/geth/startup_notifier.go` - Implémentation du notifier
- `/cmd/utils/flags.go` - Définition des flags
- `/cmd/geth/main.go` - Intégration dans le processus de démarrage

