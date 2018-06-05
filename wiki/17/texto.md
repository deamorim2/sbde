Aqui está um lembrete de algumas das funções que vimos.

Dica: eles devem ser úteis para os exercícios!

* sum(expressão) - função de agregação para retornar uma soma para um conjunto de registros
* ST_Length(linestring) - retorna o tamanho da cadeia de linhas
* ST_SRID(geometry, srid) - retorna o SRID da geometria
* ST_Transform(geometry, srid) - converte geometrias em diferentes sistemas de referência espacial
* ST_GeomFromText(text) - retorna a geometria
* ST_AsText(geometry) - retorna o texto WKT
* ST_AsGML(geometry) - retorna o texto GML

Vamos retomar alguns exercícios realizados nas sessões anteriores que utilizam distâncias de forma aproximada.

>"Qual é a área do município de 'Patos de Minas', tabela `lim_municipio_a`, determinada pelo IBGE? Coloque as medidas em km2"

    SELECT ST_Area(geom)*12321 as area_geo, ST_Area(ST_Transform(geom, 55555))/1000000 as area_albers, ST_Area(ST_Transform(geom, 5880))/1000000 as area_policonica 
    FROM lim_municipio_a
    WHERE nome = 'Patos de Minas';

>“Qual é o comprimento total das rodovias (em quilômetros) do Brasil?”

    SELECT Sum(ST_Length(geom))*111 as comp_geo, Sum(ST_Length((ST_Transform(geom, 5880))))/1000 as comp_policonica
    FROM tra_trecho_rodoviario_l;

>“Qual é o comprimento de todos os trechos rodoviários da 'BR-101' (em quilômetros)?”

    SELECT Sum(ST_Length((ST_Transform(geom, 5880))))/1000 
    FROM tra_trecho_rodoviario_l
    WHERE codtrechor = 'BR-101';

>“Qual é o comprimento total dos tipos de trechos rodoviários (em quilômetros)?”

    SELECT tipotrecho, Sum(ST_Length((ST_Transform(geom, 5880))))/1000
    FROM tra_trecho_rodoviario_l
    GROUP BY tipotrecho
    ORDER BY tipotrecho;

>Usando a geometria em WKT da cidade de "Patos de Minas" ('POINT(-46.522980843 -18.5911930409999)'), podemos encontrar as rodovias próximas (a menos de aproximadamente 5 km) dessa cidade:

    SELECT DISTINCT codtrechor
    FROM tra_trecho_rodoviario_l
    WHERE ST_DWithin(ST_Transform(geom, 5880), ST_Transform(ST_GeomFromText('POINT(-46.522980843 -18.5911930409999)',4674),5880),5000)
    AND codtrechor is not null;

>"Quais cidades estão num raio de aproximadamente 45km da capital da cidade do Brasil, cidade de Brasília"?

    SELECT nome, ST_Distance(ST_Transform(geom, 5880), ST_Transform(ST_GeomFromText('POINT(-47.880611965 -15.783689652)', 4674),5880))/1000
    FROM loc_cidade_p
    WHERE ST_DWithin(ST_Transform(geom, 5880), ST_Transform(ST_GeomFromText('POINT(-47.880611965 -15.783689652)', 4674),5880), 45000);

>"Qual é a área do município de 'Patos de Minas', tabela `lim_municipio_a`, em quilômetros quadrados?"

    SELECT ST_Area(ST_Transform(geom, 55555))/1000000
    FROM lim_municipio_a
    WHERE nome = 'Patos de Minas';

>"Quantos aeroportos existem a 100km da Terra Indígena de 'Japuira'?"

    SELECT count(*)
    FROM
    (
        SELECT ppl.gid
        FROM lim_terra_indigena_a tei
        JOIN tra_pista_ponto_pouso_l ppl ON ST_DWithin(ST_Transform(ppl.geom, 5880), ST_Transform(tei.geom, 5880), 100000)
        WHERE tei.nome = 'Japuira'

        UNION

        SELECT ppp.gid
        FROM lim_terra_indigena_a tei
        JOIN tra_pista_ponto_pouso_p ppp ON ST_DWithin(ST_Transform(ppp.geom, 5880), ST_Transform(tei.geom, 5880), 100000)
        WHERE tei.nome = 'Japuira'
     ) as a;
