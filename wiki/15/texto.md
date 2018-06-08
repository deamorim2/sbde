# 15. Indexação Espacial

Lembre-se de que o índice espacial é uma das três principais características de um banco de dados espacial. Os índices são o que possibilitam a utilização de um banco de dados espacial para grandes conjuntos de dados. Sem indexação, qualquer pesquisa por um recurso exigiria uma "varredura sequencial" de cada registro no banco de dados. A indexação acelera a pesquisa, organizando os dados em uma árvore de pesquisa que pode ser rapidamente percorrida para encontrar um registro específico.

Os índices espaciais são um dos maiores ativos do PostGIS. No exemplo anterior, a construção de junções espaciais requer a comparação de tabelas inteiras entre si. Isso pode ficar muito caro: unir duas tabelas de 10.000 registros cada, sem índices, exigiria 100.000.000 de comparações. Com índices, o custo poderia ser tão baixo quanto 20.000 comparações.
Quando carregamos as tabelas espaciais pelo pgShapeLoader, ele criou automaticamente um índice espacial para cada tabela com o nome tabelaespacial_colunageométrica_idx. Para a tabela `lim_municipio_a`, o pgShapeLoader criou um índice espacial chamado `lim_municipio_a_geom_idx`.

Para demonstrar como os índices são importantes para o desempenho, vamos pesquisar quais municípios são cruzados pela BR-040 sem nosso índice espacial.

Nosso primeiro passo é remover o índice espacial de todas as tabelas espaciais envolvidas na consulta:

    DROP INDEX lim_municipio_a_geom_idx;
    DROP INDEX public.tra_trecho_rodoviario_l_geom_idx;

***
**Nota**

A instrução DROP INDEX descarta um índice existente do sistema de banco de dados. Para mais informações, consulte a [documentação](http://www.postgresql.org/docs/7.4/interactive/sql-dropindex.html) do PostgreSQL.
***

Agora, observe o medidor “Timing” no canto inferior direito da janela de consulta do pgAdmin depois de executada a instrução SQL abaixo. Essa consulta pesquisa cada município para identificar se existe um trecho rodoviário da BR-040 cruzando-o, ou não.

    SELECT DISTINCT mue.nome
    FROM lim_municipio_a mue
    JOIN tra_trecho_rodoviario_l rod ON ST_Crosses(rod.geom, mue.geom)
    WHERE rod.codtrechor = 'BR-040';


![https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_01.png)

Observe que essa consulta levou 4.5 segundos para ser executada.

>Agora, recrie os índices espacias para as tabelas.

    CREATE INDEX lim_municipio_a_geom_idx ON public.lim_municipio_a USING gist(geom);
    CREATE INDEX tra_trecho_rodoviario_l_geom_idx ON public.tra_trecho_rodoviario_l USING gist(geom);

>rode novamente a consuta:

    SELECT DISTINCT mue.nome
    FROM lim_municipio_a mue
    JOIN tra_trecho_rodoviario_l rod ON ST_Crosses(rod.geom, mue.geom)
    WHERE rod.codtrechor = 'BR-040';

![https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_02.png)

Repare que a consulta realizada com os índices espaciais levou 1.2 segundos.
***
**Nota**

A cláusula USING GIST diz ao PostgreSQL para usar a estrutura de índice genérica (GIST) ao construir o índice.
***

# 15.1 Como funcionam os índices espaciais

Índices não espaciais de banco de dados criam uma árvore hierárquica com base nos valores da coluna sendo indexada. Os índices espaciais são um pouco diferentes, eles são incapazes de indexar os recursos geométricos e, em vez disso, indexam os limites espaciais das feições geométricas.

![https://github.com/deamorim2/sbde/blob/master/wiki/15/bbox.png](https://github.com/deamorim2/sbde/blob/master/wiki/15/bbox.png)

Na figura acima, o número de linhas que cruzam a estrela amarela é uma, a linha vermelha. Mas as caixas delimitadoras de feições que cruzam a caixa amarela são duas, as vermelhas e azuis.

A forma como o banco de dados responde eficientemente à pergunta “que linhas cruzam a estrela amarela” é primeiro responder à pergunta “que caixas cruzam a caixa amarela” usando o índice (que é muito rápido) e fazer um cálculo exato de “que linhas se cruzam? a estrela amarela” apenas para os recursos retornados pelo primeiro teste. 

Essas "caixas" são chamadas de "Retângulo Envolvente Mínimo", que compreendem o menor retângulo, que esteja paralelo aos eixos de coordenadas, capaz de conter uma determinada característica.

Para uma tabela grande, esse sistema de “duas passadas” de avaliar primeiro o índice aproximado e depois realizar um teste exato pode reduzir radicalmente a quantidade de cálculos necessários para responder a uma consulta.

O PostGIS e o Oracle Spatial compartilham a mesma estrutura de índice espacial Árvore-R ([R-Tree](http://en.wikipedia.org/wiki/R-tree)), porém, com aplicação para dados espaciais ([link](http://postgis.org/support/rtree.pdf)). As Árvores-R dividem os dados em retângulos, sub-retângulos e sub-sub-retângulos, etc. Trata-se de uma estrutura de índice de autoajuste que manipula automaticamente a densidade de dados e o tamanho de objeto variáveis.

![https://github.com/deamorim2/sbde/blob/master/wiki/15/index-01.png](https://github.com/deamorim2/sbde/blob/master/wiki/15/index-01.png)

# 15.2 Consultas que utilizam somente índice

A maioria das funções comumente usadas no PostGIS (ST_Contains, ST_Intersects, ST_DWithin, etc) inclui um filtro de índice espacial automaticamente. Mas algumas funções (por exemplo, ST_Relate) não incluem e indexam o filtro.

Para fazer uma pesquisa de Retângulo Envolvente Mínimo usando o índice (e sem filtragem), faça uso do operador `&&`. Para geometrias, o operador `&&` significa “Retângulos Envolventes Mínimos se sobrepõem ou tocam” da mesma maneira que para o número o `=` operador significa que “os valores são os mesmos”.

Vamos comparar uma consulta utilizando somente índice e uma consulta mais exata.

**Usando o operador `&&` como consulta somente de índice:**

    SELECT DISTINCT mue.nome
    FROM lim_municipio_a mue
    JOIN tra_trecho_rodoviario_l rod ON (rod.geom && mue.geom)
    WHERE rod.codtrechor = 'BR-040';


![https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_03.png)

**Agora vamos fazer a mesma consulta usando a função ST_Intersects, que é mais exata.**

    SELECT DISTINCT mue.nome
    FROM lim_municipio_a mue
    JOIN tra_trecho_rodoviario_l rod ON ST_Intersects(rod.geom, mue.geom)
    WHERE rod.codtrechor = 'BR-040';

![https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/15/pgadmin_04.png)

Uma resposta muito menor! A primeira consulta resumiu todos os blocos que cruzavam a caixa delimitadora dos trechos rodoviários da "BR-040". A segunda consulta apenas resumia os trechos rodoviários que cruzavam os municípios.

# 15.3. Analyzing

O planejador de consulta PostgreSQL escolhe inteligentemente quando usar ou não índices para avaliar uma consulta. Contra intuitivamente, nem sempre é mais rápido fazer uma pesquisa de índice: se a pesquisa vai retornar todos os registros na tabela, percorrer a árvore de índice para obter cada registro será realmente mais lento do que ler linearmente a tabela inteira desde o início.

Para descobrir com qual situação ele está lidando (lendo uma pequena parte da tabela versus lendo uma grande parte da tabela), o PostgreSQL mantém estatísticas sobre a distribuição de dados em cada coluna da tabela indexada.

Por padrão, o PostgreSQL reúne estatísticas regularmente. No entanto, se você mudar drasticamente a composição da sua tabela dentro de um curto período de tempo, as estatísticas não serão atualizadas.

Para garantir que suas estatísticas correspondam ao conteúdo da sua tabela, é aconselhável executar o comando ANALYZE depois que os dados em massa forem carregados e excluídos em suas tabelas. Isso força o sistema de estatísticas a coletar dados para todas as suas colunas indexadas.

O comando ANALYZE pede ao PostgreSQL para percorrer a tabela e atualizar suas estatísticas internas usadas para a estimativa do plano de consulta (a análise do plano de consulta será discutida mais adiante).

    ANALYZE tra_trecho_rodoviario_l;
    ANALYZE lim_municipio_a;

# 15.4 Vacuuming

Vale ressaltar que apenas criar um índice não é suficiente para permitir que o PostgreSQL o use de forma eficaz. O VACUUMing deve ser executado sempre que um novo índice é criado ou após um grande número de UPDATEs, INSERTs ou DELETEs serem emitidos em uma tabela.

O comando VACUUM solicita que o PostgreSQL recupere qualquer espaço não utilizado nas páginas da tabela deixadas por atualizações ou exclusões nos registros.

O VACUUMing é tão essencial para a execução eficiente do banco de dados que o PostgreSQL oferece uma opção de “autovacuum”. Ativado por padrão, o autovacuum remove ambos os VACUUM (recupera espaço) e analisa (atualiza as estatísticas) em suas tabelas em intervalos razoáveis determinados pelo nível de atividade. Embora isso seja essencial para bancos de dados altamente transacionais, não é aconselhável aguardar uma execução de autovacuum após adicionar índices ou dados de carregamento em massa. Se uma grande atualização em lote for executada, você deverá executar manualmente o VACUUM.

Conforme necessário, Vacuuming e Analyzing podem ser executados separadamente no banco de dados. A emissão do comando VACUUM não atualizará as estatísticas do banco de dados. Da mesma forma, a emissão de um comando ANALYZE não recuperará linhas de tabela não utilizadas. Ambos os comandos podem ser executados em todo o banco de dados, em uma única tabela ou em uma única coluna.

>Execute as lindas de comando abaixo um de cada vez:

    VACUUM ANALYZE tra_trecho_rodoviario_l;

    VACUUM ANALYZE lim_municipio_a;

# 15.5 Lista de funções

* geometry_a && geometry_b: retorna TRUE se o retângulo mínimo envolvente de A sobrepor B.
* geometry_a = geometry_b: retorna TRUE se o retângulo mínimo envolvente de A for igual à de B.
* ST_Intersects(geometry_a, geometry_b): Retorna TRUE se as Geometrias/Geografia “intersectarem espacialmente” (compartilham qualquer parte do espaço) e FALSE se não forem (elas são Disjoint).
