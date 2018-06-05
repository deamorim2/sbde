# 23. Referenciamento Linear

A referência linear é um meio de representar recursos que podem ser descritos referenciando um conjunto básico de recursos lineares.

Exemplos comuns de recursos que são modelados usando referência linear são:

* Pontos de interesse rodoviário, que são referenciados usando distância ao longo de uma rede rodoviária
* Operações de manutenção de estradas, que são referenciadas como ocorrências ao longo de uma rede rodoviária entre um par de medições de distâncias
* Inventários aquáticos, onde a presença de peixes é registrada como existente entre um par de medições de quilometragem a montante.
* Caracterizações hidrológicas (“alcances”) de fluxos, registradas com uma quilometragem

O benefício dos modelos de referência linear é que as observações espaciais dependentes não precisam ser gravadas separadamente das observações de base, e atualizações na camada de observação base podem ser realizadas sabendo que as observações dependentes rastrearão automaticamente a nova geometria.
***
**Nota**
A convenção ESRI para referência linear é ter uma tabela base de recursos espaciais lineares e uma tabela não espacial de “eventos” que inclui uma referência de chave estrangeira para o recurso espacial e uma medida ao longo do recurso referenciado. Usaremos o termo “tabela de interferências” para nos referirmos às tabelas não espaciais que construímos.
***

# 23.1. Criando Referências Lineares

Se você tiver uma tabela de pontos existente que deseja referenciar a uma rede linear, use a função **ST_LineLocatePoint**, que usa uma linha e um ponto, e retorna a proporção ao longo da linha em que o ponto pode ser encontrado.

Exemplo simples de localizar um ponto a meio caminho ao longo de uma linha:

    SELECT ST_LineLocatePoint('LINESTRING(0 0, 2 2)', 'POINT(1 1)');
***
     0.5
***

E se o ponto não estiver na linha? Projeta-se ao ponto o mais próximo:

    SELECT ST_LineLocatePoint('LINESTRING(0 0, 2 2)', 'POINT(0 2)');
***
    0.5
***

Podemos converter as `loc_cidades_p` em uma "tabela de interferências" relativa aos `tra_trecho_rodoviario_l` usando o ST_LineLocatePoint:

Todo o SQL abaixo serve para criar a nova tabela de interferências:

    DROP TABLE IF EXISTS cidade_interfere_rodovia;

    CREATE TABLE cidade_interfere_rodovia AS

    WITH ordered_nearest AS (
    SELECT
      ST_GeometryN(rod.geom,1) AS rod_geom,
      rod.gid AS rod_gid,
      cid.geom AS cid_geom,
      cid.gid AS cid_gid,
      ST_Distance((rod.geom)::geography, (cid.geom)::geography) AS distance
    FROM tra_trecho_rodoviario_l rod
      JOIN loc_cidade_p cid
      ON ST_DWithin((rod.geom)::geography, (cid.geom)::geography, 1000)
    ORDER BY cid_gid, distance ASC
    )

    SELECT DISTINCT ON (cid_gid) cid_gid, rod_gid, ST_LineLocatePoint(rod_geom, cid_geom) AS measure, distance
    FROM ordered_nearest;

Primeiro, precisamos obter um conjunto de cidades candidatas, talvez a mais próxima das rodovias, ordenadas por id e distância.

Usamos o recurso `DISTINCT ON` do PostgreSQL para obter a primeira rodovia (a mais próxima) para cada rodovia única. Podemos então passar essa rodovia para ST_LineLocatePoint junto com sua cidade candidata para calcular a distância.

Por fim, criamos a chave-primária da tabela `cidade_interfere_rodovia`

    ALTER TABLE cidade_interfere_rodovia ADD PRIMARY KEY (cid_gid);

Quando tivermos uma tabela de interferências, é divertido transformá-la em uma visualização espacial, para que possamos visualizar as interferências relativas aos pontos originais das quais foram derivadas.

Para ir de uma medida para um ponto, usamos a função **ST_LineInterpolatePoint**.

Abaixo estão nossos exemplos simples anteriores invertidos. Nesse caso, é um exemplo simples de localizar um ponto a meio caminho ao longo de uma linha:

    SELECT ST_AsText(ST_LineInterpolatePoint('LINESTRING(0 0, 2 2)', 0.5));
***
    POINT(1 1)
***

E podemos unir as tabelas `cidade_interfere_rodovia` de volta à tabela `tra_trecho_rodoviario_l` e usar o atributo `measure` para gerar os pontos de interferências espaciais, sem referenciar a tabela `loc_cidade_p` original:

    DROP VIEW IF EXISTS loc_cidade_p_rodovia;

    CREATE OR REPLACE VIEW loc_cidade_p_rodovia AS
    SELECT
      cir.cid_gid,
      ST_LineInterpolatePoint(ST_GeometryN(rod.geom, 1), cir.measure) AS geom,
      cir.rod_gid
    FROM cidade_interfere_rodovia cir
    JOIN tra_trecho_rodoviario_l rod
    ON (rod.gid = cir.rod_gid);

Ao visualizar os pontos original (estrela vermelha) e interferência (círculo azul) com as rodovias, você pode ver como as interferências são encaixadas diretamente nas linhas das rodovias que estão até 2km da cidade mais próxima.

![https://github.com/deamorim2/sbde/blob/master/wiki/23/qgis_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/23/qgis_01.png)

***
**Nota**
Um uso surpreendente das funções de referência linear não tem nada a ver com modelos de referência linear. Como mostrado acima, é possível usar as funções para encaixar pontos em recursos lineares. Para casos de uso, como trilhas de GPS ou outras entradas que devem referenciar uma rede linear, o snap é um recurso útil disponível.
***

# 23.2. Lista de funções

* ST_LineInterpolatePoint(geometria A, medida dupla): Retorna um ponto interpolado ao longo de uma linha.

* ST_LineLocatePoint (geometria A, geometria B): retorna um ponto flutuante entre 0 e 1 representando a localização do ponto mais próximo em LineString para o ponto especificado.

* ST_Line_Substring (geometria A, double de, double para): Retorna uma cadeia de linha sendo uma subseqüência da entrada, uma começando e terminando nas frações especificadas do total de 2d de comprimento.

* ST_Locate_Along_Measure (geometria A, medida dupla): retorna um valor de coleta de geometria derivada com elementos que correspondem à medida especificada.

* ST_Locate_Between_Measures (geometria A, double from, double to): retorna um valor de coleção de geometria derivada com elementos que correspondem ao intervalo especificado de medidas, inclusive.

* ST_AddMeasure (geometria A, dupla de, double para): retorna uma geometria derivada com elementos de medida linearmente interpolados entre os pontos inicial e final. Se a geometria não tiver dimensão de medida, será adicionado o valor 1.
