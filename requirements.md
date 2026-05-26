## Requisitos Funcionais (RF)
- RF01 - Gestão de Acesso: O sistema deve permitir autenticação e controle de acesso baseado em papéis (RBAC) para os perfis: Agricultor/Produtor, Técnico Agrícola, Gestor de Cooperativa/Associação e Administrador.
- RF02 - Cadastro Social (Organizações): Deve ser possível cadastrar cooperativas, associações e comunidades rurais.
- RF03 - Cadastro Social (Produtores): O sistema deve registrar os dados do produtor, sua composição familiar, situação de adimplência e histórico de financiamentos.
- RF04 - Planejamento de Safra: O sistema deve permitir o cadastro produtivo, definindo a cultura, a área plantada, o calendário previsto e a estimativa inicial de produção.
- RF05 - Execução e Acompanhamento: O sistema deve registrar o acompanhamento real da cultura, incluindo produção efetiva, ocorrência de pragas, doenças, eventos de campo e dados climáticos observados.
- RF06 - Evidências de Campo: O sistema deve permitir anexar arquivos de mídia (fotos) e observações técnicas aos registros operacionais de campo.
- RF07 - Visualização de Dados: O sistema deve prover dashboards e relatórios exportáveis com a evolução histórica dos cultivos e da produção, filtráveis por produtor, cultura, safra e comunidade.
- RF08 - Alertas: O sistema deve disparar notificações simples para eventos críticos, como baixa produtividade estimada, pragas recorrentes ou atrasos no planejamento da safra.
- RF09 - Prognóstico Analítico: O sistema deve gerar um prognóstico inicial de produção cruzando variáveis agronômicas (área, histórico, clima, pragas).

## Requisitos Não Funcionais (RNF)
- RNF01 - Usabilidade e Plataforma: A interface deve ser web e totalmente responsiva, garantindo usabilidade tanto em desktops quanto em celulares.
- RNF02 - Disponibilidade (Offline-first): O aplicativo móvel deve suportar operação offline para coleta de dados no campo e consulta local, realizando sincronização com o servidor central assim que houver conectividade. A geração de IDs deve utilizar UUIDs para evitar colisões durante a sincronização.
- RNF03 - Arquitetura de Software: O sistema deve adotar uma arquitetura modular (como a Arquitetura Hexagonal) que privilegie clareza, baixo custo operacional e permita a evolução futura sem a necessidade de reescrita do núcleo.
- RNF04 - Auditoria: O sistema deve manter uma trilha de auditoria (Audit Log) imutável para registrar alterações relevantes em dados cadastrais e produtivos.
- RNF05 - Segurança e Privacidade: O sistema deve aplicar diretrizes rígidas de proteção de dados pessoais, especialmente no trato de informações financeiras e familiares dos produtores. Exclusões de registros devem utilizar a estratégia de soft delete.
- RNF06 - Escalabilidade: A infraestrutura deve suportar a implantação inicial como MVP (Foco em Arapiraca-AL), mas deve nascer preparada arquiteturalmente para expansão em território nacional.

## Regras de Negócio (RN)
- RN01 - Vínculo de Produtor e Comunidade: Todo produtor registrado no sistema deve pertencer obrigatoriamente a uma comunidade, que por sua vez pertence a uma organização (associação/cooperativa).
- RN02 - Multiplicidade de Assistência Técnica: Um técnico agrícola pode e deve acompanhar múltiplos produtores simultaneamente, e o sistema deve manter o histórico cronológico (N:N) dessas atribuições.
- RN03 - Hierarquia de Visibilidade: Gestores de cooperativas só podem visualizar e consolidar indicadores (produção, adimplência) dos produtores e comunidades pertencentes à sua própria organização. Administradores possuem visão irrestrita.
- RN04 - Exclusão Lógica: Nenhum dado produtivo ou social pode ser apagado permanentemente do banco de dados relacional. Todas as entidades devem utilizar exclusão lógica, garantindo integridade do histórico.
- RN05 - Autenticação Única de Produtor: Cada cadastro de Agricultor/Produtor deve possuir exatamente um usuário de acesso (login) vinculado, operando em uma relação estrita de 1:1.  
