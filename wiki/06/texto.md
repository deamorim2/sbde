# 6.1 Base Cartográfica Contínua do Brasil, escala 1:250.000

Esses dados fazem parte da Base Cartográfica Contínua do Brasil, escala 1:250.000 (BC250), produzida pelo Instituto Brasileiro de Geografia e Estatística (IBGE).

A BC250 é um conjunto de dados geoespaciais de referência, estruturados em bases de dados digitais, permitindo uma visão integrada do território nacional nesta escala, sendo de grande importância para projetos de planejamento regional, de
cunho ambiental e de gestão do território.

Os dados utilizados nesse tutorial é composto por partes da **BC250**, compreende arquivos no formato **shapefile** e compõem 6 camadas com geometria do tipo ponto, 2 camadas do tipo linha e 5 camadas do tipo polígono.

O _Spatial Reference IDentifier_ (**SRID**) dessas camadas é o de número **4674** e compreende o **SIRGAS 2000**.

A codificação da fonte de dados tabulares é o **UTF-8**.

Essa base compreende a versão 2017 e pode ser obtida a partir do link: [ftp://geoftp.ibge.gov.br/cartas_e_mapas/bases_cartograficas_continuas/bc250/versao2017/](ftp://geoftp.ibge.gov.br/cartas_e_mapas/bases_cartograficas_continuas/bc250/versao2017/)

# 6.2 Infraestrutura Nacional de Dados Espaciais(INDE)

A BC250 oferece uma representação cartográfica do território brasileiro, desta forma, o IBGE contribui para a implementação da componente de dados da Infraestrutura Nacional de Dados Espaciais - INDE (Dec. nº 6666 de 27 de Novembro de 2008), Plano de Ação desenvolvido pelo Comitê Especializado da INDE – CINDE da Comissão Nacional de Cartografia – [CONCAR](www.concar.ibge.gov.br).

A modelagem conceitual da Especificação Técnica para a Estruturação de Dados Geoespaciais Vetoriais (ET-EDGV) foi elaborada seguindo metodologia orientada a objetos. Tem como premissas: a classificação da informação conforme o seu uso e a abrangência para dados vetoriais nas escalas 1:25.000 e menores, do mapeamento sistemático terrestre básico.

A modelagem da BC250 em sua versão 2017 está estruturada conforme a Especificação Técnica para Estruturação de Dados Geoespaciais Vetoriais ([ET-EDGV versão 2.13](https://github.com/deamorim2/sbde/blob/master/wiki/documentos/ET_EDGV_Vs_2_1_3.pdf)) de outubro 2010.

Cabe ressaltar que, o modelo da EDGV está em processo de atualização. Está previsto uma mudança em 2018 para um nova [ET-EDGV versão 3.0](https://github.com/deamorim2/sbde/blob/master/wiki/documentos/ET-EDGV_versao_3.0_2017_12_12.pdf) e  [Apêndice](https://github.com/deamorim2/sbde/blob/master/wiki/documentos/APENDICES_ET_EDGV_3.0_2017_12_12.pdf). Esta mudança acarretará na redução de algumas classes dentro do mapeamento, além disso a Coordenação de Cartografia do IBGE também redefiniu algumas classes dentro do mapeamento topográfico na escala 1:250.000, o que resultou
na retirada de algumas classes na publicação de 2017, que não se mostraram relevantes para a escala.

# 6.2 Dados Espaciais

## 6.2.1 - SISTEMA DE TRANSPORTE

Categoria que agrupa o conjunto de sistemas destinados ao transporte e deslocamento de carga e passageiros, bem como as estruturas de suporte ligadas a estas atividades. Correspondente a seção 4 (quatro) da ET-EDGV.

**Via Rodoviária**: conjunto de elementos agregando trechos rodoviários.
As vias rodoviárias serão adquiridas ou atualizadas pela geometria do tipo linha e o critério de seleção será baseado no tipo da via, tais como:

**Acesso**: quando o acesso a localidades ou obras civis que não sofreram processos de eliminação e que estão a uma distância superior a 1250m da rodovia principal.

**Rodovia**: só será adquirida quando sua dimensão for superior a 1.250m, exceto quando proporcionar acesso a localidades ou obras civis que não sofreram processos de eliminação e que estão a uma distância superior a 1250m da rodovia principal.

**Caminho carroçável**: só será adquirido quando sua dimensão for superior a 2.500m; exceto quando proporcionar acesso a localidades ou obras civis que não sofreram processos de eliminação e que estão a uma distância superior a 1250m da rodovia principal.

**Autoestrada**: todas serão adquiridas.

**Observação: Dentro de área edificada, visando dar continuidade ao percurso das vias, só serão adquiridas as rodovias de eixo principal.**

**Pista de Pouso**: pista ou plataforma destinada a pouso e decolagem ou taxiamento de aeronaves.

A pista de pouso será adquirida ou atualizada com geometria do tipo linha quando seu comprimento for superior ou igual a 1.000m. Abaixo de 1000m de extensão sua representação será do tipo ponto.

## 6.2.2 – LOCALIDADE

Categoria correspondente à seção 9 (nove) da ET-EDGV, e que representa os diversos tipos de concentração de habitações humanas.

**Cidade**: localidade com o mesmo nome do Município a que pertence (sede municipal) e onde está sediada a respectiva Prefeitura, excluídos os municípios das Capitais. É constituída pela área urbana do distrito sede e delimitada pelo perímetro urbano estabelecido por lei municipal.
Serão adquiridas ou atualizadas todas as cidades brasileiras, representadas com geometria do tipo ponto.

**Vila**: localidade com o mesmo nome do Distrito a que pertence (sede distrital) e onde está sediada a autoridade distrital, excluídos os distritos das sedes municipais. É delimitada pelo perímetro urbano definido, por lei municipal, como a área urbana do distrito que não a sede do município.
Serão adquiridas ou atualizadas todas as vilas brasileiras, representadas com geometria do tipo ponto.

**Aglomerado Rural Isolado**: localidade que tem as características de aglomerado rural e está localizada a uma distância igual ou superior a um quilômetro (1 km) da área urbana de uma cidade ou vila ou de um aglomerado rural já definido como de extensão urbana.
Serão adquiridos ou atualizados todos os aglomerados rurais isolados, representados com geometria do tipo ponto.

**Área Edificada**: correspondente à área densamente edificada, cuja proximidade das estruturas não permite a sua  representação individualizada e, sim, o contorno da área do conjunto.
Só serão adquiridas ou atualizadas aquelas que possuírem área igual ou superior a 140.000m2, com geometria do tipo  polígono.
Toda área edificada deverá estar associada a uma localidade.

## 6.2.3 – LIMITES

Categoria que representa os distintos níveis político-administrativos e as áreas especiais; áreas de planejamento  operacional, áreas particulares (não classificadas nas demais categorias), bem como os elementos que delimitam materialmente estas linhas no terreno, correspondente a seção 11 (onze) da ET-EDGV.

**País**: polígono referente ao espaço geográfico abrangido por um Estado soberano. Serão adquiridos ou atualizados todos os países de que se encontrarem dentro da área do mapeamento com geometria do tipo polígono.

**Unidade da Federação**: polígono referente à unidade de maior hierarquia dentro da organização político-administrativa no Brasil, criada através de leis emanadas no Congresso Nacional e sancionadas pelo Presidente da República.
Serão adquiridas ou atualizadas todas as Unidades da Federação com geometria do tipo polígono.

**Município**: polígono referente à unidade político-administrativa, criada através de leis ordinárias das Assembleias Legislativas de cada Unidade da Federação e sancionada pelo Governador.
Serão adquiridos ou atualizados todos os municípios com geometria do tipo polígono.

**Terra Indígena**: terra tradicionalmente ocupada por indígenas ou silvícolas, por eles habitada, em caráter permanente, utilizada para suas atividades produtivas, sendo imprescindível à preservação dos recursos ambientais, necessários ao seu bem-estar e necessária a sua reprodução física e cultural, segundo seus usos, costumes e tradições, conforme parágrafo 1º do artigo 231 da Constituição Federal de 1988.

# 6.3 Dados Não Espaciais

## 6.3.1 - População (IBGE)

A tabela `municipio_populacao_2017.csv` apresenta [estimativas populacionais anuais de população](https://www.ibge.gov.br/estatisticas-novoportal/sociais/populacao/9103-estimativas-de-populacao.html?=&t=downloads) do IBGE para os municípios e para as Unidades da Federação brasileiros, com data de referência em 1º de julho de 2017 e publicado no Diário Ofical da União (DOU).





