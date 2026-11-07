# Análise de Log e Relatório de Bug (Bug Report)

## Cenário Testado
Tentativa de realizar o aluguel de um veículo com um ID que não existe no banco de dados (ou foi deletado).

## Evidência (Log do Servidor)
Ao executar a requisição POST no Postman, o servidor Spring Boot retornou o seguinte log de erro:
> `java.lang.RuntimeException: Veículo não encontrado`
> `at com.projeto_pida.locadora.services.LocacaoService.salvar(LocacaoService.java:63)`

## Análise do QA
Analisando o *Stack Trace* (log de erro), identifiquei que:
1. O sistema tentou buscar o veículo no banco de dados via Hibernate (select).
2. O veículo não foi retornado.
3. O método `salvar` na linha 63 da classe `LocacaoService` disparou uma exceção (erro) do tipo `RuntimeException`.

## Relatório de Bug (Bug Report)

**Título:** Erro 500 (Internal Server Error) ao tentar alugar veículo inexistente.

**Severidade:** Alta (Impede a funcionalidade principal).
**Prioridade:** Média (Só ocorre se o ID for inválido).
**Ambiente:** Localhost, Spring Boot (Java), Postman.

**Passos para reproduzir:**
1. Acessar a rota de criação de locação (POST /locacoes).
2. Enviar um JSON com o `veiculoId` = 9999 (ID inexistente).
3. Observar a resposta do servidor.

**Resultado Esperado:**
O sistema deveria retornar um erro amigável (ex: HTTP 404 Not Found) com a mensagem "Veículo não encontrado. Por favor, selecione um veículo válido." (Tratamento de exceção com `@ExceptionHandler`).

**Resultado Obtido:**
O servidor retorna um erro HTTP 500 e "quebra" a aplicação exibindo o Stack Trace inteiro no console.

**Sugestão para o Desenvolvedor:**
Criar um controle de exceção (Exception Handler) para a classe `RuntimeException` no `LocacaoService`, retornando um status HTTP 404 ou 400 e um JSON limpo para o front-end, evitando vazar o log do servidor para o usuário final.
