# Le fichier de déclaration - format v0.2

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

**Portée : une déclaration couvre exactement l'origine qui la sert, et rien d'autre.** Le
fichier à `https://example.com/.well-known/data-dignity.json` parle pour `example.com`
seulement - pas pour `api.example.com`, ni `shop.example.com`, ni aucun autre hôte. Chaque
origine qui traite des données transmises par un agent publie sa propre déclaration. Un
vérificateur NE DOIT PAS appliquer la déclaration d'une origine à une autre.

## Structure

Ceci est un exemple illustratif, pas la déclaration réelle d'une organisation existante. Le bloc
de code ci-dessous reste en anglais, langue de travail du format lui-même - un site francophone
écrirait ses propres valeurs de texte libre (comme `notes`) dans sa propre langue :

```json
{
  "version": "0.2",
  "declared": "2026-01-01T00:00:00Z",
  "expires": "2026-07-01T00:00:00Z",
  "last_reviewed": "2026-01-01T00:00:00Z",
  "retention": {
    "content": "none",
    "identity": ["support-ticket"],
    "notes": "Support tickets are kept for 90 days to resolve disputes. Nothing else is stored."
  },
  "third_parties": [
    { "name": "Example CDN Co", "role": "infrastructure", "purpose": "content delivery",
      "data_seen": ["network_metadata"], "jurisdiction": ["example-region"] }
  ],
  "jurisdiction": {
    "processing": ["example-region"],
    "storage": ["example-region"],
    "backups": ["example-region"]
  },
  "deletion": {
    "handle": "https://example.com/api/agent-delete",
    "method": "POST",
    "auth": "receipt-token",
    "covers": ["identity"],
    "legal_holds": [
      { "covers": "consent records", "reason": "consent must remain provable under applicable privacy law" }
    ]
  },
  "receipt_support": false
}
```

### Classes de données

Des descriptions en texte libre invitent des lectures radicalement différentes d'une même
déclaration; `data_seen`, les clés de `retention` et `deletion.covers` puisent donc dans un
seul vocabulaire contrôlé :

`content` (ce que l'interaction a produit - messages, requêtes, téléversements), `identity`
(l'identité de l'interlocuteur), `authentication` (jetons, identifiants), `network_metadata`
(adresse IP, en-têtes), `device_metadata` (empreinte, agent utilisateur), `financial`,
`health`, `location`, `files`, `derived` (tout ce qui est calculé à partir du reste -
vecteurs, scores, profils).

Une déclaration PEUT ajouter des classes propres au site, mais tout ce qu'elle conserve ou
partage doit relever d'au moins une classe nommée. « Données transmises par un agent » désigne
toute classe ci-dessus que l'interaction d'un agent amène le site à recevoir - y compris les
métadonnées que l'agent n'a pas délibérément envoyées.

### Notes sur les champs

- `declared` / `expires` / `last_reviewed` - une déclaration est une affirmation sur le
  présent, pas sur toujours. `expires` est OBLIGATOIRE à partir de la v0.2 : une déclaration
  expirée est évaluée comme périmée, jamais comme une vérité actuelle, parce qu'un site peut
  changer toute son infrastructure longtemps après avoir publié une déclaration impeccable.
  Douze mois est la fenêtre maximale raisonnable.
- `retention` - ce qui est conservé, divisé entre `content` et `identity`. La plupart des sites
  réels conserveront certaines données liées à l'identité pour des raisons légitimes (soutien,
  prévention de la fraude, obligations légales) même s'ils ne stockent aucun contenu - cette
  distinction récompense une déclaration honnête plutôt qu'un « nous ne conservons rien »
  général qu'une vérification finirait par contredire.
- `third_parties` - chaque entité qui voit des données transmises par un agent, nommée, avec un
  `role` (`processor`, `subprocessor`, `infrastructure`, `analytics`, `security`,
  `model_provider`), ce qu'elle voit, et où elle traite. « Voit » inclut les transferts côté
  serveur qu'aucun observateur externe du réseau ne peut surveiller - un site qui ne stocke
  rien mais transmet tout à un fournisseur de modèle n'a rien divulgué tant que ce fournisseur
  n'est pas nommé. Un tableau vide est une réponse valide et évaluable.
- `jurisdiction` - où les données sont traitées, stockées et sauvegardées, en réponses
  distinctes, parce que ce sont habituellement des endroits différents. Les codes de région
  sont préférables lorsqu'ils sont vérifiables de façon indépendante. La juridiction juridique
  (quel droit s'applique) peut différer de l'emplacement physique; lorsqu'elles diffèrent, dire
  les deux.
- `deletion` - un mécanisme réel et activable par une machine. L'essentiel est qu'il soit
  activable par machine, pas un paragraphe de billet de soutien. À partir de la v0.2, la
  déclaration doit aussi dire comment un appelant prouve son droit de supprimer ce qu'il
  demande de supprimer (`auth`), parce qu'un point de suppression sans authentification est une
  arme pointée vers les propres utilisateurs du site. Exigences sur le point lui-même :
  - il DOIT exiger une preuve liant la demande à l'interaction visée (un jeton de reçu là où
    les reçus existent, ou un secret équivalent propre à l'interaction) - jamais de suppression
    sur une demande non authentifiée;
  - il DOIT être idempotent - la même suppression demandée deux fois n'est pas une erreur;
  - sa réponse de succès DOIT dire ce qui s'est réellement passé : `deleted`, `queued` (avec un
    délai prévu), ou `nothing-held` - un simple HTTP 200 n'est pas une réponse;
  - tout ce qu'il ne peut pas supprimer doit figurer dans `legal_holds`, chaque entrée nommant
    ce qui est conservé et la raison juridique, sous forme lisible par machine. « Nous
    conservons X en raison de la loi Y » passe; le silence échoue.
- `receipt_support` - indique si le site prend en charge les reçus par interaction. `false` est
  une réponse honnête et évaluable, pas un échec en soi. Lorsqu'il est pris en charge, un reçu
  est un petit objet JSON :

  ```json
  {
    "interaction_id": "an-opaque-id",
    "received_classes": ["content", "identity"],
    "retention": { "content": "none", "identity": "30d" },
    "third_parties": ["Example CDN Co"],
    "deletion_token": "an-opaque-single-use-token"
  }
  ```

  **Un reçu NE DOIT PAS reproduire le contenu qu'il décrit.** Il nomme des classes de données
  et leur état de traitement, jamais les mots, fichiers ou valeurs eux-mêmes - un reçu qui
  répète une question de santé est une nouvelle copie de la chose la plus sensible de
  l'interaction.
- `signature` - OPTIONNEL, réservé à partir de la v0.2 : une déclaration peut porter une
  signature détachée la liant à son origine, pour qu'un agent détecte un fichier altéré ou
  rejoué, même au-delà d'un CDN compromis. HTTP Message Signatures (RFC 9421) est le mécanisme
  candidat. Le profil concret (algorithmes, découverte des clés) n'est délibérément pas
  spécifié ici : il ne sera publié qu'après une revue cryptographique indépendante, et d'ici
  là, le transport HTTPS est le modèle de confiance, énoncé clairement comme une limite plutôt
  que déguisé en garantie.

## Règles pour les vérificateurs

Le vérificateur est un client HTTP traitant des entrées qu'un adversaire contrôle, et une
déclaration hostile est la façon évidente de l'attaquer. Tout vérificateur conforme :

- NE DOIT PAS déréférencer `deletion.handle` sans y être invité. La présence et la forme se
  vérifient depuis la déclaration seule; appeler réellement un point de suppression ne se fait
  qu'avec le consentement du site et des données synthétiques, ou pas du tout.
- DOIT refuser toute URL déclarée qui se résout vers une adresse de bouclage, privée, de lien
  local, multidiffusion ou autrement réservée, et DOIT se connecter à l'adresse qu'il a résolue
  et vérifiée, pour qu'une réponse DNS ne puisse pas changer entre la vérification et la
  requête.
- NE DOIT PAS suivre une redirection vers une origine différente de celle qu'il vérifie.
- DOIT imposer des limites strictes de taille de réponse et de délai sur tout ce qu'il
  télécharge.
- NE DOIT PAS joindre d'identifiants, de témoins ou d'autorité ambiante à une requête
  provoquée par une déclaration.
- DOIT traiter chaque octet téléchargé comme non fiable au moment de produire ses rapports -
  un nom de site est du texte contrôlé par un adversaire, jamais du balisage.

Les sites sont tenus à la règle miroir : **un site NE DOIT PAS traiter différemment un trafic
qu'il croit être un test de conformité.** Les tests de rétention utilisent des données
synthétiques aléatoires et non marquées, précisément pour que reconnaître et supprimer le test
soit plus difficile que de simplement bien se comporter; un site pris à le faire est évalué
comme ayant déclaré faux, la pire note du bulletin.

## Questions ouvertes

- Le profil concret de signature (voir `signature` ci-dessus) - défini comme point d'extension,
  délibérément non profilé en attente d'une revue cryptographique indépendante.
- Gestion des versions et migration entre versions de déclaration. Les fichiers v0.1 restent
  lisibles; un vérificateur les évalue selon les exigences plus faibles de la v0.1 et note
  l'absence de `expires`.

## Statut

Ébauche v0.2, publiée pour commentaires en parallèle avec le document principal de la norme.
