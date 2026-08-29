# The Data Dignity Standard™

*[English version](README.md)*

**v0.2 - Ébauche pour commentaires publics**

## Pourquoi ceci existe

Les agents IA deviennent une nouvelle catégorie de visiteurs sur le Web : ils naviguent,
comparent et achètent au nom d'une personne. Les outils qui rendent un site *utilisable* par un
agent se développent rapidement : données structurées, actions activables par un agent, normes
de découverte. L'Agent Readiness Score gratuit de Cloudflare, par exemple, répond déjà à la
question « un agent peut-il utiliser ce site? ».

Nous n'avons trouvé aucune autre norme publique qui répond à une question différente :
**devrait-il le faire?** Lorsqu'un site transmet des données à un agent IA agissant pour une
personne réelle - ou en reçoit de lui -, qu'advient-il de ces données? Sont-elles conservées?
Partagées? Où vont-elles? Aujourd'hui, rien ne permet à un agent, ni à la personne pour qui il
agit, de le savoir.

Data Dignity Standard est une réponse modeste et vérifiable à cette question. Ce n'est pas une
certification que l'on achète, ni une barrière que quiconque doit franchir : c'est un format de
déclaration public et un moyen gratuit de la vérifier, afin que les agents (et les personnes qui
les utilisent) puissent savoir quels sites traitent leurs données avec dignité. L'évaluation est
gratuite et ouverte à tous; un service payant distinct et optionnel de suivi continu peut être
offert aux organisations qui souhaitent que leur évaluation soit suivie dans le temps, mais rien
dans le respect de la norme elle-même n'exige de payer qui que ce soit.

## Principes

- **Vérifiable, pas aspirationnel.** Chaque clause ci-dessous est quelque chose qu'une machine
  peut vérifier. Une norme que personne ne peut vérifier n'est qu'un slogan.
- **Évaluation, pas contrôle d'accès.** Les sites sont notés, jamais bloqués. C'est un bulletin,
  jamais un péage.
- **Déclaré et vrai vaut mieux que non déclaré. Déclaré et faux est pire que les deux.** Le seul
  véritable mordant de la norme : un site qui affirme quelque chose de faux sur le traitement de
  ses propres données obtient une note pire qu'un site qui ne dit rien du tout.
- **Dire ce que vaut la preuve, rien de plus.** Chaque affirmation notée porte l'étiquette de la
  façon dont elle est connue : **déclarée** (l'organisation l'affirme), **observée** (un
  vérificateur a vu un comportement cohérent avec elle), **vérifiée** (un vérificateur l'a
  reproduite de façon indépendante), ou **inconnue** (impossible à établir de l'extérieur). Une
  note ne présente jamais une déclaration comme une vérification.

## Le fichier de déclaration

Tout site peut publier un petit fichier JSON à :

```
https://<site>/.well-known/data-dignity.json
```

décrivant ce qu'il conserve, qui d'autre voit les données transmises par un agent, où elles sont
traitées, et comment demander leur suppression. Le préfixe `/.well-known/` est l'emplacement
normalisé par l'IETF pour ce type de déclaration au niveau de l'origine (RFC 8615) - le même
modèle que `security.txt` utilise déjà. Data Dignity définit `data-dignity.json` dans cet
espace. Une déclaration ne parle que pour l'origine exacte qui la sert - les sous-domaines
publient la leur. Format complet, y compris les règles qu'un vérificateur conforme doit
suivre : `SCHEMA.md` dans ce dépôt.

## Les clauses

Cette version couvre cinq clauses, soit la moitié de la norme qui concerne le site. D'autres
clauses - portant sur ce qu'un agent IA respectueux de la dignité des données doit lui-même
faire, ainsi que des mécanismes d'application plus stricts - sont prévues pour des versions
ultérieures; elles ne sont pas omises par oubli.

### 1 - Rétention déclarée, vérifiée, jamais présumée sur parole
Le site indique ce qu'il conserve d'une interaction avec un agent, pendant combien de temps, et
pourquoi. Vérifié en soumettant de vraies transactions de test porteuses de données synthétiques
aléatoires et en vérifiant leur réapparition - jamais en lisant simplement la déclaration et en
la croyant sur parole. Les tests sont délibérément non marqués et indiscernables du trafic
ordinaire, et un site ne doit pas traiter différemment un trafic qu'il soupçonne d'être un
test : réussir en reconnaissant l'auditeur est évalué comme avoir déclaré faux.

### 2 - Aucun tiers non divulgué
Toute partie qui voit des données transmises par un agent est nommée - y compris les
fournisseurs et sous-traitants auxquels un site transmet des données côté serveur, là où aucun
observateur externe ne peut surveiller. Le trafic réseau observé pendant une véritable
interaction avec un agent est comparé à la liste déclarée; ce que l'observation du réseau ne
peut pas voir est couvert par la déclaration elle-même, et une partie découverte plus tard
absente de la liste vaut une note de déclaré faux.

### 3 - Juridiction énoncée clairement
L'endroit où les données sont traitées, stockées et sauvegardées est nommé dans la
déclaration - en réponses distinctes, parce que ce sont habituellement des endroits
différents -, jamais laissé à deviner ou à découvrir après coup.

### 4 - Un mécanisme de suppression fonctionnel
Un moyen réel, activable par une machine, de demander la suppression de ce qu'une interaction a
laissé derrière elle - et une honnêteté totale sur tout ce qu'un site est légalement tenu de
conserver, et pourquoi. Le point de suppression doit authentifier que l'appelant a le droit de
supprimer ce qu'il demande, et doit répondre par ce qui s'est réellement passé, pas par un
simple accusé de réception. Les règles du format à ce sujet sont dans `SCHEMA.md`.

### 5 - Le reçu
Par interaction, un site peut retourner un court reçu : ce qui a été reçu, ce qui est conservé
et jusqu'à quand, qui d'autre l'a vu, et le mécanisme de suppression propre à cette interaction
précise. Un reçu nomme des classes de données, jamais le contenu lui-même - il ne doit pas
devenir une seconde copie de la chose sensible qu'il décrit. C'est la clause la plus récente et
la plus exigeante, celle que très peu de sites devraient prendre en charge au départ - c'est
exactement pourquoi elle existe comme ligne évaluée à part plutôt que comme exigence.

## Évaluation

Chaque clause est notée, jamais en réussite ou échec - un site avec de réelles lacunes garde tout
de même un échelon à gravir, et une note peut être révisée dans le temps à mesure que le site
évolue. La note composite et les notes par clause sont toutes deux rapportées, chacune avec son
niveau de preuve (déclarée, observée ou vérifiée).

La règle de notation est déterministe, pour que deux vérificateurs indépendants arrivent à la
même note. Les lettres ci-dessous sont la correspondance provisoire de la v0.2, publiée pour
commentaires comme tout le reste ici :

- **A** - déclarée et vérifiée de façon indépendante.
- **B** - déclarée, comportement observé cohérent, rien ne la contredit.
- **D** - non déclarée. Basse, mais au-dessus du faux, à dessein.
- **F** - déclaration contredite par la preuve. Un F sur une clause plafonne aussi la note
  composite à F : **un site pris dans une seule fausse affirmation ne peut pas dépasser un site
  qui s'est tu.**
- Une déclaration expirée (voir `expires` dans `SCHEMA.md`) plafonne chaque clause qu'elle
  couvre à D - le niveau non déclaré - tant qu'elle n'est pas rééditée.

La note composite est la plus basse entre : la moyenne des notes par clause (arrondie vers le
bas), et tout plafond ci-dessus. Mêmes intrants, même note, peu importe qui exécute la
vérification.

**Une note Data Dignity est la preuve d'une conformité aux tests définis. Ce n'est pas la
preuve qu'un site ne contient aucun traitement non divulgué** - aucun test externe ne peut
prouver une négation sur l'infrastructure d'autrui, et une note qui prétendrait le contraire
serait exactement le genre de faux réconfort que cette norme existe pour faire cesser.

## Statut

Ceci est une ébauche v0.2, publiée pour commentaires. ArrowMem entend publier sa propre
déclaration et s'évaluer selon cette norme avant de le demander à quiconque. Un vérificateur de
référence gratuit et ouvert est prévu peu après la norme elle-même, construit selon les règles
pour vérificateurs de `SCHEMA.md`.

**Commentaires et questions :** ouvrez une issue sur ce dépôt, ou écrivez à git@arrowmem.ca

**Licence :** ce document est sous licence [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.fr)
- voir LICENSE. « Data Dignity Standard » est une marque de commerce d'ArrowMem (demande en
instance); la marque de commerce est une question distincte de cette licence et n'est pas
concédée par celle-ci.

L'équipe ArrowMem
