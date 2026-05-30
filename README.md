Projeto CTT-AP2 - Documentação da Linguagem Go

Integrantes

* Maria Eduarda – Estrutura do Projeto e Configuração Git/GitHub
* Rodrigo Caíres – Produção da Documentação em Markdown
* Isabella Macedo Marques – CI/CD e GitHub Actions

⸻

Objetivo do Projeto

Este projeto tem como objetivo criar um site de documentação da linguagem de programação Go utilizando Markdown e a ferramenta Zensical, com publicação automática no GitHub Pages.

⸻

Fluxo de Trabalho da Equipe

A equipe adotou um fluxo baseado em Git Flow simplificado utilizando a branch principal main.

Processo de Desenvolvimento

1. Cada integrante desenvolveu suas atividades em branches próprias.
2. As alterações foram enviadas para o GitHub através de commits e push.
3. Para integração do código foi utilizado Pull Request (PR).
4. Nenhuma alteração foi realizada diretamente na branch main.
5. Todo código submetido passou por revisão antes do merge.

Processo de Revisão

As revisões foram realizadas através de Pull Requests.

Fluxo adotado:

1. Criação de branch para implementação da funcionalidade.
2. Desenvolvimento da funcionalidade.
3. Abertura do Pull Request.
4. Revisão por outro integrante da equipe.
5. Aprovação do Pull Request.
6. Merge na branch main.

A branch principal foi protegida para garantir maior controle sobre as alterações realizadas no projeto.

⸻

Estrutura da Documentação

A documentação foi construída utilizando Zensical e Markdown, contemplando os seguintes tópicos da linguagem Go:

* Introdução e Instalação
* Sintaxe Básica
* Estruturas de Controle
* Arrays, Slices e Maps
* Structs e Métodos
* Tratamento de Erros
* Goroutines e Channels
* Go Modules
* Testes

⸻

Arquitetura do Workflow GitHub Actions

O projeto utiliza GitHub Actions para automatizar a validação e publicação da documentação.

Workflow

Arquivo:

.github/workflows/docs.yml

Eventos Monitorados

O workflow é executado automaticamente nos seguintes eventos:

* Push na branch main
* Pull Requests para a branch main
* Execução semanal agendada (cron)
* Execução manual através do GitHub Actions (workflow_dispatch)

Estratégia de Build

Foi utilizada uma matriz de execução (Matrix Strategy) para validar o projeto em múltiplas versões do Python:

* Python 3.10
* Python 3.11

A configuração fail-fast: false foi utilizada para evitar que a falha em uma versão interrompa a execução da outra.

Cache de Dependências

Foi implementado cache do pip através da action:

actions/cache@v4

Objetivo:

* Reduzir tempo de execução da pipeline.
* Evitar reinstalação completa das dependências em cada execução.

Job Build

Responsável por:

1. Realizar checkout do repositório.
2. Configurar o ambiente Python.
3. Restaurar cache do pip.
4. Instalar o Zensical.
5. Gerar o site estático com:

zensical build --clean

6. Gerar e enviar o artefato da documentação.

Job Deploy

Responsável pela publicação automática no GitHub Pages.

Características:

* Executado somente após sucesso do Job Build.
* Não executa durante Pull Requests.
* Publica automaticamente o artefato gerado.

Segurança Implementada

Para evitar publicações indevidas:

* O deploy não é executado em Pull Requests.
* Apenas execuções provenientes de Push, Schedule ou Workflow Manual podem publicar a documentação.
* A branch main possui controle através de Pull Requests e revisões.

⸻

Tecnologias Utilizadas

* Go
* Markdown
* Zensical
* Git
* GitHub
* GitHub Actions
* GitHub Pages

⸻

Publicação

A documentação é gerada automaticamente e publicada através do GitHub Pages utilizando GitHub Actions.
INTEGRANTES DO GRUPO: Isabella Macedo Marques, Maria Eduarda Chaves dos Santos e Rodrigo Torres Caires 
