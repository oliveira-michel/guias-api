# [WIP] Insights sobre versionamento de APIs

Neste artigo, faço uma pesquisa sobre como algumas das principais APIs do mercado tratam o assunto versionamento. Para conhecer alguns conceitos básicos sobre versionamento, recomendo primeiro a leitura do capítulo sobre versionamento no [Guia de Design REST](https://github.com/oliveira-michel/guias-api/blob/master/design-rest-api/guia.md#versionamento).

No quadro abaixo, listo algumas APIs independente do segmento, idade, qualidade da modelagem ou aderência ao REST. Observe como cada uma das empresas optou por versionar suas APIs.

| API | Onde | Como
| --- | --- | ---
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

### Minha preferência: versão MAJOR na URL




### Conclusão

Talvez com um número bem maior de APIs outros insights poderiam ter aparecido, no entanto, não me surpreendeu que após o levantamento, não apareceu nada de muito novo, senão o que já é recorrente em guias de boas práticas.
