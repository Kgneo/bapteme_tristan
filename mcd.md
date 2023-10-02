Salut Apprenant !

> Globalement tout est juste, j'ai seulement trouvé une erreur de cardinalité sur la relation USER > PRODUCT.

La cardinalité correcte (dans les deux sens) est `(0, N)` et non `(1, 1)`
<br>
Puisqu'un USER peut liker <b>aucun</b> ou <b>plusieurs</b> PRODUCT, 
et qu'un PRODUCT peut être liké par <b>aucun</b> ou <b>plusieurs</b> USER.

Bravo pour le reste !