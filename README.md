# AssistIA

Procédure pour relancer le projet quand on se remet à travailler dessus.

```
Projet S9/
├── assistia_repas/   → le serveur (Dart) : MCP pour les LLM + API pour l'app
└── assistia_app/     → l'application (Flutter)
```

Il y a jusqu'à **3 fenêtres de terminal** à ouvrir, une par élément. Chacune doit **rester ouverte** tant qu'on travaille. Pour arrêter un élément : **Ctrl+C** dans sa fenêtre.

| Fenêtre | Rôle | Quand ? |
|---|---|---|
| 1. Serveur | Données, tools MCP, chat | Toujours |
| 2. App | L'application Flutter | Pour tester l'app |
| 3. Tunnel | Adresse HTTPS publique | Seulement pour le **téléphone** ou **Le Chat** |

> Les commandes ci-dessous sont pour l'**Invite de commandes (cmd)**, dont la ligne commence par `C:\…>`.
> Dans **PowerShell** (`PS C:\…>`), seules les lignes `set` changent : voir les encadrés « PowerShell ».

---

## 1. Lancer le serveur (fenêtre 1)

```cmd
cd "C:\Users\matti\Desktop\S9\Projet S9\assistia_repas"
dart run bin/server.dart --http --port 3000 --token UnSecretBienLong123 --llm gemini --llm-model gemini-3.5-flash-lite --llm-key MA_CLE_GEMINI
```

Remplacer `MA_CLE_GEMINI` par la clé créée sur [aistudio.google.com](https://aistudio.google.com) (« Get API key »).
⚠️ Ne jamais écrire la clé dans un fichier du projet, ni la montrer dans une capture d'écran.

**C'est bon si on voit :**
```
[assistia-repas] Données chargées depuis … : XX repas, …
[assistia-repas] Chat activé : gemini-3.5-flash-lite via https://generativelanguage.googleapis.com/…
[assistia-repas] Serveur prêt : MCP sur http://localhost:3000/mcp, API app sur http://localhost:3000/api (authentification : Bearer token)
```

### Choisir un autre LLM pour le chat

Remplacer la fin de la commande (`--llm gemini …`) par :

| LLM | Fin de commande | Coût / remarque |
|---|---|---|
| Gemini | `--llm gemini --llm-key CLE` | Gratuit, parfois surchargé (erreur 503) |
| Gemini Flash (plus fort) | `--llm gemini --llm-model gemini-3.8-flash --llm-key CLE` | Gratuit, quota plus petit |
| Ollama (local) | `--llm ollama` | Gratuit, hors ligne, données restent sur le PC. Installer [ollama.com](https://ollama.com) puis `ollama pull qwen2.5:7b` une fois |
| Mistral | `--llm mistral --llm-key CLE` | Offre gratuite « Experiment » sur console.mistral.ai |
| OpenAI (ChatGPT) | `--llm openai --llm-key CLE` | Clé sur platform.openai.com (payant ou tokens gratuits avec partage de données) |
| Pas de chat | *(rien)* | Le reste fonctionne, l'écran Assistant affiche une erreur |

Les données passent par le LLM choisi : n'utiliser que des **données fictives** avec les offres gratuites.

---

## 2. Lancer l'app (fenêtre 2)

```cmd
cd "C:\Users\matti\Desktop\S9\Projet S9\assistia_app"
flutter run -d chrome
```

- **Dans Chrome** : rapide pour développer (touche `r` dans le terminal = recharger le code). Les **notifications ne marchent pas** dans Chrome.
- **Sur le téléphone Android** (branché en USB, débogage USB activé) : `flutter run` puis choisir le téléphone. Tout fonctionne, notifications comprises.
- Afficher l'écran du téléphone sur le PC (pour une démo) : `scrcpy` (installation : `winget install Genymobile.scrcpy`).

### Réglages dans l'app (icône ⚙️)

| Où tourne l'app | Adresse du serveur | Token |
|---|---|---|
| Chrome sur le PC | `http://localhost:3000` | `UnSecretBienLong123` |
| Téléphone | `https://xxxx.trycloudflare.com` (adresse du tunnel, **sans** `/mcp`) | `UnSecretBienLong123` |

Puis **« Enregistrer et tester »** → doit afficher **« Connecté au serveur ✓ »**.

---

## 3. Lancer le tunnel (fenêtre 3) — seulement pour le téléphone ou Le Chat

```cmd
cloudflared tunnel --url http://localhost:3000 --http-host-header localhost:3000
```

- Copier l'adresse affichée dans le cadre : `https://quelque-chose.trycloudflare.com`.
- ⚠️ **L'adresse change à chaque lancement du tunnel** : la remettre dans l'app (Réglages) et/ou dans Le Chat. Relancer le serveur, lui, ne change pas l'adresse.
- `--http-host-header localhost:3000` est **indispensable** (sinon erreur 403 côté MCP).

### Brancher Le Chat (Mistral) sur le serveur MCP

Le Chat → **Connecteurs** → modifier le connecteur `assistiarepas` (ou en créer un : « Connecteur MCP personnalisé ») :
- URL : `https://quelque-chose.trycloudflare.com/mcp` (**avec** `/mcp`)
- Authentification : token API → `UnSecretBienLong123`
- Ne rien mettre dans les champs description/instructions (le LLM les lit).

---

## Démarrage express (Chrome, sans téléphone)

```cmd
:: Fenêtre 1
cd "C:\Users\matti\Desktop\S9\Projet S9\assistia_repas"
dart run bin/server.dart --http --port 3000 --token UnSecretBienLong123 --llm gemini --llm-model gemini-3.5-flash-lite --llm-key MA_CLE_GEMINI

:: Fenêtre 2
cd "C:\Users\matti\Desktop\S9\Projet S9\assistia_app"
flutter run -d chrome
```

---

## En cas de problème

| Symptôme | Cause | Solution |
|---|---|---|
| `flutter` n'est pas reconnu | Flutter absent du PATH | cmd : `set PATH=%USERPROFILE%\flutter\bin;%PATH%` — PowerShell : `$env:Path = "$env:USERPROFILE\flutter\bin;" + $env:Path` — définitif : ajouter `C:\Users\matti\flutter\bin` au Path dans « Variables d'environnement » |
| `Une seule utilisation de chaque adresse de socket … port = 3000` | Un ancien serveur tourne encore | Ctrl+C dans son terminal, ou PowerShell : `Get-NetTCPConnection -LocalPort 3000 \| ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }` |
| App : « Failed to fetch » | Mauvaise adresse ou serveur arrêté | Vérifier la fenêtre 1 ; dans Chrome utiliser `http://localhost:3000` |
| App : « Token refusé » / journal `← 401` | Token différent entre app et serveur | Même token des deux côtés, sans espace |
| Journal `← 403` sur `/mcp` | Tunnel lancé sans l'option Host | Relancer cloudflared avec `--http-host-header localhost:3000` |
| Le Chat : « Nécessite une attention » | Ancienne adresse de tunnel | Mettre la nouvelle adresse + `/mcp` dans le connecteur |
| Chat : `LLM : 503 … high demand` | Gemini surchargé (le serveur a déjà réessayé 2 fois) | Attendre, ou `--llm-model gemini-3.5-flash-lite`, ou `--llm ollama` |
| Chat : `Quota du LLM atteint` (429) | Quota gratuit du jour épuisé | Changer de modèle ou de LLM, ou attendre le lendemain |
| `Chat désactivé` au démarrage | Pas d'option `--llm` | Ajouter `--llm …` (voir tableau) |
| `Clé manquante pour gemini` | Clé non transmise | Ajouter `--llm-key CLE` à la commande |
| `$env:…` → « syntaxe incorrecte » | Commande PowerShell tapée dans cmd | Utiliser `set NOM=valeur` (cmd) ou passer la clé avec `--llm-key` |

### PowerShell

Dans PowerShell, les commandes `cd`, `dart`, `flutter` et `cloudflared` sont identiques. Seule la définition d'une variable change :

```powershell
$env:GEMINI_API_KEY = "MA_CLE_GEMINI"     # au lieu de : set GEMINI_API_KEY=MA_CLE_GEMINI
```

---

## Vérifications rapides

- Serveur vivant : ouvrir `http://localhost:3000/api/health` dans le navigateur → le journal affiche `← 401` (normal sans token) : la requête arrive bien.
- Tester les tools MCP sans LLM : `npx @modelcontextprotocol/inspector` → Streamable HTTP, URL `http://localhost:3000/mcp`, en-tête `Authorization: Bearer UnSecretBienLong123` → Tools → List Tools (9 tools) et Resources (`rules://alimentation`).
- Code sans erreur : `dart analyze` (dans `assistia_repas`) et `flutter analyze` (dans `assistia_app`).

Plus de détails : `assistia_repas/README.md` (serveur, tools, API) et `assistia_app/README.md` (app, Open Food Facts, Android/iOS).

## Structure

```
assistia_repas/
├── bin/server.dart            # serveur MCP (tools + ressource) et routage HTTP
├── lib/meal_repository.dart   # données : repas, garde-manger, lieux, cache produits
├── lib/product.dart           # fiche produit filtrée (instantané Open Food Facts)
├── lib/open_food_facts.dart   # client API Open Food Facts
├── lib/meal_service.dart      # validation + enregistrement d'un repas (MCP et REST)
├── lib/meal_analysis.dart     # bilan du jour, habitudes, hypothèse, texte des règles
├── lib/meal_rules.dart        # règles de régularité (check_meal_regularity)
├── lib/rest_api.dart          # routes /api pour l'app
└── data/meals.json            # données (anciens fichiers compatibles)
```

## Tools MCP

| Tool | Rôle |
|---|---|
| `get_product(barcode)` | Fiche Open Food Facts filtrée (cache local d'abord) |
| `get_meals(start_date, end_date)` | Repas avec aliments, quantités, lieu, niveau de confiance |
| `get_daily_summary(date)` | Bilan calculé : kcal (avec indicateur « fiable »), % NOVA 4, repas sautés |
| `get_pantry()` | Garde-manger virtuel |
| `get_eating_habits(days?, until?)` | Heures habituelles, repas les plus fréquents (« comme d'habitude ») |
| `check_meal_regularity(start_date, end_date)` | Les 4 règles de bien-être |
| `get_meal_plan(start_date, end_date)` | Menus planifiés |
| `log_meal(...)` | Enregistrer un repas dit en langage naturel |
| `delete_meal(id)` | Supprimer un repas enregistré par erreur (après accord de l'utilisateur) |
| `propose_meal_hypothesis(type, latitude?, longitude?)` | Question à poser en notification (« Déjeuner au RU ? ») |

**Ressource MCP** `rules://alimentation` : créneaux, règles, niveaux de confiance et ton à adopter. Le LLM la lit comme contexte (ce n'est pas une fonction).

Principe : **le serveur calcule, le LLM explique.**

## Lancer

```bash
dart pub get
dart run bin/server.dart --http --port 3000 --token UnSecretBienLong123
cloudflared tunnel --url http://localhost:3000 --http-host-header localhost:3000
```

- `--off-staging` : utilise l'environnement de test d'Open Food Facts.
- `--http-host-header localhost:3000` est nécessaire : `mcp_dart` refuse par défaut (403) les requêtes dont l'en-tête Host n'est pas local (protection anti-DNS-rebinding).
- Le Chat : connecteur MCP personnalisé → `https://….trycloudflare.com/mcp`, authentification par token.
- App : Réglages → `https://….trycloudflare.com` (sans `/mcp`) + même token.

## API REST (app)

`GET /api/health` · `GET /api/meals?start&end` · `POST /api/meals` · `GET /api/summary?date` · `GET|POST /api/pantry` · `DELETE /api/pantry/<id>` · `GET|POST /api/places` · `GET /api/hypothesis?type&lat&lon` · `GET /api/product/<code>` · `GET /api/rules`

Toutes protégées par `Authorization: Bearer <token>`.

## Chat intégré à l'app (POST /api/chat)

Le serveur joue le rôle d'**hôte MCP** : l'app envoie la conversation, le serveur appelle le LLM, exécute les tools demandés via un client MCP (relié au serveur MCP en mémoire, `lib/in_memory_transport.dart`), puis renvoie la réponse et la liste des tools utilisés (`lib/chat_agent.dart`).


