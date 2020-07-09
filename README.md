# co-harvester

Harverster compatible avec de multiple API. Il peut faire une simple requête ou effectuer un scroll (récupération d'une succession de requêtes)

## Installation ##

```bash
npm install
```

_**Note : need at least Node v10.x.x**_

Si vous avez ce genre d'erreur, cela peut venir de la version de node qui n'est pas assez récente.

```js
ReferenceError: URL is not defined
    at Hal.requestByQuery (/xxx/co-harvester/lib/hal.js:28:18)
    at Object.<anonymous> (/xxx/co-harvester/index.js:125:18)
    at Module._compile (module.js:635:30)
    at Object.Module._extensions..js (module.js:646:10)
    at Module.load (module.js:554:32)
    at tryModuleLoad (module.js:497:12)
    at Function.Module._load (module.js:489:3)
    at Function.Module.runMain (module.js:676:10)
    at startup (bootstrap_node.js:187:16)
    at bootstrap_node.js:608:3
```

Pour *réparer* ce bug, il faut passer à une version de nodejs supérieur ou égale à 10 :

```bash
node --version # should print at least v10.x

# install via nodejs : https://nodejs.org/dist/latest-v10.x/ 

# install via nvm
nvm install 10
nvm use 10
nvm alias default 10 # mettre la version 10 de node par défaut
```

## Prérequis ##

Un fichier de configuration pour la source à moissonner (exemples dans conf/\*.json) **lorsqu'un fichier d'id(s) est utilisé**. Sinon, ce sont les paramètres passés en ligne de commande qui seront utilisés.

## Help ##

```bash
$ node index.js --help
Usage: index [options]

Options:
  --source <source>  required  targetted source (hal|conditor|crossref|pubmed)
  --query <query>    required   API query
  --ext <ext>        optionnal  archive extension (zip|gz)
  --proxy <proxy>    optionnal  set proxy url
  --ids <ids>        optionnal  path of file containing ids (one id by line)
  --conf <conf>      optionnal  conf path
  --output <output>  optionnal  output path (default : out/[source])
  --limit <limit>    optionnal  number of file(s) downloaded simultaneously
  --quiet            optionnal  quiet mode
  -h, --help         output usage information

Usages exemples: https://github.com/conditor-project/co-harvester

More infos about [CONDITOR API] here: https://github.com/conditor-project/api/blob/master/doc/records.md
More infos about [CROSSREF API] here: https://github.com/CrossRef/rest-api-doc
More infos about [HAL API] here: http://api.archives-ouvertes.fr/docs
More infos about [PUBMED API] here: https://www.ncbi.nlm.nih.gov/books/NBK25501/

```

## Proxy ##

Il est possible de configurer le harvester pour qu'il fonctionne derrière un proxy.

```bash
# Le paramètre --proxy permet de définir le proxy
--proxy=$http_proxy
--proxy=$https_proxy
--proxy="http://mon.proxy.com"
--proxy="https://mon.proxy.com"
```

## Usages ##

Voici des exemples d'usages (les plus courants) pour chaque source disponible ([plus de détails ici](#fonctionnement)) :

### Conditor ###

```bash
# query sur localhost (= un fichier .json)
node index.js --query="http://localhost:63332/v1/records?q=%22doi:*%22&page_size=1000&includes=doi&scroll=1m" --source="conditor" --output="out/conditor"
# query sur un serveur distant (= un fichier .json)
node index.js --query="https://api.conditor.fr/v1/records?q=%22doi:*%22&page_size=1000&includes=doi&scroll=1m" --source="conditor"  --output="out/conditor"

# liste d'ids sur un serveur distant (= un fichier .gz/.zip avec un fichier par id)
node index.js --ids="ids/conditor.txt" --conf="conf/conditor.json" --source="conditor" --output="out/conditor" --ext="gz"

# Traiter un fichier .json pour en extraire un corpus de document
node tools/extractFiles.js --input="out/conditor.json" --output="out/conditor.gz" --data="teiBlob" --id="idConditor" --ext="gz" --format=".tei"
# Le fichier JSON doit contenir la valeur idConditor ET teiBlob
# Plus d'infos avec la commande :
# node tools/extractFiles.js --help
```

### Crossref ###

```bash
# liste d'ids sur un serveur distant (= un fichier .gz/.zip avec un fichier par id)
node index.js --ids="ids/crossref/doi_wos_2014.txt" --conf="conf/crossref.json" --source="crossref" --output="out/crossref" --ext="gz"
```

### Hal ###

```bash
# query sur un serveur distant (= un fichier .json)
node index.js --query="https://api.archives-ouvertes.fr/search/?wt=json&q=structCountry_s:(fr OR gf OR gp OR mq OR re OR yt OR bl OR mf OR pf OR pm OR wf OR nc)&fq=producedDateY_i:2014&fl=docid,halId_s,label_xml&sort=docid+desc&rows=1000&cursorMark=*" --source="hal" --output="out/pubmed"
# query sur un serveur distant (= un fichier .json)
node index.js --query="http://api.archives-ouvertes.fr/ref/structure?fq=country_s:fr&fl=*&q=*&rows=1000&sort=docid asc&cursorMark=*" --source="hal" --output="out/pubmed"

# Traiter un fichier .json pour en extraire un corpus de document
node tools/extractFiles.js --input="out/hal.json" --output="out/hal.gz" --data="label_xml" --id="docid" --ext="gz" --format=".xml"
# Plus d'infos avec la commande :
# node tools/extractFiles.js --help
```

### Pubmed ###

```bash
# query sur localhost (= un fichier .gz/.zip avec un fichier par document)
node index.js --query="http://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=2017[DP] AND FRANCE[Affiliation]&usehistory=y&retmode=json&retmax=1000" --source="pubmed" --output="out/pubmed" --ext="gz"
```

Note : Pour la source Pubmed, une query renvoie un corpus de document XML

## Fonctionnement ##

### Moissonnage via liste d'ids ###

Liste des paramètres à renseigner :

* --ids
* --source
* --output
* --ext
* --conf

Note : Pour le moissonnage via liste d'ids, c'est un fichier de configuration qui est utilisé pour passer l'ensemble des paramètres liés à l'API interrogée : --conf=conf.json

#### Exemples de fichier ####

##### Conditor #####

```json
{
  "access_token": "monToken",
  "pattern": "https://api.conditor.fr/v1/records/:id/tei"
}
```
De cette manière, les données utilisées seront :

* le token : "monToken"
* le pattern : "https://api.conditor.fr/v1/records/:id/tei"


##### CrossRef #####

```json
{
  "userAgent": "",
  "usr": "",
  "pwd": "",
  "pattern": "https://api.crossref.org/works/:id.xml"
}
```
De cette manière, les données utilisées seront :

* le usr : user (facultatif)
* le pwd : password (facultatif)
* le pattern : "https://api.crossref.org/works/:id.xml"
* le userAgent: [User-Agent pour CrossRef](https://github.com/CrossRef/rest-api-doc#etiquette)

##### Hal #####

_Moissonnage via liste d'ids non-implémenté._

##### Pubmed #####

_Moissonnage via liste d'ids non-implémenté._

### Moissonnage via query ###

Liste des paramètres à renseigner :

* --query
* --source
* --output
* _--ext *_

Note : Pour le moissonnage via query, toutes les données nécessaires doivent être présentes dans l'URL de --query

_* Pour la source Pubmed, une query renvoie un corpus de document XML. Il faut alors préciser le format souhaité pour l'archive_


#### Exemples de query ####

##### Conditor #####

```bash
"https://api.conditor.fr/v1/records?q=%22doi:*%22&page_size=1000&includes=doi&scroll=1m"
```

##### CrossRef #####

_Moissonnage via query non-implémenté._

##### Hal #####

```bash
"https://api.archives-ouvertes.fr/search/?wt=json&q=structCountry_s:(fr OR gf OR gp OR mq OR re OR yt OR bl OR mf OR pf OR pm OR wf OR nc)&fq=producedDateY_i:2014&fl=docid,halId_s,label_xml&sort=docid+desc&rows=1000&cursorMark=*"

"http://api.archives-ouvertes.fr/ref/structure?fq=country_s:fr&fl=*&q=*&rows=1000&sort=docid asc&cursorMark=*"
```

##### Pubmed #####

```bash
"http://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=2017[DP] AND FRANCE[Affiliation]&usehistory=y&retmode=json&retmax=1000"
```
