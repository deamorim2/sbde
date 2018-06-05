# 7. Consulta Simples SQL

# 7.1 SQL

SQL, ou "Structured Query Language", é um meio de fazer perguntas e atualizar dados em bancos de dados relacionais.

Se quiser saber um pouco mais sobre esse assunto [Clique Aqui!](https://pt.wikipedia.org/wiki/SQL)

## 7.1.1 Subconjuntos do SQL

A linguagem SQL é dividida em subconjuntos de acordo com as operações que queremos efetuar sobre um banco de dados, tais como:

### 7.1.1.1 DML - Linguagem de Manipulação de Dados

O primeiro grupo é a DML (Data Manipulation Language - Linguagem de manipulação de dados).

DML é um subconjunto da linguagem SQL que é utilizado para realizar inclusões, alterações e exclusões de dados presentes em registros. Estas tarefas podem ser executadas em vários registros de diversas tabelas ao mesmo tempo. Os comandos que realizam respectivamente as funções acima referidas são:

* Inclusões de dado (INSERT);
* Atualizações de dado (UPDATE);
* Exclusões de dado (DELETE)

### 7.1.1.2 DDL - Linguagem de Definição de Dados

O segundo grupo é a DDL (Data Definition Language - Linguagem de Definição de Dados).

Uma DDL permite ao utilizador definir tabelas novas e elementos associados. Os principais comandos utilizados são: 

* Criação de Objeto (CREATE);
* Alteração de Objeto (ALTER);
* Adicionar Objeto (ADD);
* Exclusão de Objeto (DROP);
* Renomear Objeto (RENAME);
* TRUNCATE

Tipos de Objeto: Banco de Dados, Tabela, Coluna, Restrição, Usuário, Sequência, Função, Vista, Gatilho, Índice, etc 

### 7.1.1.3 DCL - Linguagem de Controle de Dados

O terceiro grupo é o DCL (Data Control Language - Linguagem de Controle de Dados).

DCL controla os aspectos de autorização de dados e licenças de usuários para controlar quem tem acesso para ver ou manipular dados dentro do banco de dados. Os principais comandos utilizados são:

* Autorização de usuário (GRANT);
* Revogação de usuário (REVOKE).

### 7.1.1.4 DTL - Linguagem de Transação de Dados

* BEGIN WORK (ou START TRANSACTION, dependendo do dialeto SQL) pode ser usado para marcar o começo de uma transação de banco de dados que pode ser completada ou não.
* COMMIT finaliza uma transação dentro de um sistema de gerenciamento de banco de dados.
* ROLLBACK faz com que as mudanças nos dados existentes desde o último COMMIT ou ROLLBACK sejam descartadas.

### 7.1.1.5 DQL - Linguagem de Consulta de Dados

Embora tenha apenas um comando, a DQL é a parte da SQL mais utilizada.

O comando SELECT permite ao usuário especificar uma consulta ("query") como uma descrição do resultado desejado. Esse comando é composto de várias cláusulas e opções, possibilitando elaborar consultas das mais simples às mais elaboradas.

>Em seguida, insira a seguinte consulta na janela de consulta

    SELECT nome
    FROM lim_municipio_a;

>Clique no botão _Executar consulta_ (o triângulo verde)

![https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_06.png](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_06.png)
 
A consulta será executada por alguns segundos (mili) e retornará os 5570 municípios como resultado.

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_10.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_10.png)
 
Trabalharemos quase exclusivamente com SELECT para fazer perguntas de tabelas usando funções espaciais.

Uma consulta de seleção geralmente é da forma:

    SELECT (SELECIONE) algumas_colunas
    FROM (DE) alguma_fonte_de_dados
    WHERE (ONDE) alguma_condição ;

* `algumas_colunas` são nomes de colunas ou funções de valores de coluna.

* `alguma_fonte_de_dados` é uma única tabela ou uma tabela composta criada ao juntar duas tabelas em uma chave ou condição.

* `alguma_condição` é um filtro que restringe o número de linhas a serem retornadas.

***

**Nota:**

Para obter uma sinopse de todos os parâmetros SELECT, consulte a [documentação do PostgreSQL](http://www.postgresql.org/docs/current/interactive/sql-select.html).

***

A linguagem SQL pode ser dividida em:

* Clausulas;
* Operadores Lógicos;
* Operadores Relacionais;
* Funções de Agregação;

#### 7.1.1.5.1 Clausulas

* `FROM` – Utilizada para especificar a tabela que se vai selecionar os registros.
* `WHERE` – Utilizada para especificar as condições que devem reunir os registros que serão selecionados.
* `GROUP BY` – Utilizada para separar os registros selecionados em grupos específicos.
* `HAVING` – Utilizada para expressar a condição que deve satisfazer cada grupo.
* `ORDER BY` – Utilizada para ordenar os registros selecionados com uma ordem especifica.
* `DISTINCT` – Utilizada para selecionar dados sem repetição.
* `UNION` - combina os resultados de duas consultas SQL em uma única tabela para todas as linhas correspondentes.

#### 7.1.1.5.2 Operadores Lógicos

* `AND` – E lógico. Avalia as condições e devolve um valor verdadeiro caso ambos sejam corretos.
* `OR` – OU lógico. Avalia as condições e devolve um valor verdadeiro se algum for correto.
* `NOT` – Negação lógica. Devolve o valor contrário da expressão.

#### 7.1.1.5.3 Operadores Relacionais

* `<` 	Menor
* `>` 	Maior
* `<=` 	Menor ou igual
* `>=` 	Maior ou igual
* `=` 	Igual
* `<>` 	Diferente
* `BETWEEN` – Intervalo de valores.
* `LIKE` – Comparação de dados
* `IN` – Se conjunto de dados está numa lista
* `IS` ou `IS NOT` – comparação de dados com dados nulos;
* `AS` – Alias (apelido)

#### 7.1.1.5.4 Funções de Agregação

* `AVG` – Utilizada para calcular a média dos valores de um campo determinado.
* `COUNT` – Utilizada para devolver o número de registros da seleção.
* `SUM` – Utilizada para devolver a soma de todos os valores de um campo determinado.
* `MAX` – Utilizada para devolver o valor mais alto de um campo especificado.
* `MIN` – Utilizada para devolver o valor mais baixo de um campo especificado.

# 7.2 Consultas de Seleção de Dados (SELECT)

Você já viu o SQL quando criamos nosso primeiro banco de dados. Lembre-se do que já fizemos anteriormente:

    SELECT postgis_full_version();

Mas essa era uma pergunta sobre o banco de dados. Agora que carregamos dados em nosso banco de dados, vamos usar o SQL para fazer perguntas sobre os dados!

Por exemplo:

>"Quais são os nomes de todos os municípios do Brasil?"

>Abra a janela de consulta SQL no pgAdmin clicando no botão SQL

![https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_05.png](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_05.png)
 
## 7.2.1 Geocódigos do IBGE para limites políticos

Atributo `geocodigo` da tabela `lim_unidade_federacao_a` - Código da Unidade da Federação. Indica o código numérico da Unidade da Federação. Possui 2 dígitos da seguinte forma: UF, onde: UF – Unidade da Federação, MMMMM – Município.

Atributo `geocodigo` da tabela `lim_municipio_a` - Código do município. Indica o código numérico completo do município. Possui 7 dígitos divididos da seguinte forma: UFMMMMM, onde: UF – Unidade da Federação, MMMMM – Município.

>"Quais são os geocódigos para cada uma das unidades da Federação?"

>Execute a seguinte consulta:

    SELECT geocodigo, nome
    FROM lim_unidade_federacao_a;

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_15.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_15.png)

Verifique que o geocódigo do estado de Minas Gerais corresponde ao número 31.

>"Quais são os nomes de todos os municípios do estado de Minas Gerais?"

Retornamos à nossa tabela `lim_municipio_a` com um filtro na mão. A tabela contém todos os municípios do estado de Minas Gerais, que possuem geocódigo que iniciam com o valor 31.

>Execute a consulta baixo:

    SELECT nome
    FROM lim_municipio_a
    WHERE geocodigo like '31%';

A consulta será executada por menos (mili) segundos e retornará os 853 resultados.

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_11.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_11.png)

Às vezes, precisamos aplicar uma função aos resultados da nossa consulta.

Por exemplo:

>"Qual é o número de letras nos nomes de todos os municípios de Minas Gerais?"

Felizmente, o PostgreSQL possui uma função de comprimento de caractere, **char_length(string)**.

    SELECT nome, char_length (nome)
    FROM lim_municipio_a
    WHERE geocodigo like '31%';

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_12.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_12.png)

Muitas vezes, estamos menos interessados ​​nas linhas individuais do que em uma estatística que se aplica a todas elas. Portanto, conhecer os comprimentos dos nomes dos bairros pode ser menos interessante do que saber o comprimento médio dos nomes. As funções que ocupam várias linhas e retornam um único resultado são chamadas de funções "agregadas".

O PostgreSQL possui uma série de funções agregadas integradas, incluindo o propósito geral **avg()** para valores médios e **stddev()** para desvios padrão.

>"Qual é o número médio de letras e o desvio padrão do número de letras nos nomes de todos os municípios de Minas Gerais?"

    SELECT avg(char_length(nome)), stddev(char_length(nome))
    FROM lim_municipio_a
    WHERE geocodigo like '31%';

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_13.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_13.png)

As funções agregadas em nosso último exemplo foram aplicadas a cada linha do conjunto de resultados. E se queremos que os resumos sejam realizados em grupos menores dentro do conjunto de resultados globais? Para isso, adicionamos uma cláusula GROUP BY (AGRUPADO POR). As funções agregadas muitas vezes precisam de uma instrução GROUP BY adicionada para agrupar o conjunto de resultados por uma ou mais colunas.

>"Qual é o número médio de letras nos nomes de todos os municípios, por unidade da federação?"

    SELECT substring(geocodigo from 1 for 2), avg(char_length(nome)), stddev(char_length(nome))
    FROM lim_municipio_a
    GROUP BY substring(geocodigo from 1 for 2);

![https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_14.png](https://github.com/deamorim2/sbde/blob/master/wiki/07/pgadmin_14.png)

Nós incluímos a coluna do geocódigo da unidade da federação no resultado do resultado para que possamos determinar qual estatística se aplica a qual unidade da federação. Em uma consulta agregada, você só pode enviar colunas que sejam:

* membros da cláusula de agrupamento ou;
* funções agregadas.

# 7.3. Lista de Funções

* [avg(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE): função agregada do PostgreSQL que retorna o valor médio de uma coluna numérica.
* [char_length(string)](https://www.postgresql.org/docs/current/static/functions-string.html): função de sequência de caracteres PostgreSQL que retorna o número de caracteres em uma sequência de caracteres.
* [substring(string [from int] [for int])](https://www.postgresql.org/docs/current/static/functions-string.html): função de sequência de caracteres PostgreSQL que retorna o subtexto de caracteres em uma sequência de caracteres a partir da posição indiciada de início e fim do texto a ser extraído.
* [stddev(expression)](https://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE): função agregada do PostgreSQL que retorna o desvio padrão dos valores de entrada.
