# 21. Validade Geométrica

Em 90% dos casos, a resposta à pergunta "por que minha consulta está causando um erro de 'TopologyException'" é "uma ou mais entradas inválidas". O que levanta a questão: o que significa ser inválido e por que devemos nos importar?

# 21.1 O que é validade geométrica?

A validade geométrica é mais importante para os polígonos, que definem áreas delimitadas e exigem uma boa estrutura. As linhas são muito simples e não podem ser inválidas, nem pontos.

Algumas das regras da validade do polígono parecem óbvias e outras parecem arbitrárias (e, na verdade, são arbitrárias).

* Anéis de polígono devem fechar.

* Os anéis que definem os "buracos" devem estar dentro dos anéis que definem os limites externos.

* Anéis não podem se auto-interceptar (eles não podem tocar nem cruzar um ao outro).

* Os anéis não podem tocar em outros anéis, exceto em um ponto.

As duas últimas regras estão na categoria arbitrária.

Existem outras maneiras de definir polígonos que são consistentes, mas as regras acima são aquelas usadas pelo padrão OGC SFSQL que o PostGIS está em conformidade.

A razão pela qual as regras são importantes é porque os algoritmos para cálculos de geometria dependem da estrutura consistente nas entradas.

É possível construir algoritmos que não tenham suposições estruturais, mas essas rotinas tendem a ser muito lentas, porque o primeiro passo em qualquer rotina livre de estrutura é analisar as entradas e construir a estrutura nelas.

Veja um exemplo de por que a estrutura é importante.

Este polígono é inválido:

    POLYGON((0 0, 0 1, 2 1, 2 2, 1 2, 1 0, 0 0));

Você pode ver a invalidade um pouco mais claramente neste diagrama:

![https://github.com/deamorim2/sbde/blob/master/wiki/21/figure_eight.png](https://github.com/deamorim2/sbde/blob/master/wiki/21/figure_eight.png)

O anel externo é na verdade a figura de um "oito", com uma auto-interseção no meio. Observe que as rotinas gráficas renderizam com sucesso o preenchimento do polígono, de modo que visualmente ele parece ser uma “área”: dois quadrados de uma unidade, portanto, uma área total de duas unidades de área.

Vamos ver o que o banco de dados acha que a área do nosso polígono é:

    SELECT ST_Area(ST_GeometryFromText('POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'));

***
    st_area
    ---------
    0
***

O que esta acontecendo aqui?

O algoritmo que calcula a área assume que os anéis não se cruzam. Um anel bem construído sempre terá a área delimitada (o interior) em um lado da linha delimitadora (não importa qual lado, apenas que está de um lado). No entanto, no nosso (mal comportado) "oito", a área delimitada está à direita da linha de um lóbulo e à esquerda do outro. Isso faz com que as áreas calculadas para cada lóbulo sejam canceladas (uma aparece como 1 e a outra como -1), portanto, o resultado de “área zero”.

# 21.2 Detectando Validade

No exemplo anterior, tínhamos um polígono que sabíamos que era inválido. Como detectamos a invalidez em uma tabela com milhões de geometrias? Resposta: com a função ST_IsValid (geometria).

Usando essa função a partir da nossa figura "oito", obtemos uma resposta rápida:

    SELECT ST_IsValid(ST_GeometryFromText('POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'));
***
    f
***

Agora sabemos que o recurso é inválido, mas não sabemos por quê.

Podemos usar a função ST_IsValidReason (geometry) para descobrir a origem da invalidade:

    SELECT ST_IsValidReason(ST_GeometryFromText('POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'));
***
    Self-intersection[1 1]
***

Observe que, além do motivo (auto-interseção), a localização da invalidez (coordenada (1 1)) também é retornada.
Podemos usar a função ST_IsValid (geometry) para testar nossas tabelas também:

    SELECT nome, geocodigo, ST_IsValidReason(geom)
    FROM lim_municipio_a
    WHERE NOT ST_IsValid(geom);

![https://github.com/deamorim2/sbde/blob/master/wiki/21/pgadmin_01.png](https://github.com/deamorim2/sbde/blob/master/wiki/21/pgadmin_01.png)

Repare que quatro municípios possuem geometria inválida.

# 21.3 Reparando Geometria Inválida

Primeiro, a má notícia: não há uma maneira 100% garantida de corrigir geometrias inválidas.

O pior cenário é identificá-los com a função ST_IsValid (geometria), movendo-os para uma tabela auxiliar, exportando essa tabela e reparando-a externamente.

Veja um exemplo de SQL para mover geometrias inválidas da tabela principal para uma tabela auxiliar adequada para o processo de consistência topológica externo.

    -- Criação da tabela auxiliar
    CREATE TABLE lim_municipio_a_invalid AS
    SELECT * FROM lim_municipio_a
    WHERE NOT ST_IsValid(geom);

    -- Remoção das geometrias originais inválidas
    DELETE FROM lim_municipio_a
    WHERE NOT ST_IsValid(geom);

    -- Inserção das geometrias corrigidas após a correção das geometrias inválidas
    INSERT INTO lim_municipio_a
    SELECT *
    FROM lim_municipio_a_invalid

A partir da instrução SQL abaixo, você pode criar uma tabela geométrica do tipo ponto com o ponto onde a geometria é inválida:

    DROP TABLE IF EXISTS lim_municipio_a_ponto_invalido;

    CREATE TABLE lim_municipio_a_ponto_invalido AS
    SELECT geocodigo, nome, ST_IsValidReason(geom) as validreason, ST_MULTI(ST_SETSRID(location(ST_IsValidDetail(geom)), ST_SRID(geom))) as geom_point
    FROM lim_municipio_a
    WHERE ST_IsValidReason(geom) <> 'Valid Geometry';

O QGIS é uma boa ferramenta para reparar manualmente a geometria inválida, pois possui uma rotina de validação em QGIS>Vetor>Verificador de topologia>Verificador de Topologia.

Agora, a boa notícia: uma grande proporção de invalidez pode ser corrigida no banco de dados usando:

* ST_MakeValid
* ST_Buffer

## 21.3.1. ST_MakeValid

ST_MakeValid tenta reparar as geometrias inválidas com alterações mínimas nas geometrias de entrada. Nenhum vértice é removido ou apagado, a estrutura do objeto é simplesmente reorganizada. Isso é bom para dados simples, mas inválidos, e uma coisa ruim para dados complexos e inválidos.

>Corrija o polígono inválido da nossa figura "oito":

    SELECT ST_AsText(ST_MakeValid(
    'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))')
    );
***
    MULTIPOLYGON(((0 0,0 1,1 1,1 0,0 0)),((1 1,1 2,2 2,2 1,1 1)))

ST_MakeValid converte com êxito a figura "oito" em um polígono múltiplo que representa a mesma área.

Vamos utilizar a função ST_MakeValid para validar as geometrias não válidas da tabela `lim_municipio_a`:

    UPDATE lim_municipio_a
    SET geom = ST_MakeValid(geom)
    WHERE NOT ST_IsValid(geom);

>Cheque novamente se todas as feições da tabela `lim_municipio_a` são válidas:

    SELECT nome, geocodigo, ST_IsValidReason(geom)
    FROM lim_municipio_a
    WHERE NOT ST_IsValid(geom);

Repare que todas as geometrias foram validadas.

## 21.3.2. ST_Buffer

A limpeza usando o truque de buffer aproveita a maneira como os buffers são construídos: uma geometria em buffer é uma nova geometria, construída por linhas de deslocamento da geometria original. Se você compensar as linhas originais por nada (zero), a nova geometria será estruturalmente idêntica à original, mas como é construída usando as regras de topologia do OGC, ela será válida.

Por exemplo, aqui está uma invalidez clássica - o "polígono da banana" - um único anel que envolve uma área, mas se inclina para se tocar, deixando um "buraco" que na verdade não é um "buraco".

    POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))

![https://github.com/deamorim2/sbde/blob/master/wiki/21/banana.png](https://github.com/deamorim2/sbde/blob/master/wiki/21/banana.png)

A execução do buffer de deslocamento zero no polígono retorna um polígono OGC válido, consistindo de um anel externo e interno que toca em um ponto.

    SELECT ST_AsText(ST_Buffer(ST_GeometryFromText('POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))'),0.0));
***
    POLYGON((0 0,0 4,4 4,4 0,2 0,0 0),(2 0,3 1,2 2,1 1,2 0))
***
**Nota**


O “polígono da banana” (ou “casca invertida”) é um caso em que o modelo de topologia OGC para geometria válida e o modelo usado internamente pela ESRI são diferentes. O modelo ESRI considera os anéis que tocam como inválidos e prefere a forma de banana para esse tipo de formato. O modelo OGC é o inverso. Nenhum deles é "correto", eles são apenas maneiras diferentes de modelar a mesma situação.
***
