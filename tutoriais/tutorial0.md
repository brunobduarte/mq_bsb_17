# Tutorial 0 - Dois exemplos para começar

## Bases de dados grandes

Programação e análise de dados costumam parecer algo assustador para leigos. Não é, e todo mundo é capaz de programar. Para exemplificar como funciona a linguagem R e "quebrar o gelo", vamos começar "pelo fim": abrindo uma base de dados grande e produzindo algumas tabelas simples.

Por enquanto, vamos apenas executar as tarefas sem prestar atenção aos detalhes da linguagem. Ainda assim, vocês verão que tarefas complexas pode ser executadas com poucas linhas de código.

## Exemplo 1 - Saques do Bols Família em janeiro de 2017 - Portal da Transparência

Nosso primeiro exemplo será a base de dados dos saques efetuados pelos beneficiários do Bolsa Família em janeiro de 2017. Em primeiro lugar, podemos ir ao [portal da transparência](http://www.portaldatransparencia.gov.br/downloads/mensal.asp?c=BolsaFamiliaSacado#exercicios2017) e fazer o download dos dados (ou pegá-lo diratemente no material do curso).

O arquivo de 2017 contém 1.6Gb. É um arquivo bastante grande, com 14 variáveis (colunas) e mais de 12 milhões de linhas. Arquivos grandes podem ser abertos no R usando a função _fread_, disponível no pacote _data.table_. Num futuro breve veremos o que são funções e pacotes. Por enquanto, basta saber que tornamos as funções de um pacote disponíveis se importarmos o pacote para nossa biblioteca usando a função _library_.

```{r}
#install.packages(data.table), caso o pacote data.table ainda nao tenha sido instalado
#install.packages(dplyr), caso o pacote data.table ainda nao tenha sido instalado
library(data.table)
library(dplyr)
```

Vamos agora abrir os dados. Nota importante: você deve inserir o caminho do arquivo em seu computador dentro das aspas.

```{r}
saques <- fread("/home/acsa/Downloads/201701_BolsaFamiliaSacado.csv", encoding = "Latin-1")
```

Simples, não? Vamos ver o resultado da importação com a função _head_, que retorna as 6 primeiras linhas do banco de dados:

```{r}
head(saques)
```

Para checar quantas linhas e colunas há nos dados importados, usamos a função _dim_:

```{r}
dim(saques)
```

E observar os nomes das variáveis:

```{r}
names(saques)
```

Nomes com espaços, acentos e caracteres especiais são bastante ruins de se trabalhar. Além disso, há um problema nesse banco de dados: a coluna "Valor Parcela", que contém os valores sacados, não foi lida automaticamente como número. Assim, vamos renomear algumas variáveis (NIS, UF, Mês do saque e código do município, por exemplo), e vamos construir uma nova variável "valor", que contenha números e não números interpretados como texto:


```{r}
saques <- saques %>% 
  rename(nis = `NIS Favorecido`, uf = UF, munic = `Nome Município`, mes = `Mês Referência Parcela`) %>%
  mutate(valor = as.numeric(gsub(",", "", saques$`Valor Parcela`)))
```

Se você prestar atenção no código acima, verá que ele é mais simples do que parece.

Agora que as variáveis têm nomes mais curtos e fáceis de trabalhar e a coluna "valor" foi construída, podemos explorar os dados. Vamos fazer uma tabela de contagem de saques por UF, um histograma com a distribuição dos valores sacados e uma tabela com a soma dos valores por UF.

Primeiro, a tabela que contém a contagem por UF de todos os beneficiários que sacaram em janeiro de 2017:
  
```{r}
saques %>% group_by(uf) %>% summarise(contagem = n())
```

Vejamos agora a distribuição de valores sacados em janeiro de 2017 em um histograma:
  
```{r}
hist(saques$valor, main = "Distribuição dos valores sacados em jan/2017", xlab = "R$", ylab = "Frequência")
```

Legal, não?

Finalmente, vamos fazer o somatório dos valores sacado por UF em 2017:
  
```{r}
saques %>% group_by(uf) %>% summarise(valores = sum(valor))
```

Note que com poucas linhas de código processamos um bocado de dados e geramos informações úteis para a compreensão do programa bolsa família. Em um futuro breve, veremos como organizar, manipular e "misturar" bases de dados volumosas e extrair informações delas.

## Exemplo 2 - Raspando dados do Portal da Transparência

Uma das grandes virtudes da linguagem R é a variedade de aplicações e ferramentas disponíveis. Raspar dados da internet com R é algo relativamente simples. Vejamos o exemplo do próprio Portal da Transparência.

Há alguns anos, não havia a possibilidade de fazer o download de grandes bases de dados do portal. Era necessário, portanto, consultar as tabelas do portal, copiar uma a uma e construir uma base de dados.

Vamos supor que queremos consultar os saques realizados em 2016 no município de Bom Jesus do Norte, no Espírito Santo (realizar a consulta no portal ou clicar em http://www.portaldatransparencia.gov.br/PortalTransparenciaPesquisaAcaoFavorecido.asp?Exercicio=2016&textoPesquisa=&textoPesquisaAcao=&codigoAcao=8442&codigoFuncao=08&siglaEstado=ES&codigoMunicipio=5621&Pagina=1)

Puxa! Mas um município tão pequeno tem 44 páginas de consulta, cada uma com uma tabela!

Para nossa sorte, todas as páginas e tabelas têm o mesmo formato. Se soubermos capturar uma tabela, saberemos capturar quantas quisermos.

Usando a bilbioteca _XML_ e a função _readHTMLTables, vamos capturar a primeira a tabela da página 1. Começamos gravando o endereço URL em um objeto e, a seguir, capturamos a tabela.

```{r}
library(XML)
url <- "http://www.portaldatransparencia.gov.br/PortalTransparenciaPesquisaAcaoFavorecido.asp?Exercicio=2016&textoPesquisa=&textoPesquisaAcao=&codigoAcao=8442&codigoFuncao=08&siglaEstado=ES&codigoMunicipio=5621&Pagina=1"
readHTMLTable(url)[[2]]
```

Simples, não. Note agora que o endereço URL contém a informação sobre o número da página que estamos capturando exatamente no final. Agora, vamos pedir para o R variar esse número de 1 a 44, para gerarmos todos o endereço das 44 páginas que queremos capturar. Veja como:

```{r}
baseurl <- "http://www.portaldatransparencia.gov.br/PortalTransparenciaPesquisaAcaoFavorecido.asp?Exercicio=2016&textoPesquisa=&textoPesquisaAcao=&codigoAcao=8442&codigoFuncao=08&siglaEstado=ES&codigoMunicipio=5621&Pagina="
for (i in 1:44) {
  url <- paste(baseurl, i, sep = "")
  print(url)
}
```

Utilizamos acima uma das ferramentas mais poderosas em programação: _loops_. Traduzindo o código acima para o português: "para cada i de 1 a 44, insira i no final do endereço base (baseurl) para criar a url que queremos capturar".

Se o código acima parece obscuro para você, não se preocupe. Veremos neste curso como construí-lo passo a passo.

Finalmente, vamos combinar os dois pedaços de código (e mais um detalhes), para criar uma base de dados chamada _bf_bom_jesus_ com os dados de transferência do Programa Bolsa Família aos cidadãos de Bom Jesus do Norte - ES em 2016.

```{r}
baseurl <- "http://www.portaldatransparencia.gov.br/PortalTransparenciaPesquisaAcaoFavorecido.asp?Exercicio=2016&textoPesquisa=&textoPesquisaAcao=&codigoAcao=8442&codigoFuncao=08&siglaEstado=ES&codigoMunicipio=5621&Pagina="
bf_bom_jesus <- data.frame()
for (i in 1:44) {
  url <- paste(baseurl, i, sep = "")
  bf_bom_jesus <- bind_rows(bf_bom_jesus, readHTMLTable(url)[[2]])
}
```

Com apenas 6 linhas de código, "hackeamos" o portal da transparência e obtemos dados que, no passado, não estavam disponíveis em formato aberto.


## Exemplos e o curso

Esses exemplos ilustram o que aprenderemos no curso. Eles representam bem 2 etapas do processo de trabalho de "cientistas de dados": (1) a organização e manipulação de bases de dados; e (2) a captura de dados. Neste curso, cujos tutoriais seguem nesta apostila, trabalharemos estes dois tópicos que, ao fim e ao cabo, condensam os elementos do programa docurso.
