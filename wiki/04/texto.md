# 4.1. PgAdmin

O PostgreSQL possui uma série de front-ends administrativos. O principal é o **psql** uma ferramenta de linha de comando para inserir consultas SQL.

Outro popular front-end do PostgreSQL é a ferramenta gráfica gratuita e de código aberto **pgAdmin**. Todas as consultas feitas no **pgAdmin** também podem ser feitas na linha de comando com **psql**.

>Encontre o software **pgAdmin** e inicie-o

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_01.png)

Se esta for a primeira vez que você executou pgAdmin, você deve ter uma entrada de servidor para PostGIS (localhost: 5432) já configurada em pgAdmin.

>Clique duas vezes na entrada e insira o que quiser no prompt da senha para se conectar ao banco de dados

O banco de dados PostGIS foi instalado com acesso irrestrito para usuários locais (usuários que se conectam da mesma máquina que o banco de dados está sendo executado). Isso significa que aceitará qualquer senha que você fornecer. Se você precisa se conectar a partir de um computador remoto, a senha para o usuário postgres foi configurada para: **postgres**.

# 4.2. Criando um banco de dados

>Abra o item da árvore de dados e veja os bancos de dados disponíveis

O banco de dados do postgres é o banco de dados do usuário para o usuário padrão do postgres e não é muito interessante para nós.

>Clique com o botão direito do mouse no item _Database_ e selecione _New Database..._

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_03.png)

>Preencha o formulário _New Database..._ como mostrado abaixo e clique em _OK_

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_02.png)

>Selecione o novo banco de dados do `sbde` e abra-o para exibir a árvore dos objetos. Você verá o esquema `public`

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_04.png)

>Clique no botão de consulta SQL indicado abaixo (ou vá em _Tools > Query Tools_)

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_05.png)

>Digite a seguinte consulta no campo de texto da consulta para carregar a extensão espacial PostGIS:

    CREATE EXTENSION postgis;

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_07.png)

>Clique no botão _Reproduzir_ (Play) na barra de ferramentas (ou pressione a tecla _F5_) indicado abaixo para Executar a consulta (Execute Query)

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_06.png)

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_08.png)

Agora, confirme se o PostGIS está instalado executando uma função PostGIS:

    SELECT postgis_full_version();

![](https://github.com/deamorim2/sbde/blob/master/wiki/04/pgadmin_09.png)

Você criou com sucesso um banco de dados espacial PostGIS!!!

## Lista de Funções

[postgis_full_version()](https://postgis.net/docs/PostGIS_Full_Version.html): Relata a versão completa do postgis e mostra as informações de configuração.