# Introdução e Instalação

Go (também conhecida como **Golang**) é uma linguagem de programação de código aberto desenvolvida pelo Google em 2007 e lançada publicamente em 2009. Ela foi projetada por Robert Griesemer, Rob Pike e Ken Thompson com um objetivo claro: ser simples, eficiente e altamente escalável para sistemas modernos.

---

## Por que aprender Go?

Go combina a velocidade de linguagens compiladas (como C/C++) com a produtividade e legibilidade de linguagens de alto nível (como Python). Ela é amplamente utilizada em:

| Área | Exemplos de uso |
|------|----------------|
| Back-end e APIs | Servidores HTTP, REST APIs, gRPC |
| DevOps e CLIs | Docker, Kubernetes, Terraform são escritos em Go |
| Sistemas distribuídos | Microserviços, sistemas de mensageria |
| Cloud | AWS Lambda, Google Cloud Functions |

---

## Características Principais

- **Tipagem estática** com inferência de tipos
- **Compilação rápida** para um único binário executável
- **Coleta de lixo (Garbage Collection)** automática
- **Concorrência nativa** com goroutines e channels
- **Sem herança** — usa composição e interfaces
- **Biblioteca padrão rica** e multiplataforma

---

## Instalando o Go

### Windows

1. Acesse [https://go.dev/dl/](https://go.dev/dl/)
2. Baixe o instalador `.msi` para Windows
3. Execute o instalador e siga os passos
4. Abra o **Prompt de Comando** e verifique a instalação:

```bash
go version
```

### macOS

Você pode instalar via **Homebrew**:

```bash
brew install go
```

Ou baixe o instalador `.pkg` em [https://go.dev/dl/](https://go.dev/dl/).

### Linux (Ubuntu/Debian)

```bash
# Baixe a versão mais recente
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# Remova instalações anteriores e extraia
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Adicione ao PATH no seu ~/.bashrc ou ~/.zshrc
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## Verificando a Instalação

Após instalar, execute no terminal:

```bash
go version
```

A saída esperada é algo como:

```
go version go1.22.0 linux/amd64
```

> [!NOTE]
> O número da versão pode variar. O importante é que o comando seja reconhecido sem erros.

---

## Configurando o Ambiente (GOPATH e GOROOT)

O Go usa duas variáveis de ambiente principais:

| Variável | Descrição |
|----------|-----------|
| `GOROOT` | Onde o Go foi instalado (definido automaticamente) |
| `GOPATH` | Diretório de trabalho do desenvolvedor (padrão: `~/go`) |

Para verificar as configurações:

```bash
go env GOROOT
go env GOPATH
```

> [!TIP]
> A partir do Go 1.11, o uso do `GOPATH` ficou menos relevante com a introdução dos **Go Modules**. Você pode criar projetos em qualquer diretório do sistema.

---

## Seu Primeiro Programa em Go

Vamos criar o clássico **"Hello, World!"** para confirmar que tudo está funcionando.

### 1. Crie um diretório para o projeto

```bash
mkdir hello-go
cd hello-go
```

### 2. Inicialize o módulo Go

```bash
go mod init hello-go
```

### 3. Crie o arquivo `main.go`

```go
package main

import "fmt"

func main() {
    fmt.Println("Olá, Mundo!")
}
```

### 4. Execute o programa

```bash
go run main.go
```

**Saída esperada:**

```
Olá, Mundo!
```

### 5. Compilando o binário

```bash
go build -o hello main.go
./hello
```

> [!NOTE]
> O comando `go build` gera um executável nativo para o seu sistema operacional. Esse binário pode ser distribuído sem precisar instalar o Go na máquina de destino.

---

## Ferramentas Essenciais do Go

| Comando | Descrição |
|---------|-----------|
| `go run` | Compila e executa diretamente |
| `go build` | Compila e gera um binário |
| `go test` | Executa os testes do projeto |
| `go fmt` | Formata o código seguindo o padrão Go |
| `go vet` | Analisa o código em busca de erros comuns |
| `go get` | Baixa e instala dependências |
| `go mod init` | Inicializa um novo módulo |

---

## IDEs e Editores Recomendados

- **VS Code** com a extensão [Go (golang.go)](https://marketplace.visualstudio.com/items?itemName=golang.Go)
- **GoLand** — IDE da JetBrains dedicada ao Go (pago, com versão trial)
- **Neovim / Vim** com `gopls` (Language Server Protocol)

> [!TIP]
> Para iniciantes, o **VS Code + extensão Go** é a combinação mais recomendada pela comunidade. Ele oferece autocompletar, formatação automática e integração com o depurador.

---

## Recursos Oficiais

- 📘 [Documentação oficial](https://go.dev/doc/)
- 🎮 [Go Playground (online)](https://go.dev/play/)
- 📚 [A Tour of Go (tutorial interativo)](https://go.dev/tour/)
- 🔍 [pkg.go.dev — Biblioteca de pacotes](https://pkg.go.dev/)
