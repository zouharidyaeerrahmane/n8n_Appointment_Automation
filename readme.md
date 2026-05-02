# 📅 Telegram Calendar Bot — Guide de Configuration

> **Bot n8n** qui transforme un message Telegram en événement Google Calendar automatiquement via une IA (Groq).

---

## 📋 Table des matières

1. [Vue d'ensemble du workflow](#vue-densemble)
2. [Prérequis](#prérequis)
3. [Étape 1 — Telegram Bot](#étape-1--configurer-le-bot-telegram)
4. [Étape 2 — Groq API (IA)](#étape-2--configurer-groq-api)
5. [Étape 3 — Google Calendar](#étape-3--configurer-google-calendar)
6. [Étape 4 — Importer dans n8n](#étape-4--importer-le-workflow-dans-n8n)
7. [Étape 5 — Connecter les credentials](#étape-5--connecter-les-credentials)
8. [Étape 6 — Tester le bot](#étape-6--tester-le-bot)
9. [Schéma du workflow](#schéma-du-workflow)
10. [Exemples de messages](#exemples-de-messages)
11. [Dépannage](#dépannage)

---

## Vue d'ensemble

```
Telegram Message → Extract Event Info (IA Groq) → Create Google Calendar Event → Send Confirmation
```

Le bot reçoit un message Telegram en langage naturel (ex: *"Réunion lundi 12 mai à 14h"*), l'IA extrait les informations, crée l'événement dans Google Calendar, puis envoie une confirmation Telegram.

---

## Prérequis

- Un compte **n8n** (cloud ou self-hosted)
- Un compte **Telegram**
- Un compte **Google** (avec Google Calendar activé)
- Un compte **Groq** (gratuit sur [console.groq.com](https://console.groq.com))

---

## Étape 1 — Configurer le Bot Telegram

### 1.1 Créer un bot avec BotFather

1. Ouvrir Telegram et chercher `@BotFather`
2. Envoyer la commande `/newbot`
3. Donner un **nom** au bot (ex: `Mon Agenda Bot`)
4. Donner un **username** (doit finir par `bot`, ex: `monagenda_bot`)
5. BotFather te donnera un **Token API** — **copie-le**, tu en auras besoin

```
Exemple de token : 7123456789:AAFxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 1.2 Créer le credential Telegram dans n8n

1. Dans n8n, aller dans **Credentials → New**
2. Chercher `Telegram`
3. Coller ton **Token API**
4. Cliquer **Save**

---

## Étape 2 — Configurer Groq API

> Groq fournit un accès rapide et gratuit aux modèles LLM (Llama, Mistral, etc.)

### 2.1 Obtenir une clé API Groq

1. Aller sur [console.groq.com](https://console.groq.com)
2. Créer un compte ou se connecter
3. Aller dans **API Keys → Create API Key**
4. Copier la clé générée

### 2.2 Créer le credential Groq dans n8n

1. Dans n8n, aller dans **Credentials → New**
2. Chercher `Groq`
3. Coller ta **clé API**
4. Cliquer **Save**

> **Note :** Le workflow utilise le modèle `openai/gpt-oss-120b` via Groq. Tu peux le remplacer par `llama3-70b-8192` ou `mixtral-8x7b-32768` si besoin.

---

## Étape 3 — Configurer Google Calendar

### 3.1 Créer un projet Google Cloud

1. Aller sur [console.cloud.google.com](https://console.cloud.google.com)
2. Créer un **nouveau projet** (ex: `n8n-calendar-bot`)
3. Dans le menu, aller dans **APIs & Services → Library**
4. Chercher **Google Calendar API** et l'activer

### 3.2 Créer des credentials OAuth2

1. Aller dans **APIs & Services → Credentials**
2. Cliquer **Create Credentials → OAuth 2.0 Client IDs**
3. Choisir **Web application**
4. Dans **Authorized redirect URIs**, ajouter :
   ```
   https://[ton-instance-n8n]/rest/oauth2-credential/callback
   ```
5. Copier le **Client ID** et le **Client Secret**

### 3.3 Créer le credential Google Calendar dans n8n

1. Dans n8n, aller dans **Credentials → New**
2. Chercher `Google Calendar OAuth2`
3. Renseigner le **Client ID** et **Client Secret**
4. Cliquer **Connect** et autoriser l'accès à Google Calendar
5. Cliquer **Save**

---

## Étape 4 — Importer le workflow dans n8n

1. Dans n8n, cliquer sur **Workflows → Import**
2. Sélectionner le fichier `Telegram_Calendar_Bot.json`
3. Le workflow apparaît avec 5 nœuds :

| Nœud | Rôle |
|------|------|
| `Telegram Trigger` | Reçoit les messages Telegram |
| `Extract Event Info` | Extrait titre, date, heure via l'IA |
| `Groq Chat Model` | Modèle de langage utilisé par l'extracteur |
| `Create Calendar Event` | Crée l'événement dans Google Calendar |
| `Send Confirmation` | Envoie un message de confirmation Telegram |

---

## Étape 5 — Connecter les credentials

Pour chaque nœud, tu dois associer les credentials créés précédemment :

### Nœud : `Telegram Trigger`
- Cliquer sur le nœud
- Dans **Credentials → Telegram API**, sélectionner ton credential Telegram
- Vérifier que `Updates` contient `message`

### Nœud : `Groq Chat Model`
- Cliquer sur le nœud
- Dans **Credentials → Groq API**, sélectionner ton credential Groq
- Le modèle par défaut est `openai/gpt-oss-120b` *(tu peux changer)*

### Nœud : `Create Calendar Event`
- Cliquer sur le nœud
- Dans **Credentials → Google Calendar OAuth2**, sélectionner ton compte Google
- Dans le champ **Calendar**, sélectionner le calendrier cible (ex: *"Mon agenda"*)

### Nœud : `Send Confirmation`
- Cliquer sur le nœud
- Dans **Credentials → Telegram API**, sélectionner le même credential Telegram

---

## Étape 6 — Tester le bot

### 6.1 Activer le workflow

1. Dans n8n, cliquer sur le toggle **Active** en haut à droite du workflow
2. Le workflow est maintenant en écoute

### 6.2 Envoyer un message test

Ouvrir Telegram, aller sur ton bot et envoyer un message comme :

```
Réunion client le 15 mai à 14h00
```

ou

```
Dentiste 20/05 à 9h30
```

### 6.3 Vérifier le résultat

- Dans **Google Calendar**, un événement d'1 heure devrait apparaître avec un rappel 24h avant
- Dans **Telegram**, tu recevras :

```
✅ Rendez-vous créé !

📌 Réunion client
📅 Date : 2026-05-15
🕐 Heure : 14:00

🔔 Tu seras notifié 1 jour avant et 1 heure avant.
```

---

## Schéma du workflow

```
[Telegram Trigger]
        │
        ▼
[Extract Event Info] ◄── [Groq Chat Model]
        │
        ▼
[Create Calendar Event]
        │
        ▼
[Send Confirmation → Telegram]
```

---

## Exemples de messages

Le bot comprend plusieurs formats de date et d'heure :

| Message envoyé | Résultat attendu |
|---------------|-----------------|
| `Réunion lundi 18 mai à 10h` | Événement le 2026-05-18 à 10:00 |
| `Dentiste 20/05 à 9h30` | Événement le 2026-05-20 à 09:30 |
| `Appel zoom demain 15h` | Événement le lendemain à 15:00 |
| `Cours de gym vendredi 7h` | Événement vendredi à 07:00 |

> **Astuce :** Plus le message est clair et précis, plus l'extraction sera fiable.

---

## Dépannage

### ❌ Le bot ne répond pas

- Vérifier que le workflow est bien **activé** (toggle vert)
- Vérifier que le **Token Telegram** est correct dans les credentials
- S'assurer que le webhook Telegram est enregistré (n8n le fait automatiquement au premier démarrage)

### ❌ L'événement n'est pas créé

- Vérifier les **credentials Google Calendar** et réautoriser si nécessaire
- Ouvrir l'exécution dans n8n (onglet **Executions**) et regarder la sortie du nœud `Extract Event Info`
- S'assurer que le format de date extrait est bien `YYYY-MM-DDTHH:MM:00`

### ❌ Erreur Groq / modèle non trouvé

- Vérifier que la clé API Groq est valide sur [console.groq.com](https://console.groq.com)
- Essayer de remplacer le modèle par `llama3-70b-8192` dans le nœud `Groq Chat Model`

### ❌ Date mal interprétée

- Le prompt est configuré pour l'année **2026** — si tu utilises ce bot après 2026, modifie la description du champ `date` dans le nœud `Extract Event Info`
- Pour les formats ambigus, précise clairement le mois : `15 juillet` plutôt que `15/07`

---

## Configuration avancée

### Changer la durée par défaut des événements

Dans le nœud `Extract Event Info`, le champ `end` est configuré pour être **1 heure après** `start`. Pour modifier cela, changer la description de l'attribut `end` :

```
end datetime in YYYY-MM-DDTHH:MM:00 format (2 hours after start)
```

### Changer le rappel

Dans le nœud `Create Calendar Event`, le reminder est réglé à `1440 minutes` (24h). Pour modifier :
- `60` = 1 heure avant
- `30` = 30 minutes avant
- `2880` = 2 jours avant

### Ajouter la gestion d'erreurs

Tu peux ajouter un nœud **IF** après `Extract Event Info` pour vérifier que tous les champs sont bien remplis avant de créer l'événement, et envoyer un message d'erreur si l'IA n'a pas pu extraire les informations.

---

*Guide généré pour le workflow `Telegram_Calendar_Bot.json` — n8n*