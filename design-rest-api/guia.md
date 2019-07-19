
# Guia de Design REST

## _Abstract_

_This document is a guideline for REST API designing based on the best practices and experiences from author who headed some REST API service governance projects in the banking environment. REST API is an architectural style used to expose services on the web._

Este documento é guia para design de REST API com base em melhores práticas e experiências do autor que liderou alguns projetos de governança de serviços com REST API em ambiente bancário. REST API é um estilo arquitetural usado para expor serviços na web.

## _Status of this document_

_This is an "alive" document where the author is updating with day-by-day experience. Therefore, it may never arrive in its final version. However, the published content already accumulates project experiences that totaled a few hundred REST APIs._

Este é um documento "vivo" em que o autor está atualizando com a experiência do dia a dia. Portanto, nunca pode chegar em sua versão final. No entanto, o conteúdo publicado já acumula experiências de projetos que totalizaram algumas centenas de REST APIs.

## Conteúdo

- [Introdução e conceitos básicos](#introdução-e-conceitos-básicos)
- [Request](#request)
	- [URL](#request--url)
		- [Base](#request--url--base)
		- [Resources](#request--url--resources)
			- [Funções que não são CRUD](#request--url--resources--fun%C3%A7%C3%B5es-que-n%C3%A3o-s%C3%A3o-crud)
		- [Domínios Funcionais](#request--url--resources--dom%C3%ADnios-funcionais)
		- [Path Parameters](#request--url--resources--path-parameters)
		- [Query Strings](#request--url--query-strings)
			- [Full text search](#request--url--query-strings--full-text-search)
			- [Paginação](#request--url--query-strings--pagina%C3%A7%C3%A3o)
				- [Range](#request--url--query-strings--pagina%C3%A7%C3%A3o--range)
				- [Page e Page Size](#request--url--query-strings--pagina%C3%A7%C3%A3o--page-e-page-size)
				- [Limit](#request--url--query-strings--pagina%C3%A7%C3%A3o--limit)
			- [Ordenação](#request--url--query-strings--ordena%C3%A7%C3%A3o)
			- [Fields](#request--url--query-strings--fields)
			- [Views](#request--url--query-strings--views)
			- [Expand](#request--url--query-strings--expand)
		- [Alias](#request--url--alias)
	- [Headers](#request--headers)
		- [Content-Type](#request--headers--content-type)
		- [Accept](#request--headers--accept)
		- [Correlation ID](#request--headers--correlation-id)
	- [Verbs](#request--verbs)
		- [GET](#request--verbs--get)
		- [POST](#request--verbs--post)
		- [PUT](#request--verbs--put)
		- [PATCH](#request--verbs--patch)
		- [DELETE](#request--verbs--delete)
	- [Body](#request--body)
- [Response](#response)
	- [Headers](#response--headers)
		- [Content-Type](#response--headers--content-type)
		- [Content-Location](#response--headers--content-location)
		- [Location](#response--headers--location)
	- [Body](#response--body)
		- [Envelope "Data"](#response--body--envelope-data)
		- [Recurso unitário, array ou nenhum](#response--body--recurso-unit%C3%A1rio-array-ou-nenhum)
		- [Full text search](#response--body--full-text-search)
		- [Paginação](#response--body--pagina%C3%A7%C3%A3o)
			- [Range](#response--body--pagina%C3%A7%C3%A3o--range)
			- [Page e Page Size](#response--body--pagina%C3%A7%C3%A3o--page-e-page-size)
			- [Limit](#response--body--pagina%C3%A7%C3%A3o--limit)
		- [Ordenação](#response--body--ordena%C3%A7%C3%A3o)
		- [Fields](#response--body--fields)
		- [Views](#response--body--views)
		- [Expand](#response--body--expand)
		- [Errors e Warnings](#response--body--errors-e-warnings)
		- [HATEOAS](#response--body--hateoas)
	- [HTTP Status Code](#response--http-status-codes)
- [Tipos de dados](#tipos-de-dados)
- [Processamento assíncrono](#processamento-ass%C3%ADncrono)
- [Processamento em lotes](#processamento-em-lotes)
- [Recursividade](#recursividade)
- [Versionamento](#versionamento)
- [Segurança](#seguran%C3%A7a)
- [Performance, Cache e compressão](#performance-cache-e-compress%C3%A3o)
- [Palavras Finais](#palavras-finais)

## Introdução e conceitos básicos

Os conteúdos abaixo cobrem alguns conceitos aliados às boas práticas no design de RESTful APIs. Quando falamos em design de RESTful APIs, estamos abordando como definir um contrato de API que expõe as entidades e funções de um determinado sistema respeitando as restrições do REST.

> Para fazer o entendimento das necessidades de negócio e definição das entidades, recomendamos o uso de Domain Driven Design.
> TO DO: Em breve será disponibilizado um guia explorando toda a fase de entendimento e transformação dos conceitos de negócio em entidades para serem expostas como recursos via REST API.

> O conteúdo deste material contempla os conceitos para especificação do contrato de REST APIs de forma genérica, não abordando necessariamente nenhuma linguagem específica de definição de contrato: como RAML, Open API, etc.
> TO DO: Em breve serão disponibilizados exemplos dos conceitos deste guia em uma destas linguagens.

**API** (**A**pplication **P**rogramming **I**nterface) é um software que permite comunicação entre sistemas onde há um fornecedor e um ou mais consumidores de informação, serviços, etc.  Para consumir uma API respeita-se um contrato que deve incluir o protocolo de comunicação, as operações (consultas e atualizações) e os formatos de dados para entradas, saídas e erros.
A maioria dos softwares nos quais nós interagimos são construídos para atender às necessidades humanas e normalmente referenciam as "coisas" reais das quais esses softwares tratam. Por exemplo, um software de biblioteca vai representar livros, seções, autores, etc. e, através de alguma interface, permitir que um usuário interaja com estas representações. Uma API difere-se deste tipo de software no que tange o usuário: quem interage com o software é outro software, não diretamente o usuário final. No entanto, quem desenvolve o software que interage com a API é um programador - e até a data deste documento, a maioria ainda são humanos - e para que a experiência deste usuário programador e do usuário do software que ele desenvolve seja a melhor possível, princípios de design devem ser respeitados. 

**REST API** (**R**epresentational **S**tate **T**ransfer **A**pplication **P**rogramming **I**nterface) é um estilo de arquitetura que define um conjunto de restrições e propriedades baseadas em HTTP fornecendo interoperabilidade entre sistemas de computadores na internet (ou rede local). REST permite que os sistemas consumidores acessem e manipulem o estado de representações textuais de recursos usando um conjunto pré definido de operações. As "coisas" da vida real em REST se chamam recursos. O conjunto de valores dos atributos de um determinado recurso em um determinado momento, se chama estado.
Por colocar mais restrições do que WebServices - que é uma outra forma de integrar sistemas - e utilizar mais recursos do HTTP do que WebServices, acabou padronizando melhor a forma de comunicação e isso agilizou o desenvolvimento das integrações.
O REST enfatiza simplicidade, extensibilidade, confiabilidade e performance: 
- **Simplicidade** porque a forma como se interage com REST APIs é bem definida e de certa forma restritiva e a interação é stateless, ou seja, todas as informações necessárias para o processamento de uma requisição (no REST, alteração do estado de um recurso) deve estar contida naquela requisição. 
- **Extensibilidade** porque um determinado recurso pode ser representado em diferentes formatos e até mesmo versões, além da possibilidade de se adicionar novos recursos sem precisar alterar os já existentes. 
- **Confiabilidade** porque existe uma separação clara das ações que podem ser feitas e dos seus efeitos colaterais.
- **Performance** porque faz parte dos principais pilares do REST o uso de cache através de uma semântica bem definida, além de uma arquitetura baseada em separação de camadas, permitindo que partes diferentes do sistema possam ser escaladas de forma independente e isolada. 

O **HTTP** (**H**yper**T**ext **T**ransfer **P**rotocol) - ou HTTPS na sua variante com encriptação - é o protocolo da internet, no qual acontece a grande maioria das comunicações. Basicamente, ele trata requisições entre um cliente e um servidor através de uma mensagem que contém um Header e um Body. Se fizer uma analogia a uma carta, eu tenho informações de entrega no envelope (header) e um conteúdo (carta dentro do envelope). O ciclo de vida contempla um envio (request que parte do cliente) e um retorno (response que parte do servidor). Na maioria das implementações, a comunicação é stateless, ou seja, ao término do ciclo "request/response", as conexões com o servidor são encerradas. Além da mensagem, o HTTP define verbos para tratar as mensagens, códigos de retorno para indicar sucesso ou insucesso no processamento, padrões de endereço do servidor, entre outros.

**RESTful** é um conceito que define uma API construída seguindo todas as restrições impostas pelo REST. Leonard Richardson definiu um [modelo de maturidade](https://martinfowler.com/articles/richardsonMaturityModel.html) onde ele divide a adoção do REST em etapas. Quando se implementa todas as etapas, considera-se que aquela é uma RESTful API.

O REST surgiu à partir da dissertação de Roy Thomas Fielding - um dos criadores da web - e se baseia fortemente no HTTP reaproveitando muitos conceitos deste protocolo. A dissertação define de forma aberta o conceito e o mercado foi refinando e definindo convenções. Dessa forma, em alguns pontos existem várias formas de fazer a mesma coisa e não há necessariamente a mais correta. Assim, cabe a você analisar os *trade-offs* de cada forma e escolher a que melhor atende às necessidades dos seus software e principalmente do cliente consumidor do seu software.

O ponto mais importante quando se fala em boas práticas é que seja adotado um padrão, pois à partir do momento que se tem um padrão, a curva de aprendizado para novas soluções é mais rápida e podem ser utilizados frameworks e automações para reduzir o tempo de desenvolvimento dos sistemas.

Neste guia, busco compartilhar os padrões que implemento nas empresas onde já trabalhei ou boas práticas definidas em livros, artigos ou até mesmo APIs de mercado.
 
## Request

Assim como no protocolo HTTP, um cliente (consumidor da API) deve enviar uma mensagem dentro de um determinado padrão a um endereço (onde o servidor responde àquele padrão de mensagem) e aguardar a resposta.

Na requisição, existem alguns padrões a serem seguidos e eles serão explicados um a um a seguir. Alguns são obrigatórios para o funcionamento da REST API, outros são boas práticas.

<sub>ir para: [índice](#conte%C3%BAdo) | [response](#response)</sub>
### Request > URL

A URL (Universal Resource Location) é endereço onde o cliente vai fazer a requisição. Cada URL identifica um recurso diferente na API. Como no navegador, quando digitamos um endereço de um site e ele nos responde com a página, no REST é enviada uma solicitação para o endereço e ele nos responde com informações sobre o recurso.

A URL é formada por basicamente 3 partes [Base Path](#request--url--base), [Resources](#request--url--resources) (ou Path) e [Query Strings](#request--url--query-strings). 

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

<sub>ir para: [índice](#conte%C3%BAdo)</sub>
### Request > URL > Base

O Base Path é a parte inicial da URL, nele você tem o protocolo (http:// ou https://, por exemplo) e o endereço do servidor na web. O Base Path se repetirá em todas as requisições.
Ex:
- https://maps.googleapis.com
- https://management.azure.com
- https://api.spotify.com

Algumas empresas têm URLs que representam os produtos da empresa, em outras a própria empresa é um produto. Nestes cenários, normalmente encontramos Base Paths com estruturas semelhantes a essas:
- https://api.empresaexemplo.com
- https://api.produtoexemplo.com

Em outros cenários, não usam o termo "api" como nos exemplos do google ou azure (acima).
Mas o ponto principal é que normalmente é utilizada uma URL diferente da URL do site da empresa (http://www.empresaexemplo.com ou http://empresaexemplo.com).

Identificar ambientes de homologação e produção no Base Path também é interessante, pois reduz a configuração do ambiente durante o processo de publicação a apenas um parâmetro. Ex:
- Homologação: https://api-hml.empresaexemplo.com
- Desenvolvimento: https://api-dev.empresaexemplo.com
- Sandbox: https://api-sbx.empresaexemplo.com
- Produção: https://api.empresaexemplo.com

<sub>ir para: [índice](#conte%C3%BAdo)</sub>
### Request > URL > Resources

No REST, a letra R significa "Representação ". Representação é a forma de apresentar os recursos ("coisas") que existem no sistema. Recursos no REST são nomeados usando substantivos. Eles são as entidades de sistema que serão expostas para outros sistemas. Cada um desses recursos têm um endereço (URL) diferente e são representados por textos separados por barras após o Base Path. É muito importante que os nomes dos recursos sejam estruturados de forma que cumpram uma hierarquia que proporcione sentido próprio à URL. Os nomes escolhidos para os recursos devem ser de fácil entendimento, de modo que ao ler a URL se obtenha rapidamente a informação sobre qual recurso ela representa.

Ex:
- Endereço da representação de um recurso chamado "artists" no Spotify: https://api.spotify.com/v1/artists/
- Endereço da representação de um recurso chamado "albums" do Spotify: https://api.spotify.com/v1/albums 
- Endereço da representação de um recurso específico chamado dentro dos "albums" do Spotify: https://api.spotify.com/v1/albums/{id}
- Endereço da representação de um recurso chamado "tracks" de um "album" específico do Spotify: https://api.spotify.com/v1/albums/{id}/tracks

Repare que existe um Base Path se repetindo em cada um dos endereços (https://api.spotify.com) e um "/v1" também. Ele representa a versão da API e também é exposto como um recurso. Há mais informações sobre versionamento no capítulo [Versionamento](#versionamento).

Observe que algumas URLs têm recursos como artists ou albums que não têm o {id} na frente. Elas deverão retornar listas, pois não se especifica nenhum "artist" ou "album" específico através de um {id}. Outras URLs têm o {id} e retornarão apenas um item cujo id seja o id definido naquela URL. Este {id} é um [Path Parameter](#request--url--resources--path-parameters).

Quando os recursos forem formados por duas palavras, é boa prática separar com hífen. E assim como no [Base Path](#request--url--base), escreve-se tudo em minúsculo. 

Os recursos na maioria dos casos serão substantivos no plural, pois geralmente definem coleções (ex: cartões, usuários, clientes, carros, endereços etc.) Ao pensar no nome do recurso, é importante verificar se o nome não conflitará com alguma possível implementação parecida no futuro. Por exemplo, imagina um recurso que se chamaria ofertas_credito. Neste caso, a deve-se perguntar: "este serviço ofertará todos os tipos de crédito no domínio de crédito?" Se a reposta for: "Não, apenas consignado", deve-se especificar o nome da entidade como ofertas-credito-consignado porque no futuro uma nova rota que oferte todos os tipos de crédito poderá ser criada com o nome oferta-credito.
Ex:
http://api.exmpresaexemplo.com/creditos/v1/ofertas-credito-consignado

Recursos criados no singular são menos frequentes e pode acontecer quando:
-	Haver requerimentos de segurança ou permissão específicos para um atributo. Neste caso, o atributo se torna um recurso na URL, podendo ser aplicadas políticas específicas de segurança naquela rota.
-	No recurso existir um [atributo](#request--body) complexo com um número elevado de "sub-atributos". Por conta da complexidade, pode-se optar por tornar este atributo um recurso.
-	Haver recursos que não existam no plural, como saldo, extrato etc.

Quando definir os recursos a serem expostos, deve-se **evitar**:
- Utilizar termos que não façam parte do nome da entidade de negócio, por exemplo: <s>detalhes_</s>lancamentos-cheque. A entidade de negócio se chama lancamentos_cheque, exibir todos os atributos ou não, fica a cargo de filtros na [query string](#request--url--query-strings), não na exposição como um recurso.
- Utilizar termos que representam as ações básicas do CRUD (consultar, gravar, atualizar, apagar, etc). Por exemplo: <s>consultar-</s>fatura. As ações básicas do CRUD são representadas pelos [verbos HTTP](#request--verbs) (GET, POST, PUT, PATCH e DELETE).
- Expor representações de detalhes do backend na entidade, por exemplo: /<s>servico-</s>transferencias. O termo "servico" neste caso está representando, por exemplo, o "serviço" do sistema que processa uma transferência. Este tipo de informação deve ser abstraída no nome do recurso. Nomeie-o apenas como .../transferencias. Ou um exemplo pior, chamar um recurso de .../X0PSD002, sendo X0PSD002 o nome interno de um serviço que processa boletos. Nomeie-o apenas como /boletos.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

### Request > URL > Resources > Funções que não são CRUD

Uma das restrições do REST é que ele se aplica a recursos que representam as "entidades" de um [Domínio Funcional](#request--url--resources--dom%C3%ADnios-funcionais). Para alterar o estado dessas entidades, usamos os verbos GET, POST, PATCH, PUT e DELETE, que basicamente fazem o CRUD.
 
No entanto, existem alguns cenários em que não temos "entidades" de um domínio, mas sim, ações e que não fazem parte do conjunto restrito de [verbos](#request--verbs) do HTTP. Aqui estão alguns exemplos:
- Distância entre dois pontos (via coordenadas de GPS, CEP etc.)
- Validação de dados de cartão de crédito
- Cálculos em geral

Para estes casos, tratamos essas funções como recursos e para facilitar a identificação de que não é uma "entidade", usamos verbos no lugar dos substantivos.
Ex:
- GET .../**calcular-distancia**?from-latitude=48,8584&from-longitude=2,2945&to-latitude=-22.951916&to-longitude=-43.2104872
- POST .../**validar-cartao**

```
{
	"numero": "5346931596124410",
	"nomeTitular": "JOSE A OLIVEIRA",
	"dataValidade": "2024-05-23",
	"CVV": 996
}
```
- GET .../**somar**?primeiro-valor=72&segundo-valor=28
  
Observem que algumas chamadas representam consultas seguras e idempotentes. Portanto, o verbo HTTP que foi utilizado foi o GET.
No exemplo da validação de cartão, trafegamos **dados sensíveis**. Mesmo sendo uma consulta segura e idempotente, por questões de segurança, o tráfego de dados sensíveis são feitos sempre via POST para permitir a encriptação do Body.

Nos casos em que a chamada não é segura, nem idempotente (por exemplo, em uma API de login, em que depois de algumas tentativas o login será bloqueado) devemos usar verbos como o POST. 

Para saber mais sobre idempotência, leia sobre os [verbos](#request--verbs) HTTP.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

###  Request > URL > Resources > Domínios Funcionais

Em algumas empresas, pode ser que não exista um Base Path para cada produto e uma mesma empresa forneça recursos de vários produtos diferentes através de API. Neste caso, pode-se criar um recurso que serve para agrupar os recursos de cada um dos produtos.
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

> Para mais detalhes sobre essas APIs veja: https://www.bbvaapimarket.com/products

A mesma abordagem de agrupamento provavelmente será útil para APIs expostas apenas internamente nas empresas para integrações entre diferentes sistemas. Isso porque definir um recurso na URL envolve menos mudanças na implantação do que criar um Base Path para cada um dos produtos/assuntos da empresa. 

Em algumas empresas, esses grandes assuntos são conhecidos como domínios funcionais. Eles agrupam entidades relacionadas por um contexto funcional como produtos (cartões, contas, etc.), processos (contratações, on-boarding, etc.) ou serviços (comunicações, chat, etc.). É boa prática, usar nomes em minúsculo e se composto, separar com hífen. Os domínios funcionais na maioria dos casos serão substantivos no plural.

Também é interessante manter os contratos de cada domínio separados para dar independência para os times que cuidam de cada um deles, além de permitir gerir o ciclo de vida de cada um deles de forma separada.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

###  Request > URL > Resources > Path Parameters

Nos exemplos acima, alguns dos recursos estavam com a notação {}. Eles são Path Parameteres, ou seja, são parâmetros que podem variar conforme a consulta. Por exemplo, na API do Spotify https://api.spotify.com/v1/albums/{id}/tracks, no lugar do {id}, em tempo de execução, o sistema deverá colocar o Id de um álbum específico para poder listar as faixas do respectivo álbum. 
Ex:
- https://api.spotify.com/v1/albums/0sNOF9WDwhWunNAHPD3Baj/tracks
- https://api.spotify.com/v1/albums/2kLIU5SVDjnEUndjks72ddP/tracks

Os path parameters são destinados exclusivamente para identificar os recursos dentro das coleções mediante seu identificador único.

É boa prática, usar nomes em minúsculo e se composto, separar com hífen. Os Path Parameters na maioria dos casos serão compostos por "id" + nome do recurso no singular em camelCase.

Ex:
- http://api.exemploempresa.com/cartoes/v1/cartoes/{idCartao}/faturas/{idFatura}/lancamentos/{idLancamento}

Desta forma, quando a aplicação recuperar a coleção de parâmetros, terá a seguinte lista:
*idCartao*, *idFatura* e *idLancamento*, cada uma com um nome que a identifica unicamente na coleção.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

###  Request > URL > Query Strings

Além dos Path Parameters, Query Strings também permite passar parâmetros na URL da chamada. Query Strings são usadas para filtrar resultados em consultas. Elas são definidas após os resources, iniciando com uma **?** e separando cada par de chave e valor  (**chave=valor**) com um **&**:

***.../recurso?chave1=valor1&chave2=valor2&chaveA=valorA***

Assim, uma API que busque restaurantes de comida japonesa na cidade de são paulo, teria uma URL semelhante a esta:

<pre>https://api.empresaturismo.com.br/locais/v1/restaurantes?tipo-comida=japonesa&cidade=S%C3%A3o%20Paulo</pre>

Sempre respeitando o conjunto caracteres permitidos em URLs. Para mais informações sobre esses valores, veja [URL Encode](https://www.w3schools.com/tags/ref_urlencode.asp).

Os filtros aplicados podem tratar diversas situaçoes e é importante convencionar o comportamento da API conforme o tipo de filtro que é feito:

| Tipo de dado | Operação | Descrição |
| ------------ | -------- | --------- |
|Numérico, Data e Booleano|Igualdade|Retorna aqueles recursos cujo valor do atributo tenha exatamente o valor especificado. Ex: ...?quantidade=5 devolverá aqueles recursos cujo atributo "quantidade" tenha o valor 5.|
|Numérico, Data e Booleano|Ou|Retorna aqueles recursos cujo valor do atributo esteja contido em uma lista de valores. Ex: ...?quantidade=5&quantidade=9&quantidade=12 retornará aqueles recursos cujo atributo quantidade seja 5, 9 ou 12.|
|Numérico e Data|Maior ou Igual|Retorna aqueles recursos cujo valor do atributo seja maior ou igual o valor definido em “from-quantidade” e menor ou igual o valor definido em "to-quantidade". Ex: ...?from-quantidade=5 retornará os recursos com quantidade maior ou igual a 5. Consulte também [Ranges](#ranges).|
|Texto|Contém|Retorna aqueles recursos cujo valor do atributo contenha o valor especificado. Ex: …?nome=Frederico retornará aqueles recursos que contenham “Frederico” no atributo nome. São retornos válidos válidos: “Frederico Garcia”, “Don Frederico”, “Frederico”.|
|Texto|Ou Contém|Retorna aqueles recursos cujo valor do atributo contenha um dos valores especificados. Ex: nome=Frederico&nome=Antonio retornará aqueles recursos que contenham “Frederico” ou "Antonio" no atributo nome. São retornos válidos válidos: “Frederico Antonio”, “Don Frederico”, “Frederico”, “Antonio Ramirez”.|

De forma resumida, o operador **&** que separa as Query Strings é um **Ou** para os valores de um mesmo atributo e um **E** entre atributos diferentes.

Como as Query Strings geralmente serão atributos dos recursos, utiliza-se o mesmo padrão (camelCase) que os utilizados no body dos recursos.

Geralmente, Query Strings não são utilizadas nos casos em que se busca um recurso cujo ID que já está definido na URL via Path Parameter. Isto porque, normalmente as Query Strings são utilizadas para filtrar dentro de uma coleção de resultados. Quando se tem o ID definido, não temos uma coleção de resultados, mas um específico já especificado pelo cliente.

As query string não são utilizadas somente para filtros, ela pode ser utilizada como parâmetros para paginação, versionamento, ordenação, etc. 

>Existem padrões de mercado para filtrar os recursos via query string como [FIQL](https://tools.ietf.org/html/draft-nottingham-atompub-fiql-00), [OData](https://www.odata.org/getting-started/basic-tutorial/), [GraphQL](https://graphql.org/) e a adoção de uma delas significa agregar mais uma especificação sobre o REST. Assim, em um primeiro momento, utilizar um conjunto padrão, mais reduzido de Query Strings para fazer filtros básicos vai permitir trazer uma grande flexibilidade às REST APIs e ao mesmo tempo entregar uma curva de aprendizado mais rápida àqueles que estão embarcando no padrão. Com a maturidade do time de TI neste assunto, a adoção futura de um desses frameworks ajudará a entregar APIs ainda mais flexíveis.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

### Request > URL > Query Strings > Full text search

Algumas APIs podem implementar a capacidade de buscar em vários atributos ao mesmo tempo. Quando se tem essa capacidade, utiliza-se o query string **q** (de query) com o termo a ser pesquisado.
Ex:
GET http://api.empresarh.com/candidatos?q=Paulo
No caso, o resultado tratá todos os recursos candidatos onde algum atributo pesquisável contém o texto Paulo.

<sub>ir para: [índice](#conte%C3%BAdo) | [response  full text search](#response--body--full-text-search)</sub>
### Request > URL > Query Strings > Paginação

Quando uma API retorna uma lista de resultados, pode ser que esses resultados cheguem a dezenas ou milhares de registros. Na maioria das vezes, as telas das aplicações não exibem todos ao mesmo tempo e quando se trata de API, espera-se respostas rápidas. Por conta disso, é uma boa prática dividir os resultados em blocos. Este processo chamamos de paginação.

Paginando as respostas, a mensagem trafegada fica menor, e por isso, o cliente recebe o resultado da requisição muito mais rápido, além de permitir um uso mais racional e otimizado dos recursos do servidor.

Existem algumas formas de definir exatamente qual "bloco de informação" consultar. Estas formas estão descritas à seguir.

### Request > URL > Query Strings > Paginação > Range

Podemos delimitar a quantidade de resultados à partir da filtragem de um determinado parâmetro, por exemplo, se o parâmetro for **data-nascimento**, a chamada à uma API ficaria assim:

http://api.empresarh.com/candidatos?from-dataNascimento=1985-01-01&to-dataNascimento=2001-12-31

Para atuar como um cursor e filtrar um range de IDs, a chamada ficaria assim:

http://api.empresarh.com/candidatos?from-id=1000&to-id=1099

É uma boa prática adotar padrões para definir a estrutura do parâmetro que trata ranges, como sugestão:
- **"from-" + nome-do-atributo**
- **"to-" + nome-do-atributo**

<sub>ir para: [índice](#conte%C3%BAdo) | [response  range](#response--body--pagina%C3%A7%C3%A3o--range)</sub>
### Request > URL > Query Strings > Paginação > Page e Page Size

A paginação baseada em  **page**  e  **pageSize,**  como o próprio nome já diz, é utilizada através dos parâmetros de número da página a ser navegada e o seu respectivo tamanho (em número de registros).

Ambos são opcionais e caso não sejam definidos na URL, é esperado que a API retorne todos os registros ou retorne na página 1 com a quantidade de registros padrão da API.
Ex:

https://api.exemploclassificados.com/veiculos?page=3&pageSize=30

<sub>ir para: [índice](#conte%C3%BAdo) | [response  page e pageSize](#response--body--pagina%C3%A7%C3%A3o--page-e-page-size)</sub>

### Request > URL > Query Strings > Paginação > Limit

O query string **limit** permite limitar a quantidade de registros que a API retorna à partir do primeiro registro da coleção.

Ex:
https://api.lojaexemplo.com/ofertas-noturnas?limit=10

<sub>ir para: [índice](#conte%C3%BAdo) | [response  limit](#response--body--pagina%C3%A7%C3%A3o--limit)</sub>
### Request > URL > Query Strings > Ordenação

Em APIs que retornem conjuntos de registros, é interessante permitir alguma ordenação básica.
A ordenação pode ser especificada através das query string **sort=[{atributo}:{asc|desc}]**.
Ex:
GET .../pedidos?sort=dataPagamento:desc,dataPedido
No exemplo, desejo que a lista de pedidos venha ordenada de forma decrescente pela dataPagamento e de forma ascendente (valor default) pela dataPedido.

No caso de não ser especificada uma ordem {asc|desc}, será utilizada a ascendente como padrão.

Importante: Quando se define o contrato da API, é importante definir a lista de quais atributos estão disponíveis para ordenação, já quem nem sempre todos eles estarão disponíveis para esse fim.

<sub>ir para: [índice](#conte%C3%BAdo) | [response  ordenação](#response--body--ordena%C3%A7%C3%A3o)</sub>

### Request > URL > Query Strings > Fields

Existem situações onde o cliente deseja obter apenas alguns dos atributos de um recurso. Para estas situações pode-se utilizar o query string **fields=[atributo.sub-atributo]** para selecionar apenas aqueles atributos do recurso que o cliente deseja receber.
Para "sub-atributos",  utiliza-se o "." para separá-los.
Veja o exemplo:

https://api.lojaexemplo.com/clientes/jose-da-silva123/enderecos/residencial-1?fields=nome,situacao,logradouro.rua,logradouro.numero

Com a chamada acima, não terei como retorno qualquer outro atributo de endereço senão os definidos no fields: _nome_, _situacao_, _logradouro.rua_ e _logradouro.numero_

O uso desta opção nos permite otimizar o uso da banda de rede em toda a cadeia de comunicação, reduzir a quantidade de logs gerados e tamanho das respostas a serem processadas, melhorando a experiência final do usuário.

<sub>ir para: [índice](#conte%C3%BAdo) | [response  fields](#response--body--fields)</sub>

### Request > URL > Query Strings > Views

Existem situações em que é comum um recurso ter vários atributos e ter um conjunto deles que é de uso muito comum entre os clientes daquela API. 

Ao invés de fazer com que esses clientes filtrem os atributos pela query string [fields](#request--url--query-strings--fields), fazendo com que a URL da requisição fique muito grande, pode-se criar conjuntos pré defindos de "visões" que retornam apenas alguns sub conjuntos de atributos.
Ex:
- GET .../cartoes/a7834dcG456?view=basico
para retornar o conjunto de atributos relacionados a alguma consulta frequente de dados básicos.
- GET .../cartoes/a7834dcG456?view=limites
para retornar todos os atributos relacionados a alguma consulta frequente de limites.
- GET .../cartoes/a7834dcG456
para retornar todos os atributos.

Dependendo da complexidade do recurso, as visões podem ser combinadas na mesma requisição e ela pode ser usada em conjunto com o [fields](#request--url--query-strings--fields) e [expand](#request--url--query-strings--expand).

<sub>ir para: [índice](#conte%C3%BAdo) | [response views](#response--body--views)</sub>

### Request > URL > Query Strings > Expand

Quando é necessário em uma única chamada retornar um determinado recurso mais os recursos relacionados com ele, pode-se utilizar um query string **expand**. Este mecanismo permite reduzir a quantidade de chamadas à API e volume de informações trafegadas.

Ex:
Tome como referência as seguintes APIs

- GET .../cartoes/{idCartao}
- GET .../cartoes/{idCartao}/faturas/{idFatura}

Para retornar os dados do cartão com id = a7834dcG456 mais o recurso de faturas associado a ele, especificamente a de agosto de 2018,  com o expand, faz-se a seguinte chamada:

GET .../cartoes/a7834dcG456?**expand**=faturas&faturas.id=ago18

Observe que há a definição do recurso a ser expandido (expand=faturas) e também um filtro (faturas.id=ago18), sendo que no filtro, utilizou-se um padrão de recurso+"."+atributo.

Caso vários recursos precisem ser expandidos, define-se separando-os por vírgulas.

Ex: GET .../cartoes/a7834dcG456?expand=faturas,adicionais,ofertas-upgrade

Caso seja necessário especificar paginação ou ordenação em um recurso expandido, deve-se definir estes Query Strings assim como se definem filtros.

Ex:
GET .../cartoes/a7834dcG456?expand=faturas&faturas.limit=5&faturas.sort=dataVencimento:desc

<sub>ir para: [índice](#conte%C3%BAdo) | [response  expand](#response--body--expand)</sub>

### Request > URL > Alias

**Alias** é o conceito de que é possível ter mais de um caminho para fazer a mesma tarefa. Não é ideal, no entanto, muitas APIs nascem com alguns poucos recursos e conforme demandas de negócio, novos recursos vão sendo adicionados. Por falta de visão do todo no início da criação da API, por vezes, é necessário criar outras formas de fazer a mesma tarefa.
No entanto, é importante que uma entidade de negócio (recurso) se mantenha a mesma, independente da forma com que a consulta é feita.
 
Por exemplo, em uma APIs de cartões, o time da TI cria o recurso de busca de transações da seguinte forma:

GET .../transacoes?idCartao=123

Em um segundo momento, novos recursos surgiram e com o amadurecimento, entenderam que a estrutura mais adequada seria essa:

GET .../cartoes/123/transacoes

Ambas implementam a mesma busca, apesar das diferentes abordagens de modelagem.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

### Request > Headers

O header é um dos componentes que fazem parte do protocolo HTTP. Como o REST é baseado neste protocolo, as REST APIs trafegam headers como parte da comunicação. O header basicamente é um conjunto de chaves/valor.

Por padrão, passamos nos headers informações não relacionadas aos recursos expostos nas URLs (que representam as entidades de negócio). De forma análoga, não colocamos atributos técnicos que não representem o negócio dentro dos recursos.

Nos headers trafegamos somente informações técnicas como informações de acesso e credenciais, tokens, chaves de API, Correlation IDs, metadados, etc. 

Alguns headers são padrões do protocolo [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers), outros podem ser definidos de forma personalizada para as necessidades da sua empresa ou por frameworks de mercado que definem conjuntos de headers para necessidades específicas.

Separar este tipo de informação nos headers evita que as entidades de negócio (recursos) trafeguem dados que só existem por conta da comunicação via REST API. 
Por exemplo, se eu estou expondo uma REST API de seguros, informações de token são desconhecidas pelo negócio de seguros e só existem na comunicação por se tratar de REST API. Caso a comunicação acontecesse via webservices, os dados de seguro seriam os mesmos, mas os metadados da comunicação seriam diferentes.

Para mais informações sobre headers, veja: [Segurança](#segurança) e [Cache](#performance-cache-e-compress%C3%A3o).

Dentre os vários headers do protocolo [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers), alguns são mais utilizados no contexto de API e serão explicados a seguir.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

### Request > Headers > Content-Type

O Content-Type é um header que define qual o formato da estrutura de dados presente no Body. Existem muitos tipos de dados, como "text/plain", "application/xml", "text/html", "application/json", "image/gif", "image/jpeg", etc. Quando falamos de REST API, na maior parte das implementações é disponibilizado apenas o "application/json".

Ex: **Content-Type**: application/json

Obs: Não utilizado no DELETE, pois o DELETE não tem Body.

<sub>ir para: [índice](#conte%C3%BAdo) | [response content type](#response--headers--content-type)</sub>

### Request > Headers > Accept

O cliente da REST API pode expressar qual o tipo de informação que ele deseja receber a resposta através do hearder **Accept** . Nota-se que nem todos os content-types são implementados pelos servidores.

Ex: **Accept**: application/json

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

###  Request > Headers > Correlation ID

Não há um header padronizado para Correlation ID, mas é de grande valor que todas as chamadas tenham um Correlation Id. 

Correlation ID é um dado geralmente gerado randomicamente (UUIDs é um bom formato para isso) que deve ser repassado em cada camada de software pelo qual a comunicação trafega. Como cada camada gera o seu próprio log e muitas vezes em banco de dados diferentes, através do Correlation ID, é possível identificar uma chamada específica entre estes diferentes bancos de dados, permitindo um mapeamento da chamada de ponta-a-ponta.

O comportamento esperado de qualquer camada é: se o Correlation ID vier preenchido, repassar para a próxima camada, senão, gerar um novo Correlation ID e repassar. O momento mais adequado para que o Correlation ID seja gerado é no início da cadeia de eventos de uma chamada, normalmente é no cliente quando uma requisição é disparada.

Quando a arquitetura das aplicações está baseada em microsserviço, o Correlation ID percorre uma tragetória mais ampla do que simplesmente algo que inicia no canal e termina na API. Quando um cliente preenche um formulário e clica em gravar, podem ocorrer validações de cartão de crédito, acionamento do sistema de pagamento, comunicação com sistema de envio postal, serviço de e-mails, serviço de geolocalização, etc. sendo cada um desses um microsserviço diferente, com suas camadas de gateway, API, sistema produto, LOGs, etc. Em todos esses sistemas, o mesmo Correlation ID deve ser compartilhado.

Ex:

Uma aplicação client faz uma chamada à API, neste momento ele cria um Correlation ID para esta chamada, acrescenta o no header e grava o seu log na sua estrutura de log com este Correlation ID. Um Gateway de API recebe este Correlation ID e grava o seu log em outra estrutura utilizando o mesmo identificador. O sistema que responde ao produto que está exposto via API recebe o Correlation ID, grava o seu log e chama outro sistema que responde a uma parte da requisição e assim todos os demais envolvidos nesta chamada mantém o mesmo comportamento. 
No momento em que se faz necessário mapear a chamada de ponta-a-ponta, basta unir os logs vindos de diversos fontes através deste identificador.

Quando se define headers que não são padrões do HTTP, muitos utilizam o formato *X-Nome-Header*, no entanto, esta abordagem não é a mais recomendada desde 2012 ([rfc6648](https://tools.ietf.org/html/rfc6648)). No caso, não se usa mais este "X-".

É interessante colocar um prefixo para diferenciar headers específicos da sua API, por exemplo, se sua empresa se chamada "Banco Dourado", os headers que não são padrões HTTP podem ser definidos como BancoDourado-Nome-Header-Customizado.

Se observar a estrutura dos headers padrões ([Headers HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)), as palavras são separadas por hífen e iniciam em maiúsculas.

Assim, para o Correlation ID é interessante assumir o formato **Empresa-Correlation-ID**.
Por exemplo, assumindo o nome da empresa como "Banco Dourado", o header fica:
- **BancoDourado-Correlation-ID**: 680987b5-c18d-4f2f-a772-2a2d422789b1

Recomenda-se que o Correlation ID siga o padrão UUID v4.
>Para conhecer mais sobre UUID, consulte:
https://en.wikipedia.org/wiki/Universally_unique_identifier
https://www.npmjs.com/package/uuid
https://www.uuidgenerator.net/

O Correlation ID também é útil para ser lançado na tela de erro. Por exemplo, como apresentado nesta tela do Office 365:

![Imagem exibindo um erro no Office 365 com o Correlation ID](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/resources/office365errorscreen.png?raw=true)

_Com o Correlation Id, é possível buscar nas bases de log a exata transação que estava acontecendo no momento do erro._

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

###  Request > Verbs

Os **métodos** (ou verbos do protocolo HTTP) são basicamente as ações permitidas dentro de uma API. Estas ações solicitam que seja processado  o CRUD (Create, Read, Update e Delete) nos recursos alterando os seus estados.

Existem vários verbos HTTP ([rfc2616](https://tools.ietf.org/html/rfc2616)), mas 5 são os principais e em muitos casos, os únicos adotados nas APIs. Cada verbo HTTP tem um objetivo bem definido dentro do contexto de REST API. São eles:
- GET: para a obtenção de um recurso ou uma coleção de recursos.
- POST: para a criação de um recurso.
- DELETE: para eliminar um recurso.
- PUT: para atualizações completas de um recurso.
- PATCH: para atualizações parciais de um recurso.

Os verbos são usados na requisição em conjunto com a URL e às vezes com um [Body](#request--body), como nos casos de inserção ou atualização.

Existem também outros verbos como OPTIONS, HEAD e TRACE que raramente são utilizados. Vale a leitura da função desses métodos [aqui](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods).

#### Idempotência e Segurança

Os verbos têm características importantes que devem ser conhecidas e respeitadas, são elas segurança e idempotência:
- **Idempotência** significa que se um cliente realiza um reenvio de uma requisição, o servidor devolve a mesma resposta da vez anterior (salvo se o recurso teve seu estado alterado neste meio tempo). Ou seja, o reenvio da requisição não gera efeito nenhum sobre o recurso.

- **Seguros** são todos os verbos que não podem provocam alterações no estado dos recursos. Ou seja, os verbos apenas de consulta (HEAD, GET, OPTIONS). Destes três, somente o GET é mais utilizado.

Abaixo a tabela exibe a relação entre Verbo HTTP e os conceitos de Idempotência e método Seguro:
 
 | Verbo | Idempotente | Seguro |
 | --- | --- | --- |
|GET|Sim|Sim|
|PUT|Sim|Não|
|DELETE|Sim|Não|
|POST|Não|Não|
|PATCH|Sim|Não|

É importante escolher o verbo correto conforme estas características ao definir uma API, assim como codificar a API respeitando estas regras. Parta do princípio que o cliente da sua API sabe que, por exemplo, o GET é idempotente e seguro. Por isso, ele não vai hesitar em implementar re-tentativas em caso de insucesso na chamada.
Se ao codificar a API, o desenvolvedor da API codificar um GET que não seja seguro ou idempotente, o cliente não terá o comportamento esperado.

###  Request > Verbs > GET

Este verbo é o mais utilizado e serve para buscar dados nas APIs. Ele é utilizado em conjunto com uma URL com seus Path Parameters e/ou Query Strings para enviar uma consulta e o servidor retorna os dados em caso de sucesso. Em requisições do tipo GET, não se envia [Body](#request--body).

Exemplo de utilização de GET para fazer uma consulta por cidades no estado de São Paulo com população maior do que 20000 habitantes:

Com a URL com o formato http://api.exemplo.com/estados/{idEstado}/cidades

**GET** http://api.exemplo.com/estados/sp/cidades?from-populacao=20000

Exemplo de utilização de GET para fazer uma consulta pela cidade de Santos:

Com a URL com o formato http://api.exemplo.com/estados/{idEstado}/cidades/{idCidade}

**GET** http://api.exemplo.com/estados/sp/cidades/santos

### Request > Verbs > POST

O **POST** é usado para criar novos recursos. Ele é utilizado em conjunto com uma URL com seus Path Parameters e um [Body](#request--body) para enviar um conjunto de atributos que represente o estado do novo recurso no momento que você está criando ele.

Exemplo utilizando o verbo POST para criar uma nova cidade:

Com a URL com o formato http://api.exemplo.com/estados/{idEstado}/cidades/{idCidade}

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

Obs: O POST também pode ser usado em casos especiais onde se faz necessário proteger as informações que seriam usadas em uma consulta, pois com o uso de POST, podemos enviar um [Body](#request--body) que segue encriptado, ao contrário do GET que passa os parâmetros na URL que não é encriptada. Ex: login. 

### Request > Verbs > PUT

O verbo  **PUT** atualiza ou cria um recurso, ou seja, se eu utilizar o PUT para enviar novamente a cidade de São Vicente (como no exemplo do POST), o servidor iria sobrepor completamente o recurso com os dados definidos no Body. Portanto, caso algum campo não seja informado, o valor dele será apagado. Caso seja utilizado um PUT e o recurso não exista, a API deveria criá-lo.

Quando usamos **PUT** normalmente estamos atualizando um recurso existente, por isso é importante definir qual é especificamente o recurso através do ID no Path Parameter.

Ex:

Com a URL com o formato http://api.exemplo.com/estados/{idEstado}/cidades/{idCidade}

**PUT** http://api.exemplo.com/estados/sp/cidades/sao-vicente
```
{
   "nome": "São Vicente",
   "DDD": 13,
   "descrição": "Primeira cidade do Brasil.", 
   "populacao": 400000
}
```
No exemplo, passamos dois Path Parameters, o {idEstado} com o valor "sp" e o {idCidade} com o valor "santos".

### Request > Verbs > PATCH

O verbo **PATCH**  serve para fazer atualizações parciais no recurso. Neste caso, ele se comporta de forma semelhante ao PUT, no entanto, define-se no Body apenas os parâmetros que serão alterados.

Ex:

Com a URL com o formato http://api.exemplo.com/estados/{idEstado}/cidades/{idCidade}

*Request*

**PATCH** http://api.exemplo.com/estados/sp/cidades/sao-vicente
```
{
   "populacao": 500000
}
```
No exemplo, a API atualizará apenas o atributo "populacao". Em uma nova chamada usando GET, o valor em "populacao" deverá ser 500000.

### Request > Verbs > DELETE

O verbo **DELETE** é o responsável por deletar os registros. Semelhante ao GET, é usado em conjunto com uma URL com seus Path Parameters e/ou Query Strings para fazer filtro no conjunto que será afetado pelo DELETE.

Caso a requisição seja em um recurso específico, o recurso será deletado, caso seja em uma coleção de recursos, toda a coleção será deletada, caso seja em um filtro (com Query Strings), todos os registros que correspondem ao filtro serão apagados.

Ex:
Com a URL com o formato http://api.exemplo.com/estados/{idEstado}/cidades/{idCidade}
- **DELETE** http://api.exemplo.com/estados/sp/cidades?to-populacao=5000<br>
Apaga todas as cidades com população até 5000 habitantes.
- **DELETE** http://api.exemplo.com/estados/sp/cidades<br>
Apaga todas as cidades do estado de São Paulo.
- **DELETE** http://api.exemplo.com/estados/sp/cidades/sorocaba<br>
Apaga a cidade de Sorocaba.

<sub>ir para: [índice](#conte%C3%BAdo) | [response body](#response--body)</sub>

### Request > Body

O envio de informações com o objetivo de filtrar informações existentes, se dá via Path Parameters e/ou Query Strings.

Quando se utiliza os verbos POST, PUT ou PATCH, estamos enviando informações para serem persistidas no servidor. Neste caso, enviamos as informações no Body.

O Body é o espaço da requisição HTTP onde se trafega as informações referentes ao recurso endereçado pela URL. Ao definir o contrato de uma REST API, cada recurso deve ser declarado como um conjunto de atributos com seus tipos definidos (números, datas, textos, booleanos, etc), descrições, exemplos, obrigatoriedade, etc. O Body trafega o conjunto destes atributos.

Os atributos podem ser definidos em diversos formatos, os mais tradicionais são XML e [JSON]([https://www.json.org/](https://www.json.org/)), sendo este o mais adotado atualmente.

Ex:

POST http://api.fabricacarros.com/carros
```
{
  "modelo": "Kicknegade Freestyle HR AT",
  "cor": "Branco Branquinho",
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
        "id": "filmc60",
        "descricao": "Película protetora UV Cinza 60%"
      }
    ],
  "valorTabela": 83590.00,
  "valorVenda": 82000.00,
  "adaptacaoPDC": false
}
```
Quando se define o contrato da API deve-se atentar para que os atributos presentes do Body sejam relacionados apenas ao recurso definido na URL. Por exemplo, na API de cidades usada em outros exemplos neste guia, quando forem feitas chamadas para "http://api.exemplo.com/estados", devem ser trabalhados apenas atributos que definam um estado; quando forem feitas chamadas para http://api.exemplo.com/estados/{idEstado}/cidades devem ser trabalhados apenas atributos que definam uma cidade.

Ao definir os atributos do contrato da REST API, não existe um consenso de mercado quando ao tipo de caixa a ser adotada (maiúsculo, minúsculo, etc), no entanto, dado que os atributos em algum momento serão associados às propriedades das classes nas linguagens de programação seja no servidor ou no cliente, é uma boa prática adotar o lowerCamelCase, dado que as principais linguagens de programação adotam esta caixa para as propriedades.

Sobre os termos usados para definir os atributos, deve-se utilizar aqueles que melhor auto-descrevam o atributo, reduzindo a necessidade de consultas às documentações. Portanto, utilize as palavras por extenso, evitando abreviações, exceto para casos amplamente conhecidos como "Id" ou acrônimos como "RG". Ainda assim, o padrão para uso de Acrônimos ou Abreviações é controverso. Dê uma lida neste fórum [acronyms-in-camelcase](https://stackoverflow.com/questions/15526107/acronyms-in-camelcase) para comprovar a complexidade deste assunto.

Também não se usa em REST APIs alguns padrões antigos de definição de variável com o tipo de dado no nome do atributo - por exemplo, dt, int, bool, etc. - dado que os atributos tipos já tão tipados na definição do contrato.

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
As linguagens de definição de contrato como Swagger e RAML documentam os data types suportados: [RAML DataTypes](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#raml-data-types) e [Swagger Data Types]([https://swagger.io/docs/specification/data-models/data-types/](https://swagger.io/docs/specification/data-models/data-types/)). E [aqui](#tipos-de-dados) tem um resumo já agregando boas práticas.

<sub>ir para: [índice](#conte%C3%BAdo) | [response body](#response--body)</sub>

## Response

Após o envio de uma requisição à partir de um cliente, o servidor onde a API está localizada deve processar a requisição e gerar uma resposta (Response).

No response, existem alguns padrões a serem seguidos e eles serão explicados um a um a seguir. Alguns são obrigatórios para o funcionamento da REST API, outros são boas práticas adotadas pelo mercado.

<sub>ir para: [índice](#conte%C3%BAdo) | [request](#request)</sub>

### Response > Headers

Assim como na requisição, o retorno da resposta pode trazer um conjunto de headers que são metadados ou informações técnicas sobre a comunicação que está sendo feita através daquela API.

<sub>ir para: [índice](#conte%C3%BAdo) | [request headers](#request--headers)</sub>

### Response > Headers > Content-Type

Assim como na requisição, o header **Content-Type** define qual é o formato da estrutura de dados presente no Body.

Ex:  **Content-Type**: application/json

Obs: Este header não é utilizado no DELETE, pois não é enviado no body de retorno.

<sub>ir para: [índice](#conte%C3%BAdo) | [request content type](#request--headers--content-type)</sub>

### Response > Headers > Content-Location

O header **Content-Location** expõe a URL relativa (somente dos recursos para frente) ou absoluta (desde o início incluindo o Base Path) que expõe um determinado recurso.

Quando uma requisição é feita com o verbo POST, por exemplo, o cliente ainda não sabe o Id do recurso que ele está criando: muitas vezes são identificadores gerados no momento da gravação. Assim, quando o servidor retorna a resposta, além de preencher a propriedade id no body da requisição, deve-se preencher o header Content-Location.

Ex:

*Request*

POST http://api.exemplo.com/estados/sp/cidades/
```
{
   "nome": "São Vicente",
   "DDD": 13,
   "descrição": "Primeira cidade do Brasil", 
   "populacao": 355542
}
```
*Response*

HTTP/1.1 201 Created

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
Obs: Não utilizado no DELETE, pois depois de um DELETE com sucesso, não existe mais o conteúdo.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

### Response > Headers > Location 

O header Location expõe a URL relativa (somente dos recursos para frente) ou absoluta (desde o início incluindo o Base Path) que expõe **outra** localização para um determinado recurso. Normalmente é utilizado em processamento assíncrono.

Ex: **Location**: http://api.exemplo.com/contas/v1/tarefas/1

<sub>ir para: [índice](#conte%C3%BAdo) | [processamento assíncrono](#processamento-ass%C3%ADncrono)</sub>

### Response > Body

Quando é feita uma requisição na API, na maioria das vezes, espera-se uma resposta com informações. Colocamos estas informações no Body. Assim como no Body do [request](#request--body), é recomendado utilizar lowerCamelCase para definir os atributos e JSON como padrão de notação.

A resposta de uma requisição de gravação (POST, PUT, PATCH) não precisa necessariamente retornar o body com o recurso gravado. O critério para a decisão de retorná-lo ou não fica em ponderar se:
- Existe transformação da informação no momento da gravação e o cliente precisa saber, então devolve-se o recurso e gasta-se banda, log, etc.
- Se não existe transformação da informação, então pode-se não enviar um body de retorno e economizar banda, log, etc.

Para DELETE, não se utiliza Body.

Quando não se retorna nada no body de uma requisição que foi processada com sucesso, é importante atentar-se em usar o HTTP Status Code correto. Para GET, utiliza-se 200, para POST 204, PUT, PATCH ou DELETE 204. Leia mais sobre o [HTTP Status Code 204](#response--http-status-codes) para mais informações.
 
Quando são usados [verbos](#request--verbs) que enviam body no request (POST, PUT e PATCH), é importante notar que o body de request e o de response não precisam ser idênticos. Há casos em que a criação de um recurso implica que se populem atributos do mesmo em tempo de criação. Um exemplo muito recorrente é o identificador (id) que normalmente é gerado pelo servidor e não precisa estar definido no body de request.

<sub>ir para: [índice](#conte%C3%BAdo) | [tipos de dados](#tipos-de-dados) | [request body](#request--body)</sub>

### Response > Body > Envelope "Data"

Chamamos de **envelope** alguns **atributos que separam conteúdos importantes na resposta**. Quando nos referimos ao body de request, apenas passamos os atributos sem nenhum tipo de envelope. Ex:
```
{
   "id": 123,
   "nome": "Carlos",
   "data": "2019-06-04"
}
```
No body de response, colocamos a informação do recurso dentro de um envelope "data".  Esse separação é necessária porque no response podemos ter outros envelopes como pagination, messages, links, summary, etc. Exemplo de Body de response:
```
{
   "data": [{
      "id": 123,
      "nome": "Carlos",
      "data": "2019-06-04"
      }],
   "pagination": {
      "page": 1,
      "pageSize": 10,
      "...": "..."
   },
   "_links": {
      "...": "..."
   }
}
```

<sub>ir para: [índice](#conte%C3%BAdo) | [tipos de dados](#tipos-de-dados) | [request body](#request--body)</sub>

### Response > Body > Recurso unitário, array ou nenhum

Quando se faz uma requisição por um elemento com um ID especificado no Path Parameter, por exemplo, GET .../pessoas/{idPessoa}, o retorno da resposta será um único elemento. Assim, coloca-se o recurso unitário diretamente no envelope "data". Ex:

*Request*

GET .../pessoas/456

*Response*

HTTP/1.1 200 OK

Content-Location: .../pessoas/456
```
{
   "data": {
      "id": 456,
      "nome": "José",
      "data": "2019-06-09"
   }
}
```

Quando se faz uma requisição sem o ID, filtrando apenas com Query Strings, pode-se ter como retorno um ou mais elementos. Neste cenário, retornamos com um array do referido recurso (mesmo que só retorne um). Ex:

*Request*

GET .../pessoas?from-data=2019-06-01

*Response*

HTTP/1.1 200 Ok

Content-Location: .../pessoas/456
```
{
   "data": [
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
   ]
}
```

Também é possível obter resposta vazia quando a busca não retorna nenhum resultado. No caso do GET, retorna-se um envelope "data" com o array vazio mais qualquer outro envelope que faça sentido naquela resposta. Ex:
```
{
   "data": [],
   "messages": [
      {
         "...":"..."
      }
   ]
}
```
Veja mais sobre o [HTTP Status Code](#response--http-status-codes) 204 para mais informações. 

<sub>ir para: [índice](#conte%C3%BAdo) | [tipos de dados](#tipos-de-dados) | [request body](#request--body)</sub>

### Response > Body > Full text search

Quando é definido um query string para buscas gerais, o termo definido na busca deve ser aplicado como filtro em todos os campos pesquisáveis daquele recurso. Por exemplo, uma requisição com a seguinte estrutura GET http://api.empresarh.com/candidatos?q=Paulo deveria retornar um array de resultados como este:
```
{
   "data": [
      {
         "id": "87426",
         "nome": "Paulo Ferreira de Araújo",
         "pai": "José de Araújo",
         "mãe": "Patrícia Silva Ferreira",
         "cidade": "Santos",
         "...": "..."
      },
      {
         "id": "87426",
         "nome": "Américo Guedes Oliveira",
         "pai": "Roberto Carvalho de Oliveira",
         "mãe": "Maria José Oliveira",
         "cidade": "São Paulo",
         "...": "..."
      },
      {
         "id": "87426",
         "nome": "Carla Mendes Pinheiros",
         "pai": "Carlos Gosmes de Pinheiros",
         "mãe": "Lívia de Paulo Pinheiros",
         "cidade": "Vitória",
         "...": "..."
      }
   ]
}
```
<sub>ir para: [índice](#conte%C3%BAdo) | [request  full text search](#request--url--query-strings--full-text-search) | [tipos de dados](#tipos-de-dados) | [request body](#request--body)</sub>

### Response > Body > Paginação

Toda REST API que retorne muitos registros deveria suportar paginação. A paginação reduz o tráfego de dados na rede e o tempo de processamento da API entregando listas menores de resultados por vez. Ainda porque, as interfaces que exibem as informações para os usuários costumam ter um espaço limitado, preenchendo gradativamente conforme o usuário "rola" o conteúdo ou "avança a página".

Sempre que se retorna um resultado paginado, utiliza-se o HTTP Status Code **206 Partial Content**. Além do envelope "data" com o resultado da requisição, deve-se criar um envelope "**pagination**" com as informações necessárias para permitir que o cliente se movimente entre as páginas.

Existem algumas técnicas diferentes para fazer a paginação. Serão explicadas abaixo.

#### Response > Body > Paginação > Range

Uma das formas de se limitar a quantidade de registros retornados é através de um filtro em algum atributo que represente um intervalo. Assim, quando se recebe um filtro nas Query Strings, deve-se retornar o resultado respeitando os critérios do filtro.

Por exemplo, em uma requisição GET .../...?from-id=3&to-id=6, possuindo o banco de dados uma coleção de 8 registros, e os ids sendo sequenciais, o retorno devem ser os registros 3, 4, 5 e 6:
```
{
   "data": [
      {
         "id": 3,
         "...": "..."
      },
      {
         "id": 4,
         "...": "..."
      },
      {
         "id": 5,
         "...": "..."
      },
      {
         "id": 6,
         "...": "..."
      }	
   ]
}
```
<sub>ir para: [índice](#conte%C3%BAdo) | [request  range](#request--url--query-strings--pagina%C3%A7%C3%A3o--range)</sub>

#### Response > Body > Paginação > Page e Page Size

Quando se utiliza a abordagem de page e page size, a requisição pode ser feita informando a página e o tamanho dela como query string (GET .../...?page=10&pageSize=50). A API deve recortar do total de respostas apenas as páginas solicitadas conforme solicitação do cliente ou, caso não tenha sido informado, à partir de valores padrões. Por exemplo, em uma requisição GET .../...?page=2&pageSize=3, possuindo o banco de dados uma coleção de 8 registros, o retorno devem ser os registros 4, 5 e 6 mais o envelope "pagination" com seguinte estrutura:

```
{
	"data": [
		{
			"id": 3,
			"...": "..."
		},
		{
			"id": 4,
			"...": "..."
		},
		{
			"id": 5,
			"...": "..."
		}
	],
	"pagination":{
		"first": "/exemplo",
		"last": "/exemplo?page=3",
		"previous": "/exemplo?page=1",
		"next": "/exemplo?page=3",
		"page": 2,
		"totalPages": 3,
		"totalElements": 8
	}
}
```

Os campos são auto-explicativos, só é preciso atenção especial para definição do que é considerado a página inicial, se 0 ou 1. Por convenção, adotar o 1 vai fazer com que a paginação coincida com a paginação que normalmente é exibida na tela para o usuário.

No caso do exemplo, para obtenção da página seguinte, o cliente faria a chamada GET .../...?page=3, conforme informado no atributo "next".

Também devem ser mantidos (repetidos) os Query Strings (filtros, ordenação, etc..) que o cliente passar na requisição, ainda porque, caso o cliente altere o filtro, todo o envelope de paginação pode ter seus valores alterados.

Quando se está na primeira página ou na última, os atributos "previous"e "next" devem ficar vazios.

Muitas vezes, criamos APIs para sistemas legados e com isso, precisamos nos ajustar a comportamentos  já existentes, por conta disso, a paginação pode ter alguns variantes, por exemplo:
- O sistema pode informar a página, não por número, mas através de um ID, nestes casos, basta substituir a informação numérica do  "previous"e "next" por string.
	- Ex: GET .../...?page=fgg12d8bfb4567820c46
- O sistema pode não ter a informação da quantidade total de registros (ex: totalElements e totalPages),  dessa forma, não temos como devolver todas as propriedades. Neste caso, devolvemos apenas as que são possível de serem informadas.

<sub>ir para: [índice](#conte%C3%BAdo) | [request  page e pageSize](#request--url--query-strings--pagina%C3%A7%C3%A3o--page-e-page-size)</sub>

#### Response > Body > Paginação > Limit

Quando se usa o query string **limit** para limitar a quantidade de registros, o retorno deve trazer apenas a quantidade de registros definida no query string. Ex:

GET .../...?limit=3
```
{
   "data": [
      {
         "id": 1,
         "...": "..."
      },
      {
         "id": 2,
         "...": "..."
      },
      {
         "id": 3,
         "...": "..."
      }
   ]
}
```
No caso do exemplo, a API deve retornar apenas os 3 primeiros registros (recursos) da coleção.

<sub>ir para: [índice](#conte%C3%BAdo) | [request  limit](#request--url--query-strings--pagina%C3%A7%C3%A3o--limit)</sub>


### Response > Body > Ordenação

Quando se recebe uma solicitação contendo Query Strings de ordenação (**sort**), deve-se retornar os resultados respeitando os critérios da query.

Ex:

*Request*

GET .../pedidos?sort=dataPagamento:desc,dataPedido

*Response*
```
{
   "data":[
         {
			"id": 456985,
			"dataPagamento": "2019-06-05",
			"dataPedido": "2019-06-02",
			"...": "..."
		},
		{
			"id": 457231,
			"dataPagamento": "2019-06-05",
			"dataPedido": "2019-06-03",
			"...": "..."
		},
		{
			"id": 455125,
			"dataPagamento": "2019-06-01",
			"dataPedido": "2019-05-29",
			"...": "..."
		}
	]
}
```
No exemplo acima, o retorno está ordenado primeiramente de forma decrescente pela dataPagamento, seguido de dataPedido de forma ascendente.
Ordernar de forma ascendente é o comportamento padrão quando a forma de ordenação não for declarada.

<sub>ir para: [índice](#conte%C3%BAdo) | [request  ordenação](#request--url--query-strings--ordena%C3%A7%C3%A3o)</sub>

### Response > Body > Fields

Quando uma requisição vem com a Query String **fields**, o body de retorno deve trazer apenas os atributos definidos na Query String.

Ex:

*Request*

GET https://api.lojaexemplo.com/clientes/jose-da-silva123/endereços/residencial-1?fields=nome,situacao,logradouro.rua,logradouro.numero

*Response*

HTTP/1.1 200 OK
```
{
   "data": {
      "nome": "Residencial",
      "situacao": "ativo",
      "logradouro": {
         "rua": "Rua das Oliveiras",
         "numero": "123"
      }   
   }
}
```

Enquanto uma chamada ao mesmo recurso sem o fields, deve retornar o recurso com todos os seus atributos.

Ex:

*Request*

GET 
https://api.lojaexemplo.com/clientes/jose-da-silva123/endereços/residencial-1

*Response*

HTTP/1.1 200 OK
```
{
   "data": {
      "nome": "Residencial",
      "situacao": "ativo",
      "dataAtualizacao": "2019-07-17T19:27:00-03:00",
      "logradouro": {
         "cep": "11001253"
         "rua": "Rua das Oliveiras",
         "numero": "123",
         "complemento": "Apto 18",
         "cidade": "Santos",
         "uf": "São Paulo"
      }   
   }
}
```

<sub>ir para: [índice](#conte%C3%BAdo) | [request fields](#request--url--query-strings--fields)</sub>

### Response > Body > Views

Quando a requisição recebe o query string **view**, o response deve devolver apenas os atributos convencionados (e documentados) como pertencentes àquela view.

Ex:

- GET .../cartoes/a7834dcG456?view=basico

*Response*
```
{
   "data": {
      "id": "a7834dcG456",
      "status": "válido",
      "produto": "Platinum",
      "bandeira": "mastercard",
      "numero": "5461********7965"
  }
}
```
- GET .../cartoes/a7834dcG456?view=limites

*Response*
```
{
   "data": {
      "id": "a7834dcG456",
      "status": "válido",
      "limites": {
         "contratado": 8000,
	     "usado": 2500,
	     "disponivel": 5500
      }
   }
}
```
- GET .../cartoes/a7834dcG456

*Response*
```
{
   "data": {
      "id": "a7834dcG456",
      "status": "válido",
      "produto": "Platinum",
      "bandeira": "mastercard",
      "numero": "5461********7965",
      "dataMelhorCompra": {
         "data": "2016-02-28",
         "quantidadeDiasParaPagar": 12
      },
      "limites": {
         "contratado": 8000,
         "usado": 2500,
         "disponivel": 5500
      }
   } 
}
```
<sub>ir para: [índice](#conte%C3%BAdo) | [request  views](#request--url--query-strings--views)</sub>

### Response > Body > Expand

Quando a requisição traz um query string **expand**, o body deverá retornar o recurso definido na URL mais os recursos definidos no expand aninhados no recurso principal.

Ex:

Sendo uma API com o formato

GET .../cartoes/{idCartao} e 

GET .../cartoes/{idCartao}/faturas/{idFatura}

*Request*

GET .../cartoes/a7834dcG456?expand=faturas&faturas.id=ago18

*Response*
```
{
	"data": {
	  "id": "a7834dcG456",
	  "status": "válido",
	  "produto": "Platinum",
	  "bandeira": "mastercard",
	  "numero": "5461********7965",
	  "dataMelhorCompra": {
	    "data": "2016-02-28",
	    "quantidadeDiasParaPagar": 12
	  },
	  "limites": {
		  "contratado": 8000,
		  "usado": 2500,
		  "disponivel": 5500
	  },
	  "faturas": [
		  {
			  "id": "ago18",
			  "valorMinimo": 178.96,
			  "valorTotal": 726.31,
			  "dataVencimento": "2019-07-17"
		  }
	  ]
	}
}
```
Observe que o conteúdo até o atributo limites são atributos do recurso cartão, no atributo faturas, é retornado o array de faturas como se tivesse havido uma chamada ao recurso .../cartoes/a7834dcG456/faturas/ago18.

<sub>ir para: [índice](#conte%C3%BAdo) | [request  expand](#request--url--query-strings--expand)</sub>

### Response > Body > Errors e Warnings

#### Errors
Quando ocorrem requisições cujo retorno seja um erro ([HTTP Status Code](#response--http-status-codes) 5xx e 4xx), o body deve retornar o detalhamento do erro. Neste caso, não são retornados os envelopes relacionados às requisições bem sucedidas como "data", "pagination", "links", etc.
O detalhamento do erro, é algo mais do que simplesmente o que o HTTP Status Code já expressa por si, mesmo. Ele deve ser suficiente para que o cliente entenda o que aconteceu e saiba o que fazer diante daquele problema.

Ex:

HTTP/1.1 422 Unprocessable Entity
```
{
   "codigo": "ER0059",
   "mensagem": "Operação não permitida fora do horário comercial.",
   "detalhes": "https://developer.empresa.com/apis/cartoes/erros/ER0059"
}
```
HTTP/1.1 428 Preconditional Required
```
{
   "codigo": "err-pto-A320",
   "mensagem": "Não é possível fazer o lançamento de horas extras sem pré-aprovação do gerente.",
   "detalhes": "https://developer.empresa.com/apis/rh/erros/err-pto-A320"
}
```
Quando a API retorna HTTP Status Code 400 e 422, muito provavelmente o erro foi causado por algum atributo específico. Nestes casos, deve-se especificar as informações sobre cada um dos atributos envolvidos no erro.

HTTP/1.1 Status Code 400 Bad Request
```
{
   "codigo": "10023",
   "mensagem": "Alguns campos estão preenchidos incorretamente.",
   "detalhes": "https://developer.empresa.com/apis/erros/10023",
   "atributos":[
      {
         "nome":"dataPedido",
         "mensagem":"O formato de data é inválido. Utilize data no padrão yyyy-MM-DD.",
         "valor":"01-05-2019",
         "detalhes": "https://developer.empresa.com/apis/erros/err-gen-086"
      },
      {
         "nome":"valorPagamento",
         "mensagem":"Valor não é number.",
         "valor":"R$ 27.568,90",
          "detalhes": "https://developer.empresa.com/apis/erros/err-gen-073"
      }
   ]
}
```

#### Warnings

Durante requisições com retorno de sucesso ([HTTP Status Code](#response--http-status-codes) 2xx e 3xx) pode haver situações que um alerta deve ser enviado ao cliente. Por exemplo, alertar que uma transação no cartão de crédito atingiu o 80% do limite disponível ou até mesmo, tendo atingido o limite do cartão, alertar que será cobrada uma taxa pelo uso do limite emergencial.

O alerta se dá através de um envelope **messages** com a estrutura semelhante às das mensagens de erro.

Ex:

HTTP/1.1 201 Created
```
{
   "data":{
      "...": "..."
   },
   "messages": [
      {
         "codigo":"CT0056",
         "mensagem":"Limite do cartão ultrapassou 80%."
      }
   ]
}
```
O atributo "atributos" não é obrigatório e neste cenário, no entanto, nos casos que fizer sentido, informá-los pode ajudar o cliente a entender mais especificamente o que provocou o alerta.

<sub>ir para: [índice](#conte%C3%BAdo) | [http status codes](#response--http-status-codes)</sub>

### Response > Body > HATEOAS

**H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate (HATEOAS) é uma dos pilares da arquitetura REST pela qual um cliente pode interagir com a API através de links informados pelo servidor. Assim, simplesmente fazendo requisições aos recursos, o cliente vai "descobrindo" as opções disponíveis sem ter de estudar uma documentação, proporcionando uma maneira de fazer os protocolos auto-documentados.

A implementação é a disponibilização de um envelope **links** na resposta de um recurso.

Por exemplo, em um serviço de consulta de cartões, podemos informar ao cliente que ele também pode consultar as transações, faturas e contratar novos cartões adicionais.

Ex:
```
{
   "data": {
      "id": "a7834dcG456",
      "status": "válido",
      "produto": "Platinum",
      "bandeira": "mastercard",
      "numero": "5461********7965",
      "dataMelhorCompra": {
         "data": "2016-02-28",
         "quantidadeDiasParaPagar": 12
      }
   },
   "links":[
      {
         "rel": "faturas",
         "href": "/cartoes/a7834dcG456/faturas",
         "title": "Retorna todas as faturas",
         "method": "GET"
      },
      {
         "rel": "adicionais",
         "href": "/cartoes/a7834dcG456/adicionais",
         "title": "Retorna a lista de cartões adicionais",
         "method": "GET"
      },
      {
         "rel": "upgrade",
         "href": "/cartoes/a7834dcG456/ofertas-upgrade",
         "title": "Retorna a lista de ofertas para fazer upgrade do cartão.",
         "method": "GET"
      }
   ]
}
```
Apesar de existir propostas para o formato da declaração dos hyperlinks como o [HAL](https://en.wikipedia.org/wiki/Hypertext_Application_Language), não há uma rigidez quanto à adoção do padrão. Por simplicidade de facilidade de entendimento, o padrão do exemplo expõe bem os links relacionados.

Com o uso de HATEOAS, o cliente pode implementar funcionalidades na tela, conforme resposta do servidor, por exemplo, associando exibição ou não de itens de menu ao retorno ou não de links no HATEOAS. Assim, como, permite que o servidor altere o caminho das URLs de forma dinâmica, sem impactar o cliente.

A adoção do HATEOAS atinge o nível mais alto no [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html).

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

### Response > HTTP Status Codes

No HTTP existem os [códigos de status](#[https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)). Eles, de forma padronizada, reportam se a requisição foi processada com sucesso ou não. Existem vários HTTP Status Codes e nem todos são adotados pelo mercado para uso com REST.

Basicamente, os códigos de resposta são agrupados: 1xx, 2xx, 3xx, 4xx e 5xx. Abaixo, seguem os mais usuais.

**Grupo 1xx**

Este é o grupo de status que dá respostas informativas. Respostas com este status são raramente utilizadas. Por isso, não listo nenhum deste grupo aqui.

**Grupo 2xx**

Os códigos deste grupo são usado em caso de sucesso na requisição. Os mais utilizados são:

- **200 OK**:  Código mais utilizado e que indica que a requisição foi processada com sucesso. Esta resposta pode ser usada em todos os verbos. Os dados solicitados serão retornados no Body.

- **201 Created**:  Indica que a requisição foi bem sucedida e que um novo recursos foi criado como resultado. Esta resposta é utilizada em requisições do método POST.

- **202 Accepted**:	O recurso será atualizado/criado de forma assíncrona. Veja [processamento assíncrono](#processamento-ass%C3%ADncrono) para mais detalhes.

- **204 No Content**: A requisição aconteceu com sucesso, no entanto, não há body na resposta. O 204 não é utilizado para o verbo GET. Nos casos de GET cujos critérios na requisição provocaram uma resposta vazia, utilizamos o HTTP code 200 com o envelope "data" vazio.

- **206 Partial Content**: O servidor encontrou uma uma grande quantidade de registros na respostas e devolveu de forma paginada. A resposta inclui um envelope "pagination" com informações sobre a paginação.

**Grupo 3xx**

Este grupo define respostas de redirecionamento. Servem para informar o cliente sobre mudanças na requisição e redirecionamento para uma nova URL. Para saber mais sobre estes Status Codes, veja [requisições assíncronas](#processamento-ass%C3%ADncrono). Os mais utilizados são:

- **301 Moved Permanently**: Informa que o recurso foi definitivamente movido para uma outra URL definida no header **Location**.

- **303 See Other**: A resposta para a requisição encontra-se em outra URL definida no header **Location**. Veja [processamento assíncrono](#processamento-ass%C3%ADncrono) para mais informações.

- **304 Not Modified**:  Resposta utilizada quando o recurso está em [cache](#performance-cache-e-compress%C3%A3o), informando ao cliente que a resposta não foi modificada, e que o cliente pode usar a mesma versão em cache da resposta.

- **307 Temporary redirect**:  Se trata de um redirecionamento de uma página para outro endereço, porém que é com caráter temporário, e não permanente. Provavelmente por conta de alguma manutenção no sistema.

**Grupo 4xx**

Esse grupo informa os erros cometidos pelo cliente durante o request. São eles:

- **400 Bad Request**: Significa que o servidor não consegue entender a requisição, pois existe uma sintaxe ou estrutura inválida, pode ser caracteres não permitidos na URL, falta de cabeçalhos obrigatórios, cabeçalhos mal formados, falta de Query Strings obrigatórias, falta de atributos obrigatórios, body com estrutura inválida, etc.

- **401 Unauthorized**: A camada de segurança do recurso solicitado ao servidor, apontou que não está sendo utilizada as credenciais corretas nessa requisição (token, por exemplo). É um erro de [autenticação](#seguran%C3%A7a).

- **403 Forbidden**: As credenciais (token) estão corretos, mas o usuário não tem permissão para acessar aquele recurso. É um erro de [autorização](#seguran%C3%A7a).

- **404 Not Found**: O servidor não encontrou o recurso solicitado pelo cliente. Provavelmente a URL está mau formada ou está sendo feita a busca com um Path Parameter inválido.
Ex: PUT .../cartoes/123 devolve 404, caso o recurso cartão com id = 123 não exista.
 
- **405 Method Not Allowed**: O recurso (URL) existe mas o verbo usado não foi definido para ela.

- **410 Gone**: O recurso (URL) não existe mais e esta condição é permanente. Este status é usado quando um recurso que um dia existiu não existe mais. Ao contrário do 404 que em que o recurso pode nunca ter existido ou ele está temporariamente indisponível.

- **414 URI Too Long**: O tamanho da URL pode ser limitado pelo servidor. Assim, este erro é lançado quando uma URL ultrapassa o tamanho máximo permitido. [+ info](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/414)

- **418 I'm a teapot**: O servidor se recusa a servir um café porque o bule é chá. Código de resposta do Hyper Text Coffee Pot Control Protocol ([HTCPCP/1.0](https://tools.ietf.org/html/rfc2324#section-2.3.2)).  Este código é só uma brincadeira de primeiro de abril. :-) [+ info](https://sitesdoneright.com/blog/2013/03/what-is-418-im-a-teapot-status-code-error)

- **422 Unprocessable Entity**:	Ocorre quando a requisição está correta ao nível sintático, mas existem erros de negócio na requisição. Por exemplo, se existe regra que o uso de um query parameter está condicionado a outro e eles não foram preenchidos, ou uma data informada é inválida para uma determinada ação, ou uma requisição de transferência financeira é feita e a conta não tem fundo, etc.

- **428 Precondition Required**: O recurso faz parte de um processo composto de vários passos. Foi feita uma tentativa de acessar um passo sem antes ter passado pelo passo anterior.

- **429 Too Many Requests**: Informa ao cliente que ele excedeu o limite permitido de requisições. Leia sobre [segurança](#seguran%C3%A7a) para entender mais sobre este código de retorno.

**Grupo 5xx**

São códigos que retornam erros que aconteceram por culpa do servidor. Ou seja, a requisição foi feita corretamente pelo cliente, porém ocorreu um erro no servidor. São eles:

- **500 Internal Server Error**: Erro mais genérico para informar que o servidor encontrou um cenário inesperado de erro que não soube tratar, e por isso não conseguiu retornar uma resposta na requisição do cliente.

- **501 Not Implemented**: O verbo HTTP não foi disponibilizado na API ou a URL informada não existe, não por conta de Path Parameters inválidos, mas por conta de ter um recurso ainda não implementado. Normalmente isto acontece quando existe a previsão de se fazer a implementação em breve, mas versão atual ainda não suporta.

- **503 Service Unavailable**: O servidor não está respondendo por que está fora do ar, em manutenção ou sobrecarregado. É um problema temporário.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Tipos de dados

No body da requisição, trafegam-se dados estruturados. No JSON (notação mais usada para isso), são definidos os seguintes tipos de dados: string, number, object, array, boolean, null. Datas, por exemplo, são trafegadas como strings, logo é necessário adotar alguns padrões para o formato delas, geralmente respeitando ISOs.

Abaixo, seguem algumas formatações padrões para os tipos de dados:

| Tipo de dado	| Descrição	 | Exemplos |
| --- | --- | --- |
| Date | String com formato ISO 8601.<br> - ano (YYYY),<br>- ano e mês (YYYY-MM)<br>- ano, mês e dia (YYYY-MM-DD) | 2016<br>2016-02<br>2016-02-26|
| Timestamp | String com formato [ISO 8601](https://www.w3.org/TR/NOTE-datetime)<br>(YYYY-MM-DDThh:mm:ss.sTZD)|2019-02-21T19:00:00Z (para UTC)<br>2019-02-21T19:21:32.285Z (com milissegundos)<br>2019-02-21T19:00:00-03:00 (para UTC-3)<br>ou 2019-02-21T19:00:00+01:00 (para UTC+1)<br>|
| Object | Objetos que definem conjuntos de atributos | {<br>&nbsp;"documento": {<br>&nbsp;&nbsp;"tipo": "rg",<br>&nbsp;&nbsp;"numero": "443335556"<br>&nbsp;}<br>}|
| String | Cadeia de texto |“texto de exemplo”|
| Number | Valores numéricos inteiros e decimais com separação de decimal usando "." (ponto) | 1.25 |
| Boolean | Define os valores true ou false | true |
| Array | Lista de objetos de um dos tipos anteriores | ["a", "b", "c"]
| null	| Valores nulos | null |
| Moeda | String com formato [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) | EUR (Euro Member Countries)<br>USD (United State Dollar)<br>BRL (Brazillian Real)|
| Idiomas | String com o formato [ISO 693(https://en.wikipedia.org/wiki/Lists_of_ISO_639_codes) |por (Portuguese)<br>eng (English)<br>spa (Spanish) |
| Países | String com o formato [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166)|BR (Brasil)<br>PT (Portugal)

<sub>ir para: [índice](#conte%C3%BAdo) | [body](#request--body)</sub>

## Processamento Assíncrono

O HTTP é um protocolo síncrono, logo, quando um cliente faz uma requisição, ele recebe uma resposta e encerra-se o ciclo. 

No entanto, em determinadas situações os servidores não processam as requisições de forma imediata, seja porque o processamento necessita de mais tempo do que o habitual, ou porque estará esperando que chegue a sua vez para ser executado, ou ainda porque depende de um agendamento batch para completar a operação.

Assim, quando sistemas deste tipo são expostos via API, o processamento segue alguns passos a mais:
1. Na primeira requisição, o cliente receberá como resposta um HTTP Status Code **202 - Accepted**. Ou seja, a requisição foi aceita, mas ainda não foi processada. E será informado no header [Location](#response--headers--location) uma URL onde é possível consultar o andamento deste processamento.

Ex:

*Request*

POST  http://api.exemplo.com/contas/v1/contas
```
{
	"agencia": 2153,
	"cliente": "José da Silva",
	"...": "..."
}
```
*Response*

HTTP/1.1 202 Accepted

Location: http://api.exemplo.com/contas/v1/contas-processamento/1

2. Na segunda requisição, o cliente deverá consultar a URL informada no Location para acompanhar o andamento da requisição.

Ex:

*Request*

GET http://api.exemplo.com/contas/v1/contas-processamento/1

*Response*

HTTP/1.1 200 Ok
```
{
   "data":{
      "id":"1",
      "situacao": "processando",
      "TTC": "2019-07-16T19:20:30.00-03:00",
      "mensagens": [
         {
		   "codigo": "103",
		   "mensagem" : "Sua requisicao está esperando pela verificação de um operador.",
		   "tipo": "info"
         }
      ]
   }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sendo:
- **id**: o identificador do processamento;
- **situacao**: [processando, sucesso, falha] indica qual é a situação do processamento;
- **TTC**: **T**ime **t**o **c**ompletion indica quando é estimado que o processamento termine. Nesta hora, é o momento de fazer uma nova requisição para checar a situação do processamento.
- **mensagens**: um array de mensagens com detalhes sobre o processamento;
- **mensagens.codigo**: identificador sistêmico da mensagem;
- **mensagens.mensagem**: mensagem para descrevendo detalhes do processamento;
- **mensagens.tipo**: [info, aviso, erro] o tipo de informação na mensagem.

3. Em algum momento, o processamento estará completo, seja com erro ou não. Em caso de sucesso, a resposta será um HTTP Status Code 303 See Other que indicará que o recurso foi criado e está na URL indicada no Location.

Ex:

*Request*

GET http://api.exemplo.com/contas/v1/contas-processamento/1

*Response*

HTTP 303 See Other

Location: http://api.exemplo.com/contas/v1/contas/21531234567

Content-Location: http://api.exemplo.com/contas/v1/contas-processamento/1
```
{
	"data":{
	  "id":"1",
	  "situacao": "sucesso",
	  "TTC": null,
      "mensagens": [
		 {
		   "codigo": "001",
		   "mensagem" : "A conta foi criada com sucesso.",
		   "tipo": "info"
		  }
		]
	}
}
```
4. Agora já é possível fazer a requisição no endereço do Location http://api.exemplo.com/contas/v1/contas/21531234567 e consultar o recurso criado.

Ex:

*Request*

GET http://api.exemplo.com/contas/v1/contas/21531234567

*Response*

HTTP/1.1 200 Ok
```
{
   "data":{
      "id": 21531234567,
      "agencia": 2153,
      "conta": 123456,
      "dac": 7,
      "segmento": "premium",
      "cliente": "José da Silva",
      "...": "..."
   }
}
```

Caso a situação no retorno no passo 2 seja falha,  deve-se retornar o motivo e o processamento é encerrado.

Ex:

*Request*

GET http://api.exemplo.com/contas/v1/contas-processamento/1

*Response*

HTTP/1.1 200 Ok

Content-Location: http://api.exemplo.com/contas/v1/contas-processamento/1
```
{
   "data":{
      "id":"1",
      "situacao": "falha",
      "TTC": null,
      "mensagens": [
         {
            "codigo": "ERR135",
            "mensagem" : "A conta não foi criada por conta de restrição de crédito.",
            "tipo": "erro"
         }
      ]
   }
}
```

Ao término do processamento:
- o cliente poderia fazer um DELETE na URL de processamento;
- e/ou o servidor pode remover automaticamente os dados de processamento após um determinado tempo e retornar um HTTP Status Code 410 Gone para quem chamar novamente a URL do processamento;
- ou o servidor pode não implementar DELETE e manter para sempre as informações sobre o processamento.

#### Callbacks

Além da possibilidade do cliente fazer consultas recorrentes no passo 2 para verificar o andamento do processamento, pode-se optar por fazer callback do servidor para o cliente quando a requisição estiver terminada. Neste caso, o cliente deverá chamar o recurso cujo processamento se dá assincronament informando o header **[Empresa]-Callback**.

Ex:

*Request*

POST  http://api.exemplo.com/contas/v1/contas

EmpresaExemplo-Callback: http://api.clienteexemplo.com/contas-callback

*Response*

HTTP/1.1 200 OK

Location:  http://api.exemplo.com/contas/v1/contas-processamento/1
```
{
   "data":{
      "id":"1",
	  "situacao": "processando",
	  "TTC": "2019-07-16T19:20:30.00-03:00",
      "mensagens": [
		 {
		   "codigo": "103",
		   "mensagem" : "Sua requisicao está esperando pela verificação de um operador.",
		   "tipo": "info"
		  }
		]
	}
}
```

Neste caso, o cliente pode optar por não chamar mais a API de processamento para verificar o andamento, pois o servidor enviará a resposta via POST para a URL informada pelo cliente (http://api.clienteexemplo.com/contas-callback) ao término do processamento. No Body enviado para o cliente, deverá conter todas informações referentes ao processamento.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Processamento em lotes

Hoje existem ferramentas que permitem alta performance para atender grandes volumes de requisições online, sem a necessidade de precisar acumular requisições para processar de uma só vez. Quando se fala em REST API, normalmente busca-se este cenário de processamento em tempo real. No entanto, principalmente em REST APIs de uso interno, existem situações onde o processamento é feito em lotes. Neste cenário, pode-se seguir o seguinte padrão:

1.	O cliente define um array de objetos, sendo cada deles um request HTTP declarado com todos os seus componentes (incluindo URL, verbos e headers);
2.	O cliente submete o array mensagem para o servidor usando o verbo POST para um recurso de API preparado para receber a requisição do passo 1;

Ex:

*Request*

POST .../batch

Content-Type: application/json
```
{
"requests":[
      {
         "method": "PUT",
         "url": "http://api.exemplo.com/recurso/123",
         "headers": [
            {"name": "Content-Type", "value": "application/json"}
         ],
         "body": {
            "...": "..."
         }
      },
      {
         "method": "POST",
         "url": "http://api.exemplo.com/recurso",
         "headers": [
            {"name": "Content-Type", "value": "application/json"}
         ],
         "body": {
            "...": "..."
         }
      }
   ]
}
```
3.	O servidor quando receber a requisição, deve processar o array separando cada item do array e despachando-os para as respectivas APIs que fazem o processamento individual de cada item.
	- Alternativamente, o servidor pode ignorar o envio para cada API e processar diretamente a lista de requests;
4. O servidor coleta a resposta de cada request e devolve para o cliente através de um array respeitando a sequência da requisição. 

*Response*

HTTP/1.1 200 OK

Content-Type: application/json

```
{
"responses":[
      {
         "httpStatus": "200",
         "message": "OK",
         "headers": [
            {"name": "Content-Type", "value": "application/json"},
            {"name": "Content-Location", "value": "http://api.exemplo.com/recurso/123"}
         ],
         "body": {
            "...": "..."
         }
      },
      {
         "httpStatus": "412",
         "message": "Precondition Failed",
         "headers": [
            {"name": "Content-Type", "value": "application/json"},
            {"name": "Content-Location", "value": "http://api.exemplo.com/recurso"}
         ],
         "body": {
            "...": "..."
         }
      }
   ]
}
```
<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Recursividade

Existem casos em que os recursos da API podem se aninhar recursivamente.
Por exemplo, uma cadeia societária:

- **JOSÉ** é sócio de:
	- Maria que é sócia de:
		- Empresas Azul que tem como sócios:
			-  Carlos
             - João
			 - Empresas Amarelas que tem como sócios:
				- Empresas Verdes que tem como sócios:
				- **JOSÉ**
				- Lisa
				- Renata

Analisando este cenário simples, já podemos concluir que se criarmos uma estrutura com os sócios "aninhados" entre eles, não temos como saber de forma genérica qual é a profundidade das ramificações possíveis, logo, fica difícil definir em um contrato de REST APIs esses objetos aninhados.

Por exemplo, estruturando os sócios em URLs como recursos, teríamos o cenário:
.../empresas/abc123Xyz/**socios**/JOSÉ/**socios**/Maria/**socios**/EmpresasAzul/**socios**/...

Esta abordagem traz as seguintes dificuldades:
- todos os sócios da rota é a entidade sócio, então ao tentar acessar os Path Parameters, teremos vários com o mesmo identificador "idSocio", ficando menos óbvia a recuperação destas informações no servidor.
- não é possível definir previamente a quantidade de aninhamentos (recursos do tipo sócio)
- um mesmo recurso (o sócio id JOSÉ) poderia estar representado em duas rotas diferentes, ora como sócio da empresa "abc123Xyz", ora como sócio do sócio "Empresas Verdes". O ideal no REST é que o recurso seja acessado apenas em uma rota.

Caso a opção fosse definir estes sócios não como recursos, mas como estruturas aninhadas dentro de um único recurso "sócio", teríamos algo assim:

*Request*

GET .../empresas/abc123Xyz/**socios**/JOSE

*Response*

HTTP/1.1 200 OK

```
{
   "data":[
      { 
         "id": "JOSE", 
         "outrosDados": "...",
         "socios": [
            {
               "id": "MARIA", 
               "outrosDados": "...",
	           "socios": [
                  {
                     "id": "EMPRESASAZUL", 
                     "outrosDados": "...",
                     "socios": [
                        {
                           "id": "CARLOS", 
                           "outrosDados": "...",
                           "socios": [
                              {
                                 "outrosSocios": "..."
                              }
                           ]
                        },
                        {
                           "id": "JOAO", 
                           "socios": [
                              {
                                 "outrosSocios": "..."
                              }
                           ]
                        }
                     ]
                  }
               ]
            }
         ]
      }
   ]
} 
```

Esta abordagem traz as seguintes dificuldades:
- Caso seja necessário trocar o telefone do sócio Maria, por exemplo, não haveria uma URL para fazer um PATCH diretamente no recurso. Seria necessário enviar o .../socios/JOSE completo com todos os seus aninhamentos para alterar uma informação pequena.
- Não temos como saber previamente a quantidade de sócios para aninhar e definir no contrato do recurso (JSON) que representa essa entidade de socios.

### Solução

Para contornar os problemas explicados acima, podemos separar o recurso sócio do relacionamento entre eles. Ex:

Recurso **sócio**:

*Request*

GET .../empresas/abc123Xyz/**socios**/JOSE

*Response*
```
{
   "data": {
      "id": "JOSE",
      "nome": "José",
      "cpf": "11122233344",
      "...": "..."
   }
}
```
Recurso **associacoes** (unitário):

*Request*

GET .../empresas/abc123Xyz/**associacoes**/123

*Response*
```
{
   "data": {
      "id": 123,
      "idSocio": "MARIA",
      "associacaoPai": "JOSE"
	}
}
```

Recurso **associacoes** (array):

*Request*

GET .../empresas/abc123Xyz/**associacoes**

*Response*
```
{
   "data": [
      {
         "id": 123,
         "idSocio": "MARIA",
         "associacaoPai": "JOSE"
      },
      {
         "id": 124,
         "idSocio": "EMPRESAAZUL",
         "associacaoPai": "MARIA"
      },
   ]
}
```

Dessa forma, deixando a associação do recurso "sócios" em uma rota separada, a implementação fica em um nível de granularidade que também reduz problemas de concorrência, pois cada associação pode ser adicionada ou removida independentemente das outras associações do mesmo sócio.
As buscas nestas rotas seguem as mesmas práticas já explicadas neste guia. Ex:
- GET .../empresas/abc123Xyz/socios
*(traria um array com todos os sócios da empresa abc123Xyz)*
- GET .../empresas/abc123Xyz/associacoes/123
*(traria uma associação específica em que já conhecemos o ID)* 
- GET .../empresas/abc123Xyz/associacoes?id_socio=CARLOS
*(traria as associações do sócio CARLOS)*
- GET .../empresas/abc123Xyz/associacoes?associacaoPai=MARIA
*(traria as associações de sócios cujo "sócio pai" seja o sócio MARIA)*
- DELETE .../empresas/abc123Xyz/associacoes/123
*(apagaria a associação com o id 123)*
- DELETE .../empresas/abc123Xyz/associacoes?associacaoPai=MARIA
*(apagaria a associação cujo pai é o sócio MARIA)*

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Versionamento

Versionamento acontece quanto ocorrem alterações no contrato da API. Contrato é a definição de todo o conjunto de verbos, códigos de resposta, recursos, etc. feita em uma notação padrão para isso como o RAML, Open API Specification, etc. O ideal é que estes documentos sejam armazenados em um repositório de código fonte e a cada alteração, versionados. Assim, existirão dois tipos de versão: aquela do contrato no repositório e aquela que repassamos para o cliente (consumidor da API).

Para realizar o versionamento dos contratos das APIs podemos fazer uso do [Semantic Versioning 2.0.0](https://semver.org/). Esta definição utiliza a seguinte estrutura **MAJOR.MINOR.PATCH**. Onde se incrementa os valores conforme a seguinte regra:
- **MAJOR**: implica mudanças incompatíveis para a API. Ou seja, que faz com que os consumidores atuais não consigam mais utilizar a API sem ter de alterar seus softwares.
Ex (era **1**.0.0 e vira **2**.0.0):
	- remoção de qualquer item que faz parte do HTTP e do REST (Status Code, Verbo, Recursos, Parâmetros, Atributos, etc);
	- tornar campos já existentes - antes não obrigatórios - como obrigatórios;
	- alteração de tipos de dados.
- **MINOR**: implica adicionar funcionalidades que mantém compatibilidade com a versão atual. Ou seja, os consumidores atuais não terão que alterar seu software para continuar usando a API. 
Ex (era 2.**0**.0 e vira 2.**1**.0):
	- adição de novos recursos;
	- em recursos já existentes, adição de novos atributos ou parâmetros não obrigatórios na requisição;
	- adição de novos atributos no response;
	- adição de novos verbos;
	- adição de novos Status Codes;
- **PATCH**: implica em mudanças que não alteram a API.
Ex (era 2.0.**0** e vira 2.1.**1**):
	- alteração nas descrições dos campos;
	- alterações nos exemplos;
	- alterações em arquivos extras ao contrato que contém metadados usados por governança ou deploy.

A versão incial do contrato da API é a 1.0.0. O valor definido no MAJOR, no caso **1** é o valor que mostramos para o cliente, dado que é a alteração no MAJOR que obriga-o a alterar o seu software.

Existem algumas formas diferentes de se versionar a API para o cliente. Nenhuma é totalmente certa, nem errada e a especificação do REST não define este item. O que existe são tendências maiores ou menores de adoção de alguns padrões por conta dos prós e contras que cada um oferece.

Do ponto de vista do cliente, também é interessante tomar alguns cuidados: para minimizar os impactos de qualquer alteração, mesmo aquelas que não quebram o contrato, os clientes destas APIs devem evitar consumir mais campos do que o necessário ao chamar uma API. Eles devem usar apenas aqueles campos que o cliente realmente precisa e deve ignorar os que não precisa. Este conceito é abordado em [Tolerant Reader (Martin Fowler)](https://martinfowler.com/bliki/TolerantReader.html).


#### Versionamento pelo host

Na estrutura de host da URL, coloca-se a versão como parte dele. Por exemplo:
https://api-v2.empresaexemplo.com/clientes

Observe o "-v2", que específica que esta é a versão 2 da API.

O ponto negativo desta abordagem é que do ponto de vista de deploy, é mais difícil criar um novo nome no host do que outras abordagens em que se cria pastas (recursos), por exemplo.

#### Versionamento pela query string

Define-se a versão a versão via  **query string**, por exemplo:

https://api.empresaexemplo.com/clientes?version=2.0

Observe o version=2.0, que especifica que essa é a versão 2.0 dessa API.

O ponto negativo dessa abordagem é que prejudica a definição de contrato para outras versões e prejudica a legibilidade da URL em cenários de muitos parâmetros. Além de facilitar com que o cliente esqueça de definir a versão ao chamar a API e talvez tome erro por estar chamando a versão default.

#### Versionamento pelo Content-Type (com o header Accept)

Define-se um tipo customizado de conteúdo com a versão dele. Por exemplo:

GET https://api.empresaexemplo.com/clientes

Accept: application/vnd.clientes.v2+json

O ponto positivo é que a URL fica mais clean, no entanto não é dev-friendly, pois a requisição tem que ser feita com muito mais cuidado, dada a passagens de mais parâmetros.

#### Versionamento por header customizado

Define-se um header customizado para requisitar a versão da API. Por exemplo:

GET https://api.empresaexemplo.com/clientes

api-version: 2

O ponto positivo é que a URL fica mais clena, no entanto não é dev-friendly, pois a requisição tem que ser feita com muito mais cuidado, dada a passagens de mais parâmetros.

#### Versionamento pelo Path (Resources)

Define-se a versão no Path como se fosse um recurso, por exemplo:

https://api.empresaexemplo.com/v1/clientes

Esta é a minha forma preferencial de versionar pois, fazendo parte da URL você força sempre o cliente a informá-la; a versão está sempre visível; em processo de deploy, é muito mais simples criar uma pasta do que definir um novo host; mantém um visual clean na URL e facilita na definição de contratos para versões novas.

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Segurança

TO DO: Aguarde! Este capítulo será escrito em breve. :-)

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Performance, Cache e compressão

TO DO: Aguarde! Este capítulo será escrito em breve. :-)

<sub>ir para: [índice](#conte%C3%BAdo)</sub>

## Palavras finais

Quase tudo em REST abre discussões sobre estar mais aderente ou menos a um modelo que é "vago" em vários pontos. Por um lado dá liberdade para evolução de acordo com as necessidades do mercado, por outro, quando você olha o StackOverflow e concorda com as n respostas diferentes para a mesma pergunta, você conclui que o importante é adotar um padrão que faça sentido para sua empresa. Certo ou errado, seguir um padrão é muito melhor do que não seguir nenhum.

A busca deve ser sempre em buscar o RESTful, no entanto, no meio do caminho vai ser normal entregar algumas APIs "quase REST" que adotam a maioria dos padrões mas abandona alguns em prol de timing de projeto, economia de recursos ou satisfação do cliente.

Alguns destes padrões que abandonamos podem não fazer falta e não fará o desenvolvedor passar vergonha ao dizer que fez uma API REST, mas não implementou o HTTP Status Code A ou B; ou talvez não implementou HATEOAS em todas as APIs; ou não implementou paginação em alguma delas. No entanto, alguns anti-patterns são praticamente inadimissíveis, como usar somente o verbo POST para resolver todos os problemas; ignorar os comportamentos de idempotência e seguro; limitar a resposta a 200 para Ok e 500 para todo o resto sem nenhum detalhe adicional ... enfim, o recado é que busquem sempre entregar o melhor contrato possível.

E com a prática, percebe-se que tudo isso aqui é muito simples, pois é técnico, é mais "preto no branco". O real desafio estará em modelar o negócio, entender como ele é e representá-lo bem através de URLs e Atributos. Mas isto é assunto pra um outro guia que será entregue em breve.
