# 12. Exercícios de Relacionamentos Espaciais

Aqui está um lembrete das funções que vimos na última seção. Eles devem ser úteis para os exercícios!

* sum(expression) - função de agregação para retornar uma soma para um conjunto de registros
* count(expression) - função de agregação para retornar o tamanho de um conjunto de registros
* ST_Contains(geometria A, geometria B) - retorna verdadeiro se a geometria A contiver geometria B
* ST_Crosses(geometria A, geometria B) - retorna verdadeiro se a geometria A cruzar a geometria B
* ST_Disjoint(geometria A, geometria B) - retorna verdadeiro se as geometrias não “intersectarem espacialmente”
* ST_Distance(geometria A, geometria B) - retorna a distância mínima entre a geometria A e a geometria B
* ST_DWithin(geometria A, geometria B, raio) - retorna verdadeiro se a geometria A estiver com raio de distância ou menor da geometria B
* ST_Equals(geometria A, geometria B) - retorna verdadeiro se a geometria A for igual à geometria B
* ST_Intersects(geometria A, geometria B) - retorna verdadeiro se a geometria A cruzar a geometria B
* ST_Overlaps(geometria A, geometria B) - retornará verdadeiro se a geometria A e a geometria B compartilharem espaço, mas não estiverem completamente contidas entre si.
* ST_Touches(geometria A, geometria B) - retorna verdadeiro se o limite da geometria A toca na geometria B
* ST_Within(geometria A, geometria B) - retorna verdadeiro se a geometria A estiver dentro da geometria B

# 12.1 Exercícios

>"Qual é a geometria em WKT da capital do Brasil, a cidade de Brasília da tabela `loc_capital_p`?"

    SELECT ST_AsText(geom)
    FROM loc_capital_p
    WHERE nome = 'Brasília';

>"Em qual unidade da federação está localizada a capital do Brasil, a cidade de Brasília?"

    SELECT nome
    FROM lim_unidade_federacao_a
    WHERE ST_Intersects(geom, ST_GeomFromText('POINT(-47.880611965 -15.783689652)', 4674));

