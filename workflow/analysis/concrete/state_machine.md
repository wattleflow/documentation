# `concrete/state_machine.py` — StateMachine

**Sloj:** 2 (radna osnova frameworka)
**Sučelje (core):** `IStateMachine`
**Datoteka:** `src/wattleflow/concrete/state_machine.py` (72 linije)
**Dijagram:** [`diagrams/state_machine-class.puml`](diagrams/state_machine-class.puml)
**Primjer:** [`state_machine.ipynb`](state_machine.ipynb)
**Verzija dokumenta:** 0.1 (2026-05-30)

---

## 1. Zašto?

### 1.1 Poslovna motivacija

Komponente frameworka (konekcije, driveri, procesori, blackboard) prolaze kroz **definirane životne cikluse**. Lifecycle se ne smije voditi ad-hoc booleanima (`is_connected`, `is_loaded`, `is_running`) jer:

- Booleani ne hvataju zabranjene tranzicije (npr. `disconnect()` u stanju `FAILED`)
- Kombinacije više booleana stvaraju nemoguća stanja koja se moraju ručno detektirati
- Nema centralnog mjesta gdje je vidljivo *koje akcije su dopuštene u kojem stanju*

Posljedica: bugovi u rubovima i loš audit trag.

### 1.2 Što rješava

`StateMachine` daje **jednu eksplicitnu tablicu** `(state, action) → state` koja:

- Garantira da nelegalne tranzicije bacaju iznimku
- Omogućuje upit *prije* tranzicije (`can(action)`)
- Čini lifecycle vidljivim u kodu (jedan dict, jedan pogled)

---

## 2. Što? (zahtjevi)

### 2.1 Funkcionalni

| ID | Zahtjev |
|---|---|
| F-SM-1 | Komponenta drži trenutno stanje tipa `State` (Enum) |
| F-SM-2 | Komponenta prima statičku tablicu dopuštenih tranzicija pri inicijalizaciji |
| F-SM-3 | `can(action)` vraća `True/False` bez efekta na stanje |
| F-SM-4 | `apply(action)` mijenja stanje ako je tranzicija dopuštena, inače baca `ValueError` |
| F-SM-5 | Tipovi `State` i `Action` su generički (`TypeVar` ograničen na `Enum`) — type checker hvata neuparene Enume |
| F-SM-6 | Opcionalno ime instance vidljivo u `__repr__` |

### 2.2 Nefunkcionalni

| ID | Zahtjev |
|---|---|
| N-SM-1 | Bez vanjskih ovisnosti (samo stdlib + `wattleflow.core`) |
| N-SM-2 | Memorija: `__slots__` (`_state`, `_transitions`) — bez `__dict__` |
| N-SM-3 | Trošak `can()`/`apply()` je O(1) (`dict` lookup) |
| N-SM-4 | Tranzicijska tablica je immutable po dogovoru — nije thread-safe za izmjene tablice tijekom rada |

---

## 3. Kako? (dizajn)

### 3.1 Klasa

```python
class StateMachine(IStateMachine, Generic[State, Action]):
    __slots__ = ("_state", "_transitions")

    def __init__(self, transitions, initial, name=None) -> None: ...

    @property
    def state(self) -> State: ...
    def can(self, action: Action) -> bool: ...
    def apply(self, action: Action) -> None: ...
```

- Nasljeđuje `IStateMachine` iz `wattleflow.core.behavioral`
- Generic: `StateMachine[ConnectionState, ConnectionAction]` tipizira instancu na specifičan par enuma
- `transitions: Mapping[Tuple[State, Action], State]` — ključ je par `(trenutno_stanje, akcija)`, vrijednost je novo stanje

### 3.2 Pravilo upotrebe

`StateMachine` se koristi **kompozicijom**, ne nasljeđivanjem. Komponenta drži privatni `_fsm: StateMachine[...]` i prosljeđuje kroz njega svoje lifecycle akcije.

### 3.3 Tipičan obrazac

```python
class ConnectionAction(Enum):
    CONNECT = "connect"
    CONNECT_OK = "connect_ok"
    CONNECT_FAIL = "connect_fail"
    DISCONNECT = "disconnect"

class ConnectionState(Enum):
    NEW = "new"
    CONNECTING = "connecting"
    CONNECTED = "connected"
    FAILED = "failed"
    DISCONNECTED = "disconnected"

TRANSITIONS = {
    (ConnectionState.NEW,          ConnectionAction.CONNECT):      ConnectionState.CONNECTING,
    (ConnectionState.CONNECTING,   ConnectionAction.CONNECT_OK):   ConnectionState.CONNECTED,
    (ConnectionState.CONNECTING,   ConnectionAction.CONNECT_FAIL): ConnectionState.FAILED,
    (ConnectionState.CONNECTED,    ConnectionAction.DISCONNECT):   ConnectionState.DISCONNECTED,
}

fsm = StateMachine(TRANSITIONS, initial=ConnectionState.NEW, name="ConnectionFSM")
fsm.apply(ConnectionAction.CONNECT)        # NEW → CONNECTING
fsm.apply(ConnectionAction.CONNECT_OK)     # CONNECTING → CONNECTED
fsm.apply(ConnectionAction.DISCONNECT)     # CONNECTED → DISCONNECTED
```

### 3.4 Gdje se koristi u frameworku

| Modul | Lifecycle |
|---|---|
| `concrete/connection.py` | `ConnectionState` ↔ `ConnectionAction` (NEW → CONNECTED → DISCONNECTED / FAILED) |
| `concrete/driver.py` | `DriverState` ↔ `DriverAction` (PENDING → LOADING → LIVE → PAUSED → UNLOADED) |
| `concrete/processor.py` | `ProcessorState` ↔ `ProcessorAction` (IDLE → RUNNING → COMPLETED / FAILED) |
| `blackboards/small.py`, `blackboards/large.py` | blackboard lifecycle (IDLE → READY → DIRTY → CLEARED) |

### 3.5 Greške i rubni slučajevi

- `apply(action)` u stanju gdje akcija nije dopuštena → `ValueError(f"{action} not allowed in state {self._state}")`. Pozivatelj treba uhvatiti i prevesti u domensku iznimku (`ConnectionException`, `DriverException`, ...).
- `can(action)` ne baca iznimku — siguran za polling.
- Tablica `transitions` se ne smije mijenjati nakon `__init__` (nije zaštićeno kodom, dogovor).

---

## 4. Primjer (Jupyter)

Praktični prolaz s vizualizacijom tranzicija nalazi se u [`state_machine.ipynb`](state_machine.ipynb).

---

## 5. UML

Class dijagram: [`diagrams/state_machine-class.puml`](diagrams/state_machine-class.puml).

---

## 6. Reference

- Sučelje: `wattleflow.core.behavioral.IStateMachine`
- TypeVars: `State`, `Action` (oba ograničena na `Enum`)
- Pattern: State (GoF) implementiran tablično, ne kroz `IState`/`IStateContext` — namjerno, jer tablica je preglednija od polimorfnih State klasa za jednostavne lifecycle-e

---

## 7. Otvoreno

- Razmotriti `__init_subclass__` ili helper za declarativnu deklaraciju tablice (sintaksni šećer, niži prioritet)
- Razmotriti immutable tablicu (`types.MappingProxyType`) za eksplicitnu zaštitu od slučajne izmjene
