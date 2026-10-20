# Comparateur de versions — extension Trimble Connect (vue 3D)

Compare deux versions d'une même maquette directement dans la vue 3D de Trimble Connect for Browser.

## Ce que fait l'outil

1. Tu choisis une maquette chargée dans la vue 3D, puis la version A (ancienne) et la version B (nouvelle).
2. L'extension charge A, lit tous les objets (GUID, propriétés, emprise), charge B, fait de même, puis compare objet par objet via le GUID.
3. Sur la version B : **vert** = ajouté, **orange** = géométrie/position modifiée, **bleu** = propriétés seules modifiées, gris transparent = inchangé.
4. Bouton « Voir les supprimés » : recharge la version A et colore en **rouge** les objets disparus.
5. Clic sur un élément de la liste : zoom dessus et affichage des propriétés modifiées (avant → après).
6. Export CSV (séparateur `;`, compatible Excel FR) : une ligne par propriété modifiée.

## Installation (≈ 10 min)

1. **Héberger les fichiers** sur une URL HTTPS publique qui autorise le CORS. Le plus simple : GitHub Pages.
   - Crée un dépôt public `tc-compare`, dépose `index.html`, `manifest.json`, `icon.svg`, `README.md`.
   - Settings → Pages → Branch `main` → Save. L'URL sera `https://<compte>.github.io/tc-compare/`.
2. **Éditer `manifest.json`** : remplace `VOTRE-COMPTE` par ton compte GitHub (3 occurrences).
3. **Installer dans le projet** (droits administrateur du projet requis) :
   Paramètres → Applications et fonctionnalités → Ajouter une extension personnalisée → colle
   `https://<compte>.github.io/tc-compare/manifest.json` → Ajouter → active-la.
4. Ouvre la **vue 3D**, charge la maquette, ouvre l'extension « Comparateur de versions ».
   Au premier lancement, Trimble Connect demande d'autoriser l'accès au jeton : accepte (nécessaire pour lire l'historique des versions).

## Limites à connaître

- Comparaison basée sur les **GUID** : IFC exportés proprement depuis Revit/Tekla OK. Un objet dont le GUID change à chaque export apparaîtra en supprimé + ajouté.
- La « géométrie modifiée » est détectée via la **boîte englobante** (tolérance réglable, 1 mm par défaut). Une modification interne qui ne change pas l'emprise (ex. une réservation au milieu d'un voile) est classée en « propriétés » si une propriété change, sinon pas détectée.
- Les propriétés volatiles (date d'export, n° de révision…) peuvent gonfler le compteur bleu : ajoute-les dans Options → « Propriétés à ignorer ».
- Grosses maquettes (> 100 000 objets) : la lecture peut prendre plusieurs minutes, le navigateur reste utilisable.
- L'extension ne stocke rien et n'envoie aucune donnée en dehors de Trimble Connect.

## Rappel

Trimble Connect for Windows possède une comparaison native (IFC/Tekla) qui crée un « diff model ». Cette extension sert à faire le même contrôle **dans le navigateur**, sans poste Windows, avec liste filtrable et export CSV.
