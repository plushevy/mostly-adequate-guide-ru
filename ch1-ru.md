# Глава 01: Чем предстоит заниматься

## Вступление

Привет! Позвольте представиться: меня зовут профессор Франклин Фрисби, и я собираюсь обучить вас основам функционального программирования. Но хватит уже обо мне, давайте поговорим о вас! Надеюсь, вы хотя бы немного знакомы с языком JavaScript, и имеете хотя бы небольшой опыт в объектно-ориентированном программировании. Вам не обязательно обладать докторской степенью в энтомологии; все, что вам нужно — уметь находить и устранять баги.

Я не ожидаю от вас познаний в функциональном программировании, потому что мы оба знаем, чем оборачиваются ожидания. Однако я предполагаю, что вы попадали в неприятные ситуации, связанные с мутабельным (изменяемым) состоянием, неограниченными побочными эффектами и безыдейным дизайном программ. 

Теперь, когда мы закончили со знакомством, предлагаю приступить к делу. Цель этой главы — дать вам представление о том, к чему мы стремимся, когда пишем функциональные программы. Для того, чтобы понимать следующие главы, мы должны усвоить, что именно делает программу «функциональной». В противном случае мы будем городить код, руководствуясь, например, избеганием объектов. Вместо этого нам необходим строгий ориентир.

Итак, существует ряд общих принципов программирования — сокращения, которыми мы руководствуемся в процессе работы над любым приложением: DRY (don't repeat yourself — не повторяйся); YAGNI (ya ain't gonna need it — тебе это не понадобится); слабая связанность и сильное зацепление (loose coupling high cohesion); принцип «наименьшей неожиданности» (principle of least surprise); принцип единственной ответственности (single responsibility) и т.д.

Я не стану перечислять все принципы, о которых я когда-либо слышал. Все они имеют отношение и к функциональному программированию, но к нашей конечной цели они относятся слабо. Прежде чем мы двинемся дальше, я хочу, чтобы вы прочувствовали замысел, который предшествует кодированию — нашу функциональную идиллию.

<!--BREAK-->

## О чем пойдет речь?

Давайте начнём с немного безумного примера. У нас есть приложение «чайка». Когда стаи чаек объединяются (conjoin), их количество складывается, а когда размножаются (breed) — умножается. Этот пример не следует рассматривать как хороший объектно-ориентированный код, — он нужен здесь для того, чтобы подчеркнуть опасности современного подхода, основанного на «присваивании». Рассмотрим подробнее:

```js
class Flock {
  constructor(n) {
    this.seagulls = n;
  }

  conjoin(other) {
    this.seagulls += other.seagulls;
    return this;
  }

  breed(other) {
    this.seagulls = this.seagulls * other.seagulls;
    return this;
  }
}

const flockA = new Flock(4);
const flockB = new Flock(2);
const flockC = new Flock(0);
const result = flockA
  .conjoin(flockC)
  .breed(flockB)
  .conjoin(flockA.breed(flockB))
  .seagulls;
// 32
```

Кто вообще мог сотворить такую мерзость? Неоправданно трудно следить за тем, в каком порядке мутирует (изменяется) внутреннее состояние, и, более того, ответ неправильный! Должно было получиться `16`, но `flockA` был цинично изменен прямо в процессе. Бедняжка `flockA`. Это какая-то анархия в ИТ! Дикая животная арифметика!

Если вы не поняли код этой программы, ничего страшного — я тоже его не понял. Запомнить стоит то, что за состоянием и мутабельными значениями трудно проследить, даже в таком небольшом примере.

Давайте попробуем снова; на этот раз мы применим более функциональный подход:

```js
const conjoin = (flockX, flockY) => flockX + flockY;
const breed = (flockX, flockY) => flockX * flockY;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result =
    conjoin(breed(flockB, conjoin(flockA, flockC)), breed(flockA, flockB));
// 16
```

Теперь мы получили правильный ответ с гораздо меньшим количеством кода. Вложенность функций немного запутывает... (мы исправим эту ситуацию в главе 5). Давайте копнём ещё немного глубже. Есть польза в том, чтобы называть вещи своими именами. Если мы взглянем критически на наши функции, то обнаружим, что работаем с обычным сложением (`conjoin`) и умножением (`breed`), ничего другого в них не происходит.

И правда, в этих функциях нет ничего особенного, кроме их имён. Давайте переименуем их в `multiply` и `add`, чтобы название отражало суть.

```js
const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result =
    add(multiply(flockB, add(flockA, flockC)), multiply(flockA, flockB));
// 16
```
А поскольку теперь мы имеем дело с арифметикой, мы можем применять магию:

```js
// сочетательное свойство (associative)
add(add(x, y), z) === add(x, add(y, z));

// переместительное свойство (commutative)
add(x, y) === add(y, x);

// свойство нейтрального элемента (identity)
add(x, 0) === x;

// распределительное свойство (distributive)
multiply(x, add(y, z)) === add(multiply(x, y), multiply(x, z));
```

Старые добрые математические свойства оказались весьма полезны. Не страшно, если вы не смогли их сразу вспомнить, многие из нас годами никак не использовали эти законы арифметики. Давайте попробуем применить те же свойства, чтобы упростить нашу программу.

```js
// Первоначальный вариант
add(multiply(flockB, add(flockA, flockC)), multiply(flockA, flockB));

// Применим свойство нейтрального элемента (напоминаю: `flockС == 0`)
// (add(flockA, flockC) == flockA)
add(multiply(flockB, flockA), multiply(flockA, flockB));

// Применим распределительное свойство и получим:
multiply(flockB, add(flockA, flockA));
```

Прекрасно! Нам не понадобилось написать ни строчки кода, только вызвать наши функции. Мы включили определения функций `add` и `multiply` в код для полноты, но писать их самостоятельно не нужно — мы можем использовать стороннюю библиотеку, где функции `add` и `multiply` уже точно есть.

Вы можете подумать: «очень хитро с твоей стороны привести такой математический пример». Или «реальные программы не такие простые, и подобные рассуждения к ним не применимы». Я выбрал этот пример, потому что большинство из нас уже знакомы с операциями сложения и умножения, поэтому легко заметить, насколько математика оказывается полезна нам в программировании.

Не грустите — в этой книге мы воспользуемся теорией категорий, теорией множеств и лямбда-исчислением так же элегантно и просто для описания программ, применимых к реальному миру. От вас не потребуется математических знаний. Это будет так же естественно и легко, как использовать обычный фреймворк или API.

Возможно, вы удивитесь, если услышите, что можно писать прикладные приложения с применением функционального подхода, аналогичного примеру выше. Лаконичные программы с понятными свойствами, о которых легко рассуждать и просто анализировать. Программы, которые не изобретают велосипед через каждую строчку. Отсутствие чётких законов может быть полезным, если вы преступник, но в этой книге мы познакомимся с законами математики и будем их соблюдать.

Мы будем использовать теорию, в которой каждая часть идеально дополняет остальные. Мы будем представлять конкретную прикладную задачу как состоящую из универсальных частей, чтобы затем максимально использовать их свойства. Это потребует немного больше дисциплины, чем подход «и так сойдёт» императивного программирования (я приведу точное определение императивного подхода позже в этой книге, а пока считайте, что это всё, что за пределами функционального программирования). Выгода от работы в жёстких математических рамках действительно поразит вас.

Мы уже заметили мерцание нашей функциональной Полярной звезды где-то вдалеке, но, прежде чем мы начнем наше путешествие, нам необходимо освоить ещё несколько базовых концепций.

[Глава 02: Функции первого класса](ch2-ru.md)
