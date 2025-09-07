

# 第十三章：使用 NgRx 进行保存、删除和更新

在上一章中，我们学习了 NgRx 的概念和功能。我们了解到状态管理的重要性，因为它为应用程序提供了一个单一的数据流源，并减少了组件的责任。我们还学习了 NgRx 的构建块，即操作、效果、reducer 和选择器。最后，我们使用 NgRx 在我们的应用程序中实现了获取和显示反英雄列表功能。

在本章中，我们将完成我们应用程序缺失的功能——通过仍然使用 NgRx 来保存、删除和更新数据。

在本章中，我们将涵盖以下主题：

+   使用 NgRx 无副作用地删除项目

+   使用 NgRx 通过副作用删除项目

+   使用 NgRx 添加具有副作用的项目

+   使用 NgRx 通过副作用更新项目

# 技术要求

代码完成版本的链接为 [`github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13/superheroes`](https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13/superheroes).

# 使用 NgRx 无副作用地删除项目

在本节中，我们将首先了解如何在 NgRx 中不使用副作用来删除项目。正如我们在上一章中学到的，副作用用于调用外部 API 以检索数据。这意味着在不使用效果的情况下，我们将通过派发一个操作来删除数据，该操作将根据派发的类型调用 reducer。本节将帮助我们了解在应用程序中使用效果时的流程和行为差异。

## 创建删除操作

第一步是创建删除功能的操作。在我们的项目中，在 `anti-hero/state/anti-hero.actions.ts` 文件中，我们将添加一个新的操作接口和一个新的删除函数。

让我们看看以下代码的实现：

```java
export enum AntiHeroActions {
  GET_ANTI_HERO_LIST = '[Anti-Hero] Get Anti-Hero list',
  SET_ANTI_HERO_LIST = '[Anti-Hero] Set Anti-Hero list',
  REMOVE_ANTI_HERO_STATE =
   '[Anti-Hero] Remove ALL Anti-Hero (STATE)',
}
export const removeAntiHeroState = createAction(
    AntiHeroActions.REMOVE_ANTI_HERO_STATE,
  props<{ antiHeroId: string }>()
);
```

在前面的代码示例中，我们可以看到我们添加了一个名为 `REMOVE_ANTI_HERO_STATE` 的新操作。我们还创建了一个具有新创建类型的操作，该类型具有一个接受反英雄 ID 的 `props` 参数。ID 是 reducer 识别我们应该从我们的存储中删除哪些数据所必需的。

## 创建删除 reducer

现在，让我们创建用于从我们的存储中删除数据的 reducer。我们首先需要考虑的是，如果我们的 reducer 能够使用提供的 ID 从数组中删除单个数据项，它会是什么样子。我们可以实现这一点的其中一种方法是通过使用 `filter()` 函数从数组中提取数据。

让我们在 `anti-hero/state/anti-hero.reducers.ts` 文件中添加以下代码：

```java
export const antiHeroReducer = createReducer(
  initialState,
  on(setAntiHeroList, (state, { antiHeroes }) => { return
    {...state, antiHeroes}}),
  on(removeAntiHeroState, (state, { antiHeroId }) => {
    return {...state, antiHeroes:
      state.antiHeroes.filter(data => data.id !=
                              antiHeroId)}
  }),
);
```

在前面的代码示例中，我们可以看到我们为我们的删除功能添加了一个新的 reducer。它接受来自`removeAntiHeroState`动作的反英雄 ID，并返回一个新的状态，其中已过滤掉具有给定 ID 的反英雄数据，并修改了`antiHeroes`值。如果 reducer 成功修改了`antiHeroes`状态值，任何订阅此状态变化的选择器将在组件中发出新值。

## 分发动作

我们需要执行的最后一个步骤是在我们的组件中分发动作。为了实现这一步骤，我们需要在点击每条反英雄数据的**删除**按钮时调用分发。

在`anti-hero/components/anti-hero-list.component.ts`文件中，我们添加了`emittethatch`，它根据用户点击的按钮传递选定的反英雄对象和`TableAction`。

让我们回顾一下我们为这个功能在以下文件中实现的代码：

anti-hero-list.component.ts

```java
// See full code on https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13 /
export class AntiHeroListComponent implements OnInit {
   // other code for component not displayed
  @Output() antiHero = new EventEmitter<{antiHero:   AntiHero, action :TableActions}>();
  selectAntiHero(antiHero: AntiHero, action: TableActions)  {
    this.antiHero.emit({antiHero, action});
 }
}
```

anti-hero-list.component.html

```java
// See full code on https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13
<button (click)="selectAntiHero(element, 1)" mat-raised-button color="warn">
           <mat-icon>delete</mat-icon> Delete
</button>
```

table-actions.enum.ts

```java
export enum TableActions {
  View,
   Delete
}
```

在前面的代码示例中，我们可以看到`1`代表**删除枚举**的值。

现在，我们需要在列表组件中分发`REMOVE_ANTI_HERO_STATE`动作，当**删除**按钮发出事件时。为了实现这部分，我们将在以下文件中添加以下代码：

list.component.ts

```java
  selectAntiHero(data: {antiHero: AntiHero, action:
    TableActions}) {
    switch(data.action) {
      case TableActions.Delete: {
        this.store.dispatch({type:
          AntiHeroActions. REMOVE_ANTI_HERO_STATE,
          antiHeroId: data.antiHero.id});
        return;
      }
      default: ""
    }
  }
```

list.component.html

```java
// See full code on https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13
<!-- Dumb component anti hero list -->
<app-anti-hero-list [antiHeroes]="antiHeroes" (antiHero)="selectAntiHero($event)" [headers]="headers"></app-anti-hero-list>
```

在前面的代码示例中，我们创建了一个函数，该函数检查由用户触发的`TableActions`值。如果`TableActions`具有删除值，我们将分发`REMOVE_ANTI_HERO_STATE`并传递将由我们创建的 reducer 使用的反英雄对象的 ID。

我们现在已成功使用 NgRx 实现了我们应用程序的删除功能，但在这个案例中，我们只删除了 UI 中的项目，并没有同步数据库中的更改。

在下一节中，我们将实现使用副作用来删除数据的方法。

# 使用 NgRx 通过副作用删除项目

在本节中，我们将通过在我们的状态中添加副作用来改进删除功能。我们当前的删除功能仅从存储中删除数据，但不会同步数据库中的更改。这意味着如果我们刷新我们的应用程序，我们已删除的数据将再次可用。

为了同步数据库中的更改，我们应该创建一个将调用删除 API 的效果。让我们看看以下章节中我们代码的逐步更改。

## 创建一个新的动作类型

我们需要做的第一步是创建一个新的动作类型。NgRx 中的效果将使用新的动作类型来删除功能。

我们将在`anti-hero/state/anti-hero.actions.ts`文件下的`AntiHeroActions`枚举中添加`REMOVE_ANTI_HERO_API`。

让我们看一下以下代码中添加的动作：

```java
export enum AntiHeroActions {
  GET_ANTI_HERO_LIST = '[Anti-Hero] Get Anti-Hero list',
  SET_ANTI_HERO_LIST = '[Anti-Hero] Set Anti-Hero list',
  REMOVE_ANTI_HERO_API =
    '[Anti-Hero] Remove Anti-Hero (API)',
  REMOVE_ANTI_HERO_STATE =
    '[Anti-Hero] Remove Anti-Hero (STATE)',
}
```

在前面的代码示例中，我们可以看到为我们的动作添加了一个新的动作类型。请注意，我们不需要为这个类型创建一个新的动作，因为一旦这个动作类型被派发，我们将调用一个副作用而不是一个动作。

## 创建删除副作用

我们接下来需要做的步骤是创建删除功能的副作用。在 `anti-hero/state/anti-hero.effect.ts` 文件中，我们将添加以下代码：

```java
 // See full code on https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13
 removeAntiHero$ = createEffect(() => {
    return this.actions$.pipe(
        ofType(AntiHeroActions.REMOVE_ANTI_HERO_API),
        mergeMap((data: { payload: string}) =>
          this.antiHeroService.deleteAntiHero(data.payload)
          .pipe(
            map(() => ({ type:
              AntiHeroActions.REMOVE_ANTI_HERO_STATE,
              antiHeroId: data.payload })),
            catchError(() => EMPTY)
          ))
        )
    }, {dispatch: true}
  );
```

在前面的代码示例中，我们可以看到我们为我们的删除动作创建了一个新的副作用；这个副作用的类型是 `REMOVE_ANTI_HERO_API`，它调用 `AntiHeroService` 中的 `deleteAntiHero()` 函数，根据传递的 ID 删除数据，一旦 API 调用成功。

该副作用将派发另一个动作，`REMOVE_ANTI_HERO_STATE`，这是我们之前创建的，它从存储中删除反英雄。这意味着我们从数据库中删除的数据也将从我们的 NgRx 存储中删除。

## 修改派发

这个功能的最后一步是在 `list.component.ts` 文件中修改派发的动作。在上一节中，我们直接在我们的组件中调用 `REMOVE_ANTI_HERO_STATE` 动作；我们将将其更改为 `REMOVE_ANTI_HERO_API`，因为我们现在应该调用副作用，这将调用 API，同时也会调用 `REMOVE_ANTI_HERO_STATE` 动作。

让我们看看以下代码示例：

```java
 selectAntiHero(data: {antiHero: AntiHero, action: TableActions}) {
    switch(data.action) {
      case TableActions.Delete: {
        this. store.dispatch({type:
          AntiHeroActions.REMOVE_ANTI_HERO_API,
          payload: data.antiHero.id});
        return;
      }
      default: ""
    }
  }
```

在前面的代码示例中，我们现在在我们的列表组件中派发副作用。这将首先调用 API，然后再更新我们的应用存储；我们存储和数据库中的更改是同步的。

在下一节中，我们将实现具有副作用的数据添加到我们的应用中。

# 使用 NgRx 添加具有副作用的项目

在本节中，我们将使用 NgRx 实现具有副作用的 *添加* 功能。步骤与我们实现删除功能的方式相似。我们将逐步创建构建块并在我们的组件中创建派发逻辑。

## 创建动作

我们需要做的第一步是创建我们添加功能所需的动作类型和动作。为了实现动作，我们可以考虑我们是如何创建删除功能的动作的。

概念是相同的。我们需要创建两种动作类型，这些是 `ADD_ANTI_HERO_API` 和 `ADD_ANTI_HERO_STATE`。第一种类型将由调用 API 的副作用使用，第二种类型将由修改状态的还原器使用，通过添加新创建的数据。

在创建了两种动作类型之后，我们还需要使用 `createAction()` 函数为 `ADD_ANTI_HERO_STATE` 类型创建一个动作。一旦 API 调用成功，副作用将派发这个动作。

让我们看看以下代码实现：

```java
 // See full code on https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13
export enum AntiHeroActions {
  GET_ANTI_HERO_LIST = '[Anti-Hero] Get Anti-Hero list',
  SET_ANTI_HERO_LIST = '[Anti-Hero] Set Anti-Hero list',
  ADD_ANTI_HERO_API = '[Anti-Hero] Add Anti-Hero (API',
  ADD_ANTI_HERO_STATE = '[Ant
    i-Hero] Add Anti-Hero (STATE)',
  REMOVE_ANTI_HERO_API =
    '[Anti-Hero] Remove Anti-Hero (API)',
  REMOVE_ANTI_HERO_STATE =
    '[Anti-Hero] Remove Anti-Hero (STATE)',
}
export const addAntiHeroState = createAction(
  AntiHeroActions.ADD_ANTI_HERO_STATE,
  props<{ antiHero: AntiHero }>()
)
```

在前面的代码示例中，我们可以看到我们在`AntiHeroActions`中添加了两个新的类型。我们还创建了一个新的动作，类型为`ADD_ANTI_HERO_STATE`，它接受一个`antiHero`属性，该属性将被推送到反英雄状态中的新条目。

## 创建效果

我们下一步需要做的是为*添加*功能创建效果。在`anti-hero/state/anti-hero.effect.ts`文件中，我们将添加以下代码：

```java
// add anti-heroes to the database
  addAntiHero$ = createEffect(() =>{
    return this.actions$.pipe(
        ofType(AntiHeroActions.ADD_ANTI_HERO_API),
        mergeMap((data: {type: string, payload: AntiHero})
          => this.antiHeroService.addAntiHero(data.payload)
          .pipe(
            map(antiHeroes => ({ type:
              AntiHeroActions.ADD_ANTI_HERO_STATE,
              antiHero: data.payload })),
            tap(() =>
              this.router.navigate(["anti-heroes"])),
            catchError(() => EMPTY)
          ))
        )
    }, {dispatch: true})
```

在前面的代码示例中，我们可以看到我们创建了一个类似于删除功能效果的效果。这个效果使用`ADD_ANTI_HERO_API`类型，并从`antiHeroService`调用`addAntiHero()`函数来调用 POST API 将新数据添加到数据库中。

在成功调用 POST API 之后，效果将分发`ADD_ANTI_HERO_STATE`动作，并将来自 API 响应的新反英雄数据传递给 reducer 进行添加。我们还添加了一个`tap`操作符，它调用一个`navigate`函数，在创建新的反英雄后将导航到列表页面。

## 创建 reducer

在创建效果之后，我们需要将数据库中实现的变化与我们的存储同步，reducer 将完成这项工作。

让我们看看以下代码实现：

```java
export const antiHeroReducer = createReducer(
  initialState,
  on(setAntiHeroList, (state, { antiHeroes }) => { return
    {...state, antiHeroes}}),
  on(removeAntiHeroState, (state, { antiHeroId }) => {
    return {...state, antiHeroes:
     state.antiHeroes.filter(data => data.id !=
       antiHeroId)}
  }),
  on(addAntiHeroState, (state, {antiHero}) => {
    return {...state, antiHeroes: [...state.antiHeroes,
            antiHero]}
  }),
);
```

在前面的代码示例中，我们可以看到我们为*添加*功能添加了一个新的 reducer。它接受来自`addAntiHeroState`动作的新反英雄数据，并返回一个新的状态，其中`antiHeroes`值已被修改，新反英雄已添加到数组中。

如果 reducer 成功修改了`antiHeroes`状态的值，任何订阅了这个状态变化的选择器将发出新的值在组件中。

## 分发动作

我们需要做的最后一步是在我们的组件中分发动作。为了实现这一步，我们将在`anti-hero/components/anti-hero-form.component.ts`文件中调用分发动作，我们已添加了一个发射器，它传递表单的值和按钮标签以识别动作是创建还是更新。

让我们回顾一下我们为这个反英雄表单实现的代码：

```java
export class AntiHeroFormComponent implements OnInit {
  @Input() actionButtonLabel: string = 'Create';
  @Output() action = new EventEmitter();
  form: FormGroup;
  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      id: [''],
      firstName: [''],
      lastName: [''],
      house: [''],
      knownAs: ['']
    })
   }
  emitAction() {
    this.action.emit({value: this.form.value,
                      action: this.actionButtonLabel})
  }
}
```

在前面的代码示例中，我们可以看到反英雄表单将表单值作为反英雄对象发出，该对象将被传递到效果中。

这也提供了当前的操作，因为我们还将使用这种反英雄表单组件进行更新。一旦按钮被点击，我们就需要在`form.component.ts`文件中有一个函数来分发效果。

让我们看看以下代码示例：

```java
// form.component.html
<app-anti-hero-form [selectedAntiHero]="antiHero" (action)="formAction($event)"></app-anti-hero-form>
// form.component.ts
 formAction(data: {value: AntiHero, action: string}) {
    switch(data.action) {
      case "Create" : {
        this.store.dispatch({type:
          AntiHeroActions.ADD_ANTI_HERO_API,
          payload: data.value});
        return;
      }
      default: ""
    }
  }
```

在前面的代码示例中，我们可以看到我们创建了一个`formAction()`函数，它根据从反英雄表单组件传递的值分发一个动作。

这使用了一个`switch`语句，因为当动作是`update`时也会被调用。现在我们已经成功地为我们的应用程序使用 NgRx 的构建块创建了*添加*功能。

在下一节中，我们将实现具有副作用的数据修改。

# 使用 NgRx 更新带有副作用的项目

在本节的最后，我们将实现最后缺失的功能，即 *更新* 功能，我们将逐步创建构建块和组件中的分发逻辑，就像我们对 *添加* 和 *删除* 功能所做的那样。

## 创建动作

我们需要做的第一步是创建更新功能所需的动作类型和动作。我们首先创建所需的两个动作类型，分别是 `MODIFY_ANTI_HERO_API` 和 `MODIFY_ANTI_HERO_STATE`。第一个类型将由调用 API 的副作用使用，第二个类型将由通过根据新的反英雄对象更改数据来修改状态的 reducer 使用。

在创建了两个动作类型之后，我们还需要使用 `createAction()` 函数为 `MODIFY_ANTI_HERO_STATE` 类型创建一个动作。效果将在 API 调用成功后分发此动作。

让我们看看以下代码实现：

```java
 // See full code on https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-13
export enum AntiHeroActions {
  GET_ANTI_HERO_LIST = '[Anti-Hero] Get Anti-Hero list',
  SET_ANTI_HERO_LIST = '[Anti-Hero] Set Anti-Hero list',
  ADD_ANTI_HERO_API = '[Anti-Hero] Add Anti-Hero (API',
  ADD_ANTI_HERO_STATE =
    '[Anti-Hero] Add Anti-Hero (STATE)',
  REMOVE_ANTI_HERO_API =
    '[Anti-Hero] Remove Anti-Hero (API)',
  REMOVE_ANTI_HERO_STATE =
    '[Anti-Hero] Remove Anti-Hero (STATE)',
  MODIFY_ANTI_HERO_API =
    '[Anti-Hero] Modify Anti-Hero (API)',
  MODIFY_ANTI_HERO_STATE =
    '[Anti-Hero] Modify Anti-Hero (STATE)',
}
export const modifyAntiHeroState = createAction(
    AntiHeroActions.MODIFY_ANTI_HERO_STATE,
    props<{ antiHero: AntiHero }>()
);
```

在前面的代码示例中，我们可以看到我们在 `AntiHeroActions` 中添加了两个新类型。我们还创建了一个新的动作，其类型为 `MODIFY_ANTI_HERO_STATE`，它接受一个 `antiHero` 属性，该属性将用于修改存储中的当前值。

## 创建效果

下一步我们需要做的是创建 *添加* 功能的效果。在 `anti-hero/state/anti-hero.effect.ts` 文件中，我们将添加以下代码：

```java
// modify anti-heroes in the database
   modifyAntiHero$ = createEffect(() =>{
    return this.actions$.pipe(
        ofType(AntiHeroActions.MODIFY_ANTI_HERO_API),
        mergeMap((data: {type: string, payload: AntiHero})
          => this.antiHeroService.updateAntiHero(
          data.payload.id, data.payload)
          .pipe(
            map(antiHeroes => ({ type:
                AntiHeroActions.MODIFY_ANTI_HERO_STATE,
                antiHero: data.payload })),
            tap(() =>
              this.router.navigate(["anti-heroes"])),
            catchError(() => EMPTY)
          ))
        )
    }, {dispatch: true})
```

在前面的代码示例中，我们可以看到我们创建了一个类似于 *添加* 和 *删除* 功能的效果。此效果使用 `MODIFY_ANTI_HERO_API` 类型，并从 `antiHeroService` 调用 `updateAntiHero()` 函数来调用 PUT API 以修改具有 ID 参数的数据库中的反英雄。

在成功调用 PUT API 后，效果将分发 `MODIFY_ANTI_HERO_STATE` 动作，并将来自 API 响应的修改后的反英雄数据传递给 reducer，就像在 *添加* 效果中一样，我们同样添加了一个 `tap` 操作符，该操作符在修改反英雄后调用一个 `navigate` 函数，该函数将导航到列表页面。

## 创建 reducer

在创建效果之后，我们需要将数据库中实现的变化与我们的存储同步，reducer 将执行此操作。

让我们看看以下代码实现：

```java
export const antiHeroReducer = createReducer(
  initialState,
  on(setAntiHeroList, (state, { antiHeroes }) => {
    return {...state, antiHeroes}}),
  on(removeAntiHeroState, (state, { antiHeroId }) => {
    return {...state, antiHeroes:
      state.antiHeroes.filter(data => data.id !=
                              antiHeroId)}
  }),
  on(addAntiHeroState, (state, {antiHero}) => {
    return {...state, antiHeroes: [...state.antiHeroes,
                                   antiHero]}
  }),
  on(modifyAntiHeroState, (state, {antiHero}) => {
    return {...state, antiHeroes: state.antiHeroes.map(data
      => data.id === antiHero.id ? antiHero : data)}
  }),
);
```

在前面的代码示例中，我们可以看到我们为更新功能添加了一个新的 reducer。它接受来自 `addAntiHeroState` 动作的修改后的反英雄数据，并返回包含修改后的 `antiHeroes` 值的新状态，其中我们使用 `map()` 操作符用新对象替换了给定 ID 的反英雄。

如果 reducer 成功修改了 `antiHeroes` 状态的值，任何订阅此状态变化的 selectors 都会在组件中发出新的值。

## 分发动作

我们需要做的最后一步是将动作分发到我们的组件。为了实现这一步，我们将执行与添加功能相同的步骤。我们仍然会使用`anti-hero/components/anti-hero-form.component.ts`文件来更新数据。

这里的唯一区别是我们将绑定所选反英雄的值到我们的表单中；反英雄表单组件应该接受一个反英雄对象，并且应该通过表单组修补值。

让我们看看下面的代码示例：

```java
export class AntiHeroFormComponent implements OnInit {
  @Input() actionButtonLabel: string = 'Create';
  @Input() selectedAntiHero: AntiHero | null = null;
  @Output() action = new EventEmitter();
  form: FormGroup;
  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      id: [''],
      firstName: [''],
      lastName: [''],
      house: [''],
      knownAs: ['']
    })
   }
  ngOnInit(): void {
    this.checkAction();
  }
  checkAction() {
    if(this.selectedAntiHero) {
      this.actionButtonLabel = "Update";
      this.patchDataValues()
    }
  emitAction() {
    this.action.emit({value: this.form.value,
      action: this.actionButtonLabel})
  }
}
```

在前面的代码示例中，我们可以看到我们添加了`checkAction()`函数，该函数检查我们是否在反英雄表单组件中传递了一个反英雄对象。

这表明如果对象不为空，这将是一个*更新*动作，我们必须通过使用`patchValue()`方法绑定表单来在每个字段中显示所选反英雄的详细信息。

现在让我们来实现`form`组件的代码：

```java
// form.component.html
<app-anti-hero-form [selectedAntiHero]="antiHero" (action)="formAction($event)"></app-anti-hero-form>
// form.component.ts
antiHero$: Observable<AntiHero | undefined>;
  antiHero: AntiHero | null = null;
  constructor(private router: ActivatedRoute,
    private store: Store<AppState>) {
    const id = this.router.snapshot.params['id'];
    this.antiHero$ = this.store.select(selectAntiHero(id));
    this.antiHero$.subscribe(d => {
      if(d) this.antiHero = d;
    });
   }
 formAction(data: {value: AntiHero, action: string}) {
    switch(data.action) {
      case "Create" : {
        this.store.dispatch({type:
          AntiHeroActions.ADD_ANTI_HERO_API,
          payload: data.value});
        return;
      }
     case "Update" : {
        this.store.dispatch({type:
          AntiHeroActions.MODIFY_ANTI_HERO_API,
          payload: data.value});
        return;
      }
      default: ""
    }
  }
```

在前面的代码示例中，我们可以看到我们在`formAction()`函数中添加了一个新的情况，它也会分发一个动作，但动作类型为`MODIFY_ANTI_HERO_API`。

我们还使用了`selectAntiHero()`选择器，通过 URL 路由中的 ID 选择反英雄，该 ID 将被传递到我们的`anti-hero-form.component.ts`文件中。

# 摘要

通过这种方式，我们已经到达了这一章的结尾。让我们回顾一下我们学到的宝贵知识；我们使用 NgRx 的构建块完成了应用的 CRUD 功能，并且我们学习了在状态管理中使用和不使用副作用之间的区别。副作用对于我们的更改在存储中与数据库同步是必不可少的。

我们也一步一步地学习了如何使用我们应用所需的不同动作来创建 NgRx 的构建块。

在下一章中，我们将学习如何在 Angular 中应用安全功能，例如添加用户登录和注销、检索用户配置文件信息、保护应用路由以及调用具有受保护端点的 API。
