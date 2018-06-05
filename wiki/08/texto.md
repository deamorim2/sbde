# 8. Junção de Tabelas em SQL(Join)

Em um banco de dados podemos ter duas ou mais tabelas relacionadas.

É comum ao elaborarmos uma consulta termos a necessidade de trazer dados de diferentes tabelas.

Para criarmos esta seleção devemos definir os critérios de agrupamento para trazer estes dados.

Estes critérios são chamados de Junções.

Uma junção de tabelas cria uma pseudo-tabela derivada de duas ou mais tabelas de acordo com as regras especificadas, e que são parecidas com as regras da teoria dos conjuntos.

Para saber mais sobre esse assunto, [Clique Aqui!](https://en.wikipedia.org/wiki/Join_(SQL)) [ou Aqui!](https://pt.wikiversity.org/wiki/Introdução_ao_SQL/Junções)

### Simplificadamente temos três tipos de junções entre tabelas:

A) **Inner Join** -  todas linhas de uma tabela se relacionam com todas as linhas de outras tabelas se elas tiverem ao menos 1 campo em comum 
   1. Equi-join - Uma equi-join é um tipo específico de junção baseada em comparador, que usa apenas comparações de igualdade no predicado de junção. Usando outros operadores de comparação (como <) desqualifica uma associação como uma junção equi.
   2. Natural join - A junção natural é um caso especial de equi-join. A junção natural (⋈) é um operador binário que é escrito como (R ⋈ S) onde R e S são relações. O resultado da junção natural é o conjunto de todas as combinações de tuplas em R e S que são iguais em seus nomes de atributos comuns.

B) **Outer Join** - é uma seleção que não requer que os registros de uma tabela possuam registros equivalentes em outras 
   1. Left Outer Join - todos os registros da tabela esquerda mesmo quando não exista registros correspondentes na tabela direita.
   1. Right Outer Join - todos os registros da tabela direita mesmo quando não exista registros correspondentes na tabela esquerda.
   1. Full Outer Join - esta operação apresenta todos os dados das tabelas à esquerda e à direita, mesmo que não possuam correspondência em outra tabela

C) **Self-Join** - Uma auto junção é juntar uma tabela a si mesma.

Quadro simplificados com os tipos de junções SQL entre tabelas:

![https://github.com/deamorim2/sbde/blob/master/wiki/07/SQL_Joins.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/SQL_Joins.png)

Os exemplos a seguir foram baseados no conteúdo do site [https://en.wikipedia.org/wiki/Join_(SQL)](https://en.wikipedia.org/wiki/Join_(SQL)) para o ambiente PostgreSQL.

>Crie as tabelas `department` e `employee` e insira os dados de cada uma a partir das instruções abaixo no banco de dados `sbde`:

    CREATE TABLE department
    (
    departmentid integer,
    departmentname varchar(20)
    );

    CREATE TABLE employee
    (
    lastname VARCHAR(20),
    departmentid integer
    );

    INSERT INTO department VALUES(31, 'Sales');
    INSERT INTO department VALUES(33, 'Engineering');
    INSERT INTO department VALUES(34, 'Clerical');
    INSERT INTO department VALUES(35, 'Marketing');

    INSERT INTO employee VALUES('Rafferty', 31);
    INSERT INTO employee VALUES('Jones', 33);
    INSERT INTO employee VALUES('Heisenberg', 33);
    INSERT INTO employee VALUES('Robinson', 34);
    INSERT INTO employee VALUES('Smith', 34);
    INSERT INTO employee VALUES('Williams', NULL);

## 8.1 Cross-Join

**CROSS JOIN** retorna o produto cartesiano de linhas das tabelas na associação. Em outras palavras, ele produzirá linhas que combinam cada linha da primeira tabela com cada linha da segunda tabela.

**Exemplo de uma junção cruzada explícita:**

    SELECT *
    FROM employee CROSS JOIN department;

**Exemplo de uma junção cruzada não explícita:**

    SELECT *
    FROM employee, department;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_24.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_24.png)

A **junção cruzada** (cross-join) não aplica nenhum predicado para filtrar linhas da tabela unida. Os resultados de uma união cruzada podem ser filtrados usando uma cláusula WHERE, que pode então produzir o equivalente a uma união interna.
No padrão [SQL 2011](https://en.wikipedia.org/wiki/SQL:2011), as junções cruzadas fazem parte do pacote opcional F401, "Extended joined table".

## 8.2 Inner-join

Uma **junção interna** requer que cada linha nas duas tabelas unidas tenha valores de coluna correspondentes e é uma operação de junção comumente usada em aplicativos, mas não deve ser considerada a melhor escolha em todas as situações.

A **junção interna** cria uma nova tabela de resultados combinando valores de coluna de duas tabelas (A e B) com base no predicado de junção. A consulta compara cada linha de `A` com cada linha de `B` para localizar todos os pares de linhas que satisfazem o predicado de união.

Quando o predicado de união é satisfeito pela correspondência de valores não-NULOS, os valores de coluna para cada par de linhas correspondentes de `A` e `B` são combinados em uma linha de resultado.

O resultado da junção pode ser definido como o resultado de se obter o produto cartesiano (ou cross-join) de todas as linhas nas tabelas (combinando todas as linhas da `tabela A` com todas as linhas da `tabela B`) e retornando todas as linhas que satisfazem a linha junto ao predicado. As implementações reais de SQL normalmente usam outras abordagens, como [hash-join](https://en.wikipedia.org/wiki/Hash_join) ou [sort-merge joins](https://en.wikipedia.org/wiki/Sort-merge_join), pois o cálculo do produto cartesiano é mais lento e geralmente exigiria uma quantidade proibitivamente grande de memória para armazenamento.

O SQL especifica duas formas sintáticas diferentes para expressar as junções: a**notação de junção explícita** e a **notação de junção implícita**. A **notação de junção implícita** não é mais considerada uma prática recomendada, embora os sistemas de banco de dados ainda a suportem.

A **notação de junção explícita** usa a palavra-chave `JOIN`, opcionalmente precedida pela palavra-chave `INNER`, para especificar a tabela a ser unida e a palavra-chave `ON` para especificar os predicados para a junção, como no exemplo a seguir:

    SELECT employee.lastname, employee.departmentid, department.departmentname 
    FROM employee INNER JOIN department ON employee.departmentid = department.departmentid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_25.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_25.png)

A **notação de junção implícita** simplesmente lista as tabelas para unir, na cláusula `FROM` da instrução `SELECT`, usando vírgulas para separá-las. Assim, ele especifica uma união cruzada, e a cláusula `WHERE` pode aplicar predicados de filtro adicionais (que funcionam comparativamente aos predicados de união **na notação explícita**).

O exemplo a seguir é equivalente ao anterior, mas desta vez usando **notação de junção implícita**:

    SELECT *
    FROM employee, department
    WHERE employee.departmentid = department.departmentid;

As consultas fornecidas nos exemplos acima se juntarão às tabelas `employee` e `department` usando a coluna `departmentid` das duas tabelas. Onde o `departmentid` dessas tabelas corresponde (ou seja, o predicado de união é satisfeito), a consulta combinará as colunas `lastname`, `departmentid` e `departmentname` das duas tabelas em uma linha de resultado. Onde o `departmentid` não corresponde, nenhuma linha de resultado é gerada.

Assim, o resultado da execução da consulta acima será:

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_26.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_26.png)

O funcionário "Williams" e o departamento "Marketing" não aparecem nos resultados da execução da consulta. Nenhum deles tem linhas correspondentes na outra tabela respectiva: "Williams" não possui departamento associado e nenhum funcionário possui o `id` de departamento 35 ("Marketing"). Dependendo dos resultados desejados, esse comportamento pode ser um bug sutil, que pode ser evitado substituindo a junção interna por uma junção externa.

Os programadores devem tomar cuidado especial ao unir tabelas em colunas que podem conter valores nulos, pois NULL nunca corresponderá a nenhum outro valor (nem mesmo NULL), a menos que a condição de associação use explicitamente um predicado de combinação que primeiro verifique se as colunas de junções são NOT NULL antes de aplicar a(s) condição(ões) de predicado restante(s).

A junção Interna só pode ser usada com segurança em um banco de dados que imponha a integridade referencial ou onde as colunas de junção não sejam nulos. Muitos bancos de dados relacionais de processamento de transações contam com os padrões de atualização de dados Atomicity, Consistency, Isolation, Durability (ACID) para garantir a integridade dos dados, tornando as junções internas uma opção apropriada. No entanto, os bancos de dados transacionais geralmente também possuem colunas de junção desejáveis ​​que podem ser nulos.

Muitos softwares de geração de relatórios baseados em bancos de dados relacionais e [Data Warehouse](https://en.wikipedia.org/wiki/Data_warehouse) usam atualizações em lote de extração, transformação, carga ([ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load)) de alto volume que dificultam ou impossibilitam a imposição da integridade referencial, resultando em colunas de junção potencialmente NULL que um autor de consulta SQL não pode modificar e que fazem com que junções internas omitidas dados sem indicação de erro. A opção de usar uma junção interna depende do design do banco de dados e das características dos dados. Uma junção externa esquerda (Left Outer Join)geralmente pode ser substituída por uma junção interna quando as colunas de junção em uma tabela podem conter valores NULL.

Qualquer coluna de dados que pode ser NULL (vazia) nunca deve ser usada como um link em uma junção interna, a menos que o resultado pretendido seja eliminar as linhas com o valor NULL. Se nas colunas de junção os valores NULL forem deliberadamente removidos do conjunto de resultados, uma junção interna poderá ser mais rápida que uma junção externa, pois a junção e a filtragem da tabela serão feitas em uma única etapa. Por outro lado, uma junção interna pode resultar em desempenho desastrosamente lento ou mesmo em uma falha do servidor quando usada em uma consulta de grande volume em combinação com funções de banco de dados em uma cláusula SQL do tipo `WHERE`. Uma função em uma cláusula SQL `WHERE` pode resultar no banco de dados ignorando índices de tabela relativamente compactos. O banco de dados pode ler e unir internamente as colunas selecionadas de ambas as tabelas antes de reduzir o número de linhas usando o filtro que depende de um valor calculado, resultando em uma quantidade relativamente grande de processamento ineficiente.

Quando um conjunto de resultados é produzido unindo várias tabelas, incluindo tabelas mestras usadas para procurar descrições de texto completo de códigos identificadores numéricos (uma tabela [Lookup](https://en.wikipedia.org/wiki/Lookup_table)), um valor NULL em qualquer uma das chaves estrangeiras pode resultar na eliminação da linha inteira o conjunto de resultados, sem indicação de erro. Uma consulta SQL complexa que inclui uma ou mais junções internas e várias junções externas tem o mesmo risco de valores NULL nas colunas do link de junção interna.

Um compromisso com o código SQL contendo junções internas pressupõe que as colunas de junção NULL não serão introduzidas por alterações futuras, incluindo atualizações de fornecedor, alterações de design e processamento em massa fora das regras de validação de dados do aplicativo, como conversões de dados, migrações, importações em massa e mesclagens.

Pode-se ainda classificar as junções internas como **Equi-join** (junções de igualdade), **Natural Joins** (junções naturais) ou **Cross-join** (junções cruzadas).

### 8.2.1 Equi-Join

Uma equi-join é um tipo específico de junção baseada em comparador, que usa apenas comparações de igualdade no predicado de junção. Usando outros operadores de comparação (como <) desqualifica uma associação como uma **equi-join**. A consulta mostrada acima já forneceu um exemplo de **equi-join**:

    SELECT *
    FROM employee JOIN department
    ON employee.departmentid = department.departmentid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_27.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_27.png)

Podemos escrever **equi-join** como:

    SELECT *
    FROM employee, department
    WHERE employee.departmentid = department.departmentid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_28.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_28.png)

Se as colunas em uma **equi-join** tiver o mesmo nome, o [SQL-92](https://en.wikipedia.org/wiki/SQL-92) fornece uma notação abreviada opcional para expressar equivalências de junção, por meio do constructo USING:

    SELECT *
    FROM employee INNER JOIN department USING (departmentid);

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_29.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_29.png)

O constructo `USING` é mais do que mero açúcar sintático, no entanto, uma vez que o conjunto de resultados difere do conjunto de resultados da versão com o predicado explícito. Especificamente, todas as colunas mencionadas na lista `USING` aparecerão somente uma vez, com um nome não qualificado, em vez de uma vez para cada tabela na junção. No caso acima, haverá uma única coluna `departmentid` e nenhum `employee.departmentid` ou `department.departmentid`.

### 8.2.2 Natural Join

A junção natural é um caso especial de **equi-join**. A junção natural (`⋈`) é um operador binário que é escrito como (`R ⋈ S`) onde `R` e `S` são relações. O resultado da junção natural é o conjunto de todas as combinações de tuplas em `R` e `S` que são iguais em seus nomes de atributos comuns.

Uma junção natural é um tipo de **equi-join** em que o predicado de junção surge implicitamente comparando todas as colunas em ambas as tabelas que possuem os mesmos nomes de coluna nas tabelas unidas. A tabela unida resultante contém apenas uma coluna para cada par de colunas igualmente nomeadas. No caso de não haver colunas com os mesmos nomes, o resultado é uma união cruzada.

A maioria dos especialistas concorda que as JUNTAS NATURAIS são perigosas e, portanto, desencorajam fortemente seu uso. O perigo vem da inclusão inadvertida de uma nova coluna, com o mesmo nome de outra coluna na outra tabela. Uma junção natural existente pode então "naturalmente" usar a nova coluna para comparações, fazendo comparações/correspondências usando critérios diferentes (de colunas diferentes) do que antes. Assim, uma consulta existente poderia produzir resultados diferentes, mesmo que os dados nas tabelas não tenham sido alterados, mas apenas aumentados. O uso de nomes de coluna para determinar automaticamente os links de tabela não é uma opção em bancos de dados grandes com centenas ou milhares de tabelas em que colocaria uma restrição irreal nas convenções de nomenclatura. Bancos de dados do mundo real são comumente projetados com dados de **chaves estrangeiras** que não são preenchidos de forma consistente (valores NULL são permitidos), devido a regras de negócio e contexto. É prática comum modificar nomes de coluna de dados semelhantes em tabelas diferentes e essa falta de consistência rígida relega as junções naturais a um conceito teórico para discussão.

A consulta de exemplo acima para junções internas pode ser expressa como uma junção natural da seguinte maneira:

    SELECT *
    FROM employee NATURAL JOIN department;

Assim como na cláusula USING explícita, apenas uma coluna `departmentid` ocorre na tabela unida, sem qualificador:

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_30.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_30.png)

PostgreSQL, MySQL e Oracle suportam junções naturais. O Microsoft T-SQL e o IBM DB2 não. As colunas usadas na junção são implícitas, portanto, o código de junção não mostra quais colunas são esperadas, e uma alteração nos nomes das colunas pode alterar os resultados. No padrão [SQL:2011](https://en.wikipedia.org/wiki/SQL:2011), as junções naturais são parte do pacote opcional F401, "Extended joined table".

Em muitos ambientes de banco de dados, os nomes das colunas são controlados por um fornecedor externo, não pelo desenvolvedor de consultas. Uma junção natural pressupõe estabilidade e consistência nos nomes das colunas, que podem ser alteradas durante as atualizações de versão obrigatórias do fornecedor.

## 8.3 Outer Join

Numa junção externa (**outer-join**), a consulta resultante retém todas as linhas, mesmo que não exista outra linha correspondente. As junções externas subdividem em junções externas à esquerda(**left outer join**), junções externas à direita (**right outer join**) e junções externas completas (**full outer join**), dependendo das linhas da tabela que são retidas (esquerda, direita ou ambas). Neste caso, esquerda e direita referem-se aos dois lados da palavra-chave `JOIN`.

Nenhuma notação de junção implícita para junções externas existe no SQL padrão.

### 8.3.1 Left Outer Join

O resultado de uma junção externa esquerda (ou simplesmente junção esquerda) para as tabelas `A` e `B` sempre contém todas as linhas da tabela "esquerda" (`A`), mesmo se a condição de junção não encontrar nenhuma linha correspondente na tabela "direita" (`B`) Isso significa que se a cláusula `ON` corresponder a 0 (zero) linhas em `B` (para uma determinada linha em `A`), a união ainda retornará uma linha no resultado (para essa linha) - mas com NULL em cada coluna de `B`. A junção externa esquerda retorna todos os valores de uma junção interna mais todos os valores na tabela à esquerda que não correspondem à tabela da direita, incluindo linhas com valores `NULL` (vazios) na coluna do link.
Por exemplo, isso nos permite encontrar o departamento de um funcionário, mas ainda mostra funcionários que não foram atribuídos a um departamento (ao contrário do exemplo de junção interna acima, em que funcionários não atribuídos foram excluídos do resultado).
Exemplo de uma junção externa esquerda (a palavra-chave `OUTER` é opcional), com a linha de resultado adicional (comparada com a junção interna):

    SELECT *
    FROM employee LEFT OUTER JOIN department ON employee.departmentid = department.departmentid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_31.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_31.png)

### 8.3.2 Right Outer Join

Uma junção externa direita (ou junção direita) se assemelha a uma junção externa esquerda, exceto com o tratamento das tabelas invertidas. Cada linha da tabela "direita" (`B`) aparecerá na tabela unida pelo menos uma vez. Se nenhuma linha correspondente da tabela "esquerda" (`A`) existir, `NULL` aparecerá nas colunas de `A` para as linhas que não tiverem correspondência em `B`.

Uma junção externa direita retorna todos os valores da tabela da direita e os valores correspondentes da tabela à esquerda (`NULL` no caso de nenhum predicado de junção correspondente). Por exemplo, isso nos permite encontrar cada funcionário e seu departamento, mas ainda mostrar departamentos que não têm funcionários.
Abaixo está um exemplo de uma junção externa direita (a palavra-chave `OUTER` é opcional):

    SELECT *
    FROM employee RIGHT OUTER JOIN department
    ON employee.departmentid = department.departmentid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_32.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_32.png)

Junções externas direita e esquerda são funcionalmente equivalentes. Nenhum dos dois fornece qualquer funcionalidade que o outro não tenha, portanto, junções externas direita e esquerda podem substituir umas às outras, desde que a ordem da tabela seja trocada.

### 8.3.3 Full Outer Join

Conceitualmente, uma junção externa completa combina o efeito de aplicar junções externas esquerda e direita. Onde as linhas nas tabelas `FULL OUTER JOIN` não correspondem, o conjunto de resultados terá valores `NULL` para cada coluna da tabela que não possui uma linha correspondente. Para aquelas linhas que correspondem, uma única linha será produzida no conjunto de resultados (contendo colunas preenchidas de ambas as tabelas).

Por exemplo, isso nos permite ver cada funcionário que está em um departamento e cada departamento que tem um funcionário, mas também ver cada funcionário que não faz parte de um departamento e cada departamento que não possui um funcionário.

Exemplo de uma junção externa completa (a palavra-chave `OUTER` é opcional):

    SELECT *
    FROM employee FULL OUTER JOIN department ON employee.departmentid= department.departmentid;


![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_33.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_33.png)

## 8.4 Self-Join

Uma self-join é unir uma tabela a si mesma.

>Insira duas novas colunas na tabela `employee`. Uma do tipo varchar(13) com o nome `country` e outra com o nome `employeeid` do tipo integer:

    ALTER TABLE employee
    ADD COLUMN country varchar(13),
    ADD COLUMN employeeid integer;

>Apague todos os registros da tabela `employee`

    DELETE FROM employee;

>Insira os seguintes dados na tabela `employee`:

    INSERT INTO employee (country, employeeid, lastname, departmentid) VALUES
    ('Australia', 123, 'Rafferty', 31),
    ('Australia', 124, 'Jones', 33),
    ('Australia', 145, 'Heisenberg', 33),
    ('United States', 201, 'Robinson', 34),
    ('Germany', 305, 'Smith', 34),
    ('Germany', 306, 'Williams', NULL);

Se houvesse duas tabelas separadas para funcionários e uma consulta que solicitasse funcionários na primeira tabela com o mesmo país que os funcionários da segunda tabela, uma operação normal de associação poderia ser usada para localizar a tabela de respostas. No entanto, todas as informações do funcionário estão contidas em uma única tabela grande.

Abaixo está uma consulta de exemplo para essa solução:

    SELECT f.employeeid, f.lastname, s.employeeid, s.lastname, f.country
    FROM employee f INNER JOIN employee s ON f.country = s.country
    WHERE f.employeeid < s.employeeid
    ORDER BY f.employeeid, s.employeeid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_34.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_34.png)

**Para este exemplo:**
* `F` e `S` são apelidos para a primeira e segunda cópias da tabela `employee`;
* A condição `f.country = s.country` exclui pares entre funcionários em diferentes países. A pergunta de exemplo só queria pares de funcionários no mesmo país.
* A condição `f.employeeid < s.employeeid` exclui os pares onde o `employeeid` do primeiro funcionário é maior ou igual ao `employeeid` do segundo empregado. Em outras palavras, o efeito dessa condição é excluir pares duplicados e auto-pares. Sem ela, a seguinte tabela menos útil seria gerada:
***
    SELECT f.employeeid, f.lastname, s.employeeid, s.lastname, f.country
    FROM employee f INNER JOIN employee s ON f.country = s.country
    --WHERE f.employeeid < s.employeeid
    ORDER BY f.employeeid, s.employeeid;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_35.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_35.png)

## 8.5 Junção entre tabelas espaciais e não espaciais

>Abra a janela de consulta SQL no pgAdmin clicando no botão SQL

>"Qual a população do Município de Patos de Minas (MG)?"

O atributo `populacao` da tabela `municipio_populacao_2017` apresenta o dado de população por geocódigo de município e não pelo nome.

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_17.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_17.png)

Já a tabela `lim_municipio_a` apresenta os atributos com o geocódigo (`geocodigo`) e o nome do município (`nome`).

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_16.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_16.png)

Para juntar essas duas tabelas basta fazer uma Junção Interna (INNER JOIN) do tipo Junção Natural (Natural Join) pois os atributos que ligam essas duas tabelas possuem o mesmo nome `geocodigo`.

>Execute a seguinte consulta:

    SELECT *
    FROM lim_municipio_a NATURAL JOIN municipio_populacao_2017;

Porém, observa-se que não foi possível fazer a conexão pois os atributos de conexão, apesar de terem o mesmo nome, possuem tipos distintos: um é do tipo texto (`lim_municipio_a`) e o outro é do tipo numérico (_municipio_populacao_2017_) (JOIN/USING types character varying and integer cannot be matched).

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_18.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_18.png)

Para solucionar esse problema, iremos alterar o tipo de dado do atributo `geocodigo` de inteiro para texto:

>Execute a seguinte consulta:

    ALTER TABLE municipio_populacao_2017
    ALTER COLUMN geocodigo TYPE character varying(15);

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_19.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_19.png)
 
>Execute novamente a seguinte consulta:

    SELECT *
    FROM lim_municipio_a NATURAL JOIN municipio_populacao_2017;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_20.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_20.png)

Observe que que todos os atributos e registros das tabelas são apresentados.

>Agora, Execute novamente a seguinte consulta como uma Junção Interna (INNER JOIN) do tipo Equi-Join filtrando as informação solicitada:

    SELECT nome, populacao
    FROM lim_municipio_a mun, municipio_populacao_2017 pop
    WHERE mun.geocodigo = pop.geocodigo
    AND nome = 'Patos de Minas';

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_21.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_21.png)

Assim, por meio da junção de tabelas foi possível obter uma informação de dados de tabelas distintas.

# 8.6 Manipulação e Definição de Dados

Nos exemplos acima foi mostrado que é possível fazer uma consulta a partir da junção de duas ou mais tabelas.

Agora iremos fixar o resultado dessas consultas a partir da atualização desses dados em uma tabela.

>"Crie uma coluna com os dados da população de cada município na tabela `lim_municipio_a`"

>Execute a seguinte consulta para ALTERAR a tabela `lim_municipio_a` e ADICIONAR uma coluna do tipo número inteiro:

    ALTER TABLE lim_municipio_a
    ADD COLUMN populacao integer;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_22.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_22.png)

>Agora, atualize os dados do atributo `populacao` da tabela `lim_municipio_a` com o atributo `populacao` da tabela municipio_populacao_2017:

    UPDATE lim_municipio_a mue
    SET populacao = pop.populacao
    FROM municipio_populacao_2017 pop
    WHERE mue.geocodigo = pop.geocodigo;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_23.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_23.png)

# 8.6 Exercícios

Usando as tabelas `lim_municipio_a` e `lim_unidade_federacao_a`, responda às seguintes perguntas:

>"Liste os municípios do Brasil com o nome, geocódigo e população"

    SELECT nome, geocodigo, populacao
    FROM lim_municipio_a;

>"Qual a população do município de Paracatu?"

    SELECT populacao
    FROM lim_municipio_a
    WHERE nome = 'Paracatu';

>"Quantos municípios possuem população maior ou igual que 100 mil habitantes?"

    SELECT count(*)
    FROM lim_municipio_a
    WHERE populacao >= 100000;

>Quais municípios possuem população maior ou igual que 101.184 habitantes e menor ou igual que 249.508 habitantes?

    SELECT nome, populacao
    FROM lim_municipio_a
    WHERE populacao >= 101184
    AND populacao <= 249508;

OU

    SELECT nome, populacao
    FROM lim_municipio_a
    WHERE populacao BETWEEN 101184 AND 249508;


>"Qual a menor população de município do Brasil?"

    SELECT min(populacao)
    FROM lim_municipio_a;

>"Qual o município menos populoso do Brasil e a sua respectiva população?"

    SELECT nome, populacao
    FROM lim_municipio_a
    ORDER BY populacao
    LIMIT 1;

>"Relacione os 10 município mais populosos do Brasil" 

    SELECT nome, geocodigo, populacao
    FROM lim_municipio_a
    ORDER BY populacao DESC
    LIMIT 10;

>"Qual a população total por unidade da unidade da federação?"

    SELECT a.nome, sum(a.populacao)
    FROM
        (
        SELECT mue.geocodigo, mue.populacao, ufe.nome, ufe.sigla
        FROM lim_municipio_a mue, lim_unidade_federacao_a ufe
        WHERE substring(mue.geocodigo from 1 for 2) = ufe.geocodigo
        ) as a
    GROUP BY a.nome
    ORDER BY a.nome;

>"Qual o total, média e desvio padrão da população dos municípios do estado de Minas Gerais"

    SELECT a.nome, sum(a.populacao), avg(a.populacao), stddev(a.populacao)
    FROM
        (
        SELECT mue.geocodigo, mue.populacao, ufe.nome, ufe.sigla
        FROM lim_municipio_a mue, lim_unidade_federacao_a ufe
        WHERE substring(mue.geocodigo from 1 for 2) = ufe.geocodigo
        ) as a
    GROUP BY a.nome
    HAVING a.nome = 'Minas Gerais';

# 8.8. Lista de Funções

* [char_length(string)](https://www.postgresql.org/docs/current/static/functions-string.html): função de sequência de caracteres PostgreSQL que retorna o número de caracteres em uma sequência de caracteres.
* [substring(string [from int] [for int])](https://www.postgresql.org/docs/current/static/functions-string.html): função de sequência de caracteres PostgreSQL que retorna o subtexto de caracteres em uma sequência de caracteres a partir da posição indiciada de início e fim do texto a ser extraído.
* [count(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE): função agregada do PostgreSQL que retorna o número de registos válidos que atendem uma determinada situação.
* [sum(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE): função agregada do PostgreSQL que retorna a soma dos valores de entrada.
* [min(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE): função agregada do PostgreSQL que retorna o valor mínimo dos valores de entrada.
* [avg(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE): função agregada do PostgreSQL que retorna o valor médio de uma coluna numérica.
* [stddev(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE): função agregada do PostgreSQL que retorna o desvio padrão dos valores de entrada.
