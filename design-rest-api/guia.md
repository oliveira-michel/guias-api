
# Guia de Design REST

## Abstract

TODO

## Status of this Document

TODO


## Table of Content

<!-- TOC -->
- [Introdução](#introdução-e-conceitos-básicos)
- [Request](#)
	- [URL](#)
		- [Base](#) 
		- [Resources](#)
			- [Domínios Funcionais](#domínios-funcionais)
			- [URI Parameters](#uri-parameters)
		- [Query String](#)
			- [Paginação](#)
				- [Range](#ranges)
				- [Page e Page Size](#)
				- [Offset e Limit](#)
			- [Filtros](#)
			- [Ordenação](#)
	- [Headers](#)
	- [Verbs](#)
	- [Body](#)
- [Response](#response)
 	- [Headers](#)
	 	- [Throttling, Rate Limiting e Quotas](#)
	- [Body](#)
		- [Paginação](#)
			- [Cursor](#)
			- [Page e Page Size](#)
			- [Offset e Limit](#)
		- [Ordenação](#)
		- [Fields](#)
	- [HTTP Status Codes](#)
- [Segurança](#)
- [Versionamento](#)
- [Performance, Cache e compressão](#)
<!-- /TOC -->

## Introdução e conceitos básicos

Os conteúdos abaixo cobrem alguns conceitos aliados às boas práticas no design de RESTful APIs. Quando falamos em design de RESTful APIs, estamos abordando como definir um contrato de API que expõe as entidades e funções de um determinado sistema respeitando as restrições do REST.

> Para fazer o entendimento da necessidade do negócio e definição das entidades, recomendamos o uso de Domain Driven Design. 
> *Em breve será disponibilizado um guia explorando toda a fase de entendimento do negócio e transformação dos conceitos de negócio em entidades para serem usadas na esposição delas via API.*

> O conteúdo deste material contempla os conceitos para especificação do contrato de forma genérica, não abordando necessariamente nenhuma linguagem específica de definição de contrato REST, como Swagger, RAML, Open API, etc.
> *Em breve será disponibilizado um guia explorando a declaração do contrato em uma destas linguagens.*

**API** (Application Programming Interface) é um software que permite comunicação entre sistemas onde há um fornecedor e um ou mais consumidores de informação, serviços, etc. Para consumir uma API respeita-se um contrato que deve incluir o protocolo de comunicação, as operações (consultas e atualizações) e os formatos de dados para entradas, saídas e erros.

A maioria dos softwares nos quais nós interagimos são construídos para atender às necessidades humanas e normalmente referenciam as "coisas" reais das quais esses softwares tratam. Por exemplo, um software de biblioteca vai representar livros, seções, autores, etc. e, através de alguma interface, permitir que um usuário interaja com estas representações. Uma API difere-se deste tipo de software no que tange o usuário: quem interage com o software é outro software, não diretamente o usuário final. No entanto, quem desenvolve o software que interage com a API é um programador - e até a data deste documento, a maioria ainda são humanos - e para que a experiência deste usuário programador e do usuário do software que ele desenvolve seja a melhor possível, princípios de design devem ser respeitados. 

**REST API** (Representational State Transfer Application Application Programming Interface) é um estilo de arquitetura que define um conjunto de restrições e propriedades baseadas em HTTP fornecendo interoperabilidade entre sistemas de computadores na internet (ou rede local). REST permite que os sistemas consumidores acessem e manipulem o estado de representações textuais de recursos usando um conjunto pré definido de operações. As "coisas" da vida real em REST se chamam recursos. O conjunto de valores dos atributos de um determinado recurso em um determinado momento, se chama estado.
Por colocar mais restrições do que WebServices - que é uma outra forma de integrar sistemas - e utilizar mais recursos do HTTP do que WebServices, acabou padronizando melhor a forma de comunicação e isso agilizou o desenvolvimento das integrações.

O REST enfatiza simplicidade, extensibilidade, confiabilidade e performance: 
 
- **Simplicidade** porque a forma como se interage com REST APIs é bem definida e de certa forma restritiva e a interação é stateless, ou seja, todos as informações necessárias para o processamento de uma requisição (no REST, alteração do estado de um recurso) deve estar contida naquela requisição. 
- **Extensibilidade** porque um determinado recurso pode ser representado em diferentes formatos e até mesmo versões, além da possibilidade de se adicionar novos recursos sem precisar alterar os já existentes. 
- **Confiabilidade** porque existe uma separação clara das ações que podem ser feitas e dos seus efeitos colaterais.
- **Performance** porque faz parte dos principais pilares do REST o uso de cache através de uma semântica bem definida, além de uma arquitetura baseada em separação de camadas, permitindo que partes diferentes do sistema possam ser escaladas de forma independente e isolada. 


O **HTTP** (HyperText Transfer Protocol) - ou HTTPS na sua variante com encriptação - é o protocolo da internet, no qual acontece a grande maioria da comunicação. Basicamente, ele trata requisições entre um cliente e um servidor através de uma mensagem que contém um Header e um Body. Se fizer uma analogia a uma carta, eu tenho informações de entrega no envelope (header) e um conteúdo (carta dentro do envelope). E o ciclo de vida contempla um envio (request que parte do cliente) e um retorno (response que parte do servidor). Na maioria das implementações, a comunicação no é stateless, ou seja, ao término do ciclo "request/response", as conexões com o servidor são encerradas. Além da mensagem, o HTTP define verbos para tratar as mensagens, códigos de retorno para indicar sucesso ou insucesso no processamento, padrões de endereço do servidor, entre outros.

O REST surgiu à partir da dissertação de Roy Thomas Fielding - um dos criadores da web - e se baseia fortemente no HTTP reaproveitando muitos conceitos deste protocolo. A dissertação define de forma aberta o conceito e o mercado foi refinando e definindo convenções. Dessa forma, em alguns pontos, existem várias formas de fazer a mesma coisa e não há necessariamente a mais correta. Assim, cabe a você analisar os *trade-offs* de cada forma e escolher a que melhor atende às necessidades dos seus software e principalmente do cliente consumidor do seu software.

O ponto mais importante quando se fala em boas práticas é que seja adotado um padrão, pois à partir do momento que se tem um padrão, a curva de aprendizado para novas soluções é mais rápida e podem ser utilizados frameworks e automações para reduzir o tempo de desenvolvimento dos sistemas.

Neste guia, busco compartilhar os padrões que implemento nas empresas onde já trabalhei e que costumo adotá-los quando os vejo em livros, artigos ou até mesmo APIs de mercado e entendo fazer sentido para a empresa.
 
## Request

Assim como no protocolo HTTP, um cliente (consumidor da API) deve enviar uma mensagem dentro de um determinado padrão a um endereço (onde o servidor responde àquele padrão de mensagem) e aguardar a resposta.

Na requisição, existem alguns padrões a serem seguidos e eles serão explicados um a um a seguir. Alguns são obrigatórios para o funcionamento da REST API, outros são boas práticas que adotadas pelo mercado.

### Request > URL
---
A URL (Universal Resource Location) é endereço onde o cliente vai fazer a requisição. Cada URL identifica um recurso diferente na API. Como no navegador, quando digitamos um endereço de um site e ele nos responde com a página, no REST é enviada uma solicitação para o endereço e ele nos responde com informações sobre o recurso.

A URL é formada por basicamente 3 partes Base Path, Resources ou Path e Query Strings. Os capítulos a seguir detalharão cada uma delas.

Dado que URLs são case sensitive, é uma boa prática usar tudo em minúsculo para evitar possibilidade de erro na digitação (o que pode ser difícil de identificar). Veja:
- I <- isto é um i maiúsculo
- l <- isto é um L minúsculo

Quando na URL existirem palavras compostas, é indicado o uso de hífen para separá-las. Nos interpretadores de texto, o hífen é um separador de palavras, ao contrário do underscore e de CamelCase). Por exemplo, dê um clique duplo nas palavras abaixo e veja como apenas no caso do hífen é feita a seleção correta:
- CamelCase: primeiraSegunda 
- Underscore: primeira_segunda
- Hífen: primeira-segunda

> Mais informações: 
> [https://support.google.com/webmasters/answer/76329](https://support.google.com/webmasters/answer/76329)
> [https://stackoverflow.com/questions/10302179/hyphen-underscore-or-camelcase-as-word-delimiter-in-uris](https://stackoverflow.com/questions/10302179/hyphen-underscore-or-camelcase-as-word-delimiter-in-uris)

### Request > URL > Base
---
O Base Path é a parte inicial da URL, nela você tem o protocolo (http:// ou https://, por exemplo) e o endereço do servidor na web. O Base Path se repetirá em todas as requisições.
Ex:
- https://maps.googleapis.com
- https://management.azure.com
- https://api.spotify.com

Algumas empresas têm URLs que representam os produtos da empresa, em outras a próprie empresa é um produto. Nestes cenários, normalmente encontramos Base Paths com estruturas semelhantes a essas:
- https://api.empresaexemplo.com
- https://api.produtoexemplo.com

Em outros cenários, não usam o termo "api" como nos exemplos do google ou azure (acima).
Mas o ponto principal é que normalmente é utilizada uma URL diferente da URL do site da empresa (http://www.empresaexemplo.com).

Identificar ambientes de homologação e produção no Base Path também é interessante, pois reduz a configuração do ambiente a apenas um parâmetro. Ex:
- Homologação: https://api-hml.empresaexemplo.com
- Desenvolvimento: https://api-dev.empresaexemplo.com
- Sandbox: https://api-sbx.empresaexemplo.com
- Produção: https://api.empresaexemplo.com

### Request > URL > Resources
---
No REST, a letra R significa "Representação ". Representação é a forma de apresentar os recursos ("coisas") que existem no sistema. Recursos no REST são batizados usando substantivos. Eles são as entidades que serão expostas para outros sistemas. Cada um desses recursos têm um endereço (URL) diferente e são representados por textos separados por barras após o Base Path. É muito importante que os nomes dos recursos sejam estruturados de forma que cumpram uma hierarquia que proporcione sentido próprio à URL. Os nomes escolhidos para os recursos devem ser de fácil entendimento, de modo que ao ler a URL se obtenha rapidamente a informação sobre qual recurso ela representa.

Ex:
- Endereço da representação de um recurso chamado "artists" no Spotify: https://api.spotify.com/v1/artists/
- Endereço da representação de um recurso chamado "albums" do Spotify: https://api.spotify.com/v1/albums 
- Endereço da representação de um recurso específico chamado dentro dos "albums" do Spotify: https://api.spotify.com/v1/albums/{id}
- Endereço da representação de um recurso chamado "tracks" de um "album" específico do Spotify: https://api.spotify.com/v1/albums/{id}/tracks

Repare que existe um Base Path se repetindo em cada um dos endereços (https://api.spotify.com) e um "/v1" também. Ele representa a versão da API e também é exposto como um recurso. Há mais informações sobre versionamento no capítulo [Versionamento](#versionamento).

Observe que algumas URLs têm recursos como artists ou albums (sem o {id} na frente) que deverão retonar listas, pois não se especifica nenhum "artist" ou "album" através de um {id}.
Outras URLs têm o {id} e retornação apenas um item cujo id seja o id definido na URL. Este {id} é um [URI Parameter](#uri-parameters).

É boa prática, usar nomes em minúsculo e se composto, separar com hífen.

Os recursos na maioria dos casos serão substantivos no plural, pois geralmente definem coleções (ex: cartões, usuários, clientes, carros, ruas, etc.) Ao pensar no nome do recurso, é importante verificar se o nome não conflitará com alguma possível implementação parecida no futuro. Por exemplo, imagina um recurso que se chamaria ofertas_credito. Neste caso, a deve-se perguntar: "este serviço ofertará todos os tipos de crédito no domínio de crédito?" Se a reposta for: "Não, apenas consignado", deve-se especificar o nome da entidade como ofertas-credito-consignado porque no futuro uma nova rota que oferte todos os tipos de crédito poderá ser criada com o nome oferta-credito.
Ex:
- http://api.exmpresaexemplo.com/creditos/v1/ofertas-credito-consignado

Recursos criados no singular são menos frequentes e pode acontecer quando:
-	Haver requerimentos de segurança ou permissão específicos para um atributo. Neste caso, o atributo se torna um recurso na URL, podendo aplicar policies específicas de segurança naquela rota.
-	Há um atributo complexo com um número elevado de "sub-atributos".
-	Há recursos que não existam no plural, como saldo, extrato, etc.

Quando definir os recursos a serem expostos, deve-se **evitar**:
- Utilizar termos que não façam parte do nome da entidade de negócio, por exemplo: <s>detalhes_</s>lancamentos-cheque. A entidade de negócio se chama lancamentos_cheque, exibir todos os atributos ou não, fica a cargo de filtros na [query string](#query-string), não na exposição como um recurso.
- Utilizar termos que representam as ações básicas do CRUD (consultar, gravar, atualizar, apagar, etc). Por exemplo: <s>consultar-</s>fatura. As ações básicas do CRUD são representadas pelos verbos HTTP (GET, POST, PUT, PATCH e DELETE).
- Expor representações de detalhes do backend na entidade, por exemplo: <s>servico-</s>transferencias. O termo "servico" neste caso está representando, por exemplo, o serviço do sistema que processa uma transferência. Este tipo de informação deve ser abstraída no nome da entidade.

### Request > URL > Resources > Funções que não são CRUD
---
Uma das restrições do REST é que ele se aplica a recursos que representam as "entidades" de um [Domínio Funcional](#). Para alterar o estado dessas entidades, usamos os [verbos](#) GET, POST, PATCH, PUT e DELETE, que basicamente fazem o CRUD.
 
No entanto, existem alguns cenários em que não temos "entidades" de um domínio, mas sim, ações e que não fazem parte do conjunto restrito de verbos do HTTP. Aqui estão alguns exemplos:
- Distância entre dois pontos (via coordenadas de GPS)
- Validação de dados de cartão de crédito
- Cálculos em geral

Para estes casos, tratamos essas funções como recursos e para facilitar a identificação de que não é uma "entidade", usamos verbos no lugar dos substantivos.
Ex:
- GET .../**calcular-distancia**?from-latitude=48,8584&from-longitude=2,2945&to-latitude=-22.951916&to-longitude=-43.2104872
- POST .../**validar-cartao**
```
{
	"numero": "5346931596124410",
	"nome-titular": "JOSE A OLIVEIRA",
	"data-validade": "2024-05-23",
	"CVV": 996
}
```
- GET .../**somar**?primeiro-valor=72&segundo-valor=28
  
Observem que algumas chamadas representam consultas seguras e idempotentes. Portanto, o verbo HTTP que foi utilizado foi o GET.
No exemplo da validação de cartão, trafegamos **dados sensíveis**. Mesmo sendo uma consulta segura e idempotente, por questões de segurança, o tráfego de dados sensíveis são feitos sempre via POST para permitir a encriptação do Body.

E há casos em que a chamada não é segura, nem idempotente. Por exemplo, em uma API de login, onde depois de algumas tentativas o login será bloqueado. Nestes casos devemos usar verbos como o POST. 

Para saber mais sobre idempotência, leia sobre os [verbos HTTP](#) .

###  Request > URL > Resources > Domínios Funcionais
---
Em algumas empresas, pode ser que não exista um Base Path para cada produto e uma mesma empresa forneça Recursos de vários produtos diferentes através de API. Neste caso, pode-se criar um recurso que serve para agrupar os recursos de cada produto.
Ex:
- https://apis.bbva.com/customers/v1
- https://apis.bbva.com/accounts/v1
- https://apis.bbva.com/cards/v2
- https://apis.bbva.com/payments/v1
- https://apis.bbva.com/loans/v1
- https://apis.bbva.com/notifications/v1

Nos caso acima, o Banco BBVA agrupa todos os recursos referentes ao assunto "cards" sempre abaixo do recurso "cards" e assim por diante. Ex:
- https://apis.bbva.com/cards/v2/cards
- https://apis.bbva.com/cards/v2/cards/{cardid}
- https://apis.bbva.com/cards/v2/cards/{cardid}/transactions

> Mais detalhes sobre essas APIs: https://www.bbvaapimarket.com/products

A mesma abordagem de agrupamento provavelmente será útil para APIs expostas apenas internamente nas empresas para integrações entre diferentes sistemas, pois definir um recurso na URL envolve menos mudanças na implantação do que criar um Base Path para cada um dos produtos/assuntos da empresa. 

Em alguns lugares esses grandes assuntos são conhecidos como domínios funcionais. Eles agrupam entidades relacionadas por um contexto funcional como produtos (cartões, contas, etc.), processos (contratações, on-boarding, etc.) ou serviços (comunicações, chat, etc.). É boa prática, usar nomes em minúsculo e se composto, separar com hífen. Os domínios funcionais na maioria dos casos serão substantivos no plural.

Também é interessante manter os contratos de cada domínio separados para dar independência para os times que cuidam de cada um deles, além de gerir o ciclo de vida de cada um deles de forma separada.

###  Request > URL > Resources > Path Parameters
---
Nos exemplos acima, alguns dos recursos estavam com a notação {}. Eles são Path Parameteres, ou seja, são parâmetros que podem variar conforme a consulta. Por exemplo, na API do Spotify https://api.spotify.com/v1/albums/{id}/tracks, no lugar do {id}, em tempo de execução, o sistema deverá colocar o Id de um álbum específico para poder listar as faixas do respectivo álbum. 
Ex:
- https://api.spotify.com/v1/albums/0sNOF9WDwhWunNAHPD3Baj/tracks
- https://api.spotify.com/v1/albums/2kLIU5SVDjnEUndjks72ddP/tracks

Os path parameters são destinados exclusivamente para identificar os recursos dentro das coleções mediante seu identificador único.

É boa prática, usar nomes em minúsculo e se composto, separar com hífen. Os URI Parameters na maioria dos casos serão compostos por "id-" + nome do recurso no singular.
Ex:
- http://api.exemploempresa.com/cartoes/v1/cartoes/{id-cartao}/faturas/{id-fatura}/lancamentos/{id-lancamento}

Desta forma, quando a aplicação recuperar a coleção de parâmetros, terá a seguinte lista:
*id-cartao*, *id-fatura* e *id-lancamento*, cada uma com um nome que a identifica unicamente na coleção.

###  Request > URL > Query Strings
---
Além dos [URI Parameters](#uri-parameters), Query Strings também permite passar parâmetros na URL da chamada. Query Strings são usadas para filtrar resultados em consultas. São definidas após os [Resources](#resources), iniciando com uma **?** e separando cada par de chave e valor  (**chave=valor**) com um **&**:

***.../...?chave1=valor1&chave2=valor2&chaveA=valorA***

Assim, uma API que busque restaurantes de comida japonesa na cidade de são paulo, teria uma URL semelhante a esta:
```
https://api.empresaturismo.com.br/locais/v1/restaurantes?tipo-comida=japonesa&cidade=S%C3%A3o%20Paulo
```
Os valores devem sempre respeitar os caracteres permitidos em URLs. Mais informações, veja [W3Schools URL Encode](https://www.w3schools.com/tags/ref_urlencode.asp).

É importante convencionar o comportamento da API conforme o tipo de filtro que é feito:

| Tipo de dado | Operação | Descrição |
| ------------ | -------- | --------- |
|Numérico, Data e Booleano|Igualdade|Retorna aqueles recursos cujo valor do atributo tenha exatamente o valor especificado. Ex: .../?quantidade=5 devolverá aqueles recursos cujo atributo "quantidade" tenha o valor 5.|
|Numérico, Data e Booleano|Ou|Retorna aqueles recursos cujo valor do atributo esteja contido em uma lista de valores. Ex: .../?quantidade=5&quantidade=9&quantidade=12 retornará aqueles recursos cujo atributo quantidade seja 5, 9 ou 12.|
|Numérico e Data|Maior ou Igual|Retorna aqueles recursos cujo valor do atributo seja maior ou igual o valor definido em “from-quantidade” e menor ou igual o valor definido em "to-quantidade". Ex: .../?from_quantidade=5 retornará os recursos com quantidade maior ou igual a 5. Consulte também [Ranges](#ranges).|
|Texto|Contém|Retorna aqueles recursos cujo valor do atributo contenha o valor especificado. Ex: …?nome=Frederico retornará aqueles recursos que contenham “Frederico” no atributo nome. São retornos válidos válidos: “Frederico Garcia”, “Don Frederico”, “Frederico”.|
|Texto|Ou Contém|Retorna aqueles recursos cujo valor do atributo contenha um dos valores especificados. Ex: nome=Frederico&nome=Antonio retornará aqueles recursos que contenham “Frederico” ou "Antonio" no atributo nome. São retornos válidos válidos: “Frederico Antonio”, “Don Frederico”, “Frederico”, “Antonio Ramirez”.|

De forma resumida, o operador **&** que separa as query strings é um **Ou** para os valores de um mesmo atributo e um **E** entre atributos diferentes.

Geralmente, query strings não são utilizadas nos casos em que se busca um recurso cujo ID que já está definido na URL via Path Parameter. Isto porque, normalmente as query strings são utilizadas para filtrar dentro de uma coleção de resultados. Quando se tem o ID definido, não temos uma coleção de resultados, mas um específico já especificado pelo cliente.

As query string não são utilizadas somente para filtros, ela pode ser utilizada como parâmetros para paginação, versionamento, ordenação, etc. 

### Request > URL > Query Strings > Paginação
---
Quando uma API retorna uma lista de resultados, pode ser que esses resultados cheguem a dezenas ou milhares de registros. Na maioria das vezes, as telas das aplicações não exibem todos ao mesmo tempo e quando se trata de API, espera-se respostas rápidas. Por conta disso, é uma boa prática dividir os resultados em blocos. Este processo chamamos de paginação.

Paginando as respostas, a mensagem trafegada fica menor, e por isso, o cliente recebe o resultado da requisição muito mais rápido, além de permitir um uso mais racional e otimizado dos recursos do servidor.

Existem algumas formas de definir exatamente qual "bloco de informação" consultar. Estas formas estão descritas à seguir.

### Request > URL > Query Strings > Paginação > Range
---
Podemos delimitar a quantidade de resultados à partir da filtragem de um determinado parâmetro, por exemplo, se o parâmetro for **data-nascimento**, a chamada à uma API ficaria assim:
```
http://api.empresarh.com/candidatos?from-data-nascimento=1985-01-01&to-data-nascimento=2001-12-31
```
Para atuar como um cursor e filtrar um range de IDs, a chamada ficaria assim:
```
http://api.empresarh.com/candidatos?from-id-candidato=1000&to-id-candidato=1099
```
É uma boa prática adotar padrões para definir a estrutura do parâmetro que trata ranges, como sugestão:
- **"from-" + nome-do-atributo**
- **"to-" + nome-do-atributo**

### Request > URL > Query Strings > Paginação > Page e Page Size
---
A paginação baseada em  **page**  e  **page-size,**  como o próprio nome já diz, é utilizada através dos parâmetros de número da página a ser navegada e o seu respectivo tamanho (em número de registros).

Ambos são opcionais e caso não sejam definidos na URL, é esperado que a API retorne todos os registros ou retorne na página e tamanho padrão dela.
Ex:
```
https://api.classificados/veiculos?page=3&page-size=30
```
### Request > URL > Query Strings > Paginação > Limit
---
O  **limit** permite limitar a quantidade de registros que a API retorna.

Ex:
https://api.lojaexemplo.com/ofertas-noturnas/?limit=10

### Request > URL > Query Strings > Paginação > Ordenação
---
Em APIs que retornem conjuntos de registros, é interessante permitir alguma ordenação básica.
A ordenação pode ser especificada através das query strings:
-	**order-by**: especifica o atributo pelo qual de deseja ordenar.
-	**order**: especifica o tipo de ordem ascendente ou descendente mediante as palavras reservadas **asc** ou **desc**. No caso de não ser especificada uma ordem concreta, será utilizada a crescente como padrão.

Ex:
https://api.lojaexemplo.com/pedidos?order-by=dataPagamento&order=asc

No entanto, neste modelo fica difícil definir mais de um campo para ordenação.

Outra forma de fazer é utilizando o padrão **sort=[{atributo}:{asc|desc}]**.
Ex:
.../pedidos?sort=dataPagamento;desc,dataPedido

No exemplo acima, desejo que a lista de pedidos venha ordenada de forma decrescente pela dataPagamento e de forma crescente - valor default - pela dataPedido.

Importante: Quando se define o contrato da API, é importante definir a lista de atributos disponíveis para ordenação, já quem nem sempre todos eles estarão disponíveis para definir ordem.

### Request > URL > Query Strings > Paginação > Fields
---
Existem situações onde o cliente deseja obter apenas alguns dos atributos de um recurso. Para estas situações pode-se utilizar o query string **fields[atributo.sub-atributo]** para selecionar apenas aqueles atributos do recurso que o cliente deseja receber.
Para "sub-atributos",  utiliza-se o "." para separá-los.
Veja o exemplo:
```
https://api.lojaexemplo.com/clientes/jose-da-silva123/endereços/residencial-1?fields=nome,situacao,logradouro.rua,logradouro.numero
```
Com a chamada acima, não terei como retorno qualquer outro atributo de endereço senão os definidos no fields: _nome_, _situacao_, _logradouro.rua_ e _logradouro.numero_

O uso desta opção nos permite otimizar o uso da banda de rede em toda a cadeia de comunicação, reduzir a quantidade de logs gerados e tamanho das respostas a serem processadas, melhorando a experiência final do usuário.

### Request > Headers
---
O header é um dos componentes que fazem parte do protocolo HTTP. Como o REST é baseado neste protocolo, as REST APIs trafegam headers como parte da comunicação. O header basicamente é um conjunto de chaves/valor.

Por padrão, passamos nos headers informações não relacionadas aos recursos  (que representam as entidades de negócio) expostos nas URLs. De forma análoga, não colocamos atributos que não representem o negócio dentro dos recursos.

Nos headers trafegamos informações técnicas como informações de acesso e credenciais, tokens, chaves de API, Correlation IDs, metadados, etc. 

Alguns headers são padrões do protocolo HTTP, outros podem ser definidos de forma personalizada para as necessidades da sua empresa ou por frameworks de mercado que definem conjuntos de headers para necessidades específicas.

Separar este tipo de informação nos headers evita que as entidades de negócio (recursos) trafeguem dados que só existem por conta da comunicação ser via REST API junto com dados de negócio. 

Por exemplo, se eu estou expondo uma REST API de seguros, informações de token são desconhecidas pelo negócio de seguros e só existem na comunicação por se tratar de REST API. Caso a comunicação acontecesse via webservices, os dados de seguro seriam os mesmos, mas os metadados da comunicação seriam diferentes.

Para mais informações sobre headers, veja: [Segurança](#segurança) e [Cache](#cache).

A seguir, alguns dos principais headers serão explicados.

### Request > Headers > Content-Type
---
O Content-Type é um header que define qual o formato da estrutura de dados presente no Body. Existem muitos tipos de dados, como "text/plain", "application/xml", "text/html", "application/json", "image/gif", "image/jpeg", etc. Quando falamos de REST API, na maior parte das implementações é disponibilizado apenas o "application/json".

Ex:
**Content-Type**: application/json

### Request > Headers > Accept
---
O cliente da REST API pode expressar qual o tipo de informação que ele deseja receber a resposta através do hearder **Accept** .

Ex:
**Accept**: application/json

###  Request > Headers > Correlation ID
---
Não há um header padronizado para Correlation ID, mas é de grande valor que todas as chamadas tenham um Correlation Id. 

Correlation ID é um dado geralmente gerado randomicamente (UUIDs é um bom formato para isso) que deve ser repassado em cada camada de software pelo qual a comunicação trafega. Como cada camada gera o seu próprio log e muitas vezes em banco de dados diferentes, através do Correlation ID, é possível identificar uma chamada específica entre estes diferentes banco de dados, permitindo um mapeamento da chamada de ponta-a-ponta.

O comportamento esperado de qualquer camada é: se o correlation id vier preenchido, repassar para a próxima camada, senão, gerar um novo e repassar. O momento mais adequado para que o Correlation ID seja gerado é no início da cadeia de eventos de uma chamada, normalmente no canal (cliente) quando uma requisição é disparada e algum log de evento do canal é gerado.

Quando a arquitetura das aplicações está baseada em microsserviço, o correlation ID percorre uma tragetória mais ampla do que simplesmente algo que inicia no [gateway](#) e termina na API. Quando um cliente preenche um formulário e clica em gravar, podem ocorrer validações de cartão de crédito, acionamento do sistema de pagamento, comunicação com sistema de envio postal, serviço de e-mails, serviço de geolocalização, etc. sendo cada um desses um microsserviço diferente, com suas camadas de gateway, API, sistema produto, LOGs, etc. Em todos esses sistemas, o mesmo Correlation ID deve ser compartilhado.

Ex:
Uma aplicação client faz uma chamada à API, neste momento cria um Correlation ID para esta chamada e grava o seu log na sua estrutura de log com este Correlation ID. Um Gateway de API recebe este Correlation ID e grava o seu log em outra estrutura utilizando o mesmo identificador. O sistema que responde ao produto que está exposto via API recebe o Correlation ID, grava o seu log e chama outro sistema que responde a uma parte da requisição e assim todos os demais envolvidos nesta chamada mantém o mesmo comportamento.
No momento em que se faz necessário mapear a chamada de ponta-a-ponta, basta unir os logs vindos de diversos fontes através deste identificador.

Quando se define headers não padrões ao HTTP, muitos utilizam a *X-Nome-Header*, no entanto, esta abordagem não é a mais atual desde 2012 ([rfc6648](https://tools.ietf.org/html/rfc6648)).

É interessante colocar um prefixo para diferenciar headers específicos da sua API, por exemplo, se sua empresa se chamada "Amarelo", os headers que não são padrões HTTP podem ser definidos como Amarelo-Nome-Header.

Se observar a estrutura dos headers padrões ([Headers HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)), as palavras são separadas por hífen e iniciam em maiúsculas.

Assim, para o Correlation ID é interessante assumir o formato **Empresa-Correlation-ID**.

Ex, assumindo o nome da empresa como "Amarelo":
- **Amarelo-Correlation-ID**: 680987b5-c18d-4f2f-a772-2a2d422789b1

Recomenda-se que o Correlation ID siga o padrão UUID v4.
>Para conhecer mais sobre UUID, consulte:
https://en.wikipedia.org/wiki/Universally_unique_identifier
https://www.npmjs.com/package/uuid
https://www.uuidgenerator.net/

O Correlation ID também é útil para ser lançado na tela de erro. Por exemplo, nesta tela do  Office 365:

![Imagem exibindo um erro no Office 365 com o Correlation ID](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/resources/office365errorscreen.png?raw=true)

_Com o Correlation Id, é possível buscar nas bases de log a exata transação que estava acontecendo no momento do erro._

###  Request > Verbs
---
Os **métodos** (ou verbos do protocolo HTTP) são basicamente as ações permitidas dentro de uma API. Para cada URL, associamos um verbo. Estas ações solicitam que seja processado CRUD (Create, Read, Update e Delete) nos recursos, alterando os seus estados.

Existem vários verbos HTTP ([rfc2616](https://tools.ietf.org/html/rfc2616)) , mas 5 são os principais e em muitos casos, os únicos adotados nas APIs. Cada verbo HTTP tem um objetido bem definido dentro do contexto de REST API. São eles:
- GET: para a obtenção de um recurso ou uma coleção de recursos.
- POST: para a criação de um recurso.
- DELETE: para eliminar um recurso.
- PUT: para atualizações completas de um recurso.
- PATCH: para atualizações parciais de um recurso.

Os verbos são usados na requisição em conjunto com a URL e às vezes com um [Body](#body), como nos casos de inserção ou atualização.

Existem também outros verbos como OPTIONS, HEAD e TRACE que raramente são utilizados. Vale a leitura da função desses métodos [aqui](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods).

#### Idempotência e Segurança

Os verbos têm características importantes que devem ser conhecidas e respeitadas, são elas segurança e idempotência.
Idempotência significa que se um cliente realiza um reenvio de uma requisição, o servidor devolve a mesma resposta da vez anterior (salvo se o recurso teve seu estado alterado neste meio tempo). Ou seja, a requisição não gera efeito nenhum sobre o recurso.

Resumindo, o envio N vezes de uma requisição deveria provocar o mesmo efeito da requisição realizada uma única vez.

Seguros são todos os verbos que não podem provocam alterações no estado dos recursos. Ou seja, os verbos de consulta apenas (HEAD, GET, OPTIONS). Destes três, somente o GET será utilizado.

Tabela de resumo onde se relaciona o Verbo/Método HTTP e os conceitos de Idempotente e método seguro:
 
 | Verbo | Idempotente | Seguro |
 | --- | --- | --- |
|GET|Sim|Sim|
|PUT|Sim|Não|
|DELETE|Sim|Não|
|POST|Não|Não|
|PATCH|Sim|Não|

É importante escolher o verbo correto conforme estas características ao definir uma API, assim como codificar a API respeitando estas regras. Parta do princípio que o cliente da sua API sabe que, por exemplo, o GET é idempotente e seguro. Por isso, ele não vai hesitar em implementar re-tentativas em caso de insucesso na chamada.
Se ao codificar a API, o desenvolvedor da API codificar um GET que não seja seguro ou idempotente, o cliente não terá o comportamento esperado.
Se o analista de sistemas que cuida do [Gateway](#) resolver colocar [caches](#) nas chamadas com GET, o cache não terá o comportamento esperado. E assim por diante.

###  Request > Headers > Verbs > GET
---
Este verbo é o mais utilizado e serve para buscar dados nas APIs. Você utiliza em conjunto com uma URL com seus [URI Parameters](#uri-parameters) e/ou [Query Strings](#query-strings) definidos para enviar uma consulta e o servidor retorna os dados em caso de sucesso. Em requisições do tipo GET, não se envia [Body](#).

Exemplo utilizando o verbo GET para fazer uma consulta por cidades:
_referência: http://api.exemplo.com/estados/{id-estado}/cidades_
**GET** http://api.exemplo.com/estados/sp/cidades?from-populacao=20000

Exemplo utilizando o verbo GET para fazer uma consulta por uma cidade específica:
_referência: http://api.exemplo.com/estados/{id-estado}/cidades/{id-cidade}_
**GET** http://api.exemplo.com/estados/sp/cidades/santos


### Request > Headers > Verbs > POST
---
O **POST** é usado para criar novos recursos. Você utiliza em conjunto com uma URL com seus [URI Parameters](#uri-parameters) e um [Body](#body) para enviar um conjunto de atributos que represente o estado do novo recurso no momento que você está criando ele.

Exemplo utilizando o verbo POST para criar uma nova cidade:
_referência: http://api.exemplo.com/estados/{id-estado}/cidades/{id-cidade}_
**POST** http://api.exemplo.com/estados/sp/cidades/
```
{
"nome": "São Vicente",
"DDD": 13,
"descrição": "Primeira cidade do Brasil", 
"populacao": 355542
}
```
Dando tudo certo, uma nova cidade será criada na coleção de cidades.

Obs: O POST também pode ser usado em casos especiais onde se faz necessário proteger as informações que seriam usadas na consulta (como em um GET), pois como uso de POST, podemos enviar [Body](#) e o body pode ser encriptado. Por exemplo, no login. 

### Request > Headers > Verbs > PUT
---
O verbo  **PUT** atualiza ou cria um recurso, ou seja, se eu utilizasse o PUT para enviar novamente a cidade de São Vicente (como no exemplo do PUT), o servidor iria sobrepor o recurso com os dados definidos no Body. Caso seja utilizado um PUT e o recurso não exista, a API deveria criá-lo.
Quando usamos **PUT** normalmente estamos atualizando um recurso existente, por isso é importante definir qual é especificamente o recurso através do ID no [URI Parameter](#uri-parameters).
Ex:
_referência: http://api.exemplo.com/estados/{id-estado}/cidades/{id-cidade}_
**PUT** http://api.exemplo.com/estados/sp/cidades/sao-vicente
```
{
"nome": "São Vicente",
"DDD": 13,
"descrição": "Primeira cidade do Brasil.", 
"populacao": 400000
}
```
No exemplo, passamos dois URI Parameters, o {id-estado} com o valor "sp" e o {id-cidade} com o valor "santos".

### Request > Headers > Verbs > PATCH
---
O verbo **PATCH**  serve para fazer atualizações parciais no recurso. Neste caso, ele se comporta de forma semelhante ao PUT, no entanto, define-se no Body apenas os parâmetros que serão alterados.

Ex:
_referência: http://api.exemplo.com/estados/{id-estado}/cidades/{id-cidade}_
**PATCH** http://api.exemplo.com/estados/sp/cidades/sao-vicente
```
{
"populacao": 500000
}
```
No exemplo, a API atualizará apenas o atributo "populacao". Em uma nova chamada usando GET, o valor em "populacao" deverá ser 500000.

### Request > Headers > Verbs > DELETE
---
O verbo DELETE é o responsável por deletar os registros. Semelhante ao GET, você utiliza em conjunto com uma URL com seus [URI Parameters](#uri-parameters) e/ou [Query Strings](#query-strings) definidos para enviar uma consulta e o servidor retorna os dados em caso de sucesso.

Caso a requisição seja em um recurso específico, o recurso será deletado, caso seja em uma coleção de recursos, toda a coleção será deletada, caso seja em um filtro (com query strings), todos os registros que retornarem no resultado serão apagados.

Ex:
_referência: http://api.exemplo.com/estados/{id-estado}/cidades/{id-cidade}_
- **DELETE** http://api.exemplo.com/estados/sp/cidades?to-populacao=5000
Apaga todas as cidades com população até 5000 habitantes.
- **DELETE** http://api.exemplo.com/estados/sp/cidades
Apaga todas as cidades do estado de São Paulo.
- **DELETE** http://api.exemplo.com/estados/sp/cidades/sorocaba
Apaga a cidade de Sorocaba.

### Request > Body
---
O envio de informações com o objetivo de filtrar informações existentes, se dá via URI Parameters e/ou query parametes.

Quando se utiliza os verbos POST, PUT ou PATCH, estamos enviando informações para serem persistidas no servidor via REST API. Neste caso, enviamos as informações no Body.

Ao definir o contrato de uma REST API, cada recurso deve ser declarado como um conjunto de atributos com seus tipos definidos (números, datas, textos, booleanos, etc), descrições, exemplos, obrigatoriedade, etc.

Ao enviar informações via REST API, respeitando as definições do contrato, devemos montar um documento estruturado em alguma notação pré-definida. Hoje, a notação mais utilizada é o [JSON]([https://www.json.org/](https://www.json.org/)).

Ex:
POST http://api.fabricacarros.com/carros
```json
{
  "modelo": "Kicknegade Freestyle HR AT",
  "cor": "Branco Sombra",
  "acessoriosExtras": [
      {
        "id": "senr002",
        "descricao": "Sensor de ré"
      },
      {
        "id": "rck300f",
        "descricao": "Rack de teto"
      },
      {
        "id": "film60",
        "descricao": "Película protetora UV Cinza 60%"
      }
    ],
  "valorTabela": 83590.00,
  "valorVenda": 82000.00,
  "adaptacaoPDC": false
}
```
Quando se define o contrato da API deve-se atentar para que os atributos presentes do Body sejam relacionados apenas ao recurso definido na URL. Por exemplo, na API de cidades usada em outros exemplos neste guia, quando forem feitas chamadas para "http://api.exemplo.com/estados", devem ser trabalhados apenas atributos que definam um estado; quando forem feitas chamadas para http://api.exemplo.com/estados/{id-estado}/cidades devem ser trabalhados apenas atributos que definam uma cidade.

Ao definir os atributos do contrato da REST API, não existe um consenso de mercado quando ao Case a ser adotado, no entanto, dado que os atributos em algum momento serão associados às propriedades das classes nas linguagens de programação seja no servidor ou no cliente, é uma boa prática adotar o lowerCamelCase, dado que as principais linguagens de programação adotam este case para as propriedades.

Deve-se utilizar termos que auto-descrevam o atributo, reduzindo a necessidade de consultas às documentações. Portanto, utilize as palavras por extenso, evitando abreviações, exceto para casos amplamente conhecidos como "Id" ou acrônimos como "BACEN". Ainda assim, o padrão para uso de Acrônimos ou Abreviações é controverso. Dê uma lida neste fórum [acronyms-in-camelcase](https://stackoverflow.com/questions/15526107/acronyms-in-camelcase) para comprovar a complexidade deste assunto.

Também não se coloca em APIs alguns padrões antigos de definição de variável escrevendo o tipo de dado no nome do atributo - por exemplo, dt, int, bool, etc. - dado que os atributos tipos já tão tipados na definição do contrato.

Exemplos dentro do padrão:
-	"id": "12ed58r9"
-	"nomeMae": "Maria"
-	"alertasNaoLidos": false
-	"cpf": "33344455566"
-	"RG": "445556667"
-	"casado": true

Exemplos fora do padrão:
-	"Id": "12ed58r9"
-	"nome-mae": "Maria"
-	"possuialertasnaolidos": false
-	"possui_Alertas_Nao_Lidos": false
-	"flagPossuiAlertasNaoLidos": false
-	"indicadorDeAlertasNaoLidos": false
-	"int_id_cli": 123
-	"flag_casado": true

Os tipos de atributos devem ser declarados e convencionados no contrato da REST API.
As linguagens de definição de contrato como Swagger e RAML documentam os data types suportados: [RAML DataTypes](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#raml-data-types) e [Swagger Data Types]([https://swagger.io/docs/specification/data-models/data-types/](https://swagger.io/docs/specification/data-models/data-types/)).
 
## Response

Após o envio de uma requisição à partir de um cliente, o servidor onde a API está localizada deve processar a requisição e gerar uma resposta (Response).

No response, existem alguns padrões a serem seguidos e eles serão explicados um a um a seguir. Alguns são obrigatórios para o funcionamento da REST API, outros são boas práticas que adotadas pelo mercado.

### Response > Headers
---
Assim como na requisição, no retorno da resposta pode trazer definir um conjunto de chaves/valor que são metadados ou informações técnicas sobre a comunicação que está sendo feita através daquela API.

### Response > Headers > Content-Type
---
Assim como na requisição, o header Content-Type define qual é o formato da estrutura de dados presente no Body.
Ex:
**Content-Type**: application/json

### Response > Headers > Content-Location
---
O header Content-Location expõe a URL relativa (somente dos recursos para frente) ou absoluta (desde o início incluindo o Base Path) que expõe um determinado recurso.

Quando uma requisição é feita com o verbo POST, por exemplo, o cliente ainda não sabe o Id do recurso que ele está criando: muitas vezes são identificadores gerados no momento da gravação. Assim, quando o servidor retorna a resposta, além de preencher a propriedade id no body da requisição, deve-se preencher o header Content-Location.

Ex:
Request
POST http://api.exemplo.com/estados/sp/cidades/
```
{
"nome": "São Vicente",
"DDD": 13,
"descrição": "Primeira cidade do Brasil", 
"populacao": 355542
}
```
Header Content-Location no Response
**Content-Location**: http://api.exemplo.com/estados/sp/cidades/saovic001
```
{
"id": "saovic001",
"nome": "São Vicente",
"DDD": 13,
"descrição": "Primeira cidade do Brasil", 
"populacao": 355542
}
```
### Response > Body
---
Quando é feita uma requisição na API, na maioria das vezes, espera-se uma resposta com informações. Colocamos estas informações no Body. Assim como no Body do [request](#), deve-se utilizar lowerCamelCase para definir os atributos e JSON como padrão de notação.

### Response > Body > Envelope "Data"
Quando nos referimos ao body de request, apenas passamos os atributos sem nenhum tipo de envelope. Ex:
```json
{
	"id": 123,
	"nome": "Carlos",
	"data": "2019-06-04"
}
```
No body de response, colocamos a informação do recurso dentro de um envelope "data". Chamamos de envelope alguns atributos que separam conteúdos importantes na resposta. Esse separação é necessária porque no response podemos ter pagination, warnings, links, summary, etc. Exemplo de Body de response:
```json
"data": [{
    {
		"id": 123,
		"nome": "Carlos",
		"data": "2019-06-04"
	}
}],
"pagination": {
	"page": 1,
	"page-size": 10,
	...
}
"_links": {
	...
}
```
### Response > Body > Recurso unitário, array ou nenhum

Quando se faz uma requisição por um elemento com um ID especificado no [URI Parameter](#), por exemplo, GET .../pessoa/{id-pessoa}, o retorno da resposta será um único elemento. Assim, coloca-se o recurso diretamente no envelope "data". Ex:

Request:
GET .../pessoas/456

Response:
``HTTP-CODE 200``
``content-location: .../pessoas/456``
```json
"data": {
    {
		"id": 456,
		"nome": "José",
		"data": "2019-06-09"
	}
}
```

Quando se faz uma requisição sem o ID, filtrando apenas com [query strings](#), pode-se ter como retorno um ou mais elementos. Neste cenário, retornamos com um array do referido recurso. Ex:

Request:
GET .../pessoas?from-data=2019-06-01

Response:
``HTTP-CODE 200``
``content-location: .../pessoas/456``
```json
"data": [{
    {
		"id": 123,
		"nome": "Carlos",
		"data": "2019-06-04"
	},
    {
		"id": 456,
		"nome": "José",
		"data": "2019-06-09"
	},
	{
		"id": 789,
		"nome": "Maria",
		"data": "2019-07-12"
	}
}]
```

Também é possível se obter resposta vazia quando a busca não retorna nenhum resultado ou nos casos de gravação (POST, PUT, PATCH e DELETE) em que o desenvolvedor da API opta por não trazer um retorno. Neste caso não se traz nada na resposta e, exceto no GET, o [código de resposta HTTP](#) é o 204.

### Response > Body > Paginação

Toda REST API que retorne muitos registros deveria suportar paginação. A paginação reduz o tráfego de dados na rede e o tempo de processamento da API entregando listas menores de resultados por vez. Ainda porque, as interfaces que exibem as informações para os usuários costumam ter um espaço limitado, preenchendo gradativamente conforme o usuário "rola" o conteúdo ou "avança a página".

Sempre que se retorna um resultado paginado, utiliza-se o HTTP Status Code **206 Partial Content**. Além do envelope "data" com o resultado da requisição, deve-se criar um envelope "pagination" com as informações necessárias para permitir que o cliente se movimente entre as páginas.

Existem algumas técnicas diferentes para fazer a paginação. Serão explicadas abaixo.

#### Cursor
#### Page e Page Size

Quando se utiliza a abordagem de page e page size, a requisição pode ser feita informando a página e o tamanho dela como query string (GET .../...?page=10&page-size=50). A API deve recortar do total de respostas apenas as páginas solicitadas conforme solicitação do cliente ou, caso não tenha sido informado, à partir de valores padrões. Por exemplo, em uma requisição GET .../...?page=2&page-size=3, possuindo o banco de dados uma coleção de 8 registros, o retorno devem ser os registros 4, 5 e 6:
- ~~registro 1 (página 1)~~
- ~~registro 2 (página 1)~~
- ~~registro 3 (página 1)~~
- **registro 4 (página 2)**
- **registro 5 (página 2)**
- **registro 6 (página 2)**
- ~~registro 7 (página 3)~~
- ~~registro 8 (página 3)~~

No body de retorno, devemos ter um envelope "data" com o array de recursos e um envelope "pagination" com seguinte estrutura:
```json
{
	"data": [{
		...
	}],
	"pagination":{
		"first": "/cartoes",
		"last": "/cartoes?page=3",
		"previous": "/cartoes?page=1",
		"next": "/cartoes?page=3",
		"page": 2,
		"totalPages": 3,
		"totalElements": 8
	}
}
```
Os campos são auto-explicativos, só é preciso atenção especial para definição do que é considerado a página inicial, se 0 ou 1. Por convenção, adotar o 1 vai fazer com que a paginação coincida com o dado que normalmente é exibido na tela para o usuário.

No caso do exemplo, para obtenção da página seguinte, o cliente faria a chamada GET .../...?page=3&page-size=3, conforme informado no atributo "next".

Também devem ser mantidos (repetidos) os query strings (filtros, ordenação, etc..) que o cliente passar na requisição, ainda porque, caso o cliente altere o filtro, toda o envelope de páginação pode ter seus valores alterados.

Quando se está na primeira página ou na última, os atributos "previous"e "next" devem ficar vazios.

Muitas vezes, criamos APIs para sistemas legados e com isso, precisamos nos ajustar a comportamentos  já existentes, por conta disso, a paginação pode ter alguns variantes, por exemplo:
- O sistema pode informar a página, não por número, mas através de um ID, nestes casos, basta substituir a informação numérica do  "previous"e "next" por string.
	- Ex: GET .../...?page=fgg12d8bfb4567820c46
- O sistema pode não ter a informação da quantidade total de registros (ex: totalElements e totalPages),  dessa forma, não temos como devolver todas as propriedades. Neste caso, devolvemos apenas as que são possível de serem informadas.

##### - Offset e Limit
##### - Ordenação
##### Fields
### HTTP Status Codes
---
No HTTP existem os [códigos de status](#[https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)). Eles de forma padronizada reportam se a requisição foi processada com sucesso ou não. Existem muitos códigos e nem todos são adotados pelo REST. A adoção veio com o uso do do mercado e alguns códigos são amplamente usados e outros nem tanto.

Abaixo, coloco os que adoto como padrão nas empresas onde trabalho.

Basicamente, os códigos de resposta são agrupados: 1xx, 2xx, 3xx, 4xx e 5xx. 

**Grupo 1xx**

Este é o grupo de status que dá respostas informativas. Respostas com este status são raramente utilizadas.

**Grupo 2xx**

Os códigos deste grupo são usado em caso de sucesso na requisição. Os mais utilizados são:

- **200 OK**:  Código mais utilizado e que indica que a requisição foi processada com sucesso. Esta resposta pode ser usada em todos os verbos. Os dados solicitados serão retornados no Body.

- **201 Created**:  Indica que a requisição foi bem sucedida e que um novo recursos foi criado como resultado. Esta resposta é utilizada em requisições do método POST.

- **204 No Content**: A requisição aconteceu com sucesso, no entanto, não há body na resposta. O 204 não é utilizado para o verbo GET. Nos casos de GET cujos critérios na requisição provocaram uma resposta vazia, utilizamos o HTTP code 200 com o envelope "data" vazio.

- **206 Partial Content**: O servidor encontrou uma uma grande quantidade de registros na respostas e devolveu de forma paginada. A resposta inclui um envelope "pagination" com informações sobre a paginação.


**Grupo 3xx**

Este grupo define respostas de redirecionamento. Servem para informar o cliente sobre mudanças na requisição e redirecionamento para uma nova URL. Para saber mais sobre estes status codes, veja [requisições assíncronas](#). Os mais utilizados são:

TODO: exemplificar melhor isso aqui!

- **301 Moved Permanently**: Informa que o recurso A agora é o recurso B, de forma que quando o cliente solicitar o recurso A ele será automaticamente encaminhado ao recurso B.

- **303 See Other**: A resposta para a requisição encontra-se em outra URL definida no header **Location**. Veja [Tarefas Assíncronas](#) para mais informações.

- **304 Not Modified**:  Resposta utilizada quando o recurso está em [cache](#), informando ao cliente que a resposta não foi modificada, e que o cliente pode usar a mesma versão em cache da resposta.

- **307 Temporary redirect**:  Se trata de um redirecionamento de uma página para outro endereço, porém que é com caráter temporário, e não permanente. Provavelmente por conta de alguma manutenção no sistema.

**Grupo 4xx**

Esse grupo informa os erros cometidos pelo cliente durante o request. São eles:

- **400 Bad Request**: Significa que o servidor não consegue entender a requisição, pois existe uma sintaxe ou estrutura inválida, pode ser caracteres não permitidos na URL, falta de cabeçalhos obrigatórios, cabeçalhos mal formados, falta de query strings obrigatórias, falta de atributos obrigatórios, body com estrutura inválida, etc.

- **401 Unauthorized**: A camada de segurança do recurso solicitado ao servidor, apontou que não está sendo utilizada as credenciais corretas nessa requisição (token, por exemplo). É um erro de autenticação.

- **403 Forbidden**: As credenciais (token) estão corretos, mas o usuário não tem permissão para acessar aquele recurso.

- **404 Not Found**: O servidor não encontrou o recurso solicitado pelo cliente. Provavelmente a URL está mau formada ou está sendo feita a busca com um [Path Parameter](#) inválido.

- **405 Method Not Allowed**: O recurso (URL) existe mas o verbo usado não foi definido para ela.

- **422 Unprocessable Entity**:	Quando a requisição está correta ao nível sintático, mas existem erros de negócio na requisição. Por exemplo, se existe regra que o uso de um query parameter está condicionado a outro e eles não foram preenchidos, ou uma data informada é inválida, ou uma requisição de transferência de dinheiro é feita e a conta não tem fundo, etc.

- **428 Precondition Required**: TODO estudar esse aqui.

- **429 Too Many Requests**: Informa ao cliente que ele excedeu o limite permitido de requisições. Veja [throttling](#) para entender mais sobre este código de retorno.

**Grupo 5xx**

São códigos que retornam erros que aconteceram por culpa do servidor. Ou seja, a requisição foi feita corretamente pelo cliente, porém ocorreu um erro no servidor. São eles:

- **500 Internal Server Error**: Erro mais genérico para informar que o servidor encontrou um cenário inesperado de erro que não soube tratar, e por isso não conseguiu retornar uma resposta na requisição do cliente.

- **501 Not Implemented**: O verbo HTTP não foi disponibilizado na API em nenhum dos recursos.

- **503 Service Unavailable**: O servidor não está respondendo por que está fora do ar, em manutenção ou sobrecarregado.


## Processamento Assíncrono
## Segurança

(o conjunto das implementações garante a segurança)

Sua API pode perger o engajamento da comunidade, caso a implementação de segurança seja muito complicada. Assim, busque a simplicidade e utilização de padrões conhecidos de mercado.

### Segurança > Autenticação e Autorização
---
Autenticação é o processo de garantir que um consumidor da API é quem ele diz que é. De forma simplificada isto se dá através do envio de "login" e "senha".

Autorização acontece após a autenticação e serve para verificar se o consumidor da API tem acesso a um determinado recurso.

Fazendo uma analogia à vida real, ao ir a um show, você passa pela bilheteria e recebe a pulseira (autenticação), passa pela catraca (autorização) e tenta entrar na pista VIP (sendo que você comprou pista comum), neste caso, você não terá acesso a este espaço (autorização).

### Segurança > Autenticação > Basic Authentication
---
**Basic authentication** é um meios de autenticação mais simples especificado no protocolo HTTP.

O cliente envia uma requisição com o header **"Authorization: "Basic"** + **usuário** + **":"** + senha, sendo "usuário:senha" em base64.

Por exemplo, para autorizar o usuário  **michel** com senha  **abc123**, o client enviaria na requisição, o seguinte header:

**Authorization**: Basic bWljaGVsOmFiYzEyMw==

Como essa é uma das formas menos segura de autenticação, normalmente é usada apenas em APIs internas, cujo escopo de acesso é limitado pela rede.

### Segurança > Autenticação > API Keys
---
Quando se usa API Keys, ao invés de enviar usuário e senha, o servidor fornece uma única chave de acesso e o cliente usa esta chave de acesso em todas as requisições.

A API Key é passada em um header customizado (varia de acordo com a empresa), assim para eu me autenticar na API - supondo que a empresa se chame "Amarelo" - eu faria uma requisição com o seguinte header:

**Amarelo-Api-Key**: d9bfb805-aea2-4647-acc7-36e91337f50a

Assim como o Basic Authentication, essa é uma forma pouco segura, devendo ser usada no pior dos casos, apenas na rede interna.

### Segurança > HTTPS
---
A utilização do HTTPS mantém as mensagens seguras e criptografadas, dificultando a interceptação da requisição. Busque configurar suas aplicações para utilizar as versões mais recentes dos protocolos do HTTPS (como o [TLS](https://pt.wikipedia.org/wiki/Transport_Layer_Security)).

### Segurança > OAuth
---
TODO: ARRUMAR
```
O OAuth está na sua versão 2.0, e não é apenas um método de autenticação, e sim um protocolo completo com diversas especificações de segurança.

Ele é extremamente útil para o processo de autenticação e autorização, e por isso, atualmente é o método mais recomendado para o cenário de APIs.

Vamos entender alguns conceitos básicos do OAuth 2:

-   Resource Owner: entidade dona do recurso, ou seja, que é capaz de controlar o acesso a um recurso. Normalmente é o próprio usuário.
-   Resource Server: servidor que possui os recursos, e por sua vez, recebe as requisições.
-   Authorization Server: servidor que gera tokens de acesso para permitir que o client acesse os recursos autorizados pelo resource owner.
-   Client: aplicação que acessa os recursos do resource server.

Para entender melhor, vamos supor que você desenvolveu uma aplicação que utiliza dados do usuário do Facebook, então vamos simular como seria um fluxo básico de autenticação via OAuth 2.0:

1.  O usuário acessa seu site ou app que teria um botão de "integre ao facebook", sendo que o seu site ou app seria o  **client**.
2.  Ao clicar no botão, o usuário é redirecionado para a tela de login do Facebook (**authorization server**).
3.  Após o usuário informar as credenciais, o Facebook fornece um código de acesso ao  **client**.
4.  Então o  **client**  solicita autorização aos recursos (endpoints da API do Facebook) para o  **resource owner** (que é o próprio usuário)  enviando  o código de acesso recebido anteriormente.
5.  O  **authorization server**  por sua vez, verifica se o código de acesso é valido, e caso positivo, ele gera um token de acesso para retornar ao  **client**.
6.  Por último, agora que o  **client**  já tem o token de acesso e autorização aos recursos, então a cada requisição, o  **resource server (API do facebook)**  irá responder com os dados protegidos.

Como é um assunto denso, vou deixar aqui o  [link](https://nordicapis.com/api-security-oauth-openid-connect-depth/)  de um ótimo artigo que explica em detalhes esse protocolo.

Outros assuntos acerca da especificação do oAuth e que são interessantes para implementação:

[OpenID Connect](https://openid.net/connect/)**:** Extensão da funcionalidade de  **autenticação**do OAuth. Como o OAuth foi projetado para autorização, o OpenID Connect acaba sendo um ótimo complemento na segurança de APIs, conseguindo ajudar no sentido de provar que o usuário é realmente quem ele diz que é.

[JWT](https://jwt.io/)**:** Formato de token seguro que utiliza JSON como base.
```

### Segurança > Dados sensíveis na URL
---
A URL é uma informação que trafega entre as diversas camadas de comunicação sem nenhum tipo de encriptação e normalmente, é gravada em logs ou caches por todas as camadas. Assim, nunca se coloca dados sensíveis como informações pessoais ou de credenciais na URL.

Ex:
GET https://api.exemploempresa.com/pedidos?apiKey=abcd123
GET https://api.exemploempresa.com/pedidos?login=joao&senha=1234

(material do magnani!!!!)

### Segurança > Throttling, Rate Limiting e Quotas
---
Excesso de requisições podem deixar a API indisponível, sejam por conta de ataques ou até mesmo por excesso de  chamadas vindas naturalmente dos clientes. É interessante implementar travas para evitar que as requisições ultrapassem a capacidade de processamento da sua API.

Com o uso de alguns headers, você consegue limitar a quantidade de requisições e informar o cliente sobre o uso dela:

- **Empresa-RateLimit-Limit**: Quantidade de requisições permitidas durante um período específico (dia, hora, minuto, segundo. etc).
- **Empresa-RateLimit-Period**: Definição do período, sendo:
	- y   = ano
	- M   = mês
	- d   = dia
	- h   = hora
	- m   = minuto
	- s   = segundo
- **Empresa-RateLimit-Remaining**: Quantidade de requisições restantes do período corrente.
- **Empresa-RateLimit-Reset**:  Quantidade de segundos restantes até o término período corrente.

Ex, assumindo o nome da empresa como "Amarelo":
- **Amarelo-RateLimit-Limit**: 200
- **Amarelo-RateLimit-Period**: m
- **Amarelo-RateLimit-Remaining**: 84
- **Amarelo-RateLimit-Reset**:  28

E nos casos em que o cliene casos que o client ultrapasse o limite de requisições, a API  deve retornar o [status code](#status-code) 429 -Too Many Requests.

> Exemplo de uso de Rate Limit no GitHub:[https://developer.github.com/v3/#rate-limiting](https://developer.github.com/v3/#rate-limiting)

### Segurança > Validações
---
Sempre valide a estrutura e tipos de dados recebidos em [URI Parameters](#), [Query Strings](#) e [Body](#) da mensagem antes de executar qualquer lógica dentro da API.

Parâmetros não especificados no contrato, podem ser indício de um ataque malicioso. 

### Segurança > API Gateway
---
Quando utilizamos um API Gateway, criamos uma camada especializada entre o cliente e as APIs. Um API gateway centraliza e implementa os padrões de autenticação, segurança, validações, throtlling, roteamento, tradução de protocolo, logs, auditoria, cache, etc. barrando acessos indevidos antes mesmo da requisição atingir a API.

```mermaid
graph LR
A[Cliente A] --> D[Gateway]
B[Cliente B] --> D[Gateway]
C[Cliente C] --> D[Gateway]
D[Gateway] --> E[API Pagamentos]
D[GatGateway] --> F[API Seguros]
D[Gateway] --> G[API Cartões]
```

## Versionamento
## Performance, Cache e compressão
