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

for (let i = 0; i < 1000; i++) {
  const livre = {
    titre: `Livre ${i + 1}`,
    auteur: `Auteur ${i + 1}`,
    annee: Math.floor(Math.random() * 100) + 1920,
    genre: genres[Math.floor(Math.random() * genres.length)],
    langue: langues[Math.floor(Math.random() * langues.length)],
    editeur: editeurs[Math.floor(Math.random() * editeurs.length)],
    prix: Math.floor(Math.random() * 30) + 5,
    note_moyenne: Math.floor(Math.random() * 6),
    disponible: Math.random() < 0.5,
    isbn: Math.floor(Math.random() * 1000000000000),
  };
  db.livres.insertOne(livre);
}
```

2. Analysez les performances des requetes suivantes sans index
   en utilisant la commande `explain("executionStats")`:

```javascript
db.livres.find({ titre: "Livre 500" }).explain("executionStats");

db.livres.find({ auteur: "Auteur 657" }).explain("executionStats");

db.livres
  .find({ prix: { $gt: 10, $lt: 20 }, note_moyenne: { $gt: 2 } })
  .explain("executionStats");

db.livres
  .find({ genre: "Fantasy", langue: "français" })
  .sort({ note_moyenne: -1 })
  .explain("executionStats");
```

| Requête                                            | totalDocsExamined | executionTimeMillis | stage    |
| -------------------------------------------------- | ----------------- | ------------------- | -------- |
| {titre: 'Livre 500'}                               | 1110              | 0                   | COLLSCAN |
| {auteur: 'Auteur 657'}                             | 1110              | 0                   | COLLSCAN |
| {prix: {$gt: 10, $lt: 20}, note_moyenne: {$gt: 2}} | 1110              | 1                   | COLLSCAN |
| {genre: 'Fantasy', langue: 'français'}             | 1110              | 1                   | COLLSCAN |

### Exercice 1.2

1. Créer des index appropriés pour optimiser chacune des requetes precedentes:

```javascript
db.livres.createIndex({ titre: 1 });
db.livres.createIndex({ auteur: 1 });
db.livres.createIndex({ prix: 1, note_moyenne: 1 });
db.livres.createIndex({ genre: 1, langue: 1, note_moyenne: -1 });
```

2. Executer les requetes precedentes et analyser les performances:

```javascript
db.livres.find({ titre: "Livre 500" }).explain("executionStats");

db.livres.find({ auteur: "Auteur 657" }).explain("executionStats");

db.livres
  .find({ prix: { $gt: 10, $lt: 20 }, note_moyenne: { $gt: 2 } })
  .explain("executionStats");

db.livres
  .find({ genre: "Fantasy", langue: "français" })
  .sort({ note_moyenne: -1 })
  .explain("executionStats");
```

| Requête                                            | totalDocsExamined | executionTimeMillis | stage |
| -------------------------------------------------- | ----------------- | ------------------- | ----- |
| {titre: 'Livre 500'}                               | 1                 | 1                   | FETCH |
| {auteur: 'Auteur 657'}                             | 1                 | 1                   | FETCH |
| {prix: {$gt: 10, $lt: 20}, note_moyenne: {$gt: 2}} | 178               | 1                   | FETCH |
| {genre: 'Fantasy', langue: 'français'}             | 15                | 1                   | FETCH |

**Comparaison des performances:**

- Les performances des requetes ont été améliorées après la création des index.
- Le nombre de documents examinés a été réduit pour chaque requete.

### Exercice 1.3

1. Créer un index de texte sur les champs titre et description des livres:

```javascript
db.livres.createIndex({ titre: "test index", description: "test index" });
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
    donnees_session: { user_id: 1, role: "admin" },
  },
  {
    id: 2,
    date_derniere_activite: new Date("2025-02-16"),
    donnees_session: { user_id: 2, role: "user" },
  },
  {
    id: 3,
    date_derniere_activite: new Date("2025-02-17"),
    donnees_session: { user_id: 3, role: "user" },
  },
]);
```

4. Créer un index TTL pour supprimer automatiquement les sessions après 30 min d'inactivité:

```javascript
db.sessions_utilisateurs.createIndex(
  { date_derniere_activite: 1 },
  { expireAfterSeconds: 1800 }
);
```

### Exercice 1.4

1. Créer un index qui permet d'obtenir une requete couverte

```javascript
db.livres.createIndex({ genre: 1, langue: 1, titre: 1, auteur: 1 });
```

```javascript
db.livres.find({genre: 'Fantasy', langue: 'français'}, {titre: 1, auteur: 1, _id: 0}).explain("executionStats");
> totalDocsExamined: 0
```

2. Créer un index unique sur le champ ISBN des livres

```javascript
db.livres.createIndex({ isbn: 1 }, { unique: true });
```

> [!NOTE]
>
> J'ai du refaire ma bdd pour ajouter un champ `isbn` aux livres.

3. Créer un index partiel qui n'indexe que les livres disponibles

```javascript
db.livres.createIndex(
  { disponible: 1 },
  { partialFilterExpression: { disponible: true } }
);
```

4. Activer le profiler MongoDB pour identifier les requetes lentes

```javascript
db.setProfilingLevel(1, {slowms: 100});
> { was: 0, slowms: 100, sampleRate: 1, ok: 1 }
```

### Rapport

[Rapport concis TP1](RapportTp1Day2.md)

## TP 2

### Exercice 2.1

1. Modifier le schéma de vos utilisateurs pour inclure des coordonnees geographiques dans leur adresse

```javascript
const Nom = [
  "Dupont",
  "Durand",
  "Lefevre",
  "Leroy",
  "Moreau",
  "Lefebvre",
  "Fournier",
  "Girard",
  "Bonnet",
  "Thomas",
];
const Prenom = [
  "Jean",
  "Pierre",
  "Jacques",
  "Paul",
  "Louis",
  "Robert",
  "Marcel",
  "Michel",
  "Georges",
  "Henri",
];
const Rue = [
  "rue de la Paix",
  "rue de la Liberté",
  "rue de la République",
  "rue du Commerce",
  "rue des Ecoles",
  "rue de la Gare",
  "rue des Fleurs",
  "rue des Lilas",
  "rue des Champs",
  "rue des Vignes",
];
const Ville = [
  "Paris",
  "Marseille",
  "Lyon",
  "Toulouse",
  "Nice",
  "Nantes",
  "Strasbourg",
  "Montpellier",
  "Bordeaux",
  "Lille",
];
const CP = [
  "75001",
  "13001",
  "69001",
  "31000",
  "06000",
  "44000",
  "67000",
  "34000",
  "33000",
  "59000",
];

for (let i = 0; i < 1000; i++) {
  const personne = {
    nom: Nom[Math.floor(Math.random() * Nom.length)],
    prenom: Prenom[Math.floor(Math.random() * Prenom.length)],
    email: `${Nom[
      Math.floor(Math.random() * Nom.length)
    ].toLowerCase()}.${Prenom[
      Math.floor(Math.random() * Prenom.length)
    ].toLowerCase()}@example.com`,
    adresse: {
      rue: `${Math.floor(Math.random() * 100) + 1} ${
        Rue[Math.floor(Math.random() * Rue.length)]
      }`,
      ville: Ville[Math.floor(Math.random() * Ville.length)],
      cp: CP[Math.floor(Math.random() * CP.length)],
      coordonnees: {
        type: "Point",
        coordinates: [Math.random() * 360 - 180, Math.random() * 180 - 90],
      },
    },
    tags: ["tag1", "tag2", "tag3"],
  };
  db.utilisateurs.insertOne(personne);
}
```

2. Créer une nouvelle collection `bibliotheques` avec un nom et adresse, des coordonnes GeoJSON et une zone de service comme un polygone GeoJSON (3 bibliothèques)

```javascript
db.bibliotheques.insertMany([
  {
    nom: "Bibliothèque Nationale",
    adresse: {
      rue: "5 rue Vivienne",
      ville: "Paris",
      cp: "75002",
      coordonnees: {
        type: "Point",
        coordinates: [2.3394, 48.8665],
      },
    },
    zone_service: {
      type: "Polygon",
      coordinates: [
        [
          [2.3394, 48.8665],
          [2.3394, 48.8666],
          [2.3395, 48.8666],
          [2.3395, 48.8665],
          [2.3394, 48.8665],
        ],
      ],
    },
  },
  {
    nom: "Bibliothèque Municipale",
    adresse: {
      rue: "10 rue de la Mairie",
      ville: "Lyon",
      cp: "69001",
      coordonnees: {
        type: "Point",
        coordinates: [4.8357, 45.764],
      },
    },
    zone_service: {
      type: "Polygon",
      coordinates: [
        [
          [4.8357, 45.764],
          [4.8357, 45.7641],
          [4.8358, 45.7641],
          [4.8358, 45.764],
          [4.8357, 45.764],
        ],
      ],
    },
  },
  {
    nom: "Bibliothèque Universitaire",
    adresse: {
      rue: "15 rue des Universités",
      ville: "Strasbourg",
      cp: "67000",
      coordonnees: {
        type: "Point",
        coordinates: [7.7521, 48.5734],
      },
    },
    zone_service: {
      type: "Polygon",
      coordinates: [
        [
          [7.7521, 48.5734],
          [7.7521, 48.5735],
          [7.7522, 48.5735],
          [7.7522, 48.5734],
          [7.7521, 48.5734],
        ],
      ],
    },
  },
]);
```

3. Créer un index geospatial sur les coordonnees des utilisateurs et des bibliothèques

```javascript
db.utilisateurs.createIndex({ "adresse.coordonnees": "2dsphere" });
db.bibliotheques.createIndex({ "adresse.coordonnees": "2dsphere" });
```

### Exercice 2.2

1. Trouver les 5 utilisateurs les plus proches d'un point donné dans un rayon de 5km

```javascript
// REQUETE
db.bibliotheques
  .find({
    "adresse.coordonnees": {
      $near: {
        $geometry: {
          type: "Point",
          coordinates: [2.3394, 48.8665],
        },
        $maxDistance: 5000,
      },
    },
  })
  .limit(5);
```

```javascript
// RESULTAT
{
  _id: ObjectId('67c836e9d436aace4eba94e1'),
  nom: 'Bibliothèque Nationale',
  adresse: {
    rue: '5 rue Vivienne',
    ville: 'Paris',
    cp: '75002',
    coordonnees: {
      type: 'Point',
      coordinates: [
        2.3394,
        48.8665
      ]
    }
  },
  zone_service: {
    type: 'Polygon',
    coordinates: [
      [
        [
          2.3394,
          48.8665
        ],
        [
          2.3394,
          48.8666
        ],
        [
          2.3395,
          48.8666
        ],
        [
          2.3395,
          48.8665
        ],
        [
          2.3394,
          48.8665
        ]
      ]
    ]
  }
}
```
 
2. Trouvez les bibliothèques les plus proches d'un utilisateur spécifique

```javascript
// REQUETE
db.utilisateurs.find(ObjectId("67c83641059f377a67c1e8e1")).forEach((user) => {
  db.bibliotheques
    .aggregate([
      {
        $geoNear: {
          near: user.adresse.coordonnees,
          distanceField: "distance",
          maxDistance: 5000000000,
          spherical: true,
        },
      },
      { $limit: 5 },
    ])
    .forEach((bibliotheque) => {
      printjson(bibliotheque);
    });
});
```

```javascript
// RESULTAT
{
  _id: ObjectId('67c836e9d436aace4eba94e2'),
  nom: 'Bibliothèque Municipale',
  adresse: {
    rue: '10 rue de la Mairie',
    ville: 'Lyon',
    cp: '69001',
    coordonnees: { type: 'Point', coordinates: [Array] }
  },
  zone_service: { type: 'Polygon', coordinates: [ [Array] ] },
  distance: 6508800.442979292
}
```

3. Utiliser l'opérateur `$geoNear` dans un pipeline d'aggregation pour obtenir les bibliothèques triées par distance et calculer précisément la distance

```javascript
db.utilisateurs.aggregate([
  {
    $match: {
      _id: ObjectId("67c83641059f377a67c1e8e1"),
    },
  },
  {
    $lookup: {
      from: "bibliotheques",
      let: { coord: "$adresse.coordonnees" },
      pipeline: [
        {
          $geoNear: {
            near: "$$coord",
            distanceField: "distance",
            maxDistance: 5000000000,
            spherical: true,
          },
        },
        { $sort: { distance: 1 } },
      ],
      as: "bibliotheques_proches",
    },
  },
  { $unwind: "$bibliotheques_proches" },
  {
    $project: {
      _id: 0,
      nom: "$bibliotheques_proches.nom",
      distance: "$bibliotheques_proches.distance",
    },
  },
]);
```

```javascript
// RESULTAT
{
  nom: 'Bibliothèque Universitaire',
  distance: 6156332.590821906
}
{
  nom: 'Bibliothèque Nationale',
  distance: 6207774.322284079
}
{
  nom: 'Bibliothèque Municipale',
  distance: 6508800.442979292
}
```

### Exercice 2.3

1. Utiliser `$geoWithin` pour trouver les utilisateurs à l'intérieur d'une zone définie par un polygone

```javascript
const bibliothèque = db.bibliotheques.findOne({
  nom: "Bibliothèque Universitaire",
});
db.utilisateurs.find({
  "adresse.coordonnees": {
    $geoWithin: {
      $geometry: bibliothèque.zone_service,
    },
  },
});
```

```javascript
// RESULTAT
{
  _id: ObjectId('67c8592a059f377a67c1ecb6'),
  nom: 'Berard',
  prenom: 'Clément',
  email: 'clement.berard@example.com',
  adresse: {
    rue: "2 rue de l'excellence",
    ville: 'Melonville',
    cp: '12345',
    coordonnees: {
      type: 'Point',
      coordinates: [
        7.7521,
        48.5734
      ]
    }
  },
  tags: [
    'tag1',
    'tag2',
    'tag3'
  ]
}
```

2. Trouver tous les utilisateurs qui se trouvent dans la zone de service d'une bibliotheque spécifique

```javascript
db.bibliotheques
  .find({ nom: "Bibliothèque Universitaire" })
  .forEach((bibliotheque) => {
    db.utilisateurs
      .find({
        "adresse.coordonnees": {
          $geoWithin: {
            $geometry: bibliotheque.zone_service,
          },
        },
      })
      .forEach((user) => {
        printjson(user);
      });
  });
```

3. Créer une collection `rues` avec au moins une rue représentée comme un LineString GeoJSON, puis utiliser `$geoIntersects` pour trouver les bibliothèques dont la zone intersecte la rue

```javascript
db.rues.insertOne({
  nom: "rue de la Paix",
  troncon: {
    type: "LineString",
    coordinates: [
      [2.3394, 48.8665],
      [2.3394, 48.8666],
    ],
  },
});

db.rues.createIndex({ troncon: "2dsphere" });

// REQUETE
db.bibliotheques.find({
  zone_service: {
    $geoIntersects: {
      $geometry: db.rues.findOne().troncon,
    },
  },
});
```

### Exercice 2.4

1. Créer une collection livraisons pour suivre les livraisons de livres

```javascript
db.livraisons.insertMany([
  {
    livre: ObjectId("67c83641059f377a67c1e8e1"),
    utilisateur: ObjectId("67c83641059f377a67c1e8e1"),
    bibliotheque: ObjectId("67c83641059f377a67c1e8e1"),
    adresse_arrivee: {
      type: "Point",
      coordinates: [7.7521, 48.5734],
    },
    position_actuelle: {
      type: "Point",
      coordinates: [7.5521, 48.5734],
    },
    itiniéraire: {
      type: "LineString",
      coordinates: [
        [7.3521, 48.5734],
        [7.7521, 48.5734],
      ],
    },
  },
  {
    livre: ObjectId("67c83641059f377a67c1e8e1"),
    utilisateur: ObjectId("67c83641059f377a67c1e8e1"),
    bibliotheque: ObjectId("67c83641059f377a67c1e8e1"),
    adresse_arrivee: {
      type: "Point",
      coordinates: [7.7521, 48.5734],
    },
    position_actuelle: {
      type: "Point",
      coordinates: [7.7521, 48.5734],
    },
    itiniéraire: {
      type: "LineString",
      coordinates: [
        [7.7521, 48.5734],
        [7.7521, 48.5735],
      ],
    },
  },
  {
    livre: ObjectId("67c83641059f377a67c1e8e1"),
    utilisateur: ObjectId("67c83641059f377a67c1e8e1"),
    bibliotheque: ObjectId("67c83641059f377a67c1e8e1"),
    adresse_arrivee: {
      type: "Point",
      coordinates: [7.7521, 48.5734],
    },
    position_actuelle: {
      type: "Point",
      coordinates: [7.7521, 48.5735],
    },
    itiniéraire: {
      type: "LineString",
      coordinates: [
        [7.7521, 48.5735],
        [7.7522, 48.5735],
      ],
    },
  },
]);

db.livraisons.createIndex({ "adresse_arrivee": "2dsphere" });
db.livraisons.createIndex({ "position_actuelle": "2dsphere" });
db.livraisons.createIndex({ "itiniéraire": "2dsphere" });
```

2. Faire une fonction pour mettre à jour la position actuelle d'une livraison

```javascript
function updatePosition(livraisonId, position) {
  db.livraisons.updateOne(
    { _id: ObjectId(livraisonId) },
    { $set: { position_actuelle: { type: "Point", coordinates: position } } }
  );
}
```

3. Créer une requete pour trouver toutes les livraisons en cours à une distance de 1km d'une position donnée

```javascript
// REQUETE
db.livraisons.find({
  position_actuelle: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [7.7521, 48.5734],
      },
      $maxDistance: 1000,
    },
  },
});
```

```javascript
// RESULTAT
{
  _id: ObjectId('67c85d2b059f377a67c1ecbd'),
  livre: ObjectId('67c83641059f377a67c1e8e1'),
  utilisateur: ObjectId('67c83641059f377a67c1e8e1'),
  bibliotheque: ObjectId('67c83641059f377a67c1e8e1'),
  adresse_arrivee: {
    type: 'Point',
    coordinates: [
      7.7521,
      48.5734
    ]
  },
  position_actuelle: {
    type: 'Point',
    coordinates: [
      7.7521,
      48.5735
    ]
  },
  'itiniéraire': {
    type: 'LineString',
    coordinates: [
      [
        7.7521,
        48.5735
      ],
      [
        7.7522,
        48.5735
      ]
    ]
  }
}
```

### Rapport

[Rapport concis TP1](RapportTp2Day2.md)
