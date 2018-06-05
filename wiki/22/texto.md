# 22. Igualdade Geométrica

# 22.1. Igualdade Geométrica

Determinar a igualdade geométrica ao lidar com geometrias pode ser complicado

O PostGIS suporta três funções diferentes que podem ser usadas para determinar diferentes níveis de igualdade, embora, para maior clareza, usaremos as definições abaixo.

Para ilustrar essas funções, usaremos os seguintes polígonos.

![https://github.com/deamorim2/sbde/blob/master/wiki/22/polygon-table.png](https://github.com/deamorim2/sbde/blob/master/wiki/22/polygon-table.png)

Esses polígonos são carregados usando os seguintes comandos.

    CREATE TABLE polygons (id integer, name varchar, poly geometry);

    INSERT INTO polygons VALUES
    (1, 'Polygon 1', 'POLYGON((-1 1.732,1 1.732,2 0,1 -1.732,-1 -1.732,-2 0,-1 1.732))'),
    (2, 'Polygon 2', 'POLYGON((-1 1.732,-2 0,-1 -1.732,1 -1.732,2 0,1 1.732,-1 1.732))'),
    (3, 'Polygon 3', 'POLYGON((1 -1.732,2 0,1 1.732,-1 1.732,-2 0,-1 -1.732,1 -1.732))'),
    (4, 'Polygon 4', 'POLYGON((-1 1.732,0 1.732, 1 1.732,1.5 0.866,2 0,1.5 -0.866,1 -1.732,0 -1.732,-1 -1.732,-1.5 -0.866,-2 0,-1.5 0.866,-1 1.732))'),
    (5, 'Polygon 5', 'POLYGON((-2 -1.732,2 -1.732,2 1.732,-2 1.732,-2 -1.732))');

## 22.1.1. Exatamente igual

A igualdade exata é determinada pela comparação de duas geometrias, vértice por vértice, em ordem, para garantir que elas sejam idênticas em posição. Os exemplos a seguir mostram como esse método pode ser limitado em sua eficácia.

    SELECT a.name, b.name, CASE WHEN ST_OrderingEquals(a.poly, b.poly)
    THEN 'Exactly Equal' ELSE 'Not Exactly Equal' end
    FROM polygons as a, polygons as b;

![https://github.com/deamorim2/sbde/blob/master/wiki/22/pgadmin_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/22/pgadmin_01.png)

Neste exemplo, os polígonos são apenas iguais a eles próprios, não a outros polígonos aparentemente equivalentes (como no caso dos polígonos 1 a 3).

No caso dos Polígonos 1, 2 e 3, os vértices estão em posições idênticas, mas são definidos em ordens diferentes.

O polígono 4 tem vértices colineares (e, portanto, redundantes) nas bordas do hexágono, causando a desigualdade com o polígono 1.

## 22.1.2. Espacialmente igual

Como vimos acima, a igualdade exata não leva em conta a natureza espacial das geometrias. Existe uma função, apropriadamente denominada ST_Equals, disponível para testar a igualdade espacial ou equivalência de geometrias.

    SELECT a.name, b.name, CASE WHEN ST_Equals(a.poly, b.poly)
    THEN 'Spatially Equal' ELSE 'Not Equal' end
    FROM polygons as a, polygons as b;

![https://github.com/deamorim2/sbde/blob/master/wiki/22/pgadmin_02.png](https://github.com/deamorim2/sbde/blob/master/wiki/22/pgadmin_02.png)

Esses resultados estão mais de acordo com nossa compreensão intuitiva da igualdade.

Os polígonos 1 a 4 são considerados iguais, uma vez que abrangem a mesma área. Note que nem a direção do polígono é desenhada, o ponto de partida para definir o polígono, nem o número de pontos usados são importantes aqui. O importante é que os polígonos contenham o mesmo espaço.

## 22.1.3. Limites iguais

Igualdade exata requer, na pior das hipóteses, a comparação de cada um dos vértices da geometria para determinar a igualdade.

Isso pode ser lento e pode não ser apropriado para comparar números enormes de geometrias.

Para permitir uma comparação mais rápida, utilize o operador de limites iguais `=`. Isso opera somente no retângulo envolvente mínimo (R.E.M), garantindo que as geometrias ocupem a mesma extensão bidimensional, mas não necessariamente o mesmo espaço.

    SELECT a.name, b.name, CASE WHEN a.poly = b.poly
    THEN 'Equal Bounds' ELSE 'Non-equal Bounds' end
    FROM polygons as a, polygons as b;

![https://github.com/deamorim2/sbde/blob/master/wiki/22/pgadmin_03.png](https://github.com/deamorim2/sbde/blob/master/wiki/22/pgadmin_03.png)

Como você pode ver, todas as nossas geometrias espacialmente iguais também têm limites iguais. Infelizmente, o Polígono 5 também é retorna como igual nesse teste, porque ele compartilha a mesma caixa delimitadora das outras geometrias.

Por que isso é útil, então? Embora isso seja abordado em detalhes mais adiante, a resposta mais rápida é que isso permite o uso de indexação espacial que pode reduzir rapidamente os conjuntos enormes de comparação em blocos mais gerenciáveis ao unir ou filtrar dados.
