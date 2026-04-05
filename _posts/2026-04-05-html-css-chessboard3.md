---
layout: post
title: "HTML, CSS: шахматное поле<br> с помощью Flexbox, часть 3"
---

В предыдущих двух постах этой серии постов я описал создание простого шахматного поля ([1]({% post_url 2026-03-18-html-css-chessboard %})) на веб-странице с помощью языка разметки HTML, языка описания стилей CSS и технологии Flexbox, а также добавил в проект не обязательные, но яркие штрихи: текстуры дерева для клеток шахматного поля, буквенно-цифровые обозначения для столбцов и рядов ([2]({% post_url 2026-03-22-html-css-chessboard2 %})). В этом посте я опишу добавление на поле шахматных фигур.

## Набор изображений фигур

Не слишком сложно самому нарисовать нужные изображения фигур. Но в интернете есть большое количество готовых наборов, мы выбирали среди них. Студенты нашли в блоге некоего Николая [пост](http://blog.kislenko.net/show.php?id=1860), из которого можно загрузить архив сразу с 47 разными наборами картинок с фигурами. Картинки имеют квадратные размеры 80&nbsp;×&nbsp;80 пикселей, фон&nbsp;— прозрачный, формат&nbsp;— PNG.

Позже я обнаружил, что у Николая есть ЖЖ ([pers-narod-ru.livejournal.com](https://pers-narod-ru.livejournal.com)), в который настроена автоматическая трансляция анонсов постов из его отдельного блога. Сам он пишет, что бывает в ЖЖ редко.

Блог Николая ([blog.kislenko.net](http://blog.kislenko.net)) и вышеуказанный пост не всегда доступны. Например, сегодня я проверял его доступность и в какой-то момент на все попытки входа получил в браузере ошибку `502 Bad Gateway`. Позже доступ появился. В случае подобных проблем можно воспользоваться Архивом Интернета ([ссылка на нужную страницу](https://web.archive.org/web/20250327002821/http://blog.kislenko.net/show.php?id=1860) в архиве). К сожалению, ZIP-архив с наборами картинок в Архиве Интернета недоступен, на него есть только ссылка.

Мы выбрали первый набор из доступных в файле-архиве наборов, он называется «adventurer». Названия файлов изображений состоят из двух букв английского алфавита и расширения `.png`. Из этих двух букв первая обозначает цвет фигуры (`b` или `w`, то есть *black* или *white*), вторая&nbsp;— название фигуры по [шахматной нотации](https://ru.wikipedia.org/wiki/%D0%A8%D0%B0%D1%85%D0%BC%D0%B0%D1%82%D0%BD%D0%B0%D1%8F_%D0%BD%D0%BE%D1%82%D0%B0%D1%86%D0%B8%D1%8F). Я собрал информацию по выбранным изображениям в следующую табличку.

| Файлы | Изображения | Chess piece | Фигура |
| :---: | :---: | :---: | :---: |
| bK.png<br>wK.png | ![](/images/adventurer/bK.png) ![](/images/adventurer/wK.png) | King | Король |
| bQ.png<br>wQ.png | ![](/images/adventurer/bQ.png) ![](/images/adventurer/wQ.png) | Queen | Ферзь |
| bN.png<br>wN.png | ![](/images/adventurer/bN.png) ![](/images/adventurer/wN.png) | Knight | Конь |
| bB.png<br>wB.png | ![](/images/adventurer/bB.png) ![](/images/adventurer/wB.png) | Bishop | Слон |
| bR.png<br>wR.png | ![](/images/adventurer/bR.png) ![](/images/adventurer/wR.png) | Rook | Ладья |
| bP.png<br>wP.png | ![](/images/adventurer/bP.png) ![](/images/adventurer/wP.png) | Pawn | Пешка |

## Вставка фигур на поле

Фигуры вставим в HTML с помощью элемента `img`. Само поле было создано и подготовлено для фигур в предыдущих двух постах этой серии постов. Пример вставки фигур на восьмой ряд поля:

```html
      <!-- 8 ряд -->
      <div class="chess-field-row">
        <div class="border-cell digit left-digit">8</div>
        <!-- a8 -->
        <div class="square white-square">
          <img src="images/adventurer/bR.png" class="chess-piece">
        </div>
        <!-- b8 -->
        <div class="square black-square">
          <img src="images/adventurer/bN.png" class="chess-piece">
        </div>
        <!-- c8 -->
        <div class="square white-square">
          <img src="images/adventurer/bB.png" class="chess-piece">
        </div>
        <!-- d8 -->
        <div class="square black-square">
          <img src="images/adventurer/bQ.png" class="chess-piece">
        </div>
        <!-- e8 -->
        <div class="square white-square">
          <img src="images/adventurer/bK.png" class="chess-piece">
        </div>
        <!-- f8 -->
        <div class="square black-square">
          <img src="images/adventurer/bB.png" class="chess-piece">
        </div>
        <!-- g8 -->
        <div class="square white-square">
          <img src="images/adventurer/bN.png" class="chess-piece">
        </div>
        <!-- h8 -->
        <div class="square black-square">
          <img src="images/adventurer/bR.png" class="chess-piece">
        </div>
        <div class="border-cell digit right-digit">8</div>
      </div>
```

Как можно видеть из кода выше, все файлы изображений фигур я храню в подпапке `adventurer` подпапки `images` папки проекта.

В описание стилей на языке CSS добавим стиль `chess-piece` (размеры изображений в оригинале уже равны 80&nbsp;×&nbsp;80 пикселей, но я решил все равно добавить свойство `width` для наглядности; высоту указывать не обязательно, браузер по умолчанию старается масштабировать изображения с сохранением пропорциональности размеров):

```css
.chess-piece {
  width: 80px;             /* размер фигур */
}
```

И внесем описание нескольких новых свойств в стиль `square` (таким образом мы выровняем фигуры внутри клеток поля по горизонтали и вертикали):

```css
.square {
  display: flex;           /* для фигур: */
  justify-content: center; /*   в центр по горизонтали */
  align-items: center;     /*   в центр по вертикали */
  /* ... */
}
```

## Результат

Вот как полученное шахматное поле с расставленными фигурами выглядит в окне моего браузера:

![](/images/chess-field-in-browser2-1.webp)

Можно посмотреть онлайн: [https://textpub.neocities.org/my/chess-board/v2.1/](https://textpub.neocities.org/my/chess-board/v2.1/).

## Развитие проекта

Если найду время, добавлю движение фигур с помощью мыши, сохранение позиции в файл и загрузку позиции из файла. Для этого уже нужно будет добавить на страницу сайта скрипты на языке JavaScript.