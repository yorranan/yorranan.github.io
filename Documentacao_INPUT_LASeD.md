Como o projeto de TCC se baseia no Skosmos, foi realizado o download da versão 2.17 do projeto original. Devido as indicações de segurança dos desenvolvedores da plataforma, foi ro download da versão 2.17.
Durante o desenvolvimento desse trabalho estava em construção a versão 3.0 do Skosmos.
## Transferir repositório

Download do arquivo do respositório

```
wget https://github.com/NatLibFi/Skosmos/archive/refs/tags/v2.17.zip
```


Desconpacte o arquivo

```
unzip v2.17.zip
```

Depois, se desejar renomear o diretório onde os arquivos estão você pode criar um novo diretório com o nome desejado, no caso de exemplo foi nomeado `LASeD_Lib` e mover todos os arquivos dentro do diretório Skosmos para o novo diretório.  

```
mkdir LASeD_Lib && mv Skosmos-2.17/* ~/LASeD_Lib
```

## Configurando arquivos

Tiveram que ser realizadas modificações no arquivos antes de construir a imagem para atender as especificações. São elas:

Para: `dockerfiles/config/config-docker-compose.ttl` e `dockerfiles/config/config-docker.ttl` e `config.ttl.dist`

```turtle
@prefix void: <http://rdfs.org/ns/void#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dc: <http://purl.org/dc/terms/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix wv: <http://vocab.org/waiver/terms/norms> .
@prefix sd: <http://www.w3.org/ns/sparql-service-description#> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix skosmos: <http://purl.org/net/skosmos#> .
@prefix isothes: <http://purl.org/iso25964/skos-thes#> .
@prefix mdrtype: <http://publications.europa.eu/resource/authority/dataset-type/> .
@prefix : <#> .

# Skosmos main configuration

:config a skosmos:Configuration ;
    # SPARQL endpoint
    # a local Fuseki server is usually on localhost:3030
    skosmos:sparqlEndpoint <http://fuseki-cache:80/skosmos/sparql> ;
    # sparql-query extension, or "Generic" for plain SPARQL 1.1
    # set to "JenaText" instead if you use Fuseki with jena-text index
    skosmos:sparqlDialect "JenaText" ;
    # whether to enable collation in sparql queries
    skosmos:sparqlCollationEnabled false ;
    # HTTP client configuration
    skosmos:sparqlTimeout 20 ;
    skosmos:httpTimeout 5 ;
    # customize the service name
    skosmos:serviceName "Skosmos" ;
    # customize the base element. Set this if the automatic base url detection doesn't work. For example setups behind a proxy.
    # skosmos:baseHref "http://localhost/Skosmos/" ;
    # interface languages available, and the corresponding system locales
    skosmos:languages (
        [ rdfs:label "en" ; rdf:value "en_GB.utf8" ]
        [ rdfs:label "en-US" ; rdf:value "en_US.utf8" ]
        [ rdfs:label "es" ; rdf:value "es_ES.utf8" ]
        [ rdfs:label "pt" ; rdf:value "pt_PT.utf8" ]
        [ rdfs:label "pt-BR" ; rdf:value "pt_BR.utf8" ]
    ) ;
    # how many results (maximum) to load at a time on the search results page
    skosmos:searchResultsSize 20 ;
    # how many items (maximum) to retrieve in transitive property queries
    skosmos:transitiveLimit 1000 ;
    # whether or not to log caught exceptions
    skosmos:logCaughtExceptions false ;
    # set to TRUE to enable logging into browser console
    skosmos:logBrowserConsole false ;
    # set to a logfile path to enable logging into log file
    # skosmos:logFileName "" ;
    # a default location for Twig template rendering
    skosmos:templateCache "/tmp/skosmos-template-cache" ;
    # customize the css by adding your own stylesheet
    skosmos:customCss "resource/css/stylesheet.css" ;
    # default email address where to send the feedback
    skosmos:feedbackAddress "" ;
    # email address to set as the sender for feedback messages
    skosmos:feedbackSender "" ;
    # email address to set as the envelope sender for feedback messages
    skosmos:feedbackEnvelopeSender "" ;
    # whether to display the ui language selection as a dropdown (useful for cases where there are more than 3 languages) 
    skosmos:uiLanguageDropdown true ;
    # whether to enable the spam honey pot or not, enabled by default
    skosmos:uiHoneypotEnabled true ;
    # default time a user must wait before submitting a form
    skosmos:uiHoneypotTime 5 ;
    # plugins to activate for the whole installation (including all vocabularies)
    skosmos:globalPlugins () .

# Skosmos vocabularies

:tbcc a skosmos:Vocabulary, void:dataset ;
    dc:title "Tesauro Brasileiro de Ciência da Computação"@pt ;
    skosmos:shortName "tesaurobrasileirodeciênciadacomputação" ;
    dc:subject :cat_general ;
    dc:type mdrtype:THESAURUS ;
    void:uriSpace "http://lod.unicentro.br/sparql" ;
    skosmos:language "en", "es", "pt" ;
    skosmos:defaultLanguage "pt" ;
    skosmos:showTopConcepts true ;
    skosmos:fullAlphabeticalIndex true ;
    skosmos:groupClass isothes:ConceptGroup ;
    void:sparqlEndpoint <http://localhost:3030/skosmos/sparql> .

```


## Criando imagem

```
docker-compose up --build
```
