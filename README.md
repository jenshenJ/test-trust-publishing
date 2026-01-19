# trusted-publish-test

Тестовый монорепо из 3 пакетов для проверки **npm Trusted Publishing (OIDC)** без `NPM_TOKEN`.

## Пакеты

| Пакет | Путь |
|-------|------|
| `@jenshenj/trusted-publish-a` | `packages/one` |
| `@jenshenj/trusted-publish-b` | `packages/two` |
| `@jenshenj/trusted-publish-c` | `packages/three` |

## Как поднять с нуля

### 1. Первая публикация каждого пакета (с токеном)

Trusted Publishing работает только для пакетов, у которых **уже есть** хотя бы одна версия. Первый publish — вручную с `npm login`:

```bash
# из корня репо
for d in packages/one packages/two packages/three; do
  (cd $d && npm login && npm publish --access public)
done
```

Или по одному: `cd packages/one && npm login && npm publish --access public` и т.д.

### 2. Trusted Publisher на npm

Для **каждого** пакета на https://www.npmjs.com:

- Открыть пакет → **Settings** → **Publishing access** → **Trusted publishers**
- **Add** → **GitHub Actions**
- Указать:
  - **GitHub user or organization:** `jenshenJ`
  - **Repository name:** `trusted-publish-test`
  - **Workflow filename:** `release.yml`

Пакеты: `@jenshenj/trusted-publish-a`, `@jenshenj/trusted-publish-b`, `@jenshenj/trusted-publish-c`.

### 3. Репозиторий на GitHub

- Создать репо **jenshenJ/trusted-publish-test** (public).
- Локально:

```bash
git init
git add .
git commit -m "chore: init, 3 packages, release.yml"
git branch -M main
git remote add origin https://github.com/jenshenJ/trusted-publish-test.git
git push -u origin main
```

### 4. Проверка

После первого push workflow **Release** запустится. Во всех пакетах версия сменилась (файлов не было в `HEAD~1`) → все три должны уйти в **Publish to npm (OIDC)**. Убедись, что шаги зелёные и версии появились на npm.

Дальше: подними версию только в `packages/one/package.json` (например `1.0.1`), закоммить и запушить. В **Publish to npm (OIDC)** должен опубликоваться только `@jenshenj/trusted-publish-a`.

## Workflow

- **Триггер:** `push` в `main` с изменением `packages/**/package.json`
- **Check version changes:** для каждого пакета сравнивает `version` с предыдущим коммитом
- **Publish to npm (OIDC):** публикует только пакеты с изменённой версией через `npm publish --access public --provenance` **без** `NPM_TOKEN`

Требования: `permissions: id-token: write`, `npm@11.5.1` в job.

## Scope

Замени `@jenshenj` на свой npm- scope в `packages/*/package.json` и в настройках Trusted Publishers, если используешь другой аккаунт.
