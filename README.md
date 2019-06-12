# Get started
A project to execute [SPARQL](https://www.w3.org/TR/sparql11-query/) queries from string, URL or multiple files using [RDF4J](http://rdf4j.org/).

* The user can execute **SPARQL queries** by
  * Passing a SPARQL **query string** in `-sp` param 
  * Providing a **URL** in `-f` param
  * Providing the **URL** of a **GitHub repository** containing `.rq` files to execute in `-f` param
  * Providing the **path to a directory** where the queries are stored in `.rq` text files and executed in the **alphabetical order** of their filename. 
  * A **YAML file** with multiple ordered queries.
* **Update**, **construct** and **select** operations supported.
* It is possible to optionally define **username** and **password** for the SPARQL endpoint.
* [data2services-transform-repository](https://github.com/MaastrichtU-IDS/data2services-transform-repository): example queries to transform biomedical data to the [BioLink](https://biolink.github.io/biolink-model/docs/) model for the [NCATS Translator program](https://ncats.nih.gov/translator).

# Pull

`BETA`: available on [DockerHub](https://hub.docker.com/r/vemonet/data2services-sparql-operations) the `latest` image is automatically built from latest branch `master` commit on [GitHub](https://github.com/MaastrichtU-IDS/data2services-sparql-operations).

```shell
docker pull vemonet/data2services-sparql-operations
```

# Build

You can also clone the [GitHub repository](https://github.com/MaastrichtU-IDS/data2services-sparql-operations) and build the docker image locally (**unecessary if you do** `docker pull`)

```shell
git clone https://github.com/MaastrichtU-IDS/data2services-sparql-operations
docker build -t vemonet/data2services-sparql-operations .
```
# Run

### Usage

```shell
docker run -it --rm vemonet/data2services-sparql-operations -h
```

### Select

On [DBpedia](http://dbpedia.org/sparql) using a SPARQL query string as argument.

```shell
docker run -it --rm vemonet/data2services-sparql-operations -op select \
  -sp "select distinct ?Concept where {[] a ?Concept} LIMIT 10" \
  -ep "http://dbpedia.org/sparql"
```

### Construct

On [graphdb.dumontierlab.com](http://graphdb.dumontierlab.com/) using GitHub URL to get the SPARQL query from a file.

```shell
docker run -it --rm vemonet/data2services-sparql-operations -op construct \
  -ep "http://graphdb.dumontierlab.com/repositories/ncats-red-kg" \
  -f "https://raw.githubusercontent.com/MaastrichtU-IDS/data2services-sparql-operations/master/src/main/resources/example-construct-pathways.rq" 
```

### Update

Multiple `INSERT` on [graphdb.dumontierlab.com](http://graphdb.dumontierlab.com/), using files in a repository from the local file system.

```shell
docker run -it --rm -v "/data/data2services-transform-repository/sparql/insert-biolink/drugbank":/data \
  vemonet/data2services-sparql-operations -f "/data" -op update \
  -ep "http://graphdb.dumontierlab.com/repositories/test/statements" \
  -un USERNAME -pw PASSWORD
```

* GraphDB requires to add `/statements` at the end of the endpoint URL for `INSERT`

### GitHub repository

We crawl the GitHub repository and execute every `.rq` file. See [example repository](https://github.com/MaastrichtU-IDS/data2services-sparql-operations/tree/master/src/main/resources/select-examples).

```shell
docker run -it --rm vemonet/data2services-sparql-operations \
  -op select -ep "http://dbpedia.org/sparql" \
  -f "https://github.com/MaastrichtU-IDS/data2services-sparql-operations/tree/master/src/main/resources/select-examples" 
```

### YAML

A YAML file can be used to provide multiple ordered queries. See [example](https://github.com/MaastrichtU-IDS/data2services-sparql-operations/blob/master/src/main/resources/example-queries.yaml).

```shell
docker run -it --rm vemonet/data2services-sparql-operations \
  -op select -ep "http://dbpedia.org/sparql" \
  -f "https://raw.githubusercontent.com/MaastrichtU-IDS/data2services-sparql-operations/master/src/main/resources/example-queries.yaml"
```



# Set variables

Variables can be set in the SPARQL queries using a `_` at the beggining: `?_myVar`. See example:

```SPARQL
PREFIX owl: <http://www.w3.org/2002/07/owl#>
CONSTRUCT { 
  ?class a <?_classType> .
} WHERE {
  GRAPH <?_graphUri> {
    [] a ?class .
  }
}
```

Execute with [2 variables](https://github.com/MaastrichtU-IDS/data2services-sparql-operations/blob/master/src/main/resources/example-select-variables.rq):

```shell
docker run -it --rm data2services-sparql-operations \
  -op select -ep "http://graphdb.dumontierlab.com/repositories/ncats-red-kg" \
  -f "https://raw.githubusercontent.com/MaastrichtU-IDS/data2services-sparql-operations/master/src/main/resources/example-select-variables.rq" \
  -var limit:10 graph:https://w3id.org/data2services/graph/biolink/date
```



# Examples

From [data2services-transform-repository](https://github.com/MaastrichtU-IDS/data2services-transform-repository), use a [federated query](https://github.com/MaastrichtU-IDS/data2services-transform-repository/blob/master/sparql/insert-biolink/drugbank/insert_drugbank_drug_CategoryOrganism.rq) to transform generic RDF generated by [AutoR2RML](https://github.com/amalic/AutoR2RML) and [xml2rdf](https://github.com/MaastrichtU-IDS/xml2rdf) to the [BioLink](https://biolink.github.io/biolink-model/docs/) model, and load it to a different repository.

```shell
# DrugBank
docker run -it --rm -v "$PWD/sparql/insert-biolink/drugbank":/data \
  data2services-sparql-operations \
  -f "/data" -un USERNAME -pw PASSWORD \
  -ep "http://graphdb.dumontierlab.com/repositories/ncats-test/statements" \
  -var serviceUrl:http://localhost:7200/repositories/test inputGraph:http://data2services/graph/xml2rdf outputGraph:https://w3id.org/data2services/graph/biolink/drugbank

# HGNC
docker run -it --rm -v "$PWD/sparql/insert-biolink/hgnc":/data \
  data2services-sparql-operations \
  -f "/data" -un USERNAME -pw PASSWORD \
  -ep "http://graphdb.dumontierlab.com/repositories/ncats-test/statements" \
  -var serviceUrl:http://localhost:7200/repositories/test inputGraph:http://data2services/graph/autor2rml outputGraph:https://w3id.org/data2services/graph/biolink/hgnc
```

