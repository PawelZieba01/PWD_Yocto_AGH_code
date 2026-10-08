# Pierwszy build i uruchomienie systemu w QEMU (propozycja treści)

Zakres: rozdział 3 z `Zawartość pracy.md` (3.2–3.4). Wersja Yocto: **Scarthgap**.
Maszyna: `qemux86-64` (emulowany komputer PC). Obraz: `core-image-minimal`.
Status: propozycja; komendy nie zostały jeszcze potwierdzone pełnym przebiegiem.

## Co chcemy osiągnąć

Zbudować minimalny obraz Linuksa dla emulowanego PC i uruchomić go w QEMU z
dostępem do konsoli. Uczestnik poznaje przebieg: pobranie Poky, inicjalizacja
środowiska builda, wybór maszyny i obrazu, `bitbake`, `runqemu`.

## Jak to osiągnąć

1. **Układ repozytoriów.** Pracujemy na trzech repozytoriach: dokumenty
   (`PWD_Yocto_AGH`), repozytorium kodu (`PWD_Yocto_AGH_code`) oraz fork Poky.
   Fork Poky jest klonowany do `poky/` wewnątrz repozytorium kodu i ma własny
   git: remote `upstream` wskazuje oficjalne Poky (źródło aktualizacji gałęzi
   `scarthgap`), a `origin` — nasz fork, do którego pushujemy. W forku trzymamy
   tylko kod. Repozytorium kodu ignoruje `poky/` i `build/` w `.gitignore`.
2. **Kontener.** Narzędzia Yocto działają w kontenerze `crops/poky:ubuntu-22.04`.
   Kontener uruchamiamy z `sudo`, a repozytorium montujemy jako `/workdir`.
3. **Poky.** Klonujemy gałąź `scarthgap` z oficjalnego
   `https://git.yoctoproject.org/poky` (lub z forka — adres do uzupełnienia).
   Protokół `git://` bywa blokowany, a klonowanie wtedy zawiesza się bez błędu.
4. **Środowisko builda.** `source poky/oe-init-build-env build` tworzy `build/conf`
   (`local.conf`, `bblayers.conf`) i przełącza powłokę do `build/`.
5. **Maszyna i obraz.** W `build/conf/local.conf` maszyną jest `qemux86-64`.
   Obraz `core-image-minimal` wskazujemy poleceniem `bitbake`.
6. **Uruchomienie.** `runqemu` startuje obraz; `nographic` kieruje konsolę do
   terminala, a `slirp` zapewnia sieć bez uprawnień do interfejsów TAP.

## Zadanie do wykonania

1. Przejdź do repozytorium kodu na gałąź zadania i sprawdź, że jesteś na niej.
2. Uruchom kontener `crops/poky:ubuntu-22.04` z repozytorium jako katalogiem
   roboczym (z `sudo`).
3. W kontenerze pobierz Poky w gałęzi `scarthgap`.
4. Zainicjuj katalog builda `build` i sprawdź w `local.conf`, że maszyną jest
   `qemux86-64`.
5. Zbuduj obraz `core-image-minimal` i sprawdź zawartość katalogu z artefaktami.
6. Uruchom obraz w QEMU bez okna graficznego, zaloguj się i sprawdź wersję
   jądra. Zamknij emulator.

Weryfikacja: build kończy się bez błędów, w `build/tmp/deploy/images/qemux86-64/`
są artefakty obrazu, a w QEMU dostępna jest konsola.

## Pełne rozwiązanie

```bash
cd ~/PWD_Yocto/PWD_Yocto_AGH_code
git branch --show-current          # 03_qemu-first-build

sudo docker run --rm -it -v "$PWD":/workdir crops/poky:ubuntu-22.04 --workdir=/workdir
```

W kontenerze:

```bash
git clone -b scarthgap https://git.yoctoproject.org/poky
source poky/oe-init-build-env build
grep -n '^MACHINE' conf/local.conf     # MACHINE ??= "qemux86-64"
bitbake core-image-minimal
ls tmp/deploy/images/qemux86-64/
runqemu qemux86-64 core-image-minimal nographic slirp
```

W QEMU: login `root` (bez hasła), `uname -r`, wyjście `Ctrl+A`, potem `X`.

## Uwagi

- Pierwszy build trwa długo i pobiera dużo źródeł.
- Opcjonalnie `--device /dev/kvm` w `docker run` przyspiesza QEMU, jeśli host
  ma KVM.
- `poky/` ma własne repozytorium git, więc repozytorium kodu go nie śledzi.
  Zmiany w Poky commitujemy i pushujemy w `poky/` do forka.

## Pytania otwarte

- Adres forka Poky i nazwa gałęzi roboczej (do uzupełnienia).
- Czy własne warstwy (`meta-course`) mają być w forku Poky (obok `meta/`),
  czy w repozytorium kodu obok `poky/`.
