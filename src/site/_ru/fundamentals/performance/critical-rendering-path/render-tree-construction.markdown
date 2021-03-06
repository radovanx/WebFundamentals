---
layout: article
title: "Создание модели визуализации и макета, вывод страницы на экран"
description: "DOM и CSSOM образуют модель визуализации. С ее помощью браузер формирует из всех видимых объектов макет, а затем выводит пиксели на экран. Чтобы ускорить загрузку страницы, нужно оптимизировать каждый из этих этапов."
introduction: "DOM и CSSOM образуют модель визуализации. С ее помощью браузер формирует из всех видимых объектов макет, а затем выводит пиксели на экран. Чтобы ускорить загрузку страницы, нужно оптимизировать каждый из этих этапов."
article:
  written_on: 2014-04-01
  updated_on: 2014-09-18
  order: 2
collection: critical-rendering-path
authors:
  - ilyagrigorik
key-takeaways:
  render-tree-construction:
    - Браузер объединяет DOM и CSSOM, формируя модель визуализации.
    - Модель визуализации содержит только те объекты, которые нужны для вывода страницы на экран.
    - Далее формируется макет, отражающий расположение и размер каждого объекта.
    - Наконец, объекты выводятся на экран.
notes:
  hidden:
    - "Обратите внимание, что параметры `visibility: hidden` и `display: none` различаются. При использовании первого параметра объект занимает место в макете, но скрывается на странице (в результате вы можете увидеть, например, пустой квадрат). В то же время `display: none` указывает, что объект будет удален из макета и вообще не появится на странице."
---
{% wrap content%}

<style>
  img, video, object {
    max-width: 100%;
  }

  img.center {
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
</style>

В предыдущей главе мы узнали о том, как на основании HTML- и CSS-файлов строятся модели DOM и CSSOM. Эти независимые друг от друга модели отвечают за разные аспекты страницы: DOM описывает контент, а CSSOM - стили, которые будут к нему применены. Рассмотрим, как браузер объединяет эти модели и выводит страницу на экран.

{% include modules/takeaway.liquid list=page.key-takeaways.render-tree-construction %}

Для начала браузер формирует модель визуализации, в которой всем видимым объектам из модели DOM присваиваются наборы стилей из модели CSSOM.

<img src="images/render-tree-construction.png" alt="Объединение DOM и CSSOM в модель визуализации" class="center">

Для формирования модели визуализации браузер выполняет следующие действия:

1. Начиная с основания модели DOM, находит все видимые объекты.
  * Этот этап не затрагивает элементы, которые не будут видны на странице, например теги скриптов, метатеги и т. п.
  * Он также не затрагивает объекты, помеченные как невидимые с помощью CSS. Взгляните на риведенную выше схему: объект span отсутствует в модели визуализации, потому что ему присвоен параметр `display: none`.
2. Находит в CSSOM наборы стилей и присваивает их соответствующим объектам.
3. Формирует модель из видимых объектов, их содержания и стилей.

{% include modules/remember.liquid list=page.notes.hidden %}

Создав модель визуализации, браузер на шаг приблизился к выводу страницы на экран! **Теперь можно приступить к формированию макета.**

Браузер уже определил, какие объекты будут видны на странице и какие стили нужно им присвоить. Пришло время создать макет, т. е. выяснить, какого размера будут объекты и как их нужно расположить в [области просмотра]({{site.fundamentals}}/layouts/rwd-fundamentals/set-the-viewport.html).

Для этого браузер вычисляет геометрическую форму объектов, анализируя модель визуализации с самого начала. Рассмотрим простой пример:

{% include_code _code/nested.html full %}

В теле этой страницы есть два блока div. Ширина родительского блока - 50% от области просмотра, а вложенного - 50% от родительского, т. е. 25% экрана.

<img src="images/layout-viewport.png" alt="Определение размера элементов в макете" class="center">

Сформировав макет, браузер получает блочную модель, точно отражающую расположение и размер каждого объекта в области просмотра. Все относительные показатели преобразуются в абсолютное положение пикселей на экране.

Наконец, когда браузеру известно, какие объекты будут отображаться на странице, где их разместить и какие стили им нужно присвоить, можно приступать к следующему этапу - выводу страницы на экран. Этот этап также называется визуализацией или растеризацией.

Как вы наверняка заметили, браузеру нужно выполнить множество действий, на которые зачастую уходит немало времени. Чтобы оптимизировать время загрузки, продолжительность каждого этапа можно отслеживать с помощью инструментов разработчика в Chrome. Рассмотрим этап формирования макета на нашем первом примере Hello World:

<img src="images/layout-timeline.png" alt="Анализ формирования макета с помощью инструментов разработчика" class="center">

* Событие Layout на вкладке Timeline отражает время, затраченное на формирование макета, а также на определение положения и размера объектов.
* Сформировав макет, браузер переходит к этапам Paint Setup и Paint для преобразования модели визуализации в пиксели.

Время, затрачиваемое на создание модели визуализации и макета, а также на вывод пикселей, варьируется в зависимости от размера документа, сложности стилей и мощности устройства. Чем больше HTML-файл, тем больше работы нужно проделать браузеру. Чем сложнее стили, тем больше времени потребуется на визуализацию (например, текст с эффектом тени обработать сложнее, чем простой одноцветный).

Тем временем, визуализация завершилась, и наша страница появилась в области просмотра!

<img src="images/device-dom-small.png" alt="Страница Hello World, выведенная на экран" class="center">

Давайте вспомним, какие действия выполнил браузер:

1. Обработка HTML-разметки и создание модели DOM.
2. Обработка CSS-файла и создание модели CSSOM.
3. Создание модели визуализации из DOM и CSSOM.
4. Определение формы и расположения объектов, создание макета.
5. Вывод объектов на экран.

Вот что нужно проделать для визуализации даже такой простой страницы, как наша. А если мы внесем изменения в DOM или CSSOM, бразеру придется повторить все действия, чтобы определить, как вывести пиксели на экран.

**Чтобы оптимизировать процесс визуализации, нужно уменьшить время, затрачиваемое на выполнение пяти действий из списка выше.** В результате контент будет выводиться на экран быстрее, время обновления сократится, а вместе с этим возрастет частота обновления интерактивного контента.

{% include modules/nextarticle.liquid %}

{% endwrap%}

