# 19. Funções de Construção de Geometria

Todas as funções que temos visto até agora apresentam informações de como geometrias são e normalmente retornam:

* análises dos objetos (ST_Length (geometry), ST_Area (geometry));
* serializações dos objetos (ST_AsText (geometry), ST_AsGML (geometry));
* partes do objeto (ST_RingN (geometry, n));
* testes verdadeiro/falso (ST_Contains (geometry, geometry), ST_Intersects (geometry, geometry)).

**Funções de construção de geometria** utilizam dados geométricos como dados de entradas e, após processamento, geram novos dados geométricos em outros formatos.

# 19.1. ST_Centroid/ST_PointOnSurface

Uma necessidade comum ao compor uma consulta espacial é substituir um recurso de polígono por uma representação de ponto do recurso. Isso é útil para junções espaciais (como discutido em Junções Poligonais / Poligonais) porque o uso de ST_Intersects (geometria, geometria) em duas camadas poligonais geralmente resulta em contagem dupla: um polígono em um limite interceptará um objeto em ambos os lados; substituí-lo por um ponto força-o a estar de um lado ou de outro, não de ambos.

* ST_Centroid (geometry) retorna um ponto que está aproximadamente no centro de massa do argumento de entrada. Esse cálculo simples é muito rápido, mas às vezes não é desejável, porque o ponto retornado não está necessariamente no próprio recurso. Se o recurso de entrada tiver convexidade (imagine a letra "C"), o centróide retornado pode não estar no interior do recurso.

* ST_PointOnSurface (geometry) retorna um ponto garantido dentro do argumento de entrada. É substancialmente mais computacionalmente caro que a operação do centróide.

![https://github.com/deamorim2/sbde/blob/master/wiki/19/centroid.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/19/centroid.jpg)

# 19.2 ST_Buffer

A operação de armazenamento em buffer([ST_Buffer](http://www.postgis.net/docs/ST_Buffer.html)) é comum em fluxos de trabalho GIS e também está disponível no PostGIS. ST_Buffer (geometry, distance)/ST_Buffer(geography, distance) recebe uma distância de buffer e um tipo de geometria e gera um polígono com um limite na distância do buffer da geometria de entrada.

![https://github.com/deamorim2/sbde/blob/master/wiki/19/st_buffer.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/st_buffer.png)

>Faça uma área de segurança de 1.000m do Arquipélago de 'Fernando de Noronha' a partir da tabela geográfica `lim_municipio_a_geog` e utilizando a função espacial **ST_Buffer**.

    SELECT geocodigo, nome, ST_Buffer(geog,1000) as geog
    FROM lim_municipio_a_geog
    WHERE nome = 'Fernando de Noronha'

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_01.png)

>Agora faça a mesma consulta acima, mas a partir da tabela geométrica `lim_municipio_a`.

    SELECT geocodigo, nome, ST_Buffer((geom)::Geography,1000) as geog
    FROM lim_municipio_a
    WHERE nome = 'Fernando de Noronha'

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_02.png)

Também é possível fazer essa mesma consulta por meio da transformação da geometria do SRID 4674 para o SRID 5880 para entrarmos com o dado de entrada de distância do buffer de 1.000 metros.

Porém, lembre-se que para converter o dado de geométrico para geográfico, tem-se que retransformar o dado para um SRID do tipo Lat/Long, como o SRID original 4674.

    SELECT geocodigo, nome, ST_Transform((ST_Buffer(ST_Transform(geom,5880), 1000)),4674)::geography as geom
    FROM lim_municipio_a
    WHERE nome = 'Fernando de Noronha'

A função ST_Buffer também aceita distâncias negativas e constrói polígonos inscritos em entradas poligonais. Para linhas e pontos, você receberá apenas um retorno vazio.

    SELECT geocodigo, nome, ST_Buffer(geog,-1000) as geog
    FROM lim_municipio_a_geog
    WHERE nome = 'Fernando de Noronha'

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_03.png)

# 19.3 ST_Intersection

Outra operação GIS clássica - a "sobreposição" - cria uma nova camada calculada a partir da interseção de duas geometrias sobrepostas. A geometria resultante possui a propriedade de que qualquer polígono em qualquer um dos pais pode ser construído mesclando polígonos no resultante.

A função ST_Intersection (geometria A, geometria B) retorna a área espacial (ou linha ou ponto) que ambos os argumentos têm em comum. Se os argumentos forem disjuntos, a função retornará uma geometria vazia.

>"Qual a geometria resultante da sobreposição de dois buffers com distância de 2 unidades a partir dos pontos: 'POINT(0 0)' e 'POINT(3 0)'"? 

**Pontos:**

    SELECT 1 as id, 'POINT(0 0)' as geom
    UNION
    SELECT 2 as id, 'POINT(3 0)' as geom

**Buffers:**

    SELECT 1 as id, ST_Buffer('POINT(0 0)', 2) as geom
    UNION
    SELECT 2 as id, ST_Buffer('POINT(3 0)', 2) as geom

**Intersection:**

    SELECT 1 as id, ST_Intersection(ST_Buffer('POINT(0 0)', 2), ST_Buffer('POINT(3 0)', 2))

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_04.png)

# 19.4. ST_Union

No exemplo anterior, cruzamos geometrias, criando uma nova geometria que possui geometria em ambas as entradas.

A ST_Union faz o inverso, ele recebe entradas e remove as áreas em comum.

Existem duas formas da função ST_Union:

* ST_Union(geometry, geometry): Uma versão de dois argumentos que recebe duas geometrias e retorna a união mesclada. Por exemplo, nosso exemplo de dois círculos da seção anterior é assim quando você substitui a interseção por uma união.

**União**

    SELECT 1 as id, ST_Union(ST_Buffer('POINT(0 0)', 2), ST_Buffer('POINT(3 0)', 2)) as geom

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_05.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_05.png)

* ST_Union ([geometry]): uma versão agregada que aceita um conjunto de geometrias e retorna a geometria mesclada para todo o grupo. O ST_Union agregado pode ser usado com a instrução SQL GROUP BY para criar subconjuntos cuidadosamente mesclados de geometrias básicas.

Como um exemplo de agregação ST_Union, considere nossa tabela `lim_unidade_federacao`.

>Crie as macrorregiões brasileira a partir da tabela `lim_unidade_federacao`.

**Crie uma nova coluna na tabela `lim_unidade_federacao` chamada `macrorregiao` do tipo char(12):**

    ALTER TABLE lim_unidade_federacao_a
    ADD COLUMN macrorregiao char(12);

**Atualize o campo `macrorregiao` com o nome de cada uma das regiões:**

    UPDATE lim_unidade_federacao_a
    SET macrorregiao = 'Norte'
    WHERE geocodigo LIKE '1%';

    UPDATE lim_unidade_federacao_a
    SET macrorregiao = 'Nordeste'
    WHERE geocodigo LIKE '2%';


    UPDATE lim_unidade_federacao_a
    SET macrorregiao = 'Sudeste'
    WHERE geocodigo LIKE '3%';


    UPDATE lim_unidade_federacao_a
    SET macrorregiao = 'Sul'
    WHERE geocodigo LIKE '4%';

    UPDATE lim_unidade_federacao_a
    SET macrorregiao = 'Centro-Oeste'
    WHERE geocodigo LIKE '5%';

**Faça a união espacial (agregação) a partir do nome da macrorregião:**

    DROP TABLE IF EXISTS lim_macrorregiao_a;

    CREATE TABLE lim_macrorregiao_a AS
    SELECT substring(geocodigo from 1 for 1) as geocodigo, macrorregiao as nome, ST_MemUnion(geom)::Geometry(MultiPolygon,4674) AS geom
    FROM lim_unidade_federacao_a
    GROUP BY macrorregiao, substring(geocodigo from 1 for 1);
***
**Nota:**
 a função **ST_MemUnion** é a mesma que **ST_Union**, apenas compatível com memória (usa menos memória e mais tempo de processador). Essa função de agregação funciona unindo as geometrias, uma de cada vez, ao resultado anterior, em oposição ao agregado ST_Union, que cria primeiro uma matriz e, em seguida, faz as uniões.
***

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_06.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_06.png)

Repare que dentro de alguns polígonos existem anéis internos (inner rings). Isso ocorre porque existe uma falha na consistência topológica entre algumas unidades da federação.

Para resolver isso, iremos atualizar a geometria da tabela `lim_macrorregiao_a` excluindo esses anéis internos por meio das instruções abaixo:

    DROP TABLE IF EXISTS lim_macrorregiao_a_temp;

    CREATE TABLE lim_macrorregiao_a_temp AS
    SELECT geocodigo, nome, (ST_Dump(geom)).geom as geom
    FROM lim_macrorregiao_a;

    UPDATE lim_macrorregiao_a_temp
    SET geom = ST_Multi(ST_MakePolygon(ST_ExteriorRing(geom)));

    UPDATE lim_macrorregiao_a
    SET geom = a.geom
    FROM
    ( 
    SELECT geocodigo, (ST_UNION(geom)) as geom
    FROM lim_macrorregiao_a_temp
    GROUP BY geocodigo
    ) as a
    WHERE a.geocodigo = lim_macrorregiao_a.geocodigo;

    DROP TABLE IF EXISTS lim_macrorregiao_a_temp;

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_07.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_07.png)

# 19.5 Buffer Geográfico

O míssel balístico intercontinental [Hwasong-14](https://en.wikipedia.org/wiki/Hwasong-14) da Coréia do Norte possui um alcance máximo estimado de 10 mil km.

Vamos criar uma tabela espacial com o buffer de 10 mil km a partir dos limites políticos da Coréia do Norte utilizando a tabela espacial `ne_50m_admin_0_countries`. Para isso, criaremos duas colunas espaciais: uma com dados geométricos e outra com dados geográficos:

    DROP TABLE IF EXISTS north_korea_hwasong_14;

    CREATE TABLE north_korea_hwasong_14 AS
    SELECT 1 as id, name, ST_Buffer((geom)::geography,10000000) as geog, ST_Buffer(geom,107) as geom
    FROM ne_50m_admin_0_countries
    WHERE name = 'North Korea';

    ALTER TABLE north_korea_hwasong_14 ADD PRIMARY KEY (id);

Visualize no **QGIS** os dados geométricos e os dados geográficos.

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_08.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_08.png)

Repare que o buffer construído pelos dados geográficos é muito mais abrangente que os dados geométricos e leva em consideração os limites internacionais.

As informações espaciais estão corretas, mas o sistema de coordenadas SRID 4326 não é o mais adequeado para a visualização desse buffer obtido a partir dos dados geográficos.

Escolha alguma projeção no **QGIS** _on the fly_ que seja do tipo _Lambert Azimutal Area_ filtrado para `WGS 84`.

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_09.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_09.png)

No nosso caso, escolhemos o SRID 3571

![https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_10.png](https://github.com/deamorim2/sbde/blob/master/wiki/19/qgis_10.png)

Faça uma consulta com as cidades com população maior que 100 mil habitantes que estão ao alcance do [Hwasong-12](https://en.wikipedia.org/wiki/Hwasong-12) da Coréia do Norte, que possui um alcance máximo estimado de 6 mil km. Apresente esses resultados agrupados por país (`ne_50m_populated_places_simple.adm0name`) e ordenados por número de cidades em cada país ao alcance do foguete.

    SELECT pais, count(cidade) as num_cidade, sum(b.populacao) as populacao  
    FROM
    (
        SELECT pop.name as cidade, pop.adm0name as pais, pop.pop_max as populacao, pop.geom as geom
        FROM ne_50m_populated_places_simple pop,
        (
            SELECT name, ST_Buffer((geom)::geography,6000000) as geom  
            FROM ne_50m_admin_0_countries
            WHERE name = 'North Korea'
        ) as a
        WHERE ST_Intersects((pop.geom)::geography, a.geom)
        AND pop.pop_max >= 100000
    ) as b
    GROUP BY pais
    ORDER BY num_cidade DESC;

Agora faça essa mesma consulta com o [Hwasong-14](https://en.wikipedia.org/wiki/Hwasong-14) da Coréia do Norte, que possui um alcance máximo estimado de 10 mil km.

    SELECT pais, count(cidade) as num_cidade, sum(b.populacao) as populacao  
    FROM
    (
        SELECT pop.name as cidade, pop.adm0name as pais, pop.pop_max as populacao, pop.geom as geom
        FROM ne_50m_populated_places_simple pop,
        (
            SELECT name, ST_Buffer((geom)::geography,10000000) as geom  
            FROM ne_50m_admin_0_countries
            WHERE name = 'North Korea'
        ) as a
        WHERE ST_Intersects((pop.geom)::geography, a.geom)
        AND pop.pop_max >= 100000
    ) as b
    GROUP BY pais
    ORDER BY num_cidade DESC;

# 19.6 Lista de funções

* ST_AsText(text): Retorna a representação WKT da geometria/geografia sem metadados SRID.
* ST_Buffer(geometry , distance): Para geometria: Retorna uma geometria que representa todos os pontos cuja distância desta geometria é menor ou igual à distância. Os cálculos estão no Sistema de Referência Espacial desta Geometria. Para geografia: usa um wrapper de transformação planar.
* ST_Intersection (geometry A, geometry B): Retorna uma geometria que representa a parte compartilhada de geomA e geomB. A implementação geográfica faz uma transformação para geometria para fazer a interseção e depois transformar de volta para WGS84.
* ST_Union(): retorna uma geometria que representa a união do conjunto de pontos das geometrias.
* substring(string [from int] [para int]): Função de string do PostgreSQL para extrair a expressão regular SQL de correspondência de substring.
* sum(expression): Função de agregação do PostgreSQL que retorna a soma dos registros em um conjunto de registros.
* ST_ExteriorRing - Retorna uma linha representando o anel externo da geometria POLYGON. Retorna NULL se a geometria não é um polígono. Não funcionará com MULTIPOLÍGONO

