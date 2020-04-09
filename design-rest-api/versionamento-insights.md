# Insights sobre versionamento de APIs

Neste artigo, faço uma pesquisa sobre como algumas das principais APIs do mercado tratam o assunto versionamento. Para conhecer alguns conceitos básicos sobre versionamento, recomendo primeiro a leitura do capítulo sobre versionamento no [Guia de Design REST](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/guia.md#versionamento).

No quadro abaixo, listo algumas APIs independente do segmento, idade, qualidade da modelagem ou aderência ao REST. Observe como cada uma das empresas optou por versionar suas APIs.

| API | Onde | Como
| :-- | :-- | :--
| [Direções do Google Maps](developers.google.com/maps/documentation/directions)<br>maps.googleapis.com/maps/api/directions/json | Não versiona | 
| [Maps em Javascript](developers.google.com/maps/documentation/javascript)<br>maps.googleapis.com/maps/api/js **?v=3.39** | query strings | v=weekly, v=quarterly, v=n.nn (ex: v=3.39)
| [Busca de Tweets no Twitter](developer.twitter.com/en/docs/tweets/search/api-reference/get-search-tweets)<br>api.twitter.com/**1.1**/search/tweets.json | path parameter | 1.1
| [Facebook](developers.facebook.com/docs/marketing-api/versions)<br>graph.facebook.com/**v2.2**/me/adaccounts | path parameter | 2.2
| [Twillio](www.twilio.com/docs/usage/api/address)<br>api.twilio.com/**2010-04-01**/Accounts/123/Addresses.json | path parameter | 2010-04-01
| [Last FM](www.last.fm/api/)<br>ws.audioscrobbler.com/**2.0**/?method=artist.getinfo&artist=Cher<br>&api_key=YOUR_API_KEY&format=json | path parameter | 2.0
| [Azure SQL Databases](docs.microsoft.com/en-us/rest/api/sql/databases/get)<br> management.azure.com/subscriptions/{subscriptionId}<br>/resourceGroups/{resourceGroupName}<br>/providers/Microsoft.Sql/servers/{serverName}<br>/databases/{databaseName}**?api-version=2017-10-01-preview** | query parameter | ?api-version=2017-10-01-preview
| [Four Square](developer.foursquare.com/docs/places-api/versioning)<br>api.foursquare.com/**v2**/venues/search?client_id=CLIENT_ID<br>&client_secret=CLIENT_SECRET&ll=40.7,-74<br>&query=sushi&**v=20180323** | path parameter, query parameter | /v2 e ?v=20180323
| [Linkedin](www.linkedin.com/developers)<br>api.linkedin.com/**v2**/me | path parameter | v2
| [PayPal](developer.paypal.com/docs/api/invoicing/v2/)<br>api.sandbox.paypal.com/**v2**/invoicing/invoices | query parameter | v2
| [Indeed](opensource.indeedeng.io/api-documentation)<br>api.indeed.com/ads/apisearch?publisher=1234&q=java+developer **&v=2** | query parameter | v=2
| [Banco do Brasil](developers.bb.com.br/docs)<br>/consultas-financeiras/**v2**/statements/checking-account | path parameter | v2
| [BBVA](www.bbvaapimarket.com/documentation/bbva/customers)<br>apis.bbva.com/customers/**v1**/me/globalposition-basic | path parameter | v1
| [Vimeo](developer.vimeo.com/api/common-formats)<br>api.vimeo.com/videos/174560759/comments<br>Accept: application/vnd.vimeo.user+json;version=3.0 | header | Accept: application/vnd.vimeo.user+json;version=3.0
| [Google Cloud Datastore](cloud.google.com/datastore/docs/reference/admin/rest)<br>datastore.googleapis.com/**v1**/projects/{projectId}<br>datastore.googleapis.com/**v1beta1**/projects/{projectId} | path parameter | v1

Com base no quadro acima, podemos observar alguns pontos:

#### **Preferência por path parameter e query parameter**

Apesar de ter buscado por APIs diversas, nenhuma delas optou pelo versionamento via [host](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/guia.md#versionamento-pelo-host) ou [header](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/guia.md#versionamento-por-header-customizado). O Vimeo foi o único a adotar a abordagem com [content-type](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/guia.md#versionamento-pelo-content-type-com-o-header-accept). A preferência geral foi por utilizar path parameter (na estrutura da URL) ou passar como query parameter.

#### **Foram utilizadas Major somente, Major.Minor e datas**

Em ordem crescente. as empresas preferiram optar por versionar considerando apenas o Major, o Major.Minor e uma minoria com data.

#### **Adotam uma única forma de versionar**

A maioria optou por um único ponto para versionar a API (path parameter ou query). Apenas a API do Four Square que optou por colocar a Major na URL e opcionalmente, uma versão específica via query string.

#### **Algumas endereçaram versões preview, beta e builds recorrentes**

Apesar de menos frequente, algumas strings de versionamento endereçam builds recorrentes, como *weekly*, *quarterly* como na Google Maps em Javascript; *v1beta1* na Google Datastore e *2017-10-01-preview* na Azure Sql Databases.

### E o Oscar vai para "versão MAJOR na URL"

Quando falamos [versionamento semântico](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/guia.md#versionamento), alterações em Minor e Path não devem fazer com que quem já consome a versão tenha qualquer problema por quebra de contrato, ou seja, nenhum campo não obrigatório passa a ser obrigatório e nenhum campo é removido. Neste tipo de alteração, são apenas adicionados novos campos ou feitas alterações em metadados. Assim, pequenas evoluções podem ser feitas na API, notificadas aos desenvolvedores, no entanto, sem demandar que atualizações sejam obrigatoriamente feitas nos seus clientes.

Entendo que clientes que venham a "quebrar" por conta da adição de novos campos, provavelmente fizeram o consumo da API de forma atípica, pra não falar errada, talvez, processando manualmente o payload e buscando os campos por posição, por exemplo.  Martin Fawler em [TolerantReader](https://martinfowler.com/bliki/TolerantReader.html) reforça conceitos do comportamento que é esperado de um consumidor de serviços.

Outro ponto a ser considerado é que, por mais que não se quebre diretamente o contrato, quem produz a API pode optar por incrementar uma MAJOR em casos raros, mas possíveis, como aumento considerável do tamanho ao payload, aumento considerável do tempo de processamento, alteração nas regras de quantidade de registros retornados etc.


#### Open Banking: E quando o modelo é definido por uma entidade externa?

##### - Abordagem 1: versão MAJOR apenas e flexibilidade timing da migração.

No cenário de open banking, por exemplo, e quem que o contrato é definido por uma entidade externa e os clientes e servidores devem respeitá-lo, também seria possível manter o versionamento apenas com a MAJOR na URL.

Imagine o cenário de um contrato como o de baixo:
| Data | Versão | Campos 
| --- | --- | --- 
| 01/2020 | 1.0 | "nome", "telefone" 
| 06/2020 | 1.1 | "nome", "telefone", "endereco" 
| 11/2020 | 1.2 | "nome", "telefone", "endereco", "email"

Agora, segue um exercício de cliente e servidor implementando em "tempos" diferentes e sem que um impacte no outro.

| Data | URL | Versão cliente | Versão servidor | Resultado
| :-- | :-- | :-- | :-- | :--
| 01/2020 | /cadastros/v1/clientes | 1.0<br>Lê os campos<br>"nome" e "telefone" | 1.0<br>fornece os campos<br>"nome" e "telefone" | 100% ok
| 07/2020 | /cadastros/v1/clientes | 1.1<br>Lê os campos<br>"nome", "telefone"<br>e implementa leitura opcional do campo "endereco" | 1.0<br>fornece os campos<br>"nome" e "telefone" | Cliente não recebe ainda o "endereço", mas sua implementação está preparada para tal.
| 09/2020 | /cadastros/v1/clientes | 1.1<br>Lê os campos<br>"nome", "telefone"<br>e implementa leitura opcional do campo "endereco" | 1.1<br>fornece os campos<br>"nome", "telefone" e "endereco" | Cliente passa a receber o "endereço" no momento que o servidor passou a enviar o dado.
| 12/2020 | /cadastros/v1/clientes | 1.1<br>Lê os campos<br>"nome", "telefone"<br>e implementa leitura opcional do campo "endereco" | 1.2<br>fornece os campos<br>"nome", "telefone", "endereco" e "email" | Servidor fornece todos os campos da nova versão, mas cliente ignora existência do campo "email".

A vantagem desta abordagem é que permite que tanto cliente, quanto servidor possam implementar as versões definidas pela entidade externa, cada um no seu tempo, independentemente se a outra parte da comunicação já o fez.

O que garantirá que as peças "se encaixem" será  o **contrato bem especificado** com **tamanho** máximo, mínimo e **tipo de dado**, **expressão regular** e exemplos, de forma que a implementação em ambos os lados não tragam elementos **surpresa** quando estiver em produção.

O que garantirá a aderência aos contratos são as ferramentas já fornecidas pelas **disciplinas de testes** e **qualidade de software** já conhecidas pelo mercado.

##### - Abordagem 2: versão MAJOR.MINOR e inflexibilidade no timing da migração.

Opcionalmente, se partirmos do princípio de que as práticas acima não serão seguidas, podemos utilizar a versão específica **MAJOR.MINOR** na URL, isso reduz o risco de alguma quebra do cliente, pois o servidor passa a ofertar sempre 'n' versões do software, no entanto, o cliente fica dependente do servidor implementar primeiro a versão para depois ele poder consumir.

As URLs, neste caso ficariam assim:

- /cadastros/v1.0/clientes
- /cadastros/v1.1/clientes
- /cadastros/v.2.1/clientes

##### - Abordagem 3: versão MAJOR ou MAJOR.MINOR e flexibilidade no timing da migração.

Neste modelo, podemos ter 3 URLs:

- /cadastros/v1/clientes
- /cadastros/v1.0/clientes
- /cadastros/v1.1/clientes
- /cadastros/v1.2/clientes

Neste modelo ofereceríamos flexibilidade aos clientes, dado que ele escolhe se prefere trabalhar com a abordagem 1 ou a 2.

Quando o cliente chamar a URL v1, ele sempre trará a **implementação mais recente existente da versão 1**. Logo, o comportamento seria o mesmo da abordagem 1, no entanto, além da v1, o servidor disponibilizaria as v1.0, v1.1 e v1.2 para aqueles clientes que prefiram a segurança de ter um *contrato congelado no tempo*, mas com o trade-off de esperar o servidor implementar a versão.


### Conclusão

Sobre Open Banking, penso no cliente que é um APP e precisará acessar ao mesmo tempo 'n' implementações de bancos diferentes. Acredito que ele prefira a flexibilidade de implementar uma única vez a capacidade de ler a nova versão e colher os benefícios de novos atributos o quanto antes, independente de os seus vários fornecedores de informação já as estarem disponibilizando. Por isso, gosto da abordagem 3, que me parece atender a qualquer um dos cenários.

Sobre os modelos de versionamento, de forma geral, talvez com um número bem maior de APIs, outros insights poderiam ter aparecido, no entanto, não me surpreendeu que após este pequeno o levantamento, não tenha aparecido nada de muito novo, senão o que já é recorrente em guias de boas práticas. Ou seja, já existe uma convergência no mercado para alguns modelos principais.
