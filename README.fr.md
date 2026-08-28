# The Data Dignity Standard™

*[English version](README.md)*

**v0.1 - Ébauche pour commentaires publics**

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

## Le fichier de déclaration

Tout site peut publier un petit fichier JSON à :

```
https://<site>/.well-known/data-dignity.json
```

décrivant ce qu'il conserve, qui d'autre voit les données transmises par un agent, où elles sont
traitées, et comment demander leur suppression. (`.well-known/` est la convention existante,
enregistrée par l'IETF, pour ce type de déclaration au niveau du site - le même modèle que
`security.txt` utilise déjà.) Format complet : `SCHEMA.md` dans ce dépôt.

## Les clauses

Cette première version couvre cinq clauses, soit la moitié de la norme qui concerne le site.
D'autres clauses - portant sur ce qu'un agent IA respectueux de la dignité des données doit
lui-même faire, ainsi que des mécanismes d'application plus stricts - sont prévues pour des
versions ultérieures; elles ne sont pas omises par oubli.

### 1 - Rétention déclarée, vérifiée, jamais présumée sur parole
Le site indique ce qu'il conserve d'une interaction avec un agent, pendant combien de temps, et
pourquoi. Vérifié en soumettant de vraies transactions marquées et en vérifiant leur
réapparition - jamais en lisant simplement la déclaration et en la croyant sur parole.

### 2 - Aucun tiers non divulgué
Tout tiers qui voit des données transmises par un agent est nommé. Le trafic réseau observé
pendant une véritable interaction avec un agent est comparé à la liste déclarée.

### 3 - Juridiction énoncée clairement
L'endroit où les données sont traitées et stockées est nommé dans la déclaration, jamais laissé
à deviner ou à découvrir après coup.

### 4 - Un mécanisme de suppression fonctionnel
Un moyen réel, activable par une machine, de demander la suppression de ce qu'une interaction a
laissé derrière elle - et une honnêteté totale sur tout ce qu'un site est légalement tenu de
conserver, et pourquoi.

### 5 - Le reçu
Par interaction, un site peut retourner un court reçu : ce qui a été reçu, ce qui est conservé
et jusqu'à quand, qui d'autre l'a vu, et le mécanisme de suppression propre à cette interaction
précise. C'est la clause la plus récente et la plus exigeante, celle que très peu de sites
devraient prendre en charge au départ - c'est exactement pourquoi elle existe comme ligne évaluée
à part plutôt que comme exigence.

## Évaluation

Chaque clause est notée, jamais en réussite ou échec - un site avec de réelles lacunes garde tout
de même un échelon à gravir, et une note peut être révisée dans le temps à mesure que le site
évolue. La note composite et les notes par clause sont toutes deux rapportées.

## Statut

Ceci est une ébauche v0.1, publiée pour commentaires. ArrowMem entend publier sa propre
déclaration et s'évaluer selon cette norme avant de le demander à quiconque. Un vérificateur de
référence gratuit et ouvert est prévu peu après la norme elle-même.

**Commentaires et questions :** ouvrez une issue sur ce dépôt, ou écrivez à git@arrowmem.ca

**Licence :** ce document est sous licence [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.fr)
- voir LICENSE. « Data Dignity Standard » est une marque de commerce d'ArrowMem (demande en
instance); la marque de commerce est une question distincte de cette licence et n'est pas
concédée par celle-ci.

L'équipe ArrowMem
