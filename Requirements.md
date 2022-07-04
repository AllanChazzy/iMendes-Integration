Requisitos - Integração Consulta Tributária Ganso iMendes
---
# Sumario
- [Sumario](#sumario)
- [Introdução](#introdução)
- [Requisitos Iniciais](#requisitos-iniciais)
- [Requisitos Específicos](#requisitos-específicos)
  - [Consulta Tributos - Cadastro de Produtos](#consulta-tributos---cadastro-de-produtos)
    - [Fluxo de Consulta](#fluxo-de-consulta)
    - [Fluxograma da Consulta Tributação](#fluxograma-da-consulta-tributação)
  - [Consulta Tributos - Gerenciador de Tributação](#consulta-tributos---gerenciador-de-tributação)
    - [Métodos Básicos](#métodos-básicos)
    - [Métodos Avançados](#métodos-avançados)

# Introdução

- O presente documento objetiva descrever em detalhes os processos e meios para integração da Consulta Tributária iMendes no Sistema Ganso. 
- Nesta integração, o Sistema Ganso comunica-se com o portal tributário do Grupo iMendes através de uma **API**, e realiza a consulta da Tributação dos produtos e retorna as informações. 
- A consulta pode ocorrer de duas maneiras: **durante a realização de um novo Cadastro** ou **consultando a tributação de um produto Cadastrado** 
- Para que a integração seja bem sucedida, é necssário que o CNPJ do estabelecimento consulente (Cliente que realizará as consultas) esteja liberado mediante aquisição de licença com o Setor Comercial do Grupo iMendes. (Levando em consideração que a Ganso Sistemas seja um parceiro ativo).

# Requisitos Iniciais
Nesta seção, serão descritos os requisitos iniciais e obrigatórios para a continuidade do processo de integração, e descreve os principais parâmetros de Configuração. 
- Para o correto funcionamento das Requisições à API iMendes, dados do Emitente da Consulta são necessários e a maioria deles estão disponíveis no Cadastro de Empresa e nos Parâmetros do Sistema Ganso. Contudo, há necessidade em criar Estrutura para as seguintes informações abaixo:
  1. (RI 1) Em **Cadastro de Empresas** (``Arquivos >> Empresa >> Empresa``), deverá existir um campo denominado **Regime Tributário** que corresponde a subclassificação do CRT **3 - Regime Normal** contendo as opções de **Lucro Real - LR** e **Lucro Presumido - LP**.
     1. Esta opção deve ser habilitada apenas se o CRT configurado for igual a 3.
     2. Esta informação é obrigatória ao enviar a requisição JSON e afeta o resultado das informações tributárias que a API retornará. Deste modo, não poderá ser fixa.
  2. (RI 2) Em **Parâmetros do Sistema** em uma Aba específica denominada **iMendes**, deverá existir campos para Configuração do acesso às APIs de **Saneamento e Envia/Recebe Dados**, que são:
     - (RI 2.1) Um parâmetro para **Ativar Integração de Consulta Tributária iMendes**.
     - (RI 2.2) Um Grupo **Configurações** contendo:
        1. **URL API Saneamento:** Link para acesso à Consulta Tributária, tipo texto com tamanho 255.
        2. **URL API Envia/Recebe Dados:** Link para acesso aos Métodos Específicos que serão descritos posteriormente, tipo texto com tamanho 255.
        3. **Senha:** Senha do parceiro integrado. Esta senha é fornecida pela iMendes, e possui padrão case sensitive. Não foi informado o tamanho máximo e as limitações, portanto, utilizar o tipo texto com tamanho de 20 caracteres sem validações.
        4. **Versão**: Versão da API contratada pelo Cliente, que deve ser definida entre as opções v2.0 e v3.0. Esta configuração deve ser discutida melhor estratégia para evitar que o Usuário selecione uma versão não contrada.
        5. **Tempo de Sincronia**: em segundos, que representa o tempo limite para receber a resposta da API. O valor padrão é 15 segundos, contudo, algumas rotinas poderão levar até 20 segundos para obter resposta. Além deste fator, o valor servirá para ajustar tempos em conexões lentas.
        6. Um Botão para verificar a disponibilidade da API, e principalmente, validar a senha informada nas configurações.
     - (RI 2.3) Um Grupo **Comportamento**: contendo Configurações da Automatização dos Processos:
        1. [x] Atualizar Tributos dos Produtos Automaticamente. 
        2. [x] Atualizar Grade Fiscal Ganso com Informações de Entrada
        3. [x] Permitir Consultar Tributos através do Cadastro de Produtos
        4. [x] Permitir Vincular Produtos utilizando Código iMendes
        5. [x] Verificar alterações de Produtos em: [   ] dias.
       - Exemplo de Tela de Configurações e Parâmetros: 
          - ![Exemplo de Parâmetros](./parametros.png)
  3. (RI 3) A Requisição iMendes requer que seja enviada uma **Operação** válida para obter o retorno com Dados Tributários. É necessária vinculação do CFOP com determinadas operações como Venda e Entrada. Por exemplo:
     - Ao realizar a consulta via Cadastro de Produtos, entende-se que o Usuário deseja atualizar informações tributárias de Saída (Venda), portanto, deve ser enviado o CFOP 5102, na requisição JSON. Mesmo que o CFOP enviado não seja o correto para a Operação, a API retornará o CFOP correto e os demais dados, inclusive, alguns de Entrada.
     - Operações de Entrada e algumas movimentações de Saída como Transferência e Devoluções, são mais abrangentes, e requerem que seja informado um CFOP coerente com a Operação. Por exemplo, para operações de Transferência, informar o CFOP 5152, e a API retornará o CFOP correto para a operação com o respectivo produto.

[Voltar ao Sumário](#sumario)

# Requisitos Específicos
Nesta seção, serão descritos os Requisitos necessários para atender à Homologação e aos Recursos mais importantes desta integração, que são:
- Funcionalidades Básicas.
- Funcionalidades Extras.
- Controle de Acesso e Acessos Restritos (Operações Especiais).
- Verificação de Conformidade com a Estrutura MVP sugerida pela iMendes.

## Consulta Tributos - Cadastro de Produtos
### Fluxo de Consulta
Conforme documentação disponibilizada pela iMendes, uma boa prática é fornecer ao Usuário um método de Consulta de Tributos no Cadastro do Produto, em dois momentos:
   1. Durante o Cadastramento de um Novo Produto;
   2. Durante o processo de Atualização de um Produto já Cadastrado.

Analisando a Estrutura de envio de Requisição, um processo comum é executado e orientado conforme os seguintes passos:
  1. Prepara a Requisição JSON, utilizando os dados do Emitente, Perfil e Produto.
     1. O Perfil possui informações chave que são:
        - UF = UF do Emitente
        - CFOP = 5102 (Operação Interna de Venda)
        - Característica Tributária = 8 (Consumidor Final Pessoa Física)
        - Finalidade = 0 (Revenda)
     2. O Produto pode ser enviado por três métodos:
        1. **Código de Barras (EAN/GTIN) e Descrição**. Identificar se o Código de Barras informado possui 8, 12, 13 ou 14 dígitos e se é um Código de Barras válido.
           1. Se Código de Barras é um GTIN válido e não iniciar com 789 ou 790, em Perfil, enviar "origem" igual a 8.
        2. **Apenas Descrição (Método Consulta Produtos iMendes)**. Quando produto não possuir Código de Barras EAN/GTIN ou em branco, e não estiver vinculado a um Código iMendes. 
        3. **Código iMendes e Descrição**. Enviar o EAN/GTIN, Descrição e Código iMendes vinculado.
   2. Realiza a Conexão com API
      1. Se método de consulta igual a 1 (Código de Barras e Descrição), enviar o Código de Barras e Descrição informados durante o Cadastramento.
      2. Se método de consulta igual a 2 (Apenas descrição), utilizar o método "Produtos iMendes" enviando apenas a descrição informada pelo Usuário.
         1. Fornecer ao Usuário um meio de selecionar um Produto localizado pela Descrição (em uma lista), e permitir vinculá-lo ao seu Cadastro utilizando o Código iMendes. O usuário poderá vincular apenas 1 Código iMendes por Produto cadastrado.
         2. Executar a Consulta de Tributação utilizando o Código iMendes vinculado. 
      3. Se método de consulta igual a 3 (Código iMendes), em Produtos, enviar preenchido o campo "CodIMendes" e a Descrição do Produto.
   2. Captura o Retorno da Tributação para a Operação padrão
   3. Efetua a Gravação de Logs de Consulta, Dados de Retorno e Produtos Alterados.

### Fluxograma da Consulta Tributação
- ![Fluxo de Consulta - Novo Produto](./Flow-Consulta-Produto.jpeg)

[Voltar ao Sumário](#sumario)

## Consulta Tributos - Gerenciador de Tributação
### Métodos Básicos
### Métodos Avançados
