@author - Carlos Claro \
@date - 2026-08-01 \
@title - Aplicando system design buscando otimizar processos e infraestrutura. \
@abstract - Utilizar microserviços que intercomunicão através de API's, otimização de banco de dados, integração e migração de dados.

# Projeto POW 3

## Descrição de cenários
### Cenário 1 (2002)
Primeiros softwares do guiasjp, portal, powadmin, todos baseados em php, mysql. Com uma boa performance, utilizando php5. Tambem era utilizado microsoft Access para gerenciamentos financeiros.\
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
Um pod para administrar a sessão e chaves de cada usuário, seja cliente ou agente externo. A liberação de uso é feita por aplicação.
Nucleos administrativo e cadastraveis contam com api e frontends separados, as apis podems ser consultadas e utilizadas por detentores de chaves, fontends recebem chaves e criam sessão de usuários, agente externos recebem chaves e liberações.





Pasta e GitHub do Projeto Admin
[GitHub](https://github.com/Carlos-Claro/admin3.git)
```
/admin/
```

