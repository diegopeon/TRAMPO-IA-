# Diagramas

Escritos em Mermaid, que o GitHub desenha direto no Markdown. Os códigos RN, V e E vêm de [regras-de-negocio.md](regras-de-negocio.md) e [validacoes.md](validacoes.md).

## Casos de uso

Há um só ator humano. A API do Claude entra como sistema externo na geração.

```mermaid
flowchart LR
    ator(("Jovem"))
    subgraph sistema["TrampoIA"]
        uc1(["Aceitar o aviso de privacidade"])
        uc2(["Preencher o relato"])
        uc3(["Gerar currículo e entrevista"])
        uc4(["Revisar e corrigir o texto"])
        uc5(["Ler as orientações de entrevista"])
        uc6(["Copiar o texto"])
        uc7(["Baixar o PDF"])
        uc8(["Começar de novo"])
    end
    ext["API do Claude"]
    ator --> uc1
    ator --> uc2
    ator --> uc3
    ator --> uc4
    ator --> uc5
    ator --> uc6
    ator --> uc7
    ator --> uc8
    uc3 --> ext
```

## Contexto

```mermaid
flowchart LR
    jovem["Jovem no celular<br/>navegador"]
    servidor["Servidor TrampoIA<br/>Node.js e Express"]
    claude["API do Claude<br/>Anthropic"]
    jovem -- "POST /api/gerar<br/>relato" --> servidor
    servidor -- "relato mascarado" --> claude
    claude -- "JSON" --> servidor
    servidor -- "currículo filtrado" --> jovem
    jovem -- "POST /api/pdf<br/>currículo revisado" --> servidor
    servidor -- "arquivo PDF" --> jovem
```

## Para onde vai cada dado

Nome e contato só saem do navegador quando a pessoa pede o PDF, e nunca chegam ao modelo (RN13). O servidor não grava nada em nenhuma das rotas (RN12).

```mermaid
flowchart TB
    subgraph nav["Navegador, memória da aba"]
        id["Nome, contato, cidade, idade, escola"]
        rel["Relato, objetivo, vaga, nível e situação escolar"]
        rev["Currículo revisado"]
    end
    subgraph srv["Servidor, sem gravar"]
        ger["/api/gerar"]
        pdf["/api/pdf"]
    end
    cl["API do Claude<br/>apaga em até 30 dias"]
    rel --> ger
    ger -- "sem CPF, telefone e e-mail" --> cl
    id -- "só ao baixar o PDF" --> pdf
    rev -- "só ao baixar o PDF" --> pdf
```

## Fluxo de uso

```mermaid
flowchart TD
    A([Abre o link]) --> B[Lê o aviso de privacidade]
    B --> C{Aceitou?}
    C -- Não --> B
    C -- Sim --> D[Preenche o formulário]
    D --> E{Campos válidos?}
    E -- Não --> F[Erro aparece no campo] --> D
    E -- Sim --> G[Aguarda a geração]
    G --> H{Deu certo?}
    H -- Não --> I[Mensagem de erro, formulário mantido] --> D
    H -- Sim --> J[Revisa e corrige o currículo]
    J --> K[Lê as orientações de entrevista]
    K --> L{O que fazer?}
    L -- Copiar --> M[Texto copiado]
    L -- Baixar --> N[PDF no celular]
    L -- Refazer --> D
    M --> O([Fecha a aba e os dados somem])
    N --> O
```

## Sequência da geração

```mermaid
sequenceDiagram
    actor J as Jovem
    participant N as Navegador
    participant S as Servidor
    participant C as API do Claude

    J->>N: Toca em Gerar
    N->>N: Valida os campos
    N->>S: POST /api/gerar (relato, nível, vaga)
    S->>S: Confere limites e valida o corpo
    S->>S: Mascara CPF, telefone e e-mail
    loop no máximo 2 tentativas
        S->>C: POST /v1/messages
        C-->>S: JSON com currículo e entrevista
        S->>S: Valida o esquema e aplica o filtro
    end
    alt sobrou conteúdo
        S-->>N: 200, currículo sem o campo origem
        N-->>J: Tela de revisão
    else nada aproveitável
        S-->>N: 422 (E04)
        N-->>J: Pede mais detalhes, formulário intacto
    end
    J->>N: Corrige e toca em Baixar PDF
    N->>S: POST /api/pdf (identificação e currículo revisado)
    S-->>N: application/pdf
    N-->>J: Download
```

## Filtro de Integridade

```mermaid
flowchart TD
    A[Item devolvido pelo modelo] --> B{origem está na entrada?}
    B -- Não --> X[Descarta o item e conta no log]
    B -- Sim --> C{Todo número do texto está na entrada?}
    C -- Não --> X
    C -- Sim --> D{Tem termo de credencial que não está na entrada?}
    D -- Sim --> X
    D -- Não --> K[Mantém o item]
    X --> F{Fim da lista: sobrou experiência ou competência?}
    K --> F
    F -- Sim --> G[Devolve para a revisão]
    F -- Não --> H{Já tentou duas vezes?}
    H -- Não --> I[Chama o modelo de novo]
    I --> A
    H -- Sim --> J[Erro E04]
```

## Estados da tela

```mermaid
stateDiagram-v2
    [*] --> Aviso
    Aviso --> Formulario: aceita
    Formulario --> Gerando: toca em Gerar com campos válidos
    Gerando --> Revisao: resposta 200
    Gerando --> Formulario: erro, mantém o texto
    Revisao --> Revisao: corrige, copia ou baixa
    Revisao --> Formulario: volta para ajustar
    Revisao --> Aviso: começar de novo, apaga tudo
    Formulario --> Aviso: começar de novo, apaga tudo
    Revisao --> [*]: fecha a aba
```

## Contrato de dados

Não há banco. São os objetos que trafegam nas duas rotas.

```mermaid
classDiagram
    class EntradaGeracao {
        +boolean aceite
        +string nivel
        +string situacao
        +string vaga
        +string objetivo
        +string vivencias
        +string voluntariado
        +string oficinas
    }
    class RespostaGeracao {
        +string objetivo
        +string resumo
    }
    class Experiencia {
        +string titulo
        +string texto
    }
    class Item {
        +string texto
    }
    class PerguntaEntrevista {
        +string pergunta
        +string[] passos
    }
    class EntradaPdf {
        +string nome
        +string telefone
        +string email
        +string cidade
        +int idade
        +string escola
        +string nivel
        +string situacao
        +int previsao
    }
    EntradaGeracao ..> RespostaGeracao : POST /api/gerar
    RespostaGeracao "1" *-- "0..6" Experiencia
    RespostaGeracao "1" *-- "0..10" Item : competencias
    RespostaGeracao "1" *-- "0..6" Item : cursos
    RespostaGeracao "1" *-- "3..5" PerguntaEntrevista
    EntradaPdf ..> RespostaGeracao : usa o texto revisado
```

No retorno do modelo, `Experiencia` e `Item` têm também o campo `origem`, que o servidor retira antes de responder ao navegador.
