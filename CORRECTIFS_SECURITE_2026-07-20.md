# Correctifs de sécurité du 20 juillet 2026

Ce lot applique les corrections directement réalisables identifiées pendant
l’audit du kit Flowio Waveshare Qwiic 2.1.0.

## Corrections appliquées

- `ESPAsyncWebServer` mis à jour de `3.6.0` vers `3.11.2`.
- `AsyncTCP` mis à jour de `3.3.2` vers `3.4.10`, version requise par
  `ESPAsyncWebServer 3.11.2`.
- `ArduinoJson` mis à jour de `6.21.5` vers `6.21.6`.
- Version Nextion de compilation alignée sur l’artefact livré `2.0.7`.
- Jeton CSRF aléatoire de 128 bits généré à chaque démarrage.
- Jeton annoncé dans `/api/web/meta` et exigé dans `X-Flow-CSRF` pour `POST`,
  `PUT`, `PATCH` et `DELETE`.
- Rejet des requêtes mutantes portant une origine intersite.
- Origine WebSocket `/wslog` obligatoirement identique au `Host` de la requête.
- Contrôle d’authentification et CSRF effectué directement dans le callback
  multipart, qui s’exécute avant la chaîne de middleware avec les versions
  récentes d’ESPAsyncWebServer.
- Ajout global des en-têtes CSP, anti-clickjacking, anti-MIME-sniffing,
  referrer, permissions et isolation des ressources.
- Sérialisation JSON de `ConfigStore` corrigée pour échapper guillemets,
  antislashs et caractères de contrôle.
- Les sorties JSON tronquées restent des objets syntaxiquement valides.
- Les valeurs flottantes non finies sont sérialisées en `null`.
- Les trois interfaces Web ajoutent automatiquement le jeton CSRF, y compris
  aux uploads `XMLHttpRequest`.
- `FLOW_BUILD_REF=YYYYMMDD.HHMMSS` permet une reconstruction reproductible.
- Le vérificateur de livraison contrôle désormais les versions durcies et les
  invariants CSRF, WebSocket, en-têtes et upload multipart.

## Validation

- Compilation PlatformIO `Waveshare-ESP32-S3` : réussie.
- Construction SPIFFS : réussie.
- Utilisation firmware : 1 646 573 octets, soit 78,5 % de la partition.
- Vérification syntaxique des trois fichiers JavaScript : réussie.
- Test du helper client CSRF : réussi.
- `scripts/verify_release.py` : `release verification: OK`.

Artefacts :

| Fichier | Taille | SHA-256 |
|---|---:|---|
| `binary/esp32s3-2.1.0.bin` | 1 646 944 | `f64f3eb3df9bcd12049c8502d44144ad2b2beae8b6d68faa15c269cebf4eeb78` |
| `binary/esp32s3-spiffs-2.1.0.bin` | 4 063 232 | `f97b13382959121956d19ed072bfab34a95d85c9edd155fe2729d78dbfa1c99f` |
| `binary/Flowio_Nextion_800x480-v2.0.7.tft` | 1 103 628 | `6e58232660c2bf911c84ee4065d40fbe8b30cbe230d4216f6e9111fad3042b08` |

## Compatibilité API

Les lectures HTTP restent inchangées. Un client qui effectue une action doit :

1. lire `/api/web/meta` avec l’authentification Digest ;
2. récupérer `csrf_token` ;
3. envoyer sa valeur dans `X-Flow-CSRF`.

Les clients WebSocket non navigateur doivent fournir un en-tête `Origin`
correspondant exactement à `http://<Host>` ou `https://<Host>`.

## Risques restant hors de ce lot

- HMI UDP utilise toujours un jeton dérivé par CRC16. Sa refonte nécessite une
  mise à jour coordonnée du client écran.
- L’administration reste en HTTP sans confidentialité de transport.
- Secure Boot v2, chiffrement flash/NVS, anti-rollback et signatures demandent
  une stratégie industrielle de clés et d’eFuses.
- Le socle Arduino-ESP32 2.0.17 reste à migrer vers une branche maintenue.

Ces risques imposent encore un VLAN d’administration de confiance et
l’interdiction d’exposer directement le port 80 à Internet.
