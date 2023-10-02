Salut Apprenant3 !


## Étape 1 : Détail d'une carte

&#9989; Ok, la page de la carte existe bien.

&#10060; Mais quand on se rend sur la page d'une carte, on obtient cette erreur, dans le fichier `views/main/cardDetails.js` : 
```
>> 8| <% for (const singleCard of card) { %>
card is not iterable
```

Cette erreur signifie qu'on ne peut pas faire une boucle sur la variable <b>card</b>.
<br>
Pourquoi ? Parce que <b>card</b> est un objet unique, et non pas une liste d'objets (array)

En effet, dans le `mainController`, dans la fonction `cardPage()`, tu récupères les données d'une <b>carte unique</b>, à l'aide de `dataMapper.getCard()`

```
//d'ailleurs, tu as appelé cette variable "oneCard"
const oneCard = await dataMapper.getCard(id);
```

Et cette fonction `getCard()` renvoie le premier élément des résultats de sa requête SQL :

```
return queryResult.rows[0];
```

Donc, <b>oneCard</b> n'est pas une liste, mais un objet, qui ne peut pas être parcourus par une boucle <% for %>

Pour corriger cette erreur, il suffit donc d'enlever la boucle, et d'utiliser directement la valeur de <b>card</b> !

Enfin... Presque...<br>
Parce que si tu veux pouvoir utiliser une variable nommée <b>"card"</b> dans la vue, il faut que le nom de la variable envoyé par le Controller soit identique.

Autrement dit, soit tu changes le nom de la variable dans le controller (remplacer <b>oneCard</b> par <b>card</b>)

Soit tu changes le nom de la variable dans la vue (remplacer <b>card</b> par <b>oneCard</b>)

En réalisant l'une ou l'autre de ces corrections, tu devrais parvenir à afficher la page sans erreur, et terminer la mise en forme de ta carte.


## Étape 2 : Recherche

> Ok, tu n'as pas pu réussit à très loin sur la recherche. Voyons ce qui cloche !

### &#10060; Recherche par élément

&#11093; Peu importe l'élément sélectionné, on obtient toujours la même erreur :<br>
`"Oups, nous rencontrons un problème technique"`

Dans le searchController, la méthode searchElement() a été habilement muni d'un try/catch pour éviter de faire crasher l'appli !

Dans le catch, on a même la chance d'avoir un console.log(error), pour visualiser l'erreur qui vient de se produire.

Dans le terminal, on peut donc lire l'erreur suivante : <br>
`ReferenceError: dataMapper is not defined`

En effet ! Pour que le dataMapper soit accessible et utilisable depuis ton controller, il faut penser à l'importer en début de fichier avec une ligne qui ressemble à ça : 

```
const dataMapper = require('../dataMapper')
```

&#11093; A partir de là, j'obtiens une autre erreur dans le terminal : <br>
`TypeError: dataMapper.cardElement is not a function`

En effet ! Dans le dataMapper, aucune fonction cardElement() n'est déclaré.
Je présume que tu as dû confondre avec la fonction que tu as appelé `getCardByElement()`

Ici aussi donc, tu as le choix entre 2 solutions : 
> remplacer le nom de getCardByElement() par cardElement() dans
 `dataMapper.js`

> ou, remplacer `dataMapper.cardElement(element)` par `dataMapper.getCardByElement(element)` dans `searchController.js`

&#11093; Parsambleu ! Une erreur sauvage apparaît à nouveau !<br>
Vite ! Attrape ta Pokerrorball pour l'attraper !

&#128561; `Error: Failed to lookup view "main/element/" in views directory "app/views"` &#128561; 

Qu'est-ce donc encore cette diablerie ?!

Pas de panique ! Tu as seulement tapé un slash de trop dans le chemin menant jusqu'à la vue attendue !<br>
A la fin de "main/element/", dans `searchController.js`

```
 l.16| response.render('main/element/', {element});
```
Ici, le dernier / empêche le script de trouver le template element.ejs.
C'est comme si tu lui donnais un chemin, mais sans le nom du fichier.
Quand tu rajoutes le dernier slash, "element" est interprété comme un répertoire.


&#11093; Ok, on continue ! 

Maintenant, la vue `main/element.ejs` est bien trouvée, mais elle contient aussi une erreur.

```
 9|     <% for (const elements of element) { %>
10|       <option value="null"><%=element.element%></option><br>
11|     <% } %><br>

element is not iterable
```

Cependant, ce n'est pas cette vue qui devrait être appelée par le `searchController` ! 

Quand tu appelles la méthode `searchElement()` du `searchController`, tu es en train de faire une recherche de carte par rapport à leur élément. 

Avec la ligne suivante, tu récupères la liste des cartes qui correspondent à l'élément que tu recherches : 
```
13| const result = await dataMapper.getCardByElement(element);
```

Ensuite, tu dois donc envoyer ces résultats à la vue qui affiche une liste de carte (cardList.ejs par exemple)

Il faut donc modifier la ligne 16 du controller, pour demander l'affichage de cette vue : 


```
16| response.render('main/cardList', {element});
```
&#11093; Autre erreur : ce n'est pas l'élément que l'on transmet à la vue, mais la liste de résultats ! Donc :

```
16| response.render('main/cardList', {result});
```

&#11093; Mais ça ne suffit pas ! Car la vue `cardList.ejs` a besoin d'un paramètre <b>title</b> : 


```
4|     <%= title %>
5|   </h1>
6| 
7|   <div class="cardsContainer">

title is not defined
```

&#11093; Et idem pour le paramètre <b>cards</b>, qui doit contenir la liste des cartes à afficher.

```
8|     <% cards.forEach( (card)=> { %>
9| 
10|       <div class="card">
11|         <a href="/card/<%= card.id %>"><%= card.name %></a>

cards is not defined
```

Pour que ces paramètres soient correctement transmis à la vue, il faut que le nom des variables déclarées dans le Controller soit identique à celui utilisé dans la vue.

Pour l'instant, dans le Controller, la liste de cartes issue de ta recherche est stockée dans une variable appelée <b>result</b>. Mais la vue attend de recevoir cette liste dans <b>cards</b>. D'où l'erreur `cards is not defined`.

Pas de problème ! On change tout ça ce qui donne : 


```
const cards = await dataMapper.getCardByElement(element);

if (cards){
  const title = 'Recherche par élément'
  response.render('main/cardList', {cards, title});
}else{
  next();
}
```

> Bien ! On a maintenant une liste de résultat qui s'affiche ! :D <br>
> Mais on a encore un problème...

&#11093; La liste s'affiche, mais elle est incorrecte.
La recherche par élément ne fonctionne pas.

Voyons ensmble la requête que tu as écrites pour filtrer par élément : 

```
getCardByElement: async () => {
  
    const queryResult = await database.query(
      `
      SELECT *
      FROM "card"
      WHERE "element" IS NOT NULL;
      `,
    );
      return queryResult.rows;
    }
   
  };
```

Si je traduis cette requête, ça signifie : <br>
`Sélectionner toutes les cartes dont l'élément n'est pas nul`.

Il manque donc une condition, pour vérifier si l'élément de la carte correspond bien à l'élément choisie pour la recherche.

Et il faut aussi faire attention au cas particulier de la recherche pour "aucun" élément : 


```
getCardsByElement: async function (element) {
    let query;
    //le piège : si l'élément n'est pas renseigné en BDD, il vaut NULL. 
    //Pour effectuer la requête, on utilise les mots-clé IS NULL
    if (element === 'null') {
      query = {
        text: `SELECT * FROM "card" WHERE "element" IS NULL`
      };

    } else {

      //sinon on fait la requête de façon classique
      query = {
        text: `SELECT * FROM "card" WHERE "element"=$1`,
        values: [element]
      };

    }

    const results = await database.query(query);
    return results.rows;
  },
```

Attention aussi à bien passer le paramètre <b>element</b>, dans la déclaration de la fonction : 

`getCardsByElement: async function (element) {`

Pour pouvoir utiliser cette valeur dans la requête SQL : 
```
query = {
  text: `SELECT * FROM "card" WHERE "element"=$1`,
  values: [element]
};
```

Ici, le $1 sera remplacé par la valeur du premier élément passé dans le tableau <b>values</b> : `values: [element]`

&#11093; Bien, on avance ! Mais il y a encore un petit soucis : la liste de résultat est vide !!!

Cette fois, le problème vient du `searchController` :

Dans la méthode `seachElement()`, tu essaies de récupérer la valeur de <b>element</b> transmis en paramètre de la requête : 

``` 
const element = request.params.element
```

Mais cette requête est effectuée avec la méthode GET : `/search/element?element=feu`
<br>
Or, les paramètres d'une méthode GET se récupère dans `request.query` et non dans `request.params` .

`request.params` est utilisé pour la méthode POST.

``` 
const element = request.query.element
```

> On y est enfin ! En effectuant toutes ces corrections, on obtient bien une liste de cartes, filtrée en fonction de l'élément choisi par l'utilisateur sur la page de recherche.

### &#10060; Recherche par niveau
### &#10060; Recherche par valeur 
### &#10060; Recherche par nom 

> Pour les autres méthodes de recherche, je t'invite à bien regarder le code fournit dans la correction, pour retracer les différentes étapes, de chaque type de recherche. C'est toujours plus ou moins le même circuit...

## Étape 3 : Construire un deck

> &#10060; Je ne trouve pas de trace de code permettant de construire un deck !

Tu as bien rajouté un lien sur les cartes, avec un [+] rouge. Mais ce lien renvoie directement vers la page de la carte, au lieu d'appeler la route qui permet de l'ajouter au deck.

``` 
<a class="link--addCard" title="Ajouter au deck" href="/card/<%= card.id %>"><%= card.name %>[ + ]</a>
``` 

Il faudrait donc commencer par rajouter une route dans ton router, puis coder un deckController, et enfin la/les vues nécessaires pour afficher les résultats : 

```
//ajout d'une carte au deck
router.get('/deck/add/:id', deckController.addCard);

//affichage du deck
router.get('/deck', deckController.showDeck);

//supression d'une carte dans le deck
router.get('/deck/remove/:id', deckController.removeCard);
```

Là aussi, regarde bien la correction fournie si tu veux approfondir la notion de Session que l'on demandait d'utiliser pour construire le deck.


### &#10060;  3.1 Activer les sessions
### &#10060; 3.2 Ajouter une carte au deck
### &#10060;  3.3 Une page pour visualiser le deck !
### &#10060; 3.4 Supprimer une carte du deck
