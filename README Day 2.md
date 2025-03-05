# TP Jour 2

## TP 1

### Exercice 1.1

1. Script pour générer des livres dans la bdd:

```javascript
const genres = [
  "Science-fiction",
  "Fantasy",
  "Policier",
  "Thriller",
  "Horreur",
  "Romance",
  "Historique",
  "Biographie",
  "Jeunesse",
  "BD",
];
const langues = [
  "français",
  "anglais",
  "espagnol",
  "allemand",
  "italien",
  "portugais",
  "russe",
  "chinois",
  "japonais",
  "arabe",
];
const editeurs = [
  "Gallimard",
  "Folio",
  "J'ai Lu",
  "Pocket",
  "LGF",
  "Le Livre de Poche",
  "Flammarion",
  "Grasset",
  "Albin Michel",
  "Hachette",
];

for (let i = 0; i < 100; i++) {
  const livre = {
    titre: `Livre ${i + 1}`,
    auteur: `Auteur ${i + 1}`,
    annee: Math.floor(Math.random() * 100) + 1920,
    genre: genres[Math.floor(Math.random() * genres.length)],
    langue: langues[Math.floor(Math.random() * langues.length)],
    editeur: editeurs[Math.floor(Math.random() * editeurs.length)],
    prix: Math.floor(Math.random() * 30) + 5,
    note_moyenne: Math.floor(Math.random() * 6),
  };
  db.livres.insertOne(livre);
}
```

2. Analysez les performances des requetes suivantes sans index
   en utilisant la commande `explain("executionStats")`:

```javascript
db.livres.find({titre: 'Livre 500'}).explain("executionStats");

db.livres.find({auteur: 'Auteur 657'}).explain("executionStats");

db.livres.find({prix: {$gt: 10, $lt: 20}, note_moyenne: {$gt: 2}}).explain("executionStats");

db.livres.find({genre: 'Fantasy', langue: 'français'}).sort({note_moyenne: -1}).explain("executionStats");
```

| Requête                                    | totalDocsExamined | executionTimeMillis | stage    |
| ------------------------------------------ | ----------------- | ------------------- | -------- |
| {titre: 'Livre 500'}                       | 1110              | 0                   | COLLSCAN |
| {auteur: 'Auteur 657'}                     | 1110              | 0                   | COLLSCAN |
| {prix: {$gt: 10, $lt: 20}, note_moyenne: {$gt: 2}} | 1110              | 1                   | COLLSCAN |
| {genre: 'Fantasy', langue: 'français'}     | 1110              | 1                   | COLLSCAN |

### Exercice 1.2

1. Créer des index appropriés pour optimiser chacune des requetes precedentes:

```javascript
db.livres.createIndex({titre: 1});
db.livres.createIndex({auteur: 1});
db.livres.createIndex({prix: 1, note_moyenne: 1});
db.livres.createIndex({genre: 1, langue: 1, note_moyenne: -1});
```

2. Executer les requetes precedentes et analyser les performances:

```javascript
db.livres.find({titre: 'Livre 500'}).explain("executionStats");

db.livres.find({auteur: 'Auteur 657'}).explain("executionStats");

db.livres.find({prix: {$gt: 10, $lt: 20}, note_moyenne: {$gt: 2}}).explain("executionStats");

db.livres.find({genre: 'Fantasy', langue: 'français'}).sort({note_moyenne: -1}).explain("executionStats");
```

| Requête                                    | totalDocsExamined | executionTimeMillis | stage |
| ------------------------------------------ | ----------------- | ------------------- | ----- |
| {titre: 'Livre 500'}                       | 1                 | 1                   | FETCH |
| {auteur: 'Auteur 657'}                     | 1                 | 1                   | FETCH |
| {prix: {$gt: 10, $lt: 20}, note_moyenne: {$gt: 2}} | 178               | 1                   | FETCH |
| {genre: 'Fantasy', langue: 'français'}     | 15                | 1                   | FETCH |

**Comparaison des performances:**

- Les performances des requetes ont été améliorées après la création des index.
- Le nombre de documents examinés a été réduit pour chaque requete.

### Exercice 1.3

1. Créer un index de texte sur les champs titre et description des livres:

```javascript
db.livres.createIndex({titre: "test index", description: "test index"});
```

2. Tester une recherche de texte simple et analyser les performances:

```javascript
db.livres.find({$text: {$search: "science"}}).explain("executionStats");
> totalDocsExamined: 0
> executionTimeMillis: 1
> stage: 'TEXT_MATCH'
```

3. Créer une nouvelle collectio `sessions_utilisateurs` avec des documents id, date de derniere activité et données de session:

```javascript
db.sessions_utilisateurs.insertMany([
  {
    id: 1,
    date_derniere_activite: new Date("2025-02-15"),
    donnees_session: {user_id: 1, role: "admin"}
  },
  {
    id: 2,
    date_derniere_activite: new Date("2025-02-16"),
    donnees_session: {user_id: 2, role: "user"}
  },
  {
    id: 3,
    date_derniere_activite: new Date("2025-02-17"),
    donnees_session: {user_id: 3, role: "user"}
  }
]);
```

4. Créer un index TTL pour supprimer automatiquement les sessions après 30 min d'inactivité:

```javascript
db.sessions_utilisateurs.createIndex({date_derniere_activite: 1}, {expireAfterSeconds: 1800});
```

### Exercice 1.4

1. Créer un index qui permet d'obtenir une requete couverte

```javascript
db.livres.createIndex({genre: 1, langue: 1, titre: 1, auteur: 1});
```

```javascript
db.livres.find({genre: 'Fantasy', langue: 'français'}, {titre: 1, auteur: 1, _id: 0}).explain("executionStats");
> totalDocsExamined: 0
```

Créer un indec unique sur le champ ISBN
