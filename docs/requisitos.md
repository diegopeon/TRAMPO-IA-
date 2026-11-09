# Requisitos

Os requisitos partem do documento do projeto de extensão (Entrega Parcial, Unidade 1). Prioridade "MVP" precisa estar pronta para o teste com estudantes. "Se der tempo" só entra se sobrar semana.

## Quem usa

Há um único perfil: jovem de 15 a 24 anos, estudante de escola pública ou morador de bairros populares de Salvador (Subúrbio Ferroviário, Miolo, Cabula/Tancredo Neves), em busca de vaga de Jovem Aprendiz, estágio ou primeiro emprego. Usa o próprio celular, em rede móvel, e pode ter pouca prática com ferramentas digitais. Não existe perfil de administrador, porque o sistema não guarda nada que alguém precise administrar.

## Requisitos funcionais

| ID | O sistema deve | Prioridade |
|---|---|---|
| RF01 | Mostrar o aviso de privacidade antes do formulário e exigir o aceite | MVP |
| RF02 | Apresentar o formulário em uma única página, com identificação, escolaridade e relato | MVP |
| RF03 | Mostrar pergunta e exemplo em cada campo de relato, sem pedir que a pessoa escreva comandos para a IA | MVP |
| RF04 | Validar os campos no navegador e de novo no servidor | MVP |
| RF05 | Enviar para `/api/gerar` somente relato, objetivo, vaga, nível e situação escolar | MVP |
| RF06 | Mascarar CPF, telefone e e-mail digitados no relato antes de chamar o modelo | MVP |
| RF07 | Gerar objetivo, resumo, experiências, competências, cursos e de 3 a 5 perguntas de entrevista em uma chamada | MVP |
| RF08 | Aplicar o Filtro de Integridade antes de devolver o resultado | MVP |
| RF09 | Mostrar mensagem de espera durante a geração e manter o formulário preenchido em caso de falha | MVP |
| RF10 | Exibir o currículo em campos editáveis para revisão | MVP |
| RF11 | Exibir as perguntas de entrevista com os passos de resposta | MVP |
| RF12 | Copiar o currículo revisado como texto puro | MVP |
| RF13 | Gerar o PDF a partir do texto revisado | MVP |
| RF14 | Limpar todos os dados da tela com o botão "Começar de novo" | MVP |
| RF15 | Copiar as orientações de entrevista | Se der tempo |
| RF16 | Baixar as orientações de entrevista em PDF separado | Se der tempo |

O projeto chama a parte de entrevista de simulador. No MVP ela é uma lista de perguntas com orientações, sem conversa.

## Perguntas do formulário

Cada campo de relato traz uma pergunta e um exemplo, e a pessoa responde como falaria com um amigo.

| Campo | Pergunta na tela | Exemplo mostrado |
|---|---|---|
| `vivencias` | O que você já fez, mesmo sem carteira assinada? | Ajudo minha tia no salão, atendo as clientes e marco horário. |
| `voluntariado` | Já ajudou em algum projeto, grupo ou na sua comunidade? | Organizei a arrecadação de alimentos da minha rua. |
| `oficinas` | Fez algum curso ou oficina? Onde? | Curso de informática no CRAS. |
| `objetivo` | Em que você gostaria de trabalhar? | Quero trabalhar em escritório, ajudando na parte administrativa. |

## Requisitos não funcionais

| ID | Requisito | Medida |
|---|---|---|
| RNF01 | Página leve | HTML, CSS e JS somados em até 50 KB comprimidos, sem framework, web font ou imagem |
| RNF02 | Abertura rápida | Primeira tela usável em até 5 s no perfil Slow 3G do Chrome DevTools |
| RNF03 | Tela pequena | Funciona a partir de 320 px de largura, sem rolagem horizontal |
| RNF04 | Navegadores | Chrome para Android e Samsung Internet dos últimos 3 anos; testar também o navegador interno do WhatsApp, por onde o link deve circular |
| RNF05 | Acessibilidade | Todo campo com `<label>`, contraste mínimo de 4,5:1, fonte base de 16 px, alvos de toque de 44 px |
| RNF06 | Linguagem | Textos e erros em português simples, sem termo técnico |
| RNF07 | Privacidade | Sem banco de dados, cookie, `localStorage` ou script de terceiros |
| RNF08 | Segurança | Chave da API só em variável de ambiente no servidor, HTTPS, corpo JSON de até 16 KB |
| RNF09 | Tempo de geração | Até 90 s por clique, somando as tentativas; passou disso, a chamada é cancelada |
| RNF10 | Custo para quem usa | Zero, sem anúncio e sem cobrança |
| RNF11 | PDF | Até 100 KB, texto selecionável, uma coluna, sem tabela nem imagem |
| RNF12 | Disponibilidade | No ar durante as oficinas; não há meta de uptime nesta versão |

## Ligação com os objetivos do projeto

| Objetivo específico | Onde é atendido |
|---|---|
| a) Interface responsiva de entrada de dados | RF01 a RF04, RNF01 a RNF06 |
| b) Engenharia de prompts e pipeline com LLM | RF05 a RF08, [integracao-claude.md](integracao-claude.md) |
| c) Gerador de 3 a 5 perguntas de entrevista | RF07, RF11 |
| d) Exportação em texto e PDF leve | RF12, RF13, RNF11 |
| e) Validação com estudantes | Seção seguinte |

## Validação com estudantes

Roda nas oficinas da Fase 2. As metas são uma proposta inicial do grupo.

| O que medir | Como | Meta |
|---|---|---|
| Precisão | Cada participante marca no currículo gerado tudo que não fez, e alguém do grupo confere com o relato | Nenhuma informação inventada |
| Usabilidade | Contar quem chega ao PDF sem ajuda e cronometrar | 80% sem ajuda, em até 10 minutos |
| Clareza | Perguntar, de 1 a 5, se a pessoa entendeu o que está escrito no próprio currículo | Média 4 ou mais |
| Leitura por ATS | Copiar o texto do PDF para um bloco de notas e ver se as seções saem na ordem certa | Todos os PDFs testados |
