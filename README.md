# Про запахи. Или что я чую в твоём коде?

### Дисклеймер

Запах не означает ошибку. Он означает что надо внимательнее присмотреться к конкретному месту в коде. У многих из ниже
перечисленных запахов есть хорошие способы применения. Всегда думайте головой. Если сомневаетесь, приходите к коллегам.

Многие примеры ниже синтетические, надеюсь что со временем я заменю их реальными. Найти сходу столько реальных примеров
очень тяжело.

### Введение

Я решил собрать в одном месте мои рассуждения о хорошем и плохом коде, с причинами, примерами и способами его поправить. 
Материал долгое время использовался в виде презентации для студентов и сотрудников. Это попытка его собрать в одном 
удобном месте и расширять по возможности. Всё ниже написанное моё личное мнение, с которым можно не соглашаться. 
Можно предлагать правки и замечания, читаемость кода часто субъективное мнение, которое тяжело измерить (хоть это 
и пытались сделать).

Основная проблема запаха в коде - увеличение когнитивной нагрузки при чтении исходников. Усложнение понимания приводит к
замедлению разработки и неочевидным багам. Исправление запахов ниже направлено на то, чтобы разработчик прочитав как
можно меньший кусок кода понял что в нём происходит.

Запахи часто зависят от используемых языков. Какой-то язык не позволит вашему коду пахнуть, какой-то даст встроенное
средство борьбы, а какой-то заставит разбираться самостоятельно.

Запахи расположены в случайном порядке. Нет понятия более или менее опасный/плохой/сложный запах. Я советую сделать себе
чек лист и просматривать его каждый раз, когда вы делаете код ревью для коллеги или отправляете на ревью свой код.

### Содержание
- [Переменные без инициализации](#var-without-initializations)  
- [Строки как значения](#strings-as-values)
- [Примитивные типы данных](#primitive-types)
- [Длинные условия](#long-conditions)
- [Функции с множеством примитивных параметров](#too-much-parameters)
- [Сайд эффекты в функциях](#side-effects)
- [Break, continue, return и другие вариации goto внутри циклов и условий](#goto)
- [Вложенные условия](#nested-ifs)
- [Длинные циклы](#long-loops)
- [Рекурсия](#recursion)
- [Несоответствие названию](#wrong-naming)
- [Функция решает больше чем одну проблему](#multiple-purpose-functions)
- [Сеттеры](#setters)
- [Опциональные поля/типы](#optionals)

<a name="var-without-initializations"/>

## Переменные без инициализации

Во-первых, уточню, что я имею в виду под не инициализированными переменными - это любая переменная, которой эксплицитно
не присвоили значение. В примерах ниже переменные получают значение по умолчанию в момент декларирования, в случае JS -
это undefined, в случае Java - null для объектов ссылочного типа и 0 для int.

```javascript
let counter; // undefined
```

```java
int userAge; // 0
String username; // null
```

Иногда мы не можем в месте объявления переменной сразу определить каким значением оно обладает. Мы оставляем переменную
со значением undefined и дальше выполняем серию условий. Пример:

```javascript
const getAnimation = (params) => {
  let animation;

  if (params && params.isMoving) {
    animation = 'moving';
  } else if (params && params.isStopped) {
    animation = 'stopping';
  }

  return animation;
};
```

### Проблемы

1. Первая проблема значений по умолчанию в том, что они не несут смысла. undefined - буквально значит "не определена".
   Как можно строить предположения о смысле программы, если смысл отсутствует у используемых ей переменных?
2. Читателю приходится держать в голове информацию о том что эта переменная пока undefined и мы не знаем что там будет,
   на случай если переменная используется.
3. Переменную невозможно объявить константой (правда не во всех языках, та же Java так умеет).
4. Можно забыть инициализировать переменную и поймать undefined is not a function и аналоги (не так страшно в
   typescript).

### Способы борьбы

1. Давай значение по умолчанию. В этом случае даже если мы добавили ещё пару возможных вариантов для переменной, но
   забыли добавить для них условия, мы будем получать дефолтное значение.

```javascript
let animation = 'idle';
```

2. Выносить логику инициализации в отдельную функцию:

```javascript
const animation = getAnimation(params);
```

3. В крайнем случае использовать ЯП, который запрещает оставлять не инициализированную переменную. Хоть это и не
   исправит проблемы с декларативностью кода.

<a name="strings-as-values"/>

## Строки как значения

Строки одни из самых популярных типов данных. Проблема строк в том, что из-за них протекает система типов. Семантика
строки подразумевает, что это массив символов по которому мы можем итерироваться и делать срез. На этом всё. При этом мы
вполне можем видеть такие примеры кода:

```javascript
const setPhone = (phoneNumber: string) => {
  // todo implement
}

const putOnTheMap = (address: string) => {
  // todo implement
}
```

### Проблемы

Телефон конечно можно представить в виде строки, но у нас сразу исчезнет гарантия на его валидность: есть ли там код
страны? Используются ли скобки и тире для разделения? Сколько в нём цифр? В адресе может не оказаться квартиры или
название улицы будет написано в произвольном формате ("улица Ленина", "Ленина улица", "ул. Ленина").

### Способы борьбы

Мы можем каждый раз делать валидацию значений, а можем использовать гарантированно корректные типы данных:

```javascript
const setPhone = (phoneNumber: PhoneNumber) => {
}
```

Возможно класс PhoneNumber имеет метод parse и знает как собрать себя из разных форматов. Возможно у вас есть отдельный
PhoneNumberParser, который умеет производить объекты типа PhoneNumber. Важно то, что когда мы вызвали setPhone мы точно
знаем чего ожидать.

Строка - это тип данных с конкретной семантикой. Используйте их только когда вам нужны именно строки. Например, вы
пишете библиотеку поиска в тексте. Тогда вам неважно какой это текст, только что это набор символов.

<a name="primitive-types"/>

## Примитивные типы данных

Проблема такая же как со строками.

```javascript
let age: number = -42; // теперь валидный код
```

### Способы борьбы

Лечить точно так же: выносим в отдельные типы. Если не хочется выносить в отдельный тип, гарантируйте что значение
всегда провалидировано и код валидации переиспользуется, а не копируется.

<a name="long-conditions"/>

## Длинные условия

Многострочные условия в if'ах, когда в логике чёрт ногу сломит.

```javascript
const bad = (value) => {
  if (value && value >= 0 && value % 2 === 1
    || value && value < 0 && value % 2 === -1) {
    console.log(value);
  }
}
```

### Проблемы

Достаточно очевидная проблема с тем, что читателю тяжело понять в каком случае выполняется тело условия.

### Способы борьбы

В идеальном случае условие можно упростить. Для этого пользуемся правилами булевой алгебры 
[Логика высказываний](https://ru.wikipedia.org/wiki/%D0%9B%D0%BE%D0%B3%D0%B8%D0%BA%D0%B0_%D0%B2%D1%8B%D1%81%D0%BA%D0%B0%D0%B7%D1%8B%D0%B2%D0%B0%D0%BD%D0%B8%D0%B9#%D0%A2%D0%BE%D0%B6%D0%B4%D0%B5%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D0%BE_%D0%B8%D1%81%D1%82%D0%B8%D0%BD%D0%BD%D1%8B%D0%B5_%D1%84%D0%BE%D1%80%D0%BC%D1%83%D0%BB%D1%8B_(%D1%82%D0%B0%D0%B2%D1%82%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D0%B8)).

![Законы булевой алгебры](https://lh3.googleusercontent.com/Q6YTpzFta3PalwmSwyYHjF61047s8HdFqj97IBeEiFmeT2R7_92Z7G0e0mOCmb5wSe-w5yNYqIaHZrPldircPKW3fVASdZQ2VP-W7XmF5YUgwJFqgHJRVGAAoToWS1okhSMr54U3)

Закон де моргана один из моих любимейших, позволят избавиться от операции отрицания (превращая 3 операции в 2 и иногда
упрощая само чтение).

Таблицы истинности, если условие переходит в разряд библейских бедствий. В таблице истинности смотрим на инварианты и
невозможные ситуации, часто получается что несколько операций сравнения можно просто выкинуть.

Если упростить выражение не получается, выносите условия в отдельные переменные и функции:

```javascript
const good = (value) => {
  const positiveOdd = value && value >= 0 && value % 2 === 1;
  const negativeOdd = value && value < 0 && value % 2 === -1;
  if (positiveOdd || negativeOdd) {
    console.log(value);
  }
}
```

<a name="too-much-parameters"/>

## Функции с множеством примитивных параметров

Выше мы уже обсудили проблему примитивных типов, они не обладают достаточной семантикой. Всё становится особенно весело,
когда функция принимает много однотипных параметров:

```javascript
const draw = (x, y, dx, dy, r, g, b, callback) => {
  // a lot in here
}

draw(10, 30, 30, 50, 255, 255, 120, () => {
});
```

### Проблема

1. При вызове метода легко перепутать порядок и статический анализатор промолчит. В лучшем случае мы упадём, в худшем мы
   получим не тот результат и будем очень долго искать проблему.
2. Ужасное понимание при чтении из-за часто используемых magic numbers, а если выносить все параметры в переменные, то
   получаем простыню сразу перед вызовом. (спорная проблема, тот же WebStorm умеет подсвечивать названия параметров)?

### Способы борьбы

1. Группировать параметры в типы:

```javascript
const anotherDraw = (rect, color, callback) => {
  const {x, y, dx, dy} = rect;
  const {r, g, b} = color;
// a lot in here
}
```

2. Если это внешняя библиотека, пишите один раз обёртку, покрывайте её тестами и дальше используете функции с красивым
   API.

<a name="side-effects"/>

## Сайд эффекты в функциях

Когда функция меняет внешнее для неё состояние:

```javascript
let external_state = {
  flag: true,
};

const bad = () => {
  external_state.flag = !external_state.flag;
}

const also_bad = (params) => {
  // меняем по ссылке внешний для нас стейт
  params.flag = !params.flag;
}
```

### Проблема

1. Невозможно просто прочитать функцию и понять что она делает, потому что часть логики находится вне её. Если есть
   зависимость от внешнего стейта, то любая другая функция может его поменять.
2. Сильно усложняется написание юнит тестов, потому что приходится мокать внешние стейты.

### Способы борьбы

1. Если функция не может жить без внешнего стейта, то это хороший кандидат на превращение в метод класса. Стейт всё ещё
   внешний для функции, но он инкапсулирован в класс, т.е. мы точно может отследить все его изменения.
2. Передаём стейт как параметр, если его надо мутировать, то используем для этого возвращаемое значение.

```javascript
const good = (params) => {
  return !params.flag;
}

const anotherState = {
  flag: true,
}
const changed_state = good(anotherState);
```

<a name="goto"/>

## Break, continue, return и другие вариации goto внутри циклов и условий

Операторы перехода в коде в том или ином виде есть в каждом современном ЯП. Обычно используются для “экстренного” выхода
из цикла или функции, если мы достигли нужно нам результата раньше времени.

```javascript
const bad = (list) => {
  let sum = 0;
  while (true) {
    sum += list.shift()
    if (list.length === 0) break;
  }

  return sum;
}
```

### Проблема

1. Усложняют линейное чтение кода, приходится всегда следить за тем что мы уже сделали, а что нет. Потому что мы вышли
   из цикла/условия/функции раньше, чем они закончились.
2. В циклах могут нарушиться инварианты.
3. При добавлении новой функциональности каждый раз надо проверять должна ли она быть выше/ниже "goto".

### Способы борьбы

1. Очень часто все условия можно описать в самом цикле.

```javascript
const good = (list) => {
  let sum = 0;
  while (list.length !== 0) {
    sum += list.shift()
  }

  return sum;
}
```

2. В функциях подобные return'ы можно переместить в начало тела и использовать как валидаторы.
3. Если без “goto” не обойтись, то кусок кода с ним лучше вынести в отдельную функцию.

<a name="nested-ifs"/>

## Вложенные условия

```javascript
if (Object.values(LocationType).includes(currentLocation)) {
  if (currentLocation === LocationType.ENDLESS_DUNGEON) {
    setCurrentEndlessDungeon(LocationType.ENDLESS_DUNGEON);

    if (
      storyStore.battlesId.endlessDungeon !== storyStore.currentBattleId &&
      worldMapStore.savedBattle.state !== null
    ) {
      setVisiblePopupUnfinished(true);
      return;
    } else {
      if (
        storyStore.battlesId.endlessDungeon ===
        storyStore.currentBattleId &&
        worldMapStore.savedBattle.state !== null
      ) {
        startUnfinishedEndlessDungeon();
      } else {
        setVisiblePopup(true);
      }
    }
  } else if (currentLocation === LocationType.ENDLESS_DUNGEON_2) {
    setCurrentEndlessDungeon(LocationType.ENDLESS_DUNGEON_2);
    if (
      storyStore.battlesId.endlessDungeon2 !== storyStore.currentBattleId &&
      worldMapStore.savedBattle.state !== null
    ) {
      setVisiblePopupUnfinished(true);
      return;
    } else {
      if (
        storyStore.battlesId.endlessDungeon2 ===
        storyStore.currentBattleId &&
        worldMapStore.savedBattle.state !== null
      ) {
        startUnfinishedEndlessDungeon();
      } else {
        setVisiblePopup(true);
      }
    }
  } else {
    worldMapStore.showLevelSelection(currentLocation);
  }
}
```

### Проблема

1. Теряется декларативность и линейность чтения кода.
2. Усложняется разработка. Если ты хочешь добавить новое условие, в какой из вложенных if'ов его нужно положить? А если
   подходящего нет, то вложенность увеличивается.

### Способы борьбы

Во-первых, можно использовать подходы из раздела про длинные условия: законы булевой алгебры и таблицы истинности.
Во-вторых, есть несколько простых техник позволяющих упростить код:

1. Верхнее условие часто превращается в sanity check. Из примера выше видимо что первое условие не имеет блока else и
   кода после него, значит мы можем проверить его отрицание и просто выйти сразу из функции - это -1 уровень
   вложенности.

```javascript
if (!Object.values(LocationType).includes(currentLocation)) return;
```

2. Схлопывать условия в более “широкие”.
3. Условия на одной переменной превращаются в switch case. При этом тело условия может быть вынесено в отдельную
   функцию.

```javascript
switch (currentLocation) {
  case LocationType.ENDLESS_DUNGEON:
    foo();
    break;
  case LocationType.ENDLESS_DUNGEON_2:
    bar();
    break;
  default:
    worldMapStore.showLevelSelection(currentLocation);
    break;
}
```

4. Обратные друг другу условия превращаются из if-if в if-else.

```javascript
if (worldMapStore.savedBattle.state === null) {
  setVisiblePopup(true);
}

if (storyStore.battlesId.endlessDungeon !== storyStore.currentBattleId) {
  setVisiblePopupUnfinished(true);
} else {
  startUnfinishedEndlessDungeon();
}
```

5. Выносить внутренние условия наружу. Код становится декларативным, но есть опасность словить несколько условий, если
   нарушается инвариант. Например, в п.4, я вынес наружу условие `worldMapStore.savedBattle.state === null`. Но если тело
   условия `setVisiblePopup(true);` меняет какой-то внешний стейт, который повлияет на следующие условия
   `storyStore.battlesId.endlessDungeon !== storyStore.currentBattleId`, то замена может оказаться не равнозначной.

<a name="long-loops"/>

## Длинные циклы

Любой цикл работает как функция с сайд эффектом: внутри цикла находится какая-то логика, она меняет состояние вне цикла.
Разница в том, что эта функция по условию не линейна - это цикл, и каждый вызове его состояние меняется.

Для тех кто хочет подробнее послушать про то почему большие циклы - это плохо и как с ними бороться советую к просмотру: https://www.youtube.com/watch?v=IzNtM038JuI

Код ниже синтетический пример того как делать не надо. Мы копируем список в свою структуру данных MyList, параллельно
сортируя его и подсчитывая количество элементов.

```javascript
class MyList {
  constructor(list, size) {
    this.list = list;
    this.size = size;
  }
}

const copy = (list) => {
  let result = [list[0]];
  let counter = 1;

  for (let i = 1; i < list.length; i++) {
    let f = true;
    for (let j = 0; j < result.length; j++) {
      if (result[j] > list[i]) {
        result.splice(j, 0, list[i]);
        f = false;
        break;
      }
    }

    if (f) {
      result.push(list[i]);
    }

    counter++;
  }

  return new MyList(result, counter);
}
```

### Проблема

Соответственно цикл обладает всеми проблемами функции с сайд эффектами и при этом не имеет возможности от них
избавиться. Наша задача уменьшить негативное влияние.

### Способы борьбы

1. Уменьшаем длину цикла. Идеальный цикл содержит не больше 2-х строчек кода.

```javascript
for (let i = 1; i < list.length; i++) {
  result.push(list[i]);
}
```

2. Уменьшаем количество сайд эффектов. 1 цикл - 1 изменение. Да, это ухудшит производительность. В 99% случаев вас это
   не волнует (смотрите на размеры данных и разницу в асимптотике, часто вы просто уменьшаете константу, которую и так
   не видно под О-большим). Вас волнует красивый поддерживаемый код. Если он будет тормозить вы всегда легко сможете
   превратить красивый медленный код в некрасивый быстрый. А вот наоборот делать тяжело.
3. Выносим циклы в отдельные функции. Думайте об этом как о замене цикла на map-filter-reduce. Отдельная функция
   инкапсулирует в себе “сайд эффект” и для внешнего пользователя тестирование и использование будет прозрачным.

```javascript
result.sort();
```

Изначальный код мог бы выглядеть так:

```javascript
const copyGood = (list) => {
  if (!list) return new MyList([], 0);

  let result = [list[0]];

  for (let i = 1; i < list.length; i++) {
    result.push(list[i]);
  }

  result.sort();

  return new MyList(result, result.length);
}
```
Помните, что forEach - это такой же цикл, который часто приезжает в язык с map/filter/reduce, 
и мимикрирует под функциональщину. Он не функциональный, к нему применяются такие же правила, как и к циклам.

<a name="recursion"/>

## Рекурсия

Вызов функции из самой себя напрямую или через цепочку других функций - это удобно. Иногда на столько удобно, что
нерекурсивное решение получается в 10 раз длиннее и в 100 раз запутаннее. Но я мою практику я видел всего 2 таких случая
в реальном коде. Намного чаще мы видим вот такой код:

```javascript
const filterUndefined = (l) => {
  let a = l.pop();
  if (l.length > 0) {
    filterUndefined(l);
  }
  if (a !== undefined) {
    l.push(a);
  }

  return l;
}
```

### Проблема

1. Часто рекурсия требует использования внешнего стейта, что автоматически добавляет функции side-effect.
2. Рекурсивный вызов делает код ещё более нелинейным чем цикл. Если вы видели проблему там, то тут её можно умножить на
    2.
3. С точки зрения производительности рекурсия в лучшем случае не хуже. А чаще всего проигрывает итеративным алгоритмам.
4. Рекурсия может быть не очевидна, если она формируется через цепочку вызовов функций: a() → b() → c() → a()

### Способы борьбы

1. Использовать итеративные алгоритмы. В большинстве случае рекурсивный алгоритм очень прозрачно переписывается на while
   loop + queue/stack.
2. Передавать стейт как параметр внутренней функции, которая рекурсивно вызывает сама себя, наружу отдавать функцию
   обёртку, чтобы стейт не утёк. Альтернатива использовать замыкание. Это всё ещё больно. Но по крайней мере боль скрыта
   от внешнего пользователя функции.

<a name="wrong-naming"/>

## Несоответствие названию

Не всегда легко придумать название функции. Но всегда нужно стараться это сделать.

```javascript
<MyButton onclick=clickHandler/>
```

### Проблема

1. Читатель не понимает, что происходит в коде пока не прочитает код.
2. Читатель понимает код неправильно, потому что название класса/функции/переменной не соответствует действительности.

### Способы борьбы

Пишите правильные названия. Посмотрите на пример выше, там вызывается функции clickHandler в момент нажатия на кнопку.
Проблема с названием clickHandler в том, что оно описывает "когда" она это делает, вместо "что".
Т.е. теперь нам надо посмотреть что происходит в коде, чтобы узнать, что произойдёт если я нажму на кнопку. И, если я
переиспользую код clickHandler в другом месте, где нет кнопки и клика, то название в целом не будет соответствовать
ничему. Из этого кода сразу ясно, что если я нажму на кнопку то я увижу попап с поздравлением.

```javascript
<MyButton onclick=showCongratPopup/>
```

Не используйте общие названия. Забудьте про process, handle, parse, etc. Есть места где их можно использовать, но чаще
всего нет. Представьте что вместо функции subString у вас была бы функция processString. Она процессит строку? Ну в
какой-то степени да. Вам из названия ясно что процессить - это получать подстроку? Вряд ли. Всегда описывайте именно
действие совершаемое функцией.

<a name="multiple-purpose-functions"/>

## Функция решает больше чем одну проблему

Частый подход в написании ПО, особенно в начале пути:

1. Выплеснуть поток сознания в код.
2. Отрефакторить.

Часто вторая часть получается не очень и некоторые функции продолжают решать несколько проблем сразу: получить данные,
провалидировать их, обработать и сохранить, — это 4 разных действия, которые должны решаться 4мя разными функциями. Но,
скорее всего, хотя бы 2 из них будут объединены в одну, а возможно и все 4, особенно если кода немного.

### Проблема

1. Нельзя использовать функцию только для решения одной проблемы.
2. Из названия может быть не очевидно, что от функции можно ожидать большего.
3. Тяжело расширять, можно сломать соседний функционал.
4. Код функции становится больше.

### Способы борьбы

1. Дробить на несколько функций.
2. Давать понятное название и которого следует что действий будет несколько.

<a name="setters"/>

### Сеттеры (раздел в работе)

```javascript
-endAnimating();
+setAnimating(false);
```

Это кстати не такая хорошая замена. Сеттеры - сложная тема в ООП, кто-то их любит, кто-то нет. Есть мнения, что они
нарушают настоящую инкапсуляцию. Но это не так важно. Важно то что методы `startAnimation()` и `endAnimation()` (только
animation, а не animating) читаются намного проще чем `setAnimating(false)`.

Потому что в терминах ООП, у тебя есть объект, который скрывает своё состояние и предоставляет API для работы с ним. На
примере более менее реальном. У тебя есть какой-то провайдер. Предположим он предоставляет стрим данных.
И скажем на него подписаны клиенты

```javascript
const streamProvider = new StreamProvider();
const client = new Client();

// добавляем клиент как наблюдателя, заметь что 
// это выглядет как сеттер, но называется адекватно
streamProvider.addListener(client);

// это аналог твоего булевого сеттера, и внутри 
// действительно может быть банальное присвоение 
// значения полю, но с точки зрения API мне всё равно 
// что там внутри, мне нужна наглядность, я сказал 
// что стрим запустился и теперь наблюдатели могут 
// сделать какие-то приготовления прежде чем получат данные
streamProvider.startStream();

// отправляю данные в стрим, опять же, не 
// выглядит как сеттер, но по факту может быть им 
streamProvider.push([1, 2, 3]); 
```

Каких правил я бы придерживался:

1. Не использовать сеттеры для булевых значений.
2. Использовать сеттеры, когда у тебя обычный data object без логики и ты действительно не более чем меняешь его
   состояние. и даже в этом случае, возможно использовать другие паттерны оставляющие иммутабельность и убирающие
   сеттеры (например builder)
3. Стараться писать понятное для пользователя API, сеттер не несёт логики в своём названии кроме как "я проставил в поле
   вот такое значение", если после проставления значения произойдёт что-то ещё (а у тебя произойдёт), то назови метод
   соответствующе

<a name="optionals"/>

## Опциональные поля/типы
Практически в каждом языке программирования и базе данных, есть возможность объявлять опциональные поля. 
Это может быть что-то олдскульное, когда мы разрешаем null или новомодные монады Optional, Maybe, или даже 
поддерживаемые языком конструкции:

```typescript
export interface TRouteParams {
   courseNumber?: string;
   lessonNumber?: string;
   cardNumber?: string;
   courseId?: string;
   lessonId?: string;
   cardId?: string;
}
```

### Проблема

Проблема практически такая же как и в “переменные без инициализации”, мы не знаем есть ли у поля значение или нет. 
И если его нет, то что это значит. В примере выше мы можем получить структуру с произвольной комбинацией полей. 
Нас это устраивает? Всегда ли это имеет смысл? А если часть комбинаций невалидна, то как мы это проверим?

### Способы борьбы

Многие языки начинают поддерживать алгебраические типы данных, TS тоже в какой-то степени это делает. 
Через них всегда можно описать возможные комбинации полей.

https://en.wikipedia.org/wiki/Algebraic_data_type
https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions

Предположим вам не повезло и такой возможности нет. Всегда есть вариант с изменением интерфейса. Мы делаем 
интерфейс, которые умеет возвращать данные в агрегированном виде (в примере ниже это строка). И 
реализации интерфейса, которые хранят разные комбинации полей. Важно помнить, что, если этот агрегированный 
вид изначально был результатом какой-то сложной логики, то стоит сделать отдельный класс для его вычисления 
и передавать его через инъекцию зависимостей.

```java
public interface Route {
    String getRoute();
}

class CourseRoute implements Route {
    private final String courseNumber;

    public CourseRoute(String courseNumber) {
        this.courseNumber = courseNumber;
    }

    @Override
    public String getRoute() {
        return courseNumber;
    }
}

class LessonRoute implements Route {
    private final String courseNumber;
    private final String lessonNumber;

    LessonRoute(String courseNumber, String lessonNumber) {
        this.courseNumber = courseNumber;
        this.lessonNumber = lessonNumber;
    }

    @Override
    public String getRoute() {
        return String.format("%s/%s", courseNumber, lessonNumber);
    }
}

class Usage {
    public void use() {
        Route courseRoute = new CourseRoute("12"); 
        Route lessonRoute = new LessonRoute("12", "42");
        System.out.println(courseRoute.getRoute());
        System.out.println(lessonRoute.getRoute());
    }
}
```

Если менять интерфейс не хочется, например, не получается придумать удобный для использования. 
Или комбинаций полей очень много и вносить в каждую из них RouteCalculator для получения агрегата 
лишь усложняет код, можно использовать приведение типов. Приведение типов в целом сам по себе запах, 
который может привести к ошибкам в рантайме, но если вы сможете гарантировать инвариант, то почему нет. 
Мы всё ещё наследуемся от общего интерфейса, но только для того, чтобы уметь отдавать тип нашего класса. 
С помощью типа мы понимаем к какому классу нам надо привестись, чтобы получить доступ ко всем полям.

```java
public enum RouteTypes {
    COURSE_TYPE,
    LESSON_TYPE,
    ;
}

public interface Route {
    RouteTypes getType();
}

class CourseRoute implements Route {
    public final String courseNumber;

    public CourseRoute(String courseNumber) {
        this.courseNumber = courseNumber;
    }

    @Override
    public RouteTypes getType() {
        return RouteTypes.COURSE_TYPE;
    }
}

class LessonRoute implements Route {
    public final String courseNumber;
    public final String lessonNumber;

    LessonRoute(String courseNumber, String lessonNumber) {
        this.courseNumber = courseNumber;
        this.lessonNumber = lessonNumber;
    }

    @Override
    public RouteTypes getType() {
        return null;
    }
}

class Usage {
    public void use(Route route) {
        switch (route.getType()) {
            case COURSE_TYPE: processCourseRoute((CourseRoute)route);
            case LESSON_TYPE: processLessonRoute((LessonRoute)route);
            default: throw new IllegalArgumentException("Unsupported type: " + route.getType());
        }
    }
    
    private void processCourseRoute(CourseRoute route) {}
    private void processLessonRoute(LessonRoute route) {}
}
```

Оба варианта выглядят многословно. Третий подход уменьшает количество опциональных полей через
отдельные классы. В примере ниже мы выносим адрес в отдельный класс. В первом варианте мы гарантировали
наличие имени и фамилии, но адрес мог состоять из произвольного набора полей, например у нас могла быть 
только квартира. Во второй варианте адрес или есть или нет, но если он есть, то мы точно знаем что в нём есть все поля.

```java
class BadProfile {
    private String firstName;
    private String lastName;
    private Optional<String> country;
    private Optional<String> city;
    private Optional<String> street;
    private Optional<String> house;
    private Optional<String> flat;
}

class GoodProfile {
    private String firstName;
    private String lastName;
    private Optional<Address> address;
}

class Address {
    private String country;
    private String city;
    private String street;
    private String house;
    private String flat;
}
```

Последний, но не менее возможный вариант валидация при создании. Например, с одним из моих любимейших 
паттернов Builder. Мы собираем все необходимые поля по частям, но прежде чем создать объект Route 
проверяет наши инварианты.

```java
class Route {
    private final Optional<String> courseNumber;
    private final Optional<String> lessonNumber;
    private final Optional<String> cardNumber;

    public Route(
            Optional<String> courseNumber,
            Optional<String> lessonNumber,
            Optional<String> cardNumber)
    {
        this.courseNumber = courseNumber;
        this.lessonNumber = lessonNumber;
        this.cardNumber = cardNumber;
    }

    public class Builder {
        private String courseNumber;
        private String lessonNumber;
        private String cardNumber;

        public Builder setCourseNumber(String courseNumber) {
            this.courseNumber = courseNumber;
            return this;
        }

        public Builder setLessonNumber(String lessonNumber) {
            this.lessonNumber = lessonNumber;
            return this;
        }

        public Builder setCardNumber(String cardNumber) {
            this.cardNumber = cardNumber;
            return this;
        }

        public Route build() {
            if (this.cardNumber != null 
                    && (this.lessonNumber == null || this.courseNumber == null)) {
                throw new IllegalStateException(
                        "Card number can't be set without lesson and course numbers"
                );
            }

            if (this.lessonNumber != null && this.courseNumber == null) {
                throw new IllegalStateException(
                        "Lesson number can't be set without course numbers"
                );
            }

            return new Route(
                    Optional.ofNullable(this.courseNumber),
                    Optional.ofNullable(this.lessonNumber),
                    Optional.ofNullable(this.cardNumber)
            );
        }
    }
}
```
