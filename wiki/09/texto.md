# 9. Dados Espaciais Geométricos

# 9.1. Introdução

Na seção anterior, nós carregamos uma variedade de dados. Antes de começarmos a brincar com nossos dados, vamos dar uma olhada em alguns exemplos mais simples.

>No pgAdmin, selecione novamente o banco de dados `sbde` e abra a ferramenta de consulta SQL.

>Cole este código SQL de exemplo na janela do Editor SQL pgAdmin (removendo qualquer texto que possa estar lá por padrão) e, em seguida, execute.

    CREATE TABLE geometries (id integer, name varchar, geom geometry);

    INSERT INTO geometries VALUES
    (1, 'Point', 'POINT(0 0)'),
    (2, 'Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
    (3, 'Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
    (4, 'PolygonWithHole', 'POLYGON((2 0, 12 0, 12 10, 2 10, 2 0),(3 1, 4 1, 4 2, 3 2, 3 1))'),
    (5, 'Collection', 'GEOMETRYCOLLECTION(POINT(2 1),POLYGON((5 3, 6 3, 6 4, 5 4, 5 3)))');

    SELECT id, name, ST_AsText(geom) FROM geometries;

![https://github.com/deamorim2/sbde/blob/master/wiki/09/start01.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/start01.png)

O exemplo acima cria uma tabela `geometries` e, em seguida, insere cinco geometrias: um ponto, uma linha, um polígono, um polígono com um furo e uma coleção. Finalmente, as linhas inseridas são SELECIONADAS e exibidas no painel Saída.

# 9.2. Tabelas de Metadados

Em conformidade com a especificação "OpenGIS Implementation Specification for Geographic information-Simple feature access" ([SFSQL](http://www.opengeospatial.org/standards/sfa)) ou pela ISO a partir da "ISO/IEC 13249-3:2016 Part 3: Spatial" ([SQLMM](https://www.iso.org/standard/60343.html)), o **PostGIS** fornece duas tabelas para rastrear e relatar os tipos de geometria disponíveis em um determinado banco de dados.

A primeira tabela, `spatial_ref_sys`, define todos os sistemas de referência espacial conhecidos pelo banco de dados e será descrita em maior detalhe posteriormente.

A segunda tabela (na verdade, uma view), `geometry_columns`, fornece uma listagem de todos os "recursos" (definidos como um objeto com atributos geométricos) e os detalhes básicos desses recursos.

![https://github.com/deamorim2/sbde/blob/master/wiki/09/table01.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/table01.png)

Vamos dar uma olhada na tabela geometry_columns em nosso banco de dados.

>Cole este comando na Ferramenta de consulta como antes:

    SELECT * FROM geometry_columns;

![https://github.com/deamorim2/sbde/blob/master/wiki/09/start08.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/start08.png)

* **f_table_catalog, f_table_schema e f_table_name** fornecem o nome completo da tabela de recursos que contém uma determinada geometria. Como o PostgreSQL não faz uso de catálogos, o f_table_catalog tenderá a ficar vazio.

* **f_geometry_column** é o nome da coluna que contém a feição geométrica (Obs.: para as tabelas de recursos com várias colunas de geometria, haverá um registro para cada).

* **coord_dimension** e **srid** definem a dimensão da geometria (2-, 3- ou 4-dimensional) e o identificador do sistema espacial de referência (Spatial Reference System - SRS) que se refere à tabela spatial_ref_sys, respectivamente.

* A coluna **type** define o tipo de geometria.

Nós já vimos os tipos Point e Linestring até agora.

Consultando a tabela `geometry_columns`, softwares ou bibliotecas GIS (clientes) podem determinar o que esperar ao recuperar dados e executar qualquer projeção, processamento ou renderização necessária sem precisar inspecionar cada geometria.

# 9.3. Representando Objetos Reais

A especificação [SFSQL](http://www.opengeospatial.org/standards/sfa), o padrão que originalmente orientou o desenvolvimento do PostGIS, define como um objeto do mundo real é representado. Ao tomar uma forma contínua e digitalizá-la em uma resolução fixa, conseguimos uma representação viável do objeto.

A [SFSQL](http://www.opengeospatial.org/standards/sfa) apenas lidou com representações bidimensionais. O PostGIS estendeu isso para incluir representações de 3 e 4 dimensões. Mais recentemente, a especificação [SQLMM](https://www.iso.org/standard/60343.html) definiu oficialmente sua própria representação em mais de uma dimensão.

Nossa tabela de exemplo contém uma mistura de diferentes tipos de geometria. Podemos coletar informações gerais sobre cada objeto usando funções que leem os metadados da geometria.

* ST_GeometryType (geometry) - retorna o tipo da geometria
* ST_NDims (geometry) - retorna o número de dimensões da geometria
* ST_SRID (geometry) - retorna o número do identificador de referência espacial da geometria (SRID)

***
    SELECT id, name, ST_GeometryType(geom), ST_NDims(geom), ST_SRID(geom)
    FROM geometries;

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_01.png)

## 9.3.1. Pontos

![https://github.com/deamorim2/sbde/blob/master/wiki/09/points.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/points.png)
 
Um **ponto** espacial representa um único local na Terra. Este ponto é representado por uma única coordenada (incluindo 2, 3 ou 4 dimensões). Os pontos são usados para representar objetos quando os detalhes exatos, como forma e tamanho, não são importantes na escala de destino. Por exemplo, cidades em um mapa do mundo podem ser descritas como pontos, enquanto um mapa de um único estado pode representar cidades como polígonos.

    SELECT ST_AsText(geom)
    FROM geometries
    WHERE name = 'Point';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_02.png)

Algumas das funções espaciais específicas para trabalhar com pontos são:

* ST_X (geometria) - retorna a ordenada em X
* ST_Y (geometria) - retorna a ordenada Y

Então, podemos ler as ordenadas de um ponto como este:

    SELECT ST_X(geom), ST_Y(geom)
    FROM geometries
    WHERE name = 'Point';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_03.png)

A tabela de cidades do Brasil `loc_cidade_p` é um conjunto de dados representado como pontos. A consulta SQL a seguir retornará a geometria associada a um ponto (na coluna ST_AsText).

    SELECT nome, ST_AsText(geom)
    FROM loc_cidade_p
    LIMIT 1;

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_04.png)

## 9.3.2. Linestrings

![https://github.com/deamorim2/sbde/blob/master/wiki/09/lines.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/lines.png)

Uma **Linestring** é um caminho entre pontos. Toma a forma de uma série ordenada de dois ou mais pontos. Estradas e rios são tipicamente representados como sequências de linhas. Dizemos que uma Linestring está fechada se iniciar e terminar no mesmo ponto. Diz-se que é simples se não se cruzar ou se tocar (exceto nos seus pontos finais se estiver fechado). Uma Linestring pode ser fechada e simples.

Os trechos rodoviários do Brasil foi carregada anteriormente no workshop. Este conjunto de dados contém detalhes como nome e tipo. Uma única rua do mundo real pode consistir em muitos Linestrings, cada um representando um segmento de estrada com atributos diferentes.

A consulta SQL a seguir retornará a geometria associada a uma Linestring (na coluna ST_AsText).

    SELECT ST_AsText(geom)
    FROM geometries
    WHERE name = 'Linestring';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_05.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_05.png)

Algumas das funções espaciais específicas para trabalhar com sequências de linhas são:

* ST_Length (geometry) - retorna o comprimento da Linestring
* ST_StartPoint (geometry) - retorna a primeira coordenada como um ponto
* ST_EndPoint (geometry) - retorna a última coordenada como um ponto
* ST_NPoints (geometry) - retorna o número de coordenadas na Linestring

Então, o comprimento da nossa Linestring é:

    SELECT ST_Length(geom)
    FROM geometries
    WHERE name = 'Linestring';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_06.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_06.png)

## 9.3.3. Polígonos

![https://github.com/deamorim2/sbde/blob/master/wiki/09/polygons.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/polygons.png)
 
Um polígono é uma representação de uma área. O limite externo do polígono é representado por um anel. Esse anel é uma linestring que é fechada e simples, conforme definido acima. Buracos dentro do polígono também são representados por anéis.
Polígonos são usados para representar objetos cujo tamanho e forma são importantes. Limites de Municípios, Unidades da Federação ou Terras Indígenas, são todos comumente representados como polígonos quando a escala é suficientemente alta para ver sua área. Estradas e rios podem ser representados como polígonos.

A consulta SQL a seguir retornará a geometria associada a um polígono (na coluna ST_AsText):

    SELECT ST_AsText(geom)
    FROM geometries
    WHERE name LIKE 'Polygon%';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_07.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_07.png)

***

**Nota:**
ao invés de usar um sinal `=` em nossa cláusula `WHERE`, estamos usando o operador `LIKE` para executar uma operação de correspondência de cadeia. Você pode estar acostumado com o símbolo `*` como um [glob](https://en.wikipedia.org/wiki/Glob_%28programming%29) para correspondência de padrões, mas no SQL o símbolo `%` é usado, junto com o operador LIKE para dizer ao sistema para fazer [globbing](https://en.wikipedia.org/wiki/Glob_%28programming%29).

***
>Abra o QGIS e insira a tabela geométrica `geometries` com o tipo espacial do tipo _Polygon_

![https://github.com/deamorim2/sbde/blob/master/wiki/09/qgis_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/qgis_01.png)

![https://github.com/deamorim2/sbde/blob/master/wiki/09/qgis_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/qgis_02.png)

O primeiro polígono tem apenas um anel. O segundo tem um “buraco” interior. A maioria dos sistemas gráficos inclui o conceito de “polígono”, mas os sistemas GIS são relativamente únicos em permitir que os polígonos tenham buracos explicitamente.
 
Algumas das funções espaciais específicas para trabalhar com polígonos são:

* ST_Area (geometry) - retorna a área dos polígonos
* ST_NRings (geometria) - retorna o número de anéis (geralmente 1, se há mais de 1 há buracos)
* ST_ExteriorRing (geometria) - retorna o anel externo como uma linestring
* ST_InteriorRingN (geometry, n) - retorna um anel interno especificado como uma linestring
* ST_Perimeter (geometry) - retorna o comprimento de todos os anéis

Podemos calcular a área de nossos polígonos usando a função de área:

    SELECT name, ST_Area(geom)
    FROM geometries
    WHERE name LIKE 'Polygon%';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_08.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_08.png)

Observe que o polígono com um furo tem uma área de 99 que é a área do polígono externo (um quadrado de 10x10 = 100) menos a área do furo (um quadrado de 1x1 = 1).

## 9.3.4. Coleções

Existem quatro tipos de **coleção**, que agrupam várias geometrias simples em conjuntos:

* MultiPoint - uma coleção de pontos
* MultiLineString - uma coleção de sequências de linestrings
* MultiPolygon - uma coleção de polígonos
* GeometryCollection - uma coleção heterogênea de qualquer geometria (incluindo outras coleções)

Coleções são outro conceito que aparece em software GIS mais do que em softwares gráficos genéricos.

Eles são úteis para modelar diretamente objetos do mundo real como objetos espaciais.

Por exemplo, como modelar um lote separado por uma passagem? Resposta: A partir de um MultiPolygon, com uma parte em ambos os lados da faixa de passagem, ou seja, duas geometrias não contínuas representadas por apenas um registro na tabela espacial.

Nossa coleção de exemplos contém um polígono e um ponto:

    SELECT name, ST_AsText(geom)
    FROM geometries
    WHERE name = 'Collection';

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_09.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_09.png)

Algumas das funções espaciais específicas para trabalhar com coleções são:

* ST_NumGeometries (geometry) - retorna o número de partes na coleção
* ST_GeometryN (geometry, n) - retorna a parte especificada
* ST_Area (geometry) - retorna a área total de todas as partes poligonais
* ST_Length (geometry) - retorna o comprimento total de todas as partes lineares

## 9.4. Entrada e Saída de Dados de Geometria

Dentro do banco de dados, as geometrias são armazenadas em disco em um formato usado apenas pelo programa PostGIS. Para que programas externos insiram e recuperem geometrias, eles precisam ser convertidos em um formato que outras aplicações possam entender. Felizmente, o PostGIS suporta a emissão e consumo de geometrias em um grande número de formatos:

Well-known text (WKT)
* ST_GeomFromText (text, srid) - retorna a geometria
* ST_AsText (geometry) - retorna texto
* ST_AsEWKT (geometry) - retorna texto

Well-known binary (WKB)
* ST_GeomFromWKB (bytea) - retorna geometria
* ST_AsBinary (geometria) - retorna bytea
* ST_AsEWKB (geometria) - retorna bytea

Geographic Mark-up Language (GML)
* ST_GeomFromGML (text) - retorna geometria
* ST_AsGML (geometry) - retorna texto

Keyhole Mark-up Language (KML)
* ST_GeomFromKML (text) - retorna geometria
* ST_AsKML (geometry) - retorna texto

GeoJSON
* ST_AsGeoJSON (geometry) - retorna texto

Scalable Vector Graphics ​​(SVG)
* ST_AsSVG (geometry) retorna texto

O uso mais comum de um construtor é transformar uma representação de texto de uma geometria em uma representação interna.

Observe que, além de um parâmetro de texto com uma representação geométrica, também temos um parâmetro numérico que fornece o SRID da geometria.

A consulta SQL a seguir mostra um exemplo de representação WKB (a chamada para encode() é necessária para converter a saída binária em um formato ASCII para impressão):

    SELECT encode(ST_AsBinary(ST_GeometryFromText('LINESTRING(0 0,1 0)')),'hex');

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_10.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_10.png)

Para os fins deste tutorial, continuaremos a usar o WKT para garantir que você possa ler e entender as geometrias que estamos visualizando.

No entanto, a maioria dos processos reais, como a visualização de dados em um aplicativo GIS, a transferência de dados para um serviço Web ou o processamento de dados remoto, o WKB é o formato escolhido.

Como o WKT e o WKB foram definidos na especificação [SFSQL](http://www.opengeospatial.org/standards/sfa), eles não manipulam geometrias de 3 ou 4 dimensões. Para esses casos, o PostGIS definiu os formatos EWKT (Extended Well Known Text) e EWKB (Extended Well Known Binary). Estes fornecem as mesmas capacidades de formatação de WKT e WKB com a dimensionalidade adicionada.

Aqui está um exemplo de uma linestring 3D em WKT:

    SELECT ST_AsText(ST_GeometryFromText('LINESTRING(0 0 0,1 0 0,1 1 2)'));

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_11.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_11.png)

Note que a representação do texto muda! Isso ocorre porque a rotina de entrada de texto do PostGIS não é muito restrita.

Exemplo:

* hex-encoded EWKB (EWKB) (Exclusivo do PostGIS)
* extended well-known text (EWKT) (Exclusivo do PostGIS)
* ISO standard well-known text (WKT) (SQLMM)

No lado da saída, a função ST_AsText é restrita e emite apenas texto padrão ISO (SQLMM).

Além da função ST_GeometryFromText, há muitas outras maneiras de criar geometrias a partir de um texto conhecido ou entradas formatadas semelhantes:

    -- Usando ST_GeomFromText com o parâmetro SRID
    SELECT ST_GeomFromText('POINT(2 2)',4326);

    -- Usando ST_GeomFromText sem o parâmetro SRID
    SELECT ST_SetSRID(ST_GeomFromText('POINT(2 2)'),4326);

    -- Usando a função ST_Make%
    SELECT ST_SetSRID(ST_MakePoint(2, 2), 4326);

    -- Usando a sintaxe do PostgreSQL por meio de _casting_ e ISO WKT (SQLMM)
    SELECT ST_SetSRID('POINT(2 2)'::geometry, 4326);

    -- Usando a sintaxe do PostgreSQL por meio de _casting_ e WKT estendido (EWKT)
    SELECT 'SRID=4326;POINT(2 2)'::geometry;

Além de disponibilizar em diversos formatos (WKT, WKB, GML, KML, JSON, SVG), o PostGIS também tem consome quatro tipos de dados (WKT, WKB, GML, KML). A maioria dos aplicativos usa as funções de criação de geometria WKT ou WKB, mas os outros também funcionam. Veja um exemplo que consome GML e gera JSON:

    SELECT ST_AsGeoJSON(ST_GeomFromGML('<gml:Point><gml:coordinates>1,1</gml:coordinates></gml:Point>'));

![https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_12.png](https://github.com/deamorim2/sbde/blob/master/wiki/09/pgadmin_sql_12.png)

## 9.5. Casting a partir de texto

As especificações do WKT que vimos até agora foram do tipo "texto" e estamos convertendo-as em "geometria" usando funções do PostGIS como ST_GeomFromText ().

O PostgreSQL inclui uma sintaxe curta que permite que os dados sejam convertidos de um tipo para outro, a sintaxe de conversão, oldata::newtype. Por exemplo, este SQL converte um dado do tipo número double em um texto.

    SELECT 0.9::text;

Menos trivialmente, esse SQL converte um texto WKT em uma geometria:

    SELECT 'POINT(0 0)'::geometry;

Uma coisa a notar sobre o uso de conversão para criar geometrias: a menos que você especifique o SRID, você obterá uma geometria com um SRID desconhecido. Você pode especificar o SRID usando o WKT “estendido” do PostGIS, que inclui um bloco SRID na frente:

    SELECT 'SRID=4326;POINT(0 0)'::geometry;

É muito comum usar a notação de conversão ao trabalhar com o WKT, além de colunas com dados geométricos e geográficos.

## 9.6 Lista de Funções

* ST_Area: Retorna a área da superfície se for um polígono ou um multi-polígono. Para área de tipo "geometria" está em unidades SRID. Para dados geográficos a área é em metros quadrados.
* ST_AsText: Retorna a representação de texto conhecido (WKT) da geometria / geografia sem metadados SRID.
* ST_AsBinary: Retorna a representação Well-Known Binary (WKB) da geometria / geografia sem metadados SRID.
* ST_EndPoint: Retorna o último ponto de uma geometria LINESTRING como um POINT.
* ST_AsEWKB: Retorna a representação Well-Known Binary (WKB) da geometria com metadados SRID.
* ST_AsEWKT: Retorna a representação WKT (Well-Known Text) da geometria com metadados SRID.
* ST_AsGeoJSON: Retorna a geometria como um elemento GeoJSON.
* ST_AsGML: retorna a geometria como um elemento da versão 2 ou 3 do GML.
* ST_AsKML: retorna a geometria como um elemento KML. Várias variantes. Versão padrão = 2, precisão padrão = 15.
* ST_AsSVG: Retorna uma Geometria em dados de caminho SVG, dados a geometria ou objeto geográfico.
* ST_ExteriorRing: Retorna uma linestring representando o anel externo da geometria do polígono. Retorna NULL se a geometria não é um polígono. Não funcionará com MULTIPOLYGON.
* ST_GeometryN: Retorna a 1ª geometria com base em 1 se a geometria for uma GEOMETRYCOLLECTION, MULTIPOINT, MULTILINESTRING, MULTICURVE ou MULTIPOLYGON. Caso contrário, retorne NULL.
* ST_GeomFromGML: assume como representação GML de entrada da geometria e gera um objeto de geometria PostGIS.
* ST_GeomFromKML: toma como entrada a representação KML da geometria e gera um objeto de geometria PostGIS.
* ST_GeomFromText: Retorna um valor de ST_Geometry especificado da representação Well-Known Text (WKT).
* ST_GeomFromWKB: Cria uma instância de geometria a partir de uma representação de geometria binária (WKB) conhecida e SRID opcional.
* ST_GeometryType: Retorna o tipo de geometria do valor ST_Geometry.
* ST_InteriorRingN: Retorna o enésimo anel de linha de linha interna da geometria do polígono. Retorna NULL se a geometria não for um polígono ou se N estiver fora do intervalo.
* ST_Length: retorna o comprimento 2d da geometria se for uma cadeia de linhas ou uma cadeia multilinha. geometria estão em unidades de referência espacial e geografia estão em metros (padrão esferóide).
* ST_NDims: Retorna a dimensão coordenada da geometria como um pequeno int. Os valores são: 2,3 ou 4.
* ST_NPoints: Retorna o número de pontos (vértices) em uma geometria.
* ST_NRings: Se a geometria é um polígono ou um multi-polígono, retorna o número de toques.
* ST_NumGeometries: Se geometry for um GEOMETRYCOLLECTION (ou MULTI *), retornará o número de geometrias, caso contrário, retornará NULL.
* ST_Perimeter: Retorna a medida do comprimento do limite de um valor ST_Surface ou ST_MultiSurface. (Polígono, multipolígono)
* ST_SRID: Retorna o identificador de referência espacial para o ST_Geometry, conforme definido na tabela spatial_ref_sys.
* ST_StartPoint: Retorna o primeiro ponto de uma geometria LINESTRING como um POINT.
* ST_X: Retorna a coordenada X do ponto ou NULL, se não estiver disponível. Entrada deve ser um ponto.
* ST_Y: Retorna a coordenada Y do ponto ou NULL, se não estiver disponível. Entrada deve ser um ponto.
