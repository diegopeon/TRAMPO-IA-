# Validações

Toda regra de campo roda duas vezes: no navegador, para o erro aparecer na hora, e no servidor, porque o navegador pode ser adulterado. Depois da geração, a resposta do modelo passa por outra rodada de verificações, já que ela também é entrada não confiável.

## Formulário

| ID | Campo | Obrigatório | Regra | Vai ao modelo |
|---|---|---|---|---|
| V01 | `aceite` | Sim | Marcado | Não |
| V02 | `nome` | Sim | 3 a 80 caracteres, ao menos duas palavras; só letras, espaço, apóstrofo e hífen | Não |
| V03 | `telefone` | Telefone ou e-mail | 10 ou 11 dígitos depois de tirar a máscara | Não |
| V04 | `email` | Telefone ou e-mail | Até 120 caracteres, formato `nome@dominio.tld` | Não |
| V05 | `cidade` | Sim, já vem com Salvador/BA | 2 a 60 caracteres | Não |
| V06 | `idade` | Não | Inteiro de 14 a 99 | Não |
| V07 | `nivel` | Sim | Fundamental, médio, técnico ou superior | Sim |
| V08 | `situacao` | Sim | Cursando, concluído ou interrompido | Sim |
| V09 | `escola` | Sim | 2 a 100 caracteres | Não |
| V10 | `previsao` | Não | Ano de conclusão, com 4 dígitos, de 2000 a 2035 | Não |
| V11 | `vaga` | Sim | Jovem Aprendiz, estágio ou primeiro emprego | Sim |
| V12 | `objetivo` | Sim | 10 a 300 caracteres | Sim |
| V13 | `vivencias` | Ver V16 | Até 1.500 caracteres | Sim |
| V14 | `voluntariado` | Ver V16 | Até 1.500 caracteres | Sim |
| V15 | `oficinas` | Ver V16 | Até 1.000 caracteres | Sim |
| V16 | `vivencias` + `voluntariado` + `oficinas` | Sim | Soma de ao menos 40 caracteres, sem contar espaços nas pontas | Sim |

## Servidor, antes de chamar o modelo

| ID | Verificação | Falha |
|---|---|---|
| V17 | Corpo JSON de até 16 KB | E02 |
| V18 | `/api/gerar` repete V01, V07, V08, V11 a V16 com o mesmo esquema Zod | E01 |
| V19 | `/api/gerar` rejeita campo desconhecido. Se `nome` ou `email` chegarem nessa rota, é erro do front e a requisição para aqui | E01 |
| V20 | Limites RN25 e RN26 | E03, E07 |
| V21 | Mascaramento, nesta ordem: CPF `\d{3}\.?\d{3}\.?\d{3}-?\d{2}`, e-mail `[\w.+-]+@[\w-]+\.[\w.-]+`, telefone `\(?\d{2}\)?\s?9?\d{4}[-\s]?\d{4}` | Troca por `[removido]` |
| V22 | Remoção de `<` e `>` do relato, para a pessoa não fechar a tag `<relato>` por acidente ou de propósito | Remove |

## Resposta do modelo

| ID | Verificação | Falha |
|---|---|---|
| V23 | `stop_reason` igual a `end_turn` | `max_tokens` gera nova tentativa; `refusal` gera E05 |
| V24 | JSON de acordo com o esquema | Nova tentativa |
| V25 | `entrevista` com 3 a 5 perguntas e 2 a 4 passos em cada | Abaixo do mínimo, nova tentativa; acima do máximo, corta |
| V26 | `objetivo` até 200 caracteres, `resumo` até 400, cada `texto` até 300 | Campo volta vazio ou item é descartado |
| V27 | No máximo 6 experiências, 10 competências e 6 cursos | Corta o excedente |

O esquema enviado à API não aceita limites de tamanho e de quantidade, por isso V25 a V27 são feitas no servidor.

## Filtro de Integridade

Roda em cada item de `experiencias`, `competencias` e `cursos`. Antes da comparação, entrada e saída passam pela mesma normalização: minúsculas, sem acento, pontuação trocada por espaço e espaços repetidos reduzidos a um. A entrada é o texto exato enviado ao modelo, já mascarado.

| ID | Verificação | Falha |
|---|---|---|
| V28 | `origem` tem ao menos 10 caracteres e aparece na entrada | Descarta o item |
| V29 | Toda sequência de dígitos em `titulo` e `texto` aparece na entrada | Descarta o item |
| V30 | Nenhum termo de credencial em `titulo` e `texto` que não esteja na entrada | Descarta o item |
| V31 | V29 e V30 também valem para `objetivo` e `resumo` | Campo volta vazio |
| V32 | Sobrou ao menos uma experiência ou competência | Nova tentativa; depois, E04 |

Termos de credencial da primeira versão, guardados em `server/filtro.js` e ampliados conforme os testes: certificado, certificação, certificada, diploma, graduação, graduado, bacharel, licenciatura, técnico em, pós-graduação, MBA, fluente, inglês, espanhol, Excel, Word, Pacote Office, CNH, habilitação.

O campo `origem` é retirado antes de a resposta ir para o navegador.

O filtro tem limites conhecidos. Número por extenso ("dois anos") passa sem conferência, e exagero dentro de um trecho real (de "ajudei" para "liderei") não é detectado. Esses casos dependem do prompt e da revisão feita pela própria pessoa.

## PDF

| ID | Verificação | Falha |
|---|---|---|
| V33 | `/api/pdf` valida a identificação (V02 a V06, V09, V10) e os limites de V26 e V27 | E01 |
| V34 | Caracteres fora do conjunto WinAnsi, como emoji, são removidos do texto | Remove |
| V35 | Nome do arquivo no formato `curriculo-primeiro-nome.pdf`, sem acento | Ajusta |

## Tela

| ID | Verificação |
|---|---|
| V36 | O resultado é inserido com `textContent`, nunca com `innerHTML`. Tanto o relato quanto a saída do modelo são texto não confiável |
| V37 | O relato vai ao modelo dentro de `<relato>`, e o prompt manda tratar o conteúdo como dado. O esquema e o filtro limitam o que uma instrução escondida conseguiria produzir |

## Mensagens de erro

| Código | Situação | HTTP | Mensagem na tela |
|---|---|---|---|
| E01 | Campo inválido | 400 | Mensagem do próprio campo, como "Faltou seu nome completo." |
| E02 | Corpo grande demais | 413 | Seu texto passou do limite. Tenta resumir um pouco. |
| E03 | Limite por IP ou limite de taxa da API | 429 | Tem muita gente usando agora. Espera uns minutinhos e tenta de novo. |
| E04 | Relato insuficiente | 422 | Não consegui montar seu currículo com essas informações. Conta um pouco mais do que você já fez: onde e fazendo o quê. |
| E05 | Recusa do modelo | 422 | Não deu para gerar com esse texto. Revisa o que você escreveu e tenta de novo. |
| E06 | Falha da API ou tempo esgotado | 502 ou 504 | A conexão falhou. O que você digitou continua aqui. Tenta de novo. |
| E07 | Limite diário atingido | 503 | O TrampoIA chegou no limite de hoje. Volta amanhã. |
