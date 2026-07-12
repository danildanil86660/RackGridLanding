# RackGrid — лендинг

Публичная посадочная страница продукта **RackGrid** (система управления майнинг-фермой на своём сервере).

**Live:** https://danildanil86660.github.io/RackGridLanding/

## Что это

Статический сайт, экспортированный из дизайн-инструмента (`*.dc.html`). Страница рендерится в браузере рантаймом `support.js` (dc-runtime): он парсит разметку внутри `<x-dc>`, компилирует встроенный JSX-компонент (живая телеметрия, переключатель темы) и монтирует всё через React.

## Структура

| Путь | Назначение |
|------|-----------|
| `index.html` | Страница (экспорт `RackGrid.dc.html` дизайнера + добавленные `<title>`/meta/OG/favicon и хук `window.__resources`) |
| `support.js` | dc-runtime (сгенерирован, **не редактировать вручную**) |
| `assets/` | Скриншоты: `shot-*-{dark,light}.webp` (десктоп, пары под тему) + `phone-*.png` (мобайл) |
| `vendor/` | Самостоятельно захостенные React 18.3.1, ReactDOM 18.3.1, Babel standalone 7.29.0 |
| `.nojekyll` | Отключает обработку Jekyll на GitHub Pages |

## Зависимости рантайма — без внешнего CDN

По умолчанию `support.js` грузит React/ReactDOM/Babel с `unpkg.com`. Чтобы сайт не зависел от внешнего CDN, они **захостены локально** в `vendor/`, а `index.html` до подключения `support.js` задаёт:

```html
<script>
window.__resources = {
  "https://unpkg.com/react@18.3.1/umd/react.production.min.js": "vendor/react.production.min.js",
  "https://unpkg.com/react-dom@18.3.1/umd/react-dom.production.min.js": "vendor/react-dom.production.min.js",
  "https://unpkg.com/@babel/standalone@7.29.0/babel.min.js": "vendor/babel.min.js"
};
</script>
```

`support.js` (`cdnScriptFor`) читает эту карту и грузит скрипты по локальным путям без `integrity`. Единственная оставшаяся внешняя ссылка — шрифты Google Fonts (Inter, JetBrains Mono); при их недоступности идёт graceful-fallback на системные шрифты.

> ⚠️ Babel компилирует JSX в браузере при каждой загрузке (dev-инструмент). Для лендинга это приемлемо, но при желании ускорить первую отрисовку — прекомпилировать компонент в build-пайплайне дизайн-инструмента.

## Деплой (GitHub Pages)

Сайт лежит в корне репозитория, поэтому используется штатный **Deploy from a branch**:

1. Settings → Pages → Source = **Deploy from a branch**, ветка `main`, папка `/ (root)`.
2. Push в `main` → Pages пересобирает сайт автоматически.

(В отличие от [RackGridLiveDemo](https://danildanil86660.github.io/RackGridLiveDemo/), где сайт в `/dist` и нужен кастомный Actions-workflow, здесь всё в корне — кастомный workflow не требуется.)

## Обновление лендинга из нового экспорта дизайнера

При новом экспорте `RackGrid.dc.html` + `support.js` + `assets/`:

1. Заменить `support.js` и `assets/`.
2. Скопировать новый `RackGrid.dc.html` в `index.html` и заново добавить в `<head>` блок `<title>`/meta/OG/favicon и `window.__resources` (см. выше) перед `<script src="./support.js">`.
3. Если сменилась версия React/Babel в `support.js` (`REACT_URL`/`BABEL_URL`) — обновить `vendor/` и ключи в `window.__resources`.
