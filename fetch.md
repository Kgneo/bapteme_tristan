Salut Salut !

> Alors comme ça t'en veux encore ? &#129321;<br>
> Très bien ! C'est parti pour le `fetch` !

Lors de l'atelier sur les cartes du jeu <b>Triple Triad</b>, nous avons mis en place un application web avec une architecture assez "classique" : un bon vieux MVC des familles !

Mais cette structure reste un peu limitée, surtout quand on veut créer des pages plus dynamiques, et réduire les temps de chargements.

C'est pour ça qu'on a inventé le `fetch` ! (pas moi, mais des gens super malins !)

La méthode Javascript `fetch` permet d'envoyer des requêtes à un serveur pour récupérer des données, sans avoir besoin de recharger toute la page. (Alléluia ! &#128519;)

En effet, pour l'instant, lorsque l'on cliquait sur une carte par exemple, le navigateur changeait d'URL, et le serveur devait reconstruire l'intégralité de la page, pour ensuite la renvoyer au navigateur, qui se contentait d'afficher le résultat.

Avec `fetch`, on va pouvoir interroger notre serveur, et exécuter les méthodes de nos Controllers, sans avoir à changer d'URL !

L'autre avantage, c'est que l'on peut récupérer soit du HTML, soit des données "brutes" (au format JSON par exemple), que l'on peut ensuite traiter côté Front, et ainsi économiser les ressources du serveurs... &#129312;

> Bien, passons au concret !

Je te propose de reprendre le code de ton atelier, pour aller modifier notre super moteur de recherche !

Jusqu'ici, ça marche, mais il faut sans arrêt cliquer sur le bouton "retour" pour effectuer une nouvelle recherche !

Ce qui serait super pratique, c'est d'afficher le résultat directement au-dessous des formulaires, sans changer de page.

> On pourrait tout à fait le faire en MVC aussi. Mais pour l'exemple, on va le faire avec `fetch`.

Pour cet exemple, nous traiterons uniquement le cas de la recherche par nom.<br>
Mais on pourrait faire la même chose pour les 4 types de recherches.

### Allez GO !!!

Tout d'abord, à la fin de ton fichier `views/search/search.ejs`, juste avant l'inclusion du footer, tu vas rajouter une `div` qui servira à afficher le résultat de notre futur `fetch`.

```
<br><br><br>
<div id="search-results"></div>
```

Ensuite, on va devoir modifier le formulaire de recherche par nom, pour lui demander d'exécuter une fonction au lieu de soumettre le formulaire directement. 

```
//Enlever les paramètres action et method sur cette balise :
<form action="/search/name" method="get">
```

```
//et rajouter le paramètre onsubmit
<form onsubmit="searchByName(event)">
```

Ensuite, rajouter une balise `<script></script>`, pour executer du javascript côté navigateur, et déclarer la fonction searchByName()


Juste après la `div` "search-results" :

```
<script>
  async function searchByName(event){
    //stop le fonctionnement "normal" du formulaire
    event.preventDefault()

    //récupère le champs de texte dans le DOM
    let input = document.getElementById("name")

    //définit l'url à appeler par le fetch
    let url = "/search/name"

    //rajoute le paramètre sur l'url
    url += "?name=" + input.value

    //execute le fetch, et récupère les données dans searchResults
    let searchResults = await fetch(url)

    console.log("input", input)
    console.log("searchResults", searchResults)
    console.log("Réponse HTML", await searchResults.text())
  }
</script>
```

Dans cette fonction : 

`event.preventDefault()` permet de "court-circuiter" la soumission du formulaire. 

Normalement, quand on click sur le bouton `<input type="submit" value="Chercher">`, le navigateur lance automatiquement une requête contenant les valeurs des différents champs du formulaire (ici, il prend la valeur de la balise `<input id="name">`) et envoie ces valeurs à l'url indiquée dans le `action` de la balise `form`.

Avec `event.preventDefault()` on stop l'execution normal du formulaire, et on peut faire nos propres traitement à la place.

Maintenant, lorsqu'on click sur le bouton, on peut visualiser le résultat du `fetch` dans la console du navigateur, grâce au `console.log` placé à la fin de la fonction.

`console.log("searchResults", await searchResults.text())` :


```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Triple Triad Deck-Builder</title>
  [...]
```

Comme tu peux le voir, la réponse du serveur contient le code HTML d'une page entière, avec les balises `<html> <body> <header>` etc... Puis, la liste des cartes obtenues comme résultat de la recherche.

Et c'est normal, puisque notre vue `main/cardList` contient les lignes : 

`<%- include('partials/header') %>`<br>
`<%- include('partials/footer') %>`

Mais ici, on n'a pas besoin de l'intégralité de cette page !

On voudrait seulement la liste des cartes...

Pour ça, on va donc créer une autre vue appelée par exemple : `cardList_partial.ejs`, en faisant un copié/collé de la vue `cardList.ejs`, puis on va tout simplement enlever ces 2 lignes, pour ne garder que le rendu de la liste, et le titre.

```
<h1><%= title %></h1>

<div class="cardsContainer">
  <% if(cards.length){ %>
  <% cards.forEach( (card) => { %>

    <div class="card">
      <a href="/card/<%= card.id %>">
        <img src="/visuals/<%= card.visual_name %>" alt="<%= card.name %> illustration">
        <p class="cardName"><%= card.name %></p>
      </a>
      <a class="link--addCard" title="Ajouter au deck" href="/deck/add/<%= card.id %>">[ + ]</a>
    </div>

  <% }) %>
  <% } else { %>
    Empty
  <% } %>
</div>
```

Et enfin, on va modifier la fonction `searchByName()` de notre vue `search.ejs`, pour afficher le résultat dans notre page.

Dans la fonction `searchByName()`, après le `fetch`, rajouter : 

```
//récupère le contenu HTML de la réponse (attention c'est une Promise ! donc await)
let resHTML = await searchResults.text()

//récupère la div search-results que l'on a rajouté tout à l'heure
let divSearchRes = document.getElementById("search-results")

//injecte le HTML dans la balise (dans le DOM)
divSearchRes.innerHTML = resHTML
```

Ici, on récupère la balise dont l'id est `search-results`, avec `document.getElementById("search-results")`

Puis, on lui injecte le contenu HTML que notre serveur nous a renvoyé via le `fetch`

#### Et voilà ! 

A chaque fois que l'on click sur le bouton `rechercher` du formulaire de recherche par nom, la liste des résultats s'affiche directement en dessous des formulaires !!!

## Pour aller plus loin...

La méthode `fetch` peut bien sûr être configurée de multiple façons, pour effectuer tout type de requête.

On peut notamment configurer les header, le cache, la méthode (GET/POST), etc.

Ex :

```
fetch(url, { method: "GET",
             headers: new Headers() 
           })
```

Pour continuer à approfondir le `fetch`, tu trouveras plus d'explications et d'exemples en consultant les quelques liens suivants : 

https://developer.mozilla.org/fr/docs/Web/API/Fetch_API/Using_Fetch

https://fr.javascript.info/fetch

https://www.youtube.com/watch?v=z9pcgJX1DdY

https://www.pierre-giraud.com/javascript-apprendre-coder-cours/api-fetch/

https://www.google.com/search?q=js+fetch

###  &#128075; Si tu as d'autres questions n'hésites pas !