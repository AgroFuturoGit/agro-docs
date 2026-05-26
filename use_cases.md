# Diagrama e Especificação de Casos de Uso

Esta secção descreve as interações entre os utilizadores (atores) e o sistema SmartAgro, mapeando as fronteiras do sistema e as permissões de cada perfil.
- Atores do Sistema

  - Produtor Rural (Agricultor): Utilizador final que gere a sua própria unidade produtiva, planeia as safras e regista a execução no campo.
  - Técnico Agrícola: Profissional responsável por prestar assistência técnica, acompanhar múltiplos produtores e registar ocorrências de campo.
  - Gestor de Cooperativa/Associação: Perfil administrativo que consulta relatórios consolidados e indicadores de produção da sua respetiva organização.
  - Administrador: Utilizador com controlo total sobre as configurações do sistema, cadastros base, utilizadores e logs de auditoria.
  - Sistema (Ator Secundário): Agente automatizado que executa rotinas de processamento de dados, como a geração de prognósticos e o disparo de alertas.
 
| ID | Caso de Uso | Ator Principal | Descrição Breve |
|----|-------------|----------------|-----------------|
|UC01|Autenticar no Sistema|Todos|Permite o acesso seguro com base no perfil do utilizador (RBAC).
|UC02|Gerir Dados Pessoais e Família|Produtor|Registo social do agregado familiar e dados de financiamento.
|UC03|Registar Planeamento de Safra|Produtor|Definição da área, cultura e estimativa inicial (Plano de Produção).
|UC04|Registar Execução e Colheita|Produtor|Acompanhamento diário e registo do volume real colhido (Suporta Offline).
|UC05|Registar Ocorrência de Campo|Técnico|Registo de pragas, doenças ou eventos climáticos observados na assistência.
|UC06|Anexar Evidências Fotográficas|Técnico|<> em UC05. Upload de fotos para comprovar o estado do campo.
|UC07|Acompanhar Produtores Assistidos|Técnico|Consulta à lista de agricultores vinculados à sua carteira de assistência.
|UC08|Visualizar Dashboards Agrícolas|Todos (com filtros de escopo)|Consulta gráfica de evolução de safras e produtividade.
|UC09|Consultar Indicadores Consolidados|Gestor|Monitorização agregada por comunidade ou cultura dentro da cooperativa.
|UC10|Gerir Utilizadores e Perfis|Administrador|Criação, edição e atribuição de permissões de utilizadores.
|UC11|Gerir Parâmetros Base|Administrador|Cadastro de Culturas Prioritárias, Comunidades e Organizações.
|UC12|Consultar Logs de Auditoria|Administrador|Rastreabilidade de alterações em dados sensíveis do sistema.
|UC13|Disparar Alertas Críticos|Sistema|Emissão de avisos de quebras de produção ou atrasos no calendário.
|UC14|Gerar Prognóstico de Produção|Sistema|Cálculo preditivo inicial com base no histórico e variáveis do campo.

### Detalhe de Caso de Uso Crítico
- UC04 - Registar Execução e Colheita
  - Atores: Produtor Rural (Principal).
  - Pré-condições: O utilizador deve estar autenticado e possuir um plano de produção ativo para a safra vigente.
  - Fluxo Principal (Online):
      1. O produtor seleciona o plano de produção ativo (ex: Milho - Safra 2026).
      2. O sistema exibe o formulário de acompanhamento.
      3. O produtor introduz os dados reais observados (área colhida, peso da colheita, observações).
      4. O produtor submete os dados.
      5. O sistema valida as informações, grava no banco de dados com um novo UUID e atualiza o estado do dashboard.

  - Fluxo Alternativo (Offline):
      1. O produtor acede ao sistema numa zona sem conectividade.
      2. O sistema carrega os dados locais armazenados em cache e permite preencher o formulário.
      3. O produtor submete os dados localmente.
      4. O sistema gera um UUID local, regista a marca temporal (created_at, updated_at) e armazena o registo na fila de sincronização do dispositivo.
      5. Quando a conectividade é restabelecida, o sistema envia o payload para o backend via API, resolvendo a sincronização sem colisões.
