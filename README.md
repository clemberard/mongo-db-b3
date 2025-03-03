# mongo-db-b3

## Définitions :
- Index en BDD relationnelle : Sorte de drapeau qui permet de retrouver plus rapidement une information dans une table.

## Installation de MongoDB (MacOS) :
```bash
brew tap mongodb/brew
brew install mongodb-community
```

## TP1
- Ajout de données dans une collection
![Affichage nouvelles données](image-1.png)

## TP2

### Exercice 1
- Création d'une collection

![Ajout collection ecommerce_produits](image-2.png)

- Ajout de données dans la collection ecommerce_produits

![Nombre d'élements ajoutés](image-3.png)

### Exercice 2
- Récupérer tous les produits ayant la catégorie "Électroménager"
```json
db.ecommerce_produits.find({categorie: "Électroménager"})
```
- Trouver les produits ayant un prix entre 50 et 200
```json
db.ecommerce_produits.find({prix: {$gt: 50, $lt: 200}})
```
- Lister tous les produits en stock
```json
db.ecommerce_produits.find({stock: {$gt: 0}})
```
- Trouver les produits avec au moins 3 avis
```json
db.ecommerce_produits.find({commentaires: {$size: 3}})
```

### Exercice 3
- Modification de produits ayant la catégorie "Électroménager"

![Modif electromenager](image-4.png)

- Ajouter un champ promotion à certains produits

```json
db.ecommerce_produits.updateMany({prix: {$gt: 100}}, {$set: {promotion: true}})
```

- Ajouter un nouveau tag à tous les produits d'une catégorie

```json
db.ecommerce_produits.updateMany({categorie: "Électroménager"}, {$push: {tags: "nouveau_tag"}})
```

- Mettre à jour le stock après une vente

```json
db.ecommerce_produits.updateOne({nom: "Lave-linge"}, {$inc: {stock: -1}})
```

### Exercice 4

- Trouver les produits disponibles avec tag1 et tag2

```json
db.ecommerce_produits.find({tags: {$all: ["tag1", "tag2"]}})
```

- Lister les produits premium avec un stock faible

```json
db.ecommerce_produits.find({promotion: true, stock: {$lt: 5}})
```

- Rechercher les produits ayant reçu au moins un avis 5 étoiles

```json
db.ecommerce_produits.find({commentaires: {$elemMatch: {note: 5}}})
```

- Trouver les produits d'une categorie, triés par prix décroissant, limité aux 5 premiers

```json
db.ecommerce_produits.find({categorie: "Électroménager"}).sort({prix: -1}).limit(5)
```

## TP3

### Partie 2

1. Listez tous les livres disponibles

```json
db.livres.find()
```

2. Trouvez les livres publiés après l'an 2000

```json
db.livres.find({annee: {$gt: 2000}})
```

3. Trouvez les livres d'un auteur spécifique

```json
db.livres.find({auteur: "Albert Camus"})
```

4. Trouvez les livres qui ont une note moyenne supérieure à 4

```json
db.livres.find({note: {$gt: 4}})
```

5. Listez tous les utilisateurs habitant dans une ville spécifique

```json
db.utilisateurs.find({ville: "Toulouse"})
```

6. Trouvez les livres qui appartiennent à un genre spécifique

```json
db.livres.find({genre: "Science-fiction"})
```

7. Trouvez les livres qui ont à la fois un prix inférieur à 15€ et une note moyenne supérieure à 4

```json
db.livres.find({prix: {$lt: 15}, note: {$gt: 4}})
```

8. Trouvez les utilisateurs qui ont emprunté un livre spécifique (par titre)

```json
db.utilisateurs.find({"livres_empruntes.titre": "Dune"})
```

### Partie 3

1. Mettez à jour le titre d'un livre spécifique

```json
db.livres.updateOne({titre: "Dune"}, {$set: {titre: "Dune - L'Étoile"}})
```

2. Ajoutez un champ stock à tous les livres avec une valeur par défaut de 5

```json
db.livres.updateMany({}, {$set: {stock: 5}})
```

3. Marquez un livre comme indisponible

```json
db.livres.updateOne({titre: "Dune - L'Étoile"}, {$set: {disponible: false}})
```

4. Ajoutez un nouvel emprunt dans la liste livres_empruntes d'un utilisateur

```json
db.utilisateurs.updateOne({nom: "Dupont"}, {$push: {livres_empruntes: {titre: "Dune - L'Étoile", date_emprunt: new Date()}}})
```

5. Changez l'adresse d'un utilisateur

```json
db.utilisateurs.updateOne({nom: "Roux"}, {$set: {adresse: "1 rue de la Paix"}})
```

6. Ajoutez un nouveau tag à un utilisateur

```json
db.utilisateurs.updateOne({nom: "Bisset"}, {$push: {tags: "Je sais pas"}})
```

7. Mettez à jour la note moyenne d'un livre

```json
db.livres.updateOne({titre: "1984"}, {$set: {note: 4.5}})
```

### Partie 4

1. Supprimez un livre spécifique par son titre

```json
db.livres.deleteOne({titre: "1984"})
```

2. Supprimez tous les livres d'un auteur spécifique

```json
db.livres.deleteMany({auteur: "Albert Camus"})
```

3. Supprimez un utilisateur par son email

```json 
db.utilisateurs.deleteOne({email: "thomas.roux@example.com"})
```

### Partie 5

1. Listez tous les livres triés par note moyenne (ordre décroissant)

```json
db.livres.find().sort({note_moyenne: -1})
```

2. Trouvez les 3 livres les plus anciens

```json
db.livres.find().sort({annee: 1}).limit(3)
```

3. Comptez le nombre de livres par auteur

```json
db.livres.aggregate([{$group: {_id: "$auteur", count: {$sum: 1}}}])
```

4. Affichez uniquement le titre, l'auteur et la note moyenne des livres (sans l'id)

```json
db.livres.find({}, {titre: 1, auteur: 1, note_moyenne: 1, _id: 0})
```

5. Trouvez les utilisateurs qui ont emprunté plus d'un livre

```json
db.utilisateurs.find( { livres_empruntes: { $gt: 1 } } )
```

6. Recherchez les livres dont le titre contient un mot spécifique

```json
db.livres.find({titre: {$regex: ".*Dune.*"}})
```

7. Trouvez les livres dont le prix est entre 10€ et 20€

```json
db.livres.find({prix: {$gt: 10, $lt: 20}})
```
