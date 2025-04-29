[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/aQ98rSRz)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=19361105&assignment_repo_type=AssignmentRepo)
# 🚀 Atividade - Configuração de Pipeline CI/CD com GitHub Actions

O objetivo da atividade é compreender como funciona o GitHub Actions e como ele pode ser aplicado para construir uma pipeline de build e deploy contínuo.

### 🛠️ Passo a passo

1. Gere um repositório aceitando a atividade no GitHub Classroom.
2. Clone o repositório em sua máquina local.
3. Instale as dependências usando `npm install`.
4. Na raiz do projeto, crie uma pasta `.github`.
5. Dentro da pasta `.github`, crie outra pasta chamada `workflows`.
6. Crie um arquivo chamado `CI_CD.yml` dentro de `workflows`.
7. Insira o conteúdo do pipeline abaixo e siga as instruções.

---

## 🛠️ Arquivo `CI_CD.yml` Explicado

### Definições iniciais

#### Aqui é definido o nome do nosso pipeline, apenas para referenciar quando executado nas actions do GitHub
```yaml
name: Deploy Vite para GitHub Pages
```

#### Aqui definimos o evento/trigger que irá iniciar o pipeline. Nesse caso estamos executando a cada push para a branch main
```yaml
on:
  push:
    branches: ["main"]
```

#### Definimos as permissões necessárias para criação de GitHub Pages
```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

#### Garantimos que apenas um deploy ocorra por vez
````yaml
concurrency:
  group: "pages"
  cancel-in-progress: false
````

---

### Job de Build

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Setup do Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Instalar dependências
        run: npm ci

      - name: Build do projeto com Vite
        run: npm run build

      - name: Upload dos arquivos para deploy
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist
```

**Explicação:**

- Fazemos o checkout do repositório.
- Preparamos o ambiente Node.js.
- Instalamos as dependências do projeto.
- Executamos o comando de build para gerar o conteúdo final.
- Salvamos o resultado para uso posterior no deploy.

---

### Job de Teste

```yaml
test:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - name: Checkout do código
      uses: actions/checkout@v4

    - name: Setup nodeJS
      uses: actions/setup-node@v4
      with:
        node-version: 22

    - name: Instalar dependências
      run: npm ci

    - name: Instalar playwright runner
      run: npx playwright install --with-deps

    - name: Rodar testes
      run: npx playwright test

    - uses: actions/upload-artifact@v4
      if: ${{ !cancelled() }}
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

**Explicação:**

- Rodamos novamente o checkout e o setup do Node.js.
- Instalamos dependências necessárias para rodar testes automatizados.
- Rodamos os testes usando Playwright.
- Salvamos o relatório de testes como artefato.

---

### Job de Deploy

```yaml
deploy:
  needs: [build, test]
  runs-on: ubuntu-latest
  environment:
    name: github-pages
    url: ${{ steps.deployment.outputs.page_url }}
  steps:
    - name: Deploy para GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4
```

**Explicação:**

- Só será executado se o build e os testes forem concluídos com sucesso.
- Publica o projeto gerado no GitHub Pages automaticamente.

---

### 🎯 últimas configurações

Antes de executarmos o fluxo, **é necessário ir nas configurações do repositório**:

- Vá em **Settings** → **Pages** → **Build and Deployment**
- Selecione "**GitHub Actions**" como a fonte de deploy.

### 📋 Requisitos da atividade

- O pipeline deve ser executado a cada push na branch `main`.
- O pipeline deve ter três jobs: `build`, `test` e `deploy`.
- O job `build` deve gerar o conteúdo do projeto e salvá-lo para o deploy.
- O job `test` deve rodar os testes automatizados.
- O job `deploy` deve publicar o conteúdo gerado pelo job `build` no GitHub Pages.

---
