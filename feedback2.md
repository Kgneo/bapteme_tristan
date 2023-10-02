Salut Apprenant2 !


## Étape 1 : Détail d'une carte

&#9989; Ok, la page de la carte existe bien.

&#9989; Les données sont bien récupérées et envoyées à la vue.

&#9989; A l'affichage on a bien l'image, le nom, et l'élément.

&#10060; En revanche,  tu as dû oublier les valeurs des directions, que tu peux afficher comme ça par exemple (comme pour les autres valeurs) :

```
<p><span>Valeurs</span> : </p>
<ul>
  <li>Nord : <%= card.value_north %></li>
  <li>Est : <%= card.value_east %></li>
  <li>Sud : <%= card.value_south %></li>
  <li>Ouest : <%= card.value_west %></li>
</ul>
```

## Étape 2 : Recherche

### \- Recherche par élément

&#9989; Bien, tu as réussi à afficher les résultats filtrés par élément.

&#10060; Mais apparement, tu n'es pas parvenu à gérer le petit "cas particulier". (tu y étais presque !)

En fait, lorsqu'on choisit "aucun" élément, on souhaite afficher la liste de toutes les cartes qui ne possèdent pas d'élément.

Or, dans ta méthode `getElements()`, ta requête SQL renvoie seulement les cartes qui ont un élément (dont l'élément n'est pas nul).

```
const query = "SELECT * FROM card WHERE element IS NOT NULL";
//traduction : sélectionner toutes les cartes dont l'élément n'est pas nul
```

Donc, lorsque tu récupères ces résultats depuis `getSearchResults()` dans le `searchController`, tu n'as que les cartes qui ont forcément un élément.

-Ensuite, pour trouver le bon résultats, tu réalises un filtre sur ta liste de carte : 

```
const results = cards.filter(cards => cards.element.includes(cardElement))
```

Dans le cas où on recherche les cartes sans élément, ça pose 2 problèmes : 
  - 1 : Tu recherches dans une liste <b>qui ne peut pas contenir le résultat que tu cherches</b>...
  - 2 : En principe, ce filtre devrait être fait à l'origine par ta requête SQL, ce qui évite de retraiter les données en JS après la requête en BDD. (comme dans la correction)


### &#10060; Recherche par niveau
### &#10060; Recherche par valeur 
### &#10060; Recherche par nom 

> Je ne trouve pas de trace des 3 autres méthodes de recherche dans le code. Mais tu as dû rester bloqué sur la recherche d'élément.

> Pareil pour l'affichage de la liste des résultats de la recherche, qui pourrait être plus complète.

## Étape 3 : Construire un deck

  >Pas mal, mais incomplet.

### &#9989;  3.1 Activer les sessions

Ok, tu as réussi à activer les sessions, et à stocker des cartes à l'intérieur.

Mais dans l'idéal, on ne fait pas l'initialisation de la session dans le code d'un Controller.

```
addDeck : async (req, res, next) => {
  try {
      if(!req.session.deck){
          req.session.deck = [];
      }
```

On fait ça directement dans un middleware, depuis le fichier `index.js`.
<br>
Par exemple : 

```
app.use((request, response, next) => {
	//si la propriété deck de la session vaut undefined, on la crée
	if (!request.session.deck) {
		request.session.deck = []
	}
	//sinon, on fait rien ...
	//et on passe la main au middleware suivant
	next();
});
```

Pourquoi ? Ça évite d'avoir à initialiser la session à chaque fois qu'on s'en sert quelque part.<br>
En initialisant directement dans le fichier index, on est certain que la session est initialisée dès le lancement de l'application, et on peut ensuite travailler avec, sans crainte, n'importe où dans le code.

### &#9989; 3.2 Ajouter une carte au deck

> &#9989; Ok, les cartes s'ajoutent bien dans le deck !

&#10060; Il manque juste la limitation à 5 cartes maximum dans le deck.

Avant d'entreprendre l'ajout d'une carte dans le deck, il faut vérifier s'il reste de la place !

Rien de bien compliqué, juste un petit if à  placer quelque part avant le <b>req.session.deck.push</b>
Dans l'idéal, avant de faire une requête en BDD pour gagner du temps.

``` 
if(request.session.deck.length < 5) 
```

### &#9989;  3.3 Une page pour visualiser le deck !

> Bon, globalement, tu arrives à afficher les cartes du deck. 
> Même si ce n'est pas très lisible !

Mais... la lecture du code de `deck.ejs` me fait poser quelques questions...

```
<td>
  <img class="img-thumbnail img-cart-list" src="/visuals/<%=card.visual_name%>" alt="">
  <div class="item-description">
      <h5 class="item-title"><%= card.name %></h5>
  </div>
</td>
<td>
  <p class="item-subtotal">$<%= card.price%></p>
</td>
<td>

</td>
<td>
```

> `<p class="item-subtotal">$<%= card.price%></p>` ??? &#128514;<br>
> Serait-ce un copié/collé d'un autre code ? d'un autre exercice ?<br>
Si c'est le cas, attention à ne pas oublier de vérifier le résultat final.<br>
Les cartes de notre jeu n'ont pas de prix ! (même si elles ont une valeur sentimentale inestimable)<br>
Par contre elles ont un niveau et une direction qui n'apparaissent pas... &#128533;


Si tu veux te perfectionner sur les tableaux, voici une version correcte possible (et un poil plus lisible &#128521;) : 

```
<tr>
  <td>
    <%= card.name %><br>
    <img class="img-thumbnail img-cart-list" src="/visuals/<%=card.visual_name%>" alt="">
  </td>
  <td style="vertical-align : middle">
    <ul>
      <li><%= card.element || 'Aucun' %></li>
      <li>Nord : <%= card.value_north %></li>
      <li>Est : <%= card.value_east %></li>
      <li>Sud : <%= card.value_south %></li>
      <li>Ouest : <%= card.value_west %></li>
    </ul>
  </td>
</tr>
```

### &#10060; 3.4 Supprimer une carte du deck

> On ne peut pas supprimer sa carte &#128557; Quelle déception ! ^^

J'imagine que tu as perdu un peu de temps sur le reste, et que tu n'as pas pu aller jusqu'à la suppression des cartes.

L'astuce pour enlever une carte facilement, c'est d'utiliser la fonction .filter() pour récupérer toutes les cartes, sauf celle qui a l'identifiant de la carte que tu veux supprimer : 

```
const newDeck = request.session.deck.filter((card) => { 
    return card.id !== parseInt(cardId, 10);
});

//puis on remplace le tableau de la session par la version filtrée
request.session.deck = newDeck
```

Essaie de perdre moins de temps la prochaine fois pour pouvoir traiter toutes les questions !