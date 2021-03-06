---
layout: section
title: "Процесс визуализации"
description: "Научитесь выбирать оптимальный порядок отображения контента в зависимости от его важности для пользователя."
introduction: "Оптимизация помогает ускорить загрузку страниц. Наша цель - выбрать оптимальный порядок отображения контента в зависимости от его важности для пользователя."
article:
  written_on: 2014-04-01
  updated_on: 2014-04-28
  order: 1
id: critical-rendering-path
collection: performance
authors:
  - ilyagrigorik
---
{% wrap content%}

При визуализации страниц браузер проделывает огромную работу, которую мы, веб-разработчики, обычно не замечаем. Мы не знаем, как разметка, стили и скрипты превращаются в страницы на экране.

Об этом и пойдет речь в наших уроках. Вы познакомитесь со всеми этапами **визуализации** - от скачивания HTML, CSS и JavaScript до вывода пикселей - и узнаете, как оптимизировать этот процесс.

<img src="images/progressive-rendering.png" class="center" alt="Оптимизированная и неоптимизированная визуализация">

Оптимизация помогает сократить время загрузки веб-страниц. Кроме того, изучив процесс визуализации, вы сможете создавать более эффективные интерактивные приложения, потому что они подчиняются тем же принципам (разве что обновление приложения происходит циклично и с частотой 60 кадров/сек. Однако не будем спешить. Сначала разберемся, как браузер визуализирует простую веб-страницу.

{% endwrap%}

