## Стартовый набор Redux

Частичный перевод официальной документации [Redux]()

# Общий обзор и понятия

1. ## Создаем проект

   [cra-template-redux](https://github.com/reduxjs/cra-template-redux)

   \*данный вариант шаблона включает предустановленные модули React-Redux и Redux Toolkit

### `npx create-react-app my-app --template redux`

2. ## Ставим расширения на Chrome:

   [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)

   [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)

3. ## Концепт Redux:

### state

истинный источник движущей силы приложения;

### view

описание интерфейса на основе текущего состояния (state);

### actions

события в приложении на основе пользовательских действий, которые вызывают изменение состояния.

### Пример движения данных:

state описывает состояние приложения в определенное время -> view отображает state -> когда происходит action (например нажатие на кнопку) - state обновляется в зависимости от того, что произошло -> view обновляет отображение на основе нового state

![data flow](./one-way-data-flow.png 'flow')

## Основная идея Redux

Единое централизованное место хранения глобального состояния в приложении и контроль (предсказуемость) поведения кода и шаблонов.

## Immutability означает неизменность

Объекты и массивы JavaScript изменяемы (mutable) по умолчанию.
Для придания значениям неизменности необходимо создавать копии объектов и затем модифицировать эти копии.

Для этого можно использовать "спред-операторы", а также методы массива, которые возвращают новые копии массива вместо изменения исходного массива:

```
const obj = {
  a: {
    c: 3
  },
  b: 2
}

const obj2 = {
  // копируем obj
  ...obj,
  // перезаписываем a
  a: {
    // копируем obj.a
    ...obj.a,
    // перезаписываем c
    c: 42
  }
}

const arr = ['a', 'b']
// создаем новую копию arr и добавляем в конец "c"
const arr2 = arr.concat('c')

// либо делаем копию оригинального arr:
const arr3 = arr.slice()
// и мутируем копию:
arr3.push('c')
```

В Redux ожидается, что все обновления state иммутабельны.

## Actions

Просто объект JS, который имеет поле `type`. О нем можно думать, как о событии, которое описывает, что что-то произошло в приложении.

Поле `type` - это всегда строка, которая получает путь, где описан action, например "todos/todoAdded", где todos - это функция или категория, которой принадлежит действие, а todoAdded - непосредственно действие.

Объект может принимать и другую информацию о действии. По соглашению дополнительная информация вводится в поле `payload`:

```
const addTodoAction = {
  type: 'todos/todoAdded',
  payload: 'Buy milk'
}
```

Как правило, action делается в виде функции, которая создает и возвращает объект action:

```
const addTodo = text => {
  return {
    type: 'todos/todoAdded',
    payload: text
  }
}
```

## Reducers

Редьюсер - это функция, которая получает текущий state и объект action, где описывается, как изменить state (если необходимо) и вернуть новый `(state, action) => newState`. Короче: редьюсер - это обработчик событий в зависимости от типа полученного действия.

### Правила редьюсеров:

- всегда рассчитывают новое состояние на основе старого state и action;
- не модифицируют текущий sate, а только копию (иммутабельность);
- не выполняют асинхронную логику, рандомные рассчеты или вызовы посторонних эффектов.

### Логика редьюсера:

Проверка состояния объекта action

? создать копию state, внести в нее новые значения и вернуть

: вернуть текущий state без изменений.

```
const initialState = { value: 0 }

function counterReducer(state = initialState, action) {
  // проверяем состояние
  if (action.type === 'counter/increment') {
    // если да, делаем копию `state`
    return {
      ...state,
      // обновляем копию с новыми значениями
      value: state.value + 1
    }
  }
  // если нет, возвращаем все без изменений
  return state
}
```

Можно провести параллель в работе Redux reducer и Array.reduce(). Разница у них в том, что Array.reduce() происходит один раз, а Redux reducer выполняется постоянно.

## Store

Текущее состояние приложения хранится в ОБЪЕКТЕ `store`

`store` создается путем присвоения редьюсера и имеет метод `getState`, который возвращает текущее значение стейта.

`const store = redux.createStore(reducer)`

### Dispatch

Метод, который обновляет state и передает action в store:

`store.dispatch(action)`

Можно воспринимать dispatch, как триггер события.

### Selectors

Функции, извлекающие фрагменты информации из store state:

```
const selectCounterValue = state => state.value

const currentValue = selectCounterValue(store.getState())
console.log(currentValue)
// 2
```

# Структура приложения

/src

- index.js: стартовая точка приложения
- App.js: реакт компонент высшего уровня
- /app
  - store.js: хранилище state
- /features
  - /counter
    - Counter.js: UI
    - counterSlice.js: логика

[Детальный разбор + примеры работы с консолью](https://redux-docs.netlify.app/tutorials/essentials/part-2-app-structure)

## Создаем Redux Store

```
import { configureStore } from '@reduxjs/toolkit'
import counterReducer from '../features/counter/counterSlice'

export default configureStore({
  reducer: {
    counter: counterReducer
  }
})
```

Когда мы передаем такой объект, как `{counter: counterReducer}`, это говорит о том, что мы хотим иметь раздел `state.counter` нашего объекта состояния Redux и что мы хотим, чтобы функция `counterReducer` отвечала за решение, нужно ли и как обновлять `state.counter` каждый раз, когда отправляется действие (`dispatch`). Сложно понять, откуда он (`counterReducer`) появляется, но на этом этапе достаточно просто знать, что если делается импорт из `createSlice`, то нам помогает [Redux Toolkit](https://redux-toolkit.js.org/introduction/quick-start) (судя по всему благодаря плагину "middleware"). Так что можно описать объект как угодно:

```
import any from '../features/counter/counterSlice'
...
{any: anyReducer}
```

будет работать.

Важно просто указать, что здесь есть редьюсер:

```
export default configureStore({
  reducer: () => ({}),
})
```

## Создаем Slice Reducers и Actions (Слайсим проект)

Slice - это набор логики и событий для одной "фичи". Мы как-бы слайсим (разделяем) корневой state на несколько частей:

```
import { configureStore } from '@reduxjs/toolkit'
import usersReducer from '../features/users/usersSlice'
import postsReducer from '../features/posts/postsSlice'
import commentsReducer from '../features/comments/commentsSlice'

export default configureStore({
  reducer: {
    users: usersReducer,
    posts: postsReducer,
    comments: commentsReducer
  }
})
```

### Пример содержимого slice:

```
import { createSlice } from '@reduxjs/toolkit'

export const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0
  },
  reducers: {
    increment: state => {
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  }
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions

export default counterSlice.reducer
```

Объекты событий в Redux обязаны содержать поле type, но мы его не указываем, так как за нас это делает Redux Toolkit в функции `createSlice`.

От нас требуется: указать имя слайса -> написать объект, содержащий reducer. Далее toolkit сам сгенерит соответствующий action. Кстати `action` писать тоже не нужно, если в объекте не планируются мутации (любые изменения).

Строка `name` - это первая часть каждого `type`, а имя каждой ф-ции редьюсера - вторая:

```
name: 'counter'
...
reducers: {
    increment: state => {
      state.value += 1
    }
...
//Сгенерирует {type: "counter/increment"}
```

Также нам нужен "отправной пункт" (initial state value for the reducer). В примере для этого используется объект `initialState`.

## React Toolkit помогает с иммутабельностью

Внутри React Toolkit есть библиотека "Immer", использующая инструмент JS Proxy для обертки данных, чтобы легче было сделать копию для внесения изменений. Это помогает упростить код:

```
function handwrittenReducer(state, action) {
  return {
    ...state,
    first: {
      ...state.first,
      second: {
        ...state.first.second,
        [action.someId]: {
          ...state.first.second[action.someId],
          fourth: action.someValue
        }
      }
    }
  }
}
```

в такой

```
function reducerWithImmer(state, action) {
  state.first.second[action.someId].fourth = action.someValue
}
```

!!!НО!!!
Такой подход можно использовать только в пределах функций `createSlice` и `createReducer`, иначе Immer не сработает, `state` мутирует и будет габелла с багами.

Имея это ввиду, в примере

```
incrementByAmount: (state, action) => {
      state.value += action.payload
    }
```

редьюсер должен знать, что нужно изменить, поэтому в объект добавляюся аргументы `state` и `action`. В этом примере мы знаем, что число будет записано в текстовом поле `action.payload`, поэтому добавляем его в `state.value`.

[Паттерны](https://redux-docs.netlify.app/recipes/structuring-reducers/immutable-update-patterns) и [гайд](https://redux-docs.netlify.app/recipes/structuring-reducers/immutable-update-patterns) по иммутабельности.

## Пишем асинхронную логику с Thunks

`thunk` - это специальная функция Redux, которая может содержать асинхронную логику.
Пишется с использованием двух функций:

- внутренняя `thunk` функция принимает аргументы `dispatch` и `getState`;
- внешняя создает и возвращает саму `thunk` функцию.

```
export const incrementAsync = amount => dispatch => {
  setTimeout(() => {
    dispatch(incrementByAmount(amount))
  }, 1000)
}
```

```
// внешняя вункция "thunk creator"
const fetchUserById = userId => {
  // внутренняя "thunk function"
  return async (dispatch, getState) => {
    try {
      //создание async вызова
      const user = await userAPI.fetchById(userId)
      // передача (dispatch) события (action) после получения ответа
      dispatch(userLoaded(user))
    } catch (err) {
      // действие, если что-то пошло не так
    }
  }
}
```

Для `thunks` требуется свой `middleware`, но Toolkit `configureStore` уже его содержит.

Если нужно создать AJAX вызов для получения данных сервера, можно передать этот вызов в `thunk`.

[Дока по thunk](https://github.com/reduxjs/redux-thunk)

## Создание компонента

В шаблоне он в этом месте `features/counter/Counter.js`

Импортируем хуки:

```
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
```

\*Уточнение: Redux имеет [кастомные](https://react-redux.js.org/api/hooks) хуки.

### Читаем данные через useSelector

state из слайса (./counterSlice.js) экспортируется

```
export const selectCount = (state) => state.counter.value
```

и используется инлайново там, где нам нужно (./Counter.js)

```
const count = useSelector(selectCount);
```

Если есть доступ к store, то код выглядит вот так:

```
const count = selectCount(store.getState())
```

### Передаем данные через useDispatch

Так как доступа к `store` нету - используем кастомный хук

```
import { useSelector, useDispatch } from 'react-redux';
...
const dispatch = useDispatch();
...
<button
    className={styles.button}
    aria-label="Increment value"
    onClick={() => dispatch(increment())}
>
...
```

## Компонент State

Нет необходимости в передаче каждого `state` в `store`, если стейт используется только в одном месте.

В шаблоне (features/counter/Counter.js)

```
...
<div className={styles.row}>
        <input
          className={styles.textbox}
          aria-label="Set increment amount"
          value={incrementAmount}
          onChange={e => setIncrementAmount(e.target.value)}
        />
        <button
          className={styles.button}
          onClick={() =>
            dispatch(incrementByAmount(Number(incrementAmount) || 0))
          }
        >
          Add Amount
        </button>
        <button
          className={styles.asyncButton}
          onClick={() => dispatch(incrementAsync(Number(incrementAmount) || 0))}
        >
          Add Async
        </button>
      </div>
```

из всего приложения `input` используется только здесь, поэтому нет преимуществ при сохранении его в глобальный `store` и можно использовать хук `useState` для обработки.

Наш глобальный `state` должен храниться в `Redux store`, а локальный лучше оставлять в компоненте.

### Чеклист по отправке в store:

- Остальные части приложения заботятся об этих данных?
- Нужно иметь возможность создавать дополнительные производные данные на основе этих исходных данных?
- Используются ли одни и те же данные для управления несколькими компонентами?
- Есть ли ценность в возможности восстановить это состояние до заданного момента времени (например, отладка путешествия во времени)?
- Кэшировать данные (т. Е. Использовать то, что находится в состоянии, если оно уже есть, вместо повторного запроса)?
- Нужно, чтобы эти данные были согласованными при горячей перезагрузке компонентов пользовательского интерфейса (которые могут потерять свое внутреннее состояние при замене)?

## Передача Store

```
import { Provider } from 'react-redux'

<Provider store={store}>
    <App />
  </Provider>
```
