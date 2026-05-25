# Testes Automatizados em Go

Go tem suporte nativo e de primeira classe para testes — sem precisar de frameworks externos. O pacote `testing` da biblioteca padrão, combinado com o comando `go test`, oferece tudo que é necessário para escrever testes unitários, de benchmark e de exemplo.

---

## Por que Testar em Go?

- Testes são **cidadãos de primeira classe** — integrados ao toolchain
- A convenção é **simples e direta** — sem anotações ou magia
- `go test` descobre e executa testes **automaticamente**
- Suporte nativo a **benchmarks**, **exemplos** e **fuzzing** (Go 1.18+)

---

## Estrutura Básica de um Teste

Por convenção:

- Arquivos de teste terminam com `_test.go`
- Funções de teste começam com `Test` e recebem `*testing.T`
- O arquivo de teste fica no **mesmo pacote** do código testado

```
calculadora/
├── calculadora.go
└── calculadora_test.go
```

```go
// calculadora.go
package calculadora

func Somar(a, b float64) float64 {
    return a + b
}

func Dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("divisão por zero")
    }
    return a / b, nil
}
```

```go
// calculadora_test.go
package calculadora

import (
    "testing"
)

func TestSomar(t *testing.T) {
    resultado := Somar(2, 3)
    esperado := 5.0

    if resultado != esperado {
        t.Errorf("Somar(2, 3) = %.1f; esperado %.1f", resultado, esperado)
    }
}
```

### Executando os Testes

```bash
# Rodar todos os testes no diretório atual
go test ./...

# Rodar testes de um pacote específico
go test ./calculadora/...

# Modo verbose (mostra o nome de cada teste)
go test -v ./...

# Rodar um teste específico
go test -run TestSomar ./...
```

**Saída verbose:**
```
=== RUN   TestSomar
--- PASS: TestSomar (0.00s)
PASS
ok      github.com/exemplo/calculadora  0.001s
```

---

## Métodos do `testing.T`

| Método | Descrição |
|--------|-----------|
| `t.Errorf(fmt, ...)` | Registra falha mas **continua** o teste |
| `t.Fatalf(fmt, ...)` | Registra falha e **para** o teste imediatamente |
| `t.Logf(fmt, ...)` | Loga mensagem (visível com `-v`) |
| `t.Fail()` | Marca o teste como falho (continua) |
| `t.FailNow()` | Marca como falho e para imediatamente |
| `t.Skip(msg)` | Pula o teste com uma mensagem |
| `t.Helper()` | Marca como helper (melhora relatórios de erro) |

---

## Table-Driven Tests (Testes com Tabela)

O padrão mais comum em Go: define casos de teste como uma tabela de structs:

```go
func TestDividir(t *testing.T) {
    casos := []struct {
        nome     string
        a        float64
        b        float64
        esperado float64
        erro     bool
    }{
        {"divisão simples", 10, 2, 5, false},
        {"divisão com decimal", 7, 2, 3.5, false},
        {"divisão por zero", 5, 0, 0, true},
        {"zero dividido", 0, 5, 0, false},
        {"negativos", -10, 2, -5, false},
    }

    for _, tc := range casos {
        t.Run(tc.nome, func(t *testing.T) {
            resultado, err := Dividir(tc.a, tc.b)

            if tc.erro {
                if err == nil {
                    t.Errorf("esperava erro, mas não houve")
                }
                return
            }

            if err != nil {
                t.Fatalf("erro inesperado: %v", err)
            }

            if resultado != tc.esperado {
                t.Errorf("Dividir(%.1f, %.1f) = %.1f; esperado %.1f",
                    tc.a, tc.b, resultado, tc.esperado)
            }
        })
    }
}
```

**Saída:**
```
=== RUN   TestDividir
=== RUN   TestDividir/divisão_simples
=== RUN   TestDividir/divisão_com_decimal
=== RUN   TestDividir/divisão_por_zero
=== RUN   TestDividir/zero_dividido
=== RUN   TestDividir/negativos
--- PASS: TestDividir (0.00s)
```

> [!TIP]
> O padrão de **Table-Driven Tests** é o idioma Go padrão para testes. Ele torna fácil adicionar novos casos, melhora a legibilidade e isola falhas por subteste.

---

## Setup e Teardown com `TestMain`

Para executar código antes e depois de todos os testes do pacote:

```go
package calculadora

import (
    "fmt"
    "os"
    "testing"
)

func TestMain(m *testing.M) {
    fmt.Println("🚀 Inicializando ambiente de teste...")

    // Setup: conectar DB, criar arquivos temporários, etc.

    codigo := m.Run() // executa todos os testes

    // Teardown: limpar recursos
    fmt.Println("🧹 Limpando ambiente de teste...")

    os.Exit(codigo)
}
```

### `t.Cleanup` — Limpeza por Teste

```go
func TestComBancoDeDados(t *testing.T) {
    db := abrirBancoDeTeste()

    t.Cleanup(func() {
        db.Close()        // executado ao final do teste
        limparTabelas(db) // mesmo se o teste falhar
    })

    // ... teste ...
}
```

---

## Benchmarks

Funções de benchmark começam com `Benchmark` e recebem `*testing.B`:

```go
func BenchmarkSomar(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Somar(10, 20) // executado b.N vezes
    }
}

func BenchmarkDividir(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Dividir(100, 7)
    }
}
```

```bash
# Executar benchmarks
go test -bench=. ./...

# Com relatório de alocações de memória
go test -bench=. -benchmem ./...
```

**Saída:**
```
BenchmarkSomar-8     1000000000   0.2541 ns/op
BenchmarkDividir-8   765432100    1.567 ns/op   0 B/op   0 allocs/op
```

> [!NOTE]
> `b.N` é ajustado automaticamente pelo framework para obter medições estatisticamente confiáveis. Nunca codifique um valor fixo para `b.N`.

---

## Testes de Exemplo (Examples)

Funções de exemplo começam com `Example` e servem como **documentação executável**:

```go
func ExampleSomar() {
    resultado := Somar(3, 4)
    fmt.Println(resultado)
    // Output: 7
}

func ExampleDividir() {
    resultado, err := Dividir(10, 2)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    fmt.Println(resultado)
    // Output: 5
}
```

- O `// Output:` é verificado pelo `go test`
- Aparece automaticamente no `go doc` e em [pkg.go.dev](https://pkg.go.dev)

---

## Cobertura de Testes

```bash
# Exibe percentual de cobertura
go test -cover ./...

# Gera relatório detalhado
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out  # abre no navegador

# Modo de cobertura por instruções (padrão) ou branches
go test -covermode=atomic ./...
```

**Exemplo de saída:**
```
ok      github.com/exemplo/calculadora   coverage: 87.5% of statements
```

> [!TIP]
> Almeje cobertura alta nos caminhos críticos, não necessariamente 100% do código. Testar casos de erro e edge cases é mais valioso do que atingir 100% de cobertura em código trivial.

---

## Testando com Mocks — Interfaces

A forma idiomática de testar código com dependências externas em Go é usar **interfaces**:

```go
// producao.go
package servico

type EmailSender interface {
    Enviar(para, assunto, corpo string) error
}

type Notificador struct {
    email EmailSender
}

func (n *Notificador) NotificarUsuario(usuario, mensagem string) error {
    return n.email.Enviar(
        usuario+"@empresa.com",
        "Notificação",
        mensagem,
    )
}
```

```go
// notificador_test.go
package servico

import (
    "fmt"
    "testing"
)

// Mock do EmailSender
type emailMock struct {
    ultimoEnvio struct {
        para    string
        assunto string
    }
    erroSimulado error
}

func (m *emailMock) Enviar(para, assunto, corpo string) error {
    m.ultimoEnvio.para = para
    m.ultimoEnvio.assunto = assunto
    return m.erroSimulado
}

func TestNotificarUsuario(t *testing.T) {
    mock := &emailMock{}
    notif := &Notificador{email: mock}

    err := notif.NotificarUsuario("alice", "Bem-vinda!")
    if err != nil {
        t.Fatalf("erro inesperado: %v", err)
    }

    if mock.ultimoEnvio.para != "alice@empresa.com" {
        t.Errorf("email enviado para %q; esperado %q",
            mock.ultimoEnvio.para, "alice@empresa.com")
    }
}

func TestNotificarUsuarioComErro(t *testing.T) {
    mock := &emailMock{erroSimulado: fmt.Errorf("servidor SMTP indisponível")}
    notif := &Notificador{email: mock}

    err := notif.NotificarUsuario("bob", "Olá!")
    if err == nil {
        t.Error("esperava erro, mas não houve")
    }
}
```

---

## Fuzzing (Go 1.18+)

Fuzzing gera automaticamente entradas aleatórias para encontrar bugs:

```go
func FuzzSomar(f *testing.F) {
    // Casos semente
    f.Add(1.0, 2.0)
    f.Add(-5.0, 10.0)

    f.Fuzz(func(t *testing.T, a, b float64) {
        resultado := Somar(a, b)
        // Propriedade que deve sempre ser verdadeira
        if resultado != a+b {
            t.Errorf("Somar(%v, %v) = %v; esperado %v", a, b, resultado, a+b)
        }
    })
}
```

```bash
# Executar fuzzing por 30 segundos
go test -fuzz=FuzzSomar -fuzztime=30s ./...
```

---

## Boas Práticas de Testes em Go

| ✅ Faça | ❌ Evite |
|--------|---------|
| Use Table-Driven Tests | Duplicar lógica de teste |
| Nomeie subtestes com `t.Run` | Testes sem contexto claro |
| Teste comportamento, não implementação | Testar detalhes internos |
| Use interfaces para dependências | Dependências hardcoded |
| Use `-race` no CI | Ignorar race conditions |
| Escreva testes de exemplo | Documentação só em comentários |
| `t.Cleanup` para liberar recursos | Vazamento de recursos em testes |
| Execute `go test -cover` | Cobertura desconhecida |
