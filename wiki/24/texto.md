As principais propostas apresentadas até hoje sobre a representação de relações topológicas entre objetos geométricos em um espaço bidimensional incluem:

* Método das Quatro Intersecções(4IM) (Egenhofer & Franzosa, 1990)
* Método das Nove Intersecções(9IM) (Egenhofer & Herring 1991; Pullar & Egenhofer, 1988)
* Método Dimensionalmente Estendido(DEM)
* Método Baseado em Cálculo(CBM) (Clementini et al., 1993)
* [Modelo de Nove Intersecções Dimensionalmente Estendido (DE-9IM)](http://en.wikipedia.org/wiki/DE-9IM) (Clementini & Felice, 1995)

O DE-9IM é método ou modelo escolhido pela OGC/ISO e implementado no PostGIS para a representação de modelos topológicos.

# 24.1 Modelo de Nove Intersecções Dimensionalmente Estendido (DE 9IM)

Os tipos de elementos espaciais suportados no Método de nove interseções estendidas dimensionalmente (DE-9IM) para representação de objetos geométricos vetoriais em um espaço bidimensional são pontos, linhas e polígonos.

Este método de representação de relações topológicas considera as operações interior(I), exterior(E) e limite(B) para cada objeto geométrico.

Outras operações dentro do método de nove interseções dimensionalmente estendidas (DE-9IM) são a matriz de interseção (∩) e o conceito de dimensionalidade (dim) de um objeto geométrico ou dimensionalidade de um produto resultante da interseção entre os interiores, limites e exteriores. das duas geometrias, que é a relação topológica em si.

## 24.1.1 Interior(I), Limite(B) e Exterior(E)

O [Modelo de Nove Intersecções Dimensionalmente Estendido (DE-9IM)](http://en.wikipedia.org/wiki/DE-9IM) é uma estrutura para modelar como dois objetos espaciais se interagem.

Primeiro, todo objeto espacial tem:

* Um interior
* Um limite
* Um exterior

Para **polígonos**, o interior, o limite e o exterior são óbvios:

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im1.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im1.jpg)

* `Interior` é a parte delimitada pelos anéis
* `Limite` são os próprios anéis
* `Exterior` é tudo o mais no plano

Para características **lineares**, o interior, limite e exterior são menos conhecidos:

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im2.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im2.jpg)

* `Interior` é a parte da linha delimitada pelas extremidades
* `Limite` é o início e o fim do traço linear (extremidades)
* `Exterior` é todo o resto do plano

Para **pontos**, as coisas são ainda mais estranhas:

* `Interior` é o próprio ponto
* `Limite` é o conjunto vazio
* `Exterior` é todo o resto do plano

Usando essas definições de `Interior`, `Exterior` e `Limite`, as relações entre qualquer par de características espaciais podem ser caracterizadas usando a dimensionalidade das nove interseções possíveis entre os interiores/limites/exteriores de um par de objetos.

## 24.1.2 Dimensionalidade da Interação Entre os Objetos

O modelo de nove interseções dimensionalmente estendido (DE-9IM) inclui a interseção de interiores, limites e exteriores entre um objeto geométrico A e um objeto geométrico B.

Dependendo do tipo de objeto geométrico, o resultado da dimensionalidade pode ser de tipos: Falso (F) ou vazio (∅), 0 (ponto), 1 (linha) e 2 (polígono). Dimensionalidade True (T) ou não-vazia (∅) corresponde a todos os resultados, exceto False (F) ou empty (∅).

Além disso, este método emprega o caractere “*” como para qualquer tipo de resultado possível (F, 0, 1 ou 2).

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im3.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im3.jpg)

Para os polígonos no exemplo acima, a interseção dos interiores é uma área bidimensional, de modo que a parte da matriz é preenchida com um “2”. Os limites apenas se cruzam em pontos, que são de dimensão zero, de modo que a parte da matriz é preenchida com um 0.

Quando não há interseção entre os componentes, o quadrado da matriz é preenchido com um “F”.
Veja outro exemplo, de uma cadeia de linhas inserindo parcialmente um polígono:

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im4.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im4.jpg)

A matriz DE9IM para a interação é esta:

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im5.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im5.jpg)

Note que os limites dos dois objetos não se cruzam de fato (o ponto final da linha interage com o interior do polígono, não o limite, e vice-versa), então a célula B/B é preenchida com um "F".

## 24.1.3 ST_Relate

Embora seja divertido preencher visualmente matrizes DE9IM, seria bom se um computador pudesse fazê-lo, e é para isso que serve a função [ST_Relate](https://postgis.net/docs/ST_Relate.html).

O exemplo anterior pode ser simplificado usando uma simples caixa e linha, com o mesmo relacionamento espacial que o nosso polígono e cadeia de linhas:

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im6.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im6.jpg)

E nós podemos gerar a informação DE9IM no SQL:

    SELECT ST_Relate('LINESTRING(0 0, 2 0)','POLYGON((1 -1, 1 1, 3 1, 3 -1, 1 -1))');

A resposta (1010F0212) é a mesma que calculamos visualmente, mas retornou como uma string de 9 caracteres, com a primeira linha, segunda linha e terceira linha da tabela anexadas juntas.
***
    101
    0F0
    212
***

No entanto, o poder das matrizes DE9IM não está em gerá-las, mas em usá-las como uma chave de correspondência para encontrar geometrias com relações muito específicas entre si.

    CREATE TABLE lakes ( id serial primary key, geom geometry );
    CREATE TABLE docks ( id serial primary key, good boolean, geom geometry );

    INSERT INTO lakes ( geom )
    VALUES ( 'POLYGON ((100 200, 140 230, 180 310, 280 310, 390 270, 400 210, 320 140, 215 141, 150 170, 100 200))');

    INSERT INTO docks ( geom, good ) VALUES
    ('LINESTRING (170 290, 205 272)',true),
    ('LINESTRING (120 215, 176 197)',true),
    ('LINESTRING (290 260, 340 250)',false),
    ('LINESTRING (350 300, 400 320)',false),
    ('LINESTRING (370 230, 420 240)',false),
    ('LINESTRING (370 180, 390 160)',false);

Suponha que tenhamos um modelo de dados que inclua Lagos(lakes) e Docas(docks) e suponha ainda que as Docas devem estar dentro de lagos e devem tocar o limite de seu lago contido em uma extremidade. Podemos encontrar todas as docas em nosso banco de dados que obedecem a essa regra?

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im7.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im7.jpg)

Nossas docas legais possuem as seguintes características:

* Seus interiores têm uma interseção linear (1D) com o interior do lago
* Seus limites têm uma interseção de ponto (0D) com o interior do lago
* Seus limites também têm uma interseção de ponto (0D) com o limite do lago
* Seus interiores não têm interseção (F) com o exterior do lago

Então, a matriz DE9IM é assim:

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im8.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im8.jpg)

Assim, para encontrar todas as docas legais, gostaríamos de encontrar todas as docas que cruzam os lagos (um super conjunto de potenciais candidatos que usamos para a nossa chave de junção) e, em seguida, encontrar todas as docas nesse conjunto que tenham o padrão legal relacionado.

    SELECT docks.*
    FROM docks JOIN lakes ON ST_Intersects(docks.geom, lakes.geom)
    WHERE ST_Relate(docks.geom, lakes.geom, '1FF00F212');

Observe o uso da versão de três parâmetros de ST_Relate, que retorna true se o padrão corresponder ou false, se não corresponder. Para um padrão totalmente definido como este, a versão de três parâmetros não é necessária - poderíamos ter usado apenas um operador de igualdade de cadeia.

No entanto, para pesquisas de padrão mais flexível, o terceiro parâmetro permite caracteres de substituição na string padrão:

* "*" Significa "qualquer valor nesta célula é aceitável"
* "T" significa "qualquer valor não falso (0, 1 ou 2) é aceitável"

Assim, por exemplo, uma possível doca que não incluímos em nosso gráfico de exemplo é uma doca com uma interseção bidimensional com o limite do lago:

    INSERT INTO docks ( geom, good )
    VALUES ('LINESTRING (140 230, 150 250, 210 230)',true);

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im9.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im9.jpg)

Se quisermos incluir este caso em nosso conjunto de docas "legais", precisamos alterar o padrão relacionado em nossa consulta. Em particular, a interseção do limite do lago interior da doca agora pode ser 1 (nosso novo caso) ou F (nosso caso original). Então, usamos o "*" que significa qualquer estado no padrão.

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im10.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im10.jpg)

O SQL se parece com isso:

    SELECT docks.*
    FROM docks JOIN lakes ON ST_Intersects(docks.geom, lakes.geom)
    WHERE ST_Relate(docks.geom, lakes.geom, '1*F00F212');

Confirme se o SQL mais estrito do exemplo anterior não retorna o novo encaixe.

# 24.2 Relacionamentos Espaciais na DE-9IM

Clementini e Felice (1995) afirmam que todas as relações aceitáveis propostas no método CBM podem ser representadas usando o método DE-9IM e que todas as possíveis relações topológicas entre objetos geométricos de tipos ponto, linha e polígono em um espaço bidimensional podem ser agrupadas em cinco categorias ou relacionamentos topológicos:

* Touch (Toca)
* In (Em/Dentro de)
* Cross (Cruza)
* Overlaps (Sobrepõe)
* Disjoint (Disjunto)

A SFSQL e a SQLMM utilizam a especificação da DE9IM e, como consequência, utilizam e especificam esses relacionamentos topológicos. 

Portanto, as seguintes equações e os padrões resultantes podem ser possíveis sob o método DE-9IM e seus relacionamentos topológicos implementado pelo PostGIS que implementa as especificações da SFSQL/SQLMM:

## 24.2.1 Touch(ST_Touches)

Aplicado para grupos: **polígono/polígono**, **linha/linha**, **linha/polígono**, **ponto/polígono** e **ponto/linha**, exceto para o **ponto/ponto** do grupo:

〈A, touch, B〉 = [I(A) ∩ I(B) = ∅] e [B(A) ∩ I(B) ≠ ∅] ou [I(A) ∩ B(B) ≠ ∅] ou [B(A) ∩ B(B) ≠ ∅]

    Padrão correspondente da matriz DE-9IM = ( F T * * * * * * *), (F * * T * * * * *) e (F * * * T * * * *)

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_touch.png](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_touch.png)

## 24.2.2 In(ST_Within/ST_Contains)

Aplicado a **todos os grupos**:

〈A, in, B〉 = [I(A) ∩ I(B) ≠ ∅] e [I(A) ∩ E(B) = ∅] e [B(A) ∩ E(B) = ∅]

    Padrão correspondente da matriz DE-9IM: (T * F * * F * * *)

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_within.png](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_within.png)

## 24.2.3 Cross(ST_Crosses)

Relacionamento cruzado (Aplicado para grupos linha/linha ou linha/polígono)

**Linha/linha**:

〈A, cross, B〉 = dim [I(A) ∩ I(B) = 0]

    Padrão correspondente da matriz DE-9IM = (0 * * * * * * * *)

**Linha/Polígono**:

〈A, cross, B〉 = [I(A) ∩ I(B) ≠ ∅] e [I(A) ∩ E(B) ≠ ∅]

    Padrão correspondente da matriz DE-9IM = (T * T * * * * * *)

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_cross.png](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_cross.png)

## 24.2.4 Overlap(ST_Overlaps)

Relação de sobreposição (Aplicada à linha / linha de grupos ou polígono / polígono):

**Linha/linha**:

〈A, overlap, B〉 = dim [I(A) ∩ I(B) = 1] e [I(A) ∩ E(B) ≠ ∅] e [E(A) ∩ I(B) ≠ ∅]

    Padrão correspondente da matriz DE-9IM = (1 * T * * * T * *)

**Polígono/Polígono**:

〈A, sobreposição, B〉 = [I(A) ∩ I(B) ≠ ∅] e [I(A) ∩ E(B) ≠ ∅] e [E(A) ∩ I(B) ≠ ∅]

    Padrão correspondente da matriz DE-9IM = (T * T * * * T * *)

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_overlap.png](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_overlap.png)

## 24.2.5 Disjoint(ST_Disjoint)

Relação de Disjunção (aplicada para **todos os grupos**):

〈A, disjunta, B〉 = [I(A) ∩ I(B) = ∅] e [B(A) ∩ I(B) = ∅] e [I(A) ∩ B(B) = ∅] e [B(A) ∩ B(B) = ∅]

    Matriz de DE-9IM padrão correspondente = (F F * F F * * * *)

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_disjoint.png](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im_disjoint.png)

# 24.3 Teste de Qualidade de Dados

Uma maneira de verificar a qualidade dos dados da tabela `lim_unidade_federacao_a` é por meio da função ST_Relate. Por exemplo: nenhuma unidade da federação deve sobrepor qualquer outra unidade da federação. Podemos testar para isso?

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im11.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im11.jpg)

Com certeza!

    SELECT a.gid, b.gid, ST_Multi((ST_Dump(ST_Intersection(a.geom, b.geom))).geom) as geom
    FROM lim_unidade_federacao_a a, lim_unidade_federacao_a b
    WHERE ST_Intersects(a.geom, b.geom)
    AND ST_Relate(a.geom, b.geom, '2********')
    AND a.gid != b.gid
    LIMIT 10;

Da mesma forma, esperamos que os dados das rodovias da tabela geométrica `tra_trecho_rodoviario_l` sejam todos de ponta. Ou seja, esperamos que as interseções só ocorram nas extremidades das linhas(nós), não nos pontos médios(aresta ou vértice).

![https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im12.jpg](https://github.com/deamorim2/sbde/blob/master/wiki/24/de9im12.jpg)

Podemos testar isso procurando por ruas que se cruzam (então temos uma junção), mas onde a interseção entre os limites não é zero-dimensional (isto é, os pontos finais não tocam):

    SELECT a.gid, b.gid
    FROM tra_trecho_rodoviario_l a, tra_trecho_rodoviario_l b
    WHERE ST_Intersects(a.geom, b.geom)
    AND NOT ST_Relate(a.geom, b.geom, '****0****')
    AND a.gid != b.gid
    LIMIT 10;

# 24.4 Lista de funções

ST_Relate(geometria A, geometria B): retorna o texto que representa o relacionamento DE9IM entre as geometrias.

# 24.5 Referência Bibliográfica

* EGENHOFER, M. J.; CLEMENTINI, E.; FELICE, P. D. Topological relations between regions with holes. International Journal of Geographic Information Systems, v 8, n. 2, p. 129-142, 1994.
* EGENHOFER, M. J.; FRANZOSA, R. D. Point-set topological spatial relations. International Journal of Geographic Information Systems. v. 5, n. 2, p. 161-174, 1990.
* EGENHOFER, M. J.; HERRING, J. A mathematical framework for the definition of topological relationships. In: 4th INTERNATIONAL SYMPOSIUM ON SPATIAL DATA HANDLING, 4., 1991, Zürich, Switzerland. Proceedings…Zürich, Switzerland, 1991. p. 803-813.
* CLEMENTINI, E. A model for uncertain lines. Journal of Visual Languages and Computing 16, p. 271-288, 2005.
* CLEMENTINI, E.; DIFELICE, P.; VAN OOSTEROM, P. A small set of formal topological relationships suitable for end-user interaction. In: 3rd SYMPOSIUM ON SPATIAL DATABASE SYSTEMS, 3., 1993, Singapore. Proceedings…Singapore. 1993. p. 277-295.
* CLEMENTINI, E.; FELICE, P. D. A Comparison of Methods for Representing Topological Relationships. Information Sciences, v. 3, p. 149-178, 1995.
* CLEMENTINI, E.; FELICE, P. D. A model for representing topological relationships between complex geometric features in spatial databases. Information Sciences: an International Journal archive, v. 90, n. 1-4, 1996.
* CLEMENTINI, E.; FELICE, P. D. Approximate Topological Relations. International Journal of Approximate Reasoning, v. 16, n. 2, p. 173-204, 1997.
* CLEMENTINI, E.; FELICE, P. D. CALIFANO, G. Composite Regions In Topological Queries. Information Systems, v. 20, n. 7, p. 579-594, 1995.
* CLEMENTINI, E.; FELICE, P. D. KOPERSKI, K. Mining multiple-level spatial association rules for objects with a broad boundary. Data & Knowledge Engineering, v. 34, p. 251-270, 2000.
* CLEMENTINI, E.; FELICE, P. D. Topological Invariants for Lines. IEEE Transactions On Knowledge And Data Engineering, v. 10, n. 1, 1998.