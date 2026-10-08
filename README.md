SENTINEL-X — Supervision et alarme sécurisées d'un data center autonome

Workshop national EPSI Bac+4 2026-27 · Consortium AetherCorp · Groupe M1-G3Cn3E

SENTINEL-X surveille un micro-data center isolé : température, détection de gaz et présence/mouvement. Les mesures remontent en MQTT chiffré (TLS) vers un serveur local. Un dashboard React les affiche en temps réel et déclenche des alertes en cas d'anomalie. Une couche cyber (segmentation, IDS/IPS, durcissement) protège l'ensemble.

Architecture

Trois zones réseau, filtrées par pfSense :

Zone	Réseau	Rôle
IoT	192.168.10.0/24	Capteurs (simulés sous Wokwi)
Serveur	192.168.152.0/24	Mosquitto, API, base de données, Nginx, Grafana
Admin	192.168.30.0/24	Poste d'administration (SSH, HTTPS)

Flux autorisé depuis l'IoT : uniquement MQTTS (8883) vers le serveur. Suricata détecte et bloque les scans (ex. nmap depuis Kali).

Capteurs (Wokwi) --MQTTS 8883--> Mosquitto --> Backend Node/Express --> Dashboard React
                                      |                 |
                                      +--> API FastAPI / PostgreSQL / Grafana (côté supervision)

À compléter : articulation précise entre le backend Express et l'API FastAPI/Grafana.

Contenu du dépôt
.
├── docker-compose.yml        # pile complète (réseau interne, conteneurs durcis)
├── .env.example              # modèle de variables (copier en .env, ne jamais committer .env)
├── infra/
│   └── mosquitto/            # mosquitto.conf, acl
├── frontend/                 # dashboard React
├── backend/                  # API Node.js / Express
├── docs/                     # dossier d'ingénierie, poster A3, schémas
└── .gitignore / .gitattributes

Adapter l'arborescence à l'état réel du dépôt.

Prérequis
Docker et Docker Compose v2
Node.js 20+ (développement du frontend et du backend)
Un serveur Linux (VM Debian ou Raspberry Pi 5) avec UFW activé
Installation et lancement
bash
git clone https://github.com/Pitou92/Ghost.git
cd Ghost

# 1. Variables d'environnement
cp .env.example .env
nano .env            # renseigner vos propres valeurs, jamais de secret dans le dépôt

# 2. Vérifier la configuration
docker compose config -q

# 3. Démarrer
docker compose up -d
docker compose ps
Certificats TLS et comptes MQTT

Ils ne sont pas versionnés (certs/, secrets/, fichier de mots de passe Mosquitto dans .gitignore). À générer localement :

Créer une CA locale, puis un certificat serveur avec SAN (IP du serveur et nom sentinel-mqtt).
Placer les fichiers dans certs/ (hors dépôt).
Créer les utilisateurs avec mosquitto_passwd, saisie interactive, sans mot de passe dans l'historique.
Backend et frontend (développement)
bash
cd backend && npm install && npm start      # API sur le port défini dans .env
cd frontend && npm install && npm start     # dashboard

À compléter : ports, routes exposées (ex. POST /api/v1/alerts), variables attendues.

Simulation des capteurs (Wokwi)

Circuit : microcontrôleur + MQ-2 (gaz) + PIR (mouvement) + DS18B20 (température) + buzzer + LED rouge/verte avec résistances.

À compléter : lien du projet Wokwi, numéros de GPIO, transport Wokwi → backend, seuils d'alerte. Note : le sujet prévoit ESP8266 + DHT22 ; le circuit actuel utilise ESP32 + DS18B20 (écart à justifier).

Alertes

Une alerte est générée si : température élevée, mouvement détecté, ou gaz détecté. Elle s'affiche sur le dashboard et active le buzzer/LED sur le circuit.

Sécurité
MQTT en TLS 1.2 uniquement sur 8883, accès anonyme désactivé, authentification par mot de passe, ACL par topic
Nginx en HTTPS (TLS 1.2/1.3) devant l'API et Grafana ; TLS 1.0/1.1 refusés (vérifié avec OpenSSL)
pfSense : segmentation et filtrage entre zones ; Suricata IDS/IPS (alerte et blocage de la source)
UFW sur le serveur, SSH par clé uniquement, Fail2ban
Conteneurs durcis : no-new-privileges, cap_drop, rotation des logs, réseau backend interne
Aucun secret dans le dépôt ; branche main protégée (pull request obligatoire)
Limites connues
Certificat auto-signé (CA locale), pas de PKI interne
mTLS non activé : TLS côté serveur uniquement
Vérifier que PostgreSQL (5432) et Grafana (3000) ne sont pas publiés sur toutes les interfaces
Intelligence artificielle
Vision par ordinateur : détection de personnes via webcam USB (YOLO), exécution locale
Maintenance prédictive : détection d'anomalies sur les séries de capteurs (Isolation Forest)
