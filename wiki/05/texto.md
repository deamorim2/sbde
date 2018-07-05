# 5. Carregando Dados Espaciais e Não Espaciais

Com suporte a uma ampla variedade de bibliotecas e aplicativos, o PostGIS oferece muitas opções para carregar dados.

# 5.1 Carregando Dados Espaciais

## 5.1.1 PgAdmin > PostGIS Shapefile and DBF Loader

>No **pgAdmin** vá no menu _Plugins_ e selecione _PostGIS Shapefile and DBF Loader 2.2_

O importador do shapefile será iniciado.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_01.png)
>Clique no botão _View connection details..._ e, se necessário, preencha os detalhes da conexão para o banco `sbde` e clique no botão _OK_.

O carregador testará a conexão e voltará novamente para a janela de registro.

Username: **postgres**

Password: **postgres**

Server Host: **localhost** **5432**

Database: **sbde**

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_02.png)

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_03.png)

>Baixe no seu computador o diretório [dados_espaciais](https://drive.google.com/drive/folders/1hOr0W6ZOdj_gZAlwUOfmhOmlT-LBfohP?usp=sharing)

>Clique no botão _Add File_ e navegue até o diretório de dados espaciais que você baixou:  \dados_espaciais

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_04.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_04.png)

>Selecione o arquivo com extensão shapefile (*.shp) `lim_municipio_a.shp` e clique no botão _Open_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_05.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_05.png)

>Clique duas vezes no valor 0 do SRID do arquivo `lim_municipio_a.shp` e altere o valor para **4674** (Sirgas 2000).

>Clique fora dos campos depois de terminar a edição para garantir que os alterações foram efetuadas.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_06.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_06.png)

Repare que o nome do esquema, tabela e coluna já foram preenchidos usando o shapefile, mas você pode alterá-los opcionalmente.

>Clique no botão _Options..._ para rever as opções de carregamento.

O carregador usará o modo rápido "COPIAR" e criará um índice espacial por padrão após o carregamento dos dados.

Repare que a codificação da fonte de dados tabulares, arquivo dbf vinculado ao shapefile, é o **UTF-8**. No brasil os mais usados são **UTF-8** e **Latin1** (ou ISO-8859-1).

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_07.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_07.png)

>Clique no botão _Import_ e acompanhe o processo de importação a partir da janela de log (_log window_). 

Pode levar alguns minutos para carregar.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_08.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_08.png)

Aparecerá uma mensagem dizendo que o arquivo foi carregado com sucesso.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_09.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_09.png)

>Repita o processo de importação para os restantes shapefiles no diretório de dados.

Você pode carregar vários arquivos em uma única importação adicionando vários arquivos antes de pressionar o botão _Importar_. Não se esqueça de colocar o valor **4674** de todas as camadas.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_10.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_10.png)

>Clique na caixa de seleção _Rm_ para remover `lim_municipio_a.shp`, já que já foi importado.

>Clique novamente no botão _Import_ para carregar todas as camadas de uma única vez.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_11.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_11.png)

>Confira no _log window_ que todas as camadas foram importadas com sucesso.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_12.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_12.png)

>No **pgAdmin** clique com o botão direito sobre o banco `sbde` e selecione _Refresh_ para atualizar a exibição da estrutura do banco de dados em árvore.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_13.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_13.png)

Opcionalmente, você poderá clicar no botão _Refresh_ para fazer a mesma coisa.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_refresh.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_refresh.png)

Você verá 14 tabelas na seção _Databases > sbde > Schemas > public > Tables_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_14.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_14.png)

>clique no ícone **+** das tabelas para expandi-las.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_15.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_15.png)

# 5.2 O que são Shapefiles?

Você pode estar perguntando a si mesmo - "O que é essa coisa do shapefile?" Um "shapefile" geralmente se refere a uma coleção de arquivos com ***.shp**, ***.shx**, ***.dbf** e outras extensões em um nome de prefixo comum (por exemplo, `lim_municipio_a.shp`).

O shapefile atual relaciona-se especificamente com arquivos com a extensão ***.shp**. No entanto, o arquivo ***.shp** sozinho está incompleto para distribuição sem os arquivos de suporte necessários.

Arquivos obrigatórios:

* formato ***.shp** - arquivo que contém a geometria/dados espaciais
* formato ***.dbf** - arquivo que compreende a tabela de dados não espaciais no formato dBase III
* formato ***.shx** - arquivo que possui índice de posicionamento da geometria

Os arquivos opcionais incluem:

* formato ***.prj** - o sistema de coordenadas e as informações de projeção, um arquivo de texto simples que descreve a projeção usando um formato de texto bem conhecido (Well-known-text - WKT)

O **pgShapeLoader** converte os arquivos binários que compõem o shapefile para comandos em **SQL** para poder carregá-los para dentro do **PostGIS**.

# 5.3 SRID 4674?

A maior parte do processo de importação é auto-explicativa, mas mesmo os profissionais de SIG experientes podem tropeçar em uma SRID.

Um "SRID" significa "Spatial Reference IDentifier". Ele define todos os parâmetros do sistema de coordenadas geográficas e projeção dos dados. Um SRID é conveniente porque ele contém todas as informações sobre uma projeção de mapa (que pode ser bastante complexa) em um único número.

Você pode ver a definição da projeção utilizada nesse tutorial a partir de um banco de dados on-line:

[http://spatialreference.org/ref/epsg/4674/](http://spatialreference.org/ref/epsg/4674/)

ou diretamente dentro do PostGIS com uma consulta para a tabela spatial_ref_sys:

    SELECT srtext
    FROM spatial_ref_sys
    WHERE srid = 4674;

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_16.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_16.png)

***
**Nota:**

A tabela spatial_ref_sys PostGIS é uma tabela OGC padrão que define todos os sistemas de referência espacial     conhecidos pelo banco de dados. Os dados fornecidos pelo PostGIS, lista mais de 3000 sistemas de referência espacial     conhecidos e detalhes necessários para transformar/re-projetar dados entre si.
***


Em ambos os casos, você vê uma representação textual do sistema de referência espacial 4674 (impresso abaixo para maior clareza):

    GEOGCS["SIRGAS 2000",
        DATUM["Sistema_de_Referencia_Geocentrico_para_las_AmericaS_2000",
            SPHEROID["GRS 1980",6378137,298.257222101,
                AUTHORITY["EPSG","7019"]],
            TOWGS84[0,0,0,0,0,0,0],
            AUTHORITY["EPSG","6674"]],
        PRIMEM["Greenwich",0,
            AUTHORITY["EPSG","8901"]],
        UNIT["degree",0.0174532925199433,
            AUTHORITY["EPSG","9122"]],
        AUTHORITY["EPSG","4674"]]

Se você abrir o arquivo `lim_municipio_a.prj` do diretório de dados, você verá a mesma definição de projeção. Um problema comum para as pessoas que começaram com o **PostGIS** é descobrir o número **SRID** a ser usado para seus dados. Tudo o que eles têm é um arquivo ***.prj**.

Mas como o usuário sabe por meio de um arquivo ***.prj** qual o número **SRID** correto? A resposta fácil é usar um computador.

>Selecione o arquivo ***.prj** em [http://prj2epsg.org](http://prj2epsg.org).

Isso lhe dará o número (ou uma lista de números) que mais se encaixa com sua definição de projeção.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_17.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_17.png)

Não há números para cada projeção de mapa no mundo, mas os mais comuns estão contidos no banco de dados **prj2epsg** de números padrão.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_18.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_18.png)

# 5.4 Visualizar os dados espaciais no QGIS

**QGIS**, é um visualizador/editor desktop de SIG para visualizar rapidamente os dados. Você pode visualizar vários formatos de dados, incluindo arquivos de forma plana e um banco de dados **PostGIS**.

Sua interface gráfica permite uma fácil exploração de seus dados, bem como testes simples e estilo rápido. Tente usar este software para conectar seu banco de dados **PostGIS**.

>Encontre o software **QGIS** e inicie-o

>Vá em _QGIS > Camada > Adicionar camada_ e clique em _PostGIS_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_19.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_19.png)

>Clique no botão _Novo_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_20.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_20.png)

>Preencha o formulário com os dados de conexão

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_21.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_21.png)

>Clique no botão _Testar conexão_ e verifique se a conexão foi estabelecida

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_22.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_22.png)

>Por fim, clique no botão _OK_

>Clique no botão _Conectar_ e expanda a árvore de objetos clicando no ícone **+**

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_23.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_23.png)

Verifique que o QGIS identifica automaticamente as tabelas geométricas, o atributo(coluna ou campo) que contém as informação geométrica, bem como o tipo de dado, o tipo espacial e o **SRID**

>Selecione as camadas `lim_unidade_federacao` e `loc_capital_a` e clique no botão _Adicionar_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_24.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_24.png)

Observe que duas camadas, uma do tipo espacial ponto e outro do tipo espacial polígono, foram inseridas na visualização do **QGIS**

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_25.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_25.png)

>Ative a camada `lim_unidade_federacao`, vá em _Camada > Abrir tabela de atributos_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_26.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_26.png)

Observe que a tabela possui 27 registros e 8 colunas. No `pgadmin` observam-se 9 colunas nessa tabela. A coluna omitida no QGIS é justamente a coluna com os dados que descrevem a geometria espacial.

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_27.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_27.png)

# 5.5 Carregando Dados Não Espaciais no QGIS

>Baixe no seu computador o arquivo de texto separado por ponto e vírgula (csv) [municipio_populacao_2017.csv](https://drive.google.com/file/d/1hhbAO5F5y6OQwV_IVnOpLIw_EEItmmbv/view?usp=sharing) no diretório _C:/wiki/dados_nao_espaciais/_

>Encontre o software **QGIS** e inicie-o

>Vá em _QGIS > Adicionar Camada >_ e clique em _A partir de um texto delimitado..._

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_28.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_28.png)

>Selecione o arquivo _C:/wiki/dados_nao_espaciais/municipio_populacao_2017.csv_, preencha as informações como consta o formulário e clique no botão _OK_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_29.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_29.png)

>Clique com o botão direito sobre a camada _município_população_2017.csv_ e clique em _Abrir tabela de atributos_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_30.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_30.png)

Observe que os dados foram importados corretamente

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_31.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_31.png)

>Vá em _QGIS > Banco de dados > Gerenciador BD_ e clique em _Gerenciador BD_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_32.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_32.png)

>Expanda a árvore +PostGIS > +sbde > +public para selecionar o esquema onde a camada será importada

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_33.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_33.png)

>Vá em _Gerenciador BD > Tabela_ e clique em _Importar camada/arquivo_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_34.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_34.png)

>Selecione o dado de entrada _município_população_2017_, clique no botão _Opções de atualização_ e depois finalize a importação clicando no botão _OK_

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_35.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_35.png)

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_36.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_36.png)

Observe que agora aparece uma camada de dados não espaciais com o nome `município_população_2017` no esquema `public`

>Clique duas vezes nessa camada para adicioná-la ao visualizador de camadas

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_37.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_37.png)

![https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_38.png](https://github.com/deamorim2/sbde/blob/master/wiki/05/pgshapeloader_38.png)

Todos os dados foram importados com sucesso!
