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
Command `npx knex seed:run` spustí všechny seed soubory umístěné v nakonfigurované složce. 

