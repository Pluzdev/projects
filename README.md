# 🚀 SaaS API - Plateforme de Notifications E-commerce

## 📋 Description du Projet

**SaaS API** est une plateforme SaaS complète qui permet aux propriétaires de boutiques Shopify de recevoir des notifications en temps réel sur leurs ventes via Discord et Telegram. L'application offre également des rapports en direct avec des métriques détaillées et un système de webhooks intelligent pour automatiser les notifications.

### 🎯 **Objectif Principal**
Simplifier la gestion des notifications e-commerce en offrant une solution centralisée qui :
- **Automatise** les notifications de nouvelles commandes
- **Centralise** les données de vente de plusieurs boutiques
- **Fournit** des rapports en temps réel
- **Intègre** facilement Discord et Telegram

### 💡 **Valeur Ajoutée**
- **Temps réel** : Notifications instantanées dès qu'une commande est passée
- **Multi-boutiques** : Gestion centralisée de plusieurs boutiques Shopify
- **Multi-canaux** : Notifications via Discord ET Telegram simultanément
- **Rapports intelligents** : Métriques automatiques avec mise à jour en direct
- **Sécurité** : Tokens uniques, authentification JWT, masquage des données sensibles

---

## 🏗️ Architecture Technique

### **Stack Technologique**
- **Backend** : Node.js + TypeScript + Express.js
- **Base de données** : PostgreSQL avec pool de connexions
- **Authentification** : JWT avec bcrypt pour le hachage des mots de passe
- **Validation** : Zod pour la validation des schémas et types
- **Sécurité** : Helmet, CORS, middleware d'authentification
- **Intégrations** : Discord Webhooks, Telegram Bot API

### **Structure du Projet**
```
src/
├── config/          # Configuration environnement
│   └── env.ts      # Variables d'environnement avec validation
├── lib/            # Utilitaires et logique métier
│   ├── auth.ts     # Authentification et JWT
│   ├── db.ts       # Connexion PostgreSQL
│   ├── middleware.ts # Middleware d'authentification
│   ├── providers.ts # Système de providers (Discord/Telegram)
│   ├── discord.ts  # Intégration Discord
│   ├── telegram.ts # Intégration Telegram
│   ├── scheduler.ts # Scheduler pour rapports en direct
│   ├── stats.ts    # Calculs de statistiques
│   └── live.ts     # Formatage des rapports en direct
├── routes/         # Endpoints API
│   ├── auth.ts     # Authentification (login/register)
│   ├── shops.ts    # Gestion des boutiques Shopify
│   ├── integrations.ts # Intégrations Discord/Telegram
│   ├── webhooks.ts # Webhooks Shopify
│   └── reports.ts  # Rapports en direct
├── types/          # Types TypeScript
└── utils/          # Helpers généraux
```

---

## 🔐 Système d'Authentification

### **Fonctionnalités de Sécurité**
- **Inscription sécurisée** : Email, mot de passe (min 8 caractères), nom d'utilisateur optionnel
- **Connexion JWT** : Tokens avec expiration configurable (7 jours par défaut)
- **Hachage des mots de passe** : bcrypt avec 10 rounds de salt
- **Middleware de protection** : Toutes les routes sensibles protégées
- **Validation stricte** : Tous les inputs validés avec Zod

### **Endpoints d'Authentification**
```http
POST /api/auth/register
Content-Type: application/json
{
  "email": "user@example.com",
  "password": "securepassword123",
  "username": "optional_username",
  "display_name": "Nom d'affichage",
  "avatar_url": "https://example.com/avatar.jpg"
}

POST /api/auth/login
Content-Type: application/json
{
  "email": "user@example.com",
  "password": "securepassword123"
}

GET /api/me
Authorization: Bearer <jwt_token>
```

---

## 🏪 Gestion des Boutiques Shopify

### **Fonctionnalités Avancées**
- **CRUD complet** : Création, lecture, mise à jour, suppression des boutiques
- **Validation stricte** : Domaines Shopify uniquement (format `xxx.myshopify.com`)
- **Métriques intégrées** : Compteurs de commandes et clients en temps réel
- **Sécurité par utilisateur** : Chaque utilisateur ne voit que ses propres boutiques
- **Statut actif/inactif** : Gestion de l'état des boutiques

### **Endpoints de Gestion des Boutiques**
```http
# Lister toutes les boutiques de l'utilisateur
GET /api/shops
Authorization: Bearer <jwt_token>

# Détails d'une boutique spécifique
GET /api/shops/:id
Authorization: Bearer <jwt_token>

# Créer une nouvelle boutique
POST /api/shops
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "name": "Ma Boutique",
  "shop_domain": "ma-boutique.myshopify.com",
  "active": true
}

# Modifier une boutique
PATCH /api/shops/:id
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "name": "Nouveau nom",
  "active": false
}

# Supprimer une boutique
DELETE /api/shops/:id
Authorization: Bearer <jwt_token>

# Métriques détaillées d'une boutique
GET /api/shops/:id/metrics
Authorization: Bearer <jwt_token>
```

### **Métriques Disponibles**
- **Total des commandes** : Nombre total de commandes
- **Revenus totaux** : Montant total par devise
- **Période d'activité** : Première et dernière commande
- **Top pays** : Répartition géographique des clients
- **Données client** : Nom, email, téléphone, adresse

---

## 🔗 Système d'Intégrations Multi-Canaux

### **Providers Supportés**

#### **Discord Integration**
- **Webhooks Discord** : Notifications via webhooks personnalisés
- **Validation d'URL** : Vérification que l'URL est bien un webhook Discord
- **Messages formatés** : Embeds Discord avec couleurs et structure
- **Masquage sécurisé** : URLs webhook masquées dans les réponses

#### **Telegram Integration**
- **Bot Telegram** : Notifications via bot personnalisé
- **Chat ID** : Support des groupes et conversations privées
- **Messages HTML** : Formatage riche avec emojis et structure
- **Masquage des tokens** : Tokens bot masqués pour la sécurité

### **Architecture des Providers**
```typescript
type Provider<Conf> = {
  key: ProviderKey;
  schema: z.ZodSchema<Conf>;           // Validation des configs
  redact: (c: Conf) => any;           // Masquage des données sensibles
  send: (c: Conf, message: string) => Promise<boolean>;
};
```

### **Endpoints d'Intégrations**
```http
# Lister toutes les intégrations
GET /api/integrations
Authorization: Bearer <jwt_token>

# Créer une intégration générique
POST /api/integrations
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "channel": "discord",
  "config": {
    "webhook_url": "https://discord.com/api/webhooks/..."
  },
  "active": true
}

# Créer une intégration Discord (endpoint dédié)
POST /api/integrations/discord
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "webhook_url": "https://discord.com/api/webhooks/...",
  "active": true
}

# Créer une intégration Telegram (endpoint dédié)
POST /api/integrations/telegram
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "bot_token": "1234567890:ABC...",
  "chat_id": "-1001234567890",
  "active": true
}

# Tester toutes les intégrations
POST /api/integrations/test
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "message": "Message de test",
  "channels": ["discord", "telegram"]
}

# Modifier une intégration
PATCH /api/integrations/:id
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "active": false,
  "config": { /* nouvelle config */ }
}

# Supprimer une intégration
DELETE /api/integrations/:id
Authorization: Bearer <jwt_token>
```

---

## 📡 Système de Webhooks Shopify

### **Fonctionnalités Avancées**
- **URL sécurisée** : Token unique par utilisateur pour la sécurité
- **Déduplication intelligente** : Évite les notifications en double
- **Parsing automatique** : Extraction automatique des données de commande
- **Notifications formatées** : Messages Discord/Telegram structurés et beaux
- **Gestion des erreurs** : Retry automatique et gestion des échecs

### **Données Extraites des Commandes**
- **Informations de commande** : ID, nom, numéro, prix total
- **Détails client** : Nom, email, téléphone, pays, adresse
- **Produits** : Noms des produits avec quantités
- **Métadonnées** : Devise, date de création, domaine de la boutique

### **Endpoints de Webhooks**
```http
# Webhook principal Shopify (appelé automatiquement par Shopify)
POST /api/webhooks/u/:token/shopify/orders-create
Content-Type: application/json
X-Shopify-Shop-Domain: ma-boutique.myshopify.com
{
  "id": 1234567890,
  "name": "#1001",
  "total_price": "99.99",
  "currency": "EUR",
  "customer": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com"
  },
  "line_items": [
    {
      "name": "Produit 1",
      "quantity": 2
    }
  ]
}

# Obtenir l'URL webhook de l'utilisateur
GET /api/webhooks/my-url
Authorization: Bearer <jwt_token>

# Rotation du token webhook (sécurité)
POST /api/webhooks/rotate
Authorization: Bearer <jwt_token>
```

### **Exemple de Notification Discord**
```json
{
  "embeds": [
    {
      "title": "New Order Created:",
      "color": 0x22c55e,
      "fields": [
        {
          "name": "🛒 Order ID",
          "value": "```#1001```",
          "inline": true
        },
        {
          "name": "🏬 Shop Domain",
          "value": "```ma-boutique.myshopify.com```",
          "inline": true
        },
        {
          "name": "💰 Total Price",
          "value": "```99.99 EUR```",
          "inline": true
        }
      ]
    }
  ]
}
```

---

## 📊 Rapports en Direct (Live Reports)

### **Fonctionnalités Révolutionnaires**
- **Mise à jour automatique** : Scheduler intelligent toutes les 30 secondes
- **Intervalles configurables** : 1, 5, 10, 15, 30, 60 minutes
- **Support multi-fuseaux** : FR, AE, US + tous les fuseaux IANA
- **Messages épinglés** : Épingle automatique sur Telegram pour visibilité
- **Formatage intelligent** : Messages structurés avec emojis et couleurs
- **Gestion d'erreurs** : Retry automatique en cas d'échec

### **Scheduler Intelligent**
```typescript
// Exécution toutes les 30 secondes
setInterval(async () => {
  // Récupération des rapports à exécuter
  // Lock distribué pour éviter les doublons
  // Mise à jour des messages existants
  // Gestion des erreurs et retry
}, 30_000);
```

### **Endpoints de Rapports**
```http
# Lister les rapports en direct
GET /api/reports/live
Authorization: Bearer <jwt_token>

# Créer un rapport en direct
POST /api/reports/live
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "tz": "Europe/Paris",
  "interval_minutes": 15,
  "bot_token": "1234567890:ABC...",
  "chat_id": "-1001234567890",
  "setup_now": true
}

# Créer et configurer immédiatement
POST /api/reports/live/setup
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "tz": "FR",
  "interval_minutes": "30",
  "bot_token": "1234567890:ABC...",
  "chat_id": "-1001234567890"
}

# Modifier un rapport
PATCH /api/reports/live/:id
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "tz": "America/New_York",
  "interval_minutes": "60",
  "active": true
}

# Activer/Désactiver un rapport
POST /api/reports/live/:id/activate
POST /api/reports/live/:id/deactivate
Authorization: Bearer <jwt_token>

# Forcer une mise à jour immédiate
POST /api/reports/live/:id/refresh
Authorization: Bearer <jwt_token>

# Supprimer un rapport
DELETE /api/reports/live/:id
Authorization: Bearer <jwt_token>
```

### **Exemple de Rapport en Direct**
```
📊 Bilan des ventes du jour
Dernière mise à jour : 26/09/2025 17:19:11

ma-boutique.myshopify.com — 17 ventes — 3405.90€
autre-boutique.myshopify.com — 3 ventes — 150.00€

Total: 20 commandes — 3555.90€
```

---

## 🗄️ Base de Données PostgreSQL

### **Tables Principales**

#### **users**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  username VARCHAR(30) UNIQUE,
  display_name VARCHAR(60),
  avatar_url TEXT,
  role VARCHAR(20) DEFAULT 'user',
  plan_active BOOLEAN DEFAULT false,
  plan_expires_at TIMESTAMP,
  webhook_token VARCHAR(32) UNIQUE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### **shops**
```sql
CREATE TABLE shops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255),
  shop_domain VARCHAR(255) NOT NULL,
  active BOOLEAN DEFAULT true,
  orders_count INTEGER DEFAULT 0,
  customers_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, shop_domain)
);
```

#### **orders**
```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  shop_id UUID REFERENCES shops(id) ON DELETE CASCADE,
  order_id VARCHAR(255) NOT NULL,
  order_name VARCHAR(255),
  total_price DECIMAL(10,2),
  currency VARCHAR(10),
  customer_name VARCHAR(255),
  customer_email VARCHAR(255),
  customer_phone VARCHAR(50),
  customer_country VARCHAR(10),
  customer_address TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(shop_id, order_id)
);
```

#### **notifications**
```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  channel VARCHAR(20) NOT NULL, -- 'discord' | 'telegram'
  active BOOLEAN DEFAULT true,
  config JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### **live_reports**
```sql
CREATE TABLE live_reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  notification_id UUID REFERENCES notifications(id) ON DELETE CASCADE,
  tz VARCHAR(50) NOT NULL,
  interval_minutes INTEGER NOT NULL,
  active BOOLEAN DEFAULT true,
  message_id VARCHAR(50),
  last_run_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, notification_id)
);
```

### **Fonctionnalités de Base de Données**
- **Pool de connexions** : Gestion optimisée avec max 10 connexions
- **Healthcheck** : Vérification automatique de l'état de la DB
- **Requêtes typées** : Helper `query<T>()` avec support TypeScript complet
- **SSL automatique** : Configuration SSL pour la production
- **Gestion d'erreurs** : Logging automatique des erreurs de connexion

---

## 🔧 Utilitaires et Helpers

### **Système de Providers (lib/providers.ts)**
```typescript
// Architecture modulaire et extensible
export const PROVIDERS: Record<ProviderKey, Provider<any>> = {
  discord: DiscordProvider,
  telegram: TelegramProvider,
};

// Fonctions utilitaires
export function getProvider(channel: string)
export function validateConfig(channel: string, config: unknown)
export function redactConfig(channel: string, config: any)
export async function sendTest(channel: string, config: any, message: string)
```

### **Scheduler Intelligent (lib/scheduler.ts)**
```typescript
// Lock distribué avec PostgreSQL
async function tryLock(jobId: string): Promise<boolean>
async function unlock(jobId: string)

// Scheduler principal
export function startLiveReportScheduler() {
  setInterval(async () => {
    // Récupération des jobs à exécuter
    // Lock distribué pour éviter les doublons
    // Exécution en parallèle
    // Gestion des erreurs et cleanup
  }, 30_000);
}
```

### **Calculs de Statistiques (lib/stats.ts)**
```typescript
export type DailyStatsDetailed = {
  total_orders: number;
  per_currency_total: Array<{ currency: string; revenue: number }>;
  shops: ShopRow[];
  start_utc: string;
  end_utc: string;
};

export async function getDailyStatsDetailed(
  userId: string, 
  tz: string
): Promise<DailyStatsDetailed>
```

---

## 🚀 Installation et Configuration

### **Prérequis**
- Node.js 18+ 
- PostgreSQL 12+
- npm ou yarn

### **Variables d'Environnement**
```env
# Environnement
NODE_ENV=development|production|test
PORT=4000

# Base de données
DATABASE_URL=postgresql://username:password@localhost:5432/saas_api

# Sécurité
JWT_SECRET=your-super-secret-jwt-key-min-10-chars
JWT_EXPIRES_IN=7d
```

### **Installation**
```bash
# Cloner le projet
git clone <repository-url>
cd saas-api

# Installer les dépendances
npm install

# Configuration de la base de données
# Créer la base de données PostgreSQL
# Configurer les variables d'environnement

# Développement
npm run dev

# Production
npm run build
npm start
```

### **Scripts Disponibles**
```json
{
  "scripts": {
    "dev": "ts-node src/index.ts",
    "build": "tsc -p tsconfig.json", 
    "start": "node dist/index.js"
  }
}
```

---

## 🔒 Sécurité et Bonnes Pratiques

### **Mesures de Sécurité Implémentées**
- **Helmet.js** : Headers de sécurité HTTP (XSS, CSRF, etc.)
- **CORS configuré** : Origines autorisées et credentials
- **JWT sécurisé** : Tokens avec expiration et secret fort
- **Validation stricte** : Tous les inputs validés avec Zod
- **Masquage des données** : Tokens et secrets masqués dans les réponses
- **Hachage des mots de passe** : bcrypt avec salt rounds
- **Protection des routes** : Middleware d'authentification obligatoire

### **Gestion des Erreurs**
- **Logging centralisé** : Toutes les erreurs loggées avec contexte
- **Réponses sécurisées** : Pas d'exposition d'informations sensibles
- **Retry automatique** : Gestion des échecs temporaires
- **Validation des inputs** : Prévention des injections

### **Performance et Scalabilité**
- **Pool de connexions DB** : Gestion optimisée des connexions
- **Requêtes typées** : Performance TypeScript
- **Traitement asynchrone** : Non-bloquant pour les notifications
- **Lock distribué** : Évite les exécutions multiples du scheduler

---

## 📈 Fonctionnalités Avancées

### **Système de Plans Utilisateur**
- **Vérification automatique** : Statut du plan vérifié à chaque requête
- **Gestion des expirations** : Dates d'expiration des plans
- **Blocage intelligent** : Fonctionnalités désactivées si plan inactif
- **API de statut** : Endpoint pour vérifier l'état du plan

### **Métriques et Analytics Avancées**
- **Compteurs en temps réel** : Commandes et clients par boutique
- **Revenus multi-devises** : Support de toutes les devises Shopify
- **Analytics géographiques** : Top pays des clients
- **Statistiques temporelles** : Première/dernière commande, tendances
- **Agrégations intelligentes** : Calculs optimisés avec jointures

### **Notifications Intelligentes**
- **Messages Discord** : Embeds formatés avec couleurs et structure
- **Messages Telegram** : HTML riche avec emojis et formatage
- **Gestion des erreurs** : Retry automatique et fallback
- **Fire-and-forget** : Performance optimale pour les notifications
- **Déduplication** : Évite les notifications en double

---

## 🎯 Cas d'Usage et Exemples

### **Scénario 1 : Boutique E-commerce Simple**
1. **Inscription** : Création du compte utilisateur
2. **Ajout boutique** : Configuration de `ma-boutique.myshopify.com`
3. **Intégration Discord** : Configuration du webhook Discord
4. **Webhook Shopify** : Configuration dans Shopify Admin
5. **Notifications** : Réception automatique des nouvelles commandes

### **Scénario 2 : Multi-boutiques avec Rapports**
1. **Plusieurs boutiques** : Ajout de 3 boutiques Shopify
2. **Intégrations multiples** : Discord + Telegram simultanément
3. **Rapport en direct** : Mise à jour toutes les 15 minutes
4. **Métriques centralisées** : Vue d'ensemble de toutes les boutiques

### **Scénario 3 : Équipe avec Gestion Avancée**
1. **Comptes multiples** : Plusieurs utilisateurs avec leurs boutiques
2. **Notifications personnalisées** : Chaque utilisateur ses propres canaux
3. **Rapports individuels** : Métriques par utilisateur
4. **Sécurité** : Isolation complète des données

---

## 🔮 Roadmap et Extensions Possibles

### **Fonctionnalités Futures**
- **Support Slack** : Intégration Slack en plus de Discord/Telegram
- **Templates personnalisés** : Messages de notification personnalisables
- **Analytics avancées** : Graphiques et tendances détaillées
- **API publique** : API REST pour intégrations tierces
- **Webhooks sortants** : Notifications vers d'autres services
- **Multi-tenant** : Support des équipes et organisations

### **Améliorations Techniques**
- **Cache Redis** : Performance améliorée avec cache
- **Queue system** : Gestion des notifications avec Bull/Agenda
- **Monitoring** : Métriques de performance et santé
- **Tests automatisés** : Suite de tests complète
- **Documentation API** : Swagger/OpenAPI automatique

---

## 📞 Support et Contribution

### **Documentation API**
- **Endpoints complets** : Tous les endpoints documentés
- **Exemples de requêtes** : cURL et JavaScript
- **Codes d'erreur** : Gestion d'erreurs détaillée
- **Rate limiting** : Limites et bonnes pratiques

### **Développement**
- **TypeScript strict** : Code type-safe à 100%
- **ESLint/Prettier** : Code style uniforme
- **Architecture modulaire** : Code maintenable et extensible
- **Tests unitaires** : Couverture de code complète

---

## 📄 Licence

Ce projet est sous licence MIT. Voir le fichier `LICENSE` pour plus de détails.

---

**🚀 SaaS API - Simplifiez vos notifications e-commerce !**
