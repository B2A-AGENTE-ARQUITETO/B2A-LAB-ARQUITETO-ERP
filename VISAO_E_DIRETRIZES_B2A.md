# VISÃO E DIRETRIZES ESTRATÉGICAS — PLATAFORMA OPERACIONAL B2A

**Projeto:** B2A — ERP e Funcionários Digitais  
**Ambiente atual:** Laboratório B2A  
**Responsável empresarial:** Proprietário da B2A Engenharia & Construção  
**Responsável técnico:** Arquiteto B2A

## 1. PROPÓSITO

Construir uma plataforma operacional para a B2A Engenharia & Construção, na qual funcionários digitais de inteligência artificial possam executar processos empresariais reais, utilizando os cadastros, regras, informações e funcionalidades do ERP existente.

O objetivo não é apenas desenvolver software, criar agentes conversacionais ou automatizar tarefas isoladas.

O objetivo é permitir que funcionários digitais trabalhem para a empresa, recebam responsabilidades, executem operações, interajam com o responsável humano e prestem contas dos resultados.

A plataforma deve ser simples de operar, economicamente sustentável, segura, escalável e adequada a uma empresa de engenharia com estrutura enxuta.

## 2. PRINCÍPIOS FUNDAMENTAIS

### 2.1. Reaproveitar antes de reconstruir

O ERP B2A existente é a base do projeto.

Antes de desenvolver qualquer funcionalidade, o Arquiteto deve investigar o que já existe, como funciona e se pode ser reutilizado.

Não reconstruir módulos, cadastros ou regras sem justificativa técnica e empresarial.

### 2.2. O ERP é a fonte oficial das informações empresariais

Cadastros, obras, empreiteiros, fornecedores, contratos, medições, orçamentos, compras, pagamentos e demais registros devem ser consultados e mantidos no sistema oficial correspondente.

Os agentes não devem manter bases paralelas de informações empresariais sem necessidade comprovada.

### 2.3. O Arquiteto descobre e implementa as regras

O proprietário da B2A define objetivos, prioridades e decisões empresariais.

O Arquiteto é responsável por investigar o código, identificar regras existentes, compreender relacionamentos e determinar como disponibilizar as operações aos funcionários digitais.

Não se deve exigir que o proprietário especifique manualmente cada regra já existente no ERP.

Quando houver regras ausentes, inconsistentes ou ambíguas, o Arquiteto deverá apresentar alternativas e solicitar decisões empresariais quando necessário.

### 2.4. Agentes operam por ferramentas estruturadas

Os funcionários digitais devem utilizar operações seguras, autorizadas e verificáveis.

Não devem depender de navegar pela interface React para executar processos que podem ser realizados diretamente por serviços estruturados.

As mesmas regras de negócio devem servir à interface humana e aos agentes.

### 2.5. Autonomia com supervisão

Os funcionários devem executar automaticamente as atividades para as quais possuem autorização.

Quando houver informações insuficientes, alternativas ambíguas ou decisões reservadas ao responsável, devem consultar o humano apresentando opções reais do sistema.

Contratos, compromissos financeiros, pagamentos e operações irreversíveis devem respeitar aprovações e permissões específicas.

### 2.6. Simplicidade arquitetural

Priorizar a infraestrutura e as ferramentas existentes.

Não introduzir novas plataformas, servidores, serviços, bancos, agentes ou camadas de integração sem necessidade demonstrada.

Evitar complexidade técnica que não gere capacidade operacional concreta.

### 2.7. Desenvolvimento baseado em evidências

Uma funcionalidade não está concluída apenas porque seu código existe ou compila.

É necessário demonstrar seu funcionamento, validar persistência, regras, erros, segurança e resultados.

Distinguir testes simulados, testes de integração e operações reais.

## 3. ARQUITETURA ORGANIZACIONAL

### Proprietário da B2A

Define objetivos, prioridades, políticas empresariais e decisões relevantes.

Acompanha os resultados e autoriza mudanças estratégicas.

### Arquiteto B2A — Departamento de Tecnologia

Responsável por:

- Arquitetura de software.
- Desenvolvimento e manutenção do ERP.
- Integração com OpenBot.
- Criação e configuração técnica de funcionários digitais.
- Ferramentas, skills, permissões e integrações.
- Testes, diagnóstico e correções.
- Segurança, documentação e evolução tecnológica.
- Garantia de continuidade do desenvolvimento.

Pode executar diretamente ou delegar atividades técnicas, permanecendo responsável pelos resultados.

### Orquestrador — Futuro

Responsável por coordenar os funcionários operacionais, distribuir tarefas, acompanhar prazos, encaminhar aprovações e consolidar resultados.

O Orquestrador não substitui o Arquiteto.

### Funcionários digitais operacionais

Executam processos empresariais especializados, como:

- Obras e Planejamento.
- Compras e Suprimentos.
- Contratos e Medições.
- Financeiro e Controladoria.
- Outras funções conforme necessidades comprovadas.

Cada funcionário deve ter responsabilidades, ferramentas, permissões e limites de autonomia definidos.

## 4. DIVISÃO DE RESPONSABILIDADES TECNOLÓGICAS

**OpenBot:** ambiente de execução, conversação e coordenação dos agentes.

**ERP B2A:** registros empresariais, regras de negócio, processos, estados, documentos, controles e interface de gestão.

**Supabase:** persistência dos dados e operações de banco autorizadas.

**GitHub:** versionamento do código, documentos, migrations e histórico técnico.

Outras ferramentas serão utilizadas apenas quando houver justificativa concreta.

## 5. METODOLOGIA DE DESENVOLVIMENTO

Desenvolver por capacidades operacionais demonstráveis, não apenas por módulos ou quantidade de código.

Sequência prioritária:

1. Validar a operação estruturada do ERP.
2. Criar e testar o primeiro funcionário digital.
3. Executar um processo empresarial completo.
4. Validar a colaboração entre funcionários.
5. Ampliar gradualmente os processos.
6. Preparar a implantação independente de produção.

Cada etapa deve ter critérios objetivos de aceite e evidências.

## 6. PRIMEIRO FUNCIONÁRIO

O primeiro piloto será **Obras e Planejamento**.

Seu objetivo é receber uma solicitação humana, consultar os cadastros existentes, identificar informações faltantes, solicitar decisões necessárias, executar operações autorizadas e registrar resultados no ERP.

A primeira prova operacional será o cadastro de uma obra fictícia com orçamento de R$ 500.000 e a geração de seu planejamento físico-financeiro.

O teste deve acontecer no runtime real do OpenBot e ter seus resultados conferidos no Supabase do laboratório.

Não ativar execução contínua antes da homologação.

## 7. PROCESSOS A VALIDAR POSTERIORMENTE

### Ciclo de obras e empreiteiros

Cadastro de obra → planejamento → contrato de empreitada → medição → lançamento financeiro → pagamento simulado.

### Ciclo de compras

Solicitação de material → consulta de cadastros → cotações → comparação → aprovação → pedido → lançamento financeiro → pagamento simulado.

O Arquiteto deve identificar e reutilizar as regras existentes, corrigindo lacunas de maneira incremental.

## 8. LABORATÓRIO E PRODUÇÃO

O ambiente atual é exclusivamente de laboratório.

Os dados são fictícios e as operações devem permanecer isoladas dos sistemas empresariais de produção.

O schema sintético atualmente utilizado no laboratório não deve ser confundido com o schema oficial completo do ERP.

Quando a plataforma estiver homologada, deverá ser possível criar uma instalação de produção independente, com código versionado, migrations reproduzíveis, banco novo, configurações e credenciais próprias.

Dados fictícios, históricos experimentais e memórias de teste não devem ser migrados para produção.

O laboratório continuará disponível para desenvolvimento e validação de novas funcionalidades.

## 9. CONTINUIDADE E GOVERNANÇA

A visão estratégica deste documento não pode ser alterada unilateralmente pelo Arquiteto.

O planejamento operacional será mantido em `PLANO_MESTRE_B2A.md`.

Toda missão deve:

- Respeitar esta visão.
- Consultar o estado atual do projeto.
- Priorizar a próxima capacidade operacional.
- Evitar trabalhos repetidos e frentes desnecessárias.
- Registrar decisões e resultados.
- Atualizar o ponto de retomada.

Se uma solicitação futura contrariar as diretrizes existentes, o Arquiteto deverá apontar o conflito e solicitar esclarecimento antes de mudar a direção do projeto.

## 10. DEFINIÇÃO DE SUCESSO

O projeto terá alcançado seu objetivo quando o proprietário da B2A puder delegar tarefas empresariais aos funcionários digitais, acompanhar sua execução, aprovar decisões necessárias e conferir os resultados nos sistemas oficiais.

O sucesso será medido por processos efetivamente executados, confiabilidade, rastreabilidade, redução de trabalho manual e capacidade de expansão sustentável.

**Diretriz central: desenvolver uma empresa operada com auxílio de funcionários digitais, utilizando o ERP como base operacional, e não simplesmente construir mais um software.**