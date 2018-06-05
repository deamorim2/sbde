# 18. Dados Espaciais Geográficos

É muito comum ter dados em que as coordenadas são do tipo "geográficas" ou "latitude/longitude".

Ao contrário das coordenadas em Mercator, UTM ou Stateplane, as coordenadas geográficas não são coordenadas cartesianas.

Coordenadas geográficas não representam uma distância linear de uma origem como plotada em um plano. Em vez disso, essas coordenadas esféricas descrevem coordenadas angulares em um globo.

Nas coordenadas esféricas, um ponto é especificado pelo ângulo de rotação de um meridiano de referência (longitude) e o ângulo do equador (latitude).

![https://github.com/deamorim2/sbde/blob/master/wiki/18/cartesian_spherical.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/18/cartesian_spherical.jpg)

Você pode tratar coordenadas geográficas como coordenadas cartesianas aproximadas e continuar fazendo cálculos espaciais. No entanto, não faz sentido medir distâncias, comprimento ou área.

Como as coordenadas esféricas medem a distância angular, as unidades estão em “graus”. Além disso, os resultados aproximados de índices e testes do tipo verdadeiro/falso, como interseções e contagens, podem se tornar terrivelmente errados. A distância entre pontos aumenta à medida que áreas problemáticas, como os polos ou a linha de dados internacional, são abordadas.

Por exemplo, aqui estão as coordenadas de Los Angeles e Paris:

* Los Angeles: POINT (-118.4079 33.9434)
* Paris: PONTO (2.3490 48.8533)

A seguir, calcula-se a distância entre Los Angeles e Paris usando o padrão PostGIS Cartesian ST_Distance (geometria, geometria). Observe que o SRID de 4326 declara um sistema de referência geográfico espacial.

    SELECT ST_Distance(
    ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
    ST_GeometryFromText('POINT(2.5559 49.0083)', 4326)     -- Paris (CDG)
    );
***
    121.898285970107
***

O que significa esse resultado? As unidades para referência espacial 4326 são graus. Portanto, nossa resposta é de 121 graus. Mas, o que isso significa?

Em uma esfera, o tamanho de um “quadrado de grau” é bastante variável, diminuindo à medida que você se afasta do equador. Pense nos meridianos (linhas verticais) do globo se aproximando um do outro à medida que você vai em direção aos pólos.

Então, uma distância de 121 graus não significa nada. É um número sem sentido.

Para calcular uma distância significativa, devemos tratar coordenadas geográficas não como coordenadas cartesianas aproximadas, mas sim como coordenadas esféricas verdadeiras. Devemos medir as distâncias entre os pontos como caminhos verdadeiros sobre uma esfera - uma parte de um grande círculo.

A partir da versão 1.5, o PostGIS fornece essa funcionalidade através do tipo de dado geográfico.

***
**Nota**

Diferentes bancos de dados espaciais têm diferentes abordagens para lidar com dados geográficos:

* A **Oracle** tenta revisar as diferenças fazendo cálculos geográficos de forma transparente quando o SRID é geográfico.
* O **SQL Server** usa dois tipos espaciais, “STGeometry” para dados cartesianos e “STGeography” para geográficos.
* O **Informix Spatial** é uma extensão cartesiana pura para o Informix, enquanto o Informix Geodetic é uma extensão geográfica pura.
* Semelhante ao **SQL Server**, o **PostGIS** usa dois tipos: dados geométricos e dados geográficos. Usando dados geográficos em vez do tipo de dado geométrico, vamos tentar medir novamente a distância entre Los Angeles e Paris. Em vez de ST_GeometryFromText(text), usaremos ST_GeographyFromText(text).
***

    SELECT ST_Distance(
    ST_GeographyFromText('POINT(-118.4079 33.9434)'), -- Los Angeles (LAX)
    ST_GeographyFromText('POINT(2.5559 49.0083)')     -- Paris (CDG)
    );
***
    9124665.27317673
***
Todos os valores de retorno dos cálculos de geografia estão em metros, então nossa resposta é 9124km.

Versões mais antigas do **PostGIS** suportavam cálculos muito básicos sobre a esfera usando a função **ST_Distance_Spheroid(point, point, measure)**. No entanto, **ST_Distance_Spheroid** é substancialmente limitado. A função só funciona em pontos e não oferece suporte para indexação entre os polos ou linha de dados internacional.
A necessidade de dar suporte a geometrias não pontuais torna-se muito clara quando se coloca uma pergunta como "O quão próximo um voo de Los Angeles para Paris chegará à Islândia?"

![https://github.com/deamorim2/sbde/blob/master/wiki/18/lax_cdg.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/18/lax_cdg.jpg)

Trabalhar com coordenadas geográficas em um plano cartesiano (a linha roxa) produz uma resposta muito errada! Usando grandes rotas circulares (as linhas vermelhas) dá a resposta correta. Se convertermos nosso voo LAX-CDG em uma linha de linha e calcularmos a distância até um ponto na Islândia usando a geografia, obteremos a resposta correta (rechamada) em metros.

    SELECT ST_Distance(
    ST_GeographyFromText('LINESTRING(-118.4079 33.9434, 2.5559 49.0083)'), -- LAX-CDG
    ST_GeographyFromText('POINT(-22.6056 63.9850)')                        -- Iceland (KEF)
    );

***
502454.90667289
***

Portanto, a aproximação mais próxima da Islândia (medida a partir de seu aeroporto internacional) na rota LAX-CDG é um número relativamente pequeno de 502 km.

A abordagem cartesiana para lidar com coordenadas geográficas divide-se inteiramente por recursos que cruzam a linha de dados internacional. A rota mais curta do circuito de Los Angeles para Tóquio cruza o Oceano Pacífico. A rota cartesiana mais curta atravessa os oceanos Atlântico e Índico.

![https://github.com/deamorim2/sbde/blob/master/wiki/18/lax_nrt.png
](https://github.com/deamorim2/sbde/blob/master/wiki/18/lax_nrt.png)

    SELECT ST_Distance(
    ST_GeometryFromText('Point(-118.4079 33.9434)'),  -- LAX
    ST_GeometryFromText('Point(139.733 35.567)'))     -- NRT (Tokyo/Narita)

    AS geometry_distance,

    ST_Distance(
    ST_GeographyFromText('Point(-118.4079 33.9434)'), -- LAX
    ST_GeographyFromText('Point(139.733 35.567)'))    -- NRT (Tokyo/Narita)
    AS geography_distance;

![https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_01.png
](https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_01.png)

# 18.1 Usando Dados Geográficos

Para carregar dados de geometria em uma tabela de dados geográficos, a geometria primeiro precisa ser projetada em EPSG:4326 (longitude/latitude), então ela precisa ser alterada para dados geográficos.

A função Geography (geometria) “coverte” os dados geométricos para dados geográficos.

    CREATE TABLE lim_municipio_a_geog AS
    SELECT nome, geocodigo, Geography(geom) AS geog
    FROM lim_municipio_a;

A construção de um índice espacial em uma tabela de dados geográficos é exatamente igual a de dados geométricos:

    CREATE INDEX lim_municipio_a_geog_gix
    ON lim_municipio_a_geog USING GIST (geog);

***
**ATENÇÃO!**

A diferença está nos bastidores: o índice geográfico lidará corretamente com as consultas que cobrem os pólos ou a linha de data internacional, enquanto a de geometria não.
***
Há apenas um pequeno número de funções nativas para o tipo de dado geográfico:

* ST_AsText(geography) - retorna o texto
* ST_GeographyFromText(text) - retorna dado geográfico
* ST_AsBinary(geography) - retorna bytea
* ST_GeogFromWKB(bytea) - retorna dado geográfico
* ST_AsSVG(geography) - retorna o texto
* ST_AsGML(geography) - retorna o texto
* ST_AsKML(geography) - retorna o texto
* ST_AsGeoJson(geography) - retorna o texto
* ST_Distance (geography, geography) - retorna double
* ST_DWithin(geography, geography, float8) - retorna booleano
* ST_Area(geography) - retorna double
* ST_Length(geography) - retorna double
* ST_Covers(geography, geography) - retorna booleano
* ST_CoveredBy(geography, geography) - retorna booleano
* ST_Intersects(geography, geography) - retorna booleano
* ST_Buffer(geography, float8) - retorna dado geográfico
* ST_Intersection(geography, geography) - retorna dado geográfico

# 18.2 Criando uma tabela de dados geográficos

O SQL para criar uma nova tabela com uma coluna de dados geográficos é muito parecido com a criação de uma tabela de dados de geometria. No entanto, o dado geográfico inclui a capacidade de especificar o tipo de objeto diretamente no momento da criação da tabela.

**Por exemplo:**

    CREATE TABLE airports (code VARCHAR(3), geog GEOGRAPHY(Point));

    INSERT INTO airports VALUES ('LAX', 'POINT(-118.4079 33.9434)');
    INSERT INTO airports VALUES ('CDG', 'POINT(2.5559 49.0083)');
    INSERT INTO airports VALUES ('KEF', 'POINT(-22.6056 63.9850)');

Na definição da tabela, o GEOGRAPHY(Point) especifica nosso tipo de dados do aeroporto como pontos. Os novos campos de geografia não são registrados na visualização `geometry_columns`. Em vez disso, eles são registrados em uma exibição chamada `geography_columns`.

    SELECT * FROM geography_columns;

![https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_02.png)
***
**Observação**

Quando da criação de uma tabela geográfica, está implícito o SRID 4326 (WGS84).
***

# 18.3 Convertendo para Geometria

Embora as funções básicas para tipos de dados geográficos possam lidar com muitos casos de uso, há momentos em que você pode precisar acessar outras funções suportadas apenas pelo tipo geométrico.

Felizmente, você pode converter objetos de um lado para o outro de dado geográfico para dado geométrico.

A convenção de sintaxe do **PostgreSQL** para conversão é colocar `::typename` ao final do valor que você deseja converter. Assim, `2::texto` converte o número dois para uma dado do tipo '2'. `'POINT(0 0)'::geometry` converterá a representação de texto do ponto(WKT) em um ponto de geometria.

A função ST_X(point) suporta apenas o tipo de dado geométrico.

Sendo assim, como podemos ler a coordenada X de nossos dados geográficos?

    SELECT code, ST_X(geog::geometry) AS longitude FROM airports;

![https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_03.png)

Convertendo o dado geográfico para geométrico(geog::geometry), convertemos o objeto em uma geometria com um SRID de 4326. A partir daí, podemos usar tantas funções de geometria quanto quisermos. Mas lembre-se: agora que o nosso objeto é uma geometria, as coordenadas serão interpretadas como coordenadas cartesianas, e não esféricas.

# 18.4 Quando Não Usar Dado Geográfico

Dados Geográficos são coordenadas universalmente aceitas - todos entendem o que latitude/longitude significam, mas muito poucas pessoas entendem o que as coordenadas UTM significam. Por que não usar dado geográfico o tempo todo?

Primeiro, como observado anteriormente, há muito menos funções disponíveis que suportam diretamente o tipo de ado geográfico. Você pode gastar muito tempo trabalhando com limitações do tipo de dado geográfico.

Em segundo lugar, os cálculos em uma esfera são computacionalmente muito mais caros do que os cálculos cartesianos. Por exemplo, a fórmula cartesiana para distância (Pitágoras) envolve uma chamada para sqrt(). A fórmula esférica para distância (Haversine) envolve duas chamadas sqrt(), uma chamada arctan(), quatro chamadas sin() e duas chamadas cos(). As funções trigonométricas são muito caras e os cálculos esféricos envolvem muitos deles.
***
**Conclusão:**

Se seus dados forem geograficamente compactos (contidos em um estado, município ou cidade), use o tipo de geometria com uma projeção cartesiana que faça sentido com seus dados.

Veja o site [http://spatialreference.org](http://spatialreference.org) e digite o nome da sua região para uma seleção de possíveis sistemas de referência.

Se você precisar medir a distância com um conjunto de dados geograficamente disperso (cobrindo grande parte do mundo), use o tipo de dado geográfico. A complexidade do aplicativo que você salva trabalhando com dado geográfico compensará qualquer problema de desempenho e a conversão para dado geométrico pode compensar a maioria das limitações de funcionalidade.
***

# 18.5 Visualizando Dados Geográficos no QGIS

>Baixe os [DADOS MUNDIAIS](https://drive.google.com/drive/folders/1WYUSG29IXcG8CcYoYGCEkfgQGLZ63Ekb?usp=sharing) que estão no formato shapefile. Fonte: [http://www.naturalearthdata.com/downloads/](http://www.naturalearthdata.com/downloads/)

>Importe esses dados para o banco de dados sbde (SRID: 4326; Codificação da fonte de dados tabulares:UTF-8) utilizando o Gerenciador BD do QGIS. Mantenha o nome das tabelas como sendo o nome dos shapefiles: `ne_50m_admin_0_countries` e `ne_50m_populated_places_simple`.

![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_01.png)

![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_02.png)

>Abra essas tabelas no QGIS e visualize os dados espaciais. Insira também a tabela `airports`.

![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_03.png)

>Vá no Gerenciador BD faça a consulta abaixo e visualize-a no mapa como `LAX-CDG`:

    SELECT 1 as id, ST_Makeline(
    (SELECT geog
    FROM airports
    WHERE code = 'LAX')::geometry
    ,
    (SELECT geog
    FROM airports
    WHERE code = 'CDG')::geometry
    )::geography as geog

![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_04.png)

![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_05.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_05.png)

A construção da feição espacial que apresenta a rota aérea LAX-CDG foi realizada em algumas etapas:

1. Duas consultas para adquirir as feições geográficas dos aeroportos de LAX e CDG;
2. Conversão (casting) das feições geográficas dos aeroportos para o tipo geométricos, já que a função espacial ST_Makeline(geometry, geometry) somente funciona com dados geométricos e não tem suporte para dados geográficos.
3. Geração da feição geométrica linear a partir dos dois pontos geométricos que representam os aeroportos de LAX e CDG
4. Conversão (casting) da feição linear geométrica para feição linear geográfica.

Repare que a rota LAX-CDG apesar de estar no formato de dados geográficos, apresenta uma linha reta como se o dado fosse geométrico.

Isso ocorre porque o QGIS cria essa "linha" a partir do traçado computacional entre os dois aeroportos.

Para solucionar esse "problema" de visualização, é necessário "segmentar" o dado geográfico do tipo linha por meio de vértices que representem o caminho traçado entre os aeroportos.

Nesse caso, utilizaremos a função espacial ST_Segmentize(geography geog, float max_segment_length), que possui suporte para dados geográficos, com segmentação da linha em vértices com espaçamento entre eles de 10m.

    SELECT 1 as id,
    ST_Segmentize(

    ST_Makeline(

    (SELECT geog
    FROM airports
    WHERE code = 'LAX')::geometry

    ,

    (SELECT geog
    FROM airports
    WHERE code = 'CDG')::geometry

    )::geography

    ,10) as geog

![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_06.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_06.png)

Veja que agora a rota LAX-CDG apresenta um caminho curvo.

>Repita a consulta, mas coloque a distância de 3.000km(ou 3 milhões de metros) entre os vértices:

    SELECT 1 as id,
    ST_Segmentize(

    ST_Makeline(

    (SELECT geog
    FROM airports
    WHERE code = 'LAX')::geometry

    ,

    (SELECT geog
    FROM airports
    WHERE code = 'CDG')::geometry

    )::geography

    ,3000000) as geog


![https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_07.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/qgis_07.png)
 
>Execute o cálculo de comprimento da rota nas três situações apresentadas acima:

    SELECT 'Distância geográfica' as cálculo,
    ST_Length(
    ST_Makeline(
    (SELECT geog
    FROM airports
    WHERE code = 'LAX')::geometry
    ,
    (SELECT geog
    FROM airports
    WHERE code = 'CDG')::geometry
    )::geography
    ) as distância

    UNION
    
    SELECT 'Distância geográfica com vértice de 10m' as cálculo,
    ST_Length(
    ST_Segmentize(
    ST_Makeline(
    (SELECT geog
    FROM airports
    WHERE code = 'LAX')::geometry
    ,
    (SELECT geog
    FROM airports
    WHERE code = 'CDG')::geometry
    )::geography
    ,10)
    ) as distância

    UNION

    SELECT 'Distância geográfica com vértice de 3.000km' as cálculo,
    ST_Length(
    ST_Segmentize(
    ST_Makeline(
    (SELECT geog
    FROM airports
    WHERE code = 'LAX')::geometry
    ,
    (SELECT geog
    FROM airports
    WHERE code = 'CDG')::geometry
    )::geography
    ,3000000)
    ) as distância;
 
![https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/18/pgadmin_04.png)

Verifique que as rotas segmentadas possuem comprimento muito próximo da linha sem segmentação, mas, mesmo assim, não apresentam valores precisos. Use essa função de segmentação somente para propósitos de visualização dos dados.

# 18.6 Lista de Funções

* ST_Distance (geometry, geometry): Para o tipo de geometria Retorna a distância mínima cartesiana bidimensional (com base na referência espacial) entre duas geometrias em unidades projetadas. Para dados geográficos digite default para retornar a distância mínima esferoidal entre duas geografias em metros.
* ST_GeographyFromText (text): Retorna um valor de geografia especificado da representação de texto conhecido ou estendido (WKT).
* ST_Transform (geometry, srid): retorna uma nova geometria com suas coordenadas transformadas no SRID referenciado pelo parâmetro inteiro.
* ST_X (ponto): Retorna a coordenada X do ponto ou NULL se não estiver disponível. Entrada deve ser um ponto.
* ST_Segmentize(geography geog, float max_segment_length): Retorna uma geometria/geografia modificada sem nenhum segmento maior que a distância especificada.
