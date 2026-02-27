# Список изменений

## [Не выпущено]

### RootConfigManager

- Добавлены методы для пакетной записи в SharedPreferences (один `edit()` вместо нескольких):
  - `setCachedLatestReleaseInfo(tag, htmlUrl, etag)` — атомарно сохраняет данные последнего релиза GitHub
  - `setCachedRemoteVersionInfo(version, code, etag)` — атомарно сохраняет версию удалённого модуля
  - `setCachedAppUpdateResult(...)` — сохраняет все поля результата проверки обновления за одну транзакцию

### MainViewModel

- Несколько последовательных вызовов `prefs.edit()` заменены новыми пакетными методами из `RootConfigManager`
- Добавлен экспоненциальный backoff при ошибках в `startStatusPolling()` и `startDaemonLogPolling()`:
  - Задержки увеличиваются по схеме `2.2с → 4.4с → 8.8с → 17.6с`, максимум `30с`
  - Счётчик сбрасывается при успешном опросе
- Добавлен переопределённый метод `onCleared()` для корректного освобождения ресурсов при уничтожении ViewModel:
  - Отмена корутин `statusJob` и `daemonLogJob`
  - Остановка executor OkHttp dispatcher и очистка пула соединений

### HomeScreen

- Добавлен composable `AnimatedWebpImage`:
  - Использует `AnimatedImageDrawable` (API 28+) для воспроизведения анимированных WebP
  - На более старых версиях API или если изображение не анимировано — fallback на статичный `Image`
  - Инвалидация ограничена фазой отрисовки (без лишних рекомпозиций)
- Кнопка питания теперь использует `AnimatedWebpImage` в состоянии ВКЛ (`power_on.webp`) и статичный `Image` в состоянии ВЫКЛ

### StatsScreen

- Логика заморозки данных при скролле переписана с `LaunchedEffect` + `mutableStateOf` на `snapshotFlow` + `scan`:
  - Более идиоматичный подход для Compose
  - Устраняет лишние рекомпозиции — обновления откладываются до завершения скролла
