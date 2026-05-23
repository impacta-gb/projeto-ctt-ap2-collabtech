# Tratamento de Erros

Go adota uma filosofia radicalmente diferente da maioria das linguagens para o tratamento de erros: **erros são valores**, não exceções. Não existe `try/catch`. Em vez disso, funções retornam o erro como último valor de retorno, e o chamador é responsável por verificá-lo explicitamente.

Essa abordagem torna o fluxo de erros **visível, explícito e rastreável** no código.

---

## O Tipo `error`

`error` é uma interface nativa do Go:

```go
type error interface {
    Error() string
}
```

Qualquer tipo que implementar o método `Error() string` satisfaz a interface `error`.

---

## Retornando e Verificando Erros

O padrão idiomático em Go é retornar `(resultado, error)`:

```go
package main

import (
    "errors"
    "fmt"
)

func dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("divisão por zero não é permitida")
    }
    return a / b, nil  // nil significa "sem erro"
}

func main() {
    resultado, err := dividir(10, 2)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    fmt.Printf("10 / 2 = %.1f\n", resultado) // 10 / 2 = 5.0

    _, err = dividir(5, 0)
    if err != nil {
        fmt.Println("Erro:", err) // Erro: divisão por zero não é permitida
    }
}
```

> [!NOTE]
> **`nil`** representa a ausência de erro. Sempre verifique `if err != nil` antes de usar o resultado de uma função que pode falhar.

---

## Criando Erros

### `errors.New`

```go
import "errors"

err := errors.New("algo deu errado")
```

### `fmt.Errorf` — Erros com contexto

```go
import "fmt"

nome := "arquivo.txt"
err := fmt.Errorf("não foi possível abrir o arquivo '%s': permissão negada", nome)
fmt.Println(err)
// não foi possível abrir o arquivo 'arquivo.txt': permissão negada
```

### `fmt.Errorf` com `%w` — Wrapping de Erros

O verbo `%w` permite **envolver (wrap)** um erro em outro, preservando a cadeia de contexto:

```go
erroOriginal := errors.New("conexão recusada")
erroContexto := fmt.Errorf("falha ao conectar ao banco de dados: %w", erroOriginal)

fmt.Println(erroContexto)
// falha ao conectar ao banco de dados: conexão recusada
```

---

## Tipos de Erro Customizados

Para erros mais ricos (com campos adicionais), implemente a interface `error`:

```go
type ErroValidacao struct {
    Campo   string
    Mensagem string
}

func (e *ErroValidacao) Error() string {
    return fmt.Sprintf("validação falhou no campo '%s': %s", e.Campo, e.Mensagem)
}

func validarIdade(idade int) error {
    if idade < 0 {
        return &ErroValidacao{Campo: "idade", Mensagem: "não pode ser negativa"}
    }
    if idade > 150 {
        return &ErroValidacao{Campo: "idade", Mensagem: "valor improvável"}
    }
    return nil
}

func main() {
    if err := validarIdade(-5); err != nil {
        fmt.Println(err)
        // validação falhou no campo 'idade': não pode ser negativa
    }
}
```

---

## `errors.Is` — Comparando Erros

Use `errors.Is` para verificar se um erro é (ou contém) um erro específico:

```go
import (
    "errors"
    "fmt"
)

var ErrNaoEncontrado = errors.New("registro não encontrado")

func buscarUsuario(id int) error {
    if id != 42 {
        return fmt.Errorf("buscarUsuario(%d): %w", id, ErrNaoEncontrado)
    }
    return nil
}

func main() {
    err := buscarUsuario(99)

    if errors.Is(err, ErrNaoEncontrado) {
        fmt.Println("Usuário não existe no sistema")
    }

    fmt.Println(err)
    // buscarUsuario(99): registro não encontrado
}
```

> [!TIP]
> Defina **erros sentinela** (variáveis globais de erro) com `var ErrAlgo = errors.New(...)` para que os chamadores possam comparar com `errors.Is`. Isso é preferível a comparar strings de erro.

---

## `errors.As` — Extraindo Tipo de Erro

Use `errors.As` para verificar e extrair um erro de um tipo específico na cadeia:

```go
type ErroHTTP struct {
    Codigo  int
    Detalhe string
}

func (e *ErroHTTP) Error() string {
    return fmt.Sprintf("HTTP %d: %s", e.Codigo, e.Detalhe)
}

func fazerRequisicao(url string) error {
    return fmt.Errorf("falha na requisição para %s: %w", url,
        &ErroHTTP{Codigo: 404, Detalhe: "página não encontrada"})
}

func main() {
    err := fazerRequisicao("https://exemplo.com/pagina")

    var httpErr *ErroHTTP
    if errors.As(err, &httpErr) {
        fmt.Printf("Código HTTP: %d\n", httpErr.Codigo)   // 404
        fmt.Printf("Detalhe: %s\n", httpErr.Detalhe)      // página não encontrada
    }
}
```

---

## Múltiplos Erros com `errors.Join` (Go 1.20+)

```go
import "errors"

err1 := errors.New("campo 'nome' obrigatório")
err2 := errors.New("campo 'email' inválido")
err3 := errors.New("campo 'idade' deve ser positivo")

errosCombinados := errors.Join(err1, err2, err3)
fmt.Println(errosCombinados)
```

**Saída:**
```
campo 'nome' obrigatório
campo 'email' inválido
campo 'idade' deve ser positivo
```

---

## `panic` e `recover`

Em situações **verdadeiramente excepcionais** (bugs, estados inválidos irreparáveis), Go tem `panic` e `recover`.

### `panic`

Interrompe a execução e desempilha o stack:

```go
func acessarIndice(s []int, i int) int {
    if i >= len(s) {
        panic(fmt.Sprintf("índice %d fora do range (len=%d)", i, len(s)))
    }
    return s[i]
}
```

### `recover`

Captura um `panic` em andamento — **deve ser usado dentro de um `defer`**:

```go
func executarComSeguranca(f func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic capturado: %v", r)
        }
    }()
    f()
    return nil
}

func main() {
    err := executarComSeguranca(func() {
        panic("algo muito errado aconteceu!")
    })

    if err != nil {
        fmt.Println("Recuperado:", err)
        // Recuperado: panic capturado: algo muito errado aconteceu!
    }
}
```

> [!WARNING]
> **Não use `panic` para tratamento de erros normais.** Use-o apenas para erros que representam bugs no código (violações de invariantes, estados impossíveis). Para erros esperados (arquivo não encontrado, input inválido), use o retorno de `error`.

---

## `defer` — Garantindo Limpeza

`defer` adia a execução de uma função para quando a função atual retornar — muito útil para liberar recursos:

```go
import "os"

func processarArquivo(nome string) error {
    arquivo, err := os.Open(nome)
    if err != nil {
        return fmt.Errorf("ao abrir arquivo: %w", err)
    }
    defer arquivo.Close() // garantido mesmo se houver erro abaixo

    // ... processamento ...
    return nil
}
```

```go
// Múltiplos defers — executam em ordem LIFO (último a entrar, primeiro a sair)
func exemplo() {
    defer fmt.Println("terceiro")
    defer fmt.Println("segundo")
    defer fmt.Println("primeiro")
    fmt.Println("executando...")
}
// Saída:
// executando...
// primeiro
// segundo
// terceiro
```

---

## Padrões Práticos de Tratamento de Erros

### Padrão "Fail Fast"

```go
func processarPedido(id int) error {
    pedido, err := buscarPedido(id)
    if err != nil {
        return fmt.Errorf("processarPedido: %w", err)
    }

    if err := validarPedido(pedido); err != nil {
        return fmt.Errorf("processarPedido: %w", err)
    }

    if err := cobrarCliente(pedido); err != nil {
        return fmt.Errorf("processarPedido: %w", err)
    }

    return nil
}
```

### Agrupando Verificações com Helper

```go
type ErrHandler struct {
    err error
}

func (e *ErrHandler) Do(f func() error) {
    if e.err == nil {
        e.err = f()
    }
}

func main() {
    eh := &ErrHandler{}

    eh.Do(abrirConexao)
    eh.Do(lerDados)
    eh.Do(processar)

    if eh.err != nil {
        fmt.Println("Erro:", eh.err)
    }
}
```

---

## Resumo: Filosofia de Erros em Go

| Situação | Abordagem |
|----------|-----------|
| Erro esperado e recuperável | Retorne `error` como último valor |
| Erro de programação (bug) | `panic` |
| Recuperar de panic em servidor | `recover` em `defer` |
| Liberar recursos | `defer` |
| Comparar erros | `errors.Is` |
| Extrair tipo de erro | `errors.As` |
| Adicionar contexto ao erro | `fmt.Errorf("contexto: %w", err)` |
