## Задание 0-1 ##
1. Во-первых, я молодец:
![avatar](./docs/0_1.png)
2. Wsl на винде тоже сойдёт:
![avatar](./docs/0_2.png)
## Задание 2 ##
_Описание задания_: Перенесите все коммиты, находящиеся в ветке ci, в ветку master с объединением всех коммитов в один и изменением сообщения таким образом, чтобы оно полностью описывало все вносимые изменения. Удалите ветку ci.
Для упрощения работы предлагаю следующее: визуализируем графом!
```bash
git log --graph --oneline --decorate --all
```
![avatar](./docs/2_1.png)
Можно заметить, что _только 2_ последних коммита из ветки ci нет в ветке master. 

```bash
git checkout ci

git reset --soft HEAD~2 && 
git commit --edit -m"$(git log --format=%B --reverse HEAD..HEAD@{1})"
```
После выполнения того, что сверху, на выходе получается один коммит, объединенный по названию в одно, при этом полностью содержит всё для понимания.
![avatar](./docs/2_2.png)
![avatar](./docs/2_3.png)

Переносим коммит в мастер и удаляем ветку:
```bash
git checkout master
git cherry-pick ci
git branch -D ci
```

## Задание 3 ##
_Описание задания_: В репозитории есть несколько альтернативных историй проекта, недоступных из текущей версии графа и не связанных с ней. Найдите последний коммит любой из версий и создайте на нём ветку old-master.

Для вывода этих версий необходимо ввести команду ```git reflog``` - рекорд всех коммитов, на которые ссылались в репозитории в любое время.

```bash
git log --graph --oneline --decorate --all
```
![avatar](./docs/3_1.png)

```aca3abb HEAD@{167}: commit: sfdgfs``` - подходит под описание. Работаем с этого коммита:

```bash
git checkout aca3abb

git branch old-master
```
## Задание 4 ##

_Описание задания_: Определите коммит, в котором строчка 32 файла prisma/seed.ts изменялась в последний раз, и его дату.

Команда ```git blame``` показывает, в каком коммите и какой автор последний раз изменял каждую строку файла

```bash
git blame -L 32,32 prisma/seed.ts
```

![avatar](./docs/4_1.png)

## Задание 5 ##
Спойлер: было очень больно. И бубунта, и винда вели себя забавно, поэтому целостность (о которой мы будем говорить дальше) будет нарушена - мне приходилось с болью переносить из wsl на windows, так как первое мне было нужно для 1 пункта работы.
_Описание задания_: В проекте существует регресс, на который имеется тест, запускающийся по команде npm run test. Найдите коммит, в котором проявился регресс.

```bash
npm install jest
git checkout master

git bisect start
git bisect bad
git bisect good 8673a612 
# Сверху указан коммит, на котором тест работал

npm run test
```

(Кстати, раз я заговорила о боли, то покажу её ещё и наглядно)
![avatar](./docs/5_1.png)

Далее возможно два исхода: тест прошел, и мы помечаем его ```git bisect good``` либо не прошел - ```git bisect bad```.

В итоге получаем: 
![avatar](./docs/5_2.png)


## Задание 6 ##
_Описание задания_: В репозитории существует файл .env, содержащий конфиденциальную информацию. Удалите его из всех коммитов, где он присутствует, и добавьте в .gitignore.

```bash
git filter-branch --tree-filter "rm -f .env" -- --all
echo .env >> .gitignore
```
Для этого задания используем ```git filter-branch``` которая позволяет изменять ветки   ```-tree-filter``` Это фильтр для переписывания дерева и его содержимого. Аргумент вычисляется в оболочке с рабочим каталогом, установленным в корневом дереве извлечения. Новое дерево затем используется как есть (новые файлы автоматически добавляются, исчезнувшие файлы автоматически удаляются - ни файлы .gitignore, ни любые другие правила игнорирования *НЕ ИМЕЮТ НИКАКОГО ЭФФЕКТА*!).

## Задание 7 ##
_Описание задания_: Сделайте так, чтобы автором коммитов в ветке feature были Вы. Для этого укажите в изменяемых коммитах почту, привязанную к GitHub, и своё ФИО.

1. Возвращаемся в ветку `feature`
2. Используем команду `rebase`, где напишем по сути изменение автора.

```bash
git rebase --onto 4cac550 --exec "git commit --amend --author=\"Ksenia Vasyutinskaya <pks2002@yandex.ru>\"" 4cac550
```
Результат: 

![avatar](./docs/7_1.png)

## Задание 8 ##
_Описание задания_: Включите запоминание разрешений конфликтов. Влейте ветку feature в master, разрешив конфликт при слиянии. Откатите слияние, внесите изменение в файл README.md и снова влейте ветку feature в master без ручного разрешения конфликта.

Включаем запоминание разрешение конфликтов:
```bash
git config rerere.enabled true
```
Вливаем ветку в мастер:
```bash
git checkout master
git merge feature
```
У нас будут конфликты в файле README.md. Необходимо решить этот конфликт. Например, nano и удалить текст из мастера.

Далее нам необходимо закоммитить этот файл и удалить предыдущий новому коммит:
```bash
git add README.md
git commit --no-edit
git reset --hard HEAD~1
git merge feature
```

Ура, запомнил наше разрешение конфликта! 
```bash
git add README.md
git commit --no-edit
```

## Задание 9 ##
В предыдущих пунктах говорила о проблемах, с которыми я столкнулась, делая лабу по гиту. Моя целостность репозитория нарушена.
 
```bash
git fsck
```

## Задание 10 ##
_описание задания_: Проверьте размер репозитория (папки .git) и добейтесь уменьшения его размера.

Почистить папку `.git` можно с помощью команды `git gc`

Мы удаляем файлы старше сегодняшнего дня и требуем большей оптимизации места.

```bash
git gc --prune=now --aggressive
```

