Salut Apprenant1 !


## Étape 1 : Détail d'une carte

&#9989; Ok super, la page de la carte fonctionne bien.

#### * Juste une petite remarque :

> Dans le `dataMapper`, tu as pris la liberté de nommer la fonction `getOneCard()` au lieu de `getCard()` comme indiqué dans l'intitulé.
C'est un détail qui ne change rien au résultat final, et qui n'a aucune importance pour cet exercice, mais en "conditions réelles", je conseillerais plutôt d'utiliser le nom le plus simple et générique possible, comme getCard(), pour éviter les erreurs quand on travaille à plusieurs sur un même code. Par expérience, ça peut être la source d'une certaine perte de temps, lors du développement de futures fonctionnalités qui utiliseront peut-être cette même méthode.
Dans un "vrai" code, on est amené à gérer différents types de données (ex : card, user, deck, message, notification, etc). Il est donc conseillé d'utiliser des conventions de nommage uniforme, pour simplifier le travaille de tout le monde. 

> Ex : `getCard() getUser() getDeck()` etc.

> Ou : `getOneCard() getOneUser() getOneDeck()` etc.

Cela dit, `getOneCard()` est plus précis que `getCard()` et reste cohérent avec la méthode `getAllCards()` qui pourraient elle aussi être appelée `getCards()` pour faire plus simple ;D 

L'important c'est que l'ensemble soit cohérent !

## Étape 2 : Recherche

Pas mal ! Globalement ça marche bien, et tu as même pu finir les autres méthodes de recherche !

J'ai juste trouvé quelques petites erreurs : 

- ### Recherche par élément 
  Lorsqu'on fait une recherche par type d'élément, la recherche et le résultats sont correctes, mais il y a une petite erreur à l'affichage du titre, sur la page de résultats : 

  > &#10060; `Résultat de la recherche : NaN`

  Dans le code du `searchController`, dans la méthode `searchByElement()` l.22, tu construis le titre en fonction de la valeur de la valeur de `searchedElement`. Je pense qu'il y a juste une petite erreur de frappe ou de copié/collé dans l'expression suivante :
  
  ```
  (searchedElement === 'null' ? ' sans élément' : + searchedElement
  ```

  > Le '+' n'a rien à faire là ! ;D

  Sinon ça roule !

- ### Recherche par niveau 

  &#9989; Parfait, on a bien le niveau exact. Pas de "au moins".

- ### Recherche par valeur 
  - &#10060; En revanche ici, on voulait <i><b>toutes les cartes qui ont au moins la valeur choisie dans la direction sélectionnée.</b></i>

    Or dans les résultats on ne trouve que des cartes qui possède la valeur exacte demandée.

    Dans la méthode `searchByValues()` du `dataMapper` l.72 : 
    
    >Remplacer le `=` par `>=`

  - Sinon bien joué ! Tu as même amélioré la requête SQL proposé dans la correction. C'est effectivement plus lisible comme ça, et aussi plus performant à l'execution puisque ta requête est moins compliqué !

    ```
    const valueColumn = 'value_${direction.replace(/'/g, "''")}'';
    ```

    Parcontre, je ne comprend pas l'utilité du replace() ?? à priori il n'est pas nécessaire ici. 
    
    ```
    const valueColumn = 'value_${direction}';
    // ça devrait suffir, pourquoi l'avoir commenté ?
    ```

- ### Recherche par nom
  &#9989; Ça marche ! 

  #### * Petite remarque sur le code de `searchByName()` dans le  `dataMapper` :
  > La correction montre comment écrire la même requête de façon plus synthétique.

  > Et on pourrait même faire encore plus court !

  ```
  getCardsByLevel: async function (level) {
    const results = await database.query({
      text: `SELECT * FROM "card" WHERE "level"=$1`,
      values: [level]
    });
    return results.rows;
  }
  ```

  Quand on peut gagner quelques lignes dans nos fichiers on hésite pas. On peut vite perdre un temps fou à scroller !!! xD

## Étape 3 : Construire un deck

  &#9989; Ça marche aussi ! 

  Comme demandé, si la carte est déjà présente dans le deck ou que le deck possède déjà 5 cartes, on ne fait rien. Ok.

  > Au niveau de l'organisation du code, il aurait été tout aussi judicieux de réutiliser la vue `main/cardList.ejs` au lieu de créer une nouvelle vue `card/deck.ejs`, pour éviter de dupliquer du code et d'avoir à modifier les 2 fichiers lors des futures évolutions du design de nos super cartes !
  > Dans la mesure du possible, essaie de dupliquer le moins possible des bouts de codes trop similaires...

### &#9989;  3.1 Activer les sessions

Ok, tu as activé les sessions dans un middlware. Ça marche.

Mais il serait plus propre de le déclarer dans le fichier `index.js` plutôt que dans le router.<br> 
De façon à garder dans `router.js` uniquement ce qui concerne le router.


### &#9989;  3.2 Ajouter une carte au deck &#128076;
### &#9989;  3.3 Une page pour visualiser le deck ! &#128076;
### &#9989;  3.4 Supprimer une carte du deck &#128076;
   
  C'est bien tu as pensé à convertir l'identifiant en Number !
   ```
  const cardId = Number(req.params.id);
  ```

### &#9989; Super ! Tout marche bien, sauf quelques petites erreurs &#128079; &#128079; &#128079;