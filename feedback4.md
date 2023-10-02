Salut Apprenant4 !

Bon, apparement tu es resté bloqué dès le début &#128553;

Je vais te donner les premières étapes qui t'ont manquées pour pouvoir avancer, mais je t'invite à bien regarder la correction pour la suite.

## Étape 1 : Détail d'une carte

> Première étape importante : pour aller sur la page d'une carte, il faut indiquer l'url de cette page dans le paramètre <b>href</b> qui englobe la carte.

Dans la vue `cardList.ejs` , il faut donc modifier la ligne 9 et remplacer le `#` par  `/card/<%= card.id %>`

```
//avant
<a href="#">
```
```
//après
<a href="/card/<%= card.id %>">
```

Ici, l'expression `<%= card.id %>` sera remplacée par la valeur <b>id</b> de la <b> carte</b>.

A l'affichage, dans le HTML de notre page, on aura donc par exemple : 

```
<a href="/card/1">
```

Ensuite, quand on clique sur ce lien, que se passe-t-il ?<br>
Ton navigateur envoie une requête vers ton serveur, en lui donnant l'url indiqué dans le href.

Pour que le serveur soit capable de traiter cette requête, il faut définir la _route_ qui lui correspond.<br>
Une route sert à définir le Controller et la méthode à executer, en fonction de l'URL de la requête reçue.

Il faut donc vérifier si notre _router_ a bien été informé de l'existance d'une route qui correspond à `/card/1`

Dans ton router, on trouve effectivement une route correspondant à `/card`, et qui renvoie vers `mainController.cardPage`.

Malheureusement, ce n'est pas suffisant : il faut aussi préciser que cette route a besoin d'un paramètre pour fonctionner.

En effet, si on veut afficher une seule carte, il faut la récupérer dans la base de donnée, et pour celà, on a besoin de son identifiant.

Pour indiquer le paramètre <b>id</b> dans notre route, il faut donc rajouter <b>/:id</b>, ce qui nous donne :

```
router.get('/card/:id', mainController.cardPage);
```

Maintenant, à chaque fois que le serveur recevra une requête ressemblant à `/card/:id`, il executera la méthode `cardPage()` du `mainController`.

Exemples d'url correctes : 

`/card/1`<br>
`/card/2`<br>
`/card/65789`<br>
`/card/ajviorepxngvuht`<br>

Voyons maintenant le code de ta méthode `cardPage()` :

```
cardPage: async (req, res) =>{
    try {
      const card = await dataMapper.getCard();
      res.render('card',{
      
      });
```

Comme on l'a dit précédemment, pour récupérer une carte, on a besoin de son <b>id</b>.<br>
On a d'ailleurs prit soin de l'indiquer dans les paramètres de la route, il faut maintenant le récupéré côté Controller.

Pour ça, il faut utiliser le paramètres `params` de l'objet `req`, passé en paramètre de ta fonction : 
```
req.params.id
```

Pour faire ça bien, on va avoir besoin de s'assurer que la valeur de <b>id</b> est bien compatible avec le type d'un identifiant, c'est à dire un nombre (Integer)

Avant d'utiliser cette valeur, on va donc la convertir à l'aide de la fonction `parseInt`, qui renvera `null` si on lui demande de convertir une valeur incorrect (par exemple : `ajviorepxngvuht` ne peut pas être traduit en nombre)

En effet, dans une URL, tout est "chaîne de caractère".<br>
Le router ne fait pas la différence entre un "1" et un "izoaijzae".<br>
Par sécurité, on préfère donc convertir cette valeur, pour éviter les injections de code dans la requête SQL qui sera executé par le `dataMapper`.

> Correction de ta méthode mainController.cardPage() : 

```
cardPage: async (req, res) =>{
  const cardId = parseInt(req.params.id, 10)

  try {
    const card = await dataMapper.getCard(cardId);

    if (card) { //équivalent à if (card !== undefined)
      //le paramètre card contient bien des infos, 
      //on les passe à la vue pour affichage
      res.render('card', {card});
  } else {
      //pas d'erreur SQL mais on n'a récupéré aucun 
      //enregistrement, on le signale au navigateur
      res.status(404).send(`Card with id ${cardId} not found`);
  }
  } catch (error) {
    console.error(error);
    res.status(500).render('error');
  }
}
```

Une fois que l'on a appelé proprement `dataMapper.getCard()`, il faut s'assurer que la requête SQL est correcte elle aussi.

Dans ta fonction `getCard()`, tu a oublié d'injecter la valeur de <b>id</b> dans ta requête.

```
getCard: async (id) => {
  //const targetId = Number(request.params.id);
  const queryResult = await client.query(
    `SELECT * 
    FROM card 
    WHERE card.id = $1;
  `);
  if (resultQuery.rowCount === 1) {
    return queryResult.rows[0] 
  } 
  return null;
}
```

Pour que <b>$1</b> soit remplacé par la valeur de l'id, il faut fournir la valeur à remplacer.

Pour ça, au lieu de passer un `string` il faut passer un `objet` contenant 2 paramètres : 

- `text` (string) pour la requête SQL<br>
- `values` (array) pour les valeurs à remplacer


> Correction de la méthode dataMapper.getCard() : 

```
getCard: async function (cardId) {
  const query = {
    text: `SELECT * FROM "card" WHERE "id"=$1`,
    values: [cardId]
  };
  const results = await database.query(query);
  return results.rows[0];
},
```
À noter aussi que l'on écrit pas : `'WHERE card.id = $1'`, mais : `'WHERE "id"=$1'`

Le `FROM "card"` permet déjà d'indiquer à la requête que l'on effectue une recherche dans la table <b>card</b>.<br>
Il n'est donc pas nécessaire de le repréciser dans le `WHERE`.

Si tu as du mal avec SQL, je viens de trouver ce cours assez complet et plutôt clair, qui pourra t'aider à mieux comprendre comment se construisent les requêtes, et survoler toutes les possibilités de ce language.

https://aymeric-auberton.fr/academie/mysql/as-alias


## &#10060; Étape 2 : Recherche (non-traitée)
## &#10060; Étape 3 : Construire un deck (non-traitée)

### &#129299; Besoin d'aide pour éclaircir certains points ?

Au vu de tes résultats sur cet atelier, tu as peut-être des difficultés à comprendre le fonctionnement général d'une architecture MVC. Ou bien des difficultés avec le langage JS ?

En tout cas, si tu as des questions précises à poser sur lesquelles je peux t'aider à avancer, n'hésites pas.<br>
Il suffit parfois d'un petit déclic pour tout piger !