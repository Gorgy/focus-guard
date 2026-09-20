# FocusGuard MVP — план реализации

**Цель:** определить, что телефон находится в руке не менее двух секунд, и показать это состояние пользователю.

**Архитектура:** камера передаёт кадры циклу приложения; YOLO и MediaPipe независимо находят телефон и кисть; `InteractionDetector` определяет контакт, `HoldTimer` отслеживает две секунды, а `Renderer` показывает состояние. Задачи 1 и 2 создают пакет, источник кадров и временный вывод видео без моделей распознавания.

**Стек:** Python 3.11, OpenCV, Ultralytics YOLO, MediaPipe, NumPy, pytest.

Принятые решения: существующий Git-репозиторий сохраняется; корневой `main.py` мигрирует в пакет `src/focusguard/` и удаляется; `requirements.txt` заменяется на единственный источник зависимостей `pyproject.toml`; существующий `README.md` дополняется. Запуск выполняется командой `python -m focusguard`, выход из окна камеры — клавишей `q`.

Перед реализацией нужно убрать ранее добавленные пустые версии файлов из индекса, не удаляя содержимое рабочего дерева:

```bash
git status --short
git diff --cached --name-status
git diff --name-status
git restore --staged PLAN.md main.py requirements.txt
git status --short
```

Ожидаемый результат: `PLAN.md`, `main.py`, `requirements.txt` и `README.md` остаются в рабочем дереве, но в индексе нет подготовленных к коммиту файлов.

Сначала нужно сохранить документацию отдельным коммитом:

```bash
git add README.md PLAN.md
git diff --cached --name-status
git diff --cached --check
git commit -m "docs: add FocusGuard MVP plan"
```

Затем нужно зафиксировать исходный прототип, чтобы его миграция и удаление в задаче 1 были видны в истории:

```bash
git add main.py requirements.txt
git diff --cached --name-status
git diff --cached --check
git commit -m "chore: record initial OpenCV prototype"
git status --short
```

Ожидаемый итог: `git status --short` не выводит строк, а реализация начинается с чистого рабочего дерева. Во всех дальнейших коммитах используются только явные пути; `git add .` не применяется.

### Задача 1: привести существующий Python-проект к пакетной структуре

**Цель:** мигрировать текущий прототип в пакет FocusGuard на Python 3.11, оставить один источник зависимостей и получить проверяемую команду запуска.

**Файлы:**

- создать: `pyproject.toml`;
- создать: `src/focusguard/__init__.py`;
- создать: `src/focusguard/__main__.py`;
- создать: `src/focusguard/app.py`;
- создать: `tests/test_app.py`;
- изменить: `README.md`;
- проверить и при необходимости изменить: `.gitignore`;
- перенести и удалить: `main.py`;
- заменить и удалить: `requirements.txt`.

**Решение о миграции:**

- ответственность точки входа переносится из корневого `main.py` в `src/focusguard/app.py` и `src/focusguard/__main__.py`;
- применимая часть прототипа с OpenCV переносится в пакет, но чтение фиктивного пути `path/to/image` не сохраняется как поведение приложения;
- после переноса корневой `main.py` удаляется;
- версии `numpy` и `opencv-python` из `requirements.txt` переносятся в зависимости `pyproject.toml`;
- `pytest` объявляется dev-зависимостью в `pyproject.toml`;
- после успешной установки из `pyproject.toml` файл `requirements.txt` удаляется, чтобы не поддерживать два источника зависимостей;
- существующий `README.md` сохраняет описание MVP и дополняется командами установки, запуска и тестирования.

**Целевое дерево:**

```text
FocusGuard/
├── .gitignore
├── PLAN.md
├── README.md
├── pyproject.toml
├── src/
│   └── focusguard/
│       ├── __init__.py
│       ├── __main__.py
│       └── app.py
└── tests/
    └── test_app.py
```

**Интерфейс приложения:**

- `focusguard.app.main() -> int` — публичная точка входа;
- при штатном завершении `main()` возвращает `0`;
- ошибки камеры в задаче 2 будут преобразовываться в понятное сообщение в `stderr` и код возврата `1`;
- `focusguard.__main__` завершает процесс через `SystemExit(main())`;
- на этом этапе `main()` не открывает камеру и не создаёт окно.

**Команды подготовки:**

```bash
python3.11 --version
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Ожидаемый результат первой команды: `Python 3.11.x`.

**Шаги:**

- [ ] выполнить подготовительный блок и убедиться, что `git status --short` не выводит строк;
- [ ] проверить `.gitignore` и добавить только отсутствующие правила для `.venv/`, `__pycache__/`, `.pytest_cache/` и `*.pyc`;
- [ ] проверить Python 3.11 и создать виртуальное окружение `.venv`;
- [ ] создать `pyproject.toml`, перенеся в него зависимости из `requirements.txt` и добавив dev-зависимость `pytest`;
- [ ] создать каталоги `src/focusguard` и `tests`, затем создать только `src/focusguard/__init__.py`; `app.py` на этом шаге ещё не создавать;
- [ ] написать в `tests/test_app.py` smoke-тест, который импортирует `main()` и ожидает код `0`;
- [ ] установить проект в editable-режиме:

```bash
python -m pip install -e '.[dev]'
```

- [ ] запустить тест до создания `app.py`:

```bash
pytest tests/test_app.py -v
```

- [ ] увидеть ожидаемую ошибку `ModuleNotFoundError: No module named 'focusguard.app'`;
- [ ] только после полученной ошибки создать `src/focusguard/app.py`, перенести туда ответственность точки входа и добавить минимальную `main()`, которая возвращает `0`;
- [ ] подключить `main()` в `src/focusguard/__main__.py` через `SystemExit`;
- [ ] удалить корневой `main.py` после переноса точки входа;
- [ ] проверить установку из `pyproject.toml`, затем удалить `requirements.txt`;
- [ ] дополнить существующий `README.md` требованиями, командами создания окружения, установки, запуска и тестирования, не удаляя описание MVP;
- [ ] повторно запустить проверки:

```bash
pytest tests/test_app.py -v
pytest -q
python -m focusguard
```

- [ ] убедиться, что тесты имеют статус `PASSED`, а команда приложения завершается без traceback с кодом `0`;
- [ ] подготовить только файлы задачи и явно зафиксировать удаления:

```bash
test ! -e main.py
test ! -e requirements.txt
git add .gitignore README.md pyproject.toml src/focusguard/__init__.py src/focusguard/__main__.py src/focusguard/app.py tests/test_app.py
git add -u -- main.py requirements.txt
git diff --cached --name-status
git diff --cached --check
```

- [ ] убедиться, что `PLAN.md` и посторонние файлы не попали в индекс;
- [ ] сделать отдельный коммит:

```bash
git commit -m "chore: migrate FocusGuard to package layout"
```

**Готово, когда:**

- существующая история Git сохранена;
- корневые `main.py` и `requirements.txt` отсутствуют;
- зависимости определены только в `pyproject.toml`;
- существующий `README.md` дополнен, а не заменён;
- `python -m pip install -e '.[dev]'` завершается без ошибок;
- `pytest -q` проходит;
- `python -m focusguard` завершается без traceback с кодом `0`;
- в коммит вошли только явно проверенные файлы задачи.

---

### Задача 2: получить изображение со встроенной камеры

**Цель:** отделить источник кадров от цикла приложения, показать видеопоток через OpenCV и гарантированно освободить камеру при завершении.

**Файлы:**

- создать: `src/focusguard/camera.py`;
- создать: `tests/test_camera.py`;
- создать: `tests/test_app_camera.py`;
- изменить: `src/focusguard/app.py`;
- изменить: `README.md`;
- проверить: `src/focusguard/__main__.py`;
- проверить вручную: только реальный видеопоток и штатный выход по `q`.

**Границы компонентов:**

- `camera.py` отвечает только за создание OpenCV-источника, проверку его доступности, чтение одного кадра и освобождение ресурса;
- `camera.py` не содержит цикл приложения, `imshow`, обработку клавиатуры или тексты CLI;
- `app.py` создаёт источник, управляет покадровым циклом, временно вызывает `cv2.imshow` и `cv2.waitKey`, обрабатывает `q` и преобразует ошибки камеры в код завершения;
- позднее вывод окна и отрисовка будут перенесены из `app.py` в `renderer.py` без изменения интерфейса источника кадров.

**Интерфейсы:**

- `CameraUnavailableError(RuntimeError)` — камера не открылась при создании источника;
- `FrameReadError(RuntimeError)` — OpenCV не смог получить очередной кадр;
- `CameraSource(index: int = 0)` — открывает камеру с указанным индексом и выбрасывает `CameraUnavailableError`, если `VideoCapture.isOpened()` возвращает `False`;
- `CameraSource.read() -> numpy.ndarray` — возвращает ровно один BGR-кадр; если `VideoCapture.read()` возвращает неуспех или пустой кадр, выбрасывает `FrameReadError`;
- `CameraSource.release() -> None` — освобождает `VideoCapture`; повторный вызов допустим и не приводит к ошибке;
- `run(camera: CameraSource) -> None` в `app.py` — выполняет цикл показа до `q`; всегда вызывает `camera.release()` и закрывает окна в блоке очистки;
- `main() -> int` — создаёт `CameraSource(0)`, возвращает `0` после штатного выхода по `q`, а при `CameraUnavailableError` или `FrameReadError` печатает понятное сообщение в `stderr` и возвращает `1`.

**Шаги:**

- [ ] зафиксировать перечисленные интерфейсы в тестах до реализации;
- [ ] в `tests/test_camera.py` создать поддельный объект `VideoCapture` и через фикстуру `monkeypatch` заменить `focusguard.camera.cv2.VideoCapture` на фабрику, возвращающую эту подделку;
- [ ] проверить, что `CameraSource(0)` передаёт индекс `0` подменённой фабрике `VideoCapture`;
- [ ] там же проверить, что успешный `read()` возвращает переданный BGR-кадр без преобразования;
- [ ] проверить, что закрытая поддельная камера сначала получает вызов `release()`, а затем конструктор выбрасывает `CameraUnavailableError`;
- [ ] проверить, что неуспешное чтение приводит к `FrameReadError`;
- [ ] проверить, что `release()` вызывает освобождение поддельной камеры и допускает повторный вызов;
- [ ] в `tests/test_app_camera.py` передать в `run()` поддельный источник кадров и подменить функции окна и клавиатуры OpenCV;
- [ ] проверить, что `run()` показывает кадры, завершает цикл после `q`, освобождает источник и закрывает окна;
- [ ] отдельно проверить очистку, если `camera.read()` выбрасывает `FrameReadError`;
- [ ] проверить `main()`: штатный выход возвращает `0`, а поддельная недоступная камера — `1` и сообщение в `stderr`;
- [ ] запустить тесты до реализации:

```bash
pytest tests/test_camera.py tests/test_app_camera.py -v
```

- [ ] увидеть ожидаемую ошибку импорта `focusguard.camera` или отсутствие зафиксированных интерфейсов;
- [ ] реализовать в `camera.py` только `CameraSource`, `CameraUnavailableError`, `FrameReadError`, `read()` и `release()`;
- [ ] реализовать цикл окна и клавиатуры в `app.py`, не перенося их в `camera.py`;
- [ ] гарантировать очистку камеры и окон через блок `finally` в `run()`;
- [ ] преобразовать ошибки камеры в сообщение `stderr` и код `1` на уровне `main()`;
- [ ] повторно запустить автоматические проверки:

```bash
pytest tests/test_camera.py tests/test_app_camera.py -v
pytest -q
```

- [ ] убедиться, что тесты недоступной камеры и ошибок чтения проходят без настоящего устройства;
- [ ] запустить единственную ручную проверку:

```bash
python -m focusguard
```

- [ ] убедиться, что окно `FocusGuard` стабильно показывает актуальный видеопоток;
- [ ] нажать `q`, убедиться, что окно закрывается без зависания, затем повторно запустить приложение и подтвердить освобождение камеры;
- [ ] дополнить `README.md` командой запуска камеры, клавишей выхода и примечанием о разрешении ОС;
- [ ] перед коммитом проверить рабочее дерево и индекс:

```bash
git status --short
git diff --cached --name-status
git diff --name-status
git add README.md src/focusguard/app.py src/focusguard/camera.py tests/test_camera.py tests/test_app_camera.py
git diff --cached --name-status
git diff --cached --check
pytest -q
```

- [ ] убедиться, что в индексе находятся только файлы задачи 2;
- [ ] сделать отдельный коммит:

```bash
git commit -m "feat: display video from camera"
```

**Готово, когда:**

- `camera.py` не управляет окном, клавиатурой или циклом приложения;
- контракты источника кадров и исключений проверены тестами;
- недоступная камера и ошибка чтения воспроизводятся стабильными автоматическими тестами;
- `python -m focusguard` показывает реальный видеопоток;
- нажатие `q` завершает приложение с кодом `0`;
- ошибка камеры приводит к сообщению в `stderr` и коду `1`;
- камера освобождается, а окна OpenCV закрываются при штатном выходе и ошибке;
- `pytest -q` проходит без доступа к реальной камере;
- в коммит вошли только явно проверенные файлы задачи.
