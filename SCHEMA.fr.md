# Le fichier de déclaration - format v0.4

*[English version](SCHEMA.md)*

Définit la grammaire que chaque clause de la norme vérifie. Volontairement ennuyeux : la leçon
de llms.txt est qu'un simple fichier JSON à un chemin fixe et facile à deviner l'emporte sur
toute solution ingénieuse.

## Emplacement

```
https://<site>/.well-known/data-dignity.json
```

Le préfixe `/.well-known/` est l'emplacement normalisé par l'IETF pour les métadonnées au
niveau de l'origine (RFC 8615) - `security.txt` s'y trouve, tout comme le modèle que ceci
reprend. `data-dignity.json` est le nom de cette norme dans cet espace; ce n'est pas encore un
suffixe « well-known » enregistré auprès de l'IETF, et son enregistrement selon le processus de
la RFC 8615 est prévu une fois le format stabilisé.

**Portée : une déclaration couvre exactement l'origine qui la sert, et rien d'autre.** Une
origine est le tuple complet que le Web définit (RFC 6454) : schéma, hôte canonique, port
effectif - jamais un préfixe de chaîne, jamais un suffixe de nom d'hôte. Le fichier à
`https://example.com/.well-known/data-dignity.json` parle pour `example.com` seulement - pas
pour `api.example.com`, ni `shop.example.com`, ni aucun autre hôte. Chaque origine qui traite
des données transmises par un agent publie sa propre déclaration. Un vérificateur NE DOIT PAS
appliquer la déclaration d'une origine à une autre. **La même règle vaut en cours de session :
lorsqu'un agent suit une navigation, une redirection ou une transition d'API vers une nouvelle
origine, celle-ci est hors de la déclaration précédente à moins de déclarer elle-même** - une
transaction qui commence sur une origine couverte n'emporte pas cette couverture vers les
origines où elle aboutit.

## Sécurité des versions

Une version de format est un contrat de sécurité, et accepter un vieux contrat pour toujours
est un canal de dégradation : un site qui préfère un examen plus faible n'aurait qu'à publier
le plus ancien format accepté et à se soustraire à chaque protection ajoutée depuis. La règle
est donc explicite :

- **Un vérificateur impose, par politique, une version minimale pour la notation.** Une
  déclaration plus ancienne que ce minimum PEUT être analysée et affichée à titre
  d'information, mais elle NE DOIT PAS recevoir une note de la version actuelle sur une clause
  ou sur la note composite - elle est rapportée comme **LEGACY**, qui n'est ni une lettre ni
  « non déclarée » : cela signifie *déclarée sous un contrat de sécurité obsolète*.
- **La négociation de version NE DOIT PAS dégrader silencieusement les exigences de
  sécurité.** Un vérificateur n'applique jamais les règles plus faibles d'un format antérieur
  pour ensuite rapporter le résultat sur l'échelle actuelle.
- Les formats plus anciens prennent leur retraite : une fois que le successeur d'une version a
  été publié pendant une période de commentaires définie, la version plus ancienne quitte
  l'ensemble noté. L'ensemble noté actuel est exactement : **v0.4**.

## Sérialisation stricte

La déclaration est un objet de politique de sécurité, alors sa façon d'être analysée est
elle-même une propriété de sécurité - la pire classe de défauts d'implémentation est le
différentiel d'analyseurs : un validateur lisant un premier sens, un vérificateur un deuxième,
un agent agissant sur le troisième, le plus privilégié. Règles normatives :

- Une déclaration DOIT être du JSON valide encodé en UTF-8. Un encodage malformé est un rejet,
  pas une réparation.
- **Des noms de membres dupliqués dans un objet DOIVENT provoquer un rejet.** La RFC 8259
  avertit que les noms dupliqués rendent le comportement dépendant de l'implémentation - dans
  ce format, c'est une attaque de confusion de politique (`"receipt_support": false` et
  `"receipt_support": true` dans le même objet), donc c'est une erreur, jamais un choix.
- **Des membres inconnus dans la grammaire de base DOIVENT provoquer un rejet.** Tout ce qui
  est propre au site va sous l'unique membre `extensions`, dont un consommateur DOIT ignorer le
  contenu qu'il ne reconnaît pas. Base stricte, extensions explicites - jamais « n'importe quoi
  n'importe où ».
- Les implémentations DOIVENT imposer une taille totale bornée, une profondeur d'imbrication
  bornée, des longueurs de chaînes bornées et des longueurs de tableaux bornées - une
  déclaration comptant dix mille tiers ou obligations légales de conservation est une attaque
  contre le vérificateur, pas une divulgation - et DOIVENT analyser de façon déterministe :
  aucune coercition dépendante de l'analyseur, aucune acceptation de presque-JSON.
- Chaque valeur de déclaration est une donnée contrôlée par un attaquant et NE DOIT PAS être
  interprétée comme des instructions exécutables par un consommateur, quel qu'il soit -
  vérificateur, agent, ou quoi que ce soit en aval.

## Structure

Ceci est un exemple illustratif, pas la déclaration réelle d'une organisation existante. Le bloc
de code ci-dessous reste en anglais, langue de travail du format lui-même - un site francophone
écrirait ses propres valeurs de texte libre (comme `notes`) dans sa propre langue :

```json
{
  "version": "0.4",
  "declared": "2026-01-01T00:00:00Z",
  "expires": "2026-07-01T00:00:00Z",
  "last_reviewed": "2026-01-01T00:00:00Z",
  "retention": {
    "content": "none",
    "identity": "90d",
    "network_metadata": "30d",
    "notes": "Identity is support tickets, kept 90 days to resolve disputes. Nothing else is stored."
  },
  "custom_classes": {
    "loyalty-profile": ["derived", "identity"]
  },
  "third_parties": [
    { "name": "Example CDN Co", "identifier": "example-cdn.example", "role": "infrastructure",
      "recipient": "direct", "purpose": "content delivery", "data_seen": ["network_metadata"],
      "jurisdiction": ["example-region"] },
    { "name": "Example Model Host", "identifier": "models.example", "role": "model_provider",
      "recipient": "downstream", "via": "example-cdn.example", "purpose": "inference",
      "data_seen": ["content"], "jurisdiction": ["example-region"] }
  ],
  "jurisdiction": {
    "processing": ["example-region"],
    "storage": ["example-region"],
    "backups": ["example-region"],
    "legal": ["example-region"]
  },
  "deletion": {
    "handle": "https://example.com/api/agent-delete",
    "method": "POST",
    "auth": "receipt-token",
    "covers": ["identity"],
    "legal_holds": [
      { "covers": "consent records", "purpose": "proving consent was given",
        "legal_basis": "applicable-privacy-law", "review_by": "2027-01-01T00:00:00Z",
        "reason": "consent must remain provable under applicable privacy law" }
    ]
  },
  "receipt_support": false,
  "extensions": {}
}
```

### Classes de données

Des descriptions en texte libre invitent des lectures radicalement différentes d'une même
déclaration; `data_seen`, les clés de `retention` et `deletion.covers` puisent donc dans un
seul vocabulaire **fermé** :

`content` (ce que l'interaction a produit - messages, requêtes, téléversements), `identity`
(l'identité de l'interlocuteur), `authentication` (jetons, identifiants), `network_metadata`
(adresse IP, en-têtes), `device_metadata` (empreinte, agent utilisateur), `financial`,
`health`, `location`, `files`, `derived` (tout ce qui est calculé à partir du reste -
vecteurs, scores, profils).

**Les classes personnalisées s'ajoutent au vocabulaire normalisé, elles ne le remplacent
jamais.** Une déclaration PEUT nommer des classes propres au site, mais chacune DOIT être
associée à une ou plusieurs classes normalisées dans `custom_classes`, et une déclaration NE
DOIT PAS utiliser une classe personnalisée pour échapper à une classe normalisée qui lui
convient manifestement. Une étiquette vague (« télémétrie opérationnelle », « renseignement de
risque ») sans association est une évasion sémantique, et un vérificateur évalue toute classe
personnalisée non associée comme non déclarée - mécaniquement, pas par jugement humain.
**L'association doit aussi préserver la sensibilité** : une classe personnalisée dont les
données relèvent manifestement de `health`, `financial`, `location` ou `authentication` DOIT
être associée à cette classe - l'associer seulement à une classe plus large et plus fade
(`derived`, `content`) pour diluer ce qu'elle est, c'est la même évasion un niveau plus haut,
et c'est noté de la même façon. « Données transmises par un agent » désigne toute classe
ci-dessus que l'interaction d'un agent amène le site à recevoir - y compris les métadonnées que
l'agent n'a pas délibérément envoyées.

### Notes sur les champs

- `declared` / `expires` / `last_reviewed` - une déclaration est une affirmation sur le
  présent, pas sur toujours. `expires` est OBLIGATOIRE : une déclaration expirée est évaluée
  comme périmée, jamais comme une vérité actuelle, parce qu'un site peut changer toute son
  infrastructure longtemps après avoir publié une déclaration impeccable. Douze mois est la
  fenêtre maximale raisonnable.
- `retention` - **une table vérifiable par machine, de classe de données vers durée de
  conservation** : `none`, `indefinite`, ou un nombre de jours/mois/années (`90d`, `6m`,
  `2y`). `content` est toujours obligatoire. **Règle de couverture : toute classe qui apparaît
  où que ce soit dans la déclaration - dans le `data_seen` d'un tiers, dans
  `deletion.covers`, ou via une association de classe personnalisée - DOIT avoir une entrée de
  conservation**, pour qu'un vérificateur confirme mécaniquement que rien de vu n'est laissé
  sans compte, sans aucune interprétation. La plupart des sites réels conserveront certaines
  données liées à l'identité pour des raisons légitimes (soutien, prévention de la fraude,
  obligations légales) même s'ils ne stockent aucun contenu - l'honnêteté par classe est
  récompensée plutôt qu'un « nous ne conservons rien » général qu'une vérification finirait
  par contredire. `notes` reste en texte libre pour l'explication humaine.
- `third_parties` - **le graphe de traitement, pas seulement le premier saut.** La divulgation
  est transitive : une déclaration couvre chaque destinataire en aval connu, ou
  contractuellement autorisé, à recevoir des données transmises par un agent à travers un
  fournisseur, sous-traitant, fournisseur de modèle, fournisseur d'infrastructure ou autre
  intermédiaire déclaré - « nous l'envoyons à A » n'est pas honnête quand A le transmet à C,
  et « A le voit » en cachant C derrière A est l'échappatoire que cette règle ferme. Chaque
  entrée porte une position `recipient` (`direct` ou `downstream`, `via` nommant
  l'intermédiaire déclaré par lequel un destinataire en aval reçoit), un `role` (`processor`,
  `subprocessor`, `infrastructure`, `analytics`, `security`, `model_provider`), ce qu'elle
  voit, et où elle traite; la chaîne DOIT se terminer à des frontières déclarées - aucune
  partie déclarée ne peut être une porte vers des parties non déclarées. **`identifier` est
  OBLIGATOIRE et constitue l'identité lisible par machine - un domaine stable ou un URI
  contrôlé par l'exploitant; `name` n'est que la présentation.** Les familles d'entreprises et
  les noms semblables (« Example AI », « Example AI Analytics », un fournisseur racheté)
  rendent un simple nom invérifiable; un identifiant est ce qu'un vérificateur peut réellement
  comparer au trafic observé. « Voit » inclut les transferts côté serveur qu'aucun observateur
  externe du réseau ne peut surveiller. Un tableau vide est une réponse valide et évaluable.
- `jurisdiction` - où les données sont traitées, stockées et sauvegardées, en réponses
  distinctes, parce que ce sont habituellement des endroits différents. Les codes de région
  sont préférables lorsqu'ils sont vérifiables de façon indépendante. La juridiction juridique
  (quel droit s'applique) peut différer de l'emplacement physique; lorsqu'elles diffèrent, dire
  les deux.
- `deletion` - un mécanisme réel et activable par une machine. L'essentiel est qu'il soit
  activable par machine, pas un paragraphe de billet de soutien. **`deletion.handle` DOIT
  avoir la même origine que la déclaration qui l'annonce** - le tuple d'origine complet,
  comparé après canonicalisation (minuscules, hôte normalisé IDNA/punycode, pas de point
  final, pas d'userinfo, port effectif 443), jamais par préfixe de chaîne. Un point de
  suppression d'une autre origine est le chemin d'exfiltration de capacité le plus net de ce
  format - un agent qui exercerait plus tard le chemin déclaré porterait son jeton de
  suppression vers l'origine que la déclaration a nommée - ce n'est donc pas une règle de
  style, c'est la frontière. Un vérificateur donne F à cette clause pour un point d'une autre
  origine. La déclaration doit aussi dire comment un appelant prouve son droit de supprimer ce
  qu'il demande (`auth`), parce qu'un point de suppression sans authentification est une arme
  pointée vers les propres utilisateurs du site. Exigences sur le point lui-même :
  - il DOIT exiger une preuve liant la demande à l'interaction visée (un jeton de reçu là où
    les reçus existent, ou un secret équivalent propre à l'interaction) - jamais de suppression
    sur une demande non authentifiée;
  - **le jeton voyage dans un en-tête ou un corps de requête, jamais dans un paramètre de
    requête d'URL** - les chaînes de requête fuient par les journaux d'accès, les journaux de
    mandataires, les outils d'analytique, l'historique du navigateur, la propagation du
    référent et la surveillance, et un jeton de suppression qui fuit est une capacité qui
    fuit;
  - il DOIT être idempotent - la même suppression demandée deux fois n'est pas une erreur;
  - il NE DOIT PAS révéler si un jeton est valide autrement qu'en traitant une demande
    autorisée - réponses d'erreur uniformes et temps de réponse uniformes, parce qu'un oracle
    de validité permet à un attaquant de confirmer un jeton volé avant de le dépenser;
  - sa réponse de succès DOIT dire ce qui s'est réellement passé : `deleted`, `queued` (avec un
    délai prévu), ou `nothing-held` - un simple HTTP 200 n'est pas une réponse;
  - tout ce qu'il ne peut pas supprimer doit figurer dans `legal_holds` - et **une obligation
    légale de conservation est bornée, jamais un mot magique** : chaque entrée nomme les
    enregistrements précis qu'elle couvre, l'objet précis, un identifiant `legal_basis`, et
    une date `review_by` (ou une expiration) après laquelle l'obligation doit être
    rejustifiée. Une obligation générale ou indéfinie, sans portée énoncée ni condition de
    révision, est invalide; une obligation prétendant couvrir toutes les classes est un signal
    d'alarme, noté comme tel. La norme ne juge pas si la loi citée est fondée - elle empêche
    « exigé par la loi applicable » de tout excuser, pour toujours;
  - il DEVRAIT limiter le débit et surveiller les abus - une suppression authentifiée reste un
    point d'écriture qu'un adversaire peut marteler.

  La forme minimale de requête et de réponse, normative pour que les implémentations
  convergent et que les sondes aient une seule surface à tester plutôt que plusieurs :

  ```json
  POST /api/agent-delete
  { "interaction_id": "an-opaque-id", "deletion_token": "an-opaque-single-use-token" }

  200 { "status": "deleted", "covers": ["identity"], "completed": "2026-01-02T00:00:00Z" }
  200 { "status": "queued", "expected_by": "2026-01-09T00:00:00Z" }
  200 { "status": "nothing-held" }
  401 { "status": "unauthorized" }
  410 { "status": "token-expired-or-used" }
  ```
- `receipt_support` - indique si le site prend en charge les reçus par interaction. `false` est
  une réponse honnête et évaluable, pas un échec en soi. Lorsqu'il est pris en charge, un reçu
  est un petit objet JSON :

  ```json
  {
    "interaction_id": "an-opaque-id",
    "received_classes": ["content", "identity"],
    "retention": { "content": "none", "identity": "30d" },
    "third_parties": ["example-cdn.example"],
    "deletion_token": "an-opaque-single-use-token"
  }
  ```

  **Un reçu NE DOIT PAS reproduire le contenu qu'il décrit.** Il nomme des classes de données
  et leur état de traitement, jamais les mots, fichiers ou valeurs eux-mêmes - un reçu qui
  répète une question de santé est une nouvelle copie de la chose la plus sensible de
  l'interaction. **Le `interaction_id` d'un reçu DOIT identifier une interaction sans
  ambiguïté au sein de son origine** - conceptuellement lié à l'origine, à un nonce
  d'interaction et à un horodatage - pour que l'entrée du registre de l'agent et le reçu du
  site puissent être réconciliés, et qu'aucune des deux parties ne puisse pointer un reçu vers
  une autre interaction après coup. Tant que les reçus ne sont pas signés, un reçu est
  lui-même une *affirmation* au niveau de preuve « déclarée » - il ne prouve pas que le site
  l'a produit pour cette interaction, et un vérificateur NE DOIT PAS étiqueter le contenu d'un
  reçu plus haut que « déclaré ». Le reçu signé (une signature sur l'enregistrement canonique
  de l'interaction) est là où l'extension `signature` ci-dessous prend toute sa valeur. **Un
  reçu portant un `deletion_token` porte une capacité vivante, et les deux côtés le traitent
  ainsi** : le site garde le jeton hors de ses propres journaux et caches, et l'agent le range
  dans son coffre de capacités protégé (voir les règles du registre dans le document
  principal) - **jamais dans un export en clair, jamais dans des journaux, des rapports de
  plantage ou des outils d'analytique, et jamais dans le contexte d'invite d'un modèle**,
  parce qu'un jeton passé par le contexte d'un modèle de langage a été divulgué à tout ce que
  ce contexte atteint. Le chemin d'émission est le maillon faible pour lequel les règles de
  liaison existent : les intermédiaires qui mettent des reçus en cache, les intergiciels de
  cadriciels qui les journalisent, les extensions qui les lisent sont autant de risques nommés
  que la liaison à usage unique et à courte durée du jeton contient, sans les effacer.
- `extensions` - le seul foyer pour tout ce qui est propre au site au-delà de
  `custom_classes`. Clés avec espace de noms (`"vendor.example/feature"`), contenu ignoré par
  les consommateurs qui ne le reconnaissent pas, jamais un endroit pour reformuler ou
  contredire un membre de base.
- `signature` - OPTIONNEL, réservé : une déclaration peut porter une signature détachée la
  liant à son origine, pour qu'un agent détecte un fichier altéré ou rejoué, même au-delà d'un
  CDN compromis. HTTP Message Signatures (RFC 9421) est le mécanisme candidat. Le profil
  concret (algorithmes, découverte des clés) n'est délibérément pas spécifié ici : il ne sera
  publié qu'après une revue cryptographique indépendante, et d'ici là, le transport HTTPS est
  le modèle de confiance, énoncé clairement comme une limite plutôt que déguisé en garantie.

## Liaison des capacités et modèle de rejeu

Un seul principe ferme toute une famille d'attaques par composition; il est donc énoncé une
fois et s'applique partout : **toute capacité sensible que cet écosystème émet - un jeton de
suppression, un jeton de reçu, une future autorisation d'opération groupée, une future
signature - DOIT être liée à son ORIGINE, son INTERACTION, son OBJET, une EXPIRATION et un
NONCE.** Une capacité que n'importe quel porteur peut dépenser n'importe où, sur n'importe
quoi, pour toujours, est un identifiant au porteur qui attend de fuir à travers une frontière
de couche; les travaux de l'IETF sur les jetons liés à l'émetteur (RFC 9449) documentent
exactement ce mode d'échec. Appliqué ici :

- un jeton de suppression n'est valide qu'à l'origine qui l'a émis, que pour l'interaction
  qu'il nomme, que pour la suppression, à usage unique et à courte durée de vie - et non
  exportable là où la plateforme peut l'imposer;
- **le modèle de rejeu est formel, pas implicite** : une capacité ou un reçu porte son nonce
  d'interaction, son heure d'émission, son expiration et son audience (l'origine à laquelle il
  est destiné), et NE DOIT PAS être accepté hors de cette portée exacte ou de cette fenêtre de
  validité - ni par le site, ni par l'agent, ni par quoi que ce soit qui rejoue le trafic
  entre eux. Les implémentations n'inventent pas leur propre sémantique de rejeu;
- posséder un jeton n'est jamais une autorisation en soi - un agent détenant un registre plein
  de jetons de suppression a les *moyens* de supprimer, et il lui faut encore l'autorisation
  explicite de la personne pour *agir* (voir les règles côté agent du document principal);
- la construction concrète du jeton est délibérément non profilée ici, en attente de la même
  revue cryptographique indépendante que l'extension de signature - les exigences de liaison
  sont normatives dès maintenant, le format ne l'est pas.

## Règles pour les vérificateurs

Le vérificateur est un client HTTP traitant des entrées qu'un adversaire contrôle, et une
déclaration hostile est la façon évidente de l'attaquer. Tout vérificateur conforme :

- NE DOIT PAS déréférencer `deletion.handle` sans y être invité. La présence et la forme se
  vérifient depuis la déclaration seule; appeler réellement un point de suppression ne se fait
  qu'avec le consentement du site et des données synthétiques, ou pas du tout.
- DOIT refuser toute URL déclarée qui se résout vers une adresse de bouclage, privée, de lien
  local, multidiffusion ou autrement réservée. **La résolution est là où cette défense se perd
  habituellement, alors l'exigence est précise : chaque adresse résolue DOIT être validée -
  chaque enregistrement A et chaque AAAA, après toute chaîne CNAME, y compris les formes IPv6
  encapsulant IPv4 - et la connexion DOIT être établie vers une adresse déjà validée, sans
  nouvelle résolution entre la validation et la connexion qui pourrait changer la destination.
  Si la résolution est ambiguë, partagée entre adresses sûres et non sûres, ou change pendant
  la vérification, la requête échoue en se fermant.**
- DOIT comparer les origines comme des origines - schéma, hôte canonique, port effectif, selon
  la RFC 6454 et les règles de normalisation de la RFC 3986 - en traitant explicitement les
  ports par défaut, les littéraux IPv6, les formes IDN/punycode, les noms d'hôtes à point
  final et l'userinfo, et NE DOIT PAS comparer des origines par préfixe de chaîne ou suffixe
  de nom d'hôte.
- NE DOIT PAS suivre une redirection vers une origine différente de celle qu'il vérifie.
- DOIT imposer des limites strictes de taille de réponse et de délai sur tout ce qu'il
  télécharge.
- NE DOIT PAS joindre d'identifiants, de témoins ou d'autorité ambiante à une requête
  provoquée par une déclaration.
- DOIT appliquer les règles de sérialisation ci-dessus (UTF-8, rejet des noms dupliqués, rejet
  des membres de base inconnus, bornes) avec un seul analyseur déterministe - et traiter
  **chaque valeur de déclaration comme une donnée contrôlée par un adversaire, jamais comme
  une instruction exécutable - dans les rapports, et tout autant en aval** : un `name`, un
  `purpose`, un `reason` ou une classe personnalisée peut finir dans une ligne de journal, un
  argument de shell, une requête, une URL, un export de tableur ou une étiquette de métriques,
  et c'est du texte non fiable dans chacun de ces endroits, pas seulement dans du HTML rendu.
- DOIT consigner la provenance de ce qu'il a noté : récupéré depuis l'origine ou servi par un
  cache (un intermédiaire peut servir un fichier périmé ou substitué), signé ou non signé - et
  NE DOIT PAS présenter une déclaration non signée servie par un cache comme une preuve
  équivalente à une déclaration authentifiée à l'origine.
- DOIT garder sa preuve déterministe sous ses notes : **chaque résultat de test individuel est
  exactement l'un de PASS, FAIL, UNKNOWN ou NOT_TESTED, et les notes sont calculées à partir
  de ces faits par la règle de notation publiée.** Le jugement peut décider quoi tester; il ne
  décide jamais ce que dit un fait consigné. Deux vérificateurs disposant des mêmes faits
  arrivent à la même note, et une preuve incomplète apparaît comme des faits
  UNKNOWN/NOT_TESTED, jamais comme des lettres optimistes.
- DOIT noter les tiers observés sur le transport et les fournisseurs déclarés côté serveur
  comme des faits séparés, jamais fusionnés en une seule affirmation « tiers vérifiés » - le
  premier est une preuve, le second une déclaration, et un attaquant déjoue le test empirique
  le plus fort en gardant le comportement dangereux entièrement côté serveur.
- DOIT énoncer sa surface d'observation quand il rapporte une preuve observée : observé au
  client, sur le transport, à l'origine, par une divulgation côté serveur, ou par un moniteur
  externe. Deux vérificateurs regardant des surfaces différentes voient un trafic différent;
  une affirmation « observée » qui ne dit pas où elle a observé n'est pas comparable, et des
  notes qui divergent entre surfaces sont un constat, pas une contradiction. **Un plafond
  structurel en découle, énoncé pour qu'aucune note ne prétende plus : un comportement purement
  côté serveur - une relation de sous-traitance, un transfert interne - ne peut jamais
  atteindre « vérifié » par la seule observation externe.** Ces affirmations restent
  « déclarées » jusqu'à ce que quelque chose de plus fort existe (une attestation signée, un
  rapport de transparence, une procédure judiciaire), et un vérificateur NE DOIT PAS les
  étiqueter plus haut.
- DEVRAIT, lors des sondes de rétention, rendre son trafic synthétique réellement difficile à
  identifier - charges aléatoires, cadence réaliste, points de vue et réseaux variés,
  chorégraphie de test variée périodiquement - et DEVRAIT revérifier la réapparition à
  intervalles espacés (immédiatement, après un jour, après une semaine ou plus) avant de
  conclure « non réapparu ». **Le test actif est classé comme preuve comportementale en boîte
  noire, jamais comme vérification d'un état interne** : un site sophistiqué peut identifier
  le *comportement* d'un test même quand il ne peut pas reconnaître ses *données*, alors une
  sonde propre est la preuve que le site s'est bien comporté pour cette sonde, depuis ce point
  de vue, dans cette fenêtre - l'affirmation honnête, et la seule qu'une sonde puisse faire.
  La méthodologie complète des canaris est son propre document sur la feuille de route; ces
  planchers tiennent d'ici là.
- DOIT appliquer les règles de sécurité des versions ci-dessus aux fichiers de formats
  antérieurs : analyser et afficher, rapporter LEGACY, jamais noter selon des règles
  remplacées sur l'échelle actuelle.

Les sites sont tenus à la règle miroir : **un site NE DOIT PAS traiter différemment un trafic
qu'il croit être un test de conformité.** Les tests de rétention utilisent des données
synthétiques aléatoires et non marquées, précisément pour que reconnaître et supprimer le test
soit plus difficile que de simplement bien se comporter; un site pris à le faire est évalué
comme ayant déclaré faux, la pire note du bulletin.

**Une limite énoncée pour que personne ne lise au-delà : le test de réapparition n'établit pas
l'absence de représentations transformées, inférées, agrégées, hachées, vectorisées,
classifiées ou autrement dérivées.** Un site peut supprimer les mots et garder un vecteur, une
classification, un score de risque, un profil - le texte exact ne réapparaît jamais, et quelque
chose qui en dérive est conservé quand même. C'est pourquoi `derived` est sa propre classe de
déclaration avec sa propre entrée de conservation et ses propres faits notés séparément,
pourquoi des sondes de reconnaissance (le site se *comporte-t-il* comme s'il se souvenait) font
partie de la feuille de route de vérification aux côtés du rejeu exact, et pourquoi un test de
réapparition propre est une preuve pour `content`, jamais pour `derived`.

## Le modèle de menace de la dignité

Les adversaires contre lesquels ce format est conçu, nommés pour que chaque règle puisse être
retracée à l'attaquant qu'elle arrête et que chaque implémentation puisse tester contre la même
liste :

- **T1 - site malhonnête** : déclare des choses fausses. Contré par les sondes, la règle du
  déclaré-faux-plafonne-la-composite, et les étiquettes de preuve.
- **T2 - déclaration malveillante** : le fichier lui-même comme entrée d'attaque. Contré par
  la sérialisation stricte, les règles anti-SSRF, la suppression de même origine, et la règle
  des valeurs non fiables.
- **T3 - CDN ou cache compromis** : sert une déclaration périmée ou substituée. Contré par la
  consignation de la provenance, `expires`, et à terme l'extension de signature.
- **T4 - fournisseur malveillant ou perméable** : une partie déclarée qui transmet plus loin.
  Contré par la divulgation transitive et la règle des frontières déclarées.
- **T5 - agent compromis** : l'outil de la personne retourné contre elle. Contré par la
  séparation du registre, la liaison des capacités, et l'autorisation de suppression groupée.
- **T6 - module d'agent malveillant ou consigne injectée** : falsification d'autorisation
  depuis l'intérieur. Contré par possession-n'est-pas-autorisation et la cérémonie explicite
  de suppression groupée.
- **T7 - attaquant du réseau** : interception et rejeu. Contré par HTTPS, le modèle de rejeu,
  et l'expiration des capacités.
- **T8 - registre compromis** : l'historique des divulgations comme butin. Contré par la
  séparation audit/capacités et la règle du sans-contenu.
- **T9 - site conscient des tests** : ne se comporte bien que pour l'auditeur. Contré par des
  canaris aléatoires non marqués, une chorégraphie variée, et la classification en preuve de
  boîte noire.
- **T10 - différentiel d'analyseurs ou d'implémentations** : trois composants lisant trois
  sens. Contré par la sérialisation stricte, la base fermée, et le modèle de faits
  déterministe.

## Les attaques par composition contre lesquelles ce format est construit

Chaque règle ci-dessus ferme un seul trou; cette section nomme les chaînes, parce que chaque
étape ci-dessous est individuellement permise quelque part et que la chaîne est l'attaque. La
chaîne canonique - la chaîne de dégradation de la dignité : un site publie une déclaration
d'apparence valide; cache des catégories significatives derrière des classes personnalisées
non associées; déclare un point de suppression d'une autre origine; émet des reçus dont les
jetons de suppression sont consciencieusement rangés par l'agent; un nettoyage autonome
ultérieur suit le chemin déclaré et porte le jeton vers l'origine de l'attaquant; l'attaquant
détient maintenant une capacité vivante, et le registre de l'agent détient l'historique
complet des divulgations pour une compromission de second niveau. La règle d'association des
classes personnalisées, le DOIT de même origine pour la suppression, la liaison des capacités,
la règle d'autorisation groupée et la séparation du registre coupent chacun un maillon de
cette chaîne; ensemble, ils les coupent tous. Les autres chaînes connues : rester sur une
version de format obsolète pour esquiver chaque règle ajoutée depuis (fermée par la sécurité
des versions), une infrastructure CDN partagée faisant passer deux origines pour une seule
(fermée par la portée d'origine exacte et la règle de transition en cours de session), cacher
un destinataire derrière un agrégateur déclaré (fermée par la divulgation transitive), et un
vérificateur transformé en arme par les URL et valeurs qu'une déclaration lui fournit (fermée
par les règles pour vérificateurs ci-dessus).

## Questions ouvertes

- Le profil concret de signature et la construction concrète du jeton de suppression (voir
  ci-dessus) - tous deux définis comme points d'extension avec des exigences de liaison
  normatives, délibérément non profilés en attente d'une revue cryptographique indépendante.
- Gestion des versions et migration. La v0.4 est un changement incompatible avec la grammaire
  publiée de la v0.3 (les `legal_holds` bornés exigent désormais `purpose`, `legal_basis` et
  `review_by`; `recipient`/`via` et `extensions` ont été ajoutés; la grammaire de base est
  fermée) - et elle est versionnée plutôt que modifiée sur place, exactement ce que la
  section de sécurité des versions ci-dessus exige de tous les autres. Les formats
  antérieurs relèvent de ces mêmes règles : analysés, affichés, rapportés LEGACY, jamais
  notés sur l'échelle actuelle.

## Statut

Ébauche v0.4, publiée pour commentaires en parallèle avec le document principal de la norme.
