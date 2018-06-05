Junções espaciais são o carro chefe de sistema de bancos de dados espaciais.

Elas permitem combinar informações de diferentes tabelas usando relacionamentos espaciais como a chave de junção. Muito do que pensamos como “análise padrão de GIS” pode ser expresso como junções espaciais.

Na seção anterior, exploramos as relações espaciais usando um processo de duas etapas: 1) primeiro extraímos um ponto da tabela de capitais (`loc_capital_p`), nesse caso, a cidade de Brasília; 2) A seguir, usamos esse ponto para fazer mais perguntas como "Em qual unidade da federação está localizada a capital do Brasil, a cidade de Brasília?"

Usando uma junção espacial (ST_Intersects/ST_Contains/ST_Within), podemos responder à pergunta em uma etapa, recuperando informações sobre a cidade (tabela `loc_capital_p`) e a unidade da federação (tabela `lim_unidade_federacao_a`) que a contém.

**Instrução 1**

    SELECT cap.nome AS cap_nome, ufe.nome AS ufe_nome
    FROM lim_unidade_federacao_a ufe JOIN loc_capital_p cap ON ST_Contains(ufe.geom, cap.geom)
    WHERE cap.nome = 'Brasília';

**Instrução 2**

    SELECT cap.nome AS cap_nome, ufe.nome AS ufe_nome
    FROM lim_unidade_federacao_a ufe JOIN loc_capital_p cap ON ST_Within(cap.geom, ufe.geom)
    WHERE cap.nome = 'Brasília';

**Instrução 3**

    SELECT cap.nome AS cap_nome, ufe.nome AS ufe_nome
    FROM lim_unidade_federacao_a ufe JOIN loc_capital_p cap ON ST_Intersects(ufe.geom, cap.geom)
    WHERE cap.nome = 'Brasília';

**Instrução 4**

    SELECT cap.nome AS cap_nome, ufe.nome AS ufe_nome
    FROM lim_unidade_federacao_a ufe JOIN loc_capital_p cap ON ST_Intersects(cap.geom, ufe.geom)
    WHERE cap.nome = 'Brasília';

![https://github.com/deamorim2/sbde/blob/master/wiki/13/pgadmin_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/13/pgadmin_01.png)
***
**Nota:**

Repare que nas instruções 1 e 2 a utilização das funções espaciais `ST_Contains` e `ST_Within` leva em consideração a sintaxe das geometrias `A` e `B`. No primeiro, `Unidade da Federação` CONTÉM uma `Capital`, enquanto que no segundo exemplo `Capital` ESTÁ DENTRO da `Unidade da Federação`.
Já nas instruções 3 e 4, tanto faz na sintaxe da função `ST_Intersects` quem é `A` ou `B`.
***


Poderíamos fazer a junção espacial de todas as capitais às unidades da federação que as contém, mas, nesse caso, queríamos informações sobre apenas uma, a cidade de Brasília. Qualquer função que forneça um relacionamento verdadeiro/falso entre duas tabelas pode ser usada para conduzir uma junção espacial.

# 13.1. Junção e Resumo

A combinação de um JOIN com um GROUP BY fornece o tipo de análise que normalmente é feita em um sistema GIS.

Por exemplo: “Qual é o número de cidades por unidade da federação?” Aqui temos uma pergunta que combina informações de cidades com os limites das unidades da federação.

    SELECT ufe.nome, count(cid.nome) as contagem
    FROM lim_unidade_federacao_a ufe JOIN loc_cidade_p cid ON ST_Intersects(ufe.geom, cid.geom)
    GROUP BY ufe.nome
    ORDER BY count(cid.nome) DESC;

![https://github.com/deamorim2/sbde/blob/master/wiki/13/pgadmin_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/13/pgadmin_02.png)

Repare que o resultado da consulta acima apresenta apenas 26 das 27 unidades da federação. Isso ocorre porque a totalidade das sedes municipais englobam os registros das tabelas `loc_cidade_p` e `loc_capital_p`. Como a capital do distrito Federal, Brasília, não está na tabela `loc_cidade_p`, aparecem apenas 26 registros. Para resolvermos isso, devemos fazer uma consulta com junção do tipo LEFT OUTER JOIN para listar todas as unidades da federação, mesmo não tendo intersecção espacial da cidade de Brasília e o a unidade da federação do Distrito Federal e adicionar o valor 1 para representar a cidade que está presente como a capital da unidade da federação, já que cada unidade da federação possui uma capital.

    SELECT ufe.nome, count(cid.nome)+1 as contagem
    FROM lim_unidade_federacao_a ufe LEFT OUTER JOIN loc_cidade_p cid ON ST_Intersects(ufe.geom, cid.geom)
    GROUP BY ufe.nome
    ORDER BY count(cid.nome) DESC;

![https://github.com/deamorim2/sbde/blob/master/wiki/13/pgadmin_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/13/pgadmin_03.png)

# 13.3. Lista de funções

* ST_Contains (geometry A, geometry B): Retorna verdadeiro se e somente se nenhum ponto de B estiver no exterior de A, e pelo menos um ponto do interior de B estiver no interior de A.
* ST_DWithin (geometry A, geometry B, raio): Retorna verdadeiro se as geometrias estiverem dentro da distância especificada uma da outra.
* ST_Intersects (geometry A, geometry B): retorna TRUE se as Geometrias/Geografia “intersectarem espacialmente” (compartilham qualquer parte do espaço) e FALSE se não forem (elas são Disjoint).

# 13.4 Referência Bibliográfica

[Manual PostGIS 2.1](http://postgis.net/docs/manual-2.1/)