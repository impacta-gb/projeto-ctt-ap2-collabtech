# Structs e Métodos

Go não é uma linguagem orientada a objetos no sentido tradicional — não há classes ou herança. Em vez disso, Go usa **structs** para agrupar dados e **métodos** para associar comportamentos a esses dados. Essa abordagem favorece a **composição** em vez da herança.

---

## Structs

Uma `struct` é um tipo de dado composto que agrupa campos relacionados sob um único nome.

### Declaração

```go
type Pessoa struct {
    Nome  string
    Idade int
    Email string
}
```

### Criando Instâncias

```go
// Forma 1: por ordem de campos (frágil — evite)
p1 := Pessoa{"Alice", 30, "alice@email.com"}

// Forma 2: por nome de campos (recomendada)
p2 := Pessoa{
    Nome:  "Bob",
    Idade: 25,
    Email: "bob@email.com",
}

// Forma 3: var (zero values)
var p3 Pessoa
p3.Nome = "Carol"
p3.Idade = 28
```

### Acessando e Modificando Campos

```go
fmt.Println(p2.Nome)  // Bob
fmt.Println(p2.Idade) // 25

p2.Email = "novo@email.com"
fmt.Println(p2.Email) // novo@email.com
```

### Zero Values em Structs

```go
var p Pessoa
fmt.Println(p.Nome)  // "" (string vazia)
fmt.Println(p.Idade) // 0
fmt.Println(p.Email) // ""
```

---

## Ponteiros para Structs

Ao passar uma struct para uma função, Go a **copia por valor** por padrão. Para modificar o original, use ponteiros.

```go
type Contador struct {
    Valor int
}

// Sem ponteiro — modifica apenas a cópia
func incrementarCopia(c Contador) {
    c.Valor++
}

// Com ponteiro — modifica o original
func incrementar(c *Contador) {
    c.Valor++
}

func main() {
    c := Contador{Valor: 0}

    incrementarCopia(c)
    fmt.Println(c.Valor) // 0 (não mudou!)

    incrementar(&c)
    fmt.Println(c.Valor) // 1 (mudou!)

    // Go permite acessar campos de ponteiros sem desreferenciar
    ptr := &c
    ptr.Valor = 100  // equivale a (*ptr).Valor = 100
    fmt.Println(c.Valor) // 100
}
```

---

## Structs Aninhadas

```go
type Endereco struct {
    Rua    string
    Cidade string
    Estado string
    CEP    string
}

type Pessoa struct {
    Nome     string
    Idade    int
    Endereco Endereco  // struct aninhada
}

func main() {
    p := Pessoa{
        Nome:  "Ana",
        Idade: 32,
        Endereco: Endereco{
            Rua:    "Rua das Flores, 123",
            Cidade: "São Paulo",
            Estado: "SP",
            CEP:    "01310-100",
        },
    }

    fmt.Println(p.Nome)             // Ana
    fmt.Println(p.Endereco.Cidade)  // São Paulo
}
```

### Embedding (Composição)

Go permite **embutir** uma struct dentro de outra sem dar um nome ao campo. Os campos da struct embutida ficam acessíveis diretamente.

```go
type Animal struct {
    Nome string
}

func (a Animal) Respirar() {
    fmt.Println(a.Nome, "está respirando")
}

type Cachorro struct {
    Animal       // embedding — sem nome de campo
    Raca string
}

func main() {
    d := Cachorro{
        Animal: Animal{Nome: "Rex"},
        Raca:   "Labrador",
    }

    fmt.Println(d.Nome)  // Rex (promovido do Animal)
    d.Respirar()         // Rex está respirando
    fmt.Println(d.Raca)  // Labrador
}
```

> [!TIP]
> O embedding é a forma de **composição** do Go. Em vez de herdar de uma classe base, você embute tipos e reutiliza seus comportamentos. É mais flexível e evita os problemas de hierarquias profundas de herança.

---

## Métodos

Um método é uma **função com um receptor** — uma struct à qual o método pertence.

### Sintaxe

```go
func (receptor TipoReceptor) NomeDoMetodo(parâmetros) tipoRetorno {
    // corpo
}
```

### Receptor por Valor vs. por Ponteiro

```go
type Retangulo struct {
    Largura float64
    Altura  float64
}

// Receptor por valor — não modifica o original
func (r Retangulo) Area() float64 {
    return r.Largura * r.Altura
}

// Receptor por valor — não modifica o original
func (r Retangulo) Perimetro() float64 {
    return 2 * (r.Largura + r.Altura)
}

// Receptor por ponteiro — pode modificar o original
func (r *Retangulo) Dobrar() {
    r.Largura *= 2
    r.Altura *= 2
}

func main() {
    ret := Retangulo{Largura: 5, Altura: 3}

    fmt.Printf("Área: %.1f\n", ret.Area())        // 15.0
    fmt.Printf("Perímetro: %.1f\n", ret.Perimetro()) // 16.0

    ret.Dobrar()
    fmt.Printf("Nova Área: %.1f\n", ret.Area())   // 60.0
}
```

> [!NOTE]
> **Regra geral:** Use receptor por ponteiro (`*T`) quando:
> - O método precisa **modificar** o receptor
> - A struct é **grande** (evitar cópia cara)
> - **Consistência** — se um método usa ponteiro, todos devem usar

---

## Interfaces

Interfaces definem um **contrato de comportamento** — qualquer tipo que implementar todos os métodos de uma interface a satisfaz automaticamente (implementação implícita, sem `implements`).

```go
type Forma interface {
    Area() float64
    Perimetro() float64
}

type Retangulo struct {
    Largura, Altura float64
}

func (r Retangulo) Area() float64 {
    return r.Largura * r.Altura
}

func (r Retangulo) Perimetro() float64 {
    return 2 * (r.Largura + r.Altura)
}

type Circulo struct {
    Raio float64
}

func (c Circulo) Area() float64 {
    return 3.14159 * c.Raio * c.Raio
}

func (c Circulo) Perimetro() float64 {
    return 2 * 3.14159 * c.Raio
}

// Função polimórfica que aceita qualquer Forma
func imprimirForma(f Forma) {
    fmt.Printf("Área: %.2f | Perímetro: %.2f\n", f.Area(), f.Perimetro())
}

func main() {
    r := Retangulo{Largura: 4, Altura: 6}
    c := Circulo{Raio: 5}

    imprimirForma(r) // Área: 24.00 | Perímetro: 20.00
    imprimirForma(c) // Área: 78.54 | Perímetro: 31.42

    // Slice de interfaces
    formas := []Forma{r, c, Retangulo{2, 3}}
    for _, f := range formas {
        imprimirForma(f)
    }
}
```

### Interface Vazia (`interface{}` ou `any`)

```go
// Aceita qualquer tipo
func imprimirQualquer(v interface{}) {
    fmt.Printf("Valor: %v | Tipo: %T\n", v, v)
}

func main() {
    imprimirQualquer(42)
    imprimirQualquer("hello")
    imprimirQualquer(true)
    imprimirQualquer([]int{1, 2, 3})
}
```

> [!NOTE]
> A partir do Go 1.18, `any` é um alias para `interface{}` e é a forma preferida.

---

## Construtores (por convenção)

Go não tem construtores nativos. O padrão é criar funções `New...`:

```go
type ContaBancaria struct {
    titular string
    saldo   float64
}

// "Construtor" — retorna ponteiro para garantir consistência
func NovaConta(titular string, saldoInicial float64) (*ContaBancaria, error) {
    if saldoInicial < 0 {
        return nil, fmt.Errorf("saldo inicial não pode ser negativo")
    }
    return &ContaBancaria{
        titular: titular,
        saldo:   saldoInicial,
    }, nil
}

func (c *ContaBancaria) Depositar(valor float64) {
    c.saldo += valor
}

func (c *ContaBancaria) Sacar(valor float64) error {
    if valor > c.saldo {
        return fmt.Errorf("saldo insuficiente: %.2f disponível", c.saldo)
    }
    c.saldo -= valor
    return nil
}

func (c ContaBancaria) Saldo() float64 {
    return c.saldo
}

func (c ContaBancaria) String() string {
    return fmt.Sprintf("Conta[%s] — Saldo: R$ %.2f", c.titular, c.saldo)
}

func main() {
    conta, err := NovaConta("Alice", 1000.0)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }

    conta.Depositar(500.0)
    fmt.Println(conta)          // Conta[Alice] — Saldo: R$ 1500.00

    if err := conta.Sacar(200.0); err != nil {
        fmt.Println("Erro:", err)
    }
    fmt.Printf("Saldo atual: R$ %.2f\n", conta.Saldo()) // R$ 1300.00

    if err := conta.Sacar(9999.0); err != nil {
        fmt.Println("Erro:", err) // Erro: saldo insuficiente
    }
}
```

> [!TIP]
> Implementar o método `String() string` faz com que a struct use esse texto automaticamente no `fmt.Println`. É equivalente ao `toString()` de outras linguagens.

---

## Comparação com OOP Tradicional

| Conceito OOP | Equivalente em Go |
|-------------|-------------------|
| Classe | `struct` + métodos |
| Herança | Embedding (composição) |
| Interface | `interface` (implícita) |
| Construtor | Função `New...` por convenção |
| `this` / `self` | Receptor do método |
| Encapsulamento | Maiúscula = público, minúscula = privado |
| Polimorfismo | Interfaces |
