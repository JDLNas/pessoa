pessoa — Integração Databricks ↔ Power BI (V0)
Projeto objetivo para ingestão e modelagem de dados no Databricks com arquitetura Medallion (Bronze → Silver → Gold) e consumo no Power BI. A V0 privilegia simplicidade e reprodutibilidade: fluxo linear, células sequenciais e sem funções personalizadas.

1) Objetivo e Escopo
Ingerir o arquivo PESSOAS.csv a partir de um Volume (landing).

Persistir dados nas camadas Bronze, Silver e Gold com Delta Lake.

Disponibilizar as tabelas Gold para o Power BI via SQL Warehouse.

Garantir nomenclatura e tipos consistentes, além de checks mínimos de qualidade.

2) Arquitetura (Medallion)
Landing (Volume UC): armazenamento de arquivos recebidos (ex.: landingzone/processar/).

Bronze (Delta): ingestão bruta com metadados técnicos (timestamp, origem).

Silver (Delta): padronização de nomes, normalização de tipos, parse de datas, remoção de duplicidades.

Gold (Delta): conjunto analítico pronto para BI e agregações leves (ex.: contagem por país/gênero).

3) Stack & Convenções
Databricks: Unity Catalog, Volumes, Delta Lake, SQL Warehouse.

Processamento: PySpark com funções nativas (sem UDFs na V0).

Consumo: Power BI via conector nativo Databricks (DirectQuery ou Import).

Convenções de nomes

Catálogo: workspace

Schemas: workspace.bronze, workspace.silver, workspace.gold

Volume (landing): workspace.default.formacaomicrosoftpowerbiprofissional

Tabelas Gold: pessoas, pessoas_por_pais_genero

Caminho base do Volume: /Volumes/workspace/default/formacaomicrosoftpowerbiprofissional

4) Contrato de Dados (V0)
Fonte: PESSOAS.csv
Colunas esperadas

Codigo (inteiro; identificador único)

PrimeiroNome (texto)

UltimoNome (texto)

email (texto; padronizar em minúsculas)

Genero (texto)

Avatar (URL)

Pais (texto)

Nascimento (data em formato dd/MM/yyyy; converter para tipo data)

Regras V0

Chave primária lógica: Codigo (garantir unicidade na Silver).

Datas inválidas/ausentes: registrar e decidir tratamento (descartar/ajustar conforme regra futura).

Campos textuais: trim e padronização de casos (minúsculas para e-mails).

5) Estrutura do Repositório (sugestão)
FONTES/ – artefatos estáticos (como o CSV), se desejar versionar.

PIPELINE/ – notebooks Databricks (Setup, Bronze, Silver, Gold).

CONSULTAS/ – SQLs auxiliares para validações.

DASHBOARD/ – artefatos do Power BI (layouts, imagens, documentação).

README.md – documentação do projeto.

6) Fluxo End-to-End (Passo a Passo, sem código)
Setup de Catálogo e Estruturas

Definir o catálogo ativo (workspace).

Criar os schemas workspace.bronze, workspace.silver, workspace.gold.

Criar (ou validar) o Volume workspace.default.formacaomicrosoftpowerbiprofissional.

Upload do Arquivo

Enviar PESSOAS.csv para o Volume em landingzone/processar/.

Conferir se o arquivo está acessível pelo caminho do Volume.

Ingestão — Bronze

Ler o CSV com schema explícito (incluindo a coluna de data como texto).

Anexar metadados técnicos (timestamp de ingestão, nome do arquivo).

Gravar a tabela Delta workspace.bronze.pessoas.

Arquivar o arquivo processado em landingzone/processados/ com carimbo de data/hora.

Padronização — Silver

Selecionar e renomear colunas para snake_case.

Normalizar tipos (inteiro, string padronizada, data em tipo nativo).

dropDuplicates pela chave codigo.

Construir nome_completo a partir de primeiro e último nome.

Gravar em workspace.silver.pessoas.

Curadoria — Gold

Derivar campos analíticos simples (ex.: idade a partir da data de nascimento).

Selecionar apenas colunas necessárias ao consumo no BI.

Gravar workspace.gold.pessoas.

Criar um agregado ilustrativo workspace.gold.pessoas_por_pais_genero.

Qualidade e Validações

Checar nulidade de data_nascimento.

Checar duplicidade da chave codigo.

Validar contagens mínimas e distribuição por país/gênero.

Verificar a existência das tabelas nas três camadas.

Publicação para BI

Garantir que o SQL Warehouse está ativo e com permissões apropriadas.

Expor as tabelas Gold para consumo externo.

7) Conexão no Power BI
Abrir/Configurar um SQL Warehouse no Databricks e coletar Server Hostname e HTTP Path.

No Power BI Desktop: Obter dados → Azure → Databricks.

Informar Hostname e HTTP Path; autenticar com Personal Access Token (ou método nativo do conector).

Selecionar Catálogo workspace → Schema gold → tabelas pessoas e pessoas_por_pais_genero.

Escolher DirectQuery (dinâmico, recomendado para dados grandes) ou Import (prototipagem).

8) Padrões de Qualidade e Modelagem (V0)
Padronização: nomes em snake_case, e-mail em minúsculas, trim em textos.

Datas: conversão para tipo nativo e verificação de formato.

Chaves: unicidade de codigo garantida na Silver.

Idempotência: pipeline repetível (sobrescrita controlada na V0).

Observabilidade mínima: contagens por camada, time de ingestão.

9) Solução de Problemas (Comum)
Erro de namespace ao criar schema: schemas usam dois níveis (catálogo.schema), enquanto volumes usam três níveis (catálogo.schema.volume).

CSV com separador/encoding divergente: ajustar parâmetros de leitura conforme a fonte real.

Datas em formato inesperado: revisar o formato de parse e alinhar com o fornecedor do arquivo.

Falha de acesso ao Warehouse: verificar permissões, estado e limites de custo/uso.

10) Critérios de Conclusão (V0)
Tabelas criadas: workspace.bronze.pessoas, workspace.silver.pessoas, workspace.gold.pessoas, workspace.gold.pessoas_por_pais_genero.

Checks de qualidade executados (nulidade de datas, unicidade de chave, contagens mínimas).

Conexão Power BI validada e visuais básicos funcionando (por país, por gênero, tabela com informações principais).

11) Roadmap
V1: ingestão incremental (Auto Loader), expectativas de qualidade, otimização (OPTIMIZE/ZORDER), particionamento.

V2: orquestração (Jobs/Workflows), ambientes (dev/hml/prd), RBAC, CI/CD de artefatos.

Power BI: RLS por país, métricas avançadas (crescimento, top N, acumulado), padronização visual corporativa.

12) Governança & Segurança (essencial)
Controle de acesso por catálogos/schemas/tabelas (princípio do menor privilégio).

Linhagem e auditoria via metadados do Delta/Unity Catalog.

Política de retenção de arquivos no Volume (processar/arquivar) e gestão de custos do Warehouse.
