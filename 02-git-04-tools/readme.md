1. Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.

git log | grep "aefea"
commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545

2. Какому тегу соответствует коммит 85024d3?

git log --all | grep "85024d3"
commit 85024d3100126de36331c6982bfaac02cdab9e76

git show 85024d3100126de36331c6982bfaac02cdab9e76
commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)
Author: tf-release-bot <terraform@hashicorp.com>
Date:   Thu Mar 5 20:56:10 2020 +0000

    v0.12.23

3. Сколько родителей у коммита b8d720? Напишите их хеши.

git log | grep "b8d720"
commit b8d720f8340221f2146e4e4870bf2ee0bc48f2d5

git log --pretty=raw --parents -1 b8d720f8340221f2146e4e4870bf2ee0bc48f2d5
commit b8d720f8340221f2146e4e4870bf2ee0bc48f2d5 56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
tree cec002aab630c8bc701cb85bc94e55e751cd2d8f
parent 56cd7859e05c36c06b56d013b55a252d0bb7e158
parent 9ea88f22fc6269854151c571162c5bcf958bee2b
author Chris Griggs <cgriggs@hashicorp.com> 1579657548 -0800
committer GitHub <noreply@github.com> 1579657548 -0800
gpgsig -----BEGIN PGP SIGNATURE-----

 wsBcBAABCAAQBQJeJ6lMCRBK7hj4Ov3rIwAAdHIIAAWogFD+6nxgm7suNlJVqGv3
 iczwk1OySRvZiDhgJuaIEZMduudoIBnv7XStZsg5wQ7a0byh4iU6z7w6vVKPyj1e
 2KyTqwriHOYqDJ/pljIX92btLU+rXDP0DpnTg8WuMthqUJGh6q8OGorvlIHDwcdN
 DKZxKaLPaBYCD5nuYMyhuh9T6+4ayEN4zUbl5vLPN7XmvhSf4yDQ0H4UaR596qOo
 phaqODHglNLegdGwi+JaTtDSB/JO5zJNfn8OnuRgyhoplmKKlhBAEwS4muxjjkzD
 cUtFA+jkXaVpbfEh/dBRb3yB4W17jrxFDuDESjXKiU61bc8Fwa1JrfQAFlXczmY=
 =s9Hk
 -----END PGP SIGNATURE-----


    Merge pull request #23916 from hashicorp/cgriggs01-stable

    [Cherrypick] community links

4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

git log --oneline v0.12.23..v0.12.24
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

5. Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).

git grep -p "func providerSource(.*)"
provider_source.go=import (
provider_source.go:func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {


git log -L :providerSource:provider_source.go
commit 8c928e83589d90a031f811fae52a81be7153e82f
Author: Martin Atkins <mart@degeneration.co.uk>
Date:   Thu Apr 2 18:04:39 2020 -0700

    main: Consult local directories as potential mirrors of providers

    This restores some of the local search directories we used to include when
    searching for provider plugins in Terraform 0.12 and earlier. The
    directory structures we are expecting in these are different than before,
    so existing directory contents will not be compatible without
    restructuring, but we need to retain support for these local directories
    so that users can continue to sideload third-party provider plugins until
    the explicit, first-class provider mirrors configuration (in CLI config)
    is implemented, at which point users will be able to override these to
    whatever directories they want.

    This also includes some new search directories that are specific to the
    operating system where Terraform is running, following the documented
    layout conventions of that platform. In particular, this follows the
    XDG Base Directory specification on Unix systems, which has been a
    somewhat-common request to better support "sideloading" of packages via
    standard Linux distribution package managers and other similar mechanisms.
    While it isn't strictly necessary to add that now, it seems ideal to do
    all of the changes to our search directory layout at once so that our
    documentation about this can cleanly distinguish "0.12 and earlier" vs.
    "0.13 and later", rather than having to document a complex sequence of
    smaller changes.

    Because this behavior is a result of the integration of package main with
    package command, this behavior is verified using an e2etest rather than
    a unit test. That test, TestInitProvidersVendored, is also fixed here to
    create a suitable directory structure for the platform where the test is
    being run. This fixes TestInitProvidersVendored.

diff --git a/provider_source.go b/provider_source.go
--- /dev/null
+++ b/provider_source.go
@@ -0,0 +19,5 @@
+func providerSource(services *disco.Disco) getproviders.Source {
+       // We're not yet using the CLI config here because we've not implemented
+       // yet the new configuration constructs to customize provider search
+       // locations. That'll come later.
+       // For now, we have a fixed set of search directories:


6. Найдите все коммиты в которых была изменена функция globalPluginDirs.

git log --oneline -G"globalPluginDirs(.*)"
22a2580e9 main: Use the new cliconfig package credentials source
35a058fb3 main: configure credentials from the CLI config file
c0b176109 prevent log output during init
8364383c3 Push plugin discovery down into command package

7. Кто автор функции synchronizedWriters?

Указанной функции не вижу:
git grep -p -n "synchronized.*"
internal/instances/expander.go=3=import (
internal/instances/expander.go:23:// Expander is a synchronized object whose methods can be safely called
website/upgrade-guides/0-12.html.markdown=58=Then, perform the following steps:
website/upgrade-guides/0-12.html.markdown:65:  synchronized.
website/upgrade-guides/0-12.html.markdown=131=once more before upgrading in order to ensure that the latest state snapshot is
website/upgrade-guides/0-12.html.markdown:132:synchronized with the latest configuration.
website/upgrade-guides/0-12.html.markdown=137=any later v0.11 release) to perform one last `terraform init` and
website/upgrade-guides/0-12.html.markdown:138:`terraform apply` to ensure that everything is initialized and synchronized.

Точки разделения для отдельного репозитория истории не вижу.
Данныя функция сущетсвует ?
