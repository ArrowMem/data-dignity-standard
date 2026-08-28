# Le fichier de déclaration - format v0.1

*[English version](SCHEMA.md)*

Définit la grammaire que chaque clause de la norme vérifie. Volontairement ennuyeux : la leçon
de llms.txt est qu'un simple fichier JSON à un chemin fixe et facile à deviner l'emporte sur
toute solution ingénieuse.

## Emplacement

```
https://<site>/.well-known/data-dignity.json
```

`.well-known/` est la convention existante, enregistrée par l'IETF (RFC 8615), pour exactement
ce type de déclaration au niveau du site - `security.txt` s'y trouve, tout comme le modèle que
ceci reprend. Réutiliser une convention existante plutôt que d'en inventer une nouvelle est
délibéré : un agent (ou un vérificateur) sait déjà où regarder.

## Structure

Ceci est un exemple illustratif, pas la déclaration réelle d'une organisation existante. Le bloc
de code ci-dessous reste en anglais, langue de travail du format lui-même - un site francophone
écrirait ses propres valeurs de texte libre (comme `notes`) dans sa propre langue :

```json
{
  "version": "0.1",
  "declared": "2026-01-01T00:00:00Z",
  "retention": {
    "content": "none",
    "identity": ["support-ticket"],
    "notes": "Support tickets are kept for 90 days to resolve disputes. Nothing else is stored."
  },
  "third_parties": [
    { "name": "Example CDN Co", "purpose": "content delivery", "data_seen": ["ip"] }
  ],
  "jurisdiction": ["example-region"],
  "deletion": {
    "handle": "https://example.com/api/agent-delete",
    "method": "POST",
    "covers": "the identity records named above; content is never stored to begin with"
  },
  "receipt_support": false
}
```

### Notes sur les champs

- `retention` - ce qui est conservé, divisé entre `content` (ce qu'une interaction a produit) et
  `identity` (tout ce qui est conservé sur l'identité de l'interlocuteur, et pourquoi). La
  plupart des sites réels conserveront certaines données liées à l'identité pour des raisons
  légitimes (soutien, prévention de la fraude, obligations légales) même s'ils ne stockent aucun
  contenu - cette distinction récompense une déclaration honnête plutôt qu'un « nous ne
  conservons rien » général qu'une vérification finirait par contredire.
- `third_parties` - chaque tiers qui voit des données transmises par un agent, nommé, avec ce
  qu'il voit. Un tableau vide est une réponse valide et évaluable.
- `jurisdiction` - où les données sont traitées et stockées, en chaînes de texte simples; les
  codes de région sont préférables lorsqu'ils sont vérifiables de façon indépendante.
- `deletion` - un mécanisme réel et activable par une machine. L'essentiel est qu'il soit
  activable par machine, pas un paragraphe de billet de soutien.
- `receipt_support` - indique si le site prend en charge les reçus par interaction. `false` est
  une réponse honnête et évaluable, pas un échec en soi - c'est la partie la plus récente et la
  plus exigeante de la norme, et peu de sites devraient la prendre en charge dès le départ.

## Questions ouvertes

- Signature et intégrité (le fichier devrait-il être signé pour ne pas pouvoir être altéré en
  transit?) Soulevée, non conçue - la v0.1 suppose qu'un simple transport HTTPS suffit pour une
  première version, le même modèle de confiance qu'utilisent la plupart des fichiers de
  déclaration similaires.
- Gestion des versions et migration une fois qu'une v0.2 existera. Non nécessaire tant qu'elle
  n'existe pas.

## Statut

Ébauche v0.1, publiée pour commentaires en parallèle avec le document principal de la norme.
