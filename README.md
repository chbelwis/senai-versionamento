# FIN – Ressarcimento de Passagens Aéreas (BR)

Artefatos para importar e configurar no Orquestra BPMS 3.x.

## Arquivos
- `process/FIN_Ressarcimento_Passagens_Aereas_v1.bpmn`: diagrama BPMN 2.0 executável
- `forms/solicitacao.json`: formulário de abertura
- `forms/aprovacao_gestor.json`: formulário de aprovação do gestor
- `forms/analise_controladoria.json`: formulário de análise
- `forms/lancamento_financeiro.json`: formulário de lançamento/ERP
- `forms/programacao_pagamento.json`: formulário de pagamento
- `config/variables.json`: dicionário de variáveis
- `config/slas.yaml`: sugestão de SLAs por etapa

## Passos no Orquestra 3.x
1. Designer → Processos → Importar → selecione `process/FIN_Ressarcimento_Passagens_Aereas_v1.bpmn`.
2. Publique uma nova versão do processo.
3. Crie formulários conforme os JSONs (ou importe se seu ambiente suportar) e associe às tarefas:
   - `Task_Enviar_Solicitacao` → `forms/solicitacao.json`
   - `Task_Aprovacao_Gestor` → `forms/aprovacao_gestor.json`
   - `Task_Analise_Controladoria` → `forms/analise_controladoria.json`
   - `Task_Lancar_ERP` → `forms/lancamento_financeiro.json`
   - `Task_Programar_Pagamento` → `forms/programacao_pagamento.json`
4. Mapeie variáveis do formulário para as variáveis do processo conforme `config/variables.json`.
5. Defina SLAs por tarefa segundo `config/slas.yaml` (ajuste calendários e escalonamento).
6. Permissões sugeridas por lane:
   - Solicitante: criar/editar antes de enviar, responder pendências, leitura da sua solicitação
   - Gestor: aprovar/reprovar/ajustar
   - Controladoria: avaliar exceções e classificar
   - Financeiro: lançar ERP e programar pagamento
   - Auditoria: leitura
7. Integrações (opcional):
   - ERP: vincule a `Task_Lancar_ERP` e/ou crie uma tarefa de serviço adicional após ela
   - E-mail/Teams: configure template de notificação para atribuição, lembrete e atraso

## Regras automatizáveis (sugestão)
- `Validar Política`:
  - Prazo: até 10 dias corridos após `viagem.data_volta`
  - Itens reembolsáveis (tarifa + taxas obrigatórias)
  - Bloqueio de upgrade/assento pago sem `excecao.flag`
- `Detectar Duplicidade`:
  - Gere `controle.duplicidade_hash = sha1(passageiro.cpf + compra.eticket + viagem.data_ida)` e pesquise instâncias abertas

Ajuste nomes, papéis e limites conforme sua política interna.

## Mapeamentos lógicos (para gateways)
- `conformePolitica` = resultado booleano da tarefa automática "Validar Política"
- `precisaControladoria` = regra (ex.: valor > limite OU excecao.flag == true)
- `conformeControladoria` = decisão "Sim/Não" da `form_analise_controladoria`
- `aprovadoGestor` = (`aprovacao.gestor == 'Aprovar'`)
- `adiantamentoExiste` = `adiantamento.existe`

Implemente essas atribuições via script/serviço no Orquestra para alimentar as condições do diagrama.