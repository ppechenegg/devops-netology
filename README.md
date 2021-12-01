# Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.

git show aefea<br/>
commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545<br/>

# Какому тегу соответствует коммит 85024d3?

git show 85024d3<br/>
commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)<br/>
Author: tf-release-bot <terraform@hashicorp.com><br/>
Date:   Thu Mar 5 20:56:10 2020 +0000<br/>

    v0.12.23
Ответ: v0.12.23

# Сколько родителей у коммита b8d720? Напишите их хеши.

git show --pretty=format:' %P' b8d720<br/>
56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b

# Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

git log  v0.12.23..v0.12.24  --oneline

33ff1c03b (tag: v0.12.24) v0.12.24<br/>
b14b74c49 [Website] vmc provider links<br/>
3f235065b Update CHANGELOG.md<br/>
6ae64e247 registry: Fix panic when server is unreachable<br/>
5c619ca1b website: Remove links to the getting started guide's old location<br/>
06275647e Update CHANGELOG.md<br/>
d5f9411f5 command: Fix bug when using terraform login on Windows<br/>
4b6d06cc5 Update CHANGELOG.md<br/>
dd01a3507 Update CHANGELOG.md<br/>
225466bc3 Cleanup after v0.12.23 release<br/>

# Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).

git log -S'func providerSource' --oneline<br/>
5af1e6234 main: Honor explicit provider_installation CLI config when present<br/>
8c928e835 main: Consult local directories as potential mirrors of providers<br/>


git show 5af1e6234<br/>
git show 8c928e835<br/>
Ответ: Коммит в котором создана функция - 5af1e6234

# Найдите все коммиты в которых была изменена функция globalPluginDirs.

git log -S'func globalPluginDirs' --oneline<br/>
8364383c3 Push plugin discovery down into command package

# Кто автор функции synchronizedWriters?

git log -S'func synchronizedWriters' --pretty=format:'%h - %an %ae'<br/>
bdfea50cc - James Bardin j.bardin@gmail.com<br/>
5ac311e2a - Martin Atkins mart@degeneration.co.uk<br/>
