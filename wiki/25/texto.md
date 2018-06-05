# 25. Clustering em Índices

Os bancos de dados só podem recuperar informações tão rapidamente quanto podem retirá-las do disco.

Bancos de dados pequenos irão flutuar totalmente para o cache de RAM e ficar longe das limitações do disco físico, mas para grandes bancos de dados, o acesso ao disco físico será uma limitação na velocidade de acesso ao disco.

Os dados são gravados no disco oportunisticamente, portanto, não há necessariamente nenhuma correlação entre os dados do pedido armazenados no disco e a maneira como eles serão acessados ou organizados pelos aplicativos.

![https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering1.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering1.jpg)

Uma maneira de acelerar o acesso aos dados é garantir que os registros, que provavelmente serão recuperados juntos no mesmo conjunto de resultados, estejam localizados em locais físicos semelhantes nos discos do disco rígido. Isso é chamado de "clustering".

O esquema de agrupamento correto a ser usado pode ser complicado, mas uma regra geral se aplica: os índices definem um esquema de ordenação natural para dados que é semelhante ao padrão de acesso que será usado na recuperação dos dados.

![https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering2.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering2.jpg)

Por causa disso, ordenar os dados no disco na mesma ordem que o índice pode fornecer uma vantagem de velocidade em alguns casos.

# 25.1. Clustering utilizando a R-Tree

Dados espaciais tendem a ser acessados em janelas correlacionadas espacialmente: pense na janela do mapa em um aplicativo da Web ou de desktop.

Todos os dados nas janelas têm um valor de localização semelhante (ou não estaria na janela!). Portanto, o armazenamento em cluster baseado em um índice espacial faz sentido para dados espaciais que serão acessados com consultas espaciais, pois coisas semelhantes tendem a ter localizações semelhantes.

Vamos agrupar nossos `lim_municipio_a` com base em seu índice espacial:

    CLUSTER lim_municipio_a USING lim_municipio_a_geom_idx;

O comando reescreve os municípios na ordem definida pelo índice espacial lim_municipio_a_geom_idx.

Você consegue perceber uma diferença de velocidade? Provavelmente não, porque a tabela é bem pequena e cabe facilmente na memória, portanto a sobrecarga de acesso ao disco não afeta o desempenho.

Uma das surpresas da R-Tree é que uma R-Tree construída incrementalmente em dados espaciais pode não ter alta coerência espacial das folhas. Por exemplo, veja esta visualização das folhas do índice espacial de um índice em estradas na província de British Columbia:

![https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering3.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering3.jpg)

Preferiríamos agrupar usando uma árvore mais espacialmente compacta, como essa R-Tree balanceada.

![https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering4.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/25/clustering4.jpg)

Não temos um algoritmo R-Tree balanceado disponível no PostGIS, mas temos um proxy útil que coloca os dados espaciais em uma ordem espacialmente autocorrelacionada, a função ST_GeoHash().

# 25.2. Clustering utilizando GeoHash

Para clusterizar na função ST_GeoHash(), você primeiro precisa ter um índice geohash em seus dados. Felizmente, eles são fáceis de construir.

O algoritmo **geohash** só funciona em dados em coordenadas geográficas (longitude/ latitude).

    CREATE INDEX lim_municipio_a_geohash ON lim_municipio_a (ST_GeoHash(geom));

Depois que você tiver um índice geohash, o armazenamento em cluster usará a mesma sintaxe que o clustering da R-Tree.

    CLUSTER lim_municipio_a USING lim_municipio_a_geohash;

Agora seus dados estão bem organizados em ordem espacialmente correlacionada!

***
**Nota:**
Se os dados espaciais estão projetados e não estão em lat/long, então precisamos transformar as geometrias ao mesmo tempo que as alteramos.
***

# 25.3. Lista de funções

ST_GeoHash(geometry A): Retorna uma string de texto representando o GeoHash dos limites do objeto.

