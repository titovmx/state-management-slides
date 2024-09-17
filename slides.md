---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://images.unsplash.com/photo-1650909085203-9205d767fd3e?q=80&w=2970&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
# some information about your slides (markdown enabled)
title: Рецепты MobX
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
themeConfig:
  primary: '#959CF5'
fonts:
  sans: Roboto
  serif: Roboto Slab
  mono: Fira Code
---

# Рецепты MobX

Продвинутые практики для больших проектов

---

<style>
.photo {
  border-radius: 50%;
  border: 0.125rem solid #4040DC;
  width: 12rem;
  height: 12rem;
}

.logo {
  height: 2rem;
}

.container {
  display: flex;
  flex-direction: column;
  gap: 4rem;
}

.content {
  flex: 1;
  display: flex;
  justify-content: space-around;
}

.info {
   display: flex;
   flex-direction: column;
   gap: 0.5rem;
   justify-content: center;
   align-items: flex-start;
}
</style>

<div class='container'>
  <h1>О себе</h1>
  <div class='content'>
    <img src='/photo.jpg' class='photo' />
    <div class='info'>
      <h2>Максим Титов</h2>
      <h3>Senior Frontend Engineer</h3>
      <img src='/qase.png' class='logo' />
    </div>
  </div>
</div>

---

# Зачем это всё?

<br/>

- Зачем нам управление состоянием
- Что не так с Redux
- Какие альтернативы
- Что так с MobX
- Примеры использования MobX и продвинутые практики

---

# Управление состоянием

<br/>

#### <span style="color: #C4CAFF;">Состояние</span> - данные приложения, которые могут поменяться.

<br/>

<v-click>

#### <span style="color: #C4CAFF;">Управление</span> включает:

- хранение
- обновление
- оповещение об изменении (механизм подписок)

</v-click>

---

# Решения

- React
- Бесконечное число библиотек:
  - Redux
  - MobX
  - Zustand
  - Jotai
  - Recoil
  - XState
  - ...

---
layout: TwoColumnCode
---

# Управление состоянием в React

::left::

#### React Hooks

- useState
- useReducer

::right::

```js
  const initialState = { count: 0 };

  function reducer(state, aqction) {
    switch (action.type) {
      case 'increment':
        return { count: state.count + 1 };
      case 'decrement':
        return { count: state.count - 1 };
      default:
        throw new Error(`Unknown action ${action.type}`);
    }
  }

  function App() {
    const [state, dispatch] = useReducer(reducer, initialState);

    return (
      <div>
        <CounterDisplay count={state.count} />
        <CounterControls dispatch={dispatch} />
      </div>
    );
  }
```

---

# Управление состоянием в React

```jsx
function App() {
  const [folders, setFolders] = useState()
  const [userPermissions, setUserPermissions] = useState()

  return (
    <div>
      <Sidebar folders={folders} userPermissions={userPermissions} />
      <MainContent userPermissions={userPermissions} />
    </div>
  );
}

function Sidebar({folders, userPermissions}) {
  return {folders.map(folder => <Folder folder={folder} userPermissions={userPermissions} />)}
}

function Folder({folder, userPermissions}) {
  return (
    <div>
      <span>{folder.title}</span>
      <Actions userPermissions={userPermissions}/>
    </div>
  );
}
```

---

# Управление состоянием в React

#### <b style="color: #C4CAFF;">React Context</b> позволяет передавать значения во вложенные компоненты

<br />

```js
const AppContext = createContext();

function AppProvider({ children }) {
  const [folders, setFolders] = useState([]);
  const [userPermissions, setUserPermissions] = useState([]);

  return (
    <AppContext.Provider value={{ folders, userPermissions, setFolders, setUserPermissions }}>
      {children}
    </AppContext.Provider>
  );
}
```

---

# Управление состоянием в React

#### <b style="color: #C4CAFF;">React Context</b> позволяет передавать значения во вложенные компоненты

```js
function App() {
  return (
    <AppProvider>
      <div>
        <Sidebar />
        <MainContent />
      </div>
    </AppProvider>
  );
}

function Actions({ userPermissions }) {
  const { userPermissions } = useContext(AppContext);
  const { canEdit, canDelete } = userPermissions;

  return (
    <ul>
      {canEdit && <li>Edit</li>}
      {canDelete && <li>Delete</li>}
    </ul>
  );
}
```

---

# Управление состоянием в React

<br />

<span style="color: #FFD666">⚠️</span> React Context - механизм инъекции зависимостей, <b>НЕ</b> управления состоянием

<br />

<v-click>
<h4 style='color: #FF859D;'>ПРОБЛЕМЫ</h4>

- Производительность
- Boilerplate
- Масштабирование
- Отладка
</v-click>

---

# Redux

```js
export const setFolders = (folders) => ({
  type: 'SET_FOLDERS',
  payload: folders,
});

export const setUserPermissions = (permissions) => ({
  type: 'SET_USER_PERMISSIONS',
  payload: permissions,
});

const initialState = { folders: [], userPermissions: {} };

function rootReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_FOLDERS':
      return { ...state, folders: action.payload };
    case 'SET_USER_PERMISSIONS':
      return { ...state, userPermissions: action.payload };
    default:
      return state;
  }
}

const store = createStore(rootReducer);
```

---

# Redux

```js
function App() {
  return (
    <Provider store={store}>
      <div>
        <Sidebar />
        <MainContent />
      </div>
    </Provider>
  );
}

function Actions({ userPermissions }) {
  const userPermissions = useSelector((state) => state.userPermissions);
  const { canEdit, canDelete } = userPermissions;

  return (
    <ul>
      {canEdit && <li>Edit</li>}
      {canDelete && <li>Delete</li>}
    </ul>
  );
}
```

---

# Redux

```js
import thunk from 'redux-thunk';

const store = createStore(rootReducer, applyMiddleware(thunk));

const fetchFolders = () => {
  return async (dispatch) => {
    const response = await fetch('/api/folders');
    const data = await response.json();
    dispatch(setFolders(data));
  };
};

const useLoadFolders = () => {
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchFolders());
  }, [dispatch]);
};
```

---

# Redux

<br />

<h4 style='color: #FF859D;'>ПРОБЛЕМЫ</h4>

- Производительность
- Сложность работы с асинхронными вызовами
- Boilerplate (action creators, reducers, cached selectors)
- Иммутабельность

---

# Альтернативы Redux

<br />

### State of JavaScript 2020 Survey

<br />

#### <b style="color: #C4CAFF;">Usage</b>

| Redux | MobX | XState |
| ----- | ---- | ------ |
| 67%   | 13%  | 5%     |

<br />

#### <b style="color: #C4CAFF;">Satisfaction</b>

| Redux | MobX | XState |
| ----- | ---- | ------ |
| 67%   | 64%  | 87%    |

---

# Альтернативы Redux

<br />

#### <b style="color: #C4CAFF;">npm-загрузки (в неделю)</b>

| Библиотека | Загрузки |
| ---------- | -------- |
| redux      | 9.98m    |
| zustand    | 3.66m    |
| xstate     | 1.47m    |
| mobx       | 1.47m    |
| jotai      | 1.05m    |

---

# Zustand

```js
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

function CounterDisplay() {
  const count = useStore((state) => state.count);
  return <div>Count: {count}</div>;
}

function CounterControls() {
  const increment = useStore((state) => state.increment);
  const decrement = useStore((state) => state.decrement);

  return (
    <div>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

---

# Zustand

<br />

#### <b style="color: #6EF0B3;">ПРЕИМУЩЕСТВА</b>

- Не требует провайдеров
- Простой, меньше бойлерплейта
- Встроенная в селекторы мемоизация
- Простые асинхронные обновления состояния

---

# XState

```js
const counterMachine = createMachine(
  {
    id: 'counter',
    initial: 'active',
    context: {
      count: 0,
    },
    states: {
      active: {
        on: {
          INCREMENT: { actions: 'incrementCount' },
          DECREMENT: { actions: 'decrementCount' },
        },
      },
    },
  },
  {
    actions: {
      incrementCount: (context) => (context.count += 1),
      decrementCount: (context) => (context.count -= 1),
    },
  },
);
```

---

# XState

```js
function App() {
  const [state, send] = useMachine(counterMachine);

  return (
    <div>
      <CounterDisplay count={state.context.count} />
      <CounterControls dispatch={send} />
    </div>
  );
}

function CounterDisplay({ count }) {
  return <h1>{count}</h1>;
}

function CounterControls({ dispatch }) {
  return (
    <div>
      <button onClick={() => dispatch('INCREMENT')}>Increment</button>
      <button onClick={() => dispatch('DECREMENT')}>Decrement</button>
    </div>
  );
}
```

---

# XState

<br />

#### <b style="color: #6EF0B3;">ПРЕИМУЩЕСТВА</b>

- Машина состояний позволяет описывать сложные состояния и переходы между ними
- Простые асинхронные обновления
- Простая реализация undo/redo и time travel

---

# MobX

<br />

```js
// observe some state
const state = observable({ count: 0 });

// create reaction
autorun(() => console.log(state.count));

// some action
state.count++;
```

---

# MobX

```js
class CounterStore {
  constructor() {
    makeAutoObservable(this);
  }

  // observable
  count = 0;

  // action
  increment() {
    this.count += 1;
  }

  // action
  decrement() {
    this.count -= 1;
  }
}
```

---

# MobX

```js
const counterStore = new CounterStore();

const App = () => {
  return (
    <div>
      <CounterDisplay />
      <CounterControls />
    </div>
  );
};

const CounterDisplay = observer(() => {
  return <h1>{counterStore.count}</h1>;
});

const CounterControls = () => {
  return (
    <div>
      <button onClick={() => counterStore.increment()}>Increment</button>
      <button onClick={() => counterStore.decrement()}>Decrement</button>
    </div>
  );
};
```

---

# MobX

<br />

### <span style="color: #FF859D;">Проблема</span>

<br />

- глобальный объект, создаётся при загрузке файла
- тестируемость

<br />

<v-click>

### <span style="color: #6EF0B3;">Решение</span>

Связать состояние с React-компонентами с помощью React.Context
</v-click>

---

# MobX - Store providers

```js
export const getStoreProvider = (StoreClass) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children }) => {
    const store = new StoreClass();

    return <StoreContext.Provider value={store}>{children}</StoreContext.Provider>;
  };

  const useStore = () => {
    const store = useContext(StoreContext);

    if (store === null) {
      throw new Error(`${StoreClass.name}} is not initialized.`);
    }

    return store;
  };

  return {
    StoreProvider,
    useStore,
  };
};
```

---

# MobX - Store providers

```js
class CounterStore {
  constructor() {
    makeAutoObservable(this);
  }

  // observable properties...

  // actions...
}

export const { StoreProvider: CounterStoreProvider, useStore: useCounterStore } = getStoreProvider(CounterStore);
```

---

# MobX - Store providers

```js
const App = () => {
  return (
    <div>
      <CounterStoreProvider>
        <CounterDisplay />
        <CounterControls />
      </CounterStoreProvider>
    </div>
  );
};
```

---

# MobX - Store providers

```js
const CounterDisplay = observer(() => {
  const counterStore = useCounterStore();

  return <h1>{counterStore.count}</h1>;
});

const CounterControls = () => {
  const counterStore = useCounterStore();

  return (
    <div>
      <button onClick={() => counterStore.increment()}>Increment</button>
      <button onClick={() => counterStore.decrement()}>Decrement</button>
    </div>
  );
};
```

---

# MobX

<br />

### <span style="color: #FF859D;">Проблема</span>

Мы хотим иметь изолированное хранилище для части приложения, но оно имеет зависимости от корневого или других хранилищ

<br />

<v-click>

### <span style="color: #6EF0B3;">Решение</span>

Добавить механизмы для внедрения зависимостей

- setFromProps - из родительского компонента
- useDeps - из других хранилищ

</v-click>

---

# MobX - Dependencies

```js
const Folder = observer(() => {
  const appStore = useAppStore();

  return (
    <FolderProvider folderId={appStore.selectedFolderId}>
      <FolderView />
    </FolderProvider>
  );
});
```

---

# MobX - Dependencies

Вернёмся к нашему провайдеру...

```js
export const getStoreProvider = (StoreClass) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children, ...props }) => {
    const store = new StoreClass();

    // init props
    store.setFromProps?.(props);

    const propsRef = useRef(props);

    // update props
    useLayoutEffect(() => {
      if (!isShallowEqual(propsRef.current, props)) {
        store.setFromProps?.(props);
      }
    }, [store, props]);

    return <StoreContext.Provider value={store}>{children}</StoreContext.Provider>;
  };

  // ...
};
```

---

# MobX - Dependencies

```js
class FolderStore {
  constructor() {
    makeAutoObservable();
  }

  folderId = undefined;

  setFromProps = ({ folderId }) => {
    this.folderId = folderId;
  };
}
```

---

# MobX - Dependencies

```js
export const getStoreProvider = (StoreClass, useDeps = () => []) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children, ...props }) => {
    const deps = useDeps();

    const store = useMemo(() => {
      return new StoreClass(...deps);
    }, deps);

    return <StoreContext.Provider value={store}>{children}</StoreContext.Provider>;
  };

  // ...
};
```

---

# MobX - Dependencies

```js
class FolderStore {
  constructor(appStore) {
    this.appStore = appStore;

    makeAutoObservable(this);
  }

  get folderId() {
    return this.appStore.selectedFolderId;
  }
}

export const { StoreProvider: FolderStoreProvider, useStore: useFolderStore } = getStoreProvider(FolderStore, () => [
  useAppStore(),
]);
```

---

# MobX - Dependencies

```js
export const getAppDependantStoreProvider = (store) => getStoreProvider(store, () => [useAppStore()]);

export const { StoreProvider: FolderStoreProvider, useStore: useFolderStore } =
  getAppDependantStoreProvider(FolderStore);
```

---

# MobX - Subscriptions

<br />

### <span style="color: #FF859D;">Проблема</span>

Мы хотим иметь автоматические действия после инициализации хранилища, например, загрузить данные

<br />

<v-click>

### <span style="color: #6EF0B3;">Решение</span>

Добавить в хранилище реакции и механизм подписки

</v-click>

---

# MobX - Subscriptions

```js
class FolderStore {
  // ...

  data = undefined;

  get folderId() {
    return this.appStore.selectedFolderId;
  }

  loadFolders = async () => {
    this.data = await myApi.loadFolders(this.folderId);
  };

  subscribe = () => reaction(() => [this.folderId], this.loadFolders, { fireImmediately: true });
}
```

---

# MobX - Subscriptions

Вернёмся к нашему провайдеру...

```js
export const getStoreProvider = (StoreClass) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children, ...props }) => {
    const store = new StoreClass();

    // ...

    useEffect(() => {
      return store.subscribe?.();
    }, [store]);

    return <StoreContext.Provider value={store}>{children}</StoreContext.Provider>;
  };

  // ...
};
```

---

# MobX - Utility stores

<br />

### <span style="color: #FF859D;">Проблема</span>

Разные страницы могут иметь повторяющуюся функциональность: состояние и действия

Например, сортировка таблицы данных

<br />

<v-click>

### <span style="color: #6EF0B3;">Решение</span>

Использовать вспомогательные хранилища (utility stores)
</v-click>

---

# MobX - Utility stores

```js
export class OrderByStore {
  constructor() {
    makeAutoObservable(this);
  }

  // field key
  orderBy = undefined;
  // asc | desc | undefined
  direction = undefined;

  toggleOrder = (orderBy) => {
    if (!this.orderBy || this.orderBy !== orderBy) {
      this.orderBy = orderBy;
      this.direction = 'desc';
    } else if (this.orderBy === orderBy) {
      if (this.direction === OrderByDirection.DESC) {
        this.direction = 'asc';
      } else {
        this.orderBy = undefined;
        this.direction = undefined;
      }
    }
  };
}

export const { StoreProvider: OrderByStoreStoreProvider, useStore: useOrderByStore } = getStoreProvider(OrderByStore);
```

---

# MobX - Utility stores

```js
class DocumentsStore() {
  constructor() {
    makeAutoObservable(this)
  }

  // array of documents
  documents = undefined

  order = new OrderByStore()

  // ...
}

export const { StoreProvider: DocumentsStoreProvider, useStore: useDocumentsStore } =
  getStoreProvider(DocumentsStore);
```

---

# MobX - Utility stores

```js
export const DocumentTable = observer(() => {
  const store = useDocumentsStore();

  return (
    <OrderByStoreProvider>
      <table>
        <thead>
          <SortableHeader orderBy="title">Name</SortableHeader>
          <SortableHeader orderBy="modified_on">Modified On</SortableHeader>
          <SortableHeader orderBy="modified_by">Modified By</SortableHeader>
        </thead>

        <tbody>
          {store.documents.map((document) => (
            <DocumentRow key={document.id} document={document} />
          ))}
        </tbody>
      </table>
    </OrderByStoreProvider>
  );
});
```

---

# MobX - Utility stores

```js
export const SortableHeader = observer(({ orderBy }) => {
  const store = useOrderByStore();

  return (
    <th onClick={() => store.toggleOrder(orderBy)}>
      {children} <SortIcon active={orderBy === store.orderBy} direction={store.direction} />
    </th>
  );
});
```

---

# Как MobX помог решить проблемы Redux

<br />

- #### Производительность

<v-click>

&nbsp;<span style="color: #6EF0B3;">-></span> нет повторных рендеров и сложных селекторов

</v-click>

- #### Сложность работы с асинхронными вызовами

<v-click>

&nbsp;<span style="color: #6EF0B3;">-></span> никаких мидлвар

</v-click>

- #### Boilerplate (action creators, reducers, cached selectors)

<v-click>

&nbsp;<span style="color: #6EF0B3;">-></span> всё необходимое инкапуслировано внутри класса

</v-click>

- #### Иммутабельность

<v-click>

&nbsp;<span style="color: #6EF0B3;">-></span> Реактивность

</v-click>

---
layout: center
---

<style>
.contacts {
  margin: 3rem auto 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  width: 100%;
}

.icons {
  display: flex;
  gap: 1rem;
}
</style>

# Спасибо за внимание!

<h3 class="contacts">
  @titovmx
  <span class="icons">
    <i class="fab fa-telegram" />
    <i class="fab fa-linkedin" />
    <i class="fab fa-github" />
  </span>
</h3>