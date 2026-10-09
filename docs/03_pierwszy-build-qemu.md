# Rozdział 3. Pierwszy build i uruchomienie systemu w QEMU (szkielet)

Status: pełny przebieg builda i uruchomienia obrazu w QEMU został
zweryfikowany dla Yocto Scarthgap i maszyny `qemux86-64`.

Ustalenia: Yocto **Scarthgap**, maszyna `qemux86-64` (emulowany komputer PC),
obraz `core-image-minimal`, narzędzia w kontenerze `crops/poky:ubuntu-22.04`.

## Co chcemy osiągnąć

Zbudować od zera minimalny system Linux dla emulowanego komputera PC i
uruchomić go w QEMU z dostępem do konsoli. Pozwala to poznać cały przebieg
pracy z Yocto bez fizycznej płytki: pobranie źródeł, przygotowanie środowiska,
wybór maszyny i obrazu, budowanie oraz start systemu.

Dlaczego od QEMU: build i uruchomienie nie wymagają sprzętu, błędy
konfiguracji widać od razu, a ten sam przebieg zastosujemy później dla
Raspberry Pi.

Rezultat: obraz w `build/tmp/deploy/images/qemux86-64/` i działająca konsola
w QEMU.

## Jak to osiągnąć

1. **Układ projektu.** Repozytorium kodu zawiera `docs/`, a Poky dołączamy
   jako submoduł git w `poky/`, przypięty do gałęzi `scarthgap`. Submoduł dodaje
   polecenie `git submodule add -b <gałąź> <adres> <katalog>`; zapisuje ono adres
   w `.gitmodules` i przypina konkretny commit. Własne warstwy będą osobnymi
   repozytoriami (od rozdziału 5). Katalog `build/` jest ignorowany.
2. **Kontener.** Narzędzia Yocto uruchamiamy w `crops/poky:ubuntu-22.04` poleceniem
   `docker run` (z `sudo`). Opcja `-v <katalog>:/workdir` montuje repozytorium
   w kontenerze, `--workdir=/workdir` ustawia katalog roboczy, a `-it` daje
   interaktywną powłokę.
3. **Środowisko builda.** `source poky/oe-init-build-env build` tworzy
   `build/conf` (`local.conf`, `bblayers.conf`) i przełącza do `build/`.
4. **Maszyna i obraz.** Maszynę ustawia `MACHINE` w `local.conf`
   (`qemux86-64`), a obraz wskazujemy jako argument `bitbake`.
5. **Budowanie.** `bitbake <obraz>` pobiera źródła, kompiluje pakiety i składa obraz.
   Wyniki trafiają do `build/tmp/deploy/images/<maszyna>/`.
6. **Uruchomienie.** `runqemu <maszyna> nographic slirp` uruchamia obraz
   zbudowany dla wskazanej maszyny; `runqemu` odnajduje go w aktywnym
   środowisku builda. Opcja `nographic` kieruje konsolę do terminala, a
   `slirp` zapewnia sieć bez uprawnień do interfejsów TAP. Emulator zamykamy
   `Ctrl+A`, potem `X`.

## Wykonana weryfikacja

1. Oficjalne Poky Scarthgap jest dołączone jako submoduł z przypiętym commitem.
2. Środowisko Yocto uruchomiono w kontenerze z repozytorium widocznym pod
   `/workdir`.
3. Zbudowano `core-image-minimal` dla `qemux86-64`; artefakty znajdują się w
   `build/tmp/deploy/images/qemux86-64/`.
4. Obraz uruchomiono w QEMU bez okna graficznego z konsolą tekstową.

## Pełne rozwiązanie

Na hoście, w repozytorium kodu:

```bash
cd ~/PWD_Yocto/PWD_Yocto_AGH_code
git submodule add -b scarthgap https://git.yoctoproject.org/poky poky
sudo docker run --rm -it -v "$PWD":/workdir --workdir=/workdir crops/poky:ubuntu-22.04
```

W kontenerze:

```bash
source poky/oe-init-build-env build
bitbake core-image-minimal
ls tmp/deploy/images/qemux86-64/
runqemu qemux86-64 nographic slirp
```

W QEMU: login `root` (bez hasła), `uname -r`, wyjście `Ctrl+A`, potem `X`.

## Do uzupełnienia

- Wyjaśnienie roli BitBake, receptur i konfiguracji (3.1).
- Objaśnienie zmiennych w `local.conf` oraz wyniku `bitbake`.
- Typowe problemy: zawieszone klonowanie przez `git://`, brak KVM.
