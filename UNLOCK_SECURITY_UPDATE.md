# Modification de sécurité : Flag --unlock

## Résumé des changements

Cette modification améliore la sécurité du déverrouillage de comptes en séparant clairement les méthodes de déverrouillage au démarrage et via RPC.

## Changements principaux

### 1. Flag `--unlock` au démarrage (✅ Autorisé)

Le flag `--unlock` fonctionne maintenant **toujours** au démarrage du nœud, indépendamment de la configuration RPC (HTTP/WebSocket).

**Exemples d'utilisation :**
```bash
# Démarrage avec RPC HTTP activé - FONCTIONNE
./geth --http --http.addr "127.0.0.1" --unlock "0xVotreAdresse"

# Démarrage avec RPC sur interface publique - FONCTIONNE
./geth --http --http.addr "0.0.0.0" --unlock "0xVotreAdresse"

# Démarrage sans RPC - FONCTIONNE
./geth --unlock "0xVotreAdresse"
```

### 2. Méthode RPC `personal.unlockAccount` (❌ Désactivée)

La méthode RPC `personal.unlockAccount` est maintenant **complètement désactivée** pour des raisons de sécurité, peu importe la configuration.

**Avant :**
- Pouvait être activée avec `--allow-insecure-unlock`
- Permettait de déverrouiller des comptes après le démarrage via HTTP/WebSocket
- Risque de sécurité : mots de passe transmis via le réseau

**Après :**
- ❌ Toujours bloquée via RPC, même avec `--allow-insecure-unlock`
- Retourne l'erreur : `"account unlock via RPC is disabled for security - use --unlock flag at startup instead"`
- ✅ Les comptes doivent être déverrouillés au démarrage uniquement

### 3. Flag `--allow-insecure-unlock` (⚠️ Deprecated)

Le flag `--allow-insecure-unlock` n'a maintenant plus d'effet sur la méthode `personal.unlockAccount` car celle-ci est complètement désactivée.

## Fichiers modifiés

1. **`cmd/geth/main.go`** (fonction `unlockAccounts`)
   - Suppression de la vérification restrictive au démarrage
   - Les comptes peuvent toujours être déverrouillés au démarrage

2. **`internal/ethapi/api.go`** (méthode `PersonalAccountAPI.UnlockAccount`)
   - Blocage complet de la méthode via RPC
   - Message d'erreur informatif redirigeant vers `--unlock`

3. **`cmd/utils/flags.go`** (définition du flag)
   - Mise à jour de la documentation du flag `--allow-insecure-unlock`
   - Indication que `personal.unlockAccount` est désactivée

4. **`node/config.go`**
   - Aucune modification fonctionnelle (nettoyage du code)

## Avantages de sécurité

1. **Protection des mots de passe** : Les mots de passe ne peuvent plus être transmis via le réseau après le démarrage
2. **Surface d'attaque réduite** : Impossible d'exploiter la méthode RPC pour déverrouiller des comptes
3. **Workflow plus sécurisé** : Force les utilisateurs à déverrouiller les comptes de manière sécurisée au démarrage
4. **Simplification** : Un seul point d'entrée pour le déverrouillage (flag `--unlock`)

## Migration

Si vous utilisiez auparavant `personal.unlockAccount` via RPC :

**Ancien workflow :**
```javascript
// Via JavaScript console ou RPC
personal.unlockAccount("0xAddress", "password", 3600)
```

**Nouveau workflow :**
```bash
# Au démarrage du nœud
./geth --unlock "0xAddress" --password /chemin/vers/password.txt
```

## Compatibilité

- ✅ Le flag `--unlock` continue de fonctionner comme avant
- ✅ Les scripts de démarrage existants ne sont pas affectés
- ❌ Les scripts utilisant `personal.unlockAccount` via RPC doivent être migrés
- ⚠️ Le flag `--allow-insecure-unlock` est conservé pour compatibilité mais n'a plus d'effet

## Notes techniques

La méthode `personal.unlockAccount` continue de fonctionner lorsque le RPC n'est pas activé (par exemple, dans la console interactive locale via IPC), mais retourne une erreur explicite quand elle est appelée via HTTP/WebSocket.

