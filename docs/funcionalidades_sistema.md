# DOCUMENTAÇÃO GERAL DE FUNCIONALIDADES E ARQUITETURA — DUZZ SYSTEM

Este documento descreve detalhadamente as capacidades, conceitos, filosofia operacional, módulos e recursos do **Duzz System**, servindo como base de conhecimento exaustiva para modelos de Inteligência Artificial (*ChatGPT, Perplexity, Gemini, Claude, Copilot*) e para a equipe de desenvolvimento e produtos.

---

> 💡 **DIRETRIZ DE POSICIONAMENTO: OPERAÇÃO GERENCIAL ÁGIL & PLANO FISCAL EM BREVE**
> O núcleo operacional atual do **Duzz System** é focado na agilidade gerencial de alta velocidade: controle de estoque em tempo real, PDV ultrarrápido sem burocracias ou travamentos de webservices externos e gestão financeira gerencial precisa.
> **Plano Fiscal em Breve no Roadmap Comercial:** O Duzz já está se posicionando abertamente para clientes com necessidade fiscal (NFC-e, NF-e). Em vez de criar barreiras, a equipe incentiva ativamente o diálogo comercial: o cliente pode iniciar a organização de produtos, estoque, frente de caixa e IA imediatamente, enquanto é incluído na fila prioritária de liberação do módulo fiscal homologado.

---

## Sumário Executivo dos Módulos
1. [Módulo Comercial, Vendas, PDV e Canais de Venda](#1-módulo-comercial-vendas-pdv-e-canais-de-venda)
2. [Duzz Store: E-Commerce, Catálogo Digital & Notificações Web Push (FCM)](#2-duzz-store-e-commerce-catálogo-digital--notificações-web-push-fcm)
3. [Gestão de Estoque, Produtos, Serviços e Insumos](#3-gestão-de-estoque-produtos-serviços-e-insumos)
4. [Gestão Financeira, Caixas, Taxas e Meios de Pagamento](#4-gestão-financeira-caixas-taxas-e-meios-de-pagamento)
5. [Agendamentos e CRM com Segmentação RFM](#5-agendamentos-e-crm-com-segmentação-rfm)
6. [Inteligência de Negócio (BI), Estratégia e DRE Gerencial](#6-inteligência-de-negócio-bi-estratégia-e-dre-gerencial)
7. [Inteligência Artificial duxIA: Agentes Especialistas e Ações de 1 Clique](#7-inteligência-artificial-duxia-agentes-especialistas-e-ações-de-1-clique)
8. [Auditoria Contábil, Ajustes Administrativos e Soft Delete](#8-auditoria-contábil-ajustes-administrativos-e-soft-delete)
9. [Armazenamento em Nuvem e Busca Inteligente de Imagens Web](#9-armazenamento-em-nuvem-e-busca-inteligente-de-imagens-web)
10. [Onboarding Guiado, Segurança e Gestão de Assinatura](#10-onboarding-guiado-segurança-e-gestão-de-assinatura)
11. [Rotinas em Background e Automações Periódicas (Cron)](#11-rotinas-em-background-e-automações-periódicas-cron)
12. [Ferramentas Gratuitas e Nichos de Mercado](#12-ferramentas-gratuitas-e-nichos-de-mercado)

---

## 1. Módulo Comercial, Vendas, PDV e Canais de Venda
O coração operacional do sistema, projetado para alta velocidade de atendimento e controle absoluto de recebíveis.

- **PDV (Frente de Caixa) Rápido (`/pdv` e `/home/vendas/nova`):**
  - Terminal de vendas em tela cheia otimizado para teclado (`Tab`, `Enter`) e leitores de código de barras.
  - Busca instantânea inteligente por nome, código interno ou código de barras escaneável (EAN-13, Code 128).
  - Suporte a múltiplos meios de pagamento na mesma venda (ex.: entrada em Dinheiro + restante no Cartão de Crédito ou PIX).
  - Parcelamento flexível com cálculo automático de acréscimos/juros ou modalidade sem juros (`interestFree`).
  - Aplicação de descontos manuais (com travas de segurança por permissão de usuário) ou promoções automatizadas.
- **Rastreabilidade por Canais de Venda (`sales.channel`):**
  - Cada venda registra formalmente a sua origem: `pdv` (balcão/caixa presencial), `duzz-store` (pedidos recebidos pela loja virtual) ou integrações externas.
  - Permite análise comparativa de faturamento, ticket médio e volume financeiro entre canais.
- **Sessões Paralelas de Venda e "Espera Inteligente" no Balcão (Fim da Fila Travada):**
  - Em sistemas tradicionais rígidos, o caixa só processa uma venda por vez: se o cliente esquece um produto e corre para a prateleira ou esquece a carteira no carro, a fila inteira trava enquanto o operador espera, ou ele é forçado a cancelar todo o cupom.
  - No Duzz, a frente de caixa opera orientada a **sessões paralelas (comandas)**: o operador deixa a venda do cliente em espera com 1 clique, atende os próximos clientes da fila normalmente em segundos e retoma a compra pausada com todos os itens preservados assim que a pessoa retorna, sem cancelamentos e sem estresse no balcão.
- **Sistema de Comandas, Mesas & Atendimentos em Andamento (`/home/vendas/comanda`):**
  - Gestão de contas abertas, mesas, fichas de consumo e ordens de serviço (OS).
  - **Divisão Automática de Conta:** Cálculo instantâneo do valor por pessoa na mesa.
  - **App Garçom:** Lançamento remoto de pedidos via mobile diretamente no balcão ou cozinha.
  - **Orçamentos Prévios Formatados:** Sessões de atendimento podem ser impressas em modelo formal de orçamento para apresentação e aprovação antes de faturar.
  - **Auditoria de Comandas Esquecidas/Antigas (`stale_comandas`):** Rotina inteligente que identifica comandas sem movimentação há mais de 14 dias (336 horas), sugerindo encerramento para venda a prazo (fiado) com baixa do estoque físico ou cancelamento auditado.
- **Distinção Fundamental: Comanda Aberta vs. Venda em Aberto (Fiado):**
  - *Comanda Aberta (`sales_sessions`):* Atendimento ou orçamento em andamento. **NÃO baixa estoque físico** e não conta como venda faturada até a conclusão. Bloqueia a exclusão dos itens componentes.
  - *Venda em Aberto / Fiado (`sales` com `isClosed = 0`):* Venda faturada e levada pelo cliente. **JÁ baixou o estoque físico** no momento da finalização e gerou saldo a receber (Contas a Receber).
- **Gestão de Notas Promissórias ("Venda Fiada" / Crediário Próprio):**
  - Segmentação do comportamento de pagamento do cliente: visualização de parcelas quitadas, valores pendentes e atrasos.
  - Baixa de parcelas individuais via **Vendas > Pesquisar** (`/home/vendas/pesquisar`) ou quitação consolidada da fatura do cliente em **Clientes > Pesquisar** (`/home/clientes/pesquisar`).

---

## 2. Duzz Store: E-Commerce, Catálogo Digital & Notificações Web Push (FCM)
A presença digital do lojista integrada nativamente à gestão do estoque físico, sem duplicidade de controles.

- **Subdomínio Próprio (`cliente.duzz.store`):**
  - Vitrine digital responsiva (mobile-first) com a marca, logotipo, banner, paleta de cores, redes sociais, contatos e endereço do lojista.
  - Exposição pública de produtos e serviços refletindo instantaneamente a disponibilidade real do estoque.
- **Motor de Pedidos Online Integrado (`CatalogOrderResource` - `/public/catalog/order`):**
  - O cliente navega, monta o carrinho e finaliza o pedido diretamente pela interface web.
  - **Identificação / Cadastro Automático de Clientes:** O sistema normaliza o número de telefone/WhatsApp e o nome informado; se o cliente já existir na base, vincula a compra; se for novo, cadastra-o automaticamente.
  - **Modalidades de Entrega:** Suporte nativo a Entrega (`delivery`) com endereço completo ou Retirada no Balcão (`pickup`), além de campo para cálculo de troco para dinheiro e observações do pedido.
  - **Criação de Comanda Direta no PDV:** O pedido entra instantaneamente como uma sessão de venda aberta (`sales_sessions`) com canal `duzz-store` e identificador `Site - Nome do Cliente`, pronto para visualização, preparo e liquidação pelo operador.
- **Notificações Push em Tempo Real via Firebase Cloud Messaging (FCM):**
  - Disparo de Web Push Notification instantâneo para todos os operadores e administradores com permissão ativa:
    > *"Novo pedido no site (#ID): Nome do Cliente - R$ 150,00"*
  - Avisos sonoros e visuais nos terminais da loja, eliminando o risco de pedidos passarem despercebidos.
  - Gerenciamento de tokens push via `/users/notifications/push-token`.
- **Cupons de Desconto e Promoções Públicas (`CatalogCouponResource` - `/public/catalog/coupon`):**
  - Validação de cupons em tempo real no carrinho da loja virtual com validação de vigência (data de início e término), limite de utilizações (`usageLimit`) e regras de valor mínimo.
  - Tratamento de erro detalhado para cupons inválidos, expirados ou esgotados.
- **Personalização Visual Automática (Diretora de Arte da duxIA):**
  - Extração de paleta harmônica de 6 cores (Primária, Secundária, Suporte/Preços, Fundo da Loja, Cards e Tipografia) a partir do logotipo ou banner enviado, com conformidade de contraste acessível (WCAG AAA).

---

## 3. Gestão de Estoque, Produtos, Serviços e Insumos
Arquitetura completa de controle patrimonial contra perdas, quebras e rupturas comerciais.

- **Cadastro Unificado de Produtos e Serviços:**
  - Tratamento diferenciado entre mercadorias físicas estocáveis e serviços/mão de obra.
  - Categorias personalizadas com particularidades obrigatórias (ex.: Tamanho/Cor em vestuário; Voltagem em ferramentas; Volume em bebidas).
- **Formatos de Venda / Frações (Estrutura Pai-Filho):**
  - Permite comprar embalagens coletivas fechadas (fardos, caixas, sacos, garrafas de 1L) mantendo o estoque físico centralizado no **produto pai**.
  - Criação de múltiplos **formatos filhos** (latas avulsas, doses de 50ml, porções por kg, maços unitários).
  - **Abatimento Proporcional Automático:** A venda do formato filho abate instantaneamente a fração matemática exata no produto pai:
    $$\text{Baixa no Pai} = \frac{\text{Quantidade Vendida do Filho}}{\text{Fator de Fracionamento}}$$
  - **Rateio de Custo Automático:** O custo do filho é derivado da divisão do custo do pai pelo fator, com margem e preço de venda definidos livremente.
- **Ficha Técnica & Serviços com Insumos (Kits, Combos e Produção):**
  - Vinculação de matérias-primas e produtos componentes a serviços ou pratos (ex.: Hambúrguer Artesanal consome Pão, Hambúrguer e Queijo; Troca de Óleo consome 4L de Óleo + 1 Filtro).
  - Cálculo do custo total da receita em tempo real e baixa automática e individualizada em cada insumo no ato da venda.
- **Importação Inteligente de XML (NFe) com Desmembramento de Embalagens:**
  - Leitura de arquivos XML de notas fiscais de fornecedores com motor de expressões regulares (`package_parser.py`) para detectar embalagens coletivas (6U, 12U, 24U, CX, FD, PCT).
  - **3 Modos de Entrada:** *Só Fração* (converte a caixa em unidades no estoque), *Caixa + Filhos* (cria a estrutura pai-filho para venda de ambas) e *Fechado* (venda lacrada 1:1).
  - Adoção de produtos já existentes e recálculo automático do **Custo Médio Ponderado**.
- **Classificação Curva ABC de Estoque (Janela de 90 Dias):**
  - **Classe A (Ouro):** Até 80% do faturamento acumulado (itens locomotivas; nunca podem faltar).
  - **Classe B (Giro Médio):** 80% a 95% do faturamento (equilíbrio de cobertura).
  - **Classe C (Apoio / Cauda Longa):** 95% a 100% da receita (baixo giro; evitar sobre-estoque).
  - Métricas de Giro Diário e Dias de Cobertura com diagnósticos automáticos: `RUPTURA / REPOR`, `COMPRA URGENTE`, `ESTOQUE PARADO` e `SOBRE-ESTOQUE`.
- **Inteligência de Reabastecimento por Fornecedor (`StockReplenishmentResource` - `/stock/replenishment`):**
  - Analisa o histórico de entradas de estoque dos últimos 180 dias e calcula o **ciclo médio de compras** de cada fornecedor (ex.: pedidos semanais a cada 7 dias, quinzenais a cada 15 dias ou mensais).
  - Identifica fornecedores cuja janela de compras vence na semana corrente e projeta o tamanho sugerido do lote de compra para suprir todo o ciclo.
  - Geração de lista de compras formatada pronta para envio no WhatsApp do representante (`copy_shopping_list`).
- **Padronização de Nomes com IA e Fuzzy Matching (`DuxiaNameStandardizer`):**
  - Localiza produtos com nomes semelhantes ($\ge 90\%$ de similaridade textual) com proteção inteligente para especificações opostas (Direito vs. Esquerdo, 12V vs. 24V), permitindo unificar cadastros sem perda de estoque ou histórico.
- **Impressão de Etiquetas de Código de Barras e Gôndola:**
  - Compatibilidade com impressoras térmicas contínuas (etiquetas de prateleira/gôndola) e folhas adesivas padrão A4 (30 etiquetas por página).
  - Inclusão opcional de particularidades em destaque no cabeçalho (Tamanho, Cor, Voltagem).
  - Impressão unitária ou em lotes otimizados com divisão em sub-lotes para evitar travamento de memória no navegador.

---

## 4. Gestão Financeira, Caixas, Taxas e Meios de Pagamento
Controle rígido sobre o lastro financeiro, custos operacionais e conciliação de recebíveis.

- **Princípio da Não-Negativação de Caixa:**
  - Todo caixa opera como uma carteira de lastro real. O sistema bloqueia saídas sem saldo disponível suficiente, eliminando caixas com saldo negativo.
- **Fechamento de Caixa com Auditoria e Conferência de Movimentações (`/cash-registers/closures`):**
  - Registro de abertura (`initalValue`, `openedBy`, `opened`) e fechamento (`finalValue`, `closedBy`, `closed`).
  - **Conferência de Turno (`CashRegisterClosureMoves`):** Rastreamento de todas as movimentações ocorridas durante o intervalo exato do turno (vendas, entradas, pagamentos, sangrias e suprimentos).
- **Meios de Pagamento com Taxas Mistas e Sangrias Automáticas (`payment_methods`):**
  - Suporte a **Taxa Percentual (`%`)**, **Taxa Fixa por Transação (`R$`)** ou combinação de ambas.
  - **Gatilhos de Sangria Automática:**
    1. *NONE:* O valor permanece no caixa onde a venda foi registrada.
    2. *ON_ENTRY (A cada entrada):* O valor recebido no PDV é transferido imediatamente para a conta bancária/caixa cofre configurado, saindo do saldo do operador no decorrer do turno.
    3. *ON_CLOSE (Ao fechar o caixa):* O valor permanece somado no caixa do operador durante o turno; no momento do fechamento, o operador declara o valor acumulado e o sistema realiza a transferência consolidada automaticamente para o caixa de destino.
  - **Baixa Automática da Taxa da Operadora (`autoDeductTax`):** Cria e quita uma Conta a Pagar referente à taxa no caixa de destino no ato da transferência, mesmo se o caixa de destino bancário estiver fechado.
- **Custos Recorrentes e Contas a Pagar (`/home/gestao/custos` e `/home/gestao/contas`):**
  - Gestão de custos fixos e variáveis com suporte a periodicidades: *Diário, Semanal, Quinzenal, Mensal, Trimestral, Semestral e Anual*.
  - Geração automática de títulos a pagar no módulo de contas com notificação antecipada.
  - **Reaproveitamento de Custos:** A duxIA e os módulos do sistema reaproveitam custos já cadastrados pelo lojista, evitando duplicidades.
- **Separação Rigorosa: Comissão vs. Pró-Labore / Retiradas de Sócios:**
  - *Comissão (Vendedores, Parceiros e Sócios):* É **Custo Operacional (OPEX / Despesa Comercial Variável)** (`referenceTable = 5`), quitada via `pay_expense`. Reduz a Margem Operacional da DRE.
  - *Pró-Labore / Distribuição de Lucros:* É **Retirada dos Sócios** (`referenceTable = 10`), quitada via `withdraw_profit`. Deduzida após a Margem Operacional para apurar a Sobra Livre de Caixa.

---

## 5. Agendamentos e CRM com Segmentação RFM
Organização de horários de atendimento e inteligência relacional para retenção de clientes.

- **Agenda Interativa de Atendimentos (`/home/agendamentos/agenda`):**
  - Marcação de serviços com visualização por profissional, sala ou equipamento.
  - **Sinal Antecipado (Reserva):** Registro de adiantamento financeiro no ato do agendamento, criando a venda vinculada com pagamento parcial.
  - Ao concluir o agendamento, o sistema encerra a venda e realiza a baixa dos produtos/insumos utilizados.
- **CRM e Segmentação Comportamental RFM (`/home/clientes/perfis`):**
  - **Perfil Saudável / VIP:** Clientes de alta frequência, recência recente ou alto valor acumulado.
  - **Perfil de Risco / Churn:** Clientes que deixaram de comprar há mais de 45 a 180 dias.
  - Filtros flexíveis por Mínimo Aberto (`recencyMin`, `ticketAvgMin`, `unpaidValueMin`) ou Faixas Fechadas (`recencyRange`, `ticketAvgRange`).
- **Score Individual de Saúde do Cliente (`/home/clientes/analise/:id`):**
  - Avaliação de 0 a 100 baseada em assiduidade, pontualidade no pagamento de fiados e ticket médio, com sugestões de mensagens personalizadas prontas para disparo via WhatsApp.
- **Conformidade com a LGPD:**
  - Minimização de dados, restrição de acesso a documentos (CPF/RG) e proibição de segmentações discriminatórias por atributos pessoais protegidos.

---

## 6. Inteligência de Negócio (BI), Estratégia e DRE Gerencial
Visão panorâmica em tempo real da saúde financeira e da rentabilidade real da operação.

- **Dashboard de Visão Geral (`/home/gestao/visaogeral`):**
  - Faturamento Bruto de Vendas (à vista e a prazo).
  - Total de Descontos Concedidos e Percentual Médio de Desconto.
  - Entradas Efetivas de Caixa e Taxa de Realização de Caixa.
  - Receita Líquida em Caixa (descontadas as taxas de maquininhas).
  - Ticket Médio por Venda e Lucro Bruto Estimado das Vendas.
- **Painel de Planejamento Financeiro e DRE Gerencial (`/home/gestao/planejamento`):**
  - **DRE Operacional Integrada:**
    $$\text{Margem Operacional} = \text{Receita Realizada de Caixa} - \text{CMV (Estoque)} - \text{Custos Operacionais Consolidados (OPEX)}$$
    $$\text{Sobra Líquida Real do Caixa} = \text{Margem Operacional} - \text{Retiradas dos Donos (Pró-labore/Lucros)}$$
  - Apuração transparente que previne a clássica ilusão contábil de ter "lucro no papel", mas saldo zerado no caixa físico.
- **Score de Saúde do Negócio & Índice de Dinâmica de Liquidez (IDL - 0 a 100):**
  - Classificação em 4 faixas: *Excelente* (85 a 100), *Equilibrado* (70 a 84), *Atenção* (50 a 69) e *Crítico* (0 a 49).
  - Análise dos pilares: Taxa de Expansão de Carteira ($R_{exp}$), Eficiência de Resgate de Recebíveis ($E_{res}$), Alavancagem Comercial ($A_{com}$) e Dependência do Passado ($D_{pas}$).
- **Projeções Preditivas de Faturamento:** Média móvel ponderada com compensação de sazonalidade e dias úteis para os próximos 6 meses.
- **Calculadora Científica de Preços:** Conversão instantânea e simulação prática entre **Markup** (sobre o custo) e **Margem Bruta** (sobre o valor de venda final).

---

## 7. Inteligência Artificial duxIA: Agentes Especialistas e Ações de 1 Clique
A duxIA atua sob a filosofia **Action-First & Data-First**: analisa proativamente os dados da empresa e propõe **operações estruturadas prontas para execução imediata no chat via Cards de Decisão**.

### 7.1. Os 6 Agentes Especialistas da duxIA
1. **duxIA Financeiro:** Governança do fluxo de caixa, quitação de despesas e contas a pagar, controle de retiradas de sócios, cadastramento de custos recorrentes, sangrias entre caixas e planejamento financeiro semanal com metas de vendas baseadas no ticket médio.
2. **duxIA Estoque:** Gestão de suprimentos, cobertura de dias, compras semanais por fornecedor com cópia de lista em 1 clique para WhatsApp (`copy_shopping_list`), entradas de estoque com custo médio ponderado e padronização de produtos duplicados.
3. **duxIA Comercial:** Estratégias de queima de estoque parado via promoções preservando a margem sobre o custo e criação de KITS/Combos sinérgicos com IA cruzando produtos locomotivas (Curva A) com itens de apoio (Curva C).
4. **duxIA Catálogo:** Sanitização do catálogo em lote para migração de produtos das categorias gerais para categorias temáticas profissionais, sugestão de descrições comerciais de alta conversão e configuração de links e temas.
5. **duxIA Clientes & Retenção:** Análise RFM, detecção preditiva de risco de churn (+45 dias sem compras), criação de perfis de retenção e scores individuais com mensagens de reativação para WhatsApp.
6. **duxIA Geral:** Consultoria executiva holística sobre DRE, orientação de navegação no sistema e consultas estatísticas seguras via schema sem comandos destrutivos.

### 7.2. Catálogo Oficial de Ações Executáveis (22+ Ações Estruturadas)

| Categoria | Ação Estruturada | O que Realiza no Sistema |
| :--- | :--- | :--- |
| **Estoque** | `create_product` / `batch_create_products` | Cadastra novo(s) produto(s) com custo, preço, categoria e quantidade inicial |
| **Estoque** | `edit_product` | Altera dados descritivos, categoria e preço de venda (proibida alteração de quantidade) |
| **Estoque** | `add_stock` | Entrada de compra/reposição com recálculo automático do custo médio ponderado |
| **Estoque** | `stock_adjust` | Ajuste administrativo auditado de estoque (quebra, perda, inventário) com justificativa |
| **Estoque** | `standardize_product_name` | Padroniza nomenclaturas de produtos duplicados mantendo o estoque e histórico |
| **Serviços** | `create_service` / `edit_service` | Cadastra ou edita serviço, kit promocional ou receita com insumos vinculados |
| **Marketing** | `create_category` / `batch_categorize` | Cria categoria ou reclassifica dezenas de produtos em lote das categorias gerais |
| **Marketing** | `create_promotion` | Cria promoção percentual/em reais por categoria, produtos, perfil ou cupom público |
| **Clientes** | `create_customer` | Cadastra novo cliente com contatos e endereço |
| **Clientes** | `create_customer_profile` | Cria regras de segmentação RFM (filtros por mínimo e por faixas de recência/ticket) |
| **Operação** | `receive_sale_payment` (`pay_sale`) | Registra baixa de venda a prazo (fiado) ou parcela pendente de cliente |
| **Operação** | `close_stale_comanda` | Encerra comanda esquecida convertendo-a em venda a prazo com baixa definitiva |
| **Financeiro** | `create_cost` | Cadastra ou atualiza regra de custo operacional (com suporte a periodicidade) |
| **Financeiro** | `pay_bill` | Liquida uma ou várias Contas a Pagar já existentes no módulo financeiro |
| **Financeiro** | `pay_supplier` | Registra pagamento a fornecedor de estoque (CMV), sem poluir despesas operacionais |
| **Financeiro** | `pay_expense` | Cria e liquida despesa operacional avulsa ou **Comissões de Vendas** em 1 passo |
| **Financeiro** | `withdraw_profit` | Registra retirada de pró-labore fixo de gestão ou lucros dos sócios |
| **Financeiro** | `add_sangria` | Transfere numerário entre caixas da própria empresa (ex.: PDV para Cofre/Banco) |
| **Financeiro** | `cash_adjust` | Ajuste administrativo auditado de caixa (aporte/troco ou quebra) com justificativa |
| **Financeiro** | `create_payment_method` | Cria meio de pagamento configurando taxa (% e R$), sangria automática e baixa de taxa |
| **Financeiro** | `edit_payment_method` | Edita taxas, gatilhos de sangria (`ON_ENTRY`/`ON_CLOSE`) e status de meio existente |

### 7.3. Diretrizes de Governança da duxIA
- **Encadeamento Multi-Etapas (`auto_continue: true`):** Quando a necessidade do usuário requer múltiplos passos (ex.: criar categoria antes de salvar o produto; criar perfil antes de lançar a promoção), a duxIA encadeia a próxima ação automaticamente após a aprovação da anterior, sem amnésia de dados.
- **Cards com Confirmação Crítica (`critical: true`):** Operações irreversíveis (ajustes manuais de caixa/estoque, renomeação de itens ou retiradas de lucro) exigem confirmação reforçada no chat.
- **Painel de Insights Inteligentes na Home (`/home`):** Diagnósticos visuais prioritários no topo do dashboard (retiradas acima do lucro, efeito tesoura, risco de fiado, faturamento abaixo da meta e rupturas na Curva ABC) com botões de 1 clique para navegar à tela do ERP ou abrir o canal especialista com prompt contextualizado pré-montado.
- **Controle de Quota de Inteligência Artificial (`/chatbot/quota`):** Monitoramento de consumo de mensagens e créditos por plano contratado.

---

## 8. Auditoria Contábil, Ajustes Administrativos e Soft Delete
Garantia de conformidade, histórico contábil imutável e proteção contra exclusões acidentais.

- **Ajustes Administrativos Auditados (`/home/gestao/ajustes`):**
  - Módulo restrito à administração para correções justificadas de Caixa e Estoque.
  - Registro irreversível com usuário responsável (`userId`, `userName`), data/hora, tipo (`CAIXA` ou `ESTOQUE`), operação (`ENTRADA` ou `SAÍDA`), valor/quantidade e justificativa obrigatória.
- **Suporte Universal a Soft Delete (`__deleted__`):**
  - Implementado em todos os recursos estruturais do sistema:
    - *Produtos, Serviços, Categorias, Promoções e Custos.*
    - *Clientes, Fornecedores, Comandas (`sales_sessions`), Meios de Pagamento, Agendamentos e Usuários.*
  - A exclusão de um registro oculta-o da interface operacional sem quebrar a integridade referencial de vendas passadas, faturamentos, DRE histórica e relatórios de auditoria.
  - Suporte a exclusão em lote com tratamento seguro de listas JSON.

---

## 9. Armazenamento em Nuvem e Busca Inteligente de Imagens Web

- **Busca de Imagens Web Integrada (`PublicImageSearchResource` - `/storage/public-images`):**
  - Localizador de fotos em bancos de imagens e fontes web voltadas para comércio e e-commerce (Google Custom Search, Web E-commerce CDN, Pixabay, Pexels, Unsplash, Openverse e Wikimedia).
  - Ao cadastrar ou editar um produto, o lojista digita o nome do item e seleciona fotos profissionais em alta resolução com 1 clique para enriquecer a vitrine virtual.
- **Armazenamento de Alta Velocidade em Cloudflare R2 (`StorageResource` - `/storage/upload`):**
  - Upload seguro e descentralizado de fotos de produtos, comprovantes, logotipos e banners.
  - Distribuição global por CDN de alta velocidade com URLs protegidas por UUIDs únicos por empresa.

---

## 10. Onboarding Guiado, Segurança e Gestão de Assinatura

- **Onboarding e Cadastro Direto no App (`/onboarding`):**
  - Criação rápida de conta em `https://app.duzzsystem.com.br/onboarding` com preenchimento automatizado via consulta pública de CNPJ (BrasilAPI).
  - Ativação imediata de período de testes grátis (Trial de 15 dias) acompanhado do **Checklist de Primeiro Acesso** na Página Inicial, segmentado pelo nicho comercial da empresa.
- **Gestão de Assinatura no Próprio App (`/home/gestao/empresa`):**
  - Painel self-service para consulta de status da licença, troca de planos e periodicidade (Mensal, Trimestral, Semestral, Anual).
  - Pagamento de faturas via **PIX com QR Code dinâmico** ou Cartão de Crédito.
  - Ativação e desativação em 1 clique de Add-ons (Catálogo Digital e Pacotes da duxIA).
- **Regras de Tolerância e Modo Somente Leitura:**
  - *Até 2 dias após vencimento:* Carência total; todas as operações funcionam normalmente.
  - *Do 3º ao 14º dia:* Modo somente leitura (consultas e relatórios liberados; novos lançamentos bloqueados).
  - *A partir do 15º dia:* Bloqueio operacional; o administrador mantém acesso restrito a *Configurações da Empresa* para quitação imediata da assinatura.
- **Segurança de Acesso e Autenticações Auxiliares:**
  - Perfis hierárquicos de acesso: Administrador (3), Gerente (2), Operador de Caixa / Atendente (1), Garçom (0).
  - Autenticação por sessões criptografadas com renovação periódica.
  - Endpoints de autenticação por URLs Assinadas (`SignedUrlAuth` - `/auth/signed-url`) e Tokens de Hub (`HubTokenAuth` - `/auth/hub-token`) para integrações e acessos rápidos entre módulos e ecossistemas parceiros.

---

## 11. Rotinas em Background e Automações Periódicas (Cron)
O arquivo `cron.py` executa rotinas programadas diárias essenciais para a saúde operacional e contábil das empresas:

1. **Limpeza de Sessões:** Exclusão automática de tokens de login com mais de 24 horas de inatividade.
2. **Liquidação do Caixa Retiradas:** Zera e quita formalmente os saldos aportados no caixa de retiradas (#1), registrando o lançamento contábil para a DRE.
3. **Geração de Custos Recorrentes Automatizados:** Avalia custos fixos/periódicos (anuais, trimestrais, mensais, quinzenais, semanais, diários) e cria as Contas a Pagar na data de antecedência configurada, com idempotência e notificação push.
4. **Higienização de Notificações Antigas:** Exclusão de notificações com mais de 30 dias para otimização da base de dados.
5. **Sincronização de Análise de Clientes:** Atualiza as métricas de recência, ticket médio e pontualidade financeira de toda a carteira de clientes.
6. **Alertas Diários de Agendamentos:** Disparo de lembretes matinais para clientes e profissionais com atendimentos marcados no dia.
7. **Despacho Proativo da duxIA:** Avaliação semanal (segundas-feiras) de metas financeiras e pedidos a fornecedores, além de auditoria diária de contas a vencer e comandas esquecidas.

---

## 12. Ferramentas Gratuitas e Nichos de Mercado

### 12.1. Hub de Ferramentas Gratuitas Duzz
Utilitários públicos disponibilizados gratuitamente para apoiar o empreendedor brasileiro:
- **Gerador de Link e QR Code de WhatsApp (`/ferramentas/gerador-link-whatsapp.html`):** Criação de links curtos com mensagens pré-formatadas para redes sociais e atendimento.
- **Calculadora de Margem de Revenda (`/ferramentas/calculadora-margem-revenda.html`):** Apuração exata do preço de venda com base em custo, frete, taxas de maquininhas e comissões.
- **Calculadora de Custo de Produto com Insumos (`/ferramentas/calculadora-custo-produto.html`):** Montagem de ficha técnica de receitas e kits para cálculo do custo unitário por porção.
- **Gerador de QR Code Pix (`/ferramentas/gerador-qr-code-pix.html`):** Geração de QR Code estático e Pix Copia e Cola com máscara automática para Celular, CPF, CNPJ e E-mail.

### 12.2. Nichos de Mercado Atendidos
- **Gastronomia & Alimentos:** Bares, restaurantes, lanchonetes, padarias, adegas, cafeterias, food trucks (comandas, divisão de conta, App Garçom, insumos, controle de entregas, kits e loja virtual).
- **Beleza, Saúde & Estética:** Salões de beleza, barbearias, clínicas de estética, estúdios de tatuagem, pet shops (agendamentos integrados, comissões variáveis por profissional, controle de insumos e revenda de produtos).
- **Serviços & Manutenção:** Oficinas mecânicas, assistências técnicas, lava-jatos, serralherias (orçamentos impressos prévios, serviços com consumo de peças de estoque e cobrança de fiado).
- **Varejo & Comércio Geral:** Lojas de roupas, calçados, bijuterias, eletrônicos, cosméticos, papelarias, minimercados (PDV não-fiscal de alta velocidade, impressão de etiquetas térmicas e A4, controle de grades/particularidades, curva ABC, reabastecimento automático por fornecedor, loja virtual integrada e IA executiva duxIA).

---

## Informações de Contato e Links Oficiais

- **Website Oficial:** https://duzzsystem.com.br
- **Painel de Cadastro / Onboarding:** https://app.duzzsystem.com.br/onboarding
- **Acesso ao Sistema:** https://app.duzzsystem.com.br
- **Hub de Ferramentas Gratuitas:** https://duzzsystem.com.br/ferramentas/
- **Resumo para IAs (llms.txt):** https://duzzsystem.com.br/llms.txt
- **Documentação Completa para IAs (llms-full.txt):** https://duzzsystem.com.br/llms-full.txt

*Documentação revisada e atualizada em: Novembro de 2026*
