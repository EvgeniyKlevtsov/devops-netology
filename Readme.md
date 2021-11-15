## 1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.

Для этого выполним команду **git show aefea**
результатом получим полный хеш и комментарий этого коммита.
  
commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>
Date:   Thu Jun 18 10:29:58 2020 -0400

    Update CHANGELOG.md

diff --git a/CHANGELOG.md b/CHANGELOG.md
index 86d70e3e0..588d807b1 100644
--- a/CHANGELOG.md
+++ b/CHANGELOG.md
@@ -27,6 +27,7 @@ BUG FIXES:
 * backend/s3: Prefer AWS shared configuration over EC2 metadata credentials by default ([#25134](https://github.com/hashicorp/terraform/issues/25134))
 * backend/s3: Prefer ECS credentials over EC2 metadata credentials by default ([#25134](https://github.com/hashicorp/terraform/issues/25134))
 * backend/s3: Remove hardcoded AWS Provider messaging ([#25134](https://github.com/hashicorp/terraform/issues/25134))
+* command: Fix bug with global `-v`/`-version`/`--version` flags introduced in 0.13.0beta2 [GH-25277]
 * command/0.13upgrade: Fix `0.13upgrade` usage help text to include options ([#25127](https://github.com/hashicorp/terraform/issues/25127))
 * command/0.13upgrade: Do not add source for builtin provider ([#25215](https://github.com/hashicorp/terraform/issues/25215))

## 2. Какому тегу соответствует коммит 85024d3?

Выполним команду **git show 85024d3**
Результатом получим 
commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)
соответственно коммиту **85024d3** соответствует тег **(tag: v0.12.23)**

## 3. Сколько родителей у коммита b8d720? Напишите их хеши.

Выполним команду **git rev-list --parents -n1 b8d720**  
Результатом получим текущий коммит и два родителя:
b8d720f8340221f2146e4e4870bf2ee0bc48f2d5 56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b

или так, выполнив команду **git show b8d720 --pretty=%P**  
56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b

## 4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

Выполним **git log v0.12.23...v0.12.24 --oneline**
Результатом получим хеши и комментарии всех коммитов между тегами
33ff1c03b (tag: v0.12.24) v0.12.24
b14b74c49 [Website] vmc provider links
3f235065b Update CHANGELOG.md
6ae64e247 registry: Fix panic when server is unreachable
5c619ca1b website: Remove links to the getting started guide's old location
06275647e Update CHANGELOG.md
d5f9411f5 command: Fix bug when using terraform login on Windows
4b6d06cc5 Update CHANGELOG.md
dd01a3507 Update CHANGELOG.md
225466bc3 Cleanup after v0.12.23 release

## 5. Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).

Выполнив **git log -S "func providerSource(" --oneline**
мы увидим самый первый коммит с этой функцией 
8c928e835 main: Consult local directories as potential mirrors of providers

## 6. Найдите все коммиты в которых была изменена функция globalPluginDirs.

Ищем совпадения **git grep -c globalPluginDirs**
commands.go:2
internal/command/cliconfig/config_unix.go:1
plugins.go:2

и находим все коммиты **git log -L :globalPluginDirs:plugins.go**
commit 8364383c359a6b738a436d1b7745ccdce178df47  
commit 66ebff90cdfaa6938f26f908c7ebad8d547fea17  
commit 41ab0aef7a0fe030e84018973a64135b11abcd70  
commit 52dbf94834cb970b510f2fba853a5b49ad9b1a46  
commit 78b12205587fe839f10d946ea3fdc06719decb05

## 7. Кто автор функции synchronizedWriters?

выполнив **git log --pretty=format:"%h : %ad : %an" -S synchronizedWriters**  
bdfea50cc : Mon Nov 30 18:02:04 2020 -0500 : James Bardin
fd4f7eb0b : Wed Oct 21 13:06:23 2020 -0400 : James Bardin
5ac311e2a : Wed May 3 16:25:41 2017 -0700 : Martin Atkins
мы видим что коммит **5ac311e2a** самый младший, значит автор **Martin Atkins**.