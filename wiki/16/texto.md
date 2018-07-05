# 16. Projeção de Dados Espaciais

A Terra não é plana, e não há uma maneira simples de colocá-la em um mapa plano (ou tela de computador), então as pessoas criaram todos os tipos de soluções engenhosas, cada uma com vantagens e desvantagens.

Algumas projeções preservam a área, portanto todos os objetos têm um tamanho relativo um ao outro. Outras projeções preservam ângulos (conformes) como a projeção de Mercator. Algumas projeções tentam encontrar uma boa mistura intermediária com pouca distorção em vários parâmetros.

Comum a todas as projeções é que elas transformam o mundo (esférico) em um sistema cartesiano de coordenadas planas, e qual projeção escolher depende de como você estará usando os dados.

Já encontramos projeções quando carregamos nossos dados no banco `sbde` (Lembre-se do [SRID 4674 - SIRGAS 2000](https://epsg.io/4674)). Às vezes, no entanto, você precisa transformar e reprojetar entre sistemas de referência espacial.

O PostGIS inclui suporte interno para alterar a projeção de dados, usando a função ST_Transform (geometry, srid).

Para gerenciar os identificadores de referência espacial em geometrias, o PostGIS fornece as funções ST_SRID(geometry) e ST_SetSRID(geometry, srid).

Podemos consultar o SRID dos nossos dados espaciais com a função ST_SRID.

>Execute a consulta abaixo:

    SELECT ST_SRID(geom) FROM lim_municipio_a LIMIT 1;

***
    4674
***

E o que é esse número “4674”? Como vimos na sessão “[Carregando Dados Espaciais e Não Espaciais](https://github.com/deamorim2/sbde/wiki/05.-Carregando-Dados-Espaciais-e-Não-Espaciais)”, as definições do SRID estão contidas na tabela `spatial_ref_sys`. De fato, duas definições estão lá. A definição em OGC WKT está na coluna `srtext`, e há uma segunda definição no formato [proj.4](proj4.org/) na coluna `proj4text`.

>Visualize essas definições executando a consulta abaixo:

    SELECT * FROM spatial_ref_sys WHERE srid = 4674;

De fato, para os cálculos internos de re-projeção **PostGIS**, é o conteúdo da coluna `proj4text` que é usada. Para nossa projeção 4674, o resultado da consulta abaixo apresenta os parâmetros do proj4:

    SELECT proj4text FROM spatial_ref_sys WHERE srid = 4674;
***
    +proj=longlat +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +no_defs
***

Na prática, as colunas `srtext` e `proj4text` são importantes: a coluna `srtext` (OGC WKT) é usada por programas externos como `GeoServer`, `uDig`, `FME`, entre outros. Os dados da coluna `proj4text` são usados internamente.

# 16.1 Comparando Dados

Juntos, uma coordenada e um SRID definem um local no globo. Sem um SRID, uma coordenada é apenas uma noção abstrata.

Um plano de coordenadas “Cartesiano” é definido como um sistema de coordenadas “plano” colocado na superfície da Terra.

Como as funções do PostGIS funcionam nesse plano, as operações de comparação exigem que as duas geometrias sejam representadas no mesmo SRID.

Se você alimentar geometrias com diferentes SRIDs, você receberá um erro:

    SELECT ST_Equals(ST_GeomFromText('POINT(0 0)', 4326), ST_GeomFromText('POINT(0 0)', 4674));

***
    ERROR:  Operation on mixed SRID geometries
    CONTEXT:  SQL function "st_equals" statement 1

    ********** Error **********

    ERROR: Operation on mixed SRID geometries
    SQL state: XX000
    Context: SQL function "st_equals" statement 1
***
**Nota:**
 cuidado com o uso do ST_Transform para conversões on-the-fly. Os índices espaciais são construídos usando o SRID das geometrias armazenadas. Se a comparação for feita em um SRID diferente, os índices espaciais(geralmente) não são utilizados. É uma boa prática escolher um SRID para todas as tabelas em seu banco de dados. Use somente a função de transformação quando estiver lendo ou gravando dados em aplicativos externos.
***

# 16.2 Transformando Dados

Se retornarmos à nossa definição do `proj4` para o SRID 4674, podemos ver que nosso sistema de coordenadas é do tipo geodésica (longlat).

    +proj=longlat +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +no_defs

Já o OGC WKT para o SRID 4674 possui os seguintes parâmetros:

    GEOGCS["SIRGAS 2000",
        DATUM["Sistema_de_Referencia_Geocentrico_para_las_AmericaS_2000",
            SPHEROID["GRS 1980",6378137,298.257222101,
                AUTHORITY["EPSG","7019"]],
            TOWGS84[0,0,0,0,0,0,0],
            AUTHORITY["EPSG","6674"]],
        PRIMEM["Greenwich",0,
            AUTHORITY["EPSG","8901"]],
        UNIT["degree",0.0174532925199433,
            AUTHORITY["EPSG","9122"]],
        AUTHORITY["EPSG","4674"]]

## 16.2.1 Parâmetros Determinados pelo IBGE

### 16.2.1.1 Cálculo de Área

O IBGE sugere parâmetros específicos de projeção para cálculo de área dos produtos da bc250.

A projeção especificada é a Equivalente de Albers:

* Longitude origem -54°
* Latitude origem: -12°
* Paralelo padrão 1: -2°
* Paralelo padrão 2: -22°

Esses parâmetros convertidos no formato **proj4** ficam da seguinte forma:

    +proj=aea +lat_1=-2 +lat_2=-22 +lat_0=-12 +lon_0=-54 +x_0=0 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs

Já no formato **OGC WKT** fica:

    PROJCS["Brazil_Albers_Equal_Area_Conic",
        GEOGCS["SIRGAS 2000",
            DATUM["Sistema_de_Referencia_Geocentrico_para_las_AmericaS_2000",
                SPHEROID["GRS 1980",6378137,298.257222101,
                    AUTHORITY["EPSG","7019"]],
                TOWGS84[0,0,0,0,0,0,0],
                AUTHORITY["EPSG","6674"]],
            PRIMEM["Greenwich",0,
                AUTHORITY["EPSG","8901"]],
            UNIT["degree",0.0174532925199433,
                AUTHORITY["EPSG","9122"]],
            AUTHORITY["EPSG","4674"]],
        PROJECTION["Albers_Conic_Equal_Area"],
        PARAMETER["False_Easting",0],
        PARAMETER["False_Northing",0],
        PARAMETER["longitude_of_center",-54],
        PARAMETER["Standard_Parallel_1",-2],
        PARAMETER["Standard_Parallel_2",-22],
        PARAMETER["latitude_of_center",-12],
        UNIT["Meter",1],
        AUTHORITY["IBGE","55555"]]

Para inserir esse novo SRID na tabela `spatial_ref_sys`, execute a seguinte instrução SQL:

    INSERT into spatial_ref_sys (srid, auth_name, auth_srid, proj4text, srtext)
    values
    (
    55555,
    'IBGE',
    55555,
    '+proj=aea +lat_1=-2 +lat_2=-22 +lat_0=-12 +lon_0=-54 +x_0=0 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs ',
    'PROJCS["Brazil_Albers_Equal_Area_Conic",GEOGCS["SIRGAS 2000",DATUM["Sistema_de_Referencia_Geocentrico_para_las_AmericaS_2000",SPHEROID["GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]],TOWGS84[0,0,0,0,0,0,0],AUTHORITY["EPSG","6674"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4674"]],PROJECTION["Albers_Conic_Equal_Area"],PARAMETER["False_Easting",0],PARAMETER["False_Northing",0],PARAMETER["longitude_of_center",-54],PARAMETER["Standard_Parallel_1",-2],PARAMETER["Standard_Parallel_2",-22],PARAMETER["latitude_of_center",-12],UNIT["Meter",1],AUTHORITY["IBGE","55555"]]'
    );

Lembrando que não existe um SRID 55555 no proj4 com esses parâmetros acima.

### 16.2.1.2 Cálculo de Extensão

Vamos converter a projeção de alguns dados para a Projeção Policônica para o Brasil/SIRGAS 2000(SRID 5880), que é a projeção indicada pelo IBGE para cálculo de extensão de feições lineares para os produtos da bc250.

Você pode ver a definição no site epsg.io: [https://epsg.io/5880](https://epsg.io/5880).

Você também pode extrair as definições textuais do OGC WKT para o SRID 5880 da tabela spatial_ref_sys:

    SELECT srtext FROM spatial_ref_sys WHERE srid = 5880;

***

    PROJCS["SIRGAS 2000 / Brazil Polyconic",
        GEOGCS["SIRGAS 2000",
            DATUM["Sistema_de_Referencia_Geocentrico_para_las_AmericaS_2000",
                SPHEROID["GRS 1980",6378137,298.257222101,
                    AUTHORITY["EPSG","7019"]],
                TOWGS84[0,0,0,0,0,0,0],
                AUTHORITY["EPSG","6674"]],
            PRIMEM["Greenwich",0,
                AUTHORITY["EPSG","8901"]],
            UNIT["degree",0.0174532925199433,
                AUTHORITY["EPSG","9122"]],
            AUTHORITY["EPSG","4674"]],
        PROJECTION["Polyconic"],
        PARAMETER["latitude_of_origin",0],
        PARAMETER["central_meridian",-54],
        PARAMETER["false_easting",5000000],
        PARAMETER["false_northing",10000000],
        UNIT["metre",1,
        AUTHORITY["EPSG","9001"]],
        AXIS["X",EAST],
        AXIS["Y",NORTH],
        AUTHORITY["EPSG","5880"]]

Já os dados do proj4 para o SRID 5880 da tabela spatial_ref_sys:

    SELECT proj4text FROM spatial_ref_sys WHERE srid = 5880;

***
    +proj=poly +lat_0=0 +lon_0=-54 +x_0=5000000 +y_0=10000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs
***
Agora, vamos converter as coordenadas.

A consulta abaixo apresenta as coordenadas do ponto da tabela `loc_cidade_p` em SRID 4674 (SIRGAS 2000, unidade em Longitude/Latitude) na coluna `srid_4674` e em SRID 5880 (Projeção Policônica, unidade em metros) na coluna `srid_5880`.

    SELECT nome, ST_AsText(ST_Transform(geom,4674)) as srid_4674, ST_AsText(ST_Transform(geom,5880)) as srid_5880
    FROM loc_cidade_p
    LIMIT 1;

![https://github.com/deamorim2/sbde/blob/master/wiki/16/pgadmin_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/16/pgadmin_01.png)

Se você carregar dados ou criar uma nova geometria sem especificar um SRID, o valor de SRID será `0`. Lembre-se, na sessão [Geometrias(Dados-Espaciais)](https://github.com/deamorim2/sbde/wiki/09.-Geometrias-(Dados-Espaciais)), quando criamos nossa tabela `geometries`, não especificamos um SRID. Se consultarmos nosso banco de dados, devemos esperar que todas as tabelas do banco `sbde` tenham um SRID de 4674, enquanto que a tabela de geometrias é padronizada com SRID de `0`.

Para ver a atribuição de SRID de uma tabela, consulte a tabela `geometry_columns` do banco de dados `sbde`:

    SELECT f_table_name AS nome, srid
    FROM geometry_columns;

![https://github.com/deamorim2/sbde/blob/master/wiki/16/pgadmin_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/16/pgadmin_02.png)

No entanto, se você souber o SRID das coordenadas, você pode defini-lo (não transformá-lo) usando a função ST_SetSRID na geometria.

    SELECT ST_AsText(ST_Transform(ST_SetSRID(geom,4674), 5880))
    FROM geometries;

![https://github.com/deamorim2/sbde/blob/master/wiki/16/pgadmin_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/16/pgadmin_03.png)

# 16.3 Consultas Avançadas Utilizando Distância

## 16.3.1 Consulta de Distância

>"Quais cidades estão num raio de 45km da capital da cidade do Brasil, cidade de Brasília ('POINT(-47.880611965 -15.783689652)', 4674)"? 

    SELECT nome
    FROM loc_cidade_p
    WHERE ST_DWithin(ST_Transform(geom, 5880), ST_Transform(ST_GeomFromText('POINT(-47.880611965 -15.783689652)', 4674), 5880), 45000);
***
    nome
    ---------------------
    Cidade Ocidental
    Valparaíso de Goiás
    Novo Gama
    Águas Lindas de Goiás
***
Observe que a unidade de medida do SRID 5880 é em metros e por isso o parâmetro de distância da função `ST_DWithin` foi colocado em metros (45.000 metros).

Também podemos responder essa questão utilizando uma junção espacial por meio da seguinte consulta:

    SELECT cid.nome
    FROM loc_capital_p cap JOIN loc_cidade_p cid ON ST_DWithin(ST_Transform(cap.geom, 5880), ST_Transform(cid.geom, 5880), 45000)
    WHERE cap.nome = 'Brasília';

Uma outra maneira de escrever essa consulta é por meio de uma INNER JOIN implícita, ou **equi-join**:

    SELECT cid.nome
    FROM loc_capital_p cap, loc_cidade_p cid
    WHERE ST_DWithin(ST_Transform(cap.geom, 5880), ST_Transform(cid.geom, 5880), 45000)
    AND cap.nome = 'Brasília';

## 16.3.2 Junções Espaciais Avançadas

Vamos supor agora que eu preciso saber "a população total dos municípios num raio de aproximadamente 45km da capital da cidade do Brasil, cidade de Brasília".

A informação de população está na tabela `lim_municipio_a` e não existe nenhum atributo que tenha um identificador único na tabela `loc_cidade_p` que diga a qual município que ele pertença. Apesar de sabermos que o nome da cidade é o mesmo nome do município, existem municípios que possuem nomes repetidos. Sendo assim, podemos utilizar a junção espacial entre as tabelas `loc_cidade_p` e `lim_municipio` para fazer essa relação entre tabelas independente do atributo `nome` de cada uma dessas tabelas.

**Intrução 1 com as cidades e a respectiva população**

    SELECT cid.nome, mue.populacao
    FROM loc_capital_p cap
    JOIN loc_cidade_p cid ON ST_DWithin(ST_Transform(cap.geom,5880), ST_Transform(cid.geom, 5880), 45000)
    JOIN lim_municipio_a mue ON ST_Within (cid.geom, mue.geom)
    WHERE cap.nome = 'Brasília';

**Intrução 2 com o somatório da população das cidades**

    SELECT sum(mue.populacao)
    FROM loc_capital_p cap
    JOIN loc_cidade_p cid ON ST_DWithin(ST_Transform(cap.geom,5880), ST_Transform(cid.geom, 5880), 45000)
    JOIN lim_municipio_a mue ON ST_Within (cid.geom, mue.geom)
    WHERE cap.nome = 'Brasília';

Repare que somente foi necessário fazer a transformação de projeção na função `ST_DWithin`.

# 16.4 Lista de funções

* ST_AsText: Retorna a representação de texto conhecido (WKT) da geometria/geografia sem metadados SRID.
* ST_SetSRID(geometry, srid): Define o SRID em uma geometria para um valor inteiro particular.
* ST_SRID(geometry): Retorna o identificador de referência espacial para o ST_Geometry, conforme definido na tabela spatial_ref_sys.
* ST_Transform(geometry, srid): retorna uma nova geometria com suas coordenadas transformadas no SRID referenciado pelo parâmetro inteiro.
