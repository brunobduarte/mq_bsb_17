# Tutorial A2 - Webscrapping de um portais de notícias - ALESP

Na primeira atividade da oficina não precisamos lidar com o conteúdo e a estrutura da página que estávamos capturando. Como o conteúdo que nos interessava estava em uma tabela, e o pacote "XML" contém uma função específica para captura de tabelas, gastamos apenas duas linhas de código para ter a informação capturada já organizada em um data frame.

O que fazer, entretanto, com páginas que não tem tabelas? Como obter apenas as informações que nos interessam quando o conteúdo está "espalhado" pela página? Utilizaremos, como veremos abaixo, a estrutura do código HTML da própria página para selecionar apenas o que desejamos e construir data frames.

Nosso objetivo nessa atividade será capturar uma única página usando a estrutura do código HTML da página. Já sabemos que, uma vez resolvida a captura de uma página, podemos usar "loop" para capturar quantas quisermos, desde que tenha uma estrutura semelhante.

Antes disso, porém, precisamos falar um pouco sobre XML e HTML.

## XML e HTML

XML significa "Extensive Markup Language". Ou seja, é uma linguagem -- e, portanto, tem sintaxe -- e é uma linguagem com marcação. Marcação, neste caso, significa que todo o conteúdo de um documento XML está dentro de "marcas", também conhecidas como "tags". É uma linguagem extremamente útil para transporte de dados -- por exemplo, a Câmara dos Deputados utiliza XML em seu Web Service para disponibilizar dados abertos (mas você não precisa saber disso se quiser usar o pacote de R bRasilLegis que nós desenvolvemos ;) -- disponível aqui: https://github.com/leobarone/bRasilLegis.

Por exemplo, se quisermos organizar a informação sobre um indivíduo que assumiu diversos postos públicos, poderíamos organizar a informação da seguinte maneira:

```{r}
<politicos>
  <politico>
    <id> 33333 </id>
    <nome> Fulano Deputado da Silva </nome>
    <data_nascimento> 3/3/66 </data_nascimento>
    <sexo> Masculino </sexo>
    <cargos>
      <cargo> 
        <cargo> prefeito </cargo> 
        <partido> PAN </partido>
        <ano_ini> 2005 </ano_ini>
        <ano_fim> 2008 </ano_fim>
      </cargo>
      <cargo> 
        <cargo> deputado federal </cargo> 
        <partido> PAN </partido>
        <ano_ini> 2003 </ano_ini>
        <ano_fim> 2004 </ano_fim>
       </cargo>
       <cargo> 
        <cargo> deputado estadual </cargo> 
        <partido> PAN </partido>
        <ano_ini> 1998 </ano_ini>
        <ano_fim> 2002 </ano_fim>
       </cargo>
      </cargos>
  </politicos>
</politicos>
```

## Exercício (difícil)

Se tivessemos que representar estes dados em um banco de dados (data frame), como seria? Quantas linhas teria? Quantas colunas teria?

Veja no link abaixo um exemplo de arquivo XML proveniente do Web Service da Câmara dos Deputados:

http://www.camara.gov.br/SitCamaraWS/Deputados.asmx/ObterDetalhesDeputado?ideCadastro=141428&numLegislatura=

## Portal da Assembléia Legislativa do Estado de São Paulo

Vamos trabalhar neste tutorial com o portal da Assembléia Legislativa do Estado de São Paulo e a busca de notícias dentro do portal.

Abra agora a página inicial da ALESP. Posicione o mouse em qualquer elemento da página e, com o botão direito, selecione "Inspecionar" (varia de navegador para navegador). Você verá o código HTML da página.

Sem precisar observar muito, é fácil identificar que o código HTML da ALESP se assemelha ao nosso breve exemplo de arquivo XML. Não por acaso: HTML é um tipo de XML. Em outra palavras, toda página de internet está em um formato de dados conhecido e, como veremos a seguir, pode ser re-organizado facilmente.

## Tags, nodes, atributos, valores e conteúdo na linguagem XML

Todas as informações em um documento XML estão dispostas em "tags" (id, nome, etc são as tags do nosso exemplo). Um documento XML é um conjunto de "tags" que contém hierarquia. Um conjunto de "tags" hierarquicamente organizado é chamado de "node". Por exemplo, no arquivo XML da Câmara dos Deputados apresentado acima, cada tag político contêm diversos outras "tags" e formam "nodes", ou seja, pedaços do arquivo XML.

Em geral, as "tags" vem em pares: uma de abertura e outra de fechamento. O que as diferencia é a barra invertida presente na tag de fechamento. Entre as "tags" de abertura e fechamento vemos o conteúdo da tag, que pode, inclusive, ser outras "tags". Veja os exemplos abaixo:

```{r}
<minha_tag> Este é o conteúdo da tag </minha_tag>

<tag_pai>
  <tag_filha>
  </tag_filha>
</tag_pai>

<tag_pai> Conteúdo da tag Pai
  <tag_filha> Conteúdo da tag Filha
  </tag_filha>
</tag_pai>
```

Identação (espaços) nos ajudam a ver a hierarquia entre as tags, mas não é obrigatória. Também as quebras de linha são opcionais.

Além do conteúdo e do nome da tag, é extremamente comum encontrarmos "atributos" nas tags em bancos de dados e, sobretudo, em códigos HTML. Atributos ajudam a especificar a tag, ou seja, identificam qual é o seu uso ou carregam quaisquer outras informações referentes. Voltando ao exemplo fictício acima, poderíamos transformar a informação do cargo, que hoje é uma tag cargo dentro de outra tag cargo (horrível, não?) em atributo.

Em vez de:
```{r}
<politicos>
  <politico>
    <id> 33333 </id>
    <nome> Fulano Deputado da Silva </nome>
    <data_nascimento> 3/3/66 </data_nascimento>
    <sexo> Masculino </sexo>
    <cargos>
      <cargo> 
        <cargo> prefeito </cargo> 
        <partido> PAN </partido>
        <ano_ini> 2005 </ano_ini>
        <ano_fim> 2008 </ano_fim>
      </cargo>
      <cargo> 
        <cargo> deputado federal </cargo> 
        <partido> PAN </partido>
        <ano_ini> 2003 </ano_ini>
        <ano_fim> 2004 </ano_fim>
       </cargo>
       <cargo> 
        <cargo> deputado estadual </cargo> 
        <partido> PRONA </partido>
        <ano_ini> 1998 </ano_ini>
        <ano_fim> 2002 </ano_fim>
       </cargo>
      </cargos>
  </politicos>
</politicos>
```
Teríamos:
```{r}
<politicos>
  <politico>
    <id> 33333 </id>
    <nome> Fulano Deputado da Silva </nome>
    <data_nascimento> 3/3/66 </data_nascimento>
    <sexo> Masculino </sexo>
    <cargos>
      <cargo tipo = 'prefeito'>
        <partido> PAN </partido>
        <ano_ini> 2005 </ano_ini>
        <ano_fim> 2008 </ano_fim>
      </cargo>
      <cargo tipo = 'deputado federal'>
        <partido> PAN </partido>
        <ano_ini> 2003 </ano_ini>
        <ano_fim> 2004 </ano_fim>
       </cargo>
      <cargo tipo = 'deputado estadual'>
        <partido> PRONA </partido>
        <ano_ini> 1998 </ano_ini>
        <ano_fim> 2002 </ano_fim>
       </cargo>
      </cargos>
  </politicos>
</politicos>
```

Veja que agora a tag "cargo" tem um atributo -- "tipo" -- cujos valores são "prefeito", "deputado federal" ou "deputado estadual". Estranho, não? Para bancos de dados em formato XML, faz menos sentido o uso de atributos. Mas para páginas de internet, atributos são essenciais. Por exemplo, sempre que encontrarmos um hyperlink em uma página, contido sempre nas tags de nome "a", veremos apenas o "texto clicável" (conteúdo), pois o hyperlink estará, na verdade, no atributo "href".  Veja o exemplo

```{r}
<a href = 'http://www.al.sp.gov.br/'> Vá o site da ALESP </a>
```

Tente, no próprio site da ALESP, clicar com o botão direito em um hyperlink qualquer (por exemplo, "TV Assembléia SP") para observar algo semelhante ao exemplo acima. Adiante vamos ver como atributos são extremamente úteis ao nosso propósito.

## Caminhos no XML e no HTML

O fato de haver hierarquia nos códigos XML e HTML nos permite construir "caminhos", como se fossem caminhos de pastas em um computador, dentro do código.

Por exemplo, o caminho das "tags" que contém a informação "nome" em nosso exemplo fictício é:

"/politicos/politico/nome". 

O caminho das "tags" que contém a informação "partido" em nosso exemplo fictício, por sua vez, é: 

"/politicos/politico/cargos/cargo/partido".

Seguindo tal caminho chegamos às três "tags" que contém a informação desejada.

Simples, não? Mas há um problema: o que fazer quando chegamos a 3 informações diferentes (o indivíduo  em nosso exemplo foi eleito duas vezes pelo PAN e uma pelo PRONA)? Há duas alternativa: a primeira, ficamos com as 3 informações armazenadas em um vetor, pois as 3 informações interessam. Isso ocorrerá com frequência.

Mas se quisermos apenas uma das informações, por exemplo, a de quando o indivíduo foi eleito deputado estadual? Podemos usar os atributos e os valores dos atributos das tag para construir o caminho. Neste caso, teríamos como caminho: 

"/politicos/politico/cargos/cargo[@tipo = 'deputado estadual']/partido"

Guarde bem este exemplo: ele será nosso modelo quando tentarmos capturar páginas.

Vamos supor que queremos poupar nosso trabalho e sabemos que as únicas "tags" com nome "partido" no nosso documento são aquelas que nos interessam (isso nunca é verdade em um documento HTML). Podemos simplificar nosso caminho de forma a identificar "todas as 'tags' '', não importa em qual nível hierárquico do documento". Neste caso, basta usar duas barras:

"//partido"

Ou "todas as tags 'partido' que sejam descendentes de 'politico', não importa em qual nível hierarquíco do documento": 

"/politicos/politico//partido"

Ou ainda "todas as tags 'partido' que sejam descendentes de quaisquer tag 'politico', não importa em qual nível hierarquíco do documento para qualquer uma das duas": 

"//politico//partido"

Ou "todas as 'tags' filhas de qualquer 'tag' 'cargo'" (usa-se um asterisco para indicar 'todas'):

"//cargo/*"

Observe o potencial dos "caminhos" para a captura de dados: podemos localizar em qualquer documento XML ou HTML uma informação usando a própria estrutura do documento. Não precisamos organizar o documento todo, basta extrair cirurgicamente o que queremos -- o que é a regra na raspagem de páginas de internet.

## Links na página de busca da ALESP

Vamos entrar na página principal da ALESP e fazer uma pesquisa simples na caixa de busca -- exatamente como faríamos em um jornal ou qualquer outro portal de internet (o processo seria o mesmo em buscadores como Google ou DuckDuckGo). Por exemplo, vamos pesquisar a palavra "merenda". A ferramenta de busca nos retorna uma lista de links que nos encaminharia para diversos documentos ou páginas da ALESP relacionadas ao termo buscado.

Nosso objetivo é construir um vetor com os links dos resultados. Em qualquer página de internet, links estão dentro da tag "a". Uma maneira equivocada de encontrar todos os links da página usando "caminhos em XML" seria:

"//a"

Entretanto, há diversos outros elementos "clicáveis" na página além dos links que nos interessam -- por exemplo, as barras laterais, o banner com os logotipos, os links do mapa do portal, etc. Precisamos, portanto, especificar bem o "caminho" para que obtivéssemos apenas os links que nos interessam.

Infelizmente não temos tempo para aprender aprofundadamente HTML, mas podemos usar lógica e intuição para obter caminhos unívocos. Neste exemplo, todos as "tags" "a" que nos interessam são filhas de alguma tag "li" (que é abreviação de "list" e indica um único elemento de uma lista). Podemos melhorar nosso caminho:

"//li/a"

A tag li, por sua vez, é filha da tag "ul" ("unordered list"), ou seja, é a tag que dá início à lista não ordenada formada pelos elementos "li". Novamente, melhoramos nosso caminho:

"//ul/li/a"

E se houver mais de uma "unordered list" na página? Observe que essa tag "ul" tem atributos: class="lista_navegacao". Algumas tem função para o usuário da página -- por exemplo, as tags "a" contém o atributo "href", que é o link do elemento "clicável". Mas, em geral, em uma página de internet os atributos não fazem nada além de identificar as tags para @ programador@. Diversos programas para construção de páginas criam atributos automaticamente. Por exemplo, se você fizer um blog em uma ferramenta qualquer de construção de blogs, sua página terá tags com atributos que você sequer escolheu.

As tags mais comum em páginas HTML são: head, body, div, p, a, table, tbody, td, ul, ol, li, etc. Os atributos mais comuns são: class, id, href (para links), src (para imagens), etc. Em qualquer tutorial básico de HTML você aprenderá sobre elas. Novamente, não precisamos aprender nada sobre HTML e suas tags. Apenas precisamos compreender sua estrutura e saber navegar nela.

Voltando ao nosso exemplo, se usarmos o atributo para especificar o "caminho" para os links teremos:

"//ul[@class='lista_navegacao']/li/a"

Poderíamos subir um nível hierárquico para melhorar o "caminho":

"//div[@id='lista_resultado']/ul[@class='lista_navegacao']/li/a"

Pegou o espírito? Vamos agora dar o nome correto a este "caminho": xPath.

Vamos agora voltar ao R e aprender a usar os caminhos para criar objetos no R.

## Capturar uma página e examinar sua estrutura

O primeiro passo na captura de uma página de internet é criar um objeto que contenha o código HTML da página. Para tanto, usamos a função "readLines" -- que é a mesma que utilizaríamos para qualquer documento de texto, Vamos usar a segunda página da busca pela palavra "merenda" na ALESP como exemplo:

```{r}
url <- "http://www.al.sp.gov.br/alesp/busca/?q=merenda&page=2"
pagina <- readLines(url)
```

Note que a estrutura do endereço URL é bastante simples: "q" é o parâmetro que informa o texto buscado e "page" o número da página. Nesta atividade vamos capturar apenas a página 2, mas você já aprendeu na atividade anterior como capturar todas as páginas usando loops.

```{r}
class(pagina)
```

Observe (usando a função "class") que o objeto "pagina" que contém o HTML da página em análise é da classe "character", ou seja, é um texto. Após usar a função "readLines" o R sabe apenas que um texto foi capturado, mas é incapaz de identificar a estrutura do documento -- tags, atributos, valores e conteúdos. Precisamos, pois, informar ao R que se trata de um documento XML. O verbo em inglês para esta tarefa se chama "parse", cuja definição é "to resolve (as a sentence) into component parts of speech and describe them grammatically". Em outras palavras, "parsear" é identificar a estrutura sintática de um objeto e dividí-lo em seus componentes.

Para páginas em HTML, usaremos a função "htmlParse":

```{r}
pagina <- htmlParse(pagina)
```

A partir de agora, sempre que capturarmos uma página, seja em HTML ou em XML (como são os RSSs), vamos fazer um "parse". Há 4 funções no pacote "XML" -- "xmlParse", "htmlParse", "xmlTreeParse" e "htmlTreeParse" -- bastante semelhantes entre si e cujo propósito é capturar um conteúdo em HTML ou XML e identificar sua estrutura, de forma a permitir a "navegação"
por meio dos nodes ou tags. Para compreender as diferenças entre as funções, vale a pena usar o help -- digite ?xmlParse na linha de comando.


Note que a classe do resultado da aplicação de uma das quatro funções não é mais um texto, mas um conjunto de objetos de classes específicas ("XMLInternalDocument" e "XMLAbstractDocument", neste caso) e que podem ser exploradas com as funções do pacote XML que veremos nesta atividade.

```{r}
class(pagina)
```

## Extraindo os conteúdos de uma página com a biblioteca XML

Vamos agora aprender a "navegar" um objeto XML dentro do R e extrair dele apenas o conteúdo que nos interessa.

Basicamente, trabalharemos nas atividades com um conjunto limitado funções: "getNodeSet", que extrai um pedaço (nodes) de um documento XML; "xmlValues", que extrai de um node o seu conteúdo; e "xmlGetAttr", que extrai os valores dos atributos especificados.

Vamos trabalhar com a ferramenta de busca da página inicial da ALESP, em particular com a página 2 de uma busca qualquer.

Em primeiro lugar, criamos um objeto com o endereço da página:

```{r}
url <- "http://www.al.sp.gov.br/alesp/busca/?q=merenda&page=2"
```

A seguir, capturamos seu código HTML com a função readLines:

```{r}
pagina <- readLines(url)
```

"Parseamos" o objeto de texto para transformá-lo em objeto XML:

```{r}
pagina <- htmlParse(pagina)
```

E, finalmente, usamos uma função nova chamada "xmlRoot" para eliminar quaisquer conteúdos (normalmente "sujeira") que estejam foram da tag de maior nível hierárquico no documento (o que é necessário com frequência em páginas de internet e é mais raro em documentos XML):

```{r}
pagina <- xmlRoot(pagina)
```

Pronto. Temos o conteúdo da página como documento XML pronto para ser "raspado".

Vamos começar usando a função "getNodeSet". Os nomes das funções do pacote XML são bastante literais. "getNodeSet" seleciona em um documento XML conjunto de nodes a partir de um "caminho". Por exemplo, podemos usar o xPath que criamos acima para pegar todos os links de resultado da busca:

```{r}
nodes_link <- getNodeSet(pagina, "//ul[@class='lista_navegacao']/li/a")
print(nodes_link)
```

Note que o resultado é uma lista que contém em cada posição o primeiro "node set". Ele poderia, inclusive, ter tags filhas, além de seu conteúdo e atributos. Vamos examinar melhor o primeiro node. Precisamos usar dois colchetes para indicar a posição, pois se trata de uma lista e não de um vetor.

```{r}
print(nodes_link[[1]])
```

Trata-se de uma tag "a", com dois atributos, "class" e "href", sendo que o valor deste último é o link para o qual seríamos direcionados ao clicar no conteúdo apresentado na página, que é o texto "Lei no. 10.945, de 26/10/2001 ( Lei 10945/2001 )". Chegamos rapidamente à informação que desejávamos, mas ainda não temos ela de forma adequadamente organizada.

O que nos interessa é extrair diretamente o valor dos atributos (se tiverem alguma informação valiosa) e o conteúdo. Vamos treinar com a primeira posição e extrair primeiro o conteúdo:

```{r}
conteudo_1 <- xmlValue(nodes_link[[1]])
print(conteudo_1)
```

E depois o valor do atributo "href":

```{r}
atributo_1 <- xmlGetAttr(nodes_link[[1]], name = "href")
print(atributo_1)
```

Excelente, não? Se quiséssemos apenas a informação do primeiro link resultante da busca, teríamos terminado nossa tarefa. Mas queremos os 10 links. Vamos ver duas maneiras ineficientes de construir um data frame que contenha uma variável com os conteúdos e a outra com os links. Tente decifrá-las:

Sem usar o "for loop":

```{r}
conteudo_1 <- xmlValue(nodes_link[[1]])
conteudo_2 <- xmlValue(nodes_link[[2]])
conteudo_3 <- xmlValue(nodes_link[[3]])
conteudo_4 <- xmlValue(nodes_link[[4]])
conteudo_5 <- xmlValue(nodes_link[[5]])
conteudo_6 <- xmlValue(nodes_link[[6]])
conteudo_7 <- xmlValue(nodes_link[[7]])
conteudo_8 <- xmlValue(nodes_link[[8]])
conteudo_9 <- xmlValue(nodes_link[[9]])
conteudo_10 <- xmlValue(nodes_link[[10]])

conteudos <- c(conteudo_1, conteudo_2, conteudo_3, conteudo_4, conteudo_5, conteudo_6, conteudo_7, conteudo_8, conteudo_9, conteudo_10)

atributo_1 <- xmlGetAttr(nodes_link[[1]], name = "href")
atributo_2 <- xmlGetAttr(nodes_link[[2]], name = "href")
atributo_3 <- xmlGetAttr(nodes_link[[3]], name = "href")
atributo_4 <- xmlGetAttr(nodes_link[[4]], name = "href")
atributo_5 <- xmlGetAttr(nodes_link[[5]], name = "href")
atributo_6 <- xmlGetAttr(nodes_link[[6]], name = "href")
atributo_7 <- xmlGetAttr(nodes_link[[7]], name = "href")
atributo_8 <- xmlGetAttr(nodes_link[[8]], name = "href")
atributo_9 <- xmlGetAttr(nodes_link[[9]], name = "href")
atributo_10 <- xmlGetAttr(nodes_link[[10]], name = "href")

atributos <- c(atributo_1, atributo_2, atributo_3, atributo_4, atributo_5, atributo_6, atributo_7, atributo_8, atributo_9, atributo_10)

dados <- data.frame(conteudos, atributos)
head(dados)
```

Usando for loops:

```{r}
conteudos <- c()
atributos <- c()
for (i in 1:10){
  
  conteudo_i <- xmlValue(nodes_link[[1]])
  conteudos <- c(conteudos, conteudo_i)
  
  atributo_i <- xmlGetAttr(nodes_link[[1]], name = "href")
  atributos <- c(atributos, atributo_i)
}

dados <- data.frame(conteudos, atributos)
head(dados)
```

Excelente! Temos um data frame após a raspagem.

Há, porém, uma forma mais rápida de resolver o problema: usando a função "xpathSApply". Basicamente, esta função serve para aplicar outra função, como "xmlValue" ou "xmlGetAttr", automaticamente para um conjunto de nodes, sem precisarmos fazer o que fizemos acima com o "for loop". Em outras palavras, é como se combinássemos a função "getNodeSet", com outra de nossa escolha e já aplicássemos a função a todo o resultado de "getNodeSet" em loop. Veja como funciona e note que a função "xmlValue" entra como argumento da função "xpathSApply":

```{r}
conteudos <- xpathSApply(pagina, "//ul[@class='lista_navegacao']/li/a", xmlValue)
print(conteudos)
```

Incrível, não? Veja como economizamos na "quantidade" de código para a tarefa de capturar os conteúdos. Vamos repetir o procedimento para os valores dos atributos, com o cuidado de nota que o argumento "name" necessário para a função "xmlGetAttr" deve vir como argumento também quando usamos a função "xpathSApply" após os demais:

```{r}
atributos <- xpathSApply(pagina, "//ul[@class='lista_navegacao']/li/a", xmlGetAttr, name = "href")
print(atributos)
```

Encerramos juntando ambos vetores em um data frame:

```{r}
dados <- data.frame(conteudos, atributos)
head(dados)
```

O desafio a partir de agora é conectar o que aprendemos da raspagem de dados de várias páginas com "for loop", mas usando a função "readHTMLTable", com as novas funções de raspagem de páginas que extraem com precisão o que conteúdo que nos interessa. Esta será nossa próxima atividade.


## Extraindo informação 

Agora que sabemos coletar de uma página da busca o link por meio da extração de atributos de um node e o título do resultado com a função "xmlValue", precisamos repetir o procedimento para todas as páginas de resultado.

Fizemos algo semelhante na tutorial 4, mas ainda usávamos a função "readHTMLTable". O objetivo é repetir o que fizemos lá, mas com as novas funções que vimos ao longo da tutorial 5.

Para isso, vamos usar o "for loop" do tutorial 4 para ir de uma página a outra.

O primeiro passo é, mais uma vez, ter o nosso link da pesquisa que queremos coletar armazenado em um objeto. 

```{r}
urlbase <- "http://www.al.sp.gov.br/alesp/busca/?q=merenda&page="
```

## Função Paste

Como é possível reparar, o número da página fica ao final do link, por isso podemos utilizar uma nova função chamada "paste" ou "colar" ao invés da função "gsub".

O que faremos é colar nosso contador (o "i", aquilo que vai mudar a cada vez que o loop realizar uma iteração) no final do link. Então o que queremos é uma combinação do nosso url com o contador sem nada separando os dois. A função "paste" é ideal para isso e funciona com os argumentos da seguinte maneira: primeiro texto a ser colado, segundo texto a ser colado, terceiro, ...,  último, e, finalmente, a especificação de qual é o separador que você deseja entre os textos (pode ser vazio "").

Na linguagem do R, escreveremos assim para o nosso caso:

```{r}
paste(urlbase, i, sep = "")
```

A "URL" é o endereço da página de busca, o "i" é o contador numérico do loop e o último argumento refere-se ao separador que igualamos a um par de aspas sem nada dentro, deixando claro que para funcionar corretamente nada, nem uma barra de espaço, deve ficar entre o endereço e o contador.

## Coletando o conteúdo e o atributo de todos os links

A lógica de coleta do atributo e do conteúdo de um node continua o mesma. A única diferença é que precisamos aplicar isso para todas as páginas. Agora que temos a URL construída, podemos montar um "for loop" que fará isso para nós.

No momento que essa atividade foi feita, a pesquisa pelo termo "merenda" tinha 162 páginas de resultado, o que implica que o nosso loop irá de 1, a primeira página, até a página 162, a última. Colocamos um "print(i)" para mostrar que página o loop está durante a execução.

```{r}
for (i in 1:162){
  print(i)
}
```

O que precisamos agora é incluir o que foi discutido ao longo tutorial. Para extrair as informações de uma página da internet precisamos examinar o código HTML, ler no R e transformar em um objeto XML.

```{r}
    pagina <- readLines(url)
    pagina <- htmlParse(pagina)
    pagina <- xmlRoot(pagina)
```

Sabemos também coletar de forma eficiente todos os títulos e links por meio do xmlValue e xmlGetAttr, respectivamente.

```{r}
     conteudos <- xpathSApply(pagina, "//ul[@class='lista_navegacao']/li/a", xmlValue)
    
    atributos <- xpathSApply(pagina, "//ul[@class='lista_navegacao']/li/a", xmlGetAttr, name = "href")
```

Falta empilhar o que produziremos em cada iteração do loop de uma forma que facilite a visualização. Usaremos a função "bind_rows" com data frames, pois para cada página agora, teremos 10 resultados em uma tabela com duas variáveis. O que queremos é a junção dos 10 resultados em cada uma das 162 páginas.

```{r}
dados <- bind_rows(dados, data.frame(conteudos, atributos))
```

Chegou o momento de colocar dentro loop tudo o que queremos que execute em cada uma das vezes que ele ocorrer. Ou seja, que imprima na tela a página que está executando, que a URL da página de resultados seja construída com a função paste, para todas elas o código HTML seja examinado, lido no R e transformado em objeto XML, colete todos os links e todos os títulos e que "empilhe". Lembrando que não podemos esquecer de definir a URK que estamos usando e criar um data frame vazio para colocar todos os links e títulos coletados antes de iniciar o loop.

```{r}
urlbase <- "http://www.al.sp.gov.br/alesp/busca/?q=merenda&page="

dados <- data.frame()

for (i in 1:162){
    print(i)
    url <- paste(urlbase, i, sep = "")
  
    pagina <- readLines(url)
    pagina <- htmlParse(pagina)
    pagina <- xmlRoot(pagina)
    
    conteudos <- xpathSApply(pagina, "//ul[@class='lista_navegacao']/li/a", xmlValue)
    
    atributos <- xpathSApply(pagina, "//ul[@class='lista_navegacao']/li/a", xmlGetAttr, name = "href")
    
    dados <- bind_rows(dados, data.frame(conteudos, atributos))
    
}
```

Pronto! Temos agora todos os títulos e links de todos os resultados do site da ALESP para a palavra "merenda" reunidos.

