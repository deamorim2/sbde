# 11. Relacionamentos Espaciais

Até agora, usamos apenas funções espaciais que medem geometrias (ST_Area, ST_Length), serializar (ST_GeomFromText) ou desserializar (ST_AsText). O que essas funções têm em comum é que elas só funcionam em uma geometria por vez.

Os bancos de dados espaciais são poderosos porque eles não apenas armazenam geometria, mas também têm a capacidade de comparar os relacionamentos entre as geometrias.

Questões como “Quais são os bicicletários mais próximos de um parque?” Ou “Onde estão as interseções de linhas de metrô e ruas?” Só podem ser respondidas comparando as geometrias que representam os bicicletários, ruas e linhas de metrô.

Os padrões [SFSQL](http://www.opengeospatial.org/standards/sfa) (OGC) ou [SQLMM](https://www.iso.org/standard/60343.html) (ISO) define o seguinte conjunto de métodos para comparar geometrias.
 
# 11.1. ST_Touches, ST_Within(ST_Contains), ST_Crosses, ST_Overlaps e ST_Disjoint

## 11.1.1 ST_Touches

ST_Touches testa se duas geometrias tocam em seus limites, mas não se cruzam em seus interiores

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_touches.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_touches.png)

ST_Touches(geometria A, geometria B) - retorna TRUE se um dos limites das geometrias se cruzar ou se apenas um dos interiores da geometria cruzar o limite do outro.

## 11.1.2 ST_Within (ST_Cointains)

ST_Within e ST_Contains testam se uma geometria está totalmente dentro da outra.
 
![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_within.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_within.png)

ST_Within(geometria A, geometria B) - retorna TRUE se a primeira geometria estiver completamente dentro da segunda geometria. ST_Within testa o resultado oposto exato de ST_Contains.

ST_Contains(geometria A, geometria B) - retorna TRUE se a segunda geometria estiver completamente contida pela primeira geometria.

## 11.1.3 ST_Crosses

Para comparações multiponto/polígono, multiponto/linestring, linestring/linestring, linestring/polígono e linestring/multipolígono, ST_Crosses(geometria A, geometria B) retorna t (VERDADEIRO) se a interseção resultar em uma geometria cuja dimensão é menor que a dimensão máxima das duas geometrias de origem e o conjunto de interseções é interior a ambas as geometrias de origem.

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_crosses.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_crosses.png)

## 11.1.4 ST_Overlaps

ST_Overlaps(geometria A, geometria B) - compara duas geometrias da mesma dimensão e retorna TRUE se o conjunto de interseções resultar em uma geometria diferente de ambas, mas da mesma dimensão.

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_overlaps.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_overlaps.png)

Vamos pegar nossa estação de metrô Broad Street e determinar sua vizinhança usando a função ST_Intersects:

## 11.1.5 ST_Disjoint

Se duas geometrias são separadas, elas não se cruzam e vice-versa. Na verdade, muitas vezes é mais eficiente testar “não intercepta” do que testar “disjunto” porque os testes de interseção podem ser indexados espacialmente, enquanto o teste não pode ser dividido.

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_disjoint.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_disjoint.png)

ST_Disjoint(geometria A, geometria B) - retorna TRUE se as Geometrias não “intersectarem espacialmente” - se elas não compartilharem nenhum espaço juntas.

# 11.2. ST_Equals
 
ST_Equals retornará TRUE se duas geometrias do mesmo tipo tiverem valores de coordenadas x, y idênticos, isto é, se a segunda forma for igual (idêntica) à primeira forma.
Primeiro, vamos recuperar uma representação de um ponto da nossa tabela nyc_subway_stations. Vamos pegar apenas a entrada para "Broad St".

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_equals.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_equals.png)

ST_Equals(geometria A, geometria B) - testa a igualdade espacial de duas geometrias.

>Primeiro, vamos recuperar uma representação em WKB de um ponto da nossa tabela `loc_cidade_p`. Vamos pegar apenas a entrada para a cidade de 'Patos de Minas'.

    SELECT geom
    FROM loc_cidade_p
    WHERE nome = 'Patos de Minas';
***
    0101000020421200004D8F4809F14247C0A5B7586D589732C0
***
>Agora, conecte a representação geométrica em WKB novamente em um teste ST_Equals:

    SELECT nome
    FROM loc_cidade_p
    WHERE ST_Equals(geom, '0101000020421200004D8F4809F14247C0A5B7586D589732C0');
***
    Patos de Minas
***
**Nota:**
 a representação do ponto não é muito legível nesse formato (0101000020421200004D8F4809F14247C0A5B7586D589732C0), mas era uma representação exata dos valores das coordenadas. Para um teste como igualdade, é necessário usar as coordenadas exatas.
***
# 11.3. ST_Intersects

ST_Intersects testa se limites ou interiores das geometrias se sobrepõem.

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_intersects.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_intersects.png)
 
ST_Intersects (geometria A, geometria B) - retorna t (VERDADEIRO) se as duas formas tiverem algum espaço em comum, ou seja, se seus limites ou interiores se cruzam.

O oposto de ST_Intersects é ST_Disjoint(geometria A, geometria B). Se duas geometrias são separadas, elas não se sobrepõem e vice-versa. Na verdade, muitas vezes é mais eficiente testar “não intercepta” do que testar “disjunto” porque os testes de interseção podem ser indexados espacialmente, enquanto o teste não pode ser dividido.

>Utilize a função ST_Intersects para saber em qual estado esta a cidade de "Patos de Minas".

>Obtenha o WKT da cidade de "Patos de Minas"

    SELECT ST_AsText(geom)
    FROM loc_cidade_p
    WHERE nome = 'Patos de Minas';
***
    POINT(-46.522980843 -18.5911930409999)
***

>Agora, utilize essa informação para saber qual estado instersecta a cidade de "Patos de Minas"

    SELECT nome
    FROM lim_unidade_federacao_a
    WHERE ST_Intersects(geom, ST_GeomFromText('POINT(-46.522980843 -18.5911930409999)',4674));
***
    Minas Gerais
***
# 11.4. ST_Distance e ST_DWithin

## 11.4.1 ST_Distance

Uma questão GIS extremamente comum é “encontrar todas as coisas dentro da distância X dessas outras coisas”.

A ST_Distance(geometria A, geometria B) calcula a menor distância entre duas geometrias e a retorna como um número do tipo _float_. Isso é útil para relatar a distância entre os objetos.

    SELECT ST_Distance(
    ST_GeometryFromText('POINT(0 5)'),
    ST_GeometryFromText('LINESTRING(-2 2, 2 2)'));
***
    3
***

## 11.4.2 ST_DWithin

Para testar se dois objetos estão a uma distância um do outro, a função ST_DWithin fornece um teste verdadeiro/falso acelerado por índice. Isso é útil para perguntas como “quantas árvores estão dentro de um buffer de 500 metros da estrada?”. Você não precisa calcular um buffer real, basta testar o relacionamento à distância.

![https://github.com/deamorim2/sbde/blob/master/wiki/11/st_dwithin.png](https://github.com/deamorim2/sbde/blob/master/wiki/11/st_dwithin.png)

Vamos selecionar todas as geometrias da tabela `geometries` que estão à distância de uma unidade do ponto `'POINT(0 1)'`

    SELECT name
    FROM geometries
    WHERE ST_DWithin(geom, ST_GeomFromText('POINT(0 1)'), 1);
***
    name
    -----------
    Point
    Linestring
    Polygon
****

# 11.5 Lista de Funções

* ST_Contains (geometria A, geometria B): Retorna verdadeiro se e somente se nenhum ponto de B estiver no exterior de A, e pelo menos um ponto do interior de B estiver no interior de A.
* ST_Crosses (geometria A, geometria B): Retorna TRUE se as geometrias fornecidas tiverem alguns, mas não todos, pontos interiores em comum.
* ST_Disjoint (geometria A, geometria B): Retorna TRUE se as Geometrias não “intersectarem espacialmente” - se elas não compartilharem nenhum espaço juntas.
* ST_Distance (geometria A, geometria B): retorna a distância mínima cartesiana bidimensional (com base na referência espacial) entre duas geometrias em unidades projetadas.
* ST_DWithin (geometria A, geometria B, raio): Retorna verdadeiro se as geometrias estiverem dentro da distância especificada (raio) uma da outra.
* ST_Equals (geometria A, geometria B): Retorna verdadeiro se as geometrias especificadas representarem a mesma geometria. A direcionalidade é ignorada.
* ST_Intersects (geometria A, geometria B): retorna TRUE se as Geometrias / Geografia “intersectarem espacialmente” (compartilham qualquer parte do espaço) e FALSE se não forem (elas são Disjoint).
* ST_Overlaps (geometria A, geometria B): Retorna TRUE se as Geometrias compartilharem espaço, forem da mesma dimensão, mas não estiverem completamente contidas umas nas outras.
* ST_Touches (geometria A, geometria B): Retorna TRUE se as geometrias tiverem pelo menos um ponto em comum, mas seus interiores não se cruzam.
* ST_Within (geometria A, geometria B): Retorna verdadeiro se a geometria A estiver completamente dentro da geometria B
