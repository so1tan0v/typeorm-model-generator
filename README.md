[![npm version](https://badge.fury.io/js/ts-typeorm-model-generator.svg)](https://badge.fury.io/js/ts-typeorm-model-generator)

# ts-typeorm-model-generator

## Gratitude
1. https://github.com/Kononnable
2. https://github.com/anapaula-noleto
3. https://github.com/0xcra5hs

## For Developers
This library works with Node.js >= 18

## Usage 
Generates models for TypeORM from existing databases.
Supported db engines:
* Microsoft SQL Server
* PostgreSQL
* MySQL
* MariaDB
* Oracle Database
* SQLite

## Installation
### Versions
Typeorm-model-generator-suffix comes with preinstalled driver for each supported db(except for oracle). However if you want to use it as a dev-dependency you may want to install your db driver manually to reduce dependency footprint, reduce time spent in the CI. In such case you can use version without preinstalled db drivers - `npm i typeorm-model-generator-suffix`.  
### Global module
To install module globally simply type `npm i -g typeorm-model-generator-suffix` in your console.
### Npx way
Thanks to npx you can use npm modules without polluting global installs. So nothing to do here :)
>To use `npx` you need to use npm at version at least 5.2.0. Try updating your npm by `npm i -g npm`
### Database drivers
All database drivers except oracle are installed by default. To use typeorm-model-generator with oracle database you need to install driver with `npm i oracledb` and configure [oracle install client](http://www.oracle.com/technetwork/database/database-technologies/instant-client/overview/index.html) on your machine.

## Usage 

### In code

```typescript
export async function generateEntity(
  databaseConfig: DatabaseConfig,
  schemas: string[],
  resultPath: string
): Promise<string> {
  fs.mkdirSync(resultPath, { recursive: true });

  let options = generateConfig(databaseConfig, schemas, resultPath);

  const driver = createDriver(options.connectionOptions.databaseType);
  await createModelFromDatabase(driver, options.connectionOptions, options.generationOptions);

  return resultPath;
}

function generateConfig(
  databaseConfig: DatabaseConfig,
  schemas: string[],
  resultPath: string
): {
  generationOptions: IGenerationOptions;
  connectionOptions: IConnectionOptions;
} {
  const generationOptions = getDefaultGenerationOptions();
  const connectionOptions = getDefaultConnectionOptions();

  connectionOptions.host = databaseConfig.host;
  connectionOptions.port = databaseConfig.port;
  connectionOptions.databaseType = databaseConfig.dialect;
  connectionOptions.user = databaseConfig.user;
  connectionOptions.password = databaseConfig.password;
  connectionOptions.databaseNames = [databaseConfig.db_name];
  connectionOptions.schemaNames = schemas;

  generationOptions.skipPKcheck = true;
  generationOptions.suffixClassName = 'Entity';
  generationOptions.suffixCaseFile = '.entity';
  generationOptions.convertCaseEntity = 'pascal';
  generationOptions.resultsPath = path.join(resultPath, 'entities');
  generationOptions.noConfigs = true;
  generationOptions.separateEntityAccordingSchemes = true;

  return {
    generationOptions,
    connectionOptions
  };
}

```

### CLI

There are two way to use this utility:
- Use step by step wizard which will guide you though the process - just type `npx typeorm-model-generator` in your console.
- Provide all parameters through command line(examples below)


Use `npx typeorm-model-generator --help` to see all available parameters with their descriptions. Some basic parameters below:
```shell
Usage: typeorm-model-generator -h <host> -d <database> -p [port] -u <user> -x
[password] -e [engine]

Options:
  --help                 Show help                                     [boolean]
  --version              Show version number                           [boolean]
  -h, --host             IP address/Hostname for database server
                                                          [default: "127.0.0.1"]
  -d, --database         Database name(or path for sqlite)            [required]
  -u, --user             Username for database server
  -x, --pass             Password for database server              [default: ""]
  -p, --port             Port number for database server
  -e, --engine           Database engine
          [choices: "mssql", "postgres", "mysql", "mariadb", "oracle", "sqlite"]
                                                              [default: "mssql"]
  -o, --output           Where to place generated models
                            [default: "./output"]
  -s, --schema           Schema name to create model from. Only for mssql
                         and postgres. You can pass multiple values
                         separated by comma eg. -s scheme1,scheme2,scheme3
  --ssl                                               [boolean] [default: false]
  --suffix-file          .entity
  --suffix-class         Entity
  --skip-pk-check        Skip primary key check
```
### Examples

* Creating model from local MSSQL database
   * Global module
      ```
      typeorm-model-generator -h localhost -d tempdb -u sa -x !Passw0rd -e mssql -o .
      ````
   * Npx Way
      ```
      npx typeorm-model-generator -h localhost -d tempdb -u sa -x !Passw0rd -e mssql -o .
      ````
* Creating model from local Postgres database, public schema with ssl connection
   * Global module
      ```
      typeorm-model-generator -h localhost -d postgres -u postgres -x !Passw0rd -e postgres -o . -s public --ssl
      ````
   * Npx Way
      ```
      npx typeorm-model-generator -h localhost -d postgres -u postgres -x !Passw0rd -e postgres -o . -s public --ssl
      ````
* Creating model from SQLite database
   * Global module
      ```
      typeorm-model-generator -d "Z:\sqlite.db" -e sqlite -o .
      ````
   * Npx Way
      ```
      npx typeorm-model-generator -d "Z:\sqlite.db" -e sqlite -o .
      ````
## Use Cases
Please take a look at [few workflows](USECASES.md) which might help you with deciding how you're gonna use typeorm-model-generator.
## Naming strategy
If you want to generate custom names for properties in generated entities you need to use custom naming strategy. You need to create your own version of [NamingStrategy](https://github.com/Kononnable/typeorm-model-generator/blob/master/src/NamingStrategy.ts) and pass it as command parameter.

```typeorm-model-generator -d typeorm_mg --namingStrategy=./NamingStrategy -e sqlite -db /tmp/sqliteto.db```
