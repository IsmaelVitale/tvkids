
RelatÃ³rio de Arquitetura: Smart TV Kids (Vanilla HTML/JS)

VisÃ£o Geral do Sistema

Um sistema de "TV Linear" simulada para crianÃ§as, focado em uso domÃ©stico (Home Lab). A aplicaÃ§Ã£o elimina o conceito de "vÃ­deo sob demanda" (onde a crianÃ§a escolhe o que e quando assistir) e impÃµe uma grade de horÃ¡rios rÃ­gida definida pelos pais.

MudanÃ§a de Paradigma: De SPA para MPA

Em vez de usar React para simular a troca de telas, o projeto agora serÃ¡ composto por arquivos HTML fÃ­sicos e independentes.
A comunicaÃ§Ã£o de dados entre essas pÃ¡ginas (ex: a pÃ¡gina de configuraÃ§Ã£o salvando um programa que a pÃ¡gina da TV precisa ler) serÃ¡ feita exclusivamente via localStorage do navegador.

O "Pulo do Gato": YouTube IFrame API para Metadados

Como nÃ£o teremos um backend e nÃ£o queremos expor uma chave de API do Google (Data API v3) no cÃ³digo fonte para buscar dados do vÃ­deo, usaremos um truque com a IFrame API:
Na tela de "Adicionar Programa", criaremos um player invisÃ­vel (ou de preview). Quando o pai colar o link, o JavaScript carrega o vÃ­deo nesse player oculto. Assim que o player carrega, o nosso script dispara player.getDuration() e player.getVideoData().title.

Resultado: O formulÃ¡rio preenche sozinho o Nome do VÃ­deo e a DuraÃ§Ã£o exata em minutos/segundos sem que o pai precise digitar!

Estrutura de Arquivos e Telas

O projeto serÃ¡ dividido em 5 arquivos HTML principais:

1. index.html (Tela Inicial)

Objetivo: O menu principal de entrada.

Interface: Dois grandes "cards" visuais.

Card 1: Ãcone de TV âž” Redireciona para tv.html.

Card 2: Ãcone de Engrenagem (Pais) âž” Redireciona para settings.html.

LÃ³gica: Apenas roteamento (links a ou window.location.href).


2. settings.html (GestÃ£o de Canais)

Objetivo: Painel raiz dos pais.

Interface: Lista os canais disponÃ­veis (Canal 1, Canal 2, Canal 3).

LÃ³gica: LÃª o localStorage para mostrar quantos programas cada canal tem. Ao clicar num canal, salva no cache qual canal foi selecionado (ex: currentEditChannel = 1) e redireciona para schedule.html.

3. schedule.html (Grade do Canal)

Objetivo: Mostrar o que vai passar e a que horas no canal selecionado.

Interface: * Se vazio: Mensagem "Adicione o primeiro programa".

Se populado: Tabela/Lista com Hora InÃ­cio, Hora Fim, TÃ­tulo do VÃ­deo e botÃ£o de excluir.

BotÃ£o flotante ou superior: (+) Adicionar Programa.

LÃ³gica: LÃª a grade especÃ­fica do canal no localStorage. Se o pai clicar em "+", redireciona para add-program.html.

4. add-program.html (FormulÃ¡rio Inteligente)

Objetivo: Adicionar um novo link Ã  grade daquele canal.

Interface: * Esquerda: Mini-grade mostrando os horÃ¡rios jÃ¡ ocupados.

Direita: Campo de Link (YouTube), Campo de TÃ­tulo, Input de "Hora de InÃ­cio".

LÃ³gica (O CoraÃ§Ã£o do Sistema):

O pai cola o link.

O JS extrai o ID (v=XYZ).

O JS inicializa um player do YouTube invisÃ­vel com esse ID.

AtravÃ©s da IFrame API, extrai o TÃ­tulo e a DuraÃ§Ã£o.

O pai define apenas a "Hora de InÃ­cio" (ex: 10:00).

O JS soma a duraÃ§Ã£o extraÃ­da com a hora de inÃ­cio para criar a "Hora Fim" (ex: 10:24).

Valida se esse bloco (10:00 - 10:24) nÃ£o choca com outro programa.

Salva no localStorage e volta para schedule.html.

5. tv.html (O Reprodutor / A TV)

Objetivo: A tela que a crianÃ§a vai assistir.

Interface: VÃ­deo em tela cheia (com a "camada de vidro" para bloquear cliques), botÃµes auto-ocultÃ¡veis para trocar de canal (CH1, CH2, CH3) e tela cheia.

LÃ³gica:

Verifica a hora exata do sistema do computador/tablet (ex: 10:15).

LÃª o localStorage do Canal Atual.

Procura se existe algum programa agendado que cubra o horÃ¡rio de 10:15.

Se existir: Pega a diferenÃ§a (15 minutos), converte para segundos (900s) e dÃ¡ um player.loadVideoById({videoId: 'XYZ', startSeconds: 900}). O vÃ­deo comeÃ§a exatamente onde deveria estar passando.

Se nÃ£o existir: Toca uma estÃ¡tica ou exibe "Fora do Ar / Intervalo".

Usa eventos da IFrame API para saber quando o vÃ­deo acaba e recalcular a grade.