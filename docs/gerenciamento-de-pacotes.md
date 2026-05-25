# Gerenciamento de Pacotes (Go Modules)

Go Modules é o sistema oficial de gerenciamento de dependências do Go, introduzido na versão 1.11 e se tornando padrão a partir do Go 1.16. Ele resolve os problemas do `GOPATH` e permite que projetos Go existam em **qualquer diretório** do sistema, com controle preciso de versões.

---

## Conceitos Fundamentais

| Conceito | Descrição |
|---------|-----------|
| **Módulo** | Uma coleção de pacotes Go com um caminho de módulo e controle de versão |
| **Pacote** | Um diretório com arquivos `.go` que compartilham o mesmo `package` |
| **`go.mod`** | Arquivo de manifesto do módulo (dependências e versão do Go) |
| **`go.sum`** | Arquivo de checksums para verificação de integridade |

---

## Criando um Novo Módulo

```bash
# Crie e entre no diretório do projeto
mkdir meu-projeto
cd meu-projeto

# Inicializa o módulo
go mod init github.com/seu-usuario/meu-projeto
```

O arquivo `go.mod` gerado terá a seguinte estrutura:

```
module github.com/seu-usuario/meu-projeto

go 1.22
```

> [!NOTE]
> O caminho do módulo geralmente corresponde ao repositório onde o código será hospedado. Para projetos internos ou locais, qualquer string é válida (ex: `meu-projeto`), mas o padrão é usar o caminho do repositório.

---

## Estrutura de um Projeto Go

```
meu-projeto/
├── go.mod
├── go.sum
├── main.go
├── internal/           ← pacotes privados do módulo
│   └── config/
│       └── config.go
├── pkg/                ← pacotes públicos reutilizáveis
│   └── utils/
│       └── utils.go
└── cmd/                ← múltiplos executáveis
    ├── api/
    │   └── main.go
    └── worker/
        └── main.go
```

---

## Pacotes Internos

### Criando um Pacote

```go
// arquivo: pkg/calculadora/calculadora.go
package calculadora

// Funções exportadas começam com maiúscula
func Somar(a, b float64) float64 {
    return a + b
}

func Subtrair(a, b float64) float64 {
    return a - b
}

// Função privada — só acessível dentro do pacote
func validar(n float64) bool {
    return n >= 0
}
```

### Importando o Pacote

```go
// arquivo: main.go
package main

import (
    "fmt"
    "github.com/seu-usuario/meu-projeto/pkg/calculadora"
)

func main() {
    resultado := calculadora.Somar(10, 5)
    fmt.Println("Soma:", resultado) // Soma: 15
}
```

> [!TIP]
> Em Go, o **nome do identificador determina sua visibilidade**: começar com **maiúscula** = exportado (público), começar com **minúscula** = não exportado (privado ao pacote). Não existe `public` ou `private`.

---

## Adicionando Dependências Externas

### `go get` — Baixar um pacote

```bash
# Baixa e adiciona ao go.mod
go get github.com/gin-gonic/gin

# Versão específica
go get github.com/gin-gonic/gin@v1.9.1

# Última versão minor compatível
go get github.com/gin-gonic/gin@latest
```

Após o `go get`, o `go.mod` é atualizado:

```
module github.com/seu-usuario/meu-projeto

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
)
```

### `go mod tidy` — Sincronizando Dependências

```bash
# Remove dependências não usadas e adiciona as faltantes
go mod tidy
```

> [!TIP]
> Execute `go mod tidy` sempre antes de fazer commit. Ele mantém os arquivos `go.mod` e `go.sum` em sincronia com o código real.

---

## O Arquivo `go.sum`

O `go.sum` contém os **checksums criptográficos** de cada módulo baixado, garantindo que as dependências não foram adulteradas:

```
github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqJ+Jmg=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL7YRdiu2BCbJFsXLOCmLiO0DL+DM7bT0muJp+/y8=
```

> [!WARNING]
> **Nunca edite `go.sum` manualmente.** Ele é gerado automaticamente pelo Go toolchain. Sempre commite o `go.sum` junto com o `go.mod`.

---

## Comandos Principais do Go Modules

| Comando | Descrição |
|---------|-----------|
| `go mod init <caminho>` | Cria um novo módulo |
| `go mod tidy` | Sincroniza dependências com o código |
| `go mod download` | Baixa todas as dependências para o cache local |
| `go mod verify` | Verifica checksums das dependências |
| `go mod vendor` | Copia dependências para a pasta `vendor/` |
| `go mod graph` | Exibe o grafo de dependências |
| `go get <pacote>@<versão>` | Adiciona ou atualiza uma dependência |
| `go get <pacote>@none` | Remove uma dependência |
| `go list -m all` | Lista todos os módulos do projeto |

---

## Versionamento Semântico (SemVer)

Go Modules usa **Semantic Versioning** (SemVer): `MAJOR.MINOR.PATCH`

| Componente | Quando muda | Exemplo |
|-----------|------------|---------|
| `MAJOR` | Breaking changes (incompatível) | `v1.0.0` → `v2.0.0` |
| `MINOR` | Novas funcionalidades (compatível) | `v1.0.0` → `v1.1.0` |
| `PATCH` | Correções de bugs (compatível) | `v1.0.0` → `v1.0.1` |

### Módulos v2+

Para módulos com breaking changes (v2+), o caminho do módulo muda:

```go
// go.mod de um módulo v2
module github.com/usuario/lib/v2

go 1.22
```

```go
// Importação
import "github.com/usuario/lib/v2/pacote"
```

---

## Substituições Locais com `replace`

Útil durante o desenvolvimento para usar uma versão local de um módulo:

```
module github.com/seu-usuario/app

go 1.22

require (
    github.com/sua-org/lib v1.0.0
)

replace github.com/sua-org/lib => ../lib
```

> [!WARNING]
> Remova as diretivas `replace` com caminhos locais antes de publicar seu módulo ou fazer deploy em produção.

---

## Exemplo Prático — Usando o pacote `resty`

Vamos criar um cliente HTTP usando uma dependência externa:

```bash
mkdir cliente-http
cd cliente-http
go mod init github.com/exemplo/cliente-http
go get github.com/go-resty/resty/v2
```

```go
// main.go
package main

import (
    "fmt"

    "github.com/go-resty/resty/v2"
)

type Usuario struct {
    ID    int    `json:"id"`
    Nome  string `json:"name"`
    Email string `json:"email"`
}

func main() {
    client := resty.New()

    var usuario Usuario
    resp, err := client.R().
        SetResult(&usuario).
        Get("https://jsonplaceholder.typicode.com/users/1")

    if err != nil {
        fmt.Println("Erro:", err)
        return
    }

    fmt.Printf("Status: %d\n", resp.StatusCode())
    fmt.Printf("Nome:   %s\n", usuario.Nome)
    fmt.Printf("Email:  %s\n", usuario.Email)
}
```

```bash
go run main.go
```

---

## Pacotes da Biblioteca Padrão

Go possui uma biblioteca padrão rica. Alguns dos pacotes mais usados:

| Pacote | Uso |
|--------|-----|
| `fmt` | Formatação e I/O |
| `os` | Sistema operacional, arquivos |
| `io` | Interfaces de I/O |
| `bufio` | I/O com buffer |
| `strings` | Manipulação de strings |
| `strconv` | Conversão de tipos |
| `math` | Operações matemáticas |
| `math/rand` | Números aleatórios |
| `time` | Data, hora, duração |
| `net/http` | Cliente e servidor HTTP |
| `encoding/json` | Serialização JSON |
| `encoding/xml` | Serialização XML |
| `sync` | Primitivas de sincronização |
| `context` | Contexto e cancelamento |
| `log` | Logging básico |
| `errors` | Criação e inspeção de erros |
| `testing` | Framework de testes |
| `sort` | Algoritmos de ordenação |
| `regexp` | Expressões regulares |
| `path/filepath` | Manipulação de caminhos |

---

## Publicando seu Módulo

Para tornar seu módulo público e disponível via `go get`:

1. Hospede o código em um repositório público (GitHub, GitLab, etc.)
2. Adicione tags de versão usando Git:

```bash
git tag v1.0.0
git push origin v1.0.0
```

3. O módulo ficará disponível automaticamente em [pkg.go.dev](https://pkg.go.dev) após a primeira vez que alguém o importar.

> [!NOTE]
> O Go não possui um registry central como o npm (Node.js) ou PyPI (Python). Qualquer repositório Git acessível publicamente funciona como registry — basta a URL do repositório.
