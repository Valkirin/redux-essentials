## Шпаргалка по Redux на русском

# Общий обзор и понятия

1. ## Создаем проект
   [cra-template-redux](https://github.com/reduxjs/cra-template-redux)

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

Объекты и массивы JavaScript, изменяемы (mutable) по умолчанию.
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

Объект может принимать и другую информацию о действии. По соглашению конвенции дополнительная информация вводится в поле `payload`:

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

Редьюсер - это функция, которая получает текущий state и объект action, где описывается, как изменить state (если необходимо) и вернуть новый `(state, action) => newState`. Короче редьюсер - это обработчик событий в зависимости от типа полученного действия.

### Правила редьюсеров:

- всегда рассчитывают новое состояние на основе старого state и action;
- не модифицируют текущий sate, а только копию (иммутабельность);
- не выполняют асинхронную логику, рандомные рассчеты или вызовы посторонних эффектов.

### Логика редьюсера:

Проверка состояния объекта action

? создать копию state, внести в нее новые значения, вернуть

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

### Dispatch

Метод `store`, который обновляет state и передает в action:

store.dispatch()

Можно воспринимать dispatch, как триггер события.

### Selectors

Функции, извлекающие фрагменты информации из store state:

```
const selectCounterValue = state => state.value

const currentValue = selectCounterValue(store.getState())
console.log(currentValue)
// 2
```
