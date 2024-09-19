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
lineNumbers: true
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
- Redux, его проблемы и альтернативы
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

На уровне компонентов

#### React Hooks

- useState
- useReducer

::right::

```js
  function reducer(state, action) {
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
    // const [count, setCount] = useState(0)
    const [state, dispatch] = useReducer(reducer, { count: 0 });

    return (
      <div>
        <CounterDisplay count={state.count} />
        <CounterControls dispatch={dispatch} />
      </div>
    );
  }
```

---
layout: center
---

# Файловый редактор

<img src="/folders_example.png" width="500rem" />

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
  const [folders, _] = useState([]);
  const [userPermissions, _] = useState([]);

  return (
    <AppContext.Provider value={{ folders, userPermissions }}>
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

<span style="color: #FFD666">⚠️</span> React Context - механизм инъекции зависимостей, <b style='color: #FF859D;'>НЕ</b> управления состоянием

<br />

<v-click>

<h4 style='color: #FF859D;'>ПРОБЛЕМЫ</h4>

<br />

- Производительность
- Boilerplate
- Масштабирование
- Отладка
</v-click>

<br />

<v-click>

Подходит для небольшого количества редко изменяющихся данных

</v-click>

---
layout: TwoColumnCode
---

# Redux

::left::

<br />

- State
- Reducers
- Actions
- Slices

::right::

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
layout: TwoColumnCode
---

# Redux

::left::

<br />

- <b style="color: #C4CAFF;">Selectors</b>
- Cached Selectors
- Cached Selectors with dynamic keys

::right::

```js
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
layout: TwoColumnCode
---

# Redux

::left::

<br />

- Selectors
- <b style="color: #C4CAFF;">Cached Selectors</b>
- Cached Selectors with dynamic keys

::right::

```js
const getUserPermissions = (state) => state.userPermissions;
const getCanEditPermission = createSelector(
  [getUserPermissions],
  (userPermissions) => userPermissions.canEdit
)
```

---
layout: TwoColumnCode
---

# Redux

::left::

<br />

- Selectors
- Cached Selectors
- <b style="color: #C4CAFF;">Cached Selectors with dynamic keys</b>

::right::

```js
const getUserPermissions = (state) => state.userPermissions;
const getUserPermissionsByFolderId = 
  (state, folderId) => state.userPermissionsByFolderId[folderId];

const getCanEditPermission = createCachedSelector(
  getUserPermissionsByFolderId,
  (userPermissionsByFolderId) => userPermissionsByFolderId.canEdit
)(
  (_, folderId) => folderId
)
```

---
layout: two-cols-header
---

# Redux

::left::

```js
const getCanEditProject = createCachedSelector(
  getUserPermissions,
  getProjectPermissionsById,
  getUserRole,
  getProjectArchivedStatus,
  getProjectStatus,
  getProjectLockedStatus,
  getFolderRestrictionsById,
  getUserSubscriptionTier,
  (
    userPermissions, 
    projectPermissions, 
    userRole, 
    isArchived, 
    projectStatus, 
    isLocked,
    folderRestrictions, 
    subscriptionTier
  ) => {
    // ...
  }
)(
  (_, projectId, folderId) => `${projectId}-${folderId}`
);
```

::right::

<br />

<v-click>
  <img src="/tinkov.png" />
</v-click>

---
layout: TwoColumnCode
---

# Redux

::left::

<br />

- Side effects
- Middlewares

::right::

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
layout: TwoColumnCode
---

# Redux

::left::

```js
export const setFolders = (folders) => ({
  type: 'SET_FOLDERS',
  payload: folders,
});

export const setUserPermissions = (permissions) => ({
  type: 'SET_USER_PERMISSIONS',
  payload: permissions,
});

function rootReducer(state = initialState, action) {
  // ...
}
```

::right::

А что если добавить TypeScript?

---
layout: two-cols-header
zoom: 0.8
---

# Redux

::left::

```js
export const setFolders = (folders) => ({
  type: 'SET_FOLDERS',
  payload: folders,
});

export const setUserPermissions = (permissions) => ({
  type: 'SET_USER_PERMISSIONS',
  payload: permissions,
});

function rootReducer(state = initialState, action) {
  // ...
}
```

<br />

<img src="/mnogobukv.png" width="250rem" />

::right::

```js
interface RootState {
  folders: Folder[];
  userPermissions: UserPermissions;
}

const SET_FOLDERS = 'SET_FOLDERS';
const SET_USER_PERMISSIONS = 'SET_USER_PERMISSIONS';

interface SetFoldersAction extends Action<typeof SET_FOLDERS> {
  payload: Folder[];
}

interface SetUserPermissionsAction extends Action<typeof SET_USER_PERMISSIONS> {
  payload: UserPermissions;
}

type RootAction = SetFoldersAction | SetUserPermissionsAction;

export const setFolders = (folders: Folder[]): SetFoldersAction => ({
  type: SET_FOLDERS,
  payload: folders,
});

export const setUserPermissions = (permissions: UserPermissions): SetUserPermissionsAction => ({
  type: SET_USER_PERMISSIONS,
  payload: permissions,
});

function rootReducer(state = initialState, action: RootAction): RootState {
  // ...
}
```

---

# Redux

<br />

<h4 style='color: #FF859D;'>ПРОБЛЕМЫ</h4>

<br />

- Производительность
- Boilerplate (action creators, reducers, cached selectors)
- Сложность работы с асинхронными вызовами
- Типизация
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
| mobx       | 1.47m    |
| xstate     | 1.47m    |
| jotai      | 1.05m    |

---

# Zustand | Jotai | Recoil

<br />

- <b style="color: #C4CAFF;">\[Zustand]</b> Глобально доступные хранилища <span style="color: #6EF0B3;">-></span> не требует провайдеров

- <b style="color: #C4CAFF;">\[Jotai | Recoil]</b> Атомизация <span style="color: #6EF0B3;">-></span> избегают необходимости оптимизаций с селекторами

- Минималистичность, связанность, меньше бойлерплейта

- Простые асинхронные обновления состояния

<br />

<h3><i class="fab fa-twitter" style="color: #1DA1F2;" /> dai_shi</h3>

---

# XState

<br />

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
layout: TwoColumnCode
---

# MobX - Store providers

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

<br />

- глобальный объект
- занимает память
- тестируемость

<br />

### <span style="color: #6EF0B3;">Решение</span>

Связать состояние с React-компонентами с помощью React.Context

::right::

```js
export const getStoreProvider = (StoreClass) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children }) => {
    const store = new StoreClass();

    return (
      <StoreContext.Provider value={store}>
        {children}
      </StoreContext.Provider>
    );
  };

  const useStore = () => {
    const store = useContext(StoreContext);

    if (store === null) { 
      throw new Error(`${StoreClass.name}} is not initialized.`); 
    }

    return store;
  };
  return { StoreProvider, useStore };
};
```

---
layout: TwoColumnCode
---

# MobX - Store providers

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

<br />

- глобальный объект
- занимает память
- тестируемость

<br />

### <span style="color: #6EF0B3;">Решение</span>

Связать состояние с React-компонентами с помощью React.Context

::right::

```js
class CounterStore {
  constructor() {
    makeAutoObservable(this);
  }

  // observable properties...

  // actions...
}

export const { 
  StoreProvider: CounterStoreProvider, 
  useStore: useCounterStore 
} = getStoreProvider(CounterStore);
```

---
layout: TwoColumnCode
---

# MobX - Store providers

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

<br />

- глобальный объект
- занимает память
- тестируемость

<br />

### <span style="color: #6EF0B3;">Решение</span>

Связать состояние с React-компонентами с помощью React.Context

::right::

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
layout: TwoColumnCode
---

# MobX - Store providers

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

<br />

- глобальный объект
- занимает память
- тестируемость

<br />

### <span style="color: #6EF0B3;">Решение</span>

Связать состояние с React-компонентами с помощью React.Context

::right::

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
layout: TwoColumnCode
---

# MobX - Зависимости

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

Изолированные хранилища имеют внешние зависимости

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить механизмы для внедрения зависимостей

- <b style="color: #C4CAFF;">из родительского компонента</b>
- из других хранилищ

::right::

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
layout: TwoColumnCode
---

# MobX - Зависимости

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

Изолированные хранилища имеют внешние зависимости

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить механизмы для внедрения зависимостей

- <b style="color: #C4CAFF;">из родительского компонента</b>
- из других хранилищ

::right::

```js
export const getStoreProvider = (StoreClass) => {
  // ...
  const StoreProvider = ({ children, ...props }) => {
    const store = new StoreClass();

    // init props
    const propsRef = useRef(props);
    store.setFromProps?.(props);

    // update props
    useLayoutEffect(() => {
      if (!isShallowEqual(propsRef.current, props)) {
        store.setFromProps?.(props);
      }
    }, [store, props]);

    return (
      <StoreContext.Provider value={store}>
        {children}
      </StoreContext.Provider>
    );
  };
  // ...
};
```
---
layout: TwoColumnCode
---

# MobX - Зависимости

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

Изолированные хранилища имеют внешние зависимости

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить механизмы для внедрения зависимостей

- <b style="color: #C4CAFF;">из родительского компонента</b>
- из других хранилищ

::right::

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
layout: TwoColumnCode
---

# MobX - Зависимости

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

Изолированные хранилища имеют внешние зависимости

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить механизмы для внедрения зависимостей

- из родительского компонента
- <b style="color: #C4CAFF;">из других хранилищ</b>

::right::

```js
export const getStoreProvider = (StoreClass, useDeps = () => []) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children, ...props }) => {
    const deps = useDeps();

    const store = useMemo(() => {
      return new StoreClass(...deps);
    }, deps);

    return (
      <StoreContext.Provider value={store}>
        {children}
      </StoreContext.Provider>
    );
  };

  // ...
};
```

---
layout: TwoColumnCode
---

# MobX - Зависимости

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

Изолированные хранилища имеют внешние зависимости

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить механизмы для внедрения зависимостей

- из родительского компонента
- <b style="color: #C4CAFF;">из других хранилищ</b>

::right::

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

export const { 
  StoreProvider: FolderStoreProvider,
  useStore: useFolderStore
} = getStoreProvider(
  FolderStore,
  () => [useAppStore()]
);
```

---
layout: TwoColumnCode
---

# MobX - Зависимости

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

Многие хранилища зависят от корневого

<br />

### <span style="color: #6EF0B3;">Решение</span>

Обернуть часто используемую зависимость

::right::

```js
export const getAppDependantStoreProvider = 
  (store) => getStoreProvider(store, () => [ useAppStore() ]);

export const { 
  StoreProvider: FolderStoreProvider,
  useStore: useFolderStore
} = getAppDependantStoreProvider(FolderStore);
```
---
layout: TwoColumnCode
---

# MobX - Подписки

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

При создании хранилища нужно заполнить его данными

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить в хранилище реакции и механизм подписки

::right::

```js
class FolderStore {
  // ...

  data = undefined;

  get folderId() {
    return this.appStore.selectedFolderId;
  }

  loadFolderData = async () => {
    this.data = await myApi.loadFolderData(this.folderId);
  };

  subscribe = () => reaction(() => [this.folderId], this.loadFolderData);
}
```

---
layout: TwoColumnCode
---

# MobX - Подписки

::left::

<br />

### <span style="color: #FF859D;">Проблема</span>

При создании хранилища нужно заполнить его данными

<br />

### <span style="color: #6EF0B3;">Решение</span>

Добавить в хранилище реакции и механизм подписки

::right::

```js
export const getStoreProvider = (StoreClass) => {
  const StoreContext = createContext(null);

  const StoreProvider = ({ children, ...props }) => {
    const store = new StoreClass();

    // ...

    useEffect(() => {
      return store.subscribe?.();
    }, [store]);

    return (
      <StoreContext.Provider value={store}>
        {children}
      </StoreContext.Provider>
    );
  };

  // ...
};
```

---

# Как MobX помогает решить проблемы Redux

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

- #### TypeScript

<v-click>

&nbsp;<span style="color: #6EF0B3;">-></span> Описываете только типы своих данных

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
  Максим @titovmx
  <span class="icons">
    <i class="fab fa-telegram" />
    <i class="fab fa-linkedin" />
    <i class="fab fa-github" />
    <i class="fab fa-twitter" />
  </span>
  <img src="/qr.png" width="128rem" style="margin-top: 1rem;" />
</h3>
