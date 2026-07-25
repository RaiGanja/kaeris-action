# Как опубликовать (когда будет свет — 4 шага)

Эта папка = содержимое отдельного публичного репо `kaeris-action`. Файлы уже готовы:
`action.yml`, `summarize.py`, `README.md`, `LICENSE`.

## 1. Создай публичный репо
На GitHub: New repository → имя `kaeris-action` → Public → без README (он тут уже есть).

## 2. Залей эти файлы в КОРЕНЬ репо
```bash
cd ~/projects/i18n_tool/launch/kaeris-action
git init -b main
git add action.yml summarize.py README.md LICENSE
git commit -m "KAERIS i18n Translate action"
git remote add origin git@github.com:RaiGanja/kaeris-action.git
git push -u origin main
```

## 3. Сделай релиз-тег
```bash
git tag -a v1 -m "v1"
git push origin v1
```
(или через UI: Releases → Draft a new release → tag `v1` → Publish)

## 4. Опубликуй в Marketplace
На странице релиза поставь галочку **«Publish this Action to the GitHub Marketplace»**
→ прими соглашение → выбери категории: **Continuous Integration** + **Utilities**
→ Publish.

Готово. Люди смогут `uses: RaiGanja/kaeris-action@v1`.

## Проверка перед публикацией (уже сделана мной)
- `action.yml` — валидный YAML, зовёт `summarize.py` из корня (`github.action_path`).
- `summarize.py` — самодостаточный (только stdlib), компилируется.
- README точен: 46 языков, 11 форматов, честно про check (JSON+ARB).
- ⚠️ Action ставит `pip install kaeris` — значит пакет `kaeris` должен быть на PyPI (твой шаг из чеклиста запуска).
