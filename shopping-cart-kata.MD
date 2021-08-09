RESTful ShoppingCart
- Jedes Produkt hat eine productId
- Ich kann neue Produkte in den Shopping Cart hinzufügen
- Ich kann Produkte aus dem Shopping Cart löschen
- Ich kann mir alle Produkte in meinem ShoppingCart anzeigen

-----------------------------
Shopping Cart starts empty

GET /shoppingcarts/gregor

>404 Not Found

----------------------------
Create ShoppingCart "gregor" with 1x product of Id 1

PUT /shoppingcarts/gregor
content-type: text/csv

productId, amount
1        , 1

>204 No Content

-----------------------------
Add Product to Shopping Cart

PUT /shoppingcarts/gregor
content-type: text/csv

productId, amount
1        , 1
2        , 9

>204 Content
-----------------------------
Delete one Product from Shopping Cart

PUT /shoppingcarts/gregor
content-type: text/csv

productId, amount
2        , 9

>204 Content
-----------------------------
Read Shopping Cart

GET /shoppingcarts/gregor

>200 OK
content-type: text/csv
productId, amount
2        , 9

-----------------------------
Delete whole Shopping Cart

DELETE /shoppingcarts/gregor

>204 No Content

----------------------------
GET /shoppingcarts/gregor

>404 No Content
