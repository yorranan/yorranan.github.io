
As instruções indicadas nesse documento são baseadas na wiki da ferramenta Skosmos, então para mais informação acesse-a [aqui](https://github.com/NatLibFi/Skosmos/wiki). Esse tutorial foi baseado em distribuições baseadas em Debian.

# Documentação Input-LASeD

## Instalando Apache Jena Fuseki

### Instalando Java 11 ou superior

Atualize os pacotes:

```
sudo apt update && sudo apt upgrade
```

Um dos pré-requisitos para o Fuseki é o Java 11 ou superior, para isso é necessário apenas o *Java Runtime Environment*  (JRE), mas  o *Java Development Kit*  (JDK) também pode ser utilizado.

> A partir da versão 5.0.0 do Fuseki é necessário instalar a versão 17 do Java devido a incompatibilidades.

~~~~
sudo apt install default-jre-headless
~~~~
Caso exista mais de uma versão do Java rodando na máquina pode se listar e definir a versão desejada para tal, uma vez que é necessário estar com a versão 11 ou superior do Java. Pode ser usado os comando para listar as opções disponíveis com a *flag* `--list` ou apenas`-l`, logo após usando a *flag* `--set` pode se definir a versão que será utilizada como padrão.

```
sudo update-java-alternatives -l
sudo update-java-alternatives --set java-1.11.0-openjdk-amd64
```

## Intalando Fuseki

Feito o processo de instalação do Java podemos instalar o Fuseki.

Note que é possível encontrar os arquivos de instalação do Fuseki neste [link](https://archive.apache.org/dist/jena/binaries/). Escolhido o arquivo da versão que deseja fazer o download, copie o link e utilize o utilitário `wget` para fazer o download. 

Visite a página de downloads e capture o link dela: https://jena.apache.org/download/

```
cd ~
wget https://archive.apache.org/dist/jena/binaries/apache-jena-fuseki-<versão>.tar.gz
```

Como no Debian do servidor não existia o wget utilizou-se o `curl`:

```
	 curl -o fuseki https://archive.apache.org/dist/jena/binaries/apache-jena-fuseki-4.9.0.tar.gz
	 tar xvf fuseki
```

Após isso é necessário realizar a instalação descompactando os arquivos baixados e criando um *link* simbólico. Isso deve ser feito dentro do diretório `/opt`, por esse motivo foi usando o comando *change directory* `cd`.

```
cd /opt
sudo tar xzf ~/apache-jena-fuseki-<versão>.tar.gz
sudo ln -s apache-jena-fuseki-<versão> fuseki
```
Verificando se tudo corretamente instalando:

```
cd /opt/fuseki/
./fuseki-server --help
./fuseki-server --version
```

Para o correto funcionamento e maior segurança é necessário criar um usuário comum denominado `fuseki`

`sudo adduser --system --home /opt/fuseki --no-create-home fuseki`

- O código Fuseki (a distribuição do servidor) entra em , como acima (na verdade, um link simbólico) `/opt/fuseki`
- as bases de dados estão em `/var/lib/fuseki`
- os arquivos de log vão em `/var/log/fuseki`
- arquivos de configuração vão em `/etc/fuseki`

```
# diretórios de banco de dados
cd /var/lib
sudo mkdir -p fuseki/{backups,databases,system,system_files}
sudo chown -R fuseki fuseki

# logs
cd /var/log
sudo mkdir fuseki
sudo chown fuseki fuseki

# configs
cd /etc
sudo mkdir fuseki
sudo chown fuseki fuseki

#link simbólico
cd /etc/fuseki
sudo ln -s /var/lib/fuseki/* .
sudo ln -s /var/log/fuseki logs
```

### Configuração para o fuseki iniciar no boot do sistema

```
sudo nano /etc/systemd/system/fuseki.service
```
Insira nesse arquivo:

```
[Unit]
Description=Fuseki
[Service]
Environment=FUSEKI_HOME=/opt/fuseki
Environment=FUSEKI_BASE=/etc/fuseki
Environment=JVM_ARGS=-Xmx2G
User=fuseki
ExecStart=/opt/fuseki/fuseki-server
Restart=on-failure
RestartSec=15
[Install]
WantedBy=multi-user.target
```

> O argumento `Environment=JVM_ARGS=-Xmx1G` define o consumo de memória da JVM
Após isso: 

```
sudo systemctl start fuseki
sudo systemctl status fuseki
```

Se ao rodar `status` não for iniciado corretamente verifique a situação usando:

```
cat /var/log/fuseki/stderrout.log
sudo journalctl -xe
```

Sendo tudo correto:

`sudo systemctl enable fuseki`

O retorno deve ser similar a esse: `Created symlink /etc/systemd/system/multi-user.target.wants/fuseki.service → /etc/systemd/system/fuseki.service.

Note that if you are running Fuseki within a VirtualBox VM and want to use the browser from the host machine, a port forwarding rule needs to be added to the VirtualBox settings (Network -> Port Forwarding...). The rule is: Name="fuseki", Protocol="TCP", Host Port="3030", Guest Port="3030", other fields can be left blank. Confirm with OK. This can be done also while the VM is running. You will also need to tell Fuseki to allow management operations for non-localhost access by commenting out the line `/$/** = anon` in the security configuration `/etc/fuseki/shiro.ini` and restarting Fuseki. Note that this is potentially dangerous if you open up Fuseki URLs to the world, since anyone will then be able to manage your datasets.

Tudo certo acesse: http://localhost:3030/

No painel do fuseki faça:

1. Crie um dataset nomeado skosmos do tipo persistente
2. Deixe em branco o espaço de nome do grafo
3. Faça upload do RDF
4. Verifique em info se é possível carregar as tuplas
Se não for possível realizar a criação do banco de dados através da ferramenta visual teste:
`curl --data "dbName=skosmos&dbType=tdb2" http://localhost:3030/$/datasets`

If you get no error, the operation was successful. To verify, you can check that the directory `/var/lib/fuseki/databases/skosmos/` exists.

### Configurando index Fuseki

```
sudo service fuseki stop
sudo nano /etc/fuseki/configuration/skosmos.ttl
```

Dentro de skosmos.ttl copie:

```
@prefix :      <http://base/#> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix tdb2:  <http://jena.apache.org/2016/tdb#> .
@prefix ja:    <http://jena.hpl.hp.com/2005/11/Assembler#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .
@prefix fuseki: <http://jena.apache.org/fuseki#> .
@prefix text:  <http://jena.apache.org/text#> .
@prefix skos:  <http://www.w3.org/2004/02/skos/core#> .

ja:DatasetTxnMem  rdfs:subClassOf  ja:RDFDataset .
ja:MemoryDataset  rdfs:subClassOf  ja:RDFDataset .
ja:RDFDatasetOne  rdfs:subClassOf  ja:RDFDataset .
ja:RDFDatasetSink  rdfs:subClassOf  ja:RDFDataset .
ja:RDFDatasetZero  rdfs:subClassOf  ja:RDFDataset .

tdb2:DatasetTDB  rdfs:subClassOf  ja:RDFDataset .
tdb2:DatasetTDB2  rdfs:subClassOf  ja:RDFDataset .

tdb2:GraphTDB  rdfs:subClassOf  ja:Model .
tdb2:GraphTDB2  rdfs:subClassOf  ja:Model .

<http://jena.hpl.hp.com/2008/tdb#DatasetTDB>
    rdfs:subClassOf  ja:RDFDataset .

<http://jena.hpl.hp.com/2008/tdb#GraphTDB>
    rdfs:subClassOf  ja:Model .

text:TextDataset
    rdfs:subClassOf  ja:RDFDataset .

:service_tdb_all  a               fuseki:Service ;
    rdfs:label                    "TDB2+text skosmos" ;
    fuseki:dataset                :text_dataset ;
    fuseki:name                   "skosmos" ;
    fuseki:serviceQuery           "query" , "" , "sparql" ;
    fuseki:serviceReadGraphStore  "get" ;
    fuseki:serviceReadQuads       "" ;
    fuseki:serviceReadWriteGraphStore "data" ;
    fuseki:serviceReadWriteQuads  "" ;
    fuseki:serviceUpdate          "" , "update" ;
    fuseki:serviceUpload          "upload" .

:text_dataset a text:TextDataset ;
    text:dataset :tdb_dataset_readwrite ;
    text:index :index_lucene . 

:tdb_dataset_readwrite
    a tdb2:DatasetTDB2 ;
    # tdb2:unionDefaultGraph true ;
    tdb2:location  "/etc/fuseki/databases/skosmos" .

:index_lucene a text:TextIndexLucene ;
    text:directory <file:/etc/fuseki/databases/skosmos/text> ;
    text:entityMap :entity_map ;
    text:storeValues true .

# Text index configuration for Skosmos
:entity_map a text:EntityMap ;
    text:entityField      "uri" ;
    text:graphField       "graph" ;
    text:defaultField     "pref" ;
    text:uidField         "uid" ;
    text:langField        "lang" ;
    text:map (
         # skos:prefLabel
         [ text:field "pref" ;
           text:predicate skos:prefLabel ;
           text:analyzer [ a text:LowerCaseKeywordAnalyzer ]
         ]
         # skos:altLabel
         [ text:field "alt" ;
           text:predicate skos:altLabel ;
           text:analyzer [ a text:LowerCaseKeywordAnalyzer ]
         ]
         # skos:hiddenLabel
         [ text:field "hidden" ;
           text:predicate skos:hiddenLabel ;
           text:analyzer [ a text:LowerCaseKeywordAnalyzer ]
         ]
         # skos:notation
         [ text:field "notation" ;
           text:predicate skos:notation ;
           text:analyzer [ a text:LowerCaseKeywordAnalyzer ]
         ]
     ) . 
```

Então:

```
sudo service fuseki start
```

Se tudo correr bem, isso criará um índice de jena-text Lucene em `/var/lib/fuseki/databases/skosmos/text`, ou seja, como um subdiretório do banco de dados TDB ao qual ele está vinculado.

### Instalando requirements do SKOSMOS

É necessário instalar o Apache2, PHP e bibliotecas necessárias:

```
	sudo apt install apache2 php7.4 libapache2-mod-php7.4 php7.4 php7.4-xsl php7.4-intl php7.4-mbstring php7.4-curl
```

Acesse: http://localhost

Se não estiver funcionando use:

```
sudo service apache2 start
sudo systemctl enable apache2
```


### Configurando o apache para o SKOMOS

Edite as configurações do apache através de:

```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

Dentro de `<VirtualHost *:80>` insira:

```
        <Directory /var/www/html>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>
```

Você também deve ativar os módulos Apache requerido pelo SKOSMOS:

```
sudo a2enmod rewrite
sudo a2enmod expires
sudo service apache2 restart
```

### Instalando o SKOMOS

A sequência de códigos abaixo instala a versão 2.17 do SKOSMOS disponível no repositório oficial dentro da pasta `/srv/`, também instala as dependências necessárias com o composer.
> Note que se houver mensagem de erro siga as instruções indicadas.
```shell
cd /srv
sudo mkdir Skosmos
sudo chown `whoami` Skosmos
sudo apt install git
git clone -b v2.17-maintenance https://github.com/NatLibFi/Skosmos.git /srv/Skosmos
cd /srv/Skosmos/
curl -sS https://getcomposer.org/installer | php
php composer.phar install --no-dev
	sudo ln -s /srv/Skosmos /var/www/html/Skosmos
```

### Configurando o SKOMOS

No arquivo `config.ttl `da pasta `/srv/Skosmos` deve passar a conter o arquivo de configuração, para mais detalhes sobre as definições utilizadas leia a documentação de [configuração](https://github.com/NatLibFi/Skosmos/wiki/Configuration) .

```
nano config.ttl
```

#### Trechos importantes


Caminho base: O caminho base deve ser adequar ao seu sistema, se estiver usando `localhost` mantenha-o assim, mas se estiver utilizando em um servidor ele deve ser adequar ao caminho real, indicando o domínio público e se há ou não certificado (`https`). Também pode editar o nome do serviço usando conforme sua necessidade.
```sh

    skosmos:serviceName "Skosmos" ;

    skosmos:baseHref "http://localhost/Skosmos/" .

```

Algumas das personalizações realizadas no sistema Input-LASeD:

```sh

:tbcc a skosmos:Vocabulary, void:dataset ;
    dc:title "Tesauro Brasileiro de Ciência da Computação"@pt;
    skosmos:shortName "TBCC" ;
    dc:subject :cat_general ;
    dc:type mdrtype:THESAURUS ;
    void:uriSpace "http://lod.unicentro.br/sparql" ;
    skosmos:language "en", "es", "pt" ;
    skosmos:defaultLanguage "pt" ;
    skosmos:showTopConcepts true ;
    skosmos:fullAlphabeticalIndex true ;
    skosmos:groupClass isothes:ConceptGroup ;
    void:sparqlEndpoint <http://localhost:3030/skosmos/sparql> ;
    skosmos:sparqlGraph <http://lod.unicentro.br/2017/DiretrizesSBC> .

:categories a skos:ConceptScheme;
    skos:prefLabel "Skosmos Vocabulary Categories"@en, "Categorias Skosmos"@pt
.

:cat_general a skos:Concept ;
    skos:topConceptOf :categories ;
    skos:inScheme :categories ;
    skos:prefLabel "Ciência da Computação"@pt,
        "Computer Sciences"@en
.
```
No caso de uso do Docker deve ser repassada a seguinte configuração:
```
:tbcc a skosmos:Vocabulary, void:dataset ;
    dc:title "Tesauro Brasileiro de Ciência da Computação"@pt,
        "Brazillian Thesaurus of Computer Science"@en;
    skosmos:shortName "tbcc" ;
    dc:subject :cat_general ;
    dc:type mdrtype:THESAURUS ;
    void:uriSpace "http://lod.unicentro.br/sparql" ;
    skosmos:language "en", "es", "pt" ;
    skosmos:defaultLanguage "pt" ;
    skosmos:showTopConcepts true ;
    skosmos:fullAlphabeticalIndex true ;
    skosmos:groupClass isothes:ConceptGroup ;
    void:sparqlEndpoint <http://fuseki-cache:80/skosmos/sparql>;
    skosmos:sparqlGraph <http://lod.unicentro.br/2017/DiretrizesSBC> .
```

> É necessário adicionar a *tag* sparqlGraph pois dermos POST na página do fuseki será realizada a procurado esse grafo.
> È necessário mudar para localhost 3030 se estiver fazendo sem docker compose

```
curl -I -X POST -H Content-Type:text/turtle -T ~/tesaurobrasileirocc.ttl -G http://localhost:9030/skosmos/data --data-urlencode graph=http://lod.unicentro.br/2017/DiretrizesSBC

```
Salve e instale o suporte as linguagens definidas:


```
	sudo apt install gettext
	sudo locale-gen en_GB.utf8
	sudo locale-gen es_ES.utf8
	sudo locale-gen pt_BR.utf8
```
Restarte o Apache

```
sudo service apache2 restart
```

Então acesse: http://localhost/Skosmos/

## Mudando textos de páginas

Se deseja realizar mudanças no front-end acesse a pasta `view`, dentro dela haverá os arquivos twig referentes as paginas. Além disso você pode editar os textos nos arquivos `.inc` que contém templates por padrão.



