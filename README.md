# QA Twilio Errors

Dicionário de erros e códigos da Twilio para apoio em testes de integração e análise de falhas.

---

## 1. O que é a Twilio?

A Twilio oferece APIs e serviços para comunicação, como envio de mensagens, chamadas e outros recursos de comunicação.

Durante a integração com uma aplicação, uma requisição pode ser processada com sucesso ou retornar um erro.

Como QA, é importante saber interpretar esse retorno para diferenciar:

* erro de dados;
* erro de configuração;
* erro de autenticação;
* recurso inexistente;
* limite de requisições;
* indisponibilidade do serviço;
* falha no processamento da aplicação.

---

## 2. Como um erro da Twilio aparece?

Quando uma requisição não é processada corretamente, a resposta pode apresentar informações como:

```json
{
  "code": 21211,
  "message": "The 'To' number is not a valid phone number.",
  "more_info": "https://www.twilio.com/docs/errors/21211",
  "status": 400
}
```

Os principais campos são:

| Campo       | O que representa                             |
| ----------- | -------------------------------------------- |
| `status`    | Código HTTP da resposta                      |
| `code`      | Código específico do erro Twilio             |
| `message`   | Descrição do problema                        |
| `more_info` | Link com informações adicionais sobre o erro |

A Twilio documenta esses campos como parte das respostas de exceção da API.

---

# 3. Código HTTP x Código Twilio

Essas duas informações não são a mesma coisa.

### Código HTTP

Indica o resultado geral da requisição.

Exemplos:

```text
200 → sucesso
201 → recurso criado
400 → requisição inválida
401 → não autorizado
404 → recurso não encontrado
429 → limite de requisições atingido
500 → erro interno
503 → serviço temporariamente indisponível
```

A documentação da Twilio descreve esses códigos e seus significados.

### Código Twilio

É um código específico que ajuda a identificar a causa do problema.

Exemplo:

```text
HTTP: 400
Código Twilio: 21211
```

Nesse caso, o `400` informa que a requisição é inválida, enquanto o `21211` ajuda a identificar especificamente o problema.

A Twilio mantém um dicionário oficial com os códigos de erro e suas descrições.

---

# 4. Como investigar um erro?

Quando encontrar um erro, não olhe somente para o código.

Siga esta sequência:

```text
1. Código HTTP
       ↓
2. Código Twilio
       ↓
3. Mensagem
       ↓
4. Dados enviados
       ↓
5. Configuração
       ↓
6. Logs
       ↓
7. Comportamento esperado
       ↓
8. Resultado do teste
```

Pergunte:

* O Request estava correto?
* Os dados enviados eram válidos?
* A autenticação estava correta?
* O recurso existia?
* A configuração estava correta?
* O erro era esperado para aquele cenário?
* A aplicação tratou o erro corretamente?

---

# 5. Onde encontrar os erros?

Os erros podem ser investigados em diferentes pontos.

### Resposta da API

A própria resposta pode apresentar:

* código HTTP;
* código Twilio;
* mensagem;
* link para documentação.

### Twilio Console

O Console possui ferramentas para investigação dos eventos.

### Twilio Debugger

O Debugger permite consultar eventos de informação, aviso e erro, além de visualizar detalhes da ocorrência.

É possível pesquisar, por exemplo, pelo código do erro ou pelo SID da operação.

A Twilio também orienta utilizar os registros para analisar requisições de API, webhooks e outros problemas operacionais.

---

# 6. Exemplo prático de QA

Imagine que uma aplicação tenta enviar uma mensagem para um número inválido.

A API retorna:

```json
{
  "code": 21211,
  "message": "The 'To' number is not a valid phone number.",
  "status": 400
}
```

### O que o QA deve verificar?

```text
1. O número enviado era realmente inválido?
2. A aplicação enviou a requisição corretamente?
3. A Twilio retornou o erro esperado?
4. O sistema identificou o código 21211?
5. O erro foi registrado?
6. A aplicação apresentou uma mensagem adequada?
7. O usuário recebeu uma orientação compreensível?
8. O sistema evitou uma tentativa indevida de reenvio?
```

### Conclusão

Nesse cenário, o `400` não significa automaticamente que existe um defeito na Twilio ou na aplicação.

É necessário verificar se o comportamento corresponde ao cenário executado.

---

# 7. Quando um erro pode ser considerado defeito?

Um erro retornado pela Twilio **não significa automaticamente que o sistema está com defeito**.

O QA deve comparar:

```text
Comportamento esperado
        ×
Comportamento obtido
```

### Exemplo

Se o requisito determina:

> Ao informar um número inválido, o sistema deve informar que o número não pode ser utilizado.

E a Twilio retorna um erro de número inválido, mas a aplicação:

```text
❌ apresenta tela quebrada
❌ não trata o erro
❌ exibe mensagem técnica para o usuário
❌ registra informação incorreta
```

Nesse caso, pode existir um problema no tratamento realizado pela aplicação.

---

# 8. Cenários de teste

## Cenário 1 — Dados válidos

**Dado:** número válido.

**Quando:** a aplicação envia a mensagem.

**Então:** a requisição deve ser processada conforme o comportamento esperado.

Validar:

* Request;
* Response;
* código HTTP;
* status da mensagem;
* tratamento da aplicação.

---

## Cenário 2 — Número inválido

**Dado:** número inválido.

**Quando:** a aplicação tenta realizar o envio.

**Então:** o erro deve ser tratado corretamente.

Validar:

* código HTTP;
* código Twilio;
* mensagem;
* comportamento da aplicação;
* mensagem apresentada ao usuário.

---

## Cenário 3 — Credenciais inválidas

**Dado:** credenciais incorretas.

**Quando:** a aplicação realiza a requisição.

**Então:** a API deve rejeitar a autenticação.

Validar:

* código HTTP;
* mensagem;
* tratamento do erro;
* segurança das informações apresentadas.

---

## Cenário 4 — Recurso inexistente

**Dado:** identificador inexistente.

**Quando:** a aplicação consulta o recurso.

**Então:** a API deve retornar o comportamento esperado para recurso não encontrado.

A Twilio documenta `404 Not Found` quando o recurso solicitado não é encontrado.

---

## Cenário 5 — Limite de requisições

**Dado:** volume de requisições acima do permitido.

**Quando:** a aplicação ultrapassa o limite.

**Então:** deve tratar adequadamente a resposta.

A Twilio documenta `429 Too Many Requests` para situações em que o limite de concorrência da API foi atingido.

---

# 9. Checklist de investigação

Utilize este checklist durante a análise:

```text
[ ] Identifiquei o endpoint
[ ] Identifiquei o método HTTP
[ ] Validei o Request
[ ] Validei os parâmetros
[ ] Validei a autenticação
[ ] Identifiquei o código HTTP
[ ] Identifiquei o código Twilio
[ ] Analisei a mensagem
[ ] Consultei a documentação do erro
[ ] Verifiquei os logs
[ ] Analisei o comportamento da aplicação
[ ] Comparei resultado esperado x obtido
[ ] Registrei evidências
[ ] Classifiquei o resultado do teste
```

---

# 10. Como usar este repositório?

Este projeto possui duas planilhas:

### Dicionário de erros

`dicionario_erros_twilio.ods`

### Versão Excel

`dicionario_erros_twilio_excel.xlsx`

Utilize o material para pesquisar códigos e entender rapidamente o significado de um erro durante uma análise.

### Fluxo recomendado

```text
Encontrou um erro
       ↓
Identifique o código HTTP
       ↓
Identifique o código Twilio
       ↓
Pesquise no dicionário
       ↓
Consulte a documentação oficial
       ↓
Analise Request e Response
       ↓
Verifique logs
       ↓
Compare esperado x obtido
       ↓
Registre o resultado
```

---

# 11. Exercício prático

Escolha um código de erro disponível na planilha.

Depois responda:

```text
1. Qual é o código HTTP?
2. Qual é o código Twilio?
3. O que significa?
4. Qual pode ser a causa?
5. O que deveria ser validado pelo QA?
6. Qual comportamento seria esperado?
7. O erro representa necessariamente um defeito?
8. Quais evidências seriam necessárias para abrir um bug?
```

Esse exercício pode ser repetido para diferentes códigos.

---

# 12. O que este projeto demonstra?

Este repositório demonstra conhecimentos relacionados a:

* Testes de API;
* Integrações;
* HTTP;
* Análise de erros;
* Investigação de falhas;
* Validação de Request e Response;
* Testes positivos e negativos;
* Tratamento de erros;
* Documentação técnica;
* Atuação de QA em integrações externas.

---

# 13. Referências

* [Twilio — API Responses](https://www.twilio.com/docs/usage/twilios-response)
* [Twilio — Error and Warning Dictionary](https://www.twilio.com/docs/api/errors)
* [Twilio — Debugger](https://www.twilio.com/docs/usage/troubleshooting/debugging-your-application)
* [Twilio — Troubleshooting](https://www.twilio.com/docs/usage/troubleshooting)

---

## Autora

**Erica de Souza**

QA Analyst com foco em qualidade de software, testes de API, automação e investigação de integrações.

Este projeto foi desenvolvido como material de estudo e apoio à atuação em QA.
