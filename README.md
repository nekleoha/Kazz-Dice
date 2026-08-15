# Kazz Dice

**Kazz Dice** — активно разрабатываемая instant-игра с проверяемой математикой,
адаптивным Svelte-интерфейсом и интеграционным слоем для Stake Engine/RGS.

Проект уже прошёл локальную инженерную проверку: математические и контрактные
тесты, статистические прогоны, production-сборку и browser smoke-test. Этот
документ фиксирует реально выполненную работу и публичную границу проекта, не
раскрывая генерацию исходов, серверное разрешение раундов и формулы выплат.

> **Статус:** working prototype / pre-release. Проект не заявляется как
> сертифицированная real-money игра. Проверка в реальном Stake sandbox и
> регуляторная сертификация ещё впереди.

## Что это за игра

Instant-игра на диапазонах: игрок выбирает окно вероятности, делает бросок и получает выплату, если результат попал внутрь окна. Чем уже окно — тем выше множитель.

Доступно 10 предустановленных вероятностей от **0.1%** до **77.2%**. Целевой RTP одинаков для всех режимов и равен **96.5%** — выбор окна меняет профиль риска, но не математическое ожидание.

### Режимы

**Play** — одиночный бросок. Три уровня объёма: Base (×1, один бросок), Boost (×10, десять бросков), Super Boost (×25, двадцать пять бросков).

**Run** — лестница с накоплением. Каждое попадание удваивает множитель, один промах сжигает банк. В любой момент можно забрать накопленное — выбор между жадностью и фиксацией прибыли.

**Bullseye** — бонусный режим с покупкой входа. Серия из 25 бросков по мишени, где близость к центру определяет множитель.

## Интерфейс

### Play — одиночный бросок

<img src="docs/screenshots/play-mode.png" width="420">

Окно 25.88–74.13 при вероятности 48.25%, выплата 1.69×. Сверху — история последних раундов, снизу — выбор объёма и размера ставки.

### Run — лестница с фиксацией

<img src="docs/screenshots/run-mode.png" width="420">

Третий шаг серии: множитель зафиксирован на ×4.00, следующее попадание даёт ×8.00. Кнопка Cash Out доступна до броска — решение принимается каждый шаг.

<img src="docs/screenshots/run-cashout.png" width="420">

Результат фиксации: забрано ×4.00 после двух успешных шагов.

### Bullseye — бонусный режим

<img src="docs/screenshots/bonus-mode.png" width="420">

Серия из 25 бросков по мишени с накоплением множителя.

## Продуктовые решения

**Единый RTP для всех окон вероятности.** Игрок выбирает не «выгодность», а профиль риска: редкие крупные выплаты против частых мелких. Это снимает необходимость искать «правильный» режим и делает выбор вопросом стиля, а не расчёта.

**Cash Out в режиме Run доступен на каждом шаге.** Ценность режима именно в решении, а не в броске: без возможности забрать банк лестница превращается в ожидание неизбежного промаха.

**Три уровня объёма вместо ручного повторения.** Boost и Super Boost решают задачу игрока, который хочет сыграть серию, не нажимая кнопку двадцать пять раз.

**Явная маркировка прототипа.** Бейдж FUN в шапке и дисклеймер в подвале: игра работает на условном балансе, не на реальных деньгах. Позиционирование зафиксировано в интерфейсе, а не только в документации.

## Моя роль

<!-- ОТРЕДАКТИРУЙ под реальное распределение работы в команде -->

Продуктовая и математическая часть: проектирование механик и режимов, расчёт вероятностей и выплат под целевой RTP, балансировка параметров, планирование этапов разработки по принципу «сначала аудит текущего состояния, затем инкремент», ведение чек-листа готовности к релизу.

## Проверенный статус проекта

| Область | Фактический результат | Статус |
|---|---:|:---:|
| Python test suite | 265 тестов | ✅ |
| Stake conformance suite | 24 проверки | ✅ |
| Frontend/Vitest | 14 тестов в 5 файлах | ✅ |
| Monte Carlo | 5 запусков × 1 000 000 итераций | ✅ |
| Целевой RTP | 96.5000% | ✅ |
| Math package | 11 режимов, 23 артефакта | ✅ |
| Контрактный слой | 13 JSON Schema | ✅ |
| Frontend production build | 4 статических файла, около 500 KB | ✅ |
| Desktop/mobile smoke-test | без console errors и horizontal overflow | ✅ |
| Stake sandbox publication | требуется внешний прогон | ⏳ |

## Что уже реализовано

### Игровой клиент

- Svelte 5 + TypeScript + Vite;
- основной instant-game flow и режимы Base, Boost, Super, Run и Bullseye;
- адаптивная компоновка для desktop и mobile;
- ручная игра и автоматические последовательности;
- обработка loading/error/disabled-состояний;
- аудио, локализация и корректное завершение пользовательской сессии;
- отделение UI от транспорта и игрового движка;
- статическая production-сборка с относительными путями к ресурсам.

### Math и статистическая проверка

- единый целевой RTP **0.965**;
- 10 базовых width-режимов проверены полным перечислением;
- отдельный Bullseye math-артефакт и characterisation-прогон;
- экспортирован пакет Stake math: **23 файла для 11 режимов**;
- deterministic unit/integration tests;
- Monte Carlo gates для сравнения наблюдаемого RTP с целевым;
- проверки денежных границ, структуры результатов и инвариантов раунда.

### Stake Engine / RGS boundary

Реализован отдельный адаптер, который принимает launch-параметры провайдера и
поддерживает жизненный цикл сессии и раунда через публичную границу RGS. В
локальной проверке покрыты:

- аутентификация игровой сессии;
- запрос конфигурации и юрисдикционных ограничений;
- отправка игрового действия и завершение раунда;
- обработка provider errors и повторных запросов;
- восстановление согласованного состояния клиента;
- преобразование игровых режимов в провайдерский контракт;
- защита от запуска игры до готовности внешней сессии.

Провайдер остаётся источником истины для сессии, конфигурации и разрешённых
действий. Реальные endpoint, credentials и сетевые traces в публичную версию не
входят.

## Результаты RTP-проверки

### Базовые режимы

Все 10 базовых width-режимов были проверены полным перечислением допустимого
пространства результатов. Для каждого режима рассчитанный RTP равен целевому:
**96.5000%**.

### Monte Carlo

Для stochastic-сценария выполнено пять независимых прогонов по одному миллиону
итераций. Публичный отчёт содержит только агрегаты и не раскрывает seeds, веса,
таблицы выплат или последовательности исходов.

| Метрика | Результат |
|---|---:|
| Общий объём | 5 000 000 итераций |
| Число независимых прогонов | 5 |
| Целевой RTP | 96.5000% |
| Минимальный observed RTP | 95.56395% |
| Максимальный observed RTP | 96.76698% |
| Средний observed RTP | 96.23495% |
| Неуспешные acceptance gates | 0 |
| Итог | **PASS** |

Для Bullseye дополнительно выполнен characterisation-прогон на **120 000
раундов**. Наблюдаемое среднее составило **96.80%**; итоговый опубликованный
набор скорректирован от sampling noise до целевых **96.50%**. Это инженерная
статистическая проверка, а не замена независимому PAR sheet или сертификации.

## Публичный статистический слой

Проверка работает с обезличенным агрегированным отчётом. Она не импортирует
закрытый движок, не генерирует исходы и не вычисляет выплаты.

```python
from dataclasses import dataclass
from math import isfinite


@dataclass(frozen=True)
class RtpReport:
    sample_size: int
    target_rtp: float
    observed_rtp: float
    tolerance: float


def verify_rtp(report: RtpReport) -> None:
    assert report.sample_size > 0
    assert isfinite(report.target_rtp)
    assert isfinite(report.observed_rtp)
    assert 0.0 < report.target_rtp <= 1.0
    assert report.tolerance > 0.0
    assert abs(report.observed_rtp - report.target_rtp) <= report.tolerance
```

Публичный fixture должен содержать только размер выборки, целевое и наблюдаемое
значения, tolerance и итог проверки. В него не попадают внутренние книги,
randomness material, payout tables или данные пользователей.

## Conformance-проверки

Conformance suite проверяет наблюдаемую форму и поведение внешней границы, не
заглядывая в реализацию раунда. В текущем проекте покрыты:

- обязательные поля, типы и версии контрактов;
- жизненный цикл сессии и раунда;
- нормализованные provider errors;
- idempotency/retry-сценарии на публичной границе;
- целостность и формат release-артефактов;
- соответствие frontend-режимов опубликованным math-режимам;
- отсутствие приватных полей в публичном отчёте;
- поведение при недоступном или неполном RGS-контексте.

Минимальная публичная проверка артефактного manifest:

```python
REQUIRED_FIELDS = {"schemaVersion", "buildId", "mathVersion", "artifacts"}
PRIVATE_FIELDS = {
    "seed",
    "nonce",
    "weights",
    "payoutTable",
    "balance",
    "transactionId",
}


def verify_public_manifest(manifest: dict) -> None:
    assert REQUIRED_FIELDS <= manifest.keys()
    assert manifest["schemaVersion"]
    assert manifest["buildId"]
    assert isinstance(manifest["artifacts"], list)
    assert not (PRIVATE_FIELDS & manifest.keys())
```

## Build configuration

Frontend собирается как self-contained static package: ресурсы лежат рядом с
`index.html`, используются относительные URL, source maps для release выключены.
Публично безопасная часть конфигурации выглядит так:

```ts
import { svelte } from "@sveltejs/vite-plugin-svelte";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [svelte()],
  publicDir: false,
  base: "./",
  build: {
    assetsDir: "",
    sourcemap: false,
  },
  test: {
    environment: "node",
    include: ["src/**/*.test.ts"],
  },
});
```

Production build выполняет не только bundling, но и обязательные quality gates:

```json
{
  "scripts": {
    "test": "vitest run",
    "build": "npm run check:boundary && npm run check:range-fixtures && npm run check:play-guard && npm run test && vite build"
  }
}
```

Проверяются архитектурная граница, соответствие range fixtures, защита игрового
действия до готовности сессии, frontend tests и только затем Vite build.

## Continuous Integration

CI воспроизводит backend, Stake-conformance и frontend-проверки на чистом runner:

```yaml
name: Verify

on:
  push:
  pull_request:

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
          cache-dependency-path: frontend/package-lock.json
      - name: Python unit suite
        run: python3 math/run_tests.py
      - name: Stake conformance suite
        run: python3 math/run_conformance_tests.py
      - name: Install frontend dependencies
        working-directory: frontend
        run: npm ci
      - name: Frontend checks and build
        working-directory: frontend
        run: npm run build
```

## Архитектурная граница

```text
┌──────────────────────┐
│  Svelte game client  │
└──────────┬───────────┘
           │ versioned public contract
┌──────────▼───────────┐
│   Stake/RGS adapter  │
└──────────┬───────────┘
           │ provider protocol
┌──────────▼───────────┐
│ Operator / Stake RGS │
└──────────────────────┘

Private boundary:
randomness · round resolution · payout math · financial operations · secrets
```

Клиент не является источником истины для результата раунда. Закрытый игровой
слой изолирован от UI и от публичных statistical/conformance fixtures.

## Что можно показать публично

- агрегированные statistical/RTP checks;
- black-box conformance tests;
- публичные JSON Schema после security review;
- описание подхода к верификации;
- release-readiness checklist;
- нейтральные build/test/CI-конфиги;
- скриншоты интерфейса без session data и внутренних endpoint.

## Что остаётся закрытым

- реализация генерации случайности и randomness material;
- серверное разрешение раунда и весь внутренний round flow;
- формулы выплат, точные веса и probability tables;
- books и исходные math-артефакты, позволяющие восстановить модель;
- работа с балансом, расчётами и транзакциями;
- session identifiers, ключи, токены, endpoint и конфиги окружения;
- сырые отчёты и fixtures с чувствительными внутренними данными.

Эта граница позволяет продемонстрировать качество разработки и воспроизводимость
проверок, не превращая публичный репозиторий в копию production-движка.

## Release readiness checklist

### Выполнено

- [x] Целевой RTP зафиксирован и проверяется автоматически.
- [x] Базовые режимы проверены полным перечислением.
- [x] Monte Carlo acceptance gates пройдены на 5 млн итераций.
- [x] Python unit/integration suite проходит полностью.
- [x] Stake conformance suite проходит полностью.
- [x] Frontend tests и pre-build guards проходят полностью.
- [x] Math-режимы и frontend mappings сверяются автоматически.
- [x] Контракты версионированы через JSON Schema.
- [x] Реализован отдельный Stake/RGS transport boundary.
- [x] Production frontend собирается в self-contained static package.
- [x] Выполнен desktop/mobile browser smoke-test.
- [x] CI запускает полную проверку на чистом окружении.
- [x] Release build не содержит source maps.
- [x] Приватный engine отделён от публичного verification layer.

### До внешнего релиза

- [ ] Опубликовать новую math/frontend version в Stake ACP.
- [ ] Пройти реальный Stake sandbox launch flow.
- [ ] Проверить recovery активного раунда на provider environment.
- [ ] Зафиксировать финальные Rules/Paytable-тексты в UI.
- [ ] Провести security review и secret scan публичного пакета.
- [ ] Получить независимый PAR/certification review, если он требуется рынком.
- [ ] Пройти operator acceptance и jurisdiction-specific проверки.

## Локальная воспроизводимость

Полный приватный репозиторий проверяется следующими командами:

```bash
python3 math/run_tests.py
python3 math/run_conformance_tests.py

cd frontend
npm ci
npm run build
```

Успешный `npm run build` означает, что boundary checks, fixture checks,
play-guard, Vitest и production bundling завершились без ошибок.

## Текущий итог

Kazz — уже не набор макетов: проект имеет работающий клиент, сформированный
math package, автоматические статистические и контрактные проверки, отдельную
RGS-границу, воспроизводимую CI-сборку и зафиксированный путь до внешнего релиза.

Следующий крупный milestone — публикация версии и end-to-end проверка в реальном
Stake sandbox. До этого момента проект корректно позиционируется как
**проверенный локально pre-release prototype**, а не как сертифицированный продукт.
