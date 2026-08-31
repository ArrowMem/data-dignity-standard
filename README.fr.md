# The Data Dignity Standard

*[English version](README.md)*

**v0.5 - Ébauche pour commentaires publics**

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

## D'où vient cette idée

Nous n'avons pas inventé l'expression « data dignity », et nous tenons à le dire clairement.

Le terme vient de **Jaron Lanier et E. Glen Weyl**, qui l'ont exposé en 2018 dans « A Blueprint
for a Better Digital Society » (Harvard Business Review). L'argument qui le sous-tend est plus
ancien encore, et traverse l'ouvrage de Lanier « Who Owns the Future? » (2013) : les données
qu'une personne produit sont sa contribution, et non une ressource gratuite; une économie
numérique qui les traite comme gratuites prend quelque chose sans le dire. Weyl parlait de
« data as labor »; Lanier, d'« humanistic digital economics ». Ils ont retenu « data dignity »
parce que l'expression nommait la chose sans engager de querelle politique, et ce nom a survécu
aux deux autres parce qu'il était le bon.

D'autres ont poussé l'idée plus loin. RadicalxChange, la fondation de Weyl, a bâti une grande
part de l'argumentaire public en sa faveur. Microsoft mène ses propres travaux sur la dignité
des données. Cette idée a été développée par des personnes qui y réfléchissaient des années
avant que les agents IA ne la rendent urgente, et elle leur appartient.

**Ce que ce document ajoute est modeste.** La dignité des données a été, pendant l'essentiel de
son existence, un principe : un énoncé sur ce qui devrait être. Ceci est une tentative d'en
rendre une partie vérifiable par une machine, afin que le traitement des données d'une personne
par un site puisse être testé plutôt qu'affirmé. C'est une contribution d'ingénierie à l'idée
d'autrui, et non une idée nouvelle. Nous sommes reconnaissants que le terrain ait été préparé,
et nous préférons le dire plutôt que de laisser un lecteur supposer que nous en sommes à
l'origine.

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

Cette version couvre six clauses : cinq pour le site, et une première clause pour l'agent
lui-même, parce que la dignité des données est une propriété à deux faces - les sites sont
notés, les agents peuvent être certifiés. Des mécanismes d'application plus stricts sont prévus
pour des versions ultérieures; ils ne sont pas omis par oubli.

### 1 - Rétention déclarée, vérifiée, jamais présumée sur parole
Le site indique ce qu'il conserve d'une interaction avec un agent, par classe de données,
pendant combien de temps, et pourquoi. Vérifié en soumettant de vraies transactions de test
porteuses de données synthétiques aléatoires et en vérifiant leur réapparition - jamais en
lisant simplement la déclaration et en la croyant sur parole. Les tests sont délibérément non
marqués et indiscernables du trafic ordinaire, et un site ne doit pas traiter différemment un
trafic qu'il soupçonne d'être un test : réussir en reconnaissant l'auditeur est évalué comme
avoir déclaré faux. Une limite honnête, énoncée plutôt que cachée : le test de réapparition
vérifie les mots, pas leurs ombres - un site peut supprimer le texte et garder un vecteur, un
score, un profil. C'est pourquoi les données dérivées sont leur propre classe déclarée avec
leur propre entrée de conservation, et pourquoi un test de rejeu propre est une preuve sur le
contenu, jamais sur ce qui en a été calculé.

### 2 - Aucun tiers non divulgué
Toute partie qui voit des données transmises par un agent est nommée - par un identifiant
stable lisible par machine, pas seulement un nom d'affichage - y compris les fournisseurs et
sous-traitants auxquels un site transmet des données côté serveur, là où aucun observateur
externe ne peut surveiller. Le trafic réseau observé pendant une véritable interaction avec un
agent est comparé à la liste déclarée; ce que l'observation du réseau ne peut pas voir est
couvert par la déclaration elle-même, et une partie découverte plus tard absente de la liste
vaut une note de déclaré faux.

### 3 - Juridiction énoncée clairement
L'endroit où les données sont traitées, stockées et sauvegardées est nommé dans la
déclaration - en réponses distinctes, parce que ce sont habituellement des endroits
différents -, jamais laissé à deviner ou à découvrir après coup.

### 4 - Un mécanisme de suppression fonctionnel
Un moyen réel, activable par une machine, de demander la suppression de ce qu'une interaction a
laissé derrière elle - et une honnêteté totale sur tout ce qu'un site est légalement tenu de
conserver, et pourquoi. Le point de suppression vit sur la même origine que la déclaration qui
l'annonce - un chemin de suppression pointant ailleurs est la façon dont un identifiant de
suppression fuit vers un attaquant, et il échoue cette clause d'emblée. Le point doit
authentifier que l'appelant a le droit de supprimer ce qu'il demande, et doit répondre par ce
qui s'est réellement passé, pas par un simple accusé de réception. Les règles du format, y
compris les formes de requête et de réponse, sont dans `SCHEMA.md`.

### 5 - Le reçu
Par interaction, un site peut retourner un court reçu : ce qui a été reçu, ce qui est conservé
et jusqu'à quand, qui d'autre l'a vu, et le mécanisme de suppression propre à cette interaction
précise. Un reçu nomme des classes de données, jamais le contenu lui-même - il ne doit pas
devenir une seconde copie de la chose sensible qu'il décrit. C'est la clause la plus récente et
la plus exigeante, celle que très peu de sites devraient prendre en charge au départ - c'est
exactement pourquoi elle existe comme ligne évaluée à part plutôt que comme exigence.

### 6 - Le registre (côté agent)
Le miroir du reçu, et la première clause qui porte sur l'agent plutôt que sur le site : un
agent respectueux de la dignité des données tient son propre registre - chaque site touché au
nom d'une personne, ce qui a été divulgué, quand, et le mécanisme de suppression de chaque
interaction. Quand un site subit une brèche des années plus tard, une seule requête répond à
« qu'avaient-ils de moi », et la suppression peut être demandée partout où la personne décide
qu'elle doit l'être.

Le registre est la base de données la plus sensible que cette norme crée, alors ses règles ont
du mordant :

- Il consigne des classes et des métadonnées - site, origine, horodatage, identifiant
  d'interaction, classes divulguées - **jamais le contenu divulgué lui-même**, la même règle
  de non-reproduction que suivent les reçus.
- **Les enregistrements d'audit et les capacités vivantes sont rangés séparément.** Les jetons
  de suppression et tout autre identifiant vivent dans un coffre de capacités protégé, chiffré
  au repos, jamais dans un export en clair de la piste d'audit - pour qu'un export de registre
  qui fuit soit un historique, pas une arme.
- Le registre appartient à la personne : exportable par elle, conservé et supprimé sur sa
  décision, jamais une archive cachée que l'agent garde pour lui-même.
- **La suppression groupée n'est jamais autonome.** Détenir un jeton est un moyen, pas une
  autorisation : toute exécution de suppression multi-sites exige l'accord explicite et
  éclairé de la personne - prévisualiser ce qui sera demandé à quelles origines et pour
  quelles classes, obtenir l'autorisation, exécuter, rapporter les résultats par origine. Un
  agent ne doit jamais déduire une permission du fait que le registre contient un jeton, quoi
  qu'en dise une consigne, un module ou une automatisation - et **l'autorisation elle-même
  doit être un acte que la personne accomplit hors du contexte du modèle** (une confirmation
  au niveau du système d'exploitation, un objet d'approbation signé), parce qu'une approbation
  qui n'est que du texte dans une invite est du texte qu'une injection de consigne peut
  fabriquer.
- Les clés du coffre de capacités sont protégées aussi fortement que la plateforme le
  permet - trousseau du système d'exploitation ou support matériel lorsque disponible, jamais
  détenues en clair par l'agent plus longtemps que l'opération ne l'exige.
- **Une interaction est consignée au registre, que le site ait émis un reçu ou non.** Les
  reçus réconcilient le registre; ils ne le définissent pas - un site qui n'en émet aucun
  apparaît quand même, son entrée marquée comme sans reçu, pour que le silence du côté du site
  ne devienne jamais un angle mort du côté de la personne.

La certification vérifie exactement ce qu'elle nomme, de façon adversariale : l'agent produit
un registre complet et exportable pour une session sans rien laisser tomber silencieusement,
ses entrées se réconcilient avec les reçus des sites qui en émettent (et les interactions sans
reçu apparaissent quand même), les capacités restent hors de l'export d'audit et survivent à
une tentative de fuite par export, et la suppression groupée refuse de s'exécuter sur une
approbation fabriquée dans le contexte du modèle - la certification comprend des tests
d'injection de consignes et de fuite de capacités, pas seulement le chemin idéal. Les sites
sont notés sur les clauses 1 à 5; les agents sont certifiables sur celle-ci.

## Évaluation

Chaque clause est notée, jamais en réussite ou échec - un site avec de réelles lacunes garde tout
de même un échelon à gravir, et une note peut être révisée dans le temps à mesure que le site
évolue. La note composite et les notes par clause sont toutes deux rapportées, chacune avec son
niveau de preuve (déclarée, observée ou vérifiée).

La règle de notation est déterministe de la seule manière qui résiste aux adversaires : sous
chaque note se trouve un ensemble de faits consignés - chaque résultat de test individuel est
exactement PASS, FAIL, UNKNOWN ou NOT_TESTED - et les lettres sont calculées à partir de ces
faits par la règle publiée. Le jugement peut décider quoi tester; il ne décide jamais ce que
dit un fait consigné. Deux vérificateurs disposant des mêmes faits arrivent à la même note, et
une preuve incomplète se manifeste comme des faits UNKNOWN, jamais comme des lettres
optimistes. Les lettres ci-dessous sont la correspondance provisoire de la v0.5, publiée pour
commentaires comme tout le reste ici :

- **A** - déclarée et vérifiée de façon indépendante.
- **B** - déclarée, comportement observé cohérent, rien ne la contredit.
- **D** - non déclarée. Basse, mais au-dessus du faux, à dessein.
- **F** - déclaration contredite par la preuve. Un F sur une clause plafonne aussi la note
  composite à F : **un site pris dans une seule fausse affirmation ne peut pas dépasser un site
  qui s'est tu.**
- Une déclaration expirée (voir `expires` dans `SCHEMA.md`) plafonne chaque clause qu'elle
  couvre à D - le niveau non déclaré - tant qu'elle n'est pas rééditée.
- Une déclaration dans un format remplacé est rapportée **LEGACY** - analysée et affichée,
  jamais notée sur l'échelle actuelle. Noter un ancien format selon ses propres règles plus
  faibles serait une invitation permanente à ne jamais migrer; la section sur la sécurité des
  versions de `SCHEMA.md` fait règle.

La note composite est la plus basse entre : la moyenne des notes par clause (arrondie vers le
bas), et tout plafond ci-dessus. Mêmes intrants, même note, peu importe qui exécute la
vérification.

**Une note Data Dignity est la preuve d'une conformité aux tests définis. Ce n'est pas la
preuve qu'un site ne contient aucun traitement non divulgué** - aucun test externe ne peut
prouver une négation sur l'infrastructure d'autrui, et une note qui prétendrait le contraire
serait exactement le genre de faux réconfort que cette norme existe pour faire cesser.

## Limites, énoncées clairement

Tant que l'extension de signature réservée n'a pas de profil concret revu, le modèle de
confiance du fichier de déclaration est HTTPS plus le contrôle de l'origine - une origine
compromise ou un CDN mal configuré peut servir une fausse déclaration, et un agent n'a encore
aucun moyen cryptographique de le détecter. Un comportement purement côté serveur peut être
déclaré et noté, mais jamais vérifié de l'extérieur à lui seul - ces affirmations restent au
niveau de preuve « déclarée » jusqu'à ce que des mécanismes plus forts existent. Et le test de
réapparition borne ce qu'un site a gardé des mots, pas ce qu'il en a calculé. Chacune de ces
limites est portée dans les notes elles-mêmes plutôt que maquillée : c'est à cela que servent
les étiquettes de preuve.

## Statut

Ceci est une ébauche v0.5, publiée pour commentaires - et gelée : les clauses, la
grammaire et les invariants sont stables pour cette période de commentaires. Les
commentaires sont triés, jamais intégrés à leur arrivée; les changements arrivent groupés
dans la version suivante, et seuls un exploit démontré, une ambiguïté rapportée par un
implémenteur, ou la revue cryptographique indépendante rouvrent l'ébauche plus tôt. ArrowMem entend publier sa propre
déclaration et s'évaluer selon cette norme avant de le demander à quiconque. Un vérificateur de
référence gratuit et ouvert est prévu peu après la norme elle-même, construit selon les règles
pour vérificateurs de `SCHEMA.md`.

**Commentaires et questions :** ouvrez une issue sur ce dépôt, ou écrivez à git@arrowmem.ca

**Licence :** ce document est sous licence [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.fr)
- voir LICENSE.

L'équipe ArrowMem
