# Lab 5 — Архітектура: Component Diagram

Онлайн-книгарня. UML Component Diagram у PlantUML: компоненти, provided/required-інтерфейси, залежності, межа системи й зовнішні системи.

## Структура

| Файл | Призначення |
|---|---|
| `spec.md` | Специфікація: компоненти, інтерфейси, рішення, критерії прийняття |
| `model/components.puml` | Джерело діаграми (PlantUML) |
| `model/components.svg` | Рендер діаграми |
| `adr/0001-architecture-style.md` | Ключове рішення: архітектурний стиль |
| `ai/prompt-1..5.md` | Слід роботи з AI (промпти й відповіді) |
| `DEFENSE.md` | Захист: розбіжності, ключове рішення, перевірка |

## Як відрендерити

SVG створено з `model/components.puml` рушієм PlantUML через Kroki:

​```powershell
curl.exe -X POST -H "Content-Type: text/plain" --data-binary "@Lab5/model/components.puml" https://kroki.io/plantuml/svg -o Lab5/model/components.svg
​```

Локально: `java -jar plantuml.jar -tsvg Lab5/model/components.puml` (потрібні Java 11+ і Graphviz).

## Зв'язок з попередніми роботами

- Lab1/Lab2: сутності ER (`Lab2/model/er.md`) і вимоги FR-01–FR-12
- Lab3: Sequence SC-09–SC-12 (назви операцій)
- Lab4: стани й процес оплати