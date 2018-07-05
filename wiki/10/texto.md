# 10. Exercícios de Dados Espaciais Geométricos

Aqui está um lembrete de todas as funções que vimos até agora. Eles devem ser úteis para os exercícios!

* sum(expression) - função de agregação para retornar uma soma para um conjunto de registros
* count(expression) - função de agregação para retornar o tamanho de um conjunto de registros
* ST_GeometryType(geometry) - retorna o tipo da geometria
* ST_NDims(geometry) - retorna o número de dimensões da geometria
* ST_SRID(geometry) - retorna o número do identificador de referência espacial da geometria
* ST_X(point) - retorna a ordenada X
* ST_Y(point) - retorna a ordenada em Y
* ST_Length(linestring) - retorna o tamanho da cadeia de linhas
* ST_StartPoint(geometry) - retorna a primeira coordenada como um ponto
* ST_EndPoint(geometry) - retorna a última coordenada como um ponto
* ST_NPoints(geometry) - retorna o número de coordenadas na cadeia de linhas
* ST_Area(geometry) - retorna a área dos polígonos
* ST_NRings(geometry) - retorna o número de anéis (geralmente 1, mais de 1 se houver furos)
* ST_ExteriorRing(polygon) - retorna o anel externo como uma cadeia de linhas
* ST_InteriorRingN(polygon, integer) - retorna um anel interno especificado como uma linestring
* ST_Perimeter(geometry) - retorna o comprimento de todos os anéis
* ST_NumGeometries(multi / geomcollection) - retorna o número de partes na coleção
* ST_GeometryN(geometry, integer) - retorna a parte especificada da coleção
* ST_GeomFromText(text) - retorna a geometria
* ST_AsText(geometry) - retorna o texto WKT
* ST_AsEWKT(geometry) - retorna o texto EWKT
* ST_GeomFromWKB (bytea) - retorna a geometria
* ST_AsBinary(geometry) - retorna WKB bytea
* ST_AsEWKB(geometry) - retorna o byte EWKB
* ST_GeomFromGML(text) - retorna a geometria
* ST_AsGML(geometry) - retorna o texto GML
* ST_GeomFromKML(text) - retorna geometria
* ST_AsKML(geometry) - retorna o texto KML
* ST_AsGeoJSON(geometry) - retorna o texto JSON
* ST_AsSVG(geometry) - retorna o texto SVG

# 10.1. Exercícios

>"Quantos municípios do Brasil possui um buraco em sua geometria?"

    SELECT Count(*)
    FROM lim_municipio_a
    WHERE ST_NumInteriorRings(ST_GeometryN(geom,1)) > 0;
***
**Nota**
 as funções ST_NRings() podem ser tentadoras, mas também conta os anéis externos de multi-polígonos, bem como os anéis internos. Para executar ST_NumInteriorRings (), precisamos converter as geometrias de MultiPolygon dos município em polígonos simples, portanto, extraímos o primeiro polígono de cada coleção usando ST_GeometryN().
***
>"Qual é a representação JSON do limite do estado do 'Sergipe'?"

    SELECT ST_AsGeoJSON(geom)
    FROM lim_unidade_federacao_a
    WHERE nome = 'Sergipe';

>"Qual é a representação em WKT do limite do estado do 'Sergipe'?"

    SELECT ST_AsText(geom)
    FROM lim_unidade_federacao_a
    WHERE nome = 'Sergipe';

>"Qual é a representação em EWKT do limite do estado do 'Sergipe'?"

    SELECT ST_AsEWKT(geom)
    FROM lim_unidade_federacao_a
    WHERE nome = 'Sergipe';

>"Quais Município possuem mais de um polígono?"

    SELECT nome, ST_NumGeometries(geom)
    FROM lim_municipio_a
    WHERE ST_NumGeometries(geom) > 1
    ORDER BY ST_NumGeometries(geom) DESC;

***
**Nota**
 não é incomum encontrar MultiPolygons de elemento único em tabelas espaciais. O uso de MultiPolygons permite que uma tabela com apenas um tipo de geometria armazene geometrias simples e múltiplas sem usar tipos mistos.
***

