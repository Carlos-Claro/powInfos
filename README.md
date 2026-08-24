@author - Carlos Claro \
@date - 2026-08-01 \
@title - Aplicando system design buscando otimizar processos e infraestrutura. \
@abstract - Utilizar microserviços que intercomunicão através de API's, otimização de banco de dados, integração e migração de dados.

# Projeto POW 3

## Descrição de cenários
### Cenário 1 (2002)
Primeiros softwares do guiasjp, portal, powadmin, todos baseados em php, mysql. Com uma boa performance, utilizando php5. Tambem era utilizado microsoft Access para gerenciamentos financeiros.
### Cenário 2 (2014)
Mixando seus antigos software, um monolito, baseado em php7 com auxilio de Codeigniter, abrange toda a edição de dados de site, portal, financeiro, clientes, integrações com softwares externos, uso de banco de dados MYsql, MongoDB, conta tambem uma API que utiliza o framework Flask para servir microserviços em Python para integração de dados internos e criação de imagens. \
Adicionado setores financeiros, ocorrências, projetos, sistema novo de montagem de sites.\
Migração de hospedagem para SysAdmin em servidor dedicado.
### Cenário 3 (Atual) 
Proposta de dividir os serviços em núcleos de importancia, afim de otimizar o desenvolvimento e tomar decisões pontuais para cada núcleo e como será a comunicação entre eles e agentes externos.\
Criar webhooks e endpoints para comunicação com agentes externos, principalmente para o setor de imóveis, afim de extrair dados externos e enriquecer as estatisticas dos clientes de imóveis, envio contante e atualizado dos imóveis.\
Centralizar adminstração interna.\
Utilizar camadas de cache otimizadas para pesquisas e páginas de portais.
Otimização das estratégias de infraestrutura baseados em kubernetes e observabilidade em Prometheus.
## Objetivo do Projeto
- Reduzir custos de armazenamento, datacenter.
- Otimizar softwares para respostas mais efetivas, menor trafeco de dados.
- Otimizar tabelas de dados e escolher alternativas para entregas mais efetivas.
- Aplicação de cache, gateways, armazenamentos internos.

## Descrição
Visando uma performance isolada para cada núcleo do projeto, separamos cada item para adquar e reduzir o consumo, facilite o diagnóstico de qual item sente mais necessidade.\
Nos núcleos de administração com componentes CAP CA, de consistência e disponibilidade, teremos dois núcleos separados que se comunicação via api, e administram suas proprias informações com banco de dados separados, sessão compartilhada, acesso controlado pela administração.\
Nos núcleos de front-end com componentes CAP AP, de alta disponibilidade e tolerancia a partições como prioridade, incluimos cache aprimorado para reduzir os custos de consultas pesadas a banco de dados, consulta inicial via api de componenentes, verificação de integridade de contratos na api de admin.
### Núcleo principal
Pod contendo Nginx, cert-manager, PostgreSQL/MySQL \
Domínio: api.powempresas.com/admin
- Administração
    - Campanhas
    - Contratos
    - Empresas
        - Autorizadores
        - CNPJ/CPF
        - CEP
    - Financeiro
    - Ocorrências
    - Projetos
    - Publicidade
    - Serviços
    - Usuários
### Itens cadastráveis
Pod contendo nginx/apache, cert-manager, Redis (cache de respostas do banco de dados), fila, webhook, PostgreSQL/MySQL. \
Domínio: api.powempresas.com/*
- Álbuns
- Canais
- Culinária
- Clientes
- Cursos
- Dicas
- Empregos
- Ponto eletrônico
- Imóveis
- Notícias
- Nossa Cidade
- Ônibus
- Produtos
- Sites
- Rastreamento
- Vídeos
### Front-ends
Pod contendo Nginx, cert-manager, Redis, Vue.js \
Sessão compartilhada
- Administrativos - Domínio: painel.powempresas.com/admin
- Cadastráveis - Domínio: painel.powempresas.com/*
- Sites - Domínios: dos clientes
- Portais - Domínios: dos portais
- Guiasjp - Domínio: guiasjp.com

### Storages
Centralização de disco para imagens gerais, notícias, sites, cadastráveis

## Detalhamento
### Infraestrutura K8s
Servidor baseado em Kubernetes, dividido em pods para cada núcleo, certificados digitais https via cert-manager, observabilidade.

Um pod para administrar a sessão e chaves de cada usuário, seja cliente (usuário), frontend de exibição ou agente externo.

Nucleos administrativo e cadastraveis contam com api e frontends separados, as apis são consultadas e utilizadas por detentores de chaves. Cadastradores e exibidores devem receber chaves únicas, o sistema valida e atualiza sessão, agente externos recebem chaves e liberações. Aqui temos o paradigma de decisão se o administrativo e o cadastráveís de clientes fica junto, utilizando o mesmo banco de dados ou separação módulos.


### Ecossistema
Hoje o ecossistema esta centralizado entre cenario 2, e legado separado em cenario 1.

A existência de comunicação externa para sites como portaisimobiliários.com.br, que consultam diretamente do banco de dados, são um desafio de estrutura mudando o contexto de comunicação nessas aplicações, isso vai influenciar no cronograma, ativação e desativação das ferramentas desenvolvidas, além de definirmos a centralização dessas ações.

O banco de dados MySQL hoje está com colunas excessivas, que na maioria já foram transformadas em tabelas secundárias, demandando a otimização e melhoria dessas tabelas. Podendo gerar a necessidade de normatizar tabelas que ainda não foram secundarizadas. O Cache, hoje, é feito no proprio MySQL gerando um excesso de uso de memória comparado ao uso de vCPU, para otimizar isso vamos separar a responsabilidade do cache para as aplicações que necessitem. Aplicação de infraestrutura K8s com readReplicas e operators de sincronização, deixando a estrutura mais forte na escritas, com filas e webhooks de ativação de informação, delay baixo de atualização de informações.

O administrativo, tem uma grande responsabilidade, a de servir os clientes, o funcionamento da POW, os portais (imobiliários/guiasjp) e sites, isso será separado para aplicarmos o teorema CAP na decisão de CA (Consistency / Disponibilidade), aqui temos informações importante e que devem estar corretas a todo momento. \
Cada item cadastrável será isolado em API, grupo de tabelas e front-end. Infraestrutura k8s isolada.

FrontEnds são elementos isolados que utilizam as apis para alimentar, podem administrar seu proprio uso de cache e precisam de webhooks receber comandos de limpeza e verificação de integridade.

O Visual de todas as aplicações são iguais, mesmo que a interface seja diferente e em aplicações isoladas, exemplo:
- painel.powempresas.com/admin
- painel.powempresas.com/imoveis
- painel.powempresas.com/*

O itens de desenvolvimento abaixo estão em ordem de prioridade para substituição de ferramentas.

### Administrativo
Maior fragmento do sistema, contendo as tabelas de administração: empresas, cep's, serviços, ocorrências, financeiro, usuários. Disponibiliza verificações para consolidação de contratos, garantindo que o cliente está usufruindo daquele serviço, em casos como imóveis, sites, portais e guiaSJP.

Verificações de login, acesso e chaves.

Todos consomem a validação de contrato.

Banco de dados MySQL/MariaDB.
### API Imóveis
Reproduzir as funcionalidades que já temos hoje, limpando os campos de tabelas, fornecendo dados para portais, gerenciamento de cache de pesquisas buscando aliviar o banco de dados principal. 

- otimizar tabelas para retirar campos desnecessários
- salvar capa do imóvel na tabela principal, objetivo de retorno mais eficaz de pesquisas simples
- otimizar o cadastro e gerenciamento de documentos
- criar tabela para caracteristicas do imóvel e tabela de pré-definidos para exportação de dados para agentes externos.
- controle de contratos 
    - clientes (proprietários, locatários, compradores),
    - espaço cliente (consultas de faturas, dados cadastrais), 
    - contratos (ligação clientes -> modelos de contrato -> imóvel)
    - modelos de contratos do cliente
    - informações financeiras do imóvel
- integrações
    - importação xml
    - exportação xml
    - integração API de leads (webhooks)
    - exportação de leads
- contatos ligados a clientes/empresas/imoveis/contratos
    - medir qualidade/efetividade dos contratos

#### Estrutura API
Banco de dados MySQL/postgreSQL isolado com tabela principal e secundárias, formando a estrutura do serviço de imóveis.
Cache de Redis para resultados de busca de imóveis por catálogo, a limpeza é feita através de tags, quando existe modificação.
BackEnd valida acesso via administração, alimenta e recebe dados 

FrontEnd 

### Sites
Serviço de exibição de sites, utilizando o modelo de montagem 2.0, mantem em admin.powempresas.com, mas migra com o tempo para a nova, com novos sites.

Gerenciador de montagem de site consome Administrativo para validar contratos.




### Portais imobiliários
FrontEnd isolado numa estrutura K8s escalável. As consultas de imóveis serão feitas via API Imóveis, com chaves de sessão. Os contatos e interações com clientes é realizada para a API Clientes, salvando contatos. A verificação de integridade contratual é feita assincronamente e diáriamente junto a API Admin.

Vamos redefinir o uso do MongoDB, para apenas Logs/Estatísticas, sendo processados e disponibilizados por API Logs.

Pensando no conceito de teorema CAP, focando em CP (Consistency/Patition tolerance) os portais tem necessidade de leitura alta e confiabilidade de informação, com tolerância de escala e rapidez das respostas, para isso, a disponibilização das páginas será feita por cache, através de hash de url e salvamento dos resultados de pesquisa como `chave:valor`, tratando um problema antigo de páginas não existentes sendo buscadas a todo momento. O processo de alimentação do cache vai verificar o redis para saber se aquela consulta já foi feita através da chave, caso seja, já retorna rapidamente o resultado, caso não tenha resultado, busca no APIImóveis, disponibiliza o resultado na página e salva no cache do redis, diminuindo assim as consultas pesadas ao banco de dados / API Imóveis. Esse artificio vai ser aplicado a todos os portais. A Atualização se dará imóvel isoladamente e para pesquisa diariamente.

#### Demais estruturas dos portais
Menus tipos por cidade: arquivo com conteudo json por cidade, gerado diariamente.

Estatisticas de tipo/bairro/quantidade/média arquivo json, gerado semanalmente.

### GuiaSJP


### Compatibilidade

### Cronograma

#### Estrutura de chave:valor do redis
Para pesquisa: 

chave = hash dos campos pesquisados, ex: tipo_imovel=apartamento,casa&tipo_negocio=venda

valor = resultado da pesquisa com imóveis em json da API Imoveis

Para ficha do imóvel:

chave = id:9999

valor = resultado em json da APIImoveis com todas as informações.



Pasta e GitHub do Projeto Admin
[GitHub](https://github.com/Carlos-Claro/admin3.git)
```
/admin/
```

