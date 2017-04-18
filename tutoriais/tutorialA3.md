# Tutorial A3 - Webscrapping de um portais de notícias - DataFolha

Nos dois tutoriais anteriores com webscraping, utilizamos as ferramentas básicas de captura de dados disponíveis na biblioteca XML. Em primeiro, aprendemos a capturar várias páginas contendo tabelas em formato HTML de uma só vez. Depois, aprendemos como um documento XML está estruturado e que podemos a extrair com precisão os conteúdos de tags e os valores dos atributos das tags de páginas escritas em HTML. Neste último tutorial vamos colocar tudo em prática e construir um banco de dados de notícias. O nosso exemplo será o conjunto de notícias (556 na data da construção deste tutorial) publicadas sobre eleições no site do instituto de pesquisa DataFolha. Ainda que o DataFolha não seja um portal, por estar vinculado ao jornal Folha de São Paulo e ao portal UOL, a busca do DataFolha se assemelha muito às ferramentas de busca destes últimos.

Entre no link abaixo e veja como está estruturada a busca do DataFolha sobre eleições:

http://search.folha.uol.com.br/search?q=elei%E7%F5es&site=datafolha%2Feleicoes&sr=1&skin=datafolha&results_count=516&search_time=0.255&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Delei%25E7%25F5es%26site%3Ddatafolha%252Feleicoes%26sr%3D26%26skin%3Ddatafolha

## Raspando uma notícia no site DataFolha

Antes de começar, vamos chamar a biblioteca "XML" para tornar suas funções disponíveis em nossa sessão do R:

```{r}
library(XML)
```
Nossa primeira tarefa será escolher uma única notícia (a primeira da busca, por exemplo), e extrair dela 4 informações de interesse: o título da notícia; a data e hora da notícia; o link para a pesquisa completa em .pdf; e o texto da notícia.

O primeiro passo é criar um objeto com endereço URL da notícia e outro que contenha o código HTML da página:

```{r}
url <- "http://datafolha.folha.uol.com.br/eleicoes/2016/02/1744581-49-nao-votariam-em-lula.shtml"
pagina <- readLines(url)
```

Para o R, o conteúdo do objeto "pagina" nada mais é do que texto puro. O R não é capaz de discernir o que são tags, atributos, valores e conteúdo das tags. Precisamos identificar essa estrutura e fazemos isso usando a função "htmlParse"

```{r}
pagina <- htmlParse(pagina)
```

Pronto! Agora temos um objeto adequadamente identificado como XML pelo R. Pode ser que alguma "sujeira" tenha sobrado fora das tags que "envelopam" o código todo -- no caso as tags <html> e </html>, que abrem e fecham o conteúdo do arquivo XML. Usamos a função xmlRoot para "limpar" o arquivo:

```{r}
pagina <- xmlRoot(pagina)
```

Excelente. Com o objeto XML preparado e representando a página com a qual estamos trabalhando, vamos à caça das informações que queremos.

Volte para a página da notícia. Procure o título da notícia e examine-o, inspencionando o código clicando com o botão direito do mouse e selecionando "Inspecionar". Note o que encontramos:

```{r}
<h1 class="main_color main_title"><!--TITULO-->49% não votariam em Lula<!--/TITULO--></h1>
```

Tente sozinh@ e por aproximadamente 1~2 minutos construir i "xpath" (caminho em XML) que nos levaria a este elemento antes de avançar. (Tente sozinh@ antes de copiar a resposta abaixo!)

A resposta é: "//h1[@class = 'main_color main_title']"

Usando agora as funções "xpathSApply" e "xmlValue", como vimos no tutorial anterior, vamos capturar o título da notícia:

```{r}
titulo <- xpathSApply(pagina, "//h1[@class = 'main_color main_title']", xmlValue)
print(titulo)
```

Simples, não? Repita agora o mesmo procedimento para data e hora (tente sozinh@ antes de copiar a resposta abaixo!):

```{r}
datahora <- xpathSApply(pagina, "//time", xmlValue)
print(datahora)
```

E também para o link do .pdf disponibilizado pelo DataFolha com o conteúdo completo da pesquisa -- dica: o link é o valor do atributo "href" da tag "a" que encontramos ao inspecionar o botão para donwload:

```{r}
pesquisa <- xpathSApply(pagina, "//p[@class = 'stamp download']/a" , xmlGetAttr, name = "href")
print(pesquisa)
```

Finalmente, peguemos o texto. Note que o texto está dividido em vários parágrafos cujo conteúdo está inseridos em tags "p", todas filhas da tag "article". Se escolhemos o xpath sem especificar a tag "p" ao final, como abaixo, capturamos um monte de "sujeira", como os botões de twitter e facebook.

```{r}
texto <- xpathSApply(pagina, "//article[@class = 'news']" , xmlValue)
print(texto)
```

Por outro lado, se espificamos a tag "p" ao final do xpath, recebemos um vetor contendo cada um dos parágrafos do texto. Precisaríamos "juntar" (concatenar) todos os parágrafos para formar um texto único.

```{r}
texto <- xpathSApply(pagina, "//article[@class = 'news']/p" , xmlValue)
print(texto)
```
Por simplicidade, usaremos a primeira opção. Ao final, construímos um código ligeiramente mais complexo do que esperamos para a atividade que dá conta deste pequeno problema.

## Sua vez - tente raspar a notícia seguinte na busca do DataFolha

Tente agora raspar a notícia seguinte usando a mesma estratégia. É fundamental notar que variamos a notícia, mas as informações continuam tendo o mesmo caminho. Essa é a propriedade fundamental do portal raspado que nos permite obter todas as notícias sem nos preocuparmos em abrir uma por uma. O link para a próxima notícia está no objeto "url" abaixo:

```{r}
url<- "http://datafolha.folha.uol.com.br/eleicoes/2015/11/1701573-russomanno-larga-na-frente-em-disputa-pela-prefeitura-de-sp.shtml"
```

## Download de arquivos

Por vezes, queremos fazer donwload de um arquivo cujo link encontramos na página raspada. Por exemplo, no datafolha seria interessante obter o relatório em .pdf da pesquisa (para extrair seu conteúdo no futuro, por exemplo). Vamos ver como fazer download de um arquivo online.

Em primeiro lugar, obtemos seu endereço URL, como acabamos de fazer com a notícia que capturamos na busca do DataFolha (tente ler o código e veja se o entende por completo):

```{r}
library(XML)
url <- "http://datafolha.folha.uol.com.br/eleicoes/2016/02/1744581-49-nao-votariam-em-lula.shtml"
pagina <- readLines(url)
pagina <- htmlParse(pagina)
pagina <- xmlRoot(pagina)
pesquisa <- xpathSApply(pagina, "//p[@class = 'stamp download']/a" , xmlGetAttr, name = "href")
```

O link está no objeto "pesquisa":

```{r}
print(pesquisa)
```

Usando a função download.file, rapidamente salvamos o link no "working directory" (use "getwd()" para descobrir qual é o seu) e com o nome "pesquisa.pdf" (poderíamos salvar com o nome que quisessemos):

```{r}
getwd()
download.file(pesquisa, "pesquisa.pdf")
```

Vá ao "working directory" e veja o arquivo!

Sempre que estiver em posse de um conjunto de links que contém arquivos, você pode colocar a função "download.file" em loop e capturar todos os objetos ao mesmo tempo. Há uma dificuldade boba: nomear sem repetir os nomes diversos arquivos. Uma dica é usar o final do endereço URL como nome, mas você pode salvar os arquivos com nomes que sejam uma sequência numérica ou que provenham de um vetor que contenha os nomes todos. Use a criatividade!

Vamos voltar agora às notícias do DataFolha em HTML e ignorar o donwload de arquivos com os relatórios das pesquisas.

## Um código, duas etapas: raspando todas as notícias de eleições do DataFolha

Vamos fazer um breve roteiro do que precisamos fazer para criar um banco de dados que contenha todos os títulos, data e hora e texto de todas as notícias sobre eleições do DataFolha (Obs: por enquanto vamos ignorar os links de pesquisa, pois nem todas as notícias contêm os links e isso causa interrupção do código. Ao final, apresentamos um código que resolve tal problema).

## Etapa 1

* Passo 1: conhecer a página de busca (e compreender como podemos "passar" de uma página para outra)
* Passo 2: raspar (em loop!) as páginas de busca para obter todos os links de notícia

Esta é a primeira etapa da captura. Em primeiro lugar temos que buscar todos os URLs que contêm as notícias buscadas. Em outras palavras, começamos obtendo "em loop" os links das notícias e, só depois de termos os links, obtemos o conteúdo destes links. Nossos passos seguintes, portanto, são:

## Etapa 2

* Passo 3: conhecer a página da notícia (e ser capaz de obter nela as informações desejadas). Já fizemos isso acima!
* Passo 4: raspar (em um novo loop!) o conteúdo dos links capturados no Passo 2.

Vamos construir o código da primeira etapa da captura e, uma vez resolvida a primeira etapa, faremos o código da segunda.

## Código da etapa 1

Em primeiro lugar, vamos observar o URL da página de busca (poderíamos buscar termos chave, mas, neste caso, vamos pegar todas as notícias relacionadas a eleições).Na página 2 da busca vemos que o final é "sr=26". Na página 3 o final é "sr=51". Há um padrão: as buscas são realizadas de 25 em 25. De fato, a 21a. é última página da busca. Para "passarmos" de página em página, portanto, temos que ter um "loop" que conte não mais de 1 até 21, mas na seguinte sequência numérica: {1, 26, 51, 76, ..., 476, 501}.

Parece difícil, mas é extremamente simples. Veja o loop abaixo, que imprime a sequência desejada multiplicando (i - 1) por 25 e somando 1 ao final:

```{r}
for (i in 1:21){
  i <- (i - 1) * 25 + 1
  print(i)
}
```

Vamos, dessa forma, criar o objeto "url_base" a partir do URL da página 2 e substituir o número 26 em "sr=26" por um "place holder", "CONTADORLINK", por exemplo:

```{r}
url_base <- "http://search.folha.uol.com.br/search?q=elei%E7%F5es&site=datafolha%2Feleicoes&skin=datafolha&results_count=516&search_time=0.044&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Delei%25E7%25F5es%26site%3Ddatafolha%252Feleicoes%26skin%3Ddatafolha&sr=CONTADORLINK"
```

Capturar os links das notícias de uma única página é simples: examinamos o código HTML, lemos no R, transformamos em um objeto XML ("parse") procuramos o "xpath" que caracteriza os links e extraímos o valor do atríbuto "href". Este seria o Passo 1 descrito acima. Veja abaixo.

```{r}
pagina <- readLines(url)
pagina <- htmlParse(pagina)
pagina <- xmlRoot(pagina)
link <- xpathSApply(pagina, "//h2[@class = 'title']/a", xmlGetAttr, name = "href")
```

Combinando o que vimos até agora, podemos executar o Passo 2. Falta apenas criar antes do loop um vetor vazio -- por exemplo, o vetor "links_datafolha" no código abaixo -- que, a cada iteração do loop "guarda" os links raspados da página. Sua tarefa é gastar MUITOS minutos no código abaixo para entendê-lo na totalidade.

```{r}
links_datafolha <- c()
for (i in 1:21){
  print(i)
  i <- (i - 1) * 25 + 1
  url <- gsub("CONTADORLINK", i, url_base)
  pagina <- readLines(url)
  pagina <- htmlParse(pagina)
  pagina <- xmlRoot(pagina)
  link <- xpathSApply(pagina, "//h2[@class = 'title']/a", xmlGetAttr, name = "href")
  links_datafolha <- c(links_datafolha, link)
}
```

Temos, ao final, o objeto links_datafolha que contém todos os links para as notícias sobre eleições no DataFolha. Encerramos com sucesso a Etapa 1 -- caracterizada pelo primeiro loop. Esta etapa se assemelha bastante ao que fizemos nos tutoriais anteriores e o código deve ser compreensível para você a essa altura do campeonato. Vamos agora iniciar a etapa 2.

## Código da etapa 2

No começo da atividade resolvemos a captura do título, data e hora, link para o relatório de pesquisa completa e texto para uma única notícia no portal do instituto DataFolha. Nos resta agora capturar, em loop, o conteúdo de cada uma das páginas cujos links estão guardados no vetor "links_datafolha".

Vamos rever o procedimento, para uma URL qualquer, da captura do título, data e hora e texto (vamos deixar o link para o relatório de pesquisa de lado por enquanto, posto que algumas notícias não contêm o link e esta pequena ausência interromperia o funcionamento do código).

```{r}
pagina <- readLines("http://datafolha.folha.uol.com.br/eleicoes/2016/02/1744581-49-nao-votariam-em-lula.shtml")
pagina <- htmlParse(pagina)
pagina <- xmlRoot(pagina)
titulo <- xpathSApply(pagina, "//h1[@class = 'main_color main_title']", xmlValue)
datahora <- xpathSApply(pagina, "//time", xmlValue)
texto <- xpathSApply(pagina, "//article[@class = 'news']" , xmlValue)
```

Para fazermos a captura de todos os links em "loop" deve ter o seguinte aspecto, como se vê no código abaixo que imprime todos os 516 links cujo conteúdo queremos capturar. Note que a forma de utilizar o loop é ligeiramente diferente da que havíamos visto até então. No lugar de uma variável "i" que "percorre" um vetor numérico (1:21, por exemplo), temos uma variável "link" que recebe, a cada iteração, um endereço URL do vetor "links_datafolha", em ordem. Assim, na primeira iteração temos que "link" será igual "links_datafolha[1]", na segunda "links_datafolha[2]" e assim por diante até a última posição do vetor "links_datafolha" -- no nosso caso a posição 516.

```{r}
for (link in links_datafolha){
  print(link)
}
```

Combinando os dois código, e criando um data frame "dados" que é vazio antes do loop temos o código completo da captura. Tal como quando trabalhamos com tabelas, utilizando a função "bind_rows" para combinar o data frame que resultou da iteração anterior com a linha que combina o conteúdo armazenado em "titulo", "datahora" e "texto".

```{r}
dados <- data.frame()
for (link in links_datafolha){
  print(link)
  pagina <- readLines(link)
  pagina <- htmlParse(pagina)
  pagina <- xmlRoot(pagina)
  titulo <- xpathSApply(pagina, "//h1[@class = 'main_color main_title']", xmlValue)
  datahora <- xpathSApply(pagina, "//time", xmlValue)
  texto <- xpathSApply(pagina, "//article[@class = 'news']" , xmlValue)
  dados <- bind_rows(dados, data.frame(titulo, datahora, texto))
}
```

O resultado do código é um data frame ("dados") que contém 3 variáveis em suas colunas: "titulo", "datahora" e "texto". A partir de agora você poderia, por exemplo, usar as ferramentas presentes no pacote "tm" da linguagem R ("tm" é acronismo de "text mining") para criar uma nuvem de palavras ("wordcloud"), fazer a contagem de termos, examinar a semelhança da linguagem usada pelo instituto DataFolha com a usada por outros institutos de opinião pública, fazer análise de sentimentos, etc.

Antes disso, sua tarefa é a seguinte: executar ambas as etapas do código e comentá-lo por completo (use # para inserir linhas de comentário). Comentar o código alheio é uma excelente maneira de ver se você conseguiu compreendê-lo por completo e serve para você voltar ao código no futuro quando for usá-lo de modelo para seus próprios programas em R.

