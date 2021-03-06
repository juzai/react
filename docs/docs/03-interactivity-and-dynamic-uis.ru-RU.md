---
id: interactivity-and-dynamic-uis-ru-RU
title: Интерактивные и динамические интерфейсы
permalink: docs/interactivity-and-dynamic-uis-ru-RU.html
prev: jsx-gotchas.html
next: multiple-components.html
---

Вы уже знаете [как показывать данные](/react/docs/displaying-data.html) с React. Теперь давайте добавим в наши интферфейсы немного интерактивности.

## Простой пример

```javascript
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'liked' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```

## Обработка событий и синтетические события

С React вы просто передаете функцию-обработчик нужного события как аргумент, почти так же, как делали это в HTML. Благодаря механизму синтетических событий React гарантирует, что все события будут вести себя одинаково во всех браузерах. Другими словами, React знает, как работает всплытие и перехват событий по спецификации. События, которые он передает в ваши обработчики, будут соответствовать [спецификации W3C](http://www.w3.org/TR/DOM-Level-3-Events/), несмотря на то, каким браузером вы пользуетесь.

## Как это работает: автоматическое связывание и делегирование событий

Чтобы ваш код был не только понятным, но и быстрым, React делает следующее:

**Автоматическое связывание:** Когда в JavaScript создаются функции обратного вызова, вам надо привязать метод к тому объекту, на котором он будет вызываться, чтобы значение `this` было корректным. С React привязка метода к компоненту происходит автоматически ([кроме тех случаев, когда вы используете классы ES6](/react/docs/reusable-components.html#no-autobinding)). И делается это с минимальной нагрузкой на процессор и память.

**Делегирование событий:** На самом же деле, React добавляет обработчики событий не к узлам дерева. Сразу после запуска, React начинает прослушивать все события с самого верхнего уровня, используя единый слушатель. Когда добавляется новый компонент или удаляется старый, обработчики событий просто добавляются или удаляются из памяти React. И когда событие наступает, React уже заранее знает какому из обработчиков его передать. Когда в памяти больше не остается обработчиков, React перестает обрабатывать события. Если хотите узнать о том, почему эта механика так быстро работает, почитайте [отличный пост в блоге David Walsh](http://davidwalsh.name/event-delegate).

## Компоненты как конечные автоматы

React считает интерфейсы обыкновенными конечными автоматами. Работать с интерфейсом становится проще, если представлять его как конечный автомат, который меняет состояния и отрисовывает их.

В React вы просто обновляете состояние компонента, а потом выводите новый интерфейс уже с новыми данными. Все изменения в DOM-дереве React сделает сам, причем наиболее эффективным способом.

## Как работает состояние

Чтобы сообщить React о том, что данные изменились, вы вызываете метод `setState(data, callback)`. В этом методе происходит обновление состояния `this.state` новыми данными из `data`, и компонент отрисуется заново. После этого вызывается функция `callback`. Но вы редко будете ей пользоваться, ведь React сам обновляет интерфейс.

## В каких компонентах хранить состояние?

Большинство компонентов должны просто брать данные из `props` и отрисовывать их. Но иногда вам надо реагировать на действия пользователя, делать запросы на сервер или просто сделать что-то по таймеру. В таких случаях используйте состояние.

**Старайтесь делать компоненты без состояния.** Следуя этому правилу, вы будете выносить работу с состоянием с уровня представления в другие, более подходящие места. Тем самым, вы снизите сложность приложения, упрощая его понимание.

Основной принцип такой: создаются несколько компонентов без состояния, которые формируют дерево. Они будут заниматься только отрисовкой данных. А все данные для них будут у родительского компонента, который будет на вершине этого дерева компонентов. Он и будет передавать данные дочерним узлам через `props`. Этот компонент с общим состоянием содержит в себе всю логику взаимодействия, а дочерние компоненты будут только отрисовывать данные, которые будут у них в `props`.

## Какие данные *надо* помещать в состояние?

**Состояние должно содержать данные, которые нужны для обновления интерфейса.** В реальных приложениях такие данные, как правило, незначительны по объему, и могут быть сериализованы в JSON. Когда вы создаете компонент с состоянием, старайтесь поместить в него минимум данных. А уже внутри метода `render()` вычисляйте остальные данные, используя значения из состояния.
Со временем вы увидите, что такой подход позволяет создавать более стройные и устойчивые к изменениям приложения. Добавление в состояние лишних данных требует от вас дополнительных затрат на их синхронизацию. Но этого можно избежать, если позволить React делать все эти вычисления за вас.

## Какие данные *не надо* помещать в состояние?

Состояние `this.state` должно содержать минимум данных, необходимых  для отображения интерфейса. Поэтому не стоит хранить в нем:

* **Вычисляемые данные:** Не волнуйтесь о данных, которые можно вычислить из состояния. Согласованность данных проще обеспечить, если производить все вычисления в методе `render()`. Например, если в состоянии хранится список элементов, и вам надо вывести его размер в виде строки, напишите `this.state.listItems.length + ' элементов'` в методе `render()`. Это будет правильнее, чем хранить размер списка в состоянии.
* **Компоненты React:** Создавайте их в методе `render()`, опираясь на данные из `props` и `state`.
* **Значения, повторяющие `props`:** Старайтесь по мере возможности использовать `props` как единственный источник данных. Хранить значения `props` в состоянии допускается, только если вам надо где-то хранить их прошлые значения, ведь `props` могут измениться после отрисовки родительского компонента.
