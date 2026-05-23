# Concorrência II: Channels

Se goroutines são as unidades de execução concorrente em Go, **channels** são os tubos por onde elas se comunicam. A filosofia do Go é:

> *"Não comunique compartilhando memória; compartilhe memória comunicando."*

Channels permitem que goroutines troquem dados de forma segura, eliminando a necessidade de locks para muitos cenários.

---

## O que é um Channel?

Um channel é um **duto tipado** por onde valores fluem entre goroutines. Ele é seguro para uso concorrente por natureza.

```
goroutine A  ──[valor]──►  channel  ──[valor]──►  goroutine B
```

---

## Criando e Usando Channels

```go
package main

import "fmt"

func main() {
    // Criando um channel de string
    ch := make(chan string)

    // Enviando para o channel em uma goroutine
    go func() {
        ch <- "Olá do canal!"  // envia valor
    }()

    // Recebendo do channel (bloqueia até receber)
    mensagem := <-ch
    fmt.Println(mensagem) // Olá do canal!
}
```

### Sintaxe Básica

```go
ch := make(chan int)    // channel de int (sem buffer)
ch <- 42               // enviar valor (operador <-)
valor := <-ch          // receber valor
<-ch                   // receber e descartar o valor
```

---

## Channels com Buffer

Por padrão, channels são **sem buffer** (síncronos) — o envio bloqueia até que haja um receptor, e vice-versa.

**Channels com buffer** aceitam um número limitado de valores sem um receptor imediato:

```go
// Channel sem buffer — envio e recebimento devem estar prontos
ch1 := make(chan int)

// Channel com buffer de capacidade 3
ch2 := make(chan int, 3)

ch2 <- 1  // não bloqueia (buffer disponível)
ch2 <- 2  // não bloqueia
ch2 <- 3  // não bloqueia
// ch2 <- 4  // BLOQUEARIA — buffer cheio!

fmt.Println(<-ch2) // 1
fmt.Println(<-ch2) // 2
```

| Tipo | Comportamento no envio | Comportamento no recebimento |
|------|----------------------|------------------------------|
| Sem buffer | Bloqueia até ter receptor | Bloqueia até ter valor |
| Com buffer | Bloqueia se buffer cheio | Bloqueia se buffer vazio |

---

## Fechando Channels

Um channel pode ser **fechado** para sinalizar que não haverá mais envios:

```go
package main

import "fmt"

func gerarNumeros(ch chan int, n int) {
    for i := 0; i < n; i++ {
        ch <- i
    }
    close(ch) // sinaliza que não haverá mais valores
}

func main() {
    ch := make(chan int)
    go gerarNumeros(ch, 5)

    // for range lê até o channel ser fechado
    for num := range ch {
        fmt.Println(num)
    }
    // Saída: 0 1 2 3 4
}
```

### Verificando se o Channel foi Fechado

```go
valor, ok := <-ch
if !ok {
    fmt.Println("Channel fechado!")
}
```

> [!WARNING]
> Apenas o **produtor** (quem envia) deve fechar um channel. Enviar para um channel fechado causa **panic**. Fechar um channel duas vezes também causa **panic**.

---

## Direção de Channels em Parâmetros

Você pode restringir a direção de um channel em parâmetros de funções para tornar a intenção clara:

```go
// Apenas envia para o channel
func produtor(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
}

// Apenas recebe do channel
func consumidor(ch <-chan int) {
    for v := range ch {
        fmt.Println("Recebido:", v)
    }
}

func main() {
    ch := make(chan int)
    go produtor(ch)
    consumidor(ch)
}
```

> [!TIP]
> Restringir a direção de channels em assinaturas de funções é uma **boa prática** — o compilador garante que a função não use o channel de forma não intencional.

---

## `select` — Multiplexando Channels

O `select` permite esperar em múltiplos channels simultaneamente, executando o case do primeiro que estiver pronto:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(200 * time.Millisecond)
        ch1 <- "resultado do canal 1"
    }()

    go func() {
        time.Sleep(100 * time.Millisecond)
        ch2 <- "resultado do canal 2"
    }()

    // Espera pelo primeiro que chegar
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println(msg1)
        case msg2 := <-ch2:
            fmt.Println(msg2)
        }
    }
}
// Saída:
// resultado do canal 2
// resultado do canal 1
```

### `select` com `default` — Não Bloqueante

```go
ch := make(chan int, 1)

select {
case v := <-ch:
    fmt.Println("Recebido:", v)
default:
    fmt.Println("Nenhum valor disponível — continuando...")
}
```

### `select` com Timeout

```go
import "time"

ch := make(chan string)

select {
case resultado := <-ch:
    fmt.Println("Resultado:", resultado)
case <-time.After(2 * time.Second):
    fmt.Println("Timeout! Operação demorou demais.")
}
```

---

## Padrões Comuns com Channels

### 1. Pipeline

Encadeie goroutines onde a saída de uma é a entrada da próxima:

```go
package main

import "fmt"

func gerar(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

func quadrado(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}

func main() {
    // Pipeline: gerar → quadrado
    c := gerar(2, 3, 4, 5)
    out := quadrado(c)

    for v := range out {
        fmt.Println(v) // 4, 9, 16, 25
    }
}
```

### 2. Fan-Out / Fan-In

Distribua trabalho para múltiplos workers e colete os resultados:

```go
package main

import (
    "fmt"
    "sync"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    for j := range jobs {
        resultado := j * j  // processamento
        results <- resultado
        fmt.Printf("Worker %d processou job %d → %d\n", id, j, resultado)
    }
}

func main() {
    jobs := make(chan int, 10)
    results := make(chan int, 10)
    var wg sync.WaitGroup

    // Fan-Out: 3 workers
    for w := 1; w <= 3; w++ {
        wg.Add(1)
        go worker(w, jobs, results, &wg)
    }

    // Envia 9 jobs
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)

    // Fecha results quando todos os workers terminarem
    go func() {
        wg.Wait()
        close(results)
    }()

    // Fan-In: coleta todos os resultados
    total := 0
    for r := range results {
        total += r
    }
    fmt.Println("Soma dos quadrados:", total) // 285
}
```

### 3. Canal de Cancelamento com `context`

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func trabalharComContexto(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d cancelado: %v\n", id, ctx.Err())
            return
        default:
            fmt.Printf("Worker %d trabalhando...\n", id)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // Cancela automaticamente após 1.5 segundos
    ctx, cancel := context.WithTimeout(context.Background(), 1500*time.Millisecond)
    defer cancel()

    for i := 1; i <= 3; i++ {
        go trabalharComContexto(ctx, i)
    }

    time.Sleep(2 * time.Second)
}
```

> [!TIP]
> O pacote `context` é a forma idiomática em Go para propagar cancelamento, deadlines e valores entre goroutines, especialmente em servidores HTTP e sistemas distribuídos.

---

## Goroutine Leak — O que Evitar

Uma goroutine leak acontece quando uma goroutine fica bloqueada para sempre sem poder terminar:

```go
// ❌ GOROUTINE LEAK — o channel nunca é fechado
func vazar() {
    ch := make(chan int)
    go func() {
        v := <-ch  // bloqueia para sempre
        fmt.Println(v)
    }()
    // ch nunca recebe valor, goroutine vaza!
}

// ✅ Use context ou close para sinalizar término
func semVazar(ctx context.Context) {
    ch := make(chan int)
    go func() {
        select {
        case v := <-ch:
            fmt.Println(v)
        case <-ctx.Done():
            return  // goroutine termina limpa
        }
    }()
}
```

---

## Resumo: Channels vs. Mutex

| Situação | Use |
|----------|-----|
| Transferir dados entre goroutines | Channel |
| Sinalizar eventos / conclusão | Channel |
| Pipeline de processamento | Channel |
| Proteger estado compartilhado | Mutex |
| Operações atômicas em inteiros | `sync/atomic` |
| Execução única (singleton) | `sync.Once` |

> [!NOTE]
> A diretriz do Go é preferir channels quando possível, mas não há problema em usar mutex quando faz mais sentido. O importante é que o acesso concorrente ao estado seja **sempre sincronizado**.
