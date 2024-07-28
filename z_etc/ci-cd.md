Configurar um pipeline básico de CI/CD no GitHub Actions envolve a criação de um arquivo YAML na pasta `.github/workflows/` do seu repositório. Este arquivo define os jobs e steps que serão executados automaticamente para realizar tarefas como construção, teste e implantação do seu software sempre que uma mudança for feita no repositório.

Aqui estão os fundamentos do que um arquivo `.github/workflows/ci-cd.yml` deve conter:

### Estrutura Básica do Arquivo YAML

Um pipeline CI/CD básico geralmente inclui:

1. **Definição do Evento**: Especifica quando o workflow deve ser executado.
2. **Jobs**: Define uma sequência de etapas a serem executadas.
3. **Steps**: Instruções individuais que compõem cada job.

### Exemplo de Configuração Básica

Aqui está um exemplo de um arquivo YAML básico para um pipeline CI/CD:

```yaml
name: CI/CD Pipeline

# Define os gatilhos que iniciarão o workflow
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

# Define os jobs que serão executados
jobs:
  build:
    # Define o ambiente de execução
    runs-on: ubuntu-latest

    # Define a sequência de steps para este job
    steps:
      # Step 1: Checkout do código-fonte
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Configuração do ambiente de build
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      # Step 3: Construção do projeto
      - name: Build with Maven
        run: mvn clean install

      # Step 4: Executar testes
      - name: Run tests
        run: mvn test

  deploy:
    # O job de deploy só deve rodar após o job de build
    needs: build
    runs-on: ubuntu-latest

    steps:
      # Step 1: Checkout do código-fonte
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Implantação (exemplo usando AWS CLI)
      - name: Deploy to AWS
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: 'us-east-1'
        run: |
          # Comando de exemplo para implantar na AWS
          aws s3 sync ./target/ s3://your-bucket-name --delete
```

### Explicação dos Componentes

1. **Nome do Workflow**:
   - `name`: Define o nome do workflow para facilitar a identificação no GitHub Actions.

2. **Gatilhos**:
   - `on`: Especifica os eventos que disparam o workflow. No exemplo, ele é acionado em push ou pull requests na branch `main`.

3. **Jobs**:
   - Cada job é executado em um ambiente independente. Aqui temos dois jobs: `build` e `deploy`.

4. **Steps**:
   - **Checkout do código**: Usa a ação `actions/checkout@v3` para clonar o repositório no ambiente de execução.
   - **Configuração do JDK**: Usa `actions/setup-java@v3` para definir a versão do JDK necessária para o projeto.
   - **Construção do Projeto**: Executa `mvn clean install` para construir o projeto Java usando Maven.
   - **Execução de Testes**: Executa `mvn test` para rodar os testes automatizados.
   - **Implantação**: No job `deploy`, executa comandos para implantar o software em um ambiente específico, como a AWS. Ele utiliza credenciais armazenadas como secrets no repositório.

### Boas Práticas

- **Segurança**: Armazene informações sensíveis, como chaves de acesso, nos **Secrets** do repositório, e referencie-os no arquivo YAML com `${{ secrets.NOME_DO_SECRET }}`.
- **Modularização**: Separe jobs de construção, teste e implantação para facilitar a manutenção e a execução paralela.
- **Feedback Rápido**: Priorize a execução de testes rápidos no pipeline para detectar falhas precocemente.

### Conclusão

Este exemplo fornece uma visão geral de como configurar um pipeline CI/CD básico usando GitHub Actions. À medida que você avança, pode adicionar complexidade e funcionalidades adicionais, como mais etapas de teste, linting, análise de código, deploy para diferentes ambientes, e rollback automático em caso de falhas. GitHub Actions oferece grande flexibilidade e integração com diversas ferramentas, permitindo que você adapte seus pipelines às necessidades específicas do seu projeto.




```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Build with Maven
      run: mvn clean install

    - name: Run tests
      run: mvn test

    - name: Check tests results
      if: failure()
      run: echo "Tests failed. Commit will not be pushed."

    - name: Deploy to production
      if: success() && github.ref == 'refs/heads/main'
      run: |
        echo "Deploying to production server..."
        # Adicione aqui os comandos para deploy, por exemplo:
        # scp -r . user@server:/path/to/deploy

    # - name: Deploy to production
    #   if: github.ref == 'refs/heads/main'
    #   run: |
    #     echo "Deploying to production server..."
    #     # Adicione aqui os comandos para deploy, por exemplo:
    #     # scp -r . user@server:/path/to/deploy


```