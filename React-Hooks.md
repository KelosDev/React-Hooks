# **REACT HOOKS**

## ***useState()***

Serve a gestire lo stato del componente. Voglio avere una variabile, voglio poterla aggiornare per fare delle cose. 

Questa variabile rappresenta lo stato del nostro componente.

***useState()*** restituisce una variabile e una funzione che aggiorna la variabile.

Facciamo subito un esempio: **CONTATORE**

```tsx
const App: React.FC = () => {

const [count, setCount] = useState(0) // valore di count inizializzato a 0 tra le parentesi dello useState(*)

const increment = () => {
    setCount(count + 1)
    console.log(count)
}

const decrement = () => {
    setCount(count - 1)
    console.log(count)
}

console.log('rendering')

return (

    <>
    <p> Count : {count} </p>
    <button onClick={increment}> + </button>
    <button onClick={decrement}> - </button>
    </>

    )
}
```

**Problema**

- Se eseguiamo il codice, noteremo nella console che ogni volta che incremento o decremento il contatore, il componente verrà nuovamente renderizzato.
- Un'altro problema è che il console.log() non sta funzionando correttamente, questo perchè ***setCount*** è **asincrono** --> Questo problema si risolve con il prossimo Hook

---

## ***useEffect()***

Riprendendo il codice di prima (riscrivo con una sintassi analoga):

```tsx

const [count, setCount] = useState(0)

function increment() {
    setCount(count => count + 1)
}

function decrement() {
    setCount(count => count - 1)
}

useEffect(() =>{
    console.log(count)
}, [count])
```
***useEffect()*** serve a gestire quello che è il ciclo di vita di un componente. Per esempio se vogliamo eseguire qualcosa all'avvio  del componente, oppure se vogliamo eseguire qualcosa ogni volta che cambia una variabile, oppure se chiamo un Api all'avvio del componente e voglio riempire una variabile.

Struttura: il **primo** parametro è una callback che posso scrivere con un arrow function che va a fare quello che vogliamo (in questo caso specifico il console.log di count).

il **secondo** parametro (fondamentale) è un array di dipendenze. Lasciato vuota significa "esegui l'effect all'avvio del componente una sola volta". Posso legare quest'array di dipendenze anche ad una variabile o a più variabili e in questo caso l'effect verrà eseguito ogni volta che cambia il valore di quella variabile (nel nostro caso specifico l'effect verrà richiamato ogni volta che cambia il valore di count. Infatti se lascio le parentesi vuote si può osservare nella console che verrà stampato solo il valore iniziale di count -> 0 e basta. Non verrà più eseguito il conosole.log tutte le altre volte che cambia il valore di count).

**Problema** Portiamo al limite l'uso di useEffect() e risolviamo il problema con il prossimo Hook

Nell'esempio seguente vogliamo gestire: 'username', 'role' e 'theme' che sono 3 proprietà semplici (nello specifico 3 stringhe di cui il tema può essere o light o dark).

Abbiamo poi una funzione che va ad aggiornare lo username, una funzione che va ad aggiornare il ruolo e una funzione che fa il toggle del tema. Prenderemo i valori di username e role dal currentTarget.value

```tsx

type Theme = 'light' | 'dark'

const App: React.FC = () => {
    const [username, setUsername] = useState<string>('')
    const [role, setRole] = useState<string>('')
    const [theme, setTheme] = useState<Theme>('light')
    const player = {username, role}

    function inputUsernameHandler(e: React.ChangeEvent<HTMLInputElement>) {
        setUsername(e.currentTarget.value)
    }

    function selectRoleHandler(e: React.ChangeEvent<HTMLSelectElement>) {
        setRole(e.currentTarget.value)
    }

    function handleTheme() {
        setTheme((curr: Theme) => {
            return curr === 'light' ? 'dark' : 'light'
        })
    }

    useEffect(()=> {
        console.log(player)
    }, [player])

    return (
        <div>
            <h1> Use Effect Example </h1>
            <button onClick={handleTheme}>Toggle Theme </button>
            <p> Current theme: {theme} </p>
            </hr>
            <p> Username: {username} - role: {role} </p>
            <input type='text' value={username} onChange={inputUsernameHandler} />
            <select onChange={selectRoleHandler}> 
                <option value='magician'>Magician </option> 
                <option value='barbarina'>Barbarian </option> 
                <option value='alchemist'>Alchemist </option> 
            </select>
        </div>
    )
}

export default App
```

Il ***problema*** che si presenta è il seguente --> abbiamo una variabile nel componente: ***const player = {username, role}*** che è un oggetto che lega insieme username e role andando a creare una variabile player.

Questa variabile è nel componente e questo è un problema, perchè per esempio ho lo useEffect che va a fare un colsole.log() di player ogni volta che la variabile player cambia.

se scrivessimo: 
```tsx
 useEffect(()=> {
        console.log(player.username)
    }, [player])
```

All'avvio ho la stampa di una stringa vuota (così come è stato inizializzato username). Quando cambio il valore di username nell'input o cambio il role nella select viene triggerato nuovamente lo useEffect questo perchè le due variabili sono legate tra loro.

Il problema è che l'effect viene triggerato anche quando cambio il tema. Questo succede perchè useState() rirenderizza il componente. Quando il componente viene rirenderizzato, viene ricreato const ***player = {username, role}*** e quindi viene ritriggerato l'effect.

Questo lo risolviamo con il prossimo Hook.

---

## ***useMemo()***

useMemo ha la sintassi uguale a quella di useEffect. Riceve un ***funzione in ingresso*** che deve ritornare qualcosa. Questo qualcosa che viene ritornato viene memorizzato o ***memoizzato***. Ovvero, se non cambieranno **username** o **role** non verrà rieseguita questa funzione. Nel nostro caso il valore di ritorno sarà un oggetto contenente username e role. Come useEffect anche ***useMemo vuole l'array di dipendenze*** per determinare se deve rieseguire la funzione in ingresso oppure no.

***PRIMA***
```tsx
const player = { username, role }
```

***DOPO***
```tsx
const player = useMemo(() => {
    return {username, role}
}, [username, role])
```

Questo è il tipico caso di utilizzo di useMemo. Ovviamente noi vogliamo evitare di ricomputare operazioni pesanti. Quindi quando rirenderizziamo il componente evitiamo di lanciare ad esempio quella funzione o quella variabile che va a fare quei calcoli matematici o calcoli pesanti. In questo caso è inutile perchè la rirenderizzazione dell'oggetto player che contiene solo due proprietà non è un'azione dispendiosa. Però il senso è questo.

---









