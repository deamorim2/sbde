# 30. Tuning PostgreSQL para Dados Espaciais

O PostgreSQL é um sistema de banco de dados muito versátil, capaz de funcionar eficientemente em ambientes de recursos muito baixos e ambientes compartilhados com uma variedade de outros aplicativos.

Para garantir que ele seja executado adequadamente em muitos ambientes diferentes, a configuração padrão é muito conservadora e não é muito apropriada para um banco de dados de produção de alto desempenho.

Aqui está uma sugestão de tuning para dados não espaciais para postgreSQL: [Tuning Your PostgreSQL Server](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)

Adicione o fato de que os bancos de dados espaciais têm padrões de uso diferentes, e os dados tendem a consistir em menos registros, muito maiores do que os bancos de dados não espaciais, e você pode ver que a configuração padrão não será totalmente apropriada para nossos propósitos.

Todos esses parâmetros de configuração podem ser editados no arquivo de configuração do banco de dados.

No Windows, isso é `C:\Program Files\PostgreSQL\9.5\data\postgresql.conf`.

Este é um arquivo de texto regular e pode ser editado usando o Bloco de Notas ou qualquer outro editor de texto. As alterações não terão efeito até que o servidor seja reiniciado.

![https://github.com/deamorim2/sbde/blob/master/wiki/30/explorer_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/explorer_01.png)

Uma maneira mais fácil de editar essa configuração é usando o `Backend Configuration Editor` integrado. No pgAdmin, vá em File> Open postgresql.conf .... !

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_01.png)

Ele pedirá a localização do arquivo e navegue até `C:\Program Files\PostgreSQL\9.5\data\postgresql.conf`.

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_02.png)

Esta seção descreve alguns dos parâmetros de configuração que devem ser ajustados para um banco de dados espacial pronto para produção.

Para cada seção, encontre o item apropriado na lista, clique duas vezes na linha para editar a configuração. Altere o Valor para o valor recomendado, conforme descrito, **verifique se o item está Ativado**, clique em OK.

***
**Nota**
Esses valores são apenas recomendações; cada ambiente será diferente e o teste será necessário para determinar a configuração ideal. Mas esta seção deve levá-lo a um bom começo. O computador em questão possui 8GB de memória RAM e as configurações abaixo serão baseadas nesse valor.
***

# 30.1 shared_buffers

Define a quantidade de memória que o servidor de banco de dados usa para buffers de memória compartilhada. Estes são compartilhados entre os processos de back-end, como o nome sugere. Os valores padrão são normalmente insuficientes para bancos de dados de produção.

    Valor padrão: normalmente 32MB
    Valor recomendado: 2GB(25% da memória do banco de dados)

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_03.png)

# 30.2 work_mem

Define a quantidade de memória que as operações internas de classificação e as tabelas de hash podem consumir antes do banco de dados alternar para arquivos em disco. Este valor define a memória disponível para cada operação. Consultas complexas podem ter várias operações de classificação ou hash sendo executadas em paralelo e cada sessão conectada pode estar executando uma consulta.

Dessa forma, você deve considerar quantas conexões e a complexidade das consultas esperadas antes de aumentar esse valor.

O benefício de aumentar esse parâmetro é que o processamento de mais dessas operações (incluindo cláusulas dos tipos ORDER BY e DISTINCT, merge, hash joins e hash baseada em agregações e processamento de subconsultas), pode ser realizado sem incorrer em gravações em disco.

    Valor padrão: 1 MB
    Valor recomendado: 16MB

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_04.png)

# 30.3 maintenance_work_mem

Define a quantidade de memória usada para operações de manutenção, incluindo vacuuming, criação de índice e de chave-estrangeira. Como essas operações não são muito comuns, o valor padrão pode ser aceitável. Este parâmetro pode alternativamente ser aumentado para uma única sessão antes da execução de um número de chamadas CREATE INDEX ou VACUUM como mostrado abaixo.

    SET maintenance_work_mem TO '128MB';
    VACUUM ANALYZE;
    SET maintenance_work_mem TO '16MB';
***
    Valor padrão: 16MB
    Valor recomendado: 128MB

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_05.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_05.png)

# 30.4 wal_buffers

Define a quantidade de memória usada para dados de write-ahead log (WAL). Os registros de write-ahead fornecem um mecanismo de alto desempenho para garantir a integridade dos dados. Durante cada comando de alteração, os efeitos das alterações são gravados primeiro nos arquivos WAL e liberados no disco.

Apenas quando os arquivos WAL forem liberados, as alterações serão gravadas nos próprios arquivos de dados. Isso permite que os arquivos de dados sejam gravados no disco de maneira ideal e assíncrona, garantindo que, em caso de falha, todas as alterações de dados possam ser recuperadas do WAL.

O tamanho desse buffer precisa ser grande o suficiente para conter os dados do WAL para uma única transação típica. Embora o valor padrão geralmente seja suficiente para a maioria dos dados, os dados espaciais tendem a ser muito maiores. Portanto, recomenda-se aumentar o tamanho desse parâmetro.

    Valor padrão: 64kB
    Valor recomendado: 1MB (1/32 shared_buffer)

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_06.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_06.png)

# 30.5 checkpoint_segments

Esse valor define o número máximo de segmentos do arquivo de log (geralmente 16 MB) que podem ser preenchidos entre os pontos de verificação automáticos da WAL. Um ponto de verificação do WAL é um ponto na sequência de transações do WAL no qual é garantido que os arquivos de dados foram atualizados com todas as informações antes do ponto de verificação.

Neste momento, todas as páginas de dados sujas são liberadas para o disco e um registro de ponto de verificação é gravado no arquivo de log. Isso permite que o processo de recuperação de falhas localize o último registro de ponto de verificação e aplique todos os segmentos de log a seguir para concluir a recuperação dos dados.

Como o processo de ponto de verificação exige a liberação de todas as páginas de dados sujos no disco, ele cria uma carga de I/O significativa.

O mesmo argumento acima se aplica para dados espaciais, que são grandes o suficiente para desequilibrar as otimizações não espaciais. Aumentar esse valor impedirá pontos de verificação excessivos, embora isso possa fazer com que o servidor reinicie mais lentamente no caso de uma falha.

    Valor padrão: 3
    Valor recomendado: 6

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_07.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_07.png)

***
**Nota:**
Caso o servidor não inicie, desmarque essa opção.
***

# 30.6 random_page_cost

Esse é um valor sem unidade que representa o custo de um acesso aleatório a uma página do disco.

Esse valor é relativo a vários outros parâmetros de custo, incluindo acesso sequencial à página e custos de operação da CPU. Embora não haja um marcador mágico para esse valor, o padrão é geralmente conservador. Esse valor pode ser definido em uma base por sessão usando o comando:

    SET random_page_cost TO 2.0
***
    Valor padrão: 4.0
    Valor recomendado: 2.0

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_08.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_08.png)

# 30.7 seq_page_cost

Esse é o parâmetro que controla o custo de um acesso de página seqüencial.

Esse valor geralmente não requer ajuste, mas a diferença entre esse valor e random_page_cost afeta muito as escolhas feitas pelo planejador de consulta. Esse valor também pode ser definido em uma base por sessão.

    Valor padrão: 1.0
    Valor recomendado: 1.0

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_09.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_09.png)

# 30.8 effective_cache_size

**effective_cache_size** deve ser configurado para uma estimativa de quanto de memória está disponível para o cache de disco pelo sistema operacional e dentro do próprio banco de dados, depois de levar em consideração o que é usado pelo próprio SO e outros aplicativos.

Esta é uma diretriz para a quantidade de memória que você espera estar disponível nos caches de buffer do sistema operacional e do PostgreSQL, não uma alocação!

Este valor é usado apenas pelo planejador de consultas do PostgreSQL para descobrir se os planos que ele está considerando seriam adequados para a memória RAM ou não. Se estiver definido muito baixo, os índices não poderão ser usados ​​para executar consultas da maneira esperada.

Configurar effective_cache_size para 50% da memória total seria uma configuração conservadora normal, e 75% da memória é uma quantidade mais agressiva, mas ainda assim razoável.

Você pode encontrar uma estimativa melhor observando as estatísticas do seu sistema operacional. Em sistemas semelhantes ao UNIX, adicione os números gratuitos e em cache livre ou superior para obter uma estimativa.

No Windows, consulte o tamanho "Cache do Sistema" na guia Desempenho do Gerenciador de Tarefas do Windows.

![https://github.com/deamorim2/sbde/blob/master/wiki/30/ger_tarefa_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/ger_tarefa_01.png)

Alterar essa configuração não requer a reinicialização do banco de dados(o HUP é suficiente).

    Valor padrão: -
    Valor recomendado: 4.0GB(50-75% Memória RAM)

# 30.9 Recarregar configuração

Depois que essas alterações forem feitas, salve as alterações e recarregue a configuração.

A maneira mais fácil de fazer isso é reiniciar o serviço PostgreSQL.

* No _pgAdmin_, clique com o botão direito do mouse no servidor PostgreSQL 9.5(localhost:5432) e selecione _Disconnect server_

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_10.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_10.png)

* No Windows Services (services.msc), clique com o botão direito do mouse em **postgresql-x64-9.5** e selecione _Reiniciar_.

![https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_11.png](https://github.com/deamorim2/sbde/blob/master/wiki/30/pgadmin_11.png)

* De volta ao pgAdmin, clique no servidor novamente e selecione _Connect_.

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

# 30.10 Analyzing

O planejador de consulta PostgreSQL escolhe inteligentemente quando usar ou não índices para avaliar uma consulta. Contra intuitivamente, nem sempre é mais rápido fazer uma pesquisa de índice: se a pesquisa vai retornar todos os registros na tabela, percorrer a árvore de índice para obter cada registro será realmente mais lento do que ler linearmente a tabela inteira desde o início.

Para descobrir com qual situação ele está lidando (lendo uma pequena parte da tabela versus lendo uma grande parte da tabela), o PostgreSQL mantém estatísticas sobre a distribuição de dados em cada coluna da tabela indexada.

Por padrão, o PostgreSQL reúne estatísticas regularmente. No entanto, se você mudar drasticamente a composição da sua tabela dentro de um curto período de tempo, as estatísticas não serão atualizadas.

Para garantir que suas estatísticas correspondam ao conteúdo da sua tabela, é aconselhável executar o comando ANALYZE depois que os dados em massa forem carregados e excluídos em suas tabelas. Isso força o sistema de estatísticas a coletar dados para todas as suas colunas indexadas.

O comando ANALYZE pede ao PostgreSQL para percorrer a tabela e atualizar suas estatísticas internas usadas para a estimativa do plano de consulta (a análise do plano de consulta será discutida mais adiante).

    ANALYZE tra_trecho_rodoviario_l;
    ANALYZE lim_municipio_a;

# 30.11 Vacuuming

Vale ressaltar que apenas criar um índice não é suficiente para permitir que o PostgreSQL o use de forma eficaz. O VACUUMing deve ser executado sempre que um novo índice é criado ou após um grande número de UPDATEs, INSERTs ou DELETEs serem emitidos em uma tabela.

O comando VACUUM solicita que o PostgreSQL recupere qualquer espaço não utilizado nas páginas da tabela deixadas por atualizações ou exclusões nos registros.

O VACUUMing é tão essencial para a execução eficiente do banco de dados que o PostgreSQL oferece uma opção de “autovacuum”. Ativado por padrão, o autovacuum remove ambos os VACUUM (recupera espaço) e analisa (atualiza as estatísticas) em suas tabelas em intervalos razoáveis determinados pelo nível de atividade. Embora isso seja essencial para bancos de dados altamente transacionais, não é aconselhável aguardar uma execução de autovacuum após adicionar índices ou dados de carregamento em massa. Se uma grande atualização em lote for executada, você deverá executar manualmente o VACUUM.

Conforme necessário, Vacuuming e Analyzing podem ser executados separadamente no banco de dados. A emissão do comando VACUUM não atualizará as estatísticas do banco de dados. Da mesma forma, a emissão de um comando ANALYZE não recuperará linhas de tabela não utilizadas. Ambos os comandos podem ser executados em todo o banco de dados, em uma única tabela ou em uma única coluna.

>Execute as lindas de comando abaixo um de cada vez:

    VACUUM ANALYZE tra_trecho_rodoviario_l;

    VACUUM ANALYZE lim_municipio_a;
