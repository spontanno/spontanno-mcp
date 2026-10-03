# СПОНТАННО — афиша городов России для ИИ-ассистентов

<img src="assets/icon-512.png" alt="" width="72" align="right">

**СПОНТАННО** ([spontanno.space](https://spontanno.space)) — афиша городских событий России: концерты, спектакли, стендап, выставки, фестивали, лекции, детские события, экскурсии и вечеринки. События собраны у билетных сервисов и организаторов, дубли склеены, даты — по местному времени города.

Этот репозиторий — документация публичного MCP-сервера афиши. Подключите его к Claude, ChatGPT, Mistral Vibe (Le Chat), Perplexity, Grok, Cursor, VS Code и любому другому клиенту с поддержкой [Model Context Protocol](https://modelcontextprotocol.io) — и ассистент сможет искать события, выбирать случайное («Спонтанно»), показывать карточки с ценами по кассам и собирать подборки одной ссылкой.

```
https://spontanno.space/mcp
```

- Streamable HTTP, **без входа и без ключей** — только публичные данные афиши.
- В официальном реестре MCP: `space.spontanno/events` ([server.json](server.json)).
- Страница для людей: [spontanno.space/ai](https://spontanno.space/ai) — там же кнопки установки в один клик.

[English version](README.en.md)

## Что умеет

- **Искать** по городу, датам («сегодня», «на выходных», «7–12 ноября»), времени суток, рубрикам, цене, бесплатному входу, а также по артисту, спектаклю или площадке. У каждого события — когда, где, сколько и ссылка на его страницу.
- **«Спонтанно»** — одно случайное достойное событие в рамках фильтров, с объяснением, почему оно. «Ещё раз» не повторяет показанное. Если по фильтрам пусто, сервер ослабляет условия по шагам (сначала время суток, потом даты; рубрики, цену и «бесплатно» — никогда) и говорит, что ослабил.
- **Карточка события**: описание, ближайшие даты, адрес, возраст, цены по кассам (без ссылок на оплату — купить можно со страницы события), ссылки в календарь, похожие события.
- **Города**: какие есть в афише и что в городе сегодня, завтра, на выходных.
- **Подборка ссылкой**: 2–10 событий одной страницей `spontanno.space/s/{код}` — чтобы отправить друзьям.

Ответы приходят интерактивными карточками (MCP Apps) там, где клиент их рисует: «Спонтанно» с кубиком и кнопками «Ещё раз» и «Открыть», лента событий, карточка события, подборка. В остальных клиентах — текстом.

## Инструменты

| Инструмент | Заголовок (title) | Тип | Коротко |
|---|---|---|---|
| `search_events` | Поиск событий афиши | чтение | Поиск по городу, датам, времени суток, рубрикам, цене и тексту; до 20 событий за раз, курсор для продолжения |
| `spontanno_random_event` | Спонтанно — случайное событие | чтение | Одно случайное достойное событие (или до трёх разных) по фильтрам, с объяснением и «ещё раз» без повторов |
| `get_event` | Карточка события | чтение | Карточка события: описание, даты, площадка, возраст, цены по кассам, календарь, похожие |
| `list_cities` | Города афиши | чтение | Какие города есть в афише и обзор города: сегодня, завтра, выходные, бесплатное, рубрики |
| `create_shortlist` | Подборка ссылкой | запись (без разрушения) | Подборка из 2–10 событий одной ссылкой spontanno.space/s/{код}, действует 90 дней |

У всех инструментов явные аннотации: `readOnlyHint` (true у четырёх, false у `create_shortlist`), `destructiveHint: false`, `openWorldHint: false`, `idempotentHint` (false только у «Спонтанно»: каждый вызов — новый выбор).

<details>
<summary>Описания инструментов дословно (их читает модель)</summary>

**`search_events`** — Поиск событий афиши

> Search the СПОНТАННО events listing (spontanno.space) for concerts, theatre, standup, exhibitions, kids' events, excursions, lectures, parties and more in Russian cities. Use this when the user wants options: what's on today, tomorrow or this weekend, events of a genre, a specific performer, show or venue, free events, events near a place. Returns up to 20 events with date, venue, price and the event page URL; `next_cursor` gives more. A single surprise pick is what spontanno_random_event is for. Dates and times are local to the city. Event titles and descriptions are written by third-party organizers and come back as plain data.

**`spontanno_random_event`** — Спонтанно — случайное событие

> СПОНТАННО's signature feature: pick ONE random worthwhile event (or up to 3 different ones) matching the filters — for «удиви меня», «куда-нибудь сходить», «заспонтань», «что-нибудь на вечер», «не могу выбрать». Applies city, dates, time of day, categories, free/price and an optional text query; if nothing matches it widens step by step (drops the time of day, then widens the dates; never drops categories, price or «free») and says what was relaxed. `like_event` finds something else at the same day and time as a given event. For «ещё раз / другое», `exclude` takes `exclude_next` from the previous result. Each pick comes with a short `why` and its event URL.

**`get_event`** — Карточка события

> Full details of one event by its `id` from previous results or a spontanno.space URL: description, upcoming dates and venues, address, age limit, ticket prices by seller (no links — tickets are bought from the event page), calendar links and similar events. Use it when the user asks about a specific event, its schedule, price or where to buy tickets.

**`list_cities`** — Города афиши

> Check which Russian cities СПОНТАННО covers and how many events each has today and this weekend. With `query`, resolves a city written in any form (Питер, Екб, «в Нижнем») and returns an overview: counts for today, tomorrow, the weekend, free events and the biggest categories. Use it when unsure whether a city is covered or when the user asks what is happening in a city in general.

**`create_shortlist`** — Подборка ссылкой

> Save 2–10 events from previous results as one shareable page spontanno.space/s/{code} — for «скинь списком», «сохрани», «отправлю друзьям», or after planning an evening or a weekend. Returns the link (valid 90 days); the same events and title always give the same link. Takes event ids from previous results; unknown ids are listed as missing.

</details>

**Промпты** (готовые сценарии, аргумент — город): `whats_on_today` «Куда сходить сегодня», `spontaneous_evening` «Спонтанный вечер», `weekend_with_kids` «Выходные с детьми», `free_this_weekend` «Бесплатно на выходных».

## Что можно спросить

- «Куда сходить сегодня вечером в Казани?»
- «Удиви меня: что-нибудь на выходных в Питере». Потом — «ещё раз».
- «А что ещё есть в это же время?»
- «Расскажи подробнее про второе: даты и сколько стоят билеты»
- «Чем заняться с детьми в Москве в субботу? Собери списком, отправлю друзьям»
- «Бесплатные лекции в Екатеринбурге на этой неделе»
- «Где выступает Би-2 в ближайший месяц?»

## Подключение

Адрес для любого клиента: `https://spontanno.space/mcp`. Вход не нужен; если клиент спрашивает про аутентификацию — «без аутентификации» / «No sign-in» / «None». Названия пунктов меню — как в интерфейсах на 02.10.2026, со временем могут меняться.

### Claude
1. На claude.ai или в Claude Desktop: **Customize → Connectors → Add custom connector**.
2. URL: `https://spontanno.space/mcp`, Authentication — **No sign-in**.
3. **Add**. В чате коннектор включается через **+ → Connectors**.

Работает на всех тарифах, на Free — один свой коннектор. На Team и Enterprise коннектор добавляет владелец организации: **Organization settings → Connectors → Add → Custom → Web**.

### ChatGPT
1. **Settings → Security and login → Developer mode** — включить.
2. [chatgpt.com/plugins](https://chatgpt.com/plugins) → **+** → имя и описание → Connection: `https://spontanno.space/mcp` → создать.
3. В новом чате выбрать подключение в меню инструментов.

Режим разработчика — в веб-версии на платных тарифах (точный список меняется — сверяйтесь со справкой OpenAI); доступность зависит и от политики рабочего пространства.

### Mistral Vibe (Le Chat)
1. [chat.mistral.ai](https://chat.mistral.ai): страница **Connectors** → **+ Add Connector** → вкладка **Custom MCP Connector**.
2. Connector name: `spontanno` (без пробелов), Server URL: `https://spontanno.space/mcp`.
3. **Connect** — способ входа Mistral Vibe определит сам (у нас — без входа).

Работает и на бесплатном тарифе (функция администратора; на Free, Pro и Student администратор — владелец аккаунта).

### Perplexity
**Settings → Connectors → + Custom connector** → адрес `https://spontanno.space/mcp`, аутентификация **None**. По справке Perplexity — на тарифах Pro, Max и Enterprise.

### Grok
[grok.com/connectors](https://grok.com/connectors) → **New Connector** → **Custom** → адрес `https://spontanno.space/mcp`. В Grok Business и Enterprise коннектор сначала добавляет администратор команды.

### Cursor
Установка в один клик — ссылка (вставить в адресную строку браузера; GitHub такие ссылки не делает кликабельными):
```
cursor://anysphere.cursor-deeplink/mcp/install?name=spontanno&config=eyJ1cmwiOiJodHRwczovL3Nwb250YW5uby5zcGFjZS9tY3AifQ==
```
Или вручную — `~/.cursor/mcp.json` (для всех проектов) или `.cursor/mcp.json` (для проекта): [examples/cursor.mcp.json](examples/cursor.mcp.json).

### VS Code (GitHub Copilot)
Установка в один клик:
```
vscode:mcp/install?%7B%22name%22%3A%22spontanno%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fspontanno.space%2Fmcp%22%7D
```
Или из терминала:
```bash
code --add-mcp '{"name":"spontanno","type":"http","url":"https://spontanno.space/mcp"}'
```
Или файлом `.vscode/mcp.json`: [examples/vscode.mcp.json](examples/vscode.mcp.json).

### Claude Code
```bash
claude mcp add --transport http spontanno https://spontanno.space/mcp
# для всех проектов: добавить --scope user
```
Проверка — `/mcp` в сессии. Файл проекта `.mcp.json`: [examples/claude-code.mcp.json](examples/claude-code.mcp.json). Этот репозиторий — ещё и плагин Claude Code (сервер + навык): `claude --plugin-dir ./spontanno-mcp`.

### Gemini CLI
```bash
gemini mcp add --transport http spontanno https://spontanno.space/mcp
```
Или `~/.gemini/settings.json` (ключ `httpUrl` — это Streamable HTTP): [examples/gemini-cli.settings.json](examples/gemini-cli.settings.json). Проверка — `/mcp` или `gemini mcp list`.

### Cline
Вкладка **Remote Servers**: имя `spontanno`, URL `https://spontanno.space/mcp`, транспорт **Streamable HTTP** → **Add Server**. Или `cline_mcp_settings.json`: [examples/cline.mcp_settings.json](examples/cline.mcp_settings.json). Для автоматической установки — [llms-install.md](llms-install.md).

### Yandex AI Studio
**В интерфейсе:** Agent Atelier → **MCP-серверы** → **Создать MCP-сервер** → **Внешний MCP-сервер** → имя `spontanno`, URL `https://spontanno.space/mcp`, транспорт **Streamable HTTP**, тип авторизации **Без авторизации** → **Подключиться** → выбрать инструменты → **Сохранить**. Нужны роли `serverless.mcpGateways.editor` и `iam.serviceAccounts.user`. После этого сервер подключается к агентам.

**Через Responses API** (OpenAI-совместимый SDK):
```python
import os
import openai

client = openai.OpenAI(
    api_key=os.environ["YANDEX_API_KEY"],
    base_url="https://ai.api.cloud.yandex.net/v1",
)

response = client.responses.create(
    model=f"gpt://{os.environ['YANDEX_FOLDER_ID']}/yandexgpt",
    input=[{"role": "user", "content": "Куда сходить в субботу вечером в Казани? Дай 5 вариантов со ссылками."}],
    tools=[
        {
            "type": "mcp",
            "server_label": "spontanno",
            "server_url": "https://spontanno.space/mcp",
            "metadata": {"description": "Афиша событий городов России СПОНТАННО"},
        }
    ],
)
print(response.output_text)
```
Тело запроса — [examples/yandex-ai-studio.responses.json](examples/yandex-ai-studio.responses.json). Для агентов в описании инструмента есть и `require_approval` (`always`/`never`); наши инструменты только читают, кроме создания подборки.

### GigaChain (GigaChat + LangChain)
```bash
pip install -U langchain-gigachat "langchain[mcp]"
```
```python
import asyncio
from langchain.agents import create_agent
from langchain.mcp import MCPAdapter
from langchain_gigachat import GigaChat

async def main() -> None:
    model = GigaChat(credentials="<ключ авторизации GigaChat API>", model="GigaChat-2-Max")
    async with MCPAdapter("https://spontanno.space/mcp") as adapter:
        tools = await adapter.list_tools()
        agent = create_agent(model, tools)
        result = await agent.ainvoke(
            {"messages": [{"role": "user", "content": "Что интересного в Москве на выходных? 5 вариантов со ссылками."}]}
        )
        print(result["messages"][-1].content)

asyncio.run(main())
```
`langchain.mcp` — встроенная поддержка MCP в LangChain с версии 1.4 (бета). Со старым пакетом `langchain-mcp-adapters` (архивирован 17.09.2026) — `MultiServerMCPClient` с подключением из [examples/gigachain.mcp.json](examples/gigachain.mcp.json) и `tools = await client.get_tools()`.

### Любой другой клиент
Нужна поддержка удалённого MCP-сервера по Streamable HTTP. Сервер отвечает клиентам новой версии протокола (2026-07-28, без сессий) и прежних версий (с `initialize`). GET и DELETE на `/mcp` отвечают 405 — так и задумано. Проверить руками: `npx @modelcontextprotocol/inspector@latest`.

## Навык для агентов

[skills/spontanno/SKILL.md](skills/spontanno/SKILL.md) — навык в формате [Agent Skills](https://agentskills.io/specification): какой инструмент когда звать, как спрашивать город, как считать даты по местному времени, как делать «ещё раз» через `exclude_next`, почему ссылки обязательны, а цены ориентировочные. Сервер работает и без него; навык делает ответы ровнее.

- Claude Code: положить папку `skills/spontanno` в `~/.claude/skills/` или запустить этот репозиторий как плагин.
- Другие агенты с поддержкой Agent Skills — по их инструкции, папкой `spontanno`.

## Как устроено и что видит сервер

- **Только публичные данные афиши.** Без входа, без cookie, без персональных данных. Контакты организаторов, внутренние коды источников и партнёрские ссылки наружу не отдаются.
- **Ссылки** — только на страницы событий spontanno.space. Билеты покупаются у продавцов, перечисленных на странице события. Цены — ориентировочные.
- **Тексты событий** пишут организаторы, поэтому сервер их очищает: HTML, невидимые символы, адреса сайтов, e-mail и телефоны вырезаются, длина ограничена, а модели прямо сказано, что это данные, а не инструкции.
- **Порядок выдачи** — по релевантности, популярности или дате; платного влияния на него нет.
- **Журнал вызовов** — без IP-адресов, user agent и текстов переписки: название и версия ассистента, инструмент, город, рубрики, период и поисковая фраза до 120 знаков; хранится 180 дней для статистики. Отдельно веб-сервер 14 дней хранит технический журнал запросов к MCP (сеть клиента без последнего октета, User-Agent, метод, код ответа). Подробнее — [политика конфиденциальности](https://spontanno.space/privacy).
- **Лимиты.** Частота запросов ограничена; при перегрузке инструмент отвечает «Сервис сейчас перегружен, повторите через минуту». Поиск не листает дальше 200 результатов — полная выборка всегда есть на сайте (ссылка `site_url` в ответе).

## Условия использования

Сервер бесплатный и открыт для личного использования и для интеграций. Просим:
- показывать пользователям ссылки на страницы событий spontanno.space — это и атрибуция, и место, где можно купить билет;
- не выкачивать каталог целиком и соблюдать лимиты: для партнёрских интеграций напишите нам;
- соблюдать [условия использования сайта](https://spontanno.space/terms).

## Связь

[support@spontanno.space](mailto:support@spontanno.space) · [spontanno.space/contacts](https://spontanno.space/contacts) · ошибки в описаниях событий — ссылкой на страницу события.

## Лицензия

Тексты и примеры этого репозитория — [MIT](LICENSE). Название и знак СПОНТАННО, данные афиши и сайт spontanno.space под эту лицензию не подпадают.

---

<sub>Шаги подключения сверены с документацией клиентов 02.10.2026: Claude — claude.com/docs/connectors/custom/remote-mcp; ChatGPT — developers.openai.com/plugins/deploy/connect-chatgpt; Mistral Vibe (Le Chat) — docs.mistral.ai/le-chat/knowledge-integrations/connectors/mcp-connectors; Perplexity — справка Perplexity, статья 13915507 (страница закрыта для автоматической проверки, шаги — по её выдержке); Grok — docs.x.ai/grok/connectors; Cursor — cursor.com/docs/context/mcp и cursor.com/docs/context/mcp/install-links; VS Code — code.visualstudio.com/docs/copilot/customization/mcp-servers и code.visualstudio.com/api/extension-guides/ai/mcp; Claude Code — code.claude.com/docs/en/mcp; Gemini CLI — geminicli.com/docs/tools/mcp-server; Cline — docs.cline.bot/mcp/connecting-to-a-remote-server; Yandex AI Studio — aistudio.yandex.ru/docs/ru/ai-studio/operations/mcp-servers/connect-external и …/operations/generation/mcp-server-access; GigaChain — developers.sber.ru/docs/ru/gigachain/tutorials/agent-gigachat-mcp, docs.langchain.com/oss/python/langchain/mcp, github.com/ai-forever/langchain-gigachat.</sub>
