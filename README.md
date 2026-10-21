# \# Comparateur de versions — extension Trimble Connect (vue 3D) · v1.9

# 

# Hébergée sur https://gadeaubtp-coder.github.io/tc-compare/ — manifest : https://gadeaubtp-coder.github.io/tc-compare/manifest.json

# 

# Compare deux versions d'une même maquette directement dans la vue 3D de Trimble Connect for Browser.

# 

# \## Ce que fait l'outil

# 

# 1\. Tu choisis une maquette chargée dans la vue 3D, puis la version A (ancienne) et la version B (nouvelle).

# 2\. L'extension charge A, lit tous les objets (GUID, propriétés, emprise), charge B, fait de même, puis compare objet par objet via le GUID.

# 3\. Sur la version B : \*\*vert\*\* = ajouté, \*\*orange\*\* = géométrie/position modifiée, \*\*bleu\*\* = propriétés seules modifiées, gris transparent = inchangé.

# 4\. Bascule « Version A (avant) / Version B (après) » à caméra fixe. Sur A : supprimés en \*\*rouge\*\*, anciennes positions des modifiés en orange.

# 5\. \*\*Maquettes de référence\*\* (ex. CFO) : cochées, elles restent affichées sans couleur pendant la comparaison ; les autres maquettes sont masquées.

# 5\. Clic sur un élément de la liste : zoom dessus et affichage des propriétés modifiées (avant → après).

# 6\. Export CSV (séparateur `;`, compatible Excel FR) : une ligne par propriété modifiée.

# 

# \## Installation (≈ 10 min)

# 

# 1\. \*\*Héberger les fichiers\*\* sur une URL HTTPS publique qui autorise le CORS. Le plus simple : GitHub Pages.

# &#x20;  - Crée un dépôt public `tc-compare`, dépose `index.html`, `manifest.json`, `icon.svg`, `README.md`.

# &#x20;  - Settings → Pages → Branch `main` → Save. L'URL sera `https://<compte>.github.io/tc-compare/`.

# 2\. \*\*Éditer `manifest.json`\*\* : remplace `VOTRE-COMPTE` par ton compte GitHub (3 occurrences).

# 3\. \*\*Installer dans le projet\*\* (droits administrateur du projet requis) :

# &#x20;  Paramètres → Applications et fonctionnalités → Ajouter une extension personnalisée → colle

# &#x20;  `https://<compte>.github.io/tc-compare/manifest.json` → Ajouter → active-la.

# 4\. Ouvre la \*\*vue 3D\*\*, charge la maquette, ouvre l'extension « Comparateur de versions ».

# &#x20;  Au premier lancement, Trimble Connect demande d'autoriser l'accès au jeton : accepte (nécessaire pour lire l'historique des versions).

# 

# \## Limites à connaître

# 

# \- Appariement d'abord par \*\*GUID\*\*. Si les GUID ont été régénérés à l'export (cas constaté sur MAT TETRA v1 → v5), l'outil apparie par \*\*géométrie\*\* : même classe IFC et même emprise, ou emprise voisine (100 mm par défaut, réglable) pour les objets déplacés. Les propriétés d'identité (GUID, Tag, Id…) sont alors ignorées. Désactivable dans Options.

# \- Un export \*\*fusionné\*\* (toute la maquette en quelques blocs, cas de MAT TETRA v6) ne peut être comparé avec aucune méthode : il faut demander un réexport avec éléments séparés.

# \- La ligne « Objets comparés : A = …, B = … » sert de contrôle : un écart énorme entre A et B signale un export incomplet ou fusionné.

# \- La « géométrie modifiée » (orange) combine trois contrôles, avec le motif affiché dans la liste et le CSV :

# &#x20; 1. \*\*emprise / position\*\* (boîte englobante, tolérance 1 mm) ;

# &#x20; 2. \*\*réservations\*\* : chaque IfcOpeningElement est rattaché à son hôte (relation IfcRelVoidsElement) ; un mur dont les ouvertures changent (ajout, suppression, déplacement) passe en orange ;

# &#x20; 3. \*\*quantités\*\* (NetVolume, NetSideArea, Volume, Surface…) : capte un trou modélisé directement dans le mur, sans objet ouverture.

# \- Un trou modélisé \*\*sans ouverture IFC\*\* dans une maquette exportée \*\*sans quantités\*\* reste indétectable : l'API de Trimble Connect ne donne pas accès au maillage. Demander l'export IFC avec les quantités de base (Revit : « Exporter les quantités de base »).

# \- Les réservations (IfcOpeningElement) sont \*\*masquées par défaut\*\* dans Trimble Connect : œil ▾ de la barre d'outils → « IFC Opening Elements ». Cliquer une réservation dans la liste sélectionne aussi son hôte.

# \- Les propriétés volatiles (date d'export, n° de révision…) peuvent gonfler le compteur bleu : ajoute-les dans Options → « Propriétés à ignorer ».

# \- Grosses maquettes (> 100 000 objets) : la lecture peut prendre plusieurs minutes, le navigateur reste utilisable.

# \- L'extension ne stocke rien et n'envoie aucune donnée en dehors de Trimble Connect.

# 

# \## Rappel

# 

# Trimble Connect for Windows possède une comparaison native (IFC/Tekla) qui crée un « diff model ». Cette extension sert à faire le même contrôle \*\*dans le navigateur\*\*, sans poste Windows, avec liste filtrable et export CSV..

