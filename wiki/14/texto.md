Aqui está um lembrete de algumas das funções que vimos. Dica: eles devem ser úteis para os exercícios!

* sum(expression): função de agregação para retornar uma soma para um conjunto de registros
* count(expression): função de agregação para retornar o tamanho de um conjunto de registros
* ST_Area(geometry): retorna a área dos polígonos
* ST_AsText(geometry): retorna o texto WKT
* ST_Contains(geometry A, geometry B): retorna o verdadeiro se a geometria A contiver geometria B
* ST_Distance(geometry A, geometry B): retorna a distância mínima entre a geometria A e a geometria B
* ST_DWithin(geometry A, geometry B, raio): retorna o verdadeiro se a geometria A é a distância do raio ou menor da geometria B
* ST_GeomFromText(text): retorna a geometria
* ST_Intersects(geometry A, geometry B): retorna o verdadeiro se a geometria A cruzar a geometria B
* ST_Length(Linestrings): retorna o tamanho da cadeia de linhas
* ST_Touches(geometry A, geometry B): retorna o verdadeiro se o limite da geometria A toca na geometria B
* ST_Within(geometry A, geometry B): retorna o verdadeiro se a geometria A estiver dentro da geometria B

# 14.1 Exercícios

Utilizando as tabelas `lim_municipio_a` e `tra_trecho_rodoviario_l`, responda as seguintes questões:

>"A BR-040 cruza quais municípios?"

    SELECT DISTINCT mue.nome
    FROM lim_municipio_a mue
    JOIN tra_trecho_rodoviario_l rod ON ST_Crosses(rod.geom, mue.geom)
    WHERE rod.codtrechor = 'BR-040';

>"Quais Terras Indígenas estão complemante inseridas dentro do município de 'Juara', geocódigo 5105101?"

    SELECT tei.nome
    FROM lim_municipio_a mue
    JOIN lim_terra_indigena_a tei ON ST_Within(tei.geom, mue.geom)
    WHERE mue.geocodigo = '5105101';

>"Quais Terras Indígenas sobrepõem o município de 'Juara', geocódigo 5105101?"

    SELECT tei.nome
    FROM lim_municipio_a mue
    JOIN lim_terra_indigena_a tei ON ST_Overlaps(tei.geom, mue.geom)
    WHERE mue.geocodigo = '5105101';

>"Quais Terras Indígenas interagem espacialmente com o município de 'Juara', geocódigo 5105101?"

    SELECT tei.nome
    FROM lim_municipio_a mue
    JOIN lim_terra_indigena_a tei ON ST_Intersects(tei.geom, mue.geom)
    WHERE mue.geocodigo = '5105101';
