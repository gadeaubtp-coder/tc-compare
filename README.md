# Comparateur de versions — extension Trimble Connect (vue 3D) · v3.6

Hébergée sur https://gadeaubtp-coder.github.io/tc-compare/ — manifest : https://gadeaubtp-coder.github.io/tc-compare/manifest.json

Compare deux versions d'une même maquette directement dans la vue 3D de Trimble Connect for Browser.

## Ce que fait l'outil

1. Tu choisis une maquette chargée dans la vue 3D, puis la version A (ancienne) et la version B (nouvelle).
2. L'extension charge A, lit tous les objets (GUID, propriétés, emprise), charge B, fait de même, puis compare objet par objet via le GUID.
3. Sur la version B, **règle unique** — la couleur découle de la fiche de l'élément :
   - **orange** = géométrie modifiée **confirmée** : écart net (emprise / centre de gravité / réservation > 20 mm, volume / surface > 5 %), réservation ajoutée ou supprimée, quantité écrite dans l'IFC (Revit, Qto) modifiée, ou petit écart Trimble confirmé par une quantité IFC ;
   - **jaune** = géométrie **à vérifier** : écart au-dessus du bruit mais sous les seuils orange, sans confirmation par l'IFC (variation probable du calcul de Trimble, ex. formes courbes) — ne pas présenter comme modifié sans contrôle ;
   - **bleu** = géométrie inchangée, métadonnées modifiées (classe IFC, propriétés, quantités d'exporteur) ;
   - **vert** = ajouté, **rouge** = supprimé, gris = inchangé (bruit : emprise ≤ 1 mm, centre de gravité ≤ 2 mm, volume / surface ≤ 0,01 %).
   Couleurs opaques. Un contrôle automatique vérifie à chaque comparaison que chaque couleur correspond à sa fiche (« ✔ Contrôle de cohérence »).
   Seuils réglables dans Options. **Test de calibration (spécificité)** : redéposer le même IFC comme nouvelle version → attendu 0 orange, 0 jaune, 0 bleu.
   Réalisé le 18/11/2026 sur 4 maquettes L1MA : SBA_VGE_STR_Marcory, CRL_CFO_MARCORY_VGE, CRL_CFO_PDT_MARCORY_VGE, BTP_MAT.TETRA.GIN.
   - Première passe (v3.1) : 0 orange sur les 4. CFO_PDT : 41 bleus (journal d'import Trimble renuméroté à chaque dépôt) et 5 jaunes (facettage des isolateurs des combinés GIS 90 kV : emprise −5 à −10 mm sur un seul axe, soit 0,11 à 0,24 %, volume calculé conservé).
   - Les deux causes ont été traitées en v3.2 (journal d'import ignoré par défaut) et v3.3 (signature du bruit de facettage).
   - **Seconde passe (v3.3) : 100 % gris sur les 4 maquettes, sans réglage manuel.**
   Reste à faire : le test symétrique de **sensibilité** (modifications connues injectées dans un IFC, vérifier que l'outil les retrouve toutes avec les bonnes valeurs).
4. Bascule « Version A (avant) / Version B (après) » à caméra fixe. Sur A : supprimés en **rouge**, anciennes positions des modifiés en orange.
5. **Maquettes de référence** (ex. CFO) : cochées, elles restent affichées sans couleur pendant la comparaison ; les autres maquettes sont masquées.
6. Clic sur un élément de la liste : zoom dessus et fiche des changements (avant → après, écarts chiffrés). Clic sur un objet dans la vue 3D : sa ligne est surlignée dans la liste.
7. Export CSV (séparateur `;`, compatible Excel FR) : une ligne par élément (interprétation, changements géométriques, propriétés modifiées).

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

- Appariement d'abord par **GUID**. Si les GUID ont été régénérés à l'export (cas constaté sur MAT TETRA v1 → v5), l'outil apparie par **géométrie** : même classe IFC et même emprise, ou emprise voisine (100 mm par défaut, réglable) pour les objets déplacés. Les propriétés d'identité (GUID, Tag, Id…) sont alors ignorées. Désactivable dans Options.
- Un export **fusionné** (toute la maquette en quelques blocs, cas de MAT TETRA v6) ne peut être comparé avec aucune méthode : il faut demander un réexport avec éléments séparés.
- La ligne « Objets comparés : A = …, B = … » sert de contrôle : un écart énorme entre A et B signale un export incomplet ou fusionné.
- La « géométrie modifiée » (orange) combine trois contrôles, avec le motif affiché dans la liste et le CSV :
  1. **emprise / position** (boîte englobante, tolérance 1 mm) ;
  2. **réservations** : chaque IfcOpeningElement est rattaché à son hôte (relation IfcRelVoidsElement) ; un mur dont les ouvertures changent (ajout, suppression, déplacement) passe en orange ;
  3. **géométrie calculée** : volume et surface (CalculatedGeometryValues de Trimble Connect, quantités IFC, propriétés Revit) et **centre de gravité** ; capte un trou modélisé directement dans le mur, sans objet ouverture.
- **Principe directeur (v3.0) : une donnée absente, illisible ou incohérente ne peut JAMAIS produire un orange.** Elle est signalée comme « non comparable » dans la fiche et le CSV, et l'objet n'est pas déclaré modifié. Cela couvre : emprise non fournie par Trimble, centre de gravité sans emprise de contrôle, unité de centre de gravité indéterminable ou discordante entre les deux versions, emprise de réservation illisible d'un seul côté, quantité présente dans une seule version, quantité non numérique, changement d'unité à l'export (rapport voisin de 10³/10⁶/10⁹).
- **Valeurs « non calculées » distinguées des mesures (v3.1).** Trimble ne calcule volume, surface et centre de gravité que s'il a pu reconstruire un solide fermé (`CalculatedGeometryValues › IsSolid`). Quand `IsSolid = 0`, ces valeurs tombent à 0 ou deviennent arbitraires : elles sont écartées de la comparaison et l'objet est signalé comme **défaut de modélisation ou d'export**, jamais comme une modification géométrique. De même, un volume ou une surface qui passe à 0 ou devient négatif d'un seul côté est traité comme « non calculé », pas comme « devenu nul ». L'emprise, lue indépendamment, reste exploitée : un objet réellement déplacé reste orange même si son solide n'est pas fermé.
- **Bruit de facettage des formes courbes (v3.3).** L'emprise est calculée sur le maillage d'affichage : sur les cylindres, isolateurs et tubes, le facettage peut varier d'un import Trimble à l'autre et décaler les points extrêmes de quelques mm sans que l'objet change. Signature retenue : **volume calculé conservé + un seul axe d'emprise touché + écart ≤ 0,5 % de la dimension**. L'objet est alors gris, avec la mention « bruit de reconstruction » et le chiffre exact. Les garde-fous sont vérifiés par test : une translation, une rotation (≥ 2 axes), un écart important, un volume modifié ou l'absence de volume calculé par Trimble ne sont jamais silencieux. Seuil réglable dans Options.
- **Journal d'import Trimble ignoré par défaut** : `Warning.Description`, `Warning.Type`, `Warning.Reference` sont renumérotés à chaque dépôt et ne décrivent pas la maquette. Pré-remplis dans « Propriétés à ignorer », et retirables.
- **Réservations : le compte ne suffit pas, l'identité est exigée (v3.4).** Le nombre de réservations d'un mur vient du rattachement `IfcRelVoidsElement` que Trimble reconstruit à l'import, et ce rattachement échoue précisément quand le solide ne se ferme pas. Une variation du compte n'est déclarée que si les réservations concernées ont réellement disparu de la maquette (ou y sont réellement apparues) ; si la réservation existe toujours dans l'autre version mais n'est plus rattachée au même hôte, c'est le lien Trimble qui a changé, pas la géométrie → « non comparable ». Si `IsSolid = 0`, le compte n'est pas exploité du tout.
- **L'emprise seule ne prouve pas un redimensionnement (v3.5).** L'emprise est une boîte alignée sur les axes X-Y-Z du projet, reconstruite par Trimble : pour un objet qui n'est pas parallèle à ces axes, elle ne correspond à aucune dimension propre de l'objet, et elle peut varier de plusieurs centimètres quand Trimble reconstruit la géométrie autrement. Règle : un changement des **dimensions** de l'emprise n'est déclaré que si le **volume calculé le corrobore**. Volume conservé + un seul axe touché → **gris**, quelle que soit l'ampleur de l'écart : un allongement ou un rabotage réel changerait le volume bien au-dessus du seuil de bruit (60 mm sur une poutre de 4,46 m = 1,34 % de volume, soit 130 × le seuil). Le centre de gravité, contrôlé indépendamment, reste le garde-fou : s'il bouge, la matière a été redistribuée et l'objet n'est pas gris. Volume indisponible → jaune, jamais orange. Volume modifié → orange. Un **déplacement** (dimensions conservées, emprise entière décalée) reste une preuve valable et peut être orange.
- **Preuve exigée pour chaque orange.** Un orange porte toujours, dans sa fiche et dans le CSV, la preuve sur laquelle il repose : « écart net au-delà des seuils de bruit », « quantité écrite dans l'IFC modifiée » ou « écart confirmé par une quantité de l'IFC ».
- **Filet de sécurité à l'exécution.** Avant affichage, chaque orange est contrôlé : au moins une ligne géométrique, au moins une valeur chiffrée, aucune valeur non numérique (∞, NaN), une preuve enregistrée. Tout élément qui échoue est **automatiquement rétrogradé en jaune « à vérifier »** et signalé dans le bandeau de résultat. L'outil ne peut donc pas afficher un orange non étayé.
- **Appariement par proximité** (cas des GUID régénérés) : l'appariement est une hypothèse, pas une certitude. Un déplacement mesuré sur un objet apparié ainsi ne peut pas produire un orange à lui seul ; il faut une confirmation par une quantité de l'IFC, sinon l'objet est jaune.
- **Rotation distinguée du redimensionnement** : une emprise modifiée à volume constant est annoncée comme « objet pivoté ou réorienté », et non comme un redimensionnement ; l'écart résiduel du centre de gravité n'est alors pas interprété comme une découpe intérieure.
- Comparaison **avec tolérances** (v2.0) : volume/surface ±0,01 %, centre de gravité 2 mm, autres valeurs numériques ±1e-6 relatif ; réglables dans Options. Le bruit numérique d'un réexport (µm, cm³) n'est plus compté. Les axes « OrientedBoundingBox » sont ignorés : Trimble peut réorienter la boîte d'un export à l'autre sans que l'objet change.
- Un trou modélisé **sans ouverture IFC** dans une maquette exportée **sans quantités** reste indétectable : l'API de Trimble Connect ne donne pas accès au maillage. Demander l'export IFC avec les quantités de base (Revit : « Exporter les quantités de base »).
- Les réservations (IfcOpeningElement) sont **masquées par défaut** dans Trimble Connect : œil ▾ de la barre d'outils → « IFC Opening Elements ». Cliquer une réservation dans la liste sélectionne aussi son hôte.
- Les propriétés volatiles (date d'export, n° de révision…) peuvent gonfler le compteur bleu : ajoute-les dans Options → « Propriétés à ignorer ».
- Grosses maquettes (> 100 000 objets) : la lecture peut prendre plusieurs minutes, le navigateur reste utilisable.
- **Erreur à la lecture des versions** (v2.8) : le message indique le serveur interrogé et le code. **401** = jeton refusé → Ctrl + F5, puis réinitialiser l'autorisation de l'extension dans ses paramètres et l'accepter à nouveau. **403** = droits insuffisants → l'administrateur du projet donne au compte (ou à son groupe) l'accès en lecture au dossier de la maquette.
- L'extension ne stocke rien et n'envoie aucune donnée en dehors de Trimble Connect.

## Rappel

Trimble Connect for Windows possède une comparaison native (IFC/Tekla) qui crée un « diff model ». Cette extension sert à faire le même contrôle **dans le navigateur**, sans poste Windows, avec liste filtrable et export CSV.
