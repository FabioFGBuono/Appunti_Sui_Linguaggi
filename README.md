# Appunti Sui Linguaggi

---


**Attenzione questo testo è una bozza... probabilmente è il testo meno maturo di tutto il repository**


---


Questo testo l'ho scritto inizialmente pensando a me stesso di tanti anni fa, avrei voluto che qualcuno mi spiegasse quelle due o tre cose chiaramente così ho pensato di buttare giù due righe due... Vi assicuro che è interessante! Leggetelo e fatemi sapere se vi è piaciuto.

---

## Introduzione

Quando inizi a programmare magari impari un po' di sintassi, qualcuno ti mostra come scrivere una variabile, come fare un `if`, come chiamare una funzione, ma quello che quasi nessuno ti insegna è cosa accade davvero mentre il codice gira, e questa lacuna porta tanta confusione. Un linguaggio di programmazione è un insieme di regole su come i dati vengono archiviati, come le variabili esistono nel tempo e come i nomi vengono risolti durante l'esecuzione, e la sintassi è soltanto la superficie visibile di qualcosa di molto più articolato.


## Il Primo Grande Fraintendimento

La gente pensa che JavaScript e Python abbiano lo stesso sistema di tipi perché in nessuno dei due si scrivono i tipi in modo esplicito come in C, e questa è una conclusione sbagliata che nasce da una confusione tra due dimensioni completamente diverse.

### Forte vs Debole, Statico vs Dinamico

Quando qualcuno vi parla di tipizzazione **statica** sta dicendo che i tipi vengono verificati prima dell'esecuzione, al momento della compilazione, mentre la tipizzazione **dinamica** significa che il controllo avviene durante l'esecuzione.  La tipizzazione **forte** invece significa che il linguaggio rifiuta le operazioni tra tipi incompatibili producendo un errore, mentre la tipizzazione **debole** significa che il linguaggio accetta quelle operazioni facendo conversioni implicite per tenere il programma in piedi. Le due dimensioni sono ortogonali e non si implicano l'una con l'altra.

Python è dinamico e forte:

```python
x = 5
y = "ciao"
z = x + y  # TypeError: Python rifiuta, non converte.
```

JavaScript è dinamico e debole:

```javascript
x = 5;
y = "ciao";
z = x + y;  // "5ciao"
```

Compilatore o interprete deducono i tipi per te, ma come li gestiscono dopo averli dedotti è radicalmente diverso tra i linguaggi. C, Java, Rust e TypeScript occupano il quadrante statico e forte, dove si dichiarano i tipi, il compilatore li controlla e il linguaggio rifiuta le conversioni implicite. Python occupa il quadrante dinamico e forte, dove i tipi vengono dedotti a runtime e le operazioni tra tipi incompatibili producono errori. JavaScript occupa il quadrante dinamico e debole, dove i tipi vengono dedotti a runtime e il linguaggio fa conversioni creative per evitare qualsiasi crash.

```python
# Python
x = "5"
y = 10
x + y   # TypeError: can only concatenate str (not "int") to str
```

```javascript
// JavaScript
x = "5";
y = 10;
x + y   // "510" y viene convertito a stringa
```

La ragione della confusione è che in entrambi i linguaggi si scrive solo `x = 5` invece di `int x = 5`, ma quello riguarda la sintassi, non il sistema di tipi, che rimane irriducibilmente diverso tra i due. 

Ogni valore in Python è un oggetto che porta con sé un puntatore al suo tipo e il valore effettivo, per cui quando scrivi `x = 5` si crea un oggetto di tipo `int` con valore 5 e si memorizza un puntatore a quell'oggetto nella variabile `x`, e quando esegui `x + 10` Python consulta il tipo dell'oggetto a cui punta, vede che è `int` e usa il metodo di addizione specifico degli interi. JavaScript usa un meccanismo strutturalmente simile con una filosofia opposta: dove Python dice "ogni tipo ha regole rigide, se le violi, crash", JavaScript dice "farò tutto quello che posso per andare avanti, anche se devo fare conversioni inattese", e quando incontra operazioni tra tipi diversi cerca di tenere il programma in piedi anche se questo significa produrre risultati che a Python sembrerebbero assurdi.


## Il Secondo Grande Fraintendimento

Quando vedi `x = 5`, l'immagine mentale spontanea è che `x` sia una scatola con il numero 5 dentro ma non è così semplice, non sempre...

Entriamo in cucina.

## L'Environment

Immaginate che stia preparando una salsa béarnaise e abbia bisogno del burro che ho appena preso dal frigo, dell'aceto che ho versato nel pentolino, del timer per i cinque minuti, e devo anche sapere 
dove trovare il pentolone che mi serve.

Ora questa catena di mappe dove ogni livello dice "qui c'è questo, e se manca cerca al livello sopra" si chiama environment:

| Livello | Cosa contiene | Dove cercate |
|---------|---------------|-------------|
| **LOCAL (Salsa béarnaise)** | burro=100g, aceto=50ml, timer=5min | Prima qui |
| **FUNCTION PARENT** | pentolone=5L, sale=1kg | Se non trovo sopra, guardo qui |
| **GLOBAL** | tutte_ricette=[], print=stampa | Ultima risorsa |

Quando dico "dammi il burro", si cerca prima nel banco locale, poi al banco del sous-chef, poi al magazzino generale, risalendo la catena in modo lineare verso l'alto finché si trova quello che si cerca.

```
ENVIRONMENT CHAIN per salsa béarnaise:
┌─────────────────────────────────┐
│ LOCAL SCOPE (salsa béarnaise)   │
│ burro = ref_0x2000              │ ← Cercate qui primo
│ aceto = ref_0x2001              │
│ timer = 5                       │
│ parent → ↓                      │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│ FUNCTION SCOPE (cucina)         │
│ pentolone = ref_0x3000          │ ← Se non trovate sopra, qui
│ sale = ref_0x3001               │
│ parent → ↓                      │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│ GLOBAL SCOPE                    │
│ tutte_ricette = ref_0x4000      │ ← Infine qui
│ print = <builtin>               │
└─────────────────────────────────┘
```

Sul banco c'è un **bigliettino** che dice dove trovare il burro, e il burro vero sta altrove.


## Lo Store: Il Frigorifero

Il bigliettino sul banco dice "il burro è nella cella `0x2000`", e il vero burro sta nel **frigorifero**, che è lo **store**, un grande elenco di celle con numeri di indirizzo.

| Indirizzo | Che c'è dentro | Tipo | Quante mani lo toccano |
|-----------|----------------|------|------------------------|
| **0x2000** | 100g burro | DATO | 2 (io + env locale) |
| **0x2001** | 50ml aceto | DATO | 1 (io) |
| **0x3000** | 5L pentolone | DATO | 1 (env parent) |
| **0x4000** | {nome: "béarnaise", steps: [...]} | OGGETTO | 2 |

Quando dico "aumenta il burro a 150g", si consulta l'environment per trovare `burro = ref_0x2000`, si va al frigorifero alla cella `0x2000` e si cambia il contenuto da `100g` a `150g`, per cui se cambio il burro tutti quelli che lo referenziano vedono il burro nuovo, perché esiste un'unica verità condivisa.

Questo spiega un comportamento che confonde ogni principiante Python:

```python
x = [1, 2, 3]
y = x
y.append(4)
print(x)  # [1, 2, 3, 4]
```

`x` e `y` sono due bigliettini che puntano alla stessa cella del frigorifero, e modificare il contenuto della cella lo rende visibile attraverso entrambi.

## Tabelle e Paradigmi

Non so se la metafora della cucina ha funzionato ma.... paradigmi... env, store come tabelle, anzi prima ENV e poi per spiegare i puntatori introduco lo STORE.


## Lo Stack di Chiamata

Avete presente una pila di vassoi in un ristorante? Lo stack è simile e si chiama una funzione si mette un nuovo vassoio in cima con tutto quello che serve, i parametri e le variabili locali, e quando la funzione finisce si toglie il vassoio e tutto quello che c'era sopra sparisce.

```python
def somma(x, y):
    locale_z = x + y
    return locale_z

def main():
    a = 5
    b = 10
    risultato = somma(a, b)
```

Quando si chiama `main` lo stack ha un frame con `a=5` e `b=10`; quando `main` chiama `somma` si aggiunge un frame con `x=5`, `y=10`, `locale_z=15`; quando `somma` ritorna il suo frame viene rimosso, `locale_z` cessa di esistere e il risultato (15) viene memorizzato in `risultato` nel frame di `main`. Quando uno scope finisce il suo environment viene distrutto, mentre gli oggetti a cui l'environment puntava restano nello store finché qualcuno li referenzia ancora. Capire come avviene la gestione dello stack di chiamata nei vari tipi di linguaggio è fondamentale per essere buoni programmatori ma anche per essere buoni progettisti.


## Scope Lessicale

La maggior parte dei linguaggi moderni usa lo scope lessicale, per cui quando si vuole sapere quale variabile intende il codice si guarda il testo del codice stesso.

```python
x = 10  # Globale

def funzione_esterna():
    x = 20  # Locale a funzione_esterna

    def funzione_interna():
        print(x)  # Il testo mostra che funzione_interna
                  # è dentro funzione_esterna -> x vale 20

    funzione_interna()

funzione_esterna()  # Stampa: 20
```

La risoluzione dei nomi procede guardando prima lo scope locale, poi quello che lo racchiude e poi il globale, producendo un errore se il nome manca ovunque.

### Scope Dinamico

Esiste anche lo scope dinamic*, dove la risoluzione dei nomi dipende da chi ha chiamato la funzione piuttosto che dalla sua posizione nel testo, e Bash lo usa mentre alcuni dialetti Lisp lo usavano in passato. I linguaggi moderni lo evitano perché rende il codice difficile da ragionare, dato che lo stesso nome può riferirsi a cose diverse a seconda del percorso di chiamata:

```bash
# Pseudocodice con scope dinamico
x = 10

function esterno() {
    x = 20
    interno       # chiama interno con x=20 nel contesto
}

function interno() {
    echo $x       # con scope dinamico: chi mi ha chiamato?
                  # esterno mi ha chiamato, con x=20
                  # risultato: 20
}

esterno           # stampa: 20

# Se chiamo interno direttamente:
interno           # chi mi ha chiamato? Lo script principale, con x=10
                  # risultato: 10
```

Con scope lessicale `interno` produrrebbe sempre un errore perché `x` manca nel suo scope testuale, mentre con scope dinamico il risultato dipende dalla storia delle chiamate, e questa imprevedibilità è esattamente ciò che lo rende una scelta progettuale fragile.


## Quando lo Scope Cattura il Tempo

Una **closure** è una funzione che cattura l'environment dello scope in cui è stata definita, anche dopo che quello scope è finito.

```javascript
function crea_contatore() {
    let count = 0;

    return function() {
        count++;
        return count;
    };
}

const contatore = crea_contatore();

console.log(contatore());  // 1
console.log(contatore());  // 2
console.log(contatore());  // 3
```

`crea_contatore()` viene eseguita e crea un environment locale con `count = 0`, poi una funzione anonima viene creata dentro quell'environment e viene ritornata, quando `crea_contatore()` finisce il suo environment locale sarebbe destinato a sparire, ma la funzione ritornata contiene un riferimento a quell'environment e lo mantiene attivo, per cui ogni volta che si chiama il contatore si modifica la stessa variabile `count` intrappolata nel closure. Le closure nascono dalla combinazione tra scope lessicale e funzioni di primo livello, cioè funzioni trattate come valori ordinari che si possono passare, ritornare e assegnare a variabili. Python li ha, JavaScript li ha, C no.


## Garbage Collection

Quando finisco la salsa béarnaise l'environment della funzione muore, ma il burro è ancora nel frigorifero, e qualcuno deve pulire. In realtà il paragone è un po' forzato perché questa metafora per essere vera dovrebbe considerare lo stesso burro che era nella salsa ma quello è nella pancia dell'avventore del ristorante, ma facciamo finta sia nel frigo.

### Il Metodo del Sous-Chef Paranoico (Reference Counting)

Ogni volta che qualcuno usa il burro si aggiunge una tacca, e quando la tacca scende a zero si butta via perché non serve più... lo so anche questa è forzata ma avere capito il concetto.

| Fase | Burro (0x2000) | Aceto (0x2001) | Cosa succede |
|------|----------------|----------------|--------------|
| **INIZIO** | ref_count=1 | ref_count=1 | Creati |
| **SALSA USA BURRO** | ref_count=2 | ref_count=1 | Una mano in più |
| **SALSA FINISCE** | ref_count=1 | ref_count=0 | Aceto rimosso |
| **NESSUNO USA BURRO** | ref_count=0 | — | Burro rimosso |

Quando il conteggio arriva a zero la memoria viene liberata subito, ma se il burro punta all'aceto e l'aceto punta al burro, nessuno dei due arriva mai a zero e si ottiene memoria occupata da oggetti che nessuno usa, insomma, una versione informatica del frigorifero pieno di roba marcia.

### Il Metodo del Direttore Ossessionato (Mark and Sweep)

A intervalli periodici il garbage collector fa un'ispezione, prima marcando tutto ciò che è raggiungibile dall'ambiente attivo e poi eliminando tutto il resto.

```
PRIMA:  frigorifero ha -> burro, aceto, ricetta_vecchia, ...

GC PARTE:
  Mark: burro ✓, ... ✓
  Sweep: aceto ✗ DELETE, ricetta_vecchia ✗ DELETE

DOPO:   frigorifero ha -> burro, ...
```

E funziona anche con cicli, infatti se il burro e l'aceto si puntano a vicenda ma nessuno li raggiunge dall'ambiente esterno vengono eliminati comunque, al costo di dover fermare tutto per qualche istante durante l'ispezione. In **C** la pulizia è responsabilità del programmatore:

```c
int *x = malloc(sizeof(int));
*x = 5;
free(x);   // dimenticare questo produce memory leak
```

In Python e JavaScript il garbage collector lavora in automatico, con un costo in termini di CPU e pause occasionali.


## I Puntatori

In linguaggi come C e C++, si può chiedere esplicitamente l'indirizzo di memoria di una variabile con l'operatore `&` e memorizzarlo in un **puntatore**.

```c
int main() {
    int x = 10;
    int *ptr = &x;

    printf("%d\n", *ptr);  // stampa il valore puntato: 10
    *ptr = 20;             // modifica il valore attraverso il puntatore
    printf("%d\n", x);     // x è ora 20
}
```

I puntatori diventano insidiosi quando si memorizza un puntatore a una variabile locale che poi sparisce, perché il puntatore si ritrova a indicare un'area di memoria dismessa, il cosiddetto dangling pointer:

```c
int* funzione_cattiva() {
    int x = 10;
    return &x;  // ERRORE: il puntatore sopravvive alla variabile
}
```

Python e JavaScript nascondono al programmatore tutta questa meccanica, gestendola in modo trasparente, anche se i puntatori esistono a livello di implementazione perché ogni variabile Python è un puntatore a un oggetto nello store.


## Tipi Espliciti vs Dedotti

C'è una terza cosa che genera confusione, quella tra dichiarare esplicitamente il tipo e lasciare che il linguaggio lo deduca. In C si scrive `int x = 5` con il tipo dichiarato dal programmatore, in Python si scrive `x = 5` e il linguaggio deduce che `x` è un `int` dal valore assegnato, mentre in TypeScript si può fare entrambe le cose:

```typescript
let x: number = 5;  // esplicito
let y = 5;          // dedotto da TypeScript che sa che y è number
```

Python usa solo la deduzione ed è fortemente tipizzato, TypeScript usa sia dichiarazioni esplicite sia deduzione ed è statico.

| | Esplicito | Dedotto |
|---|---|---|
| **Statico** | C, Java (`int x = 5`) | Rust, Go, TypeScript con inference |
| **Dinamico** | raro | Python, JavaScript |

Vedere `int x = 5` porta a pensare "questo linguaggio è come C", ma Python, pur scrivendo solo `x = 5`, mantiene una tipizzazione forte che fa rispettare a runtime invece che a compile-time.


## L'Environment in un Linguaggio Dinamico

In un linguaggio tipizzato staticamente come C l'environment è prevedibile perché il compilatore sa a compile-time quanto spazio allocare per ogni variabile e il tipo di ciascuna rimane fisso per tutta la sua vita, mentre in un linguaggio tipizzato dinamicamente come Python l'environment è molto più flessibile:

```python
x = 5          # x punta a un oggetto int
x = "ciao"     # x punta ora a un oggetto str
x = [1, 2, 3]  # x punta ora a una lista
```

L'environment traccia il nome e l'indirizzo a cui punta in quel momento, e il tipo è una proprietà dell'oggetto nel frigorifero piuttosto che del bigliettino sul banco, per cui Python può permettere che `x` cambi tipo aggiornando il bigliettino a un nuovo indirizzo, mentre l'oggetto precedente, se nessuno lo referenzia più, viene rimosso dal garbage collector.

Formalmente si scrive `Γ ⊢ e : τ`, dove `Γ` è l'environment, una mappatura da nomi a tipi, `⊢` significa "dimostra che", `e` è l'espressione e `τ` è il tipo. Per un linguaggio fortemente tipizzato le regole sono rigide:

```
REGOLA DI ADDIZIONE (Forte - Python)
Γ ⊢ e₁ : int    Γ ⊢ e₂ : int
─────────────────────────────
Γ ⊢ e₁ + e₂ : int
```

Per un linguaggio debolmente tipizzato le regole ammettono coercioni:

```
REGOLA DI ADDIZIONE (Debole - JavaScript)
Γ ⊢ e₁ : number    Γ ⊢ e₂ : string
─────────────────────────────────────
Γ ⊢ e₁ + e₂ : string       (coercione implicita)
```

JavaScript ha molte più regole che ammettono conversioni tra tipi diversi, mentre Python ne ha meno perché rifiuta le operazioni prive di senso semantico... come è giusto che sia secondo me.


## I Livelli dell'Environment

In un linguaggio moderno non c'è l'environment ma molti e sono organizzati in livelli con responsabilità distinte, per esempio l'environment globale esiste per tutta la durata del programma e contiene i nomi delle funzioni definite a livello principale, le costanti globali e le funzioni built-in. L'environment di modulo isola i nomi definiti in ciascun file, e importare un modulo significa rendere accessibile il suo environment da quello chiamante. Invece l'environment di funzione viene creato ogni volta che si chiama una funzione e distrutto quando ritorna, a meno che un closure non lo stia catturando. Infine l'environment di blocco, presente in C e in JavaScript con `let` e `const` ma assente in Python, crea un nuovo scope per ogni coppia di parentesi graffe.

```python
x = 1  # Globale

def funzione():
    x = 2  # Locale alla funzione

    if True:
        x = 3  # In Python, questo modifica lo stesso x della funzione
               # perché Python non ha scope di blocco.
               # In JavaScript con 'let', creerebbe un x separato.

    print(x)  # 3
```

In JavaScript `let x = 3` dentro un `if` crea una variabile che esiste solo dentro quel blocco, mentre in Python `x = 3` dentro un `if` modifica la variabile locale della funzione corrente.

## La Confusione

Le persone imparano la sintassi ma non il modello di esecuzione e questo è tremendo perché imparano che in C si scrive `int x = 5`, che in Python si scrive `x = 5`, che in JavaScript si scrive `let x = 5`, e pensano che in Python e JavaScript sia tutto uguale perché si omette il tipo. Rimane però nascosto che Python è fortemente tipizzato e ogni valore porta con sé un tipo rigido, che JavaScript è debolmente tipizzato e i valori vengono convertiti in modo silenzioso, e che entrambi usano scope lessicale ma con regole di blocco diverse, e che entrambi usano l'environment per tracciare le variabili ma gestiscono i tipi in modo radicalmente diverso.

Imparare la teoria dei linguaggi, la semantica operazionale, gli environment e i sistemi di tipi, rende visibili queste distinzioni, il problema è che queste fondamenta vengono insegnate come dettagli tecnici opzionali quando invece sono fondamentali per capire come funziona un linguaggio.


## Il λ-Calcolo

Tutto quello che abbiamo descritto, environment, scope, tipi e closure, ha radici in un sistema formale inventato da Alonzo Church negli anni '30, il **λ-calcolo**. La sintassi completa del λ-calcolo è questa:

```
e ::= x          (variabile)
    | λx.e       (astrazione: funzione con parametro x e corpo e)
    | e₁ e₂      (applicazione: applica e₁ a e₂)
```

Notate ci sono solo variabili, funzioni e applicazione di funzioni. E quindi come si rappresenta il numero 5? Come comportamento:

```
5 = λf. λx. f (f (f (f (f x))))
```

Se si passa una funzione `f` e un valore `x`, `f` viene applicata cinque volte, e il numero 5 è codificato come trasformazione piuttosto che come simbolo. Quando si scrive una funzione in Python:

```python
def somma(x, y):
    return x + y
```

Si sta descrivendo, nel senso formale di Church, `λx. λy. x + y`, e quando si chiama `somma(5, 10)` si esegue una **β-riduzione** dove i parametri vengono sostituiti con i valori effettivi:

```
(λx. λy. x + y) 5 10
→ (λy. 5 + y) 10        (sostituisci x con 5)
→ 5 + 10                 (sostituisci y con 10)
→ 15
```

La freccia `→` indica "si riduce a" ed è il passo elementare di calcolo.

### Due Modi per Eseguire la Sostituzione

Quando si chiama una funzione, come si gestisce l'ambiente? Nel modello a sostituzione i parametri vengono rimpiazzati direttamente nel corpo della funzione:

```
(λx. x + 1) 5
→ 5 + 1        (x viene rimpiazzato con 5 nel corpo)
→ 6
```

Nel modello a environment si crea un nuovo environment locale che contiene le associazioni tra parametri e valori, e il corpo viene valutato in quell'environment:

```
(λx. x + 1) 5

Crea environment: {x ↦ 5}
Valuta (x + 1) in quell'environment
Risultato: 6
```

Python e JavaScript usano il modello a environment mentre C usa la sostituzione, e anche se i risultati coincidono il percorso è diverso influendo sulle closure. Un closure, in questo formalismo, è una coppia `⟨λx. e, Γ⟩` formata dalla funzione anonima e dall'environment al momento della sua definizione, e quando si applica il closure a un valore il corpo viene valutato nell'environment catturato esteso con la nuova associazione.


## Strategie di Valutazione

Ma quando si valutano gli argomenti di una funzione? La valutazione stretta, spesso chiamata *eager* o *call-by-value*, valuta tutti gli argomenti prima di passarli alla funzione, e quasi tutti i linguaggi moderni adottano questa strategia, tra cui Python, JavaScript, C e Java:

```python
def foo(x):
    return 10

result = foo(5 + 5)  # Prima calcola 5+5=10, poi chiama foo(10)
```

Nel λ-calcolo questo si scrive `(λx. 10) (5 + 5) → (λx. 10) 10 → 10`, dove l'argomento viene valutato prima dell'applicazione. La valutazione pigra, chiamata anche *lazy* o *call-by-need*, aspetta a valutare un argomento finché strettamente necessario, e Haskell lo fa nativamente: `(λx. 10) (5 + 5) → 10`, dove l'applicazione avviene prima e `5 + 5` rimane non valutato. La differenza sembra marginale ma con una valutazione pigra se una funzione ignora un argomento quell'argomento viene ignorato del tutto con risparmio di tempo e memoria, e si possono definire strutture dati teoricamente infinite lavorandoci in pezzi. La semantica cambia in modo fondamentale davanti a espressioni come `let x = 1/0 in 42`: con valutazione stretta questo produce un errore di divisione per zero, con valutazione pigra produce 42 perché `x` viene ignorato e quindi rimane non valutato, e questa è una differenza filosofica su cosa significa "il valore di una variabile".


## L'Inferenza di Tipo

Come fa un linguaggio a conoscere il tipo di un'espressione senza che il programmatore lo dichiari? Tramite **type inference**, e l'algoritmo più raffinato si chiama **Hindley-Milner**, usato in Haskell, OCaml e Rust. Quando si incontra un'espressione di tipo sconosciuto si osserva come viene usata e si deduce cosa deve essere.

```haskell
f x = x + 1
```

`x` viene usato con `+`, che richiede un numero, quindi `x` deve essere numerico, e `x + 1` produce un numero, quindi il tipo di ritorno è numerico e la soluzione è `f :: Num a => a -> a`. Il processo genera un sistema di vincoli e li risolve tramite una unificazione, così se si scopre che una variabile di tipo deve essere simultaneamente `int` e `string`, una contraddizione, si segnala un errore, mentre se i vincoli sono coerenti la soluzione viene propagata a tutto il codice. TypeScript fa lo stesso automaticamente:

```typescript
let f = (x) => x + 1;
// TypeScript deduce: f : (x: number) => number
```

E se si tenta `f("ciao")`, TypeScript ha trovato la contraddizione prima che il programma girasse, con il tipo di `f` dedotto automaticamente senza che nessuno lo dichiarasse esplicitamente.

## Polimorfismo Parametrico

La funzione funziona per qualsiasi tipo, specificato come parametro. In Haskell:

```haskell
id :: a -> a
id x = x
```

La variabile di tipo `a` è un parametro, e formalmente il tipo di `id` si scrive `∀a. a → a`, che si legge "per ogni tipo a, id ha il tipo a → a". TypeScript chiama questo meccanismo "generics":

```typescript
function id<T>(x: T): T {
    return x;
}

id<number>(5);      // T = number
id<string>("hi");   // T = string
```

## Polimorfismo Ad-Hoc

Il polimorfismo ad-hoc fornisce versioni diverse della stessa funzione per tipi diversi e sceglie quale usare in base al tipo degli argomenti; formalmente sono funzioni distinte con lo stesso nome. In Haskell questo meccanismo si realizza attraverso le **type classes**:

```haskell
class Eq a where
    (==) :: a -> a -> Bool

instance Eq Int where
    x == y = ...

instance Eq String where
    x == y = ...
```

Il tipo di `(==)` è `∀a. Eq a ⇒ a → a → Bool`, che si legge "per ogni tipo `a` che appartiene alla class `Eq`, l'operatore di uguaglianza ha tipo `a → a → Bool`", e il compilatore sceglie quale istanza usare in base al tipo degli argomenti, risolvendo i vincoli a compile-time.

## Duck Typing

Python e JavaScript adottano un approccio ancora più libero, dove il tipo formale di un valore conta meno del suo comportamento, e basta che implementi i metodi richiesti:

```python
def somma(x, y):
    return x + y

somma(5, 10)           # int + int
somma("hi", "there")   # str + str
somma([1], [2, 3])     # list + list = [1, 2, 3]
```


## La Corrispondenza di Curry-Howard

Le regole di tipo in un linguaggio sono matematicamente identiche alle regole della logica intuizionista quindi abbiamo che il tipo `A → B` è una funzione che prende un `A` e ritorna un `B`, e corrisponde all'implicazione logica "se A allora B", e la regola di eliminazione dell'implicazione in logica è la stessa della regola di tipizzazione per l'applicazione di funzione:

```
A ⊃ B    A          f : A → B    x : A
───────────         ──────────────────
     B                   f x : B
```

I tipi sono proposizioni logiche, i termini sono prove di quelle proposizioni, e il type checker è una macchina che verifica automaticamente la correttezza delle prove, in quella che si chiama **corrispondenza di Curry-Howard**:

```haskell
-- Questo tipo è abitabile?
f :: (A -> B) -> (B -> C) -> (A -> C)

-- Sì, ecco la prova:
f g h x = h (g x)

-- Cosa significa logicamente?
-- "Se A implica B, e B implica C, allora A implica C"
-- Per transitività è sempre vera.
```

Un tipo privo di termini corrisponde a una proposizione falsa, e quando un programma compila senza errori di tipo il compilatore ha verificato una prova, mentre quando il programma termina correttamente la prova è costruttiva.

## Mutabilità

Finora abbiamo assunto che la stessa funzione con stesso input e stesso output ma nei linguaggi, specialmente imperativi, abbiamo la mutabilità, la capacità di cambiare il valore di un oggetto nel frigorifero, il modello formale diventa più complesso perché occorre tracciare lo stato del sistema nel tempo, e formalmente lo store viene rappresentato come una funzione da indirizzi a valori `σ : Loc → Val`.

Quando si scrive `y = x` in Python, si crea un alias, due nomi che puntano allo stesso indirizzo, per cui `σ = {loc₀ ↦ [1, 2, 3]}` con `x ↦ loc₀` e `y ↦ loc₀`, e quando si esegue `y.append(4)` si modifica il valore in `loc₀` ottenendo `σ' = {loc₀ ↦ [1, 2, 3, 4]}`, per cui sia `x` che `y` vedono il nuovo valore: due bigliettini, stesso frigorifero.

Una funzione che dipende da stato mutabile esterno cessa di essere **pura** e può indurre effetti collaterali:

```python
counter = 0

def increment():
    global counter
    counter = counter + 1
    return counter

increment()  # 1
increment()  # 2
```

Qui lo stesso input produce output diversi a ogni chiamata, e una funzione del genere resiste alla parallelizzazione e al ragionamento formale. Haskell risolve il problema vietando la mutabilità per default, e quando si vuole stato mutevole occorre usare un'astrazione esplicita come la monade `IO` o `State`, che rende visibile nel tipo della funzione la presenza di effetti collaterali.


## Semantica Denotazionale

E se ci chiedessimo qual è il significato matematico di un programma? Ogni termine del linguaggio denota un oggetto matematico astratto, e il simbolo `⟦⟧` indica "il significato di": `⟦5⟧ = 5 ∈ ℕ`, `⟦λx. x + 1⟧ = { (0, 1), (1, 2), (2, 3), ... } ⊆ ℕ × ℕ`, e `⟦somma(5, 10)⟧ = 15`.

Programmi sintatticamente diversi possono avere la stessa denotazione:

```haskell
f x = x + 1   -- Programma 1
f x = 1 + x   -- Programma 2

⟦Programma 1⟧ = ⟦Programma 2⟧ = { (0, 1), (1, 2), ... }
```

Sono funzioni diverse nella scrittura ma denotano la stessa funzione matematica, per i linguaggi imperativi con stato le denotazioni diventano più elaborate perché una funzione che modifica lo stato denota una relazione tra stati, e per i programmi che non terminano si introduce il simbolo ⊥, per cui `⟦ while True: pass ⟧ = ⊥`.

## Analisi Statica

Una delle capacità più preziose dei compilatori moderni è l'analisi statica, questa è l'analisi del comportamento del programma senza eseguirlo. L'analisi del flusso di dati traccia dove vanno i valori nel programma, quindi per il codice `x = 5`, `y = x`, `z = y + 10`, si costruisce un grafo dove `5 → x → y → z` con `z = y + 10`, e analizzando questo grafo il compilatore deduce che `z` vale sempre 15 e può sostituirlo direttamente, eliminando il calcolo a runtime.


```python
x = 5
y = 10
print(x)  # y non verrà mai più usata quindi non serve più
```

Un compilatore attento vede che `y` viene assegnata ma mai usata e può evitare di allocare memoria per essa. Per i linguaggi che ammettono funzioni impure, si possono analizzare gli effetti di una funzione e annotarli nel tipo:

```haskell
f :: Int -> Int {readFile}   -- f può leggere file
g :: Int -> Int {noeffects}  -- g è pura
```

Conoscere gli effetti permette di parallelizzare le funzioni pure, memoizzare i loro risultati e riordinare le computazioni. Ad esempio se abbiamo una funzione che calcola f(x) e abbiamo f(x)+f(x) potrei calcolare solo una volta f(x) e fare 2*f(x), se invece f(x) non è pura può avere effetti collaterali, cioè se f(x) non è sempre uguale, perché magari c'è un input, allora il compilatore non può effettuare l'ottimizzazione.

## Tipi Dipendenti

Un tipo dipendente è un tipo che dipende da un valore, per capirci il tipo "lista di lunghezza esatta n", scritto `Vec 5 Int`. In Agda o Idris si può scrivere:

```agda
append :: Vec n a -> Vec m a -> Vec (n + m) a

x : Vec 3 Int
y : Vec 2 Int
z : Vec 5 Int := append x y  -- il compilatore verifica che 3 + 2 = 5
```

Il compilatore sa che `append x y` ha lunghezza 5 perché 3 + 2 = 5, verificato a compile-time, e intere categorie di bug come buffer overflow e accesso fuori dai limiti diventano impossibili a livello di tipo.

## Universe Levels e la Gerarchia dei Tipi

Nelle teorie dei tipi più elaborate, i tipi stessi hanno tipi, e questa gerarchia va tenuta sotto controllo per evitare paradossi analoghi al paradosso di Russell:

```
Type : Type 1
Type 1 : Type 2
Type 2 : Type 3
...
```

Ammettere `T : T` apre la strada a costruzioni paradossali che rendono il sistema inconsistente, mentre la gerarchia garantisce che ogni tipo appartenga a un universo strettamente superiore a quello dei suoi elementi, spezzando la circolarità.


## Continuazioni

Una continuazione è tutto quello che il programma farà dopo un certo punto, il futuro del calcolo rappresentato come funzione `k : A → Result`, dove `k` prende il valore prodotto dall'espressione corrente e produce il risultato finale. Il linguaggio gestisce questo automaticamente, ma **call/cc** permette di accedere alla continuazione corrente come valore, salvarla, passarla e richiamarla in qualsiasi momento:

```scheme
(call/cc (λk
  (+ 1 (k 10))))  ; k rappresenta "il resto del programma"
                   ; chiamare (k 10) salta al futuro con valore 10
                   ; la (+ 1 ...) viene ignorata -> risultato: 10
```

Con le continuazioni si può implementare qualsiasi struttura di controllo come early return, eccezioni, coroutine e backtracking come funzione di libreria, con supporto proveniente dalla semantica del linguaggio piuttosto che da primitive speciali.

## Continuazioni e Logica

Il tipo di `call/cc` è `((A → ⊥) → A) → A`, che corrisponde alla **legge di Peirce** nella logica classica, cioè "se assumendo che A sia falso puoi provare A, allora A è vero", il principio di reductio ad absurdum. Le continuazioni sono logicamente equivalenti al ragionamento classico, per cui aggiungere `call/cc` a un linguaggio è lo stesso che arricchire la logica sottostante passando dall'intuizionismo alla logica classica.

---

## In chiusura

I linguaggi di programmazione sono una applicazione diretta della logica matematica e quando si scrive un programma type-safe si sta dimostrando un teorema a tutti gli effetti quindi se il programma compila senza errori di tipo il compilatore ha verificato una prova, e quando il programma termina correttamente la prova è stata verificata in modo costruttivo. Qualche paragrafo fa dicevo che serve capire il modello di esecuzione e forse ora è più chiaro che la sintassi è soltanto la superficie, il modello di esecuzione è la sostanza.

## Le Frontiere Attuali

I sistemi di tipi continuano a evolversi verso forme sempre più espressive, i tipi dipendenti, i linear types, adottati da Rust con il sistema di ownership e da Linear Haskell che tracciano chi possiede una risorsa e quando può essere rilasciata, eliminando intere categorie di bug di memoria. I sistemi di effetti che annotano le funzioni con le risorse che usano, permettendo ottimizzazioni aggressive sulle funzioni pure. E il gradual typing che permette di mescolare codice staticamente tipizzato e codice dinamico nello stesso programma, migrando gradualmente verso garanzie più forti. Il sogno è scrivere specifiche matematiche precise e farle verificare automaticamente dal compilatore, e un giorno sarò cosi e i bug saranno strutturalmente impossibili.

Ora andate a cucinare, e fate attenzione alla memoria. 👨‍🍳




---


# 📜 **Licenza (IT + EN)**


## 🇮🇹 **Italiano**

Questo documento è rilasciato sotto licenza **Creative Commons CC BY‑NC‑ND 4.0**.  
È consentita la **condivisione** del testo, a condizione che:

- venga fornita **attribuzione** all’autore;  
- non venga effettuato **alcun uso commerciale**;  
- il contenuto non venga **modificato**, **adattato**, **remixato** o trasformato in alcun modo.

**Eccezione per la traduzione:** In deroga alla clausola *ND (NoDerivatives)*, è **espressamente consentita la traduzione integrale** del documento, purché:

- la traduzione sia **fedele** al testo originale;  
- venga mantenuta l’**attribuzione** all’autore;  
- non venga effettuato **alcun uso commerciale**;  
- sia chiaramente indicato che si tratta di una **traduzione autorizzata**.

Qualsiasi altra forma di modifica o opera derivata non è permessa.

---

## 🇬🇧 **English**

This document is licensed under the **Creative Commons CC BY‑NC‑ND 4.0** license.  
You may **share** the text provided that:

- proper **attribution** is given to the author;  
- no **commercial use** is made;  
- the content is not **modified**, **adapted**, **remixed**, or transformed in any way.

**Translation Exception**: As an explicit exception to the *ND (NoDerivatives)* clause, **full and faithful translations** of this document are permitted, provided that:

- the translation remains **faithful** to the original text;  
- proper **attribution** to the author is maintained;  
- no **commercial use** is made;  
- it is clearly stated that the work is an **authorized translation**.

Any other form of modification or derivative work is not allowed.

---

---
