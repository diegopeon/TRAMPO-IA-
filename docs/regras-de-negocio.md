# Regras de negócio

Cada regra tem um código para ser citada no código e nos testes. As verificações técnicas correspondentes estão em [validacoes.md](validacoes.md).

## Acesso

| ID | Regra | Motivo |
|---|---|---|
| RN01 | O uso é gratuito e não exige cadastro. | O público não pode pagar, e uma conta criaria dado para guardar. |
| RN02 | A pessoa só chega ao formulário depois de aceitar o aviso de privacidade. | Ela precisa saber que o texto passa por uma empresa de IA. |
| RN03 | Quem tem menos de 18 anos é orientado, no aviso, a usar a ferramenta com um responsável ou na oficina. | Art. 14 da LGPD: o tratamento de dados de adolescentes deve atender ao melhor interesse deles. |
| RN04 | A idade é opcional. Quando informada, precisa ser 14 ou mais. | A Constituição só admite trabalho a partir dos 14 anos, como aprendiz (art. 7º, XXXIII). |

## Integridade do currículo

O projeto define a IA como tradutora fiel do relato, sem criar diplomas ou experiências. O filtro que aplica estas regras é código comum, sem IA, e roda no servidor.

| ID | Regra | Motivo |
|---|---|---|
| RN05 | Nenhum curso, diploma, certificado, idioma, ferramenta, empresa, cargo, data ou número pode aparecer se não estiver no relato. | Currículo com informação falsa prejudica quem o entrega. |
| RN06 | Todo item de experiência, competência ou curso cita o trecho do relato de onde saiu (campo `origem`). Item sem trecho válido é descartado. | Dá para conferir em código, sem depender do modelo. |
| RN07 | Número ou termo de credencial que não esteja na entrada derruba o item inteiro. Em `objetivo` e `resumo`, o campo volta vazio. | São as invenções mais comuns e as mais fáceis de detectar. |
| RN08 | A seção de formação é montada com os dados do formulário, sem passar pelo modelo. | É o lugar onde um diploma inventado faria mais estrago. |
| RN09 | Trabalho informal é descrito pela atividade, sem cargo formal nem vínculo que a pessoa não teve. | "Ajudo no bar do meu tio" pode virar "Atendimento em bar familiar", nunca "Gerente de bar". |
| RN10 | Nada é exportado sem passar pela tela de revisão. | A pessoa é quem sabe o que fez. |
| RN11 | Se depois do filtro não sobrar experiência nem competência, o sistema tenta mais uma vez. Na segunda falha, pede mais detalhes. | Relato curto demais não vira currículo. |

## Privacidade

| ID | Regra | Motivo |
|---|---|---|
| RN12 | O TrampoIA não grava relato, currículo nem identificação. Os dados ficam na memória da aba e somem quando ela fecha. | Minimização de dados (LGPD). |
| RN13 | Nome, contato, cidade, idade e escola não são enviados ao modelo. | O texto do currículo não depende deles. |
| RN14 | CPF, telefone e e-mail digitados no relato são trocados por `[removido]` antes da chamada. | Em campo livre as pessoas escrevem de tudo. |
| RN15 | O currículo não menciona religião, cor ou raça, saúde, orientação sexual nem posição política. Atividade em igreja, terreiro, partido ou sindicato aparece como "organização comunitária", e a pessoa pode mudar isso na revisão. | São dados sensíveis pela LGPD e podem gerar discriminação na seleção. |
| RN16 | Os logs registram data, rota, status, duração, tokens e quantidade de itens descartados. Texto digitado nunca entra em log. | Log é persistência. |
| RN17 | O aviso informa que o relato é processado pela Anthropic e apagado por ela em até 30 dias. | É o prazo publicado pela Anthropic para a API. Conteúdo marcado como violação da política de uso pode ficar até 2 anos. |

Texto do aviso:

```text
Para montar seu currículo, o que você escrever aqui é enviado para uma
inteligência artificial da empresa Anthropic. Seu nome e seu contato não vão
para a IA.

O TrampoIA não guarda nada: fechou a página, apagou. A Anthropic apaga em até
30 dias (ou guarda por mais tempo se o texto violar as regras de uso dela).

Não escreva CPF, RG, endereço ou informações de saúde.

Tem menos de 18 anos? Mostre esta página para seu responsável ou use na oficina.

[ ] Li e quero continuar
```

## Conteúdo do currículo

| ID | Regra | Motivo |
|---|---|---|
| RN18 | Seções, nesta ordem: identificação, objetivo, resumo, formação, experiências, competências, cursos e oficinas. Seção vazia não aparece. | Ordem padrão, que o ATS reconhece. |
| RN19 | O texto usa linguagem de RH, sem gíria e sem primeira pessoa. | Padrão esperado pelos recrutadores. |
| RN20 | Termos comuns da área pretendida só entram quando descrevem o que foi relatado. | O ATS busca palavras-chave, e a palavra-chave precisa ser verdadeira. |
| RN21 | O currículo não tem foto, tabela nem colunas. | O ATS lê mal esses elementos, e foto abre espaço para discriminação. |

## Entrevista

| ID | Regra | Motivo |
|---|---|---|
| RN22 | São de 3 a 5 perguntas comuns em entrevistas para a vaga escolhida. | Quantidade definida no projeto. |
| RN23 | Cada pergunta vem com 2 a 4 passos em português simples, e ao menos um passo aproveita algo que a pessoa contou. | A orientação precisa partir da realidade dela. |
| RN24 | As perguntas não passam pelo filtro de origem e não entram no PDF do currículo. | Uma pergunta pode citar Excel sem que a pessoa saiba Excel. |

## Limites de uso

| ID | Regra | Motivo |
|---|---|---|
| RN25 | Até 30 gerações a cada 10 minutos por IP. | Uma sala no mesmo Wi-Fi sai com um IP só, e um limite menor travaria a oficina. |
| RN26 | Até 300 gerações por dia no total, ajustável em `LIMITE_DIARIO`. | A chave da API é paga e a página é aberta. |
| RN27 | Cada clique em "Gerar" dispara no máximo 2 chamadas ao modelo. | Tentativa sem limite vira gasto sem limite. |
| RN28 | A conta da Anthropic tem limite de gasto configurado no Console. | Os contadores acima ficam em memória e zeram se o servidor reiniciar. |
