# **REACT HOOKS**

## ***useState***

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

## ***useEffect***

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

## ***useMemo***

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

Questo è il tipico caso di utilizzo di useMemo. Ovviamente noi vogliamo evitare di ricomputare operazioni pesanti. Quindi quando rirenderizziamo il componente evitiamo di lanciare ad esempio quella funzione che va a fare quei calcoli matematici o calcoli pesanti. In questo caso è inutile perchè la rirenderizzazione dell'oggetto player che contiene solo due proprietà non è un'azione dispendiosa. Però il senso è questo.

Come esiste useMemo che ci serve per salvare dei valori, possiamo usare useCallback che ci serve per salvare delle funzioni.

---
## ***useCallback***

Riprendiamo il codice precedente con qualche modifica.

Ci appoggeremo anche ad un altro componente chiamato **SomeChild** che è un componente generico.

Allora abbiamo:

- una variabile count inizializzata al valore 60 che aggiorniamo e memorizziamo con ***useState***
- abbiamo poi similmente un variabile per il toggle del tema
- poi abbiamo una funzione **showCount** (per il momento non facciamo caso a useCallback) che fa un alert di count
- poi abbiamo una funzione **expensiveCount** che è particolarmente dispendiosa perchè fa il quadrato di count e utilizziamo **useMemo** perchè appunto è un'elaborazione dispendiosa quindi è bene che non venga ritriggerata ogni volta che il componente viene renderizzato.
- poi abbiamo la funzione di **trigger** che va a richiamare la funzione **expensiveCount** e che va ad aggiornare il count con il nuovo valore
- abbiamo poi la funzione di **toggle** del tema

Ora che cosa succede; Se noi passiamo la funzione **showCount** a tanti componenti figli, come può capitare, ogni volta che andiamo a rirenderizzare il componente padre, andiamo a rirenderizzare anche tutti i componenti figli anche se magari count non è cambiato e quindi questa funzione è inutile da rigenerare. 

Allora ecco che possiamo usare ***useCallback***.

Il funzionamento è esattamente lo stesso di ***useMemo*** con la differenza che invece di salvare un valore stiamo salvando una funzione. Salvare una funzione significa: ogni volta che noi rigeneriamo il componente, anche se la funzione è sempre la stessa, è comunque diversa da quella precedente perchè le funzioni sono oggetti in javascript.

```tsx
import React, {useCallback, useMemo, useState} from 'react'
import './App.css'
import SomeChild from './SomeChild'

const App: React.FC = () => {

type Theme = 'light' | 'dark'

const [count, setCount] = useState(60)
const [theme, setTheme] = useState<Theme>('light')

const showCount = useCallback(() => {
    alert(`Count: ${count}`)
}, [count])

const expensiveCount = useMemo(() => {
    return count ** 2
}, [count])

const trigger = () => {
    const newCount = expensiveCount
    setCount(newCount)
}

function handleTheme() {
    setTheme((curr:Theme) => {
        return curr === 'light' ? 'dark' : 'light'
    })
}

console.log('rendering App')

return (
    <div>
        <h1> Use Callback Example </h1>
        <button onClick={handleTheme}> Toggle Theme </button>
        <p>Current theme: {theme} </p>
        </hr>
        <button onClick={trigger}> Expensive Count </button>
        <SomeChild showCount={showCount} />
    </div>
)

export default App
}
```

---

## ***useContext***

Un problema che capita spesso con i componenti è quello di dover condividere delle informazioni con altri componenti che sono dislocati in punti diversi dell'albero dei componenti. **L'albero dei componenti** -> la radice è **App** e tutti gli altri componenti man mano sono i rami di quest'albero e ovviamente si possono trovare su livelli diversi. Quindi noi passiamo da App a tutti i figli le informazioni, i figli passano le informazioni ai figli dei figli e così via. Quando poi aggiorniamo l'informazione in App dobbiamo poi aggiornarla anche in tutti gli altri componenti figli ecc.. Diventa impossibile ad un certo punto.

Allora possiamo utilizzare i **context** che sono delle Api di React che ci permettono di condividere uno stato con tutti i componenti dell'applicazione.

Vediamo un esempio:

***App.tsx***

```tsx
import React, {createContext} from 'react'
import './App.css'
import Number from './Numeber'

export const AppContext ? createContext(0)

const App: React.FC = () => {
    return (
        <AppContext.Provider value={42}>
            <h1> Use Context Example </h1>
            <Number />
            </AppContext.Provider>
    )
}

export default App
```

Nell'esempio sopra abbiamo:

- creazione del context usando **createContext()** (con valore di partenza 0 nel nostro caso ma può essere un oggetto, un array ecc). Gli diamo il nome AppContext e lo esportiamo perchè ci servirà negli altri componenti.
- Nel return abbiamo un **.Provider** Se wrappiamo il componente **App** con questo Provider, tutti i componenti figli di questo componente avranno accesso al ***context***. Può essere in App ma può può essere anche in altri punti dell'applicazione. Dipende dalla complessità. Nel nostro caso vogliamo condivere l'informazione -> **value={42}** con **<Number />**.  Immaginando che <Number /> sia figlio del figlio del figlio ..... ecc avremmo dovuto passare l'iformazione 1000 volte. 

***Number.tsx***

```tsx
import React, { useContext } from 'react'
import { AppContext } from './App'

const Number: React.FC = () => {
    const number = useContext(AppContext)

    return (
        <div>
        <p> Number: {number} </p>
        </div>
    )
}

export default Number
```
- Per leggere il valore dal context all'interno della variabile **number** utilizziamo **useContext** a cui passiamo il context **(AppContext)**. Quest' Hook ci restituisce il valore , in questo caso è il number e lo andiamo a stampare.

---

## ***useReducer***

Questo Hook serve a gestire lo stato di un componente però quando si avvia verso una determinata complessità. Non abbiamo quindi una gestione degli stati troppo semplice e quindi preferiamo un approccio stile Redux che tra l'altro ci permette di avere un codice "Redux ready"

Vediamo un breve esempio di utilizzo:

```tsx
import React, { useReducer } from 'react'
import './App.css'

type CounterAction = { type: 'inc'} | {type: 'dec'}

function counterReducer(state: number, action: CounterAction) {
    switch (action.type) {
        case 'inc':
            return state + 1
        case 'dec': 
            return state -1 
        default:
            throw new Error()
    }
}

const App: React.FC = () => {
    const [state, dispatch] = useReducer(counterReducer, 0)

    return (
        <div>
            <h1> Use Reducer Example </h1>
            
            Count: {state}
            <button onClick={() => dispatch({ type : 'inc' })}> Inc </button>
            <button onClick={() => dispatch({ type : 'dec' })}> Dec </button>
        </div>
    )
}

export default App
```

- ***useReducer*** ci restituisce uno state e un dispatch. Con useReducer non possiamo modificare direttamente lo stato ma prendendo spunto da redux "dispatchamo" (chiamiamo) un'azione e l'azione arriva in una funzione che si chiama reducer (che è una funzione pura, concetto ripreso dalla programmazione funzionale) e sostanzialmente in base al type della action viene aggiornato lo stato. Quindi chi è che aggiorna lo stato? E' il ***reducer***. Le ***actions*** sono dei semplici oggetti javascript che hanno almeno una proprietà: il **type** che identificano il tipo dell'azione: incremento, decremento, aggiungi elemento, rimuovi elemento.... Nel nostro caso abbiamo due action possibili: incremento e decremento perchè stiamo creando un counter.
- Allora al click dei pulsanti anzichè chiamare dei setCount e quant'altro abbiamo il dispatch di un oggetto. Quest'oggetto ha proprio un type. Può avere anche altre proprietà, per esempio: se voglio eliminare un elemento da una lista, oltre al type elimina elemento, passiamo anche quello che viene chiamato ***payload*** cioè l'informazione utile affinche l'azione possa essere eseguita (es: l'ID dell'elemento che voglio eliminare)

N.B. Il nostro count (riga 327) coincide con lo state --> quindi lo stato che ci restituisce useReducer è il valore della variabile che stiamo gestendo.

Il ***reducer*** (riga 309) è una semplice funzione che riceve 2 parametri. Il primo è il valore dello stato attuale (nel nostro caso il valore del number) mentre il secondo parametro è la action che è { type : 'inc} (riga 328) 

Il secondo valore dello ***useReducer*** (riga 321) è il valore di partenza dello state

---

## ***useRef***

Esistono 2 casi di utilizzo.

***Primo caso***: Quando voglio tenere il riferimento di qualcosa per esempio il click su un pulsante (quante volte è stato cliccato) o anche per tenere il tempo di riferimento di esecuzione di un'operazione.

Vediamo subito un esempio:

```tsx
import React, { useRef } from 'react'
import './App.css'

const App: React.FC = () => {
    const countRef = useRef(0)

    const handle = () => {
        countRef.current++
        console.log(`Clicked ${countRef.current} times`)
    }

    console.log('rendering App')

    return (
        <div>
            <h1> Use Ref Example </h1>
            <button onClick = {handle}> Click me! </button>
        </div>
    )
}

export default App
```

**useRef** riceve un valore iniziale come useState con la differenza che questo countRef restituisce un oggetto e il valore corrente (riga 364) quindi lo 0 è dentro current.

La bellezza di useRef è che non causa il rirendering del componente. Inoltre il current dopo l'operazione di incremento è gia aggiornato al valore corretto. Inoltre un'altra differenza con useState è che useRef è ***sincrono***


***Secondo Caso***: Quando useRef viene utilizzato per prendere elementi HTML

Esempio:

```tsx
import React, { useRef, useEffect } from 'react'
import './App.css'

const App: React.FC = () => {
    const inputRef = useRef<HTMLInputElement>(null)

    useEffect(() => {
        inputRef?.current?.focus()
    }, [])

    return (
        <div>
            <h1> Use Ref Example </h1>
            <input
                ref={inputRef}
                type='text'
                placeholder='Inserisci email'
            />
        </div>
    )
}
export default App
```

Nell'esempio sopra, voglio triggerare il focus su quest'input all'avvio del componente. Che cosa possiamo fare:

- useRef su HTMLInputElement e gli passo come valore iniziale null. Lo utilizzo per valorizzare la ref dell'input con inputRef.
- useEffect per avviare il focus all'avvio del componente

