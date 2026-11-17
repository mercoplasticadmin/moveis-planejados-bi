# Planejamento e Estrutura Completa do Projeto de BI — Moveis Planejados
**Tema:** Sistema de Controle de Corte de Chapas e Aproveitamento de Material  
**Base de Dados:** `Lucas_Controle_Cortes.xlsx` (Google Sheets)  

---

## 1. Escolha do Problema

### 1.1 Contexto Real e Autenticidade do Caso
A empresa atua no segmento de **moveis planejados sob medida**, fabricando cozinhas, closets, escritorios, home offices, paineis de TV, estantes e demais mobiliarios residenciais e comerciais. A producao depende do corte preciso de chapas de madeira industrializada (MDF, MDP, Compensado e HDF) em diferentes espessuras, cores e acabamentos, fornecidas por empresas como *Duratex, Eucatex, Berneck, Arauco, Sudati e Greenplac*.

Para atender ao criterio de autenticidade da avaliacao, distingue-se claramente:
* **Fatos e Dados Reais:** A estrutura operacional da fabrica (tipos de chapa utilizados, espessuras padrao do mercado, fornecedores reais do setor, nomes de ambientes e fluxo de producao por lote/projeto).
* **Dados Simulados:** Os 100 registros de corte foram gerados com parametros realistas do setor moveleiro (faixas de entrada de 3 a 30 chapas por operacao, sobra entre 2% e 38% conforme o tipo de chapa, produtividade de 3 a 9 pecas por chapa cortada). Os nomes de clientes e operadores sao ficticios.
* **Suposicoes (Parametros de Gestao):** A meta gerencial de **85% de aproveitamento de material** (ou seja, sobra media maxima de 15%) e a meta de **5 pecas por chapa cortada** como produtividade minima.

### 1.2 O Problema e suas Consequencias
* **O Problema:** Dificuldade em monitorar o nivel de desperdicio de chapas no processo de corte e a produtividade dos operadores de forma centralizada e em tempo real. Sem um painel de controle, o gestor de producao depende de conferencias manuais em planilhas avulsas para identificar se o material esta sendo bem aproveitado ou se ha excesso de sobras e retrabalho.
* **Consequencias para a Organizacao:**
  1. **Desperdicio financeiro:** Chapas de MDF e MDP representam o principal custo de materia-prima (60-70% do custo total do movel). Uma sobra media de 20% ao inves de 15% pode significar milhares de reais desperdicados por mes.
  2. **Retrabalho e atraso nas entregas:** Cortes mal planejados geram pecas irregulares que precisam ser refeitas, atrasando a montagem e a entrega ao cliente.
  3. **Falta de visibilidade gerencial:** Sem indicadores claros, o gestor nao consegue identificar quais operadores precisam de treinamento, quais tipos de chapa geram mais desperdicio ou se a produtividade esta caindo ao longo dos meses.

### 1.3 Pergunta Analitica do Sistema
> *"Qual e o nivel atual de aproveitamento das chapas no processo de corte, qual a produtividade de cada operador em pecas por chapa cortada, e quais tipos de chapa ou periodos apresentam desempenho abaixo da meta estabelecida?"*

### 1.4 Decisao Apoiada e Tomador de Decisao
* **Quem toma a decisao:** Gestor de Producao / Encarregado do Setor de Corte.
* **Decisoes apoiadas pelo painel:**
  1. **Decisao Operacional (Curto Prazo — Producao):** Identificar operadores com produtividade abaixo da meta e redirecionar treinamento ou redistribuir tarefas entre a equipe, alem de priorizar o uso de chapas com maior indice de sobra para cortes menores e reaproveitamento.
  2. **Decisao Tatica (Medio Prazo — Compras e PCP):** Renegociar com fornecedores cujas chapas geram mais desperdicio, ajustar o plano de corte para otimizar o aproveitamento e reavaliar a necessidade de investir em software de otimizacao de corte (nesting).

---

## 2. Planejamento do Sistema

### 2.1 Dados Necessarios, Origem e Granularidade
* **De onde vem os dados:** Registros operacionais do setor de corte da fabrica, preenchidos pelo operador ou encarregado ao final de cada operacao de corte.
* **O que cada linha da planilha representa:** Cada linha representa **uma operacao de corte de um lote de chapas para um projeto/cliente especifico**, registrando a quantidade de chapas recebidas, cortadas, pecas produzidas e o percentual de sobra.
* **Estrutura de Colunas da Planilha (Google Sheets):**
  1. `DATA`: Data da operacao de corte (formato `DD/MM/AAAA`).
  2. `LOTE`: Codigo sequencial do lote de corte (ex.: `L-0001`, `L-0002`).
  3. `TIPO_CHAPA`: Categoria do material (`MDF`, `MDP`, `Compensado`, `HDF`).
  4. `ESPESSURA_MM`: Espessura da chapa em milimetros (ex.: `3`, `6`, `12`, `15`, `18`, `25`).
  5. `COR_ACABAMENTO`: Cor ou padrao de acabamento (ex.: `Branco TX`, `Freijo`, `Noce Milano`, `Carvalho Dover`).
  6. `FORNECEDOR`: Empresa fornecedora da chapa (ex.: `Duratex`, `Eucatex`, `Berneck`, `Arauco`).
  7. `CHAPAS_ENTRADA`: Quantidade de chapas recebidas/disponibilizadas para o corte.
  8. `CHAPAS_CORTADAS`: Quantidade de chapas efetivamente cortadas (sempre <= entrada).
  9. `PECAS_PRODUZIDAS`: Numero total de pecas obtidas apos o corte.
  10. `PROJETO_CLIENTE`: Identificacao do ambiente e cliente (ex.: `Cozinha - Ana Silva`).
  11. `OPERADOR`: Nome do operador da maquina de corte.
  12. `SOBRA_PERCENTUAL`: Percentual de material desperdicado na operacao (0 a 100%).
  13. `OBSERVACAO`: Anotacoes livres sobre a operacao (ex.: `Bom aproveitamento`, `Muitos recortes`).

### 2.2 Organizacao e Regras de Qualidade dos Dados
Os dados sao preparados e mantidos no **Google Sheets**, aplicando as seguintes regras de qualidade:

1. **Regra 1 — Validacao de Consistencia Entrada vs. Corte:**
   * As chapas cortadas nunca podem exceder as chapas de entrada. O sistema aplica a regra: se `CHAPAS_CORTADAS > CHAPAS_ENTRADA`, o valor e limitado ao maximo da entrada, evitando dados inconsistentes nos calculos de aproveitamento.

2. **Regra 2 — Validacao de Faixa para Sobra Percentual:**
   * O percentual de sobra deve estar entre 0% e 100%. Valores fora dessa faixa sao automaticamente ajustados pelo sistema (negativos viram 0, acima de 100 viram 100), prevenindo distorcoes nos indicadores de aproveitamento.

3. **Regra 3 — Padronizacao de Datas:**
   * Todas as datas seguem o formato `DD/MM/AAAA`. O sistema detecta automaticamente se o formato esta em `MM/DD/AAAA` (analisando valores > 12 no primeiro campo) e converte, evitando erros de agrupamento mensal nos graficos.

4. **Regra 4 — Exclusao de Registros Vazios:**
   * Linhas onde tanto `CHAPAS_ENTRADA` quanto `CHAPAS_CORTADAS` sao zero sao descartadas automaticamente, impedindo que registros em branco ou incompletos contaminem os calculos de media e os totalizadores.

### 2.3 Indicadores do Sistema e Definicao do KPI

| Atributo | Indicador 1 — **KPI Principal** | Indicador 2 — **Indicador Operacional** |
| :--- | :--- | :--- |
| **Nome do Indicador** | **Taxa de Aproveitamento de Material** | **Produtividade de Corte (Pecas por Chapa)** |
| **Formula de Calculo** | Taxa (%) = (1 - Sobra Media / 100) x 100 | Produtividade = Pecas Produzidas / Chapas Cortadas |
| **Unidade de Medida** | Percentual (`%`) | Pecas por chapa (`pc/ch`) |
| **Frequencia de Atualizacao** | Automatica (a cada ciclo de 60 segundos apos edicao na planilha) | Automatica (a cada ciclo de 60 segundos apos edicao na planilha) |
| **Meta / Parametro** | **Meta:** >= 85% de aproveitamento | **Meta:** >= 5 pecas por chapa cortada |
| **Sinalizacao de Desempenho** | Verde: >= 85% (Meta atingida) / Amarelo: 75% a 84,9% (Atencao) / Vermelho: < 75% (Critico) | Verde: >= 5 pc/ch (Meta atingida) / Amarelo: 4 a 4,9 pc/ch (Atencao) / Vermelho: < 4 pc/ch (Critico) |

* **Justificativa do KPI:** A **Taxa de Aproveitamento de Material** foi escolhida como **KPI (Key Performance Indicator)** porque reflete diretamente o impacto financeiro da operacao de corte: cada ponto percentual de melhoria no aproveitamento representa economia direta em materia-prima, que e o principal custo da producao de moveis planejados. A *Produtividade de Corte* atua como indicador operacional complementar, medindo a eficiencia tecnica do corte (quantas pecas uteis sao extraidas de cada chapa).

### 2.4 Tipo de Analise Realizada
O sistema realiza **Analise Descritiva e Diagnostica**:
* **Analise Descritiva:** Consolida os dados de todas as operacoes de corte para apresentar o panorama atual da producao — total de chapas processadas, pecas produzidas, taxa de aproveitamento global e evolucao mensal do volume de corte.
* **Analise Diagnostica:** Por meio dos graficos e filtros por **Tipo de Chapa**, **Espessura**, **Fornecedor**, **Operador** e **Periodo**, permite investigar as causas de baixo aproveitamento — identificando, por exemplo, se o desperdicio e concentrado em um tipo especifico de chapa (como Compensado, que tem sobra media maior), em um operador com menos experiencia, ou em um fornecedor cujas chapas apresentam defeitos.

### 2.5 Decisao, Acao Pratica e Acompanhamento de Melhoria
* **Resultado Possivel Identificado:** O painel revela que o aproveitamento medio esta em **85.0%** (dentro da meta) com produtividade de **6.2 pc/ch** (acima da meta), porem o tipo *Compensado* apresenta sobra media de ~30% (bem acima da media geral de 15%), e o operador com menor produtividade fica abaixo de 5 pc/ch em determinados meses.
* **Acao Concreta Apoiada:**
  1. Reforcar o treinamento do operador com menor produtividade especificamente no corte de chapas de Compensado, que exige tecnica diferente devido a maior espessura e rigidez.
  2. Avaliar a troca de fornecedor de Compensado ou renegociar as dimensoes das chapas recebidas para reduzir a sobra media.
  3. Implementar um plano de reaproveitamento das sobras maiores (> 30cm) para corte de pecas menores como fundos de gaveta e prateleiras.
* **Como Acompanhar se Houve Melhoria:** Nos meses seguintes, verificar no painel se a **Taxa de Aproveitamento (%)** se mantem >= 85% e se a **Produtividade (pc/ch)** do operador em questao subiu para >= 5 pc/ch, alem de monitorar a **Sobra Media por Tipo** para validar se a intervencao no Compensado surtiu efeito.

### 2.6 Limitacao do Sistema
* **Cuidado na Interpretacao:** O percentual de sobra registrado pelo operador e uma estimativa visual/pratica no momento do corte e pode conter imprecisao. Alem disso, o sistema nao captura automaticamente o **reaproveitamento de retalhos** — uma sobra de 20% em uma operacao pode ser parcialmente reutilizada em outra, mas isso nao e refletido nos indicadores. Portanto, a sobra real desperdicada (enviada ao descarte) pode ser menor do que a sobra registrada. Para maior precisao, seria necessario integrar o sistema com um software de otimizacao de corte (nesting) que calcula a sobra liquida.

### 2.7 Fluxo do Sistema e Conceitos de Dados

**Fluxo de Dados:**

Fontes (Ordem de Corte / Ficha de Producao) -> Extracao (Registro pelo Operador) -> Preparacao (Regras de Qualidade) -> Planilha (Google Sheets) -> Painel (HTML + JS) -> Decisao (Gestor de Producao)

* **Como a proposta utiliza BI (Business Intelligence):** O projeto estrutura dados operacionais do setor de corte em uma base padronizada no Google Sheets, calcula indicadores de desempenho (aproveitamento e produtividade) comparados a metas gerenciais e disponibiliza paineis visuais com graficos e filtros interativos em tempo real para fundamentar decisoes do gestor de producao.
* **Ha necessidade de Big Data ou Ciencia de Dados?**
  * **Big Data:** **Nao ha necessidade.** A base opera com centenas de registros estruturados (volume reduzido) e atualizacoes diarias, nao exigindo infraestrutura distribuida para alto volume ou variedade de dados.
  * **Ciencia de Dados:** **Nao e necessaria para o escopo atual**, pois formulas matematicas diretas (media de sobra, razao pecas/chapas) ja respondem a pergunta de negocio. Em uma evolucao futura com anos de historico, tecnicas de Machine Learning poderiam prever a sobra esperada por tipo de chapa e sugerir o plano de corte otimo antes da execucao.

---

## 3. Especificacao do Desenvolvimento Tecnico

1. **Base de Dados no Google Sheets (`Controle de Cortes`):**
   * Contem **100 registros** de operacoes de corte (superando o minimo de 50 registros), cobrindo o periodo de janeiro a outubro de 2025.
   * Compartilhada no Google Sheets como "Qualquer pessoa com o link" para leitura automatica pelo painel.

2. **Pagina HTML + JavaScript Conectada a Planilha:**
   * A pagina HTML utiliza a **Google Visualization API (JSONP)** para buscar dados da planilha sem restricoes de CORS, funcionando tanto de arquivo local quanto de servidor web.
   * **Atualizacao Automatica a cada 60 segundos:** Um temporizador em JavaScript (`setInterval` configurado para 60 segundos) busca os dados atualizados da planilha sem recarregar a pagina.
   * **Data e Horario da Ultima Leitura:** Exibicao visivel no cabecalho do painel com o carimbo exato (`DD/MM/AAAA HH:MM:SS`) e contagem regressiva para a proxima atualizacao.

3. **Componentes Visuais do Painel:**
   * **Cards de Indicadores:** Exibindo a **Taxa de Aproveitamento (%)** e a **Produtividade (pc/ch)** calculados dinamicamente, acompanhados de **sinalizacao visual (Verde / Amarelo / Vermelho)** em relacao a meta, alem de totalizadores de Chapas Cortadas e Pecas Produzidas.
   * **6 Filtros Combinaveis:** Tipo de Chapa, Espessura, Fornecedor, Operador, Data Inicial e Data Final.
   * **5 Graficos:**
     1. Barras: Chapas cortadas por mes;
     2. Linha: Entradas vs. Cortes mensais (evolucao);
     3. Donut: Distribuicao por tipo de chapa;
     4. Barras horizontais: Sobra media (%) por tipo de chapa;
     5. Linha com meta: Aproveitamento mensal vs. meta de 85%.
   * **Ranking de Operadores:** Barras horizontais comparativas da produtividade de cada operador com sinalizacao por cor.
   * **Tabela Completa de Registros:** Listagem detalhada de todos os cortes com colunas ordenáveis e status (BOM / REGULAR / CRITICO) por operacao.

---

## 4. Testes de Validacao e Registro de Uso de IA

### 4.1 Checklist de Testes Antes da Entrega
1. **Conferencia Independente dos Indicadores:**
   * Total de registros na base: **100 operacoes de corte**.
   * Total de chapas de entrada: **1.210** / Total de chapas cortadas: **1.150**.
   * Total de pecas produzidas: **7.111**.
   * Taxa de Aproveitamento media: **85.0%** (dentro da meta de >= 85%).
   * Produtividade global: 7.111 / 1.150 = **6.2 pc/ch** (acima da meta de >= 5 pc/ch).
2. **Teste de Funcionamento dos Filtros:** Ao selecionar um tipo de chapa (ex.: `Compensado`), os KPIs, graficos e tabela recalculam instantaneamente apenas para aquele tipo.
3. **Teste de Atualizacao Automatica (60s):** Editar uma celula no Google Sheets (ex.: alterar o valor de `SOBRA_PERCENTUAL` de um registro) e aguardar ate 60 segundos para verificar a mudanca automatica nos indicadores e o novo horario de leitura.

### 4.2 Registro de Erro Identificado e Como Foi Corrigido
* **Erro Identificado:** Durante o desenvolvimento, o painel nao conseguia conectar com a planilha do Google Sheets quando aberto como arquivo local (`file://`). O sistema tentava buscar os dados via XMLHttpRequest (PapaParse), porem o navegador Chrome bloqueia requisicoes CORS de origem `file://` para dominios externos, fazendo todas as tentativas falharem silenciosamente.
* **Como foi corrigido:** A conexao foi reescrita para utilizar a **Google Visualization API via JSONP** (injecao de tag `<script>`), que contorna completamente a restricao de CORS por nao utilizar XMLHttpRequest. Alem disso, foi identificado que a planilha utilizava `gid=716748947` (nao o padrao `gid=0`), e o codigo foi ajustado para omitir o parametro gid quando nao presente na URL, permitindo que o Google retorne automaticamente a primeira aba.

### 4.3 Registro do Uso de Ferramenta de IA
* **Ferramenta utilizada:** Claude Code (Assistente de IA da Anthropic).
* **Pedido inicial realizado:** Criacao de um sistema de BI para controle de corte de chapas de uma fabrica de moveis planejados, incluindo painel HTML conectado ao Google Sheets, geracao de 100 registros simulados em XLSX e documento de planejamento conforme orientacoes da atividade academica.
* **Correcao realizada com apoio da IA:** Identificacao e correcao do problema de conexao CORS descrito acima (substituicao de PapaParse download por JSONP) e correcao do parametro `gid=0` que impedia o acesso a planilha cuja aba possuia identificador diferente do padrao.

---

## 5. Roteiro Pratico para a Apresentacao Individual (Ate 5 Minutos)

| Tempo | Etapa da Apresentacao | O que mostrar e falar |
| :--- | :--- | :--- |
| **0:00 - 1:00** | **1. Problema, Pergunta e Decisao** | Explicar o cenario da fabrica de moveis planejados: o corte de chapas de MDF/MDP e a operacao central e o desperdicio de material impacta diretamente o custo. O sistema responde qual o nivel de aproveitamento e quais operadores ou tipos de chapa estao abaixo da meta, apoiando decisoes do gestor de producao. |
| **1:00 - 2:00** | **2. Planilha e Indicadores** | Mostrar a planilha no Google Sheets com os 100 registros e as 13 colunas. Explicar as regras de qualidade (validacao entrada >= corte, sobra entre 0-100%) e apresentar o **KPI de Aproveitamento (%)** com meta >= 85% e a **Produtividade (pc/ch)** com meta >= 5. |
| **2:00 - 3:00** | **3. Demonstracao do Painel e Filtros** | Exibir o painel HTML com os KPIs sinalizados (verde/amarelo/vermelho), aplicar um filtro ao vivo (ex.: filtrar por `Compensado` e mostrar que a sobra e maior, ou filtrar por um operador especifico) e apontar os graficos mudando em tempo real. |
| **3:00 - 4:00** | **4. Alteracao ao Vivo na Planilha** | Ir na aba do Google Sheets, alterar o valor de `SOBRA_PERCENTUAL` de um registro (ex.: mudar de 25 para 10) e voltar ao painel para demonstrar a atualizacao automatica em ate 60 segundos e o novo horario de leitura. |
| **4:00 - 5:00** | **5. Interpretacao, Acao e Limitacao** | Interpretar o resultado (Compensado tem sobra ~30%, operador X tem produtividade menor), propor acoes (treinamento direcionado, troca de fornecedor, reaproveitamento de retalhos) e citar a limitacao (sobra registrada e estimativa visual do operador, nao captura reaproveitamento posterior). |

---

## 6. Instrucoes de Acesso e Execucao

1. **Acesse a planilha no Google Sheets** pelo link compartilhado e verifique que esta como "Qualquer pessoa com o link" (Leitor ou Editor).
2. **Abra o arquivo `painel.html`** clicando duas vezes nele (abre no navegador padrao).
3. **Cole o link da planilha** no campo de texto exibido e clique em **"Conectar Planilha"**.
4. O painel carregara automaticamente os dados e atualizara a cada 60 segundos.
5. Use os filtros no topo para segmentar a analise por tipo, espessura, fornecedor, operador ou periodo.
6. Para testar a atualizacao automatica: edite qualquer celula na planilha do Google Sheets e aguarde ate 60 segundos.
