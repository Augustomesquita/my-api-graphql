# API GraphQL com SpringBoot + SpringData + Flyway (PostgreSQL 10.3)
Aplica��o que cria uma API GraphQL juntamente com SpringBoot + SpringData + Flyway + PostgreSQL.
O conceito e dicas sobre API GraphQL pode ser encontrado na p�gina oficial: https://graphql.org/learn/

## A��es necess�rias para utiliza��o
Para rodar a aplica��o da maneira como ela j� est� configurada, basta criar um banco de dados com o nome 'mygraphql_db' em seu PostgreSQL na porta padr�o 5432. O usu�rio e senha do banco devem ser 'postgres'.

Caso queira realizar modifica��es quanto ao:
 - Tipo de banco utilizado;
 - Porta;
 - Nome do banco de dados;
 - Usu�rio e senha.

Basta modificar o arquivo de propriedades da aplica��o, localizado em '/resource/application.properties'.

# Testando a API Graph
Ao executar a aplica��o, dois endpoints ficar�o automaticamente dispon�veis. S�o eles o "/graphql" e o "/graphiql". 

/graphql: Endpoint respons�vel por realizar as a��es, ou seja, em testes fora do "/graphiql" (que ser� explicado logo abaixo) ou em alguma aplica��o que esteja consumindo esta API, este endpoint que dever� ser usado para realizar as requisi��es.

/graphiql: Nos disponibilizar� a interface gr�fica GraphiQL, onde poderemos ter acesso a documenta��o de nossa API e tamb�m realizar Queries e Mutations de forma mais elegante, sem precisarmos recorrer a ferramentas mais verbosas para testar este tipo de API, como o Postman. Por baixo dos panos as Queries e Mutations realizadas dentro desta ferramenta, ser�o encaminhadas para o endpoint "/graphql".

## Testes atrav�s do GraphiQL (interface gr�fica do GraphQL)
Exemplo de realiza��o de uma Query (exemplo funcionando, baseado nos dados desta aplica��o) que retorna todos os usu�rios do banco, trazendo o nome/idade de cada um, juntamente do nome do filme favorito dos mesmos:
```js
{
  getAllUser { 
    name 
    movie {
      name
    }
  }
}
```

Exemplo de realiza��o de uma Mutation (exemplo funcionando, baseado nos dados desta aplica��o) que cria um filme (com os dados do nome e diretor) no banco e retorna como resposta o objeto filme criado com os campos 'id, name e director' preenchidos:
```js
mutation {
  saveMovie(name: "Filme Teste 6", director: "Diretor Teste 6") {
    id
    name
    director
  }
}
```

## Testes atrav�s do Postman
Embora o Postman n�o seja a ferramenta ideal para testes de API GraphQL, muita gente o usa por causa das API Rest. Por este motivo, deixarei aqui a sintaxe para realizar testes usando-o. Mas recomendo fortemente, caso voc� queira usar uma ferramente a parte para testes, a utiliza��o do Insomnia (https://insomnia.rest/), pois ele j� possui uma forma espec�fica para trabalhar com API GraphQL (https://support.insomnia.rest/article/61-graphql).

Exemplo de realiza��o de uma Query no Postman (exemplo funcionando, baseado nos dados desta aplica��o):
![](screenshots/query_postman.png)


Exemplo de realiza��o de uma Mutation no Postman (exemplo funcionando, baseado nos dados desta aplica��o):
![](screenshots/mutation_postman.png)


# Carregando dados do banco de maneira eficiente com GraphQL e Spring Data
Caso esteja usando spring data, igual a este projeto, � importante que as rela��es entre as entidades sejam, na maior parte das vezes, configuradas no modo "pregui�oso" para que voc� usuflua corretamente dos benef�cios do GraphQL. Assim, caso o consumidor de sua API GraphQL informe que precisa apenas dos dados de uma determinada entidade, seu backend n�o precisa realizar a��es extras desnecess�rias no banco para carregador dados de objetos que s�o atributos da entidade que era realmente desejada.
No exemplo deste projeto, a classe "Movie" � um atributo da classe "User". Sendo assim, caso o consumidor da mesma queira apenas o nome de um determinado "User", o backend n�o ir� realizar um JOIN de "Movie" no momento de buscar as informa��es de "User".
Exemplo de configura��o da entidade usando Spring Data (exemplo funcionando, baseado nos dados desta aplica��o):

```java
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "movie_id", nullable = false, updatable = false)
    private Movie movie;
```
Isso faz com que o "User" instancie um objeto "Movie" e preencha apenas o atributo de "ID" do filme, pois o "ID" j� est� presente na tabela de "User" no momento da busca.
Mas e quando o consumidor quiser dados de "Movie", como o nome do filme ou o diretor? Nesse momento as classes Resolvers s�o chamadas, para "resolver" as lacunas dos objetos, que em tese (caso voc� tenha configurado o carregamento "pregui�oso corretamente", existir�o.