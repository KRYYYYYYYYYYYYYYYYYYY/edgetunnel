# EdgeTunnel (Cloudflare Workers) — гайд для 2026

Этот репозиторий запускает VLESS over WebSocket на Cloudflare Workers.
Ниже — практичный план, как привести проект к современному состоянию (2026) и стабильно деплоить через GitHub Actions.

## Что обновлено в этой версии

- Добавлен современный CI/CD workflow для Cloudflare Workers (`.github/workflows/cloudflare-workers.yml`).
- Обновлены npm-скрипты под актуальный `wrangler deploy`.
- Обновлён `compatibility_date` в `wrangler.toml` для предсказуемого поведения рантайма.

---

## 1) Что нужно перед деплоем

1. Аккаунт Cloudflare.
2. Созданный Worker (или первый деплой создаст его автоматически).
3. API Token с правами:
   - **Account → Cloudflare Workers Scripts:Edit**
   - **Zone → Zone:Read** (если используешь зону/маршруты)
4. `Account ID` из Cloudflare Dashboard.

---

## 2) Локальный запуск и проверка

```bash
npm install
npm run dev:vless
```

Локальный deploy (ручной):

```bash
npm run deploy
```

---

## 3) Настройка переменных Worker

В `wrangler.toml` уже подготовлен блок `[vars]`.

Минимум:
- `UUID` — твой UUID клиента.
- `PROXYIP` — опционально, можно оставить пустым.

Для production лучше хранить чувствительные данные в Secrets:

```bash
npx wrangler secret put UUID
npx wrangler secret put PROXYIP
```

> Если переменная задана и в `[vars]`, и в secrets — для чувствительных данных лучше использовать именно `secret`.

---

## 4) Какой GitHub Action использовать

### Рекомендуемый вариант для 2026

Используй официальный action:

- `cloudflare/wrangler-action@v3`

Почему он:
- официальный и поддерживаемый Cloudflare;
- проще, чем писать ручной `npm i -g wrangler` + `wrangler deploy`;
- хорошо работает с `CF_API_TOKEN`.

---

## 5) Настройка GitHub Secrets

В репозитории GitHub открой:
`Settings → Secrets and variables → Actions` и добавь:

- `CF_API_TOKEN`
- `CF_ACCOUNT_ID`
- `UUID` (если хочешь прокидывать как секрет на этапе deploy)
- `PROXYIP` (опционально)

---

## 6) Автодеплой через GitHub Actions

Workflow уже добавлен в репо и делает следующее:

- на `push` в `main` и в `pull_request` запускает валидацию (install + syntax check);
- деплой запускается только вне PR и только если заданы secrets `CF_API_TOKEN` и `CF_ACCOUNT_ID`;
- если секретов нет, workflow явно пишет, что деплой пропущен.

Также есть ручной запуск (`workflow_dispatch`) из вкладки **Actions**.

---

## 7) Рекомендуемый рабочий процесс (prod-ready)

1. Работаешь в feature-ветке.
2. PR в `main`.
3. После merge в `main` запускается автодеплой.
4. Для безопасных релизов добавь GitHub Environment `production` с required reviewers.

---

## 8) Быстрый чеклист, если деплой не проходит

1. Неверный `CF_API_TOKEN`/`CF_ACCOUNT_ID` или недостаточные права.
2. Не заполнены обязательные переменные (`UUID`).
3. В `wrangler.toml` указан не тот `name`.
4. Ограничения по политике аккаунта/зоны в Cloudflare.

---

## 9) Команды, которые нужны чаще всего

```bash
# локально
npm run dev:vless

# ручной деплой
npm run deploy

# dry-run для проверки сборки без публикации
npm run deploy:dry
```

---

Если хочешь, в следующем шаге могу сделать ещё более «боевую» схему:
- отдельные env `staging` / `production`;
- автокомментарий в PR с URL задеплоенного worker;
- безопасный progressive rollout через routes.
