# Concorrência I: Goroutines

A concorrência é um dos pontos mais poderosos e distintivos de Go. A linguagem foi projetada desde o início para facilitar a escrita de programas concorrentes, com primitivas nativas leves e eficientes. O pilar dessa concorrência são as **goroutines**.

---

## O que é uma Goroutine?

Uma goroutine é uma **thread leve gerenciada pelo runtime do Go**. Ao contrário das threads do sistema operacional (que consomem cerca de 1–2 MB de memória cada), uma goroutine começa com apenas **~2 KB de stack**, que cresce dinamicamente conforme necessário.

O Go scheduler (agendador) multiplexa N goroutines em M threads do SO — esse modelo é chamado de **M:N scheduling**.

| Comparação | Thread do SO | Goroutine |
|-----------|-------------|-----------|
| Memória inicial | ~1–2 MB | ~2 KB |
| Criação | Custosa (syscall) | Muito barata |
| Troca de contexto | Cara | Muito barata |
| Gerenciamento | SO | Runtime do Go |
| Quantidade típica | Centenas | Milhares a milhões |

---

## Criando Goroutines

Para iniciar uma goroutine, basta prefixar uma chamada de função com a palavra-chave `go`:

```go
package main

import (
    "fmt"
    "time"
)

func saudar(nome string) {
    fmt.Printf("Olá, %s!\n", nome)
}

func main() {
    go saudar("Alice")  // inicia goroutine
    go saudar("Bob")    // inicia goroutine
    go saudar("Carol")  // inicia goroutine

    // Sem isso, o programa termina antes das goroutines executarem!
    time.Sleep(100 * time.Millisecond)
    fmt.Println("Programa encerrado")
}
```

> [!WARNING]
> O uso de `time.Sleep` para sincronização é **apenas didático**. Na prática, use `sync.WaitGroup` ou channels para coordenar goroutines. Nunca confie em timers para sincronização em produção.

---

## `sync.WaitGroup` — Aguardando Goroutines

O `WaitGroup` é a forma correta de esperar um grupo de goroutines terminar:

```go
package main

import (
    "fmt"
    "sync"
)

func trabalhador(id int, wg *sync.WaitGroup) {
    defer wg.Done()  // sinaliza conclusão ao terminar
    fmt.Printf("Trabalhador %d iniciando\n", id)
    // Simulando trabalho...
    fmt.Printf("Trabalhador %d concluído\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)               // incrementa o contador
        go trabalhador(i, &wg)  // inicia goroutine
    }

    wg.Wait()  // bloqueia até o contador chegar a 0
    fmt.Println("Todos os trabalhadores concluídos!")
}
```

**Saída (ordem pode variar):**
```
Trabalhador 3 iniciando
Trabalhador 1 iniciando
Trabalhador 5 iniciando
Trabalhador 2 iniciando
Trabalhador 4 iniciando
Trabalhador 4 concluído
...
Todos os trabalhadores concluídos!
```

---

## Race Conditions — O Perigo da Concorrência

Quando múltiplas goroutines acessam e modificam a mesma variável sem sincronização, temos uma **condição de corrida (race condition)**:

```go
// ❌ CÓDIGO COM RACE CONDITION — NÃO FAÇA ISSO
package main

import (
    "fmt"
    "sync"
)

func main() {
    var contador int
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            contador++  // RACE CONDITION! Leitura e escrita não atômica
        }()
    }

    wg.Wait()
    fmt.Println("Contador:", contador) // Resultado imprevisível!
}
```

> [!WARNING]
> Race conditions são bugs extremamente difíceis de reproduzir e depurar. O resultado pode variar a cada execução. Use sempre mecanismos de sincronização ao compartilhar estado entre goroutines.

### Detectando Race Conditions

O Go tem uma ferramenta embutida para detecção:

```bash
go run -race main.go
go test -race ./...
```

---

## `sync.Mutex` — Exclusão Mútua

Um **Mutex** garante que apenas uma goroutine acesse um recurso por vez:

```go
package main

import (
    "fmt"
    "sync"
)

type ContadorSeguro struct {
    mu    sync.Mutex
    valor int
}

func (c *ContadorSeguro) Incrementar() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.valor++
}

func (c *ContadorSeguro) Valor() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.valor
}

func main() {
    contador := &ContadorSeguro{}
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            contador.Incrementar()
        }()
    }

    wg.Wait()
    fmt.Println("Contador:", contador.Valor()) // Sempre 1000
}
```

### `sync.RWMutex` — Leitura/Escrita

Quando há muito mais leituras do que escritas, use `RWMutex`:

```go
type Cache struct {
    mu   sync.RWMutex
    dados map[string]string
}

func (c *Cache) Get(chave string) (string, bool) {
    c.mu.RLock()         // múltiplos leitores simultâneos OK
    defer c.mu.RUnlock()
    v, ok := c.dados[chave]
    return v, ok
}

func (c *Cache) Set(chave, valor string) {
    c.mu.Lock()          // exclusão total para escrita
    defer c.mu.Unlock()
    c.dados[chave] = valor
}
```

---

## `sync/atomic` — Operações Atômicas

Para operações simples em inteiros, `atomic` é mais eficiente que um mutex:

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var contador int64
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&contador, 1) // operação atômica segura
        }()
    }

    wg.Wait()
    fmt.Println("Contador:", atomic.LoadInt64(&contador)) // 1000
}
```

---

## `sync.Once` — Executar Apenas Uma Vez

Útil para inicialização preguiçosa (lazy initialization) thread-safe:

```go
package main

import (
    "fmt"
    "sync"
)

type Singleton struct {
    dados string
}

var (
    instancia *Singleton
    once      sync.Once
)

func GetInstancia() *Singleton {
    once.Do(func() {
        fmt.Println("Inicializando singleton...")
        instancia = &Singleton{dados: "valor inicial"}
    })
    return instancia
}

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            s := GetInstancia()
            fmt.Println("Usando:", s.dados)
        }()
    }

    wg.Wait()
}
// "Inicializando singleton..." aparece apenas UMA vez
```

---

## Exemplo Prático — Download Paralelo

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Resultado struct {
    URL   string
    Bytes int
    Erro  error
}

func baixar(url string) Resultado {
    // Simulando download
    time.Sleep(100 * time.Millisecond)
    return Resultado{URL: url, Bytes: len(url) * 100}
}

func baixarEmParalelo(urls []string) []Resultado {
    resultados := make([]Resultado, len(urls))
    var wg sync.WaitGroup

    for i, url := range urls {
        wg.Add(1)
        go func(idx int, u string) {
            defer wg.Done()
            resultados[idx] = baixar(u)
        }(i, url)
    }

    wg.Wait()
    return resultados
}

func main() {
    urls := []string{
        "https://go.dev",
        "https://github.com",
        "https://golang.org",
        "https://pkg.go.dev",
    }

    inicio := time.Now()
    resultados := baixarEmParalelo(urls)
    elapsed := time.Since(inicio)

    for _, r := range resultados {
        fmt.Printf("%-25s → %d bytes\n", r.URL, r.Bytes)
    }
    fmt.Printf("\nTempo total: %v (paralelo vs ~400ms sequencial)\n", elapsed)
}
```

---

## Goroutines e Closures

Cuidado com closures em loops — um erro clássico:

```go
// ❌ BUG COMUM — todas as goroutines capturam a mesma variável i
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i)  // i pode ser 5 em todas!
    }()
}

// ✅ CORRETO — passa i como argumento (cria uma cópia)
for i := 0; i < 5; i++ {
    go func(n int) {
        fmt.Println(n)  // cada goroutine tem seu próprio n
    }(i)
}

// ✅ TAMBÉM CORRETO — cria variável local no loop (Go 1.22+)
for i := range 5 {
    go func() {
        fmt.Println(i)  // Go 1.22+ cria nova variável por iteração
    }()
}
```

> [!WARNING]
> O bug da closure em loop é um dos mais frequentes em código Go concorrente. Execute `go run -race` regularmente para detectar esse tipo de problema antes de ir para produção.

---

## Goroutines — Boas Práticas

| ✅ Faça | ❌ Evite |
|--------|---------|
| Use `WaitGroup` para sincronizar | Usar `time.Sleep` para sincronizar |
| Proteja estado compartilhado com Mutex | Acessar variáveis compartilhadas sem lock |
| Use `-race` no desenvolvimento | Ignorar race conditions |
| Passe dados por valor para goroutines | Compartilhar ponteiros sem cuidado |
| Garanta que goroutines terminam | Goroutines "vazando" (goroutine leak) |
| Comunique por channels (próxima seção) | Compartilhar memória diretamente |
