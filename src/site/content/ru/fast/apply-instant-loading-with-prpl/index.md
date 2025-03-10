---
layout: post
title: Применение мгновенной загрузки с помощью шаблона PRPL
authors:
  - houssein
description: PRPL — это аббревиатура, описывающая шаблон, который используется для ускорения загрузки и перехода к интерактивности веб-страниц. Из этого руководства вы узнаете, как использовать каждый из этих методов по отдельности и сочетать их друг с другом для улучшения производительности.
date: 2018-11-05
tags:
  - performance
---

PRPL — это аббревиатура, описывающая шаблон, который используется для ускорения загрузки и перехода к интерактивности веб-страниц:

- **Push** — отправка, или **preload** — предварительная загрузка, наиболее важных ресурсов.
- **Render** — обработка начального маршрута в кратчайшие сроки.
- **Pre-cache** — предварительное кеширование оставшихся ресурсов.
- **Lazy load** — отложенная загрузка других маршрутов и некритических ресурсов.

Из этого руководства вы узнаете, как использовать каждый из этих методов по отдельности и сочетать их друг с другом для улучшения производительности.

## Аудит вашей страницы в Lighthouse

Запустите Lighthouse, чтобы определить возможности для улучшения в соответствии с методами PRPL:

{% Instruction 'devtools-lighthouse', 'ol' %}

1. Установите метки **«Производительность»** и «**Прогрессивное веб-приложение**».
2. Щелкните «**Выполнить аудит**», чтобы создать отчет.

Дополнительные сведения см. в разделе «[Откройте для себя возможности повышения производительности с помощью Lighthouse](/discover-performance-opportunities-with-lighthouse)».

## Предварительная загрузка критических ресурсов

Следующий аудит Lighthouse завершается неудачей, если определенный ресурс анализируется и загружается слишком поздно:

{% Img src="image/admin/tgcMfl3HJLmdoERFn7Ji.png", alt="Lighthouse: аудит ключевых запросов на предварительную загрузку", width="745", height="97" %}

[**Предварительная загрузка**](https://developer.mozilla.org/docs/Web/HTML/Preloading_content) — это декларативный запрос на выборку, который сообщает браузеру как можно скорее запросить ресурс. Предварительно загрузите критически важные ресурсы, добавив тег `<link>` с атрибутом `rel="preload"` в заголовок своего HTML-документа:

```html
<link rel="preload" as="style" href="css/style.css">
```

Браузер устанавливает подходящий уровень приоритета для ресурса, чтобы попытаться загрузить его раньше, не задерживая событие `window.onload`.

Дополнительные сведения о предварительной загрузке критических ресурсов см. в руководстве «[Предварительная загрузка критических ресурсов](/preload-critical-assets)».

## Обработка начального маршрута в кратчайшие сроки

Lighthouse выдает предупреждение, если есть ресурсы, которые задерживают [**First Paint**](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics#first_paint_and_first_contentful_paint), момент, когда ваш сайт отображает пиксели на экране:

{% Img src="image/admin/gvj0jlCYbMdpLNtHu0Ji.png", alt="Lighthouse: аудит устранения ресурсов, блокирующих рендеринг", width="800", height="111" %}

Чтобы улучшить показатель First Paint, Lighthouse рекомендует встраивать критический JavaScript и откладывать остальное с помощью [`async`](/critical-rendering-path-adding-interactivity-with-javascript/), а также встраивать критический CSS, используемый в верхней части страницы. Это повышает производительность за счет исключения циклических обращений к серверу для получения ресурсов, блокирующих рендеринг. Однако встроенный код сложнее поддерживать с точки зрения разработки, и браузер не может кешировать его отдельно.

Другой подход к улучшению показателя First Paint — это **рендеринг на стороне сервера** исходного HTML-кода вашей страницы. С этим подходом контент будет отображен для пользователя немедленно, пока скрипты все еще выбираются, анализируются и выполняются. Однако это может значительно увеличить полезную нагрузку HTML-файла, что может увеличить [**Time to Interactive**](/tti/), т.е. время, необходимое для того, чтобы ваше приложение стало интерактивным и могло реагировать на ввод пользователя.

Не существует единого правильного решения для уменьшения First Paint в вашем приложении, и следует рассматривать встраивание стилей и рендеринг на стороне сервера только в том случае, если польза перевешивает потенциальный риск для вашего приложения. Вы можете узнать больше об обеих концепциях в следующих ресурсах.

- [Оптимизация доставки CSS](https://developers.google.com/speed/docs/insights/OptimizeCSSDelivery)
- [Что такое рендеринг на стороне сервера?](https://www.youtube.com/watch?v=GQzn7XRdzxY)

<figure data-float="right">{% Img src="image/admin/xv1f7ZLKeBZD83Wcw6pd.png", alt="Запросы/ответы с сервис-воркером", width="800", height="1224" %}</figure>

## Предварительное кеширование ресурсов

Действуя как прокси, **сервис-воркеры** могут при повторных посещениях извлекать ресурсы непосредственно из кеша, а не с сервера. Это не только позволяет пользователям работать с вашим приложением вне сети, но и сокращает время загрузки страницы при повторных посещениях.

Используйте стороннюю библиотеку, чтобы упростить процесс создания сервис-воркера, если ваши требования к кешированию не выходят за рамки возможностей библиотеки. Например, [Workbox](/workbox) предоставляет набор инструментов, которые позволяют создавать и поддерживать сервис-воркер для кеширования ресурсов. Для получения дополнительной информации о сервис-воркерах и бесперебойной работе вне сети обратитесь к [руководству по сервис-воркерам](/service-workers-cache-storage) в рамках программы обучения надежности.

## Отложенная загрузка

Аудит Lighthouse завершается неудачей, если вы отправляете слишком много данных по сети.

{% Img src="image/admin/Ml4hOCqfD4kGWfuKYVTN.png", alt="Lighthouse: аудит чрезмерной сетевой нагрузки", width="800", height="99" %}

Сюда входят все типы ресурсов, но большие полезные данные JavaScript особенно дороги из-за того, что браузеру требуется время, чтобы проанализировать и скомпилировать их. В соответствующих случаях Lighthouse также предупреждает об этом.

{% Img src="image/admin/aKDCV8qv3nuTVFt0Txyj.png", alt="Lighthouse: аудит времени загрузки JavaScript", width="797", height="100" %}

Чтобы отправить меньший пакет JavaScript, который содержит только код, необходимый при первоначальной загрузке вашего приложения пользователем, разделите весь пакет и  [загрузите фрагменты](/reduce-javascript-payloads-with-code-splitting) по запросу.

После того, как вам удалось разделить пакет, предварительно загрузите более важные фрагменты (см. руководство по [предварительной загрузке критических ресурсов](/preload-critical-assets)). Предварительная загрузка гарантирует, что более важные ресурсы будут извлечены и загружены браузером раньше.

Помимо разделения и загрузки различных фрагментов JavaScript по запросу, Lighthouse также предоставляет аудит отложенной загрузки некритических изображений.

{% Img src="image/admin/sEgLhoYadRCtKFCYVM1d.png", alt="Lighthouse: аудит откладывания закадровых изображений", width="800", height="90" %}

Если вы загружаете много изображений на свою веб-страницу, отложите все, что находится ниже первого экрана или вне области просмотра устройства, когда страница загружается (см. «[Использование lazysizes для отложенной загрузки изображений](/use-lazysizes-to-lazyload-images)»).

## Следующие шаги

Теперь, когда вы понимаете некоторые из основных концепций, лежащих в основе шаблона PRPL, перейдите к следующему руководству в этом разделе, чтобы узнать больше. Важно помнить, что не все техники нужно применять вместе. Использование любой из следующих методик обеспечит заметное улучшение производительности.

- **Push** — отправка, или **preload** — предварительная загрузка, критических ресурсов.
- **Render** — обработка начального маршрута в кратчайшие сроки.
- **Pre-cache** — предварительное кеширование оставшихся ресурсов.
- **Lazy load** — отложенная загрузка других маршрутов и некритических ресурсов.
