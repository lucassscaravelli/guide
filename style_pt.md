<!--

Editing this document:

- Discuss all changes in GitHub issues first.
- Update the table of contents as new sections are added or removed.
- Use tables for side-by-side code samples. See below.

Code Samples:

Use 2 spaces to indent. Horizontal real estate is important in side-by-side
samples.

For side-by-side code samples, use the following snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
BAD CODE GOES HERE
```

</td><td>

```go
GOOD CODE GOES HERE
```

</td></tr>
</tbody></table>
~~~

(You need the empty lines between the <td> and code samples for it to be
treated as Markdown.)

If you need to add labels or descriptions below the code samples, add another
row before the </tbody></table> line.

~~~
<tr>
<td>DESCRIBE BAD CODE</td>
<td>DESCRIBE GOOD CODE</td>
</tr>
~~~

-->

# Guia de estilo Uber Go (traduzido de [uber-go/guide](https://github.com/uber-go/guide))

## Sumário

- [Introdução](#introdução)
- [Diretrizes](#diretrizes)
  - [Ponteiros para interfaces](#ponteiros-para-interfaces)
  - [Verificar o "Contrato" da Interface](#verificar-o-"contrato"-da-interface)
  - [Receptores e Interfaces](#receptores-e-interfaces)
  - [Mutexes com valor zero são validos](#mutexes-com-valor-zero-são-validos)
  - [Copiar slices e mapas com limites](#copiar-slices-e-mapas-com-limites)
  - [Defer para "limpar"](#defer-para-limpar)
  - [Tamanho no canal é um ou nenhum](#tamanho-no-canal-é-um-ou-nenhum)
  - [Iniciar enums em um](#iniciar-enums-em-um)
  - [Utilizar `"time"` para lidar com tempo](#utilizar-"time"-para-lidar-com-tempo)
  - [Erros](#erros)
    - [Tipos de erro](#tipos-de-erro)
    - [Utilizando Error Wrapping](#utilizando-error-wrapping)
    - [Nomeando erros](#nomeando-erros)
  - [Manipular falhas de asserção de tipo](#manipular-falhas-de-asserção-de-tipo)
  - [Não utilize panic](#não-utilize-panic)
  - [Utilize go.uber.org/atomic](#utilize-gouberorgatomic)
- [Performance](#performance)
  - [Utilize strconv ao invés de fmt](#utilize-strconv-ao-invés-de-fmt)
  - [Evite a conversão de string para byte](#evite-a-conversao-de-string-para-byte)
  - [Utilize tamanho ao criar mapas](#utilize-tamanho-ao-criar-mapas)
- [Estilo](#estilo)
  - [Seja consistente](#seja-consistente)
  - [Agrupar declarações similares](#agrupar-declarações-similares)
  - [Ordenação e agrupamento de imports](#ordenação-e-agrupamento-de-imports)
  - [Nome de pacotes](#nome-de-pacotes)
  - [Nome de funções](#nome-de-funções)
  - [Alias de Import](#alias-de-import)
  - [Agrupamento e ordenação de funções](#agrupamento-e-ordenação-de-funções)
  - [Reduzir o aninhamento](#reduzir-o-aninhamento)
  - [Else desnecessário](#else-desnecessário)
  - [Declaração de variavéis top-level](#declaração-de-variavéis-top-level)
  - [Utilize o prefixo _ para globais não exportados](#utilize-o-prefixo-_-para-globais-não-exportados)
  - [Tipos embutidos em structs](#tipos_embutidos_em_structs)
  - [Utilize nome dos campos para inicializar uma struct](#utilize-nome-dos-campos-para-inicializar-uma-struct)
  - [Declaração de variáveis locais](#declaração-de-variáveis-locais)
  - [Nil é um slice válido](#nil-é-um-slice-válido)
  - [Reduzir o escopo das váriaveis](#reduzir-o-escopo-das-váriaveis)
  - [Evitar passar parâmetros sem identificação](#evitar-passar-parâmetros-sem-identificação)
  - [Use strings literais para evitar escape](#use-strings-literais-para-evitar-escape)
  - [Inicializando referências de structs](#inicializando-referências-de-structs)
  - [Inicializando mapas](#inicializando-mapas)
  - [Formate strings fora da função Printf](#formate-strings-fora-da-função-printf)
  - [Nomeando funções da família Printf](#nomeando-funções-da-família-printf)
- [Patterns](#patterns)
  - [Test Tables](#test-tables)
  - [Functional Options](#functional-options)

## Introdução

Estilos são as convenções que governam nosso código. O termo estilo é um pouco inadequado, uma vez que essas convenções abrangem muito mais do que apenas a formatação de arquivos de origem - o pacote gofmt lida com isso para nós. 

O objetivo deste guia é gerenciar essa complexidade, descrevendo em detalhes os prós e contras de escrever o código Go no Uber. Essas regras existem para manter o código base gerenciável e, ao mesmo tempo, permitir que os engenheiros usem os recursos da linguagem Go de forma produtiva.

Este guia foi criado originalmente por [Prashant Varanasi] e [Simon Newton] como uma maneira de atualizar alguns colegas sobre o uso do Go. Ao longo dos anos, foi adicionado com base no feedback de outras pessoas.

[Prashant Varanasi]: https://github.com/prashantv
[Simon Newton]: https://github.com/nomis52

Este guia documenta as convenções idiomáticas no código Go que seguimos no Uber. Muitas delas são diretrizes gerais para o Go, enquanto outras se estendem a recursos externos:

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [The Go common mistakes guide](https://github.com/golang/go/wiki/CodeReviewComments)
3. [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

Todo o código deve estar livre de erros ao executar o golint e go vet. Recomendamos configurar seu editor para:

- Executar `goimports` ao salvar
- Executar `golint` and `go vet` para verificar se há erros

Você pode encontrar informações no suporte do editor para as ferramentas Go aqui: https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins

## Diretrizes

### Ponteiros para interfaces

Você quase nunca precisa de um ponteiro para uma interface. Você deve passar interfaces como valores pois quando a mesma for implementada, ainda pode ser um ponteiro.

Uma interface é divida em dois campos:

1. Um ponteiro para um tipo específico.

2. _Data pointer_. Se o dado armazenado é um ponteiro, então o valor salvo na interface é o próprio ponteiro. Se o dado armazenado é um valor, então um ponteiro para este valor é armazenado na interface. 

Se você deseja que os métodos da interface modifiquem os dados que a mesma possuirá, use um ponteiro.

### Verificar o "Contrato" da Interface 

Verificar o "contrato" da interface no momento do _build_ é apropriado. Isso incluí:

- Tipos exportados que são requiridos para implementar interfaces específicas como parte de seu contrato de API.
- Tipos exportados ou não exportados que são parte de conjunto de outros tipos implementando a mesma interface.
- Outros casos em que violação da interface irá travar usuários.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

O código `var _ http.Handler = (*Handler)(nil)` irá falhar se o `*Handler` parar de corresponder a interface `http.Handler`.

O valor atribuido deve ser o "valor zero" do tipo correspondente. Sendo `nil` para tipos de ponteiros (como `*Handler`), _slices_, _maps_ e uma _struct_ vazia para tipos de _struct_. 

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

### Receptores e Interfaces

Métodos com valores receptores podem ser chamados tanto como ponteiros ou como valores.

Por exemplo:

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// Você pode chamar o metódo Read apenas utilizando o valor
sVals[1].Read()

// Isso não irá compilar
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// Você pode chamar os metódos Read e Write utilizando o ponteiro
sPtrs[1].Read()
sPtrs[1].Write("test")
```

Da mesma forma que uma interface aceita um ponteiro ou valor como receptor em seus métodos.

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// Isso não compila, pois s2Val é um valor da struct S2 e portanto não pode utilizar-se do metódo f(), que tem como receptor um ponteiro
//   i = s2Val
```
_Effective Go_ tem uma boa abordagem sobre isso em _Pointers_ and Values [Pointers vs. Values].

[Pointers vs. Values]: https://golang.org/doc/effective_go.html#pointers_vs_values

### Mutexes com valor zero são validos

Quando utiliza-se sync.Mutex e sync.RWMutex com valores zero, ou seja, sem inicialização, eles são válidos, então você nunca precisa de um ponteiro para um mutex.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

Se você usa uma estrutura a partir de um ponteiro, o mutex pode ser um "acesso sem ponteiro" 

Structs privadas que utilizam um mutex para proteger algum campo, devem utilizar o mutex de forma "embutida".

<table>
<tbody>
<tr><td>

```go
type smap struct {
  sync.Mutex // only for unexported types

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

</tr>
<tr>
<td>Uso embutido para structs que precisam de um mutex embutido.</td>
<td>Para tipos exportados, utilize um campo privado.</td>
</tr>

</tbody></table>

### Copiar slices e mapas com limites

Slices e mapas contêm ponteiros para os dados que armazenam, portanto, tenha cuidado quando eles precisarem ser copiados.

#### Recebendo slices e mapas

Lembre-se de que os usuários podem modificar um mapa ou slice que você recebeu como argumento de uma função, se você armazenar uma referência a ele.

<table>
<thead><tr><th>Ruim</th> <th>Bom</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Você pode modificiar o valor depois de ter armazenado no Driver
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// Agora podemos moficiar trips sem alterar a informação no Driver
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Retornando slices e mapas

Da mesma forma do item anterior, tenha cuidado com as modificações do usuário nos mapas ou slices retornados por alguma função.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Retorna o snapshot protegido por um mutex
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// porém agora o snapshot não esta mais protegido por mutex e pode ser modificado
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Agora snapshot é uma cópia
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Defer para "limpar"

Use "defer" para limpar recursos como arquivos e locks.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// Muito fácil de errar e esquecer um "Unlock" e deixar o mutex travado
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// mais legível
```

</td></tr>
</tbody></table>

O "Defer" tem um custo computacional extremamente pequeno e deve ser evitado apenas se você puder provar que o tempo de execução da sua função é da ordem de nanossegundos. O ganho de legibilidade do uso de "Defer" vale o custo minúsculo de usá-los. Isso vale especialmente para métodos maiores que têm mais do que simples acessos à memória, onde os outros cálculos são mais significativos que a instrução chamada no "Defer"

### Tamanho no canal é um ou nenhum

Os canais geralmente devem ter o tamanho um ou não serem armazenados em buffer. Por padrão, os canais são sem buffer e têm tamanho zero. Qualquer outro tamanho deve estar sujeito a um alto nível de análise. Considere que, ao utilizar canais com um tamanho determinado, você precisa decidir o que vai impedir de o canal se encher e bloquear "escritores", e o que acontece quando isso ocorre.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// Deveria ser suficiente para qualquer um!
c := make(chan int, 64)
```

</td><td>

```go
// Tamanho 1
c := make(chan int, 1) // or
// Canal sem bufffer, tamanho zero
c := make(chan int)
```

</td></tr>
</tbody></table>

### Iniciar enums em um

A maneira padrão de introduzir enumerações no Go é declarar um tipo personalizado
e um grupo `const` com `iota`. Como as variáveis têm um valor padrão 0, você
geralmente deve iniciar suas enumerações com um valor diferente de zero.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

Há casos em que o uso do valor zero faz sentido, por exemplo, quando o
caso com valor zero é o comportamento padrão desejável.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```
### Utilizar `"time"` para lidar com tempo

Tempo é complicado. Suposições incorretas frequentemente feitas sobre o tempo incluem o seguinte:

1. Um dia tem 24 horas
2. Uma hora tem 60 minutos
3. Uma semana tem 7 dias
4. Um ano tem 365 dias
5. [E muito mais](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

Por exemplo, *1* significa que adicionando 24 horas em um instante de tempo nem sempre resultará em um novo dia de calendário.

Portanto, utilize sempre o pacote [`"time"`] ao lidar com o tempo, porque ele
ajuda a com estes pressupostos incorrectos de uma forma mais segura e precisa.

  [`"time"`]: https://golang.org/pkg/time/

#### Utilizar `time.Time` para instantes de tempo

Utilizar [`time.Time`] quando lidar com instantes de tempo e os métodos no pacote `time.Time` ao comparar, adicionar e subtrair tempo.

  [`time.Time`]: https://golang.org/pkg/time/#Time

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

</td><td>

```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

</td></tr>
</tbody></table>

#### Utilizar `time.Duration` para períodos de tempo

Utilizar [`time.Duration`] quando lidar com períodos de tempo.

[`time.Duration`]: https://golang.org/pkg/time/#Duration

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // was it seconds or milliseconds?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

Voltando ao exemplo de adicionar 24 horas em um instante de tempo, o método que foi utilizado para adicionar tempo depende da intenção. Se deseja o mesmo tempo do dia, porém no próximo dia do calendário, deve utilizar [`Time.AddDate`]. No entanto, se deseja um instante de tempo (garantido) de 24 horas após o horário anterior, devemos utilizar [`Time.Add`].

  [`Time.AddDate`]: https://golang.org/pkg/time/#Time.AddDate
  [`Time.Add`]: https://golang.org/pkg/time/#Time.Add

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```
#### Utilizar `time.Time` e `time.Duration` com sistemas externos

Utilizar `time.Duration` e `time.Time` em interações com sistemas externos quando possível. Por exemplo:

- Flags de linha de comando: [`flag`] suporta `time.Duration` via [`time.ParseDuration`]
- JSON: [`encoding/json`] suporta _enconding_ `time.Time` no formato [RFC 3339]
- SQL: [`database/sql`] suporta converter colunas `DATETIME` ou `TIMESTAMP` em `time.Time` e ao contrário também, se o driver (subjacente) suportar isso
- YAML: [`gopkg.in/yaml.v2`] suporta `time.Time` no formato [RFC 3339] _string_, e `time.Duration` via [`time.ParseDuration`]

  [`flag`]: https://golang.org/pkg/flag/
  [`time.ParseDuration`]: https://golang.org/pkg/time/#ParseDuration
  [`encoding/json`]: https://golang.org/pkg/encoding/json/
  [RFC 3339]: https://tools.ietf.org/html/rfc3339
  [`UnmarshalJSON` method]: https://golang.org/pkg/time/#Time.UnmarshalJSON
  [`database/sql`]: https://golang.org/pkg/database/sql/
  [`gopkg.in/yaml.v2`]: https://godoc.org/gopkg.in/yaml.v2

Quando não for possível utilizar `time.Duration` nessas interações, utilize `int` ou `float64`e inclua a unidade de medida no nome do campo.

Por exemplo, desde que `enconding/json` não suporta `time.Duration`, a unidade de medida é incluida no nome do campo.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

Quando não é possível utilizar `time.Time` nessas interações, a menos que haja um acordo em comum, utilize `string` e formate _timestamps_ no formato [RFC 3339]. Esse formato é utilizado como _default_ em [`Time.UnmarshalText`] e esta disponível para uso no `Time.Format` e `time.Parse` via [`time.RFC3339`].

  [`Time.UnmarshalText`]: https://golang.org/pkg/time/#Time.UnmarshalText
  [`time.RFC3339`]: https://golang.org/pkg/time/#RFC3339

Embora isto tenda a não ser um problema na prática, tenha em mente que o pacote "`time`" não suporta "_parsear_" _timestamps_ com segundos intercalares ([8728]) nem inclui os mesmos nos cálculos ([15190]). Se você comparar dois instantes de tempo, a difereça não inclui os segundos intercalares que podem ter ocorrido entre esses dois instantes.

  [8728]: https://github.com/golang/go/issues/8728
  [15190]: https://github.com/golang/go/issues/15190

<!-- TODO: seção sobre métodos de string para enumerações -->

### Erros

#### Tipo Erros

Existem várias opções para declarar erros:

- [`errors.New`] para erros formados por simples strings estáticos
- [`fmt.Errorf`] para erros com strings formatados
- Tipos customizados que implemetam o método `Error()`
- Erros "wrapped" utilizando [`"pkg/errors".Wrap`]

Ao retornar erros, considere o seguinte para determinar a melhor opção:

- Este é um erro simples que não precisa de informações extras? Nesse caso, [`errors.New`] deve ser suficiente.
- Os clientes precisam detectar e manipular esse erro? Nesse caso, você deve usar um tipo personalizado, utilize o método `Error ()`.
- Você está propagando um erro retornado por uma função downstream? Se sim, verifique o tópico [Utilizando Error Wrapping](utilizando-error-wrapping).
- Para outros casos, [`fmt.Errorf`] esta ok.

  [`errors.New`]: https://golang.org/pkg/errors/#New
  [`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf
  [`"pkg/errors".Wrap`]: https://godoc.org/github.com/pkg/errors#Wrap

Se o cliente precisar detectar o erro e você tiver criado um erro simples
usando [`errors.New`], use uma var para o erro.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Se você tiver um erro que os clientes talvez precisem detectar e você gostaria de adicionar
para obter mais informações (por exemplo, não é uma sequência estática), você deve usar um
tipo personalizado.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

Cuidado ao exportar tipos de erro personalizados diretamente, pois eles se tornam parte do
a API pública do pacote. É preferível expor funções de correspondência a
verifique o erro.

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```
#### Tipos de erro

Existem poucas maneiras de declarar erros.
Considere as seguintes informações antes de escolher a opção mais adequada para o seu caso de uso.

- O _caller_ (código que consumirá o erro) necessita identificar o erro para lidar com isso?
  Se sim, é necessário suportar as funções [`errors.Is`] ou [`errors.As`] através da declaração _top-level_ da váriavel de erro ou tipo customizado.
- O erro é uma mensagem _string_ estática ou é uma mensagem dinâmica que necessita informações contextuais?
  No primeiro caso, podemos utilizar [`errors.New`] e no segundo devemos utilizar [`fmt.Errorf`] ou um tipo customizado de erro.
- Esta propagando um novo erro retornado através de uma outra função?
  Se sim, veja [seção de error wrapping](#utilizando-error-wrapping).

[`errors.Is`]: https://golang.org/pkg/errors/#Is
[`errors.As`]: https://golang.org/pkg/errors/#As

|Identificar Erro?|Tipo de mensagem| Guidance                            |
|-----------------|---------------|-------------------------------------|
| Não              | estática        | [`errors.New`]                      |
| Não              | dinâmica       | [`fmt.Errorf`]                      |
| Sim             | estática        | `var` _top-level_ utilizando [`errors.New`] |
| Sim             | dinâmica       | Tipo de erro customizado           |

[`errors.New`]: https://golang.org/pkg/errors/#New
[`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf

Por exemplo, utilize [`errors.New`] para um erro com _string_ estática.
Exporte esse erro como variável para suportar sua identificação (_matching_) com a função `errors.Is`se o _caller_ precisa identificar e lidar com esse erro.

<table>
<thead><tr><th>Sem identificação do erro</th><th>Com identificação do erro</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```
Para um erro com _string_ dinâmico utilize [`fmt.Errorf`] se o _caller_ não precisa identificá-lo, e um `error` customizado se o _caller_ precisa identificá-lo.

</td></tr>
</tbody></table>

<table>
<thead><tr><th>Sem identificação do erro</th><th>Com identificação do erro</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  var notFound *NotFoundError
  if errors.As(err, &notFound) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Observe que se você exportar variáveis ou tipos de erro de um pacote,
eles se tornarão parte da API pública do pacote.

#### Utilizando Error Wrapping

Existem três opções principais para propagar erros se uma chamada falhar:

- Retorne o erro original se não houver contexto adicional a ser adicionado e você
  deseja manter o tipo de erro original.
- Adicione o contexto utilizando `fmt.Errorf` e a diretiva `%w`
- Adicione o contexto utilizando `fmt.Errorf` e a diretiva `%v`

Retornando o erro original (nativo - _as-is_) se não há contexto adicional para adicionar.
Isso mantém o tipo e messagem do erro original.
Isso é adequado para casos em que a mensagem de erro subjacente tem informações suficientes para permitir o rastreio (_tracking_).

Caso contrário, adicione contexto à mensagem de erro sempre que possível para que, em vez de um erro vago como "conexão recusada", você obtém erros mais úteis, como "chamar serviço foo: conexão recusada".

Utilize `fmt.Errorf` para adicionar contexto nos erros, escolhendo entre as diretivas `%w` ou `%v`
baseado em que o _caller_ deveria identificar e extrair a causa subjacente.

- Utilize `%w` se o _caller_ deveria ter acesso ao erro subjacente.
  Isso é um bom padrão para a maioria dos erros _wrapped_, mas esteja ciente que os _callers_ podem confiar nesse comportamento.
  Então nos casos quando o erro _wrapped_ é uma `var` ou tipo conhecido, documente e teste isso como parte da sua função.
- Utilize `%v` para ofuscar o erro adjacente. Os _callers_ não vão conseguir identificar ele. Mas se você pode alterar para `%w` no futuro se for necessário.

Ao adicionar contexto nos erros retornados, mantenha-o sucinto evitando frases como "falha ao"(_failed to_), o qual é um estado óbvio se acumulam através da _stack_ de erro.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "falha ao criar uma nova store: %s", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "nova store: %s", err)
}
```

<tr><td>

```
falhou ao chamar x: falhou ao chamar y: falha ao criar uma nova store: o erro
```

</td><td>

```
x: y: nova store: o erro
```

</td></tr>
</tbody></table>

No entanto, uma vez que o erro é enviado para outro sistema, deve ficar claro o
mensagem é um erro (por exemplo, uma tag `err` ou prefixo" Failed "nos logs).

Veja também [Don't just check errors, handle them gracefully].

[`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
[Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

#### Nomeando erros

Para valores de errors definidos como variáveis globais, use o prefixo `Err`ou `err` dependendo se eles são exportados. 
Esta orientação substitui a [Utilize o prefixo _ para globais não exportados](#utilize-o-prefixo-_-para-globais-não-exportados)

```go
var (
  // Os erros abaixo são exportados, sendo assim
  // os usuários deste pacote conseguem
  // identificá-los (comparar) com errors.Is.

  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  // Este erro não é exportado porque não
  // queremos que ele seja parte da nossa API
  // pública.
  // Ainda podemos utilizar ele dentro do pacote
  // chamando errors.Is.

  errNotFound = errors.New("not found")
)
```

Para tipos de erros customizados, utilize o sufixo `Erro`.

```go
// De forma similar, esse erro é exportado
// então os usuários do pacote conseguem
// identificar (comparar) com errors.As.

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

// Este erro não é exportado porque não
// queremos que ele seja parte da nossa API
// pública.
// Ainda podemos utilizar ele dentro do pacote
// chamando errors.As.

type resolveError struct {
  Path string
}

func (e *resolveError) Error() string {
  return fmt.Sprintf("resolve %q", e.Path)
}
```

### Manipular falhas de asserção de tipo

O retorno de um valor único de [type assertion] entrará em pânico devido a uma falha.
Portanto, sempre use o idioma "vírgula ok".

[type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // lide com o erro normalmente
}
```

</td></tr>
</tbody></table>

<!-- TODO: Existem algumas situações o retorno de um valor único é ok. -->

### Não utilize panic

O código em execução na produção deve evitar panic. O panic é uma importante fonte de
[cascading failures]. Se ocorrer um erro, a função deve retornar-lo de uma forma que
permita o chamador decida como lidar com isso.

[cascading failures]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0 {
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

O panic não é uma estratégia de tratamento de erros. Um programa deve entrar em panic apenas quando
algo irrecuperável acontece como uma dereferência nula. Uma exceção a isso é
inicialização do programa: coisas ruins na inicialização do programa que devem abortar o
programa pode causar panic.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```
Mesmo em testes, prefira `t.Fatal` ou` t.FailNow` sobre chamar panic para garantir que o
teste está marcado como falhou.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

<!-- TODO: Como utilizar _test packages. -->

### Utilize go.uber.org/atomic

Operações atômicas com o pacote [sync / atomic] operam nos tipos brutos
(`int32`,` int64` etc.), portanto é fácil esquecer de usar a operação atômica para
ler ou modificar as variáveis.

[go.uber.org/atomic] adiciona segurança de tipo a essas operações ocultando o
tipo utilizado. Além disso, inclui um conveniente tipo `atomic.Bool`.

  [go.uber.org/atomic]: https://godoc.org/go.uber.org/atomic
  [sync/atomic]: https://golang.org/pkg/sync/atomic/

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // ja esta executando
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // ja esta executando
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

## Performance

Performance-specific guidelines apply only to the hot path.

### Utilize strconv ao invés de fmt

Ao converter primitivas para / de strings, `strconv` é mais rápido que
`fmt`.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Evite a conversão de string para byte

Não crie slices de bytes a partir de um string repetidamente. Em vez disso, execute o
conversão uma vez e capture o resultado.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Utilize tamanho ao criar mapas

Sempre que possível, utilize o parametro de tamanho ao criar mapas com `make()`.

```go
make(map[T1]T2, hint)
```

Fornecer o parâmetro de capacidade para `make ()` faz com que o sistema tente
dimensionar memória corretamente no momento da inicialização, o que reduz a necessidade
de crescimento do mapa e alocações como elementos são adicionados ao mapa. Nota-se
que o parâmetro de capacidade não é garantida para mapas, portanto, é possível que
novos elementos ainda tenham que ser alocados, mesmo que o parâmetro de capacidade e seja fornecido.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := ioutil.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := ioutil.ReadDir("./files")

m := make(map[string]os.FileInfo, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` é criado sem parâmetro de tamanho; pode haver mais
alocações no tempo de atribuição.

</td><td>

`m` é criado com parâmetro de tamanho; pode haver menos
alocações no momento da atribuição.

</td></tr>
</tbody></table>

## Estilo

### Seja consistente

Algumas das diretrizes descritas neste documento podem ser avaliadas objetivamente;
outros são situacionais, contextuais ou subjetivos.

Acima de tudo, **seja consistente**.

Código consistente é mais fácil de manter, mais fácil de racionalizar, requer menos
sobrecarga cognitiva e é mais fácil migrar ou atualizar conforme novas convenções surgem
ou classes de bugs são corrigidas.

Por outro lado, ter vários estilos diferentes ou conflitantes em um único
base de código causa sobrecarga de manutenção, incerteza e dissonância cognitiva,
tudo isso pode contribuir diretamente para velocidades mais baixas, revisões de código dolorosas,
e bugs.

Ao aplicar essas diretrizes a uma base de código, é recomendável que as alterações
sejam feitas em um nível de pacote (ou maior): aplicação em um nível de subpacote
viola a preocupação acima, introduzindo vários estilos no mesmo código.

### Agrupar declarações similares

Go suporta o agrupamento de declarações semelhantes.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

Isso também se aplica a constantes, variáveis e declarações de tipo.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

Apenas declarações relacionadas ao grupo. Não agrupe declarações não relacionadas.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

Os grupos não são limitados no local em que podem ser usados. Por exemplo, você pode usá-los
dentro das funções.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  var red = color.New(0xff0000)
  var green = color.New(0x00ff00)
  var blue = color.New(0x0000ff)

  ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  ...
}
```

</td></tr>
</tbody></table>

### Ordenação e agrupamento de imports

Deve haver dois grupos para imports:

- Biblioteca do go
- Todo o resto

Esse é o agrupamento aplicado pelo goimports por padrão.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Nome de pacotes

Ao nomear pacotes, escolha um nome que seja:

- Todas as letras minúsculas. Sem maiúsculas ou sublinhados.
- Não precisa ter o import renomeado na sua utilização.
- Curto e sucinto. Lembre-se que o pacote deve ser identificado por inteiro a
cada chamada.
- Não utilize plural. Por exemplo, `net / url`, e não ` net / urls`.
- Não utilize "common", "util", "shared", or "lib". Estes são nomes ruins e pouco informativos.

Veja também [Package Names] e [Style guideline for Go packages].

  [Package Names]: https://blog.golang.org/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/

### Nome de Funções

Seguimos a convenção da comunidade Go de usar [MixedCaps for function
nomes]. É feita uma exceção para as funções de teste, que podem conter sublinhados
para fins de agrupar casos de teste relacionados, por exemplo,
`TestMyFunction_WhatIsBeingTested`.

  [MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

### Alias de Import

Alias para imports deve ser usado se o nome do pacote não corresponder ao último
elemento do caminho de importação

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

Em todos os outros cenários, os alias de importação devem ser evitados, a menos que haja um
conflito direto entre importações.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Agrupamento e ordenação de funçoes

- As funções devem ser ordenadas em ordem de chamada "aproximada".
- As funções em um arquivo devem ser agrupadas pelo receptor.

Portanto, as funções exportadas devem aparecer primeiro em um arquivo, após
Definições `struct`,` const`, `var`

Chamadas como `newXYZ ()` / `NewXYZ ()` podem aparecer após a definição do tipo, mas antes dos
restante dos métodos com receptor.

Como as funções são agrupadas pelo receptor, funções simples de utilidade devem aparecer
no final do arquivo

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reduzir o aninhamento

O código deve reduzir o aninhamento sempre que possível, tratando casos de erro ou
condições especiais primeiro e parando antes de percorer a iteração por completa.
Diminua a quantidade de código aninhado em vários níveis.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Else desnecessário

Se uma variável tem seu valor atribuído nas duas condições de um if/else, 
ela poderá ser substituída por um único if.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Declaração de variavéis top-level

Na declaração de variavéis top-level, utilize o termo 'var'. Não especifique o tipo,

At the top level, use the standard `var` keyword. Não especifique o tipo,
a menos que não seja do mesmo tipo que a expressão.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Nota-se que a função F() sempre retornará um string
// entao não devemos especificar o tipo de _s

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Especifique o tipo somente se o mesmo não corresponder ao tipo desejado
exatamente.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

### Utilize o prefixo _ para globais não exportados

Nomeie top-level `var`s e `const`s com o prefixo `_` para deixar claro quando
eles são usados, que os mesmos são globais.

Exceção: Erros não exportados, devem ser nomeados com o prefixo `err`.

Justificativa: Variáveis e constantes de nível superior têm um escopo de pacote. Usando um
nome genérico facilita o uso acidental do valor errado em outro
Arquivo

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // Neste caso, não haverá erro de compilação
  // se apagarmos a primeira linha da funçao Bar()
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

### Tipos embutidos em structs

Tipos embutidos (como mutexes) devem estar ao topo da lista de campos da struct,
e com uma linha em branco separando tipos embutidos dos campos "normais".

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

### Utilize nome dos campos para inicializar uma struct

Você quase sempre deve especificar nome dos campos ao inicializar structs. Essa regra
agora é imposta eplo [`go vet`]

  [`go vet`]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Exceção: os nomes dos campos *podem* ser omitidos nas tabelas de teste quando houver 3 ou
menos campos.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

### Declaração de variáveis locais

Declaração de variavéis de forma curta (`:=`) deve ser usada se a variável esta
sendo definida com um valor explicitamente.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

No entanto, há casos em que o valor padrão é mais claro quando o termo `var`
é usado. [Declarando fatias vazias], por exemplo.

  [Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### Nil é um slice válido

`nil` é um slice valido de tamanho 0. Isso significa que,

- Você não deve retornar um slice de tamanho zero explicitamente. Retorne `nil`

  <table>
  <thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- Para verificar se um slice esta vazio, sempre use `len(s) == 0`.
Não verifique pela expressão `nil`

  <table>
  <thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- O valor zero (declarado através do termo `var`) pode ser utilizado sem a chamada `make()`

  <table>
  <thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

### Reduzir o escopo das váriaveis

Sempre que possível, reduza o escopo das váriaveis. Não faça isso se ocasionará
um conflito com [Reduzir o aninhamento](#reduzir-o-aninhamento).

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

Se você precisa utilizar um resultado da função fora do if, então você não deve
reduzir o escopo.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Evitar passar parâmetros sem identificação

Parâmetros "naked", ou seja, passados sem nenhuma identificação podem
prejudicar a legibilidade. Adicione comentários no estilo de linguagem C 
(`/* ... */`) para esses parâmetros

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Ainda melhor, troque tipos "naked" booleanos para tipos customizados para ter
uma melhor legibilidade e tornar o código "type-safe". Isso permite mais que dois estados
(true/false) para esse parâmetro no futuro

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Talvez teremos o status StatusInProgress no futuro.
)

func printInfo(name string, region Region, status Status)
```

### Use strings literais para evitar escape 

Go suporta [raw string literals](https://golang.org/ref/spec#raw_string_lit), o que permite
gerar linhas multiplas, incluindo vírgulas. Isso serve para evitar "hand-escaped" strings
que são muito mais dificeis de ler

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Inicializando referências de structs

Utilize `&T{}` em vez de `new(T)` na inicialização de referências de structs, 
isso é mais consistence ao inicializar uma struct

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Inicializando mapas

Prefira `make(..)` para mapas vazios, e popule o mesmo programaticamente. 
Isso torna a inicialização de mapa visualmente distinta da declaração, 
e também torna mais fácil adicionar um tamanho depois.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 pode ser utilizado (escrita/leitura)
  // m2 gerará um panic ao receber uma escrita
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 pode ser utilizado (escrita/leitura)
  // m2 gerará um panic ao receber uma escrita
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

Declaração e incialização são visualmente parecidas.

</td><td>

Declaração e incialização são visualmente distintas.

</td></tr>
</tbody></table>

Quando possível, forneça o tamanho em uma inicialização de um mapa
com `make()`. Veja em [Utilize tamanho ao criar mapas](#utilize-tamanho-ao-criar-mapas)
for mais informações.

Se o mapa armazenar uma lista fixa de elementos, utilize a inicialização literal.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

A regra de forma geral é utilizar o modo literal para inicialização de mapas
quando há elementos fixos, caso contrário utilize `make` e especifique o tamanho

### Formate strings fora da função Printf

Se você declarar strings formatados no estilo de `Printf` fora dessas funções,
utilize valores do tipo `const`.

Isso ajuda a ferramenta `go vet` realizar uma análise estática do string formatado.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Nomeando funções da família Printf

Quando você declara uma função da família `Printf`, certifique-se que `go vet` pode detectar
and verificar o formtado do string.

Isso significa que você deve usar nomes predefinidos das funções da família `Printf`.
Dessa forma a ferramenta `go vet` vai verifica-la por padrão. Veja [Printf family] para
mais informações

  [Printf family]: https://golang.org/cmd/vet/#hdr-Printf_family

Se não é possível utilizar estes nomes predefinidos, o fim do nome que foi escolhido
deve ter o f: `Wrapf`, e não `Wrap`. `go vet` pode ser configurado para verificar
específicas funções da família `Printf`, mas seuos nomes devem acabar com f.

If using the predefined names is not an option, end the name you choose with
f: `Wrapf`, not `Wrap`. `go vet` can be asked to check specific `Printf`-style
names but they must end with f.

```shell
$ go vet -printfuncs=wrapf,statusf
```

Veja também [go vet: Printf family check].

  [go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/

## Patterns

### Test Tables

Utilize "table-driven tests" com o a técnica [subtests] para evitar código duplicado quando a 
principal lógica do teste é repetitiva.

  [subtests]: https://blog.golang.org/subtests

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

As "Test tables" tornam mais fácil adicionar contexto nas mensagens de erro, 
reduzir lógica duplicada e adicionar novos casos de teste.

Seguimos a convenção de que o slice de estruturas é referido como `tests`
e cada caso de teste `tt`. Além disso, incentivamos a explicar as entradas e saídas
valores para cada caso de teste com os prefixos `give` e` want`.

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

### Functional Options

Functional options é uma pattern que você declara um tipo `Option` oculto
que armazena a informação em alguma struct interna. Você aceita um número
variável dessas opções e age com base nas informações completas registradas
pelas opções na struct interna.

Utilize essa pattern para argumentos opcionais em construtores e outras APIs públicas
que você prevê a necessidade de expandir, especialmente se você já tem três ou
mais argumentos sobre essas funções.

<table>
<thead><tr><th>Ruim</th><th>Bom</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

</td><td>

```go
type options struct {
  timeout time.Duration
  caching bool
}

// Option overrides behavior of Connect.
type Option interface {
  apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
  f(o)
}

func WithTimeout(t time.Duration) Option {
  return optionFunc(func(o *options) {
    o.timeout = t
  })
}

func WithCaching(cache bool) Option {
  return optionFunc(func(o *options) {
    o.caching = cache
  })
}

// Connect creates a connection.
func Connect(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    timeout: defaultTimeout,
    caching: defaultCaching,
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}

// Options must be provided only if needed.

db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
  addr,
  db.WithCaching(false),
  db.WithTimeout(newTimeout),
)
```

</td></tr>
</tbody></table>

Veja também,

- [Self-referential functions and the design of options]
- [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->
