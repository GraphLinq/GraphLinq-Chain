# Exemples d'utilisation du fichier metadata.json

## Exemples simples

### 1. Afficher la hauteur actuelle de la blockchain
```bash
cat ~/.ethereum/metadata.json | jq '.blockNumber'
```

### 2. Afficher toutes les informations du dernier bloc
```bash
cat ~/.ethereum/metadata.json | jq '.'
```

### 3. Afficher uniquement le hash du bloc
```bash
cat ~/.ethereum/metadata.json | jq -r '.blockHash'
```

### 4. Afficher le nombre de transactions dans le dernier bloc
```bash
cat ~/.ethereum/metadata.json | jq '.txCount'
```

## Scripts de monitoring

### 1. Surveiller en temps réel les nouveaux blocs
```bash
#!/bin/bash
# watch_blocks.sh

watch -n 1 'cat ~/.ethereum/metadata.json | jq "{
  block: .blockNumber,
  txs: .txCount,
  gasUsed: .gasUsed,
  miner: .miner
}"'
```

### 2. Vérifier l'état de synchronisation
```bash
#!/bin/bash
# check_sync.sh

CURRENT_BLOCK=$(cat ~/.ethereum/metadata.json | jq '.blockNumber')
CURRENT_TIME=$(cat ~/.ethereum/metadata.json | jq '.timestamp')
NOW=$(date +%s)
DIFF=$((NOW - CURRENT_TIME))

echo "Hauteur actuelle: $CURRENT_BLOCK"
echo "Dernière mise à jour: il y a $DIFF secondes"

if [ $DIFF -lt 30 ]; then
    echo "✓ Le nœud est synchronisé"
    exit 0
else
    echo "✗ Le nœud est en retard de $DIFF secondes"
    exit 1
fi
```

### 3. Envoyer une alerte si le nœud ne se synchronise plus
```bash
#!/bin/bash
# alert_sync.sh

METADATA_FILE=~/.ethereum/metadata.json
MAX_DELAY=300  # 5 minutes

while true; do
    if [ ! -f "$METADATA_FILE" ]; then
        echo "⚠️  Fichier metadata.json introuvable"
        # Envoyer une alerte (email, Slack, etc.)
        sleep 60
        continue
    fi
    
    TIMESTAMP=$(cat $METADATA_FILE | jq '.timestamp')
    NOW=$(date +%s)
    DIFF=$((NOW - TIMESTAMP))
    
    if [ $DIFF -gt $MAX_DELAY ]; then
        echo "🚨 ALERTE: Le nœud n'a pas synchronisé depuis $DIFF secondes"
        # Envoyer une alerte
    else
        echo "✓ Nœud OK (dernier bloc il y a $DIFF secondes)"
    fi
    
    sleep 60
done
```

## Intégration avec des outils de monitoring

### 1. Prometheus exporter simple
```bash
#!/bin/bash
# ethereum_exporter.sh

cat << EOF > /var/lib/prometheus/node_exporter/ethereum.prom
# HELP ethereum_block_height Hauteur actuelle de la blockchain
# TYPE ethereum_block_height gauge
ethereum_block_height $(cat ~/.ethereum/metadata.json | jq '.blockNumber')

# HELP ethereum_block_transactions Nombre de transactions dans le dernier bloc
# TYPE ethereum_block_transactions gauge
ethereum_block_transactions $(cat ~/.ethereum/metadata.json | jq '.txCount')

# HELP ethereum_block_gas_used Gas utilisé dans le dernier bloc
# TYPE ethereum_block_gas_used gauge
ethereum_block_gas_used $(cat ~/.ethereum/metadata.json | jq '.gasUsed')

# HELP ethereum_block_timestamp Timestamp du dernier bloc
# TYPE ethereum_block_timestamp gauge
ethereum_block_timestamp $(cat ~/.ethereum/metadata.json | jq '.timestamp')
EOF
```

### 2. Script pour Grafana/InfluxDB
```bash
#!/bin/bash
# send_to_influxdb.sh

INFLUXDB_URL="http://localhost:8086/write?db=ethereum"

BLOCK=$(cat ~/.ethereum/metadata.json | jq '.blockNumber')
TX_COUNT=$(cat ~/.ethereum/metadata.json | jq '.txCount')
GAS_USED=$(cat ~/.ethereum/metadata.json | jq '.gasUsed')
TIMESTAMP=$(cat ~/.ethereum/metadata.json | jq '.timestamp')

curl -i -XPOST "$INFLUXDB_URL" --data-binary "
ethereum_block,host=$(hostname) height=$BLOCK,txs=$TX_COUNT,gas=$GAS_USED $((TIMESTAMP * 1000000000))
"
```

## Exemples Python

### 1. Lire le fichier metadata.json en Python
```python
#!/usr/bin/env python3
import json
from pathlib import Path

def read_metadata():
    metadata_path = Path.home() / '.ethereum' / 'metadata.json'
    with open(metadata_path, 'r') as f:
        return json.load(f)

def main():
    metadata = read_metadata()
    print(f"Block: {metadata['blockNumber']}")
    print(f"Hash: {metadata['blockHash']}")
    print(f"Transactions: {metadata['txCount']}")
    print(f"Gas Used: {metadata['gasUsed']}/{metadata['gasLimit']}")

if __name__ == '__main__':
    main()
```

### 2. Surveiller les blocs en Python
```python
#!/usr/bin/env python3
import json
import time
from pathlib import Path

def watch_blocks():
    metadata_path = Path.home() / '.ethereum' / 'metadata.json'
    last_block = 0
    
    while True:
        try:
            with open(metadata_path, 'r') as f:
                metadata = json.load(f)
            
            current_block = metadata['blockNumber']
            if current_block != last_block:
                print(f"Nouveau bloc {current_block}: {metadata['txCount']} txs")
                last_block = current_block
            
            time.sleep(1)
        except FileNotFoundError:
            print("Fichier metadata.json introuvable")
            time.sleep(5)
        except Exception as e:
            print(f"Erreur: {e}")
            time.sleep(5)

if __name__ == '__main__':
    watch_blocks()
```

## Exemples Node.js

### 1. Lire le fichier en Node.js
```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');

function readMetadata() {
    const metadataPath = path.join(os.homedir(), '.ethereum', 'metadata.json');
    const data = fs.readFileSync(metadataPath, 'utf8');
    return JSON.parse(data);
}

const metadata = readMetadata();
console.log(`Block: ${metadata.blockNumber}`);
console.log(`Hash: ${metadata.blockHash}`);
console.log(`Transactions: ${metadata.txCount}`);
```

### 2. Watcher en Node.js
```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');

const metadataPath = path.join(os.homedir(), '.ethereum', 'metadata.json');

fs.watchFile(metadataPath, { interval: 1000 }, (curr, prev) => {
    if (curr.mtime !== prev.mtime) {
        const data = fs.readFileSync(metadataPath, 'utf8');
        const metadata = JSON.parse(data);
        console.log(`Nouveau bloc ${metadata.blockNumber}: ${metadata.txCount} transactions`);
    }
});

console.log('Surveillance des nouveaux blocs...');
```

## Intégration avec Docker

### Dockerfile pour un exporter Prometheus
```dockerfile
FROM alpine:latest

RUN apk add --no-cache jq bash curl

COPY ethereum_exporter.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/ethereum_exporter.sh

CMD ["/usr/local/bin/ethereum_exporter.sh"]
```

### Docker Compose
```yaml
version: '3.8'

services:
  geth:
    image: ethereum/client-go:latest
    volumes:
      - ethereum-data:/root/.ethereum
    ports:
      - "8545:8545"
      - "30303:30303"
  
  monitor:
    build: ./monitor
    volumes:
      - ethereum-data:/root/.ethereum:ro
    depends_on:
      - geth

volumes:
  ethereum-data:
```

## Utilisation avec cron

### Vérification périodique de la synchronisation
```bash
# Ajouter dans crontab -e
*/5 * * * * /path/to/check_sync.sh >> /var/log/ethereum_sync.log 2>&1
```

### Sauvegarde périodique des métriques
```bash
# Sauvegarder les métriques toutes les heures
0 * * * * cat ~/.ethereum/metadata.json >> ~/.ethereum/block_history_$(date +\%Y\%m\%d).log
```

## API REST simple

### Flask API en Python
```python
from flask import Flask, jsonify
import json
from pathlib import Path

app = Flask(__name__)

@app.route('/api/block/current')
def get_current_block():
    metadata_path = Path.home() / '.ethereum' / 'metadata.json'
    with open(metadata_path, 'r') as f:
        return jsonify(json.load(f))

@app.route('/api/block/height')
def get_block_height():
    metadata_path = Path.home() / '.ethereum' / 'metadata.json'
    with open(metadata_path, 'r') as f:
        metadata = json.load(f)
        return jsonify({'height': metadata['blockNumber']})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## Conseils d'utilisation

1. **Performance**: Le fichier metadata.json est très léger, la lecture est quasi instantanée
2. **Fréquence**: Vous pouvez le lire aussi souvent que nécessaire sans impact sur les performances
3. **Atomicité**: L'écriture du fichier est atomique, vous ne lirez jamais un fichier corrompu
4. **Erreurs**: Si le fichier n'existe pas, vérifiez que le nœud est démarré et que le dataDir est configuré
5. **Permissions**: Le fichier est créé avec les permissions 0644 (lecture pour tous)




