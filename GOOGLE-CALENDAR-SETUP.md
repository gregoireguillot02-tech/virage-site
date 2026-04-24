# Configuration Google Calendar pour le dashboard VIRAGE

Guide stepbystep pour connecter le widget "Ma journee" du dashboard a Google Calendar. Temps estime : 10 min.

**Compte a utiliser** : `virage.team.app@gmail.com` (compte Google VIRAGE partage).

---

## Etape 1 : Creer un projet Google Cloud

1. Connecte-toi sur `virage.team.app@gmail.com`
2. Va sur https://console.cloud.google.com
3. En haut a gauche, clique sur le selecteur de projet > **"Nouveau projet"**
4. Nom du projet : `VIRAGE Dashboard`
5. Organisation : laisse vide (ou "Aucune organisation")
6. Clique **"Creer"**
7. Attends ~10s que le projet soit cree, puis selectionne-le dans le selecteur

---

## Etape 2 : Activer Google Calendar API

1. Menu hamburger > **"APIs et services"** > **"Bibliotheque"**
2. Recherche : `Google Calendar API`
3. Clique dessus, puis **"Activer"**
4. Attends ~5s

**Bonus a activer aussi** (meme methode, pour pouvoir etendre plus tard) :
- `Gmail API` (rediger des mails depuis les agents IA)
- `Google Drive API` (lire des docs du Drive virage)
- `Google Sheets API` (compta, suivi demarchage)

Tu peux activer tout ca d'un coup, ca ne coute rien. Les scopes seront demandes individuellement au moment de la connexion OAuth.

---

## Etape 3 : Configurer l'ecran de consentement OAuth

1. **"APIs et services"** > **"Ecran de consentement OAuth"**
2. User type : **External** (public externe). Clique **"Creer"**
3. Remplis :
   - Nom de l'application : `VIRAGE Dashboard`
   - Email d'assistance utilisateur : `virage.team.app@gmail.com`
   - Logo : optionnel (tu peux uploader `02-Identite-Marque/Brand-Kit/app-icon/...`)
   - Domaine autorise : ajoute `virage.one`
   - Lien page d'accueil : `https://virage.one`
   - Lien politique de confidentialite : `https://virage.one/confidentialite.html`
   - Email developpeur : `virage.team.app@gmail.com`

4. **"Enregistrer et continuer"**

5. **Etape Scopes** : clique simplement **"Enregistrer et continuer"** sans rien ajouter. Les scopes sont demandes au runtime par le dashboard (pas besoin de les pre-declarer ici).

6. **Etape Utilisateurs de test** :
   - Ajoute `virage.team.app@gmail.com`
   - Ajoute `gregoire.guillot02@gmail.com`
   - Ajoute l'email Google de Paul
   - **"Enregistrer et continuer"**

7. **Resume** : verifie, puis **"Retour au tableau de bord"**

**Important** : tant que l'app est en mode "Test", seuls les test users peuvent se connecter. C'est suffisant pour Greg + Paul. Pour passer en production (user externe quelconque), il faudra une revue Google, pas necessaire ici.

---

## Etape 4 : Creer le Client ID OAuth

1. **"APIs et services"** > **"Identifiants"**
2. **"Creer des identifiants"** > **"ID client OAuth"**
3. Type d'application : **Application Web**
4. Nom : `VIRAGE Dashboard Web Client`
5. **Origines JavaScript autorisees** (clique "Ajouter un URI" pour chaque) :
   - `https://virage.one`
   - `http://localhost:8000` (pour tests locaux si besoin)
   - `http://localhost:5500` (pour Live Server VS Code)
6. **URI de redirection autorises** : **LAISSE VIDE** (on utilise le flow implicit token, pas de redirect)
7. **"Creer"**

8. Une fenetre popup affiche ton **ID client** du format :
   ```
   123456789012-abcdefghijklmnop.apps.googleusercontent.com
   ```
   **Copie-le entierement**. Tu en as besoin a l'etape 5.
   (Le "Secret client" n'est PAS utilise en flow implicit. Tu peux l'ignorer.)

---

## Etape 5 : Coller le Client ID dans le dashboard

1. Ouvre le fichier `04-Marketing-Com/Site-Vitrine/dashboard.html`
2. Recherche (Cmd+F) : `GOOGLE_CLIENT_ID`
3. Trouve cette ligne (vers la ligne 1232) :
   ```javascript
   const GOOGLE_CLIENT_ID='';
   ```
4. Remplace par :
   ```javascript
   const GOOGLE_CLIENT_ID='123456789012-abcdefghijklmnop.apps.googleusercontent.com';
   ```
   (avec ton vrai Client ID)
5. Sauvegarde

---

## Etape 6 : Deployer

```bash
cd "/Users/gregoire/Library/CloudStorage/GoogleDrive-virage.team.app@gmail.com/Mon Drive/projet-virage/04-Marketing-Com/Site-Vitrine"
git add dashboard.html GOOGLE-CALENDAR-SETUP.md
git commit -m "Widget Ma journee : Google Calendar integration"
git push origin main
```

GitHub Pages deploie automatiquement (~1 min).

---

## Etape 7 : Tester

1. Va sur https://virage.one/dashboard.html (Cmd+Shift+R pour vider le cache)
2. Connecte-toi au dashboard (mot de passe habituel)
3. Sur la page "Pilote 360", en haut, tu dois voir le widget "Ma journee" vert
4. Clique **"Connecter Google Calendar"**
5. Popup Google : choisis `virage.team.app@gmail.com`
6. Avertissement "Cette application n'a pas ete verifiee par Google" (normal en mode test)
   - Clique **"Parametres avances"** > **"Acceder a VIRAGE Dashboard (non securise)"**
   - C'est ton app, c'est normal
7. Autorise `Voir les evenements de votre agenda`
8. Le widget affiche tes evenements aujourd'hui + demain

---

## Quel agenda est lu ?

Le widget lit l'agenda **principal** du compte connecte (`virage.team.app@gmail.com`).

**Pour que Greg et Paul voient leurs RDV VIRAGE** : trois options.

### Option A : tout le monde met ses RDV VIRAGE dans le calendar team
- Greg cree un event "RDV Dr Talbot" directement dans `virage.team.app@gmail.com`
- Paul fait pareil
- Le dashboard affiche tout
- **Avantage** : simple, un seul agenda source de verite
- **Inconvenient** : Greg doit jongler entre 2 agendas (perso + team)

### Option B : partage d'agenda (recommande)
- Greg partage son agenda perso `gregoire.guillot02@gmail.com` avec `virage.team.app@gmail.com` (lecture seule ou "voir toutes les infos")
- Paul fait pareil avec son agenda perso
- Dans Calendar `virage.team.app@gmail.com`, tu verras les agendas de Greg + Paul en overlay
- **Pour que le widget les affiche aussi** : il faudrait lister les calendriers via `calendarList.list` et fetch chaque agenda
- **Avantage** : chacun garde son agenda perso, visibilite croisee
- **Inconvenient** : demande un peu plus de code dans le dashboard (TODO V2)

### Option C : un calendrier partage dedie
- Dans Calendar, cree un nouvel agenda "VIRAGE - RDV pros"
- Partage-le avec Greg et Paul (ecriture)
- Tout le monde ajoute les RDV dans cet agenda
- Le widget lit cet agenda specifique (il faut preciser son ID dans le code au lieu de "primary")

**Pour la V1, on part sur Option A** : tu mets les RDV VIRAGE directement dans l'agenda du compte team. Simple et immediate. On ira vers B ou C plus tard.

---

## Problemes courants

**"L'application demande un acces non verifie"** : normal en mode test. Click "Parametres avances" > continuer.

**"403 Forbidden"** : ton compte n'est pas dans la liste des test users. Ajoute-le dans "Ecran de consentement OAuth" > "Utilisateurs de test".

**"redirect_uri_mismatch"** : ton origine (https://virage.one) n'est pas dans la liste des "Origines JavaScript autorisees". Retourne dans Identifiants > ton Client ID > ajoute l'origine.

**Widget n'apparait pas du tout** : la variable `GOOGLE_CLIENT_ID` est toujours vide. Verifie dashboard.html.

**"Google Identity Services pas encore charge"** : le script https://accounts.google.com/gsi/client met 1-2s a charger. Attends puis reessaie.

**Token expire apres 1h** : normal. Re-clique "Connecter Google Calendar" pour obtenir un nouveau token. En V2 on ajoutera un refresh automatique (necessite backend).

---

## Securite

- Le Client ID n'est PAS un secret : il est public par design, visible dans le code JS
- Ce qui protege ton agenda : **les origines JavaScript autorisees** (seuls https://virage.one et localhost peuvent utiliser ce Client ID)
- Le token OAuth est stocke en `sessionStorage` : il disparait a la fermeture de l'onglet
- Tu peux **revoquer l'acces** a tout moment :
  - Dans le dashboard : bouton "X" dans les actions du widget
  - Chez Google : https://myaccount.google.com/permissions > VIRAGE Dashboard > Supprimer l'acces

---

## V2 (apres validation)

- Lister plusieurs calendriers (agenda perso Greg + Paul + team)
- Widget Gmail (5 derniers mails non lus importants)
- Widget Drive (derniers docs VIRAGE modifies)
- Widget Sheets (KPI synchro depuis Google Sheets compta)
- Refresh token cote Supabase Edge Function pour eviter la reconnexion apres 1h
