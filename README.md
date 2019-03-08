# db-seeding
Docasny navod na db seeding

# Db seeding

## Úvod.

Db seeding narozdíl od DB migrací, neslouží pro aktualizací DB schématu, ale pouze pro generování dat. 
Existují většínou dvě skupiny dat, která můžeme generovat. 
1. Inicializační data nutná pro fungování aplikace. Typicky se může jednat o nějaký základní administrační účet apod.
2. Náhodná data pro testování. Zde většinou využijeme nějaký generátor náhodných dat, aby data byla [realisticka](https://medium.freecodecamp.org/how-our-test-data-generator-makes-fake-data-look-real-ace01c5bde4a). 

## Nástroje
V Ipex projektech budeme používat pro DB seeding [knex](https://knexjs.org/#Seeds-CLI). Pro DB migrace budeme ale používat [db-migrate](https://db-migrate.readthedocs.io/en/latest/#db-migrate)

## Konfigurace 
Každý projekt s nakonfigurovaným DB seedingem bude v root složce obsahovat soubor `knexfile.js`. zde je příklad konfigurace 
```javascript
const path = require('path')

module.exports = {
    client: 'mysql',
    connection: process.env.DATABASE_DSN || 'mysql://root:ipex@localhost:3306/ipbxdata',
    seeds: {
        directory: path.resolve(__dirname, 'db/seeds')
    }
}
```
Důležité je nastavení přípojení k DB, které se získává z [proměnného prostředí](https://12factor.net/config)(proměnná `DATABASE_DSN`) a dále `seed.directory`, která určuje umístění vygenerovaných seed souborů. Pokud není proménná `DATABASE_DSN` nastavena používá se defaultní hodnota, která odpovída vývojovému prostředí v dockeru, které postupně budeme přidávat do projektů. Pokud si chcete upravit lokální nastavení nastavte proměnnou `DATABASE_DSN` či proměnnou uvedenou v `knexfile.js` nastavení projektů se může mírně lišit. Zde jsou příklady jak se nastavuje proměnné prostředí pro [Windows](http://www.dowdandassociates.com/blog/content/howto-set-an-environment-variable-in-windows-command-line-and-registry/), [Bash a Zsh](https://flaviocopes.com/shell-environment-variables/) a [Fish](https://fishshell.com/docs/current/faq.html).

## Vygenerování seed souboru
Nejjednodušší způsb je použít knex CLI např. `npx knex seed:make test-data-seed` vytvoří soubor `db/seeds/test-data-seed.js` s obsahem:
```javascript
exports.seed = function(knex, Promise) {
  // Deletes ALL existing entries
  return knex('table_name').del()
    .then(function () {
      // Inserts seed entries
      return knex('table_name').insert([
        {id: 1, colName: 'rowValue1'},
        {id: 2, colName: 'rowValue2'},
        {id: 3, colName: 'rowValue3'}
      ]);
    });
};
```
u seed souborů je důležité že musí exportovat metodu seed, kterou knex runner potom volá. Soubory lze vytváře i ručně pokud budete chtít. V seed souborech je již možné používat cokoliv ze syntaxe [knexu](https://knexjs.org/) případně psát jakýkoliv kód. Myslete jen na to, že by jste neměli měnít DB schéma. Dále je dobré psát soubory tak, aby generovali data pouze pokud je DB prázdná, jelikož knex seed runner spustí vždy všechny seed soubory. Dále si dejte pozor na jednu neštastnou vlastnost knexu, pokud vytvoríte seed se jménem, které již existuje soubor se Vám přemaže. 

## Spustění DB seedingu
Command `npx knex seed:run` spustí všechny seed soubory umístěné ve složce definované v konfiguračním souboru. Narozdíl od migrací se pouští se vždy všechny soubory. 

## Ukázka složitějšího seed souboru
Jedna se o example blog aplikaci a vygenerování náhodných dat. Pro generovani nahodnych dat používám [faker](https://www.npmjs.com/package/faker)
```javascript
const R = require('ramda')
const faker = require('faker')
faker.locale = 'cs'
const usersCount = 25
const articlesCount = 500
const tagsCount = 18

const usersTable = 'users'
const tagsTable = 'tags'
const articlesTable = 'articles'
const taggingsTable = 'taggings'

const createRandomUser = () => {
  return {
    email: faker.internet.email(),
    password_digest: faker.internet.password()
  }
}

const createRandomArticle = () => {
  return {
    title: faker.random.words(faker.random.number({min: 3, max: 7})),
    body: faker.lorem.paragraphs(faker.random.number({min: 5, max: 12})),
    author_id: faker.random.number({min: 1, max: usersCount})
  }
}

const createRandomTagging = () => {
  return {
    tag_id: faker.random.number({min: 1, max: tagsCount})
  }
}

const addIdsToListOfObjects = (list) => {
  const ids = R.range(1, list.length + 1)
  return R.zipWith((value, id) => R.mergeDeepLeft({id}, value), list, ids)
}

const users = addIdsToListOfObjects(R.times(createRandomUser, usersCount))
const tags = addIdsToListOfObjects([
  {name: 'Behavioral Psychology'},
  {name: 'Habits'},
  {name: 'Motivation'},
  {name: 'Procrastination'},
  {name: 'Creativity'},
  {name: 'Decision Making'},
  {name: 'Focus'},
  {name: 'Mental Toughness'},
  {name: 'Constant Improvement'},
  {name: 'Deliberate Practice'},
  {name: 'Goal Setting'},
  {name: 'Productivity'},
  {name: 'Better Sleep'},
  {name: 'Eating Healthy'},
  {name: 'Strength Training'},
  {name: 'Life Lessons'},
  {name: 'Reading List'},
  {name: 'Self-Improvement'}
])
const articles = addIdsToListOfObjects(R.times(createRandomArticle, articlesCount))

const taggings = []

for (const article of articles) {
  const tags = R.uniq(R.times(createRandomTagging, faker.random
                .number({min: 2, max: 5}))
                .map(i => R.mergeDeepLeft({article_id: article.id}, i)))
  taggings.push(...tags)
}

exports.seed = async (knex, Promise) => {
  await knex(usersTable).del()
  await knex(tagsTable).del()
  await knex(articlesTable).del()
  await knex(taggingsTable).del()
  await knex(usersTable).insert(users)
  await knex(tagsTable).insert(tags)
  await knex(articlesTable).insert(articles)
  await knex(taggingsTable).insert(taggings)
}
```

## Další zdroje

* https://gist.github.com/NigelEarle/70db130cc040cc2868555b29a0278261
* https://medium.com/@jaeger.rob/seed-knex-postgresql-database-with-json-data-3677c6e7c9bc
* https://blog.bitsrc.io/seeding-your-database-with-thousands-of-users-using-knex-js-and-faker-js-6009a2e5ffbf
* https://elliotekj.com/2017/04/26/seed-postgres-database-knex/
* https://www.jernejsila.com/2016/09/04/creating-database-migrations-seeds-node-js/
