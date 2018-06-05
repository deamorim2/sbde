# 2.1 Sistema de Banco de Dados Espaciais

Um exemplo de Sistema de Banco de Dados Espaciais é o Sistema Gerenciador de Banco de Dados Objeto-Relacional distribuído [PostgreSQL](http://www.postgresql.org/) e a extensão espacial [PostGIS](https://postgis.net/).

Assim como o PostGIS, o Oracle Spatial é a extensão espacial para o Sistema Gerenciador de Banco de Dados Objeto-Relacional Oracle. Mas o que isso significa? O que torna um sistema de banco de dados convencionais em um sistema de banco de dados espaciais?

A resposta curta, é...

Os sistemas de bancos de dados espaciais armazenam e manipulam objetos espaciais como qualquer outro objeto no sistema de banco de dados.

A seguir, abordaremos brevemente a evolução dos sistema de dados espaciais e, em seguida, analisaremos três aspectos que associam dados espaciais a um sistema de banco de dados - tipos de dados, índices e funções.

Os tipos de dados espaciais referem-se aos formatos vetorial e raster. Ponto, Linha e Polígono são exemplos de dados  espaciais vetoriais. Uma imagem de satélite ou um modelo digital de elevação são exemplos de dados espaciais do tipo raster, chamados também de dados matriciais.

A indexação espacial multidimensional é utilizada para processamento eficiente de operações espaciais.

As funções espaciais, utilizadas em SQL, são para consultas de propriedades geométricas e relacionamentos espaciais.

Combinados, tipos de dados espaciais, índices espaciais e funções espaciais, fornecem uma estrutura flexível para desempenho e análise espaciais otimizadas, a partir de um grande volume de dados ou por meio de consultas complexas.

Nas implementações de Sistemas de Informações Geográficas (SIG) de primeira geração, todos os dados espaciais são armazenados em arquivos planos e é necessário um software SIG especial para interpretar e manipular os dados. Esses sistemas de gerenciamento de primeira geração são projetados para atender às necessidades dos usuários, onde todos os dados necessários estão dentro do domínio organizacional. Eles são sistemas proprietários e autônomos construídos especificamente para manipulação de dados espaciais.

Os sistemas espaciais de segunda geração armazenam alguns dados em bancos de dados relacionais (geralmente o "atributo" ou partes não espaciais), mas ainda não possuem a flexibilidade oferecida com a integração direta.

Os verdadeiros sistemas de bancos de dados espaciais nasceram quando as pessoas começaram a tratar os recursos espaciais como objetos de banco de dados de primeira classe.

Os sistemas de banco de dados espaciais integram dados espaciais em um sistema de banco de dados objeto-relacional. O foco das aplicações muda do SIG para o Sistema de Banco de Dados Espaciais.

![](https://github.com/deamorim2/sbde/blob/master/wiki/02/beginning.png)

Nota:
No Brasil, o termo Sistema de Banco de Dados Geográficos é bem mais difundido que o termo Sistema de Banco de Dados Espaciais. Porém, os Sistemas de Banco de Dados Espaciais podem ser utilizado em aplicações além do mundo geográfico. Os bancos de dados espaciais podem ser usados para gerenciar dados relacionados à anatomia do corpo humano, circuitos integrados de grande escala, estruturas moleculares, campos eletromagnéticos, entre outros.

## 2.1.1 Tipos de dados espaciais

Um banco de dados convencionais possui dados básicos como dos tipos texto, número e data. Um banco de dados espacial adiciona tipos adicionais (espaciais) para representar recursos geográficos. Esses tipos de dados espaciais abstraem e encapsulam estruturas espaciais, como limites e dimensões. Em muitos aspectos, os tipos de dados espaciais podem ser entendidos simplesmente como formas.

![](https://github.com/deamorim2/sbde/blob/master/wiki/02/hierarchy.png)

Os tipos de dados espaciais são organizados em uma hierarquia de tipos. Cada sub-tipo herda a estrutura (atributos) e o comportamento (métodos ou funções) do seu super-tipo.

## 2.1.2 Índices espaciais e Retângulo Envolvente Mínimo R.E.M (Bounding Boxes)

Um banco de dados comum fornece "métodos de acesso" (comumente conhecidos como índices) para permitir acesso rápido e aleatório a subconjuntos de dados. A indexação de tipos padrão (números, textos, datas) geralmente é feita com índices de árvore binária. Uma árvore binária divide os dados usando a ordem de classificação natural para colocar os dados em uma árvore hierárquica.

A ordem de classificação natural de números, textos e datas é simples de determinar, onde cada valor é menor do que, maior ou igual a qualquer outro valor. Mas, como os polígonos podem se sobrepor, podem estar contidos um em relação ao outro e estão dispostos em um espaço bidimensional (ou mais), uma árvore binária não pode ser usada para indexá-los eficientemente. Os bancos de dados espaciais fornecem um "índice espacial" que, em vez disso, responde a seguinte pergunta: "quais objetos estão dentro deste retângulo envolvente mínimo em particular?".

Um Retângulo Envolvente Mínimo é o menor retângulo, que esteja paralelo aos eixos de coordenadas, capaz de conter uma determinada característica.

![](https://github.com/deamorim2/sbde/blob/master/wiki/02/boundingbox.png)

Retângulos Envolventes Minimos são usados porque respondem facilmente a pergunta "está A dentro de B?". Caso essa pergunta seja feita a polígonos irregulares com muitos vértices, este tipo de processamento computacional pode demorar muito se comparado a polígonos regulares, como é o caso dos retângulos envolventes mínimos. Até mesmo os polígonos e as linhas mais complexas podem ser representados por um simples retângulo envolvente mínimo.

A principal utilidade dos índices é sua resposta rápida. Por isso, ao invés de fornecerem resultados exatos, como fazem as árvores binárias, os índices espaciais fornecem resultados aproximados. A pergunta "quais linhas estão dentro desse polígono?" deve ser interpretada a partir de índice espacial como "quais linhas possuem caixas de delimitação contidas dentro da caixa delimitadora deste polígono?"

Existem vários tipos de índices espaciais implementados em sistema de bancos de dados espaciais. A implementação mais comum é a Árvore-R ([R-Tree](http://en.wikipedia.org/wiki/R-tree)), usado no PostGIS, mas também há [Quadtrees](http://en.wikipedia.org/wiki/Quadtree), Árvore-k-d ([k-d-trees](https://en.wikipedia.org/wiki/K-d_tree)), [Grid Files](http://en.wikipedia.org/wiki/Grid_(spatial_index)), entre outros.

## 2.1.3 Funções Espaciais

Para manipular dados durante uma consulta, um banco de dados comum fornece funções como concatenar texto, fazer cálculos e extrair informações de datas. Um banco de dados espacial fornece um conjunto completo de funções para análise de componentes geométricos, determinação de relações espaciais e manipulação de geometrias. Essas funções espaciais servem de base para qualquer projeto espacial.

A maioria das funções espaciais podem ser agrupadas em uma das seguintes categorias:

* Conversão: funções de conversão entre geometrias e formatos de dados externos

* Gerenciamento: funções que gerenciam informações sobre tabelas espaciais e administração dos dados em PostGIS

* Recuperação: Funções que recuperam propriedades e medidas de uma Geometria

* Comparação: Funções que comparam duas geometrias em relação à sua relação espacial

* Geração: funções que geram novas geometrias a partir de outros formatos de dados

A lista de funções espaciais é bem ampla. As mais comumente implementadas são definida pela Open Geospatial Consortium ([OGC](http://www.opengeospatial.org/)) a partir da especificação "OpenGIS Implementation Specification for Geographic information-Simple feature access" ([SFSQL](http://www.opengeospatial.org/standards/sfa)) ou pela ISO a partir da "ISO/IEC 13249-3:2016 Part 3: Spatial" ([SQLMM](https://www.iso.org/standard/60343.html)). Mas nada impede que os softwares de sistema de banco de dados espaciais adotem as suas própras funções espaciais. No caso do PostGIS, ele possui várias funções espaciais implementadas pela OGC/ISO, bem como funções espaciais próprias.

# 2.2 O que é o PostGIS?

O [PostGIS](https://postgis.net/) transforma o Sistema de Gerenciamento de Banco de Dados [PostgreSQL](http://www.postgresql.org/) em um banco de dados espaciais, adicionando suporte para os três recursos: tipos espaciais, índices espaciais e funções espaciais. Como ele é criado no PostgreSQL, o PostGIS herda automaticamente recursos importantes de sistema de banco de dados, bem como utiliza padrões abertos (SQL98, SFSQL/SQLMM) em sua implementação.

## 2.2.1 Mas o que é PostgreSQL?

O PostgreSQL é um poderoso sistema de gerenciamento de banco de dados objeto-relacional (ORDBMS). É lançado sob uma licença de estilo BSD e, portanto, é um software livre e de código aberto. Tal como acontece com muitos outros programas de código aberto, o PostgreSQL não é controlado por nenhuma empresa, mas tem uma comunidade global de desenvolvedores e empresas para desenvolvê-lo.

Desde o início, o PostgreSQL foi projetado para trabalhar com extensão, com capacidade de adicionar novos tipos de dados, funções e métodos de acesso em tempo de execução. Por isso, a extensão PostGIS pode ser desenvolvida por uma equipe de desenvolvimento separada, mas ainda assim continua firmemente integrada ao sistema de banco de dados do PostgreSQL.

### 2.2.1.1 Por que escolher o PostgreSQL?

Uma questão comum de pessoas familiarizadas com bancos de dados de código aberto é "Por que o PostGIS não foi construído no MySQL?".

O PostgreSQL possui:

* Confiabilidade comprovada e integridade transacional por padrão (ACID)

* Suporte a padrões SQL (SQL92 completo)

* Tipos de Extensão e funções de extensão plugáveis

* Modelo de desenvolvimento orientado para a comunidade

* Sem limites nos tamanhos das colunas para suportar grandes objetos geométricos

* Estrutura genérica do índice (GiST) para permitir o índice R-Tree

* Fácil implementação de funções personalizadas

Combinado, o PostgreSQL fornece um caminho de desenvolvimento muito fácil para adicionar novos tipos espaciais. No mundo proprietário, apenas o Illustra (agora o Informix Universal Server) permite uma extensão tão fácil. Isso não é coincidência, Illustra é um software proprietário que aproveitou a base de código original do PostgreSQL da década de 1980.

Como o desenvolvimento para adicionar tipos ao PostgreSQL era tão direto, fazia sentido começar lá. Quando o MySQL lançou tipos espaciais básicos na versão 4.1, a equipe PostGIS examinou seu código e esse exercício reforçou a decisão original de usar o PostgreSQL. Como os objetos espaciais MySQL tiveram que ser construídos sobre tipos do tipo texto como um caso especial, o código MySQL foi espalhado por todo o código base. O desenvolvimento do PostGIS 0.1 levou menos de um mês. Fazer um "MyGIS" 0.1 teria demorado muito e, como tal, talvez nunca tivesse visto a luz do dia.

### 2.2.1.2 Por que não usar Shapefiles?

O shapefile (e outros formatos de arquivo) tem sido a maneira padrão de armazenar e interagir com dados espaciais desde que o primeiro software de SIG foi escrito. No entanto, esses arquivos de sistema apresentam as seguintes desvantagens:

* **Os arquivos requerem software especial para ler e escrever**: O SQL é uma abstração para acesso e análise de dados aleatórios. Sem essa abstração, você precisará escrever todo o código de acesso e análise você mesmo.

* **Usuários simultâneos podem causar corrupção**: Embora seja possível escrever código extra para garantir que múltiplas escritas no mesmo arquivo não corrompam os dados, você ainda terá que resolver o problema de desempenho e terá que escrever o melhor código de um sistema de banco de dados. Por que não usar apenas um banco de dados padrão?

* **Perguntas complicadas requerem um software complicado para responder**: Perguntas complexas, como associações ou agregações espaciais, que são realizadas ??em uma linha de SQL no sistema de banco de dados espaciais, terá que conter centenas de linhas de código de programação para responder a mesma coisa utilizando arquivos de sistema.

A maioria dos usuários do PostGIS configuram sistemas em que vários aplicativos acessam os mesmos dados, de modo que ter um método de acesso SQL padrão simplifica a implantação e o desenvolvimento da solução. Alguns usuários trabalham com grandes conjuntos de dados. Com arquivos de sistema, muitas vezes esses dados tem que ser segmentados em vários arquivos, mas em um sistema de banco de dados esses dados podem ser armazenados em uma única tabela.

Em resumo, o acesso simultâneo aos dados por vários usuários, consultas ad hoc complexas e alto desempenho em grandes conjuntos de dados são o que separa os sistemas de bancos de dados espaciais dos sistemas baseados em arquivos.

### 2.2.1.3 Por que usar o Geopackage?

GeoPackage (GPKG) é um formato de dados espaciais aberto, não proprietário, independente de plataforma e é baseado em padrões para sistema de informações geográficas implementado como um contêiner de banco de dados SQLite. Definido pela Open Geospatial Consortium (OGC) com o apoio dos militares dos EUA e publicado em 2014, o GeoPackage recebeu amplo apoio  de várias organizações governamentais, comerciais e de código aberto.

Um GeoPackage é construído como um arquivo de banco de dados SQLite 3 estendido (*.gpkg) que contém tabelas de dados e metadados com definições especificadas, restrições de integridade, limitações de formato e restrições de conteúdo. O padrão GeoPackage descreve um conjunto de convenções (requisitos) para armazenar dados nos formatos vetoriais e matriciais em várias escalas, esquemas e metadados. Um GeoPackage pode ser estendido usando as regras de extensão conforme definido na cláusula 2.3 do padrão. O padrão OGC GeoPackage especifica um conjunto de extensões aprovadas pelos membros OGC no Anexo F.

O GeoPackage foi projetado para ser o mais leve possível, compartilhado em um único arquivo e pronto para ser usado. Isso o torna adequado para aplicações mobile em modo offline e para compartilhamento rápido por meio de armazenamento em nuvem, unidades USB e etc. O formato Geopackagepossui índices espaciais RTree em SQLite que melhoram a performance em consultas espaciais se comparados aos formatos tradicionais de arquivos geoespaciais.

Se comparado com o shapefile, o geopackage suporta tipos de dados não espaciais como inteiro, real, texto, blob, data, valores nulos, bem como não possui limitação no comprimento do nome da coluna das tabelas, que no shapefile possui limitação de 10 caracteres. Mas, uma das principais diferenças entre o Shapefile e o Geopackage é que o shapefile possui limite em sua capacidade de armazenamento de 2 GB, enquanto o limite do Geopakcage é bem superior: 140 mil GB.

## 2.2.3 Um breve histórico do PostGIS

Em maio de 2001, a [Refractions Research](http://www.refractions.net/) lançou a primeira versão do PostGIS. O PostGIS 0.1 teve objetos, índices e umas poucas funções. O resultado foi um banco de dados adequado para armazenamento e para recuperação de dados, mas não adequado para análise de dados.

À medida que o número de funções aumentou, a necessidade de organização tornou-se clara. A especificação "OpenGIS Implementation Specification for Geographic information-Simple feature access" (SFSQL) forneceu a estrutura necessária com diretrizes para nomeação e requisitos de funções.

Com o suporte do PostGIS para análise simples e junções espaciais, o Mapserver tornou-se o primeiro aplicativo externo a fornecer visualização de dados a partir do sistema de banco de dados espaciais.

Nos anos seguintes, o número de funções do PostGIS cresceu bastante, mas seu poder de processamento permaneceu limitado. Muitas das funções mais interessantes como ST_Intersects (), ST_Buffer () ou ST_Union () foram muito difíceis de serem codificar. Partindo-se do zero, escrever esses códigos levariam anos para ficarem prontos. Felizmente, surgiu um segundo projeto, o “Geometry Engine, Open Source” ou [GEOS](http://trac.osgeo.org/geos). A biblioteca GEOS fornece os algoritmos necessários para implementar as especificações da SFSQL. Ao associar-se ao GEOS, o PostGIS forneceu suporte completo para o SFSQL a partir da versão 0.8.

Surgiu um outro problema com o aumento da capacidade de dados do PostGIS: a representação usada para armazenar a geometria se mostrou relativamente ineficiente. Para objetos pequenos, como pontos e pequenas linhas, os metadados na representação das feições geométricas compreendiam até 300% desses dados. Por razões de desempenho, foi necessário enxugar a representação. Os custos de leitura e processamento desses dados foram bastante reduzidos ao encolher o cabeçalho dos metadados e as dimensões necessárias.

No PostGIS 1.0, esta nova representação, mais rápida e leve, tornou-se o padrão. As atualizações mais recentes do PostGIS trabalharam na expansão da conformidade dos padrões, adicionando suporte para geometrias baseadas em curva e funções especificadas pelo padrão SQLMM (ISO/IEC 13249-3:2016 Part 3: Spatial).

Com foco contínuo no desempenho, o PostGIS 1.4 melhorou significativamente a velocidade de processamneto das funções e consultas que utilizam geometrias.

## 2.2.4 Quem usa PostGIS?

Para uma lista completa de estudos de caso, veja a página de [estudos de caso que utilizam PostGIS](http://postgis.net/casestudy).

## 2.2.5 Quais aplicativos oferecem suporte ao PostGIS?

A PostGIS tornou-se um banco de dados espacial amplamente utilizado, e o número de programas de terceiros que oferecem suporte ao armazenamento e recuperação de dados usando essa extensão também aumentou. [programas que oferecem suporte ao PostGIS](http://trac.osgeo.org/postgis/wiki/UsersWikiToolsSupportPostgis) incluem softwares de código aberto e proprietário em sistemas servidor e desktop.

### LEITURAS COMPLEMENTARES

Casanova, M., et. al.: Bancos de Dados Geográficos. Cap. 1, 6, 8 e 11. MundoGEO. 2005

Yeung, A., Hall, G.: Spatial Database Systems. GeoJournal Library, vol 87. Partes 1 e 2. Springer, Heidelberd (2007)

