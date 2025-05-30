# Opis przebiegu zadania 2: Budowanie i publikacja obrazu Dockera za pomocą GitHub Actions


W ramach tego zadania skonfigurowałam pełen proces CI/CD, który umożliwia automatyczne budowanie, skanowanie i publikowanie wieloarchitekturowego obrazu Dockera do GitHub Container Registry (GHCR). Cały proces uruchamiany jest automatycznie w momencie wypchnięcia tagu do repozytorium. Workflow został przygotowany w oparciu o GitHub Actions i wspiera zarówno architekturę amd64, jak i arm64.

# Przebieg zadania

## Inicjalizacja repozytorium

Na początku utworzyłam nowe repozytorium publiczne i połączyłam je z lokalnym repozytorium jako zdalny origin. Do nowo utworzonego repozytorium dodałam wszystkie pliki z poprzedniego zadania. Następnie zatwierdziłam je (git commit) i wypchnęłam (git push) do zdalnego repozytorium. Dzięki temu wszystkie potrzebne pliki – w tym Dockerfile i plik workflow – znalazły się w repozytorium zadanie2.

## Uwierzytelnianie zewnętrznych rejestrów (DockerHub i GHCR)

Aby workflow mógł logować się do DockerHub i GitHub Container Registry, wygenerowałam osobne tokeny dostępu: Token do DockerHub, który pozwala na logowanie i dostęp do mechanizmu cache. Personal Access Token (PAT) do GitHub, z uprawnieniami do publikowania obrazów do GHCR. Te dane zostały zapisane jako sekrety w GitHubie, co umożliwia bezpieczne ich wykorzystanie w ramach workflow bez umieszczania ich bezpośrednio w kodzie.

<img width="1192" alt="image" src="https://github.com/user-attachments/assets/db3414de-14b1-460f-9d1d-77da84a737ad" />

<img width="817" alt="image" src="https://github.com/user-attachments/assets/0cc2b547-bc70-409c-956a-ec27fc19a515" />


## Konfiguracja pliku workflow

W repozytorium utworzyłam plik docker-build.yml, który definiuje cały proces automatyzacji. Workflow ten uruchamiany jest w dwóch przypadkach: ręcznie (z poziomu interfejsu GitHub – workflow_dispatch), automatycznie, gdy do repozytorium zostaje wypchnięty nowy tag Git (np. v1.0.2).

Główne działania w workflow:

Checkout kodu źródłowego – pobiera aktualny kod repozytorium.
Konfiguracja QEMU – umożliwia emulację wielu architektur systemowych.
Konfiguracja Docker Buildx – umożliwia budowanie wieloarchitekturowych obrazów.
Logowanie do DockerHub i GHCR – przy użyciu wcześniej utworzonych sekretów.
Generowanie metadanych i tagów obrazu – na podstawie tagu Git i SHA commita.
Budowanie i publikowanie obrazu Dockera:
Obraz budowany jest dla amd64 i arm64,
Wykorzystany jest cache (ci-cache) z DockerHub, co znacznie przyspiesza kolejne budowania,
Obraz publikowany jest do ghcr.io.
Skanowanie bezpieczeństwa za pomocą Trivy:
Obraz jest analizowany pod kątem podatności typu HIGH i CRITICAL. W przypadku wykrycia takich podatności workflow kończy się błędem, aby nie wypuszczać niebezpiecznego obrazu.

## Wypchnięcie tagu i uruchomienie workflow
Aby uruchomić workflow, utworzyłam nowy tag Git (v1.0.2) i wypchnęłam go do repozytorium. GitHub Actions automatycznie wykrył nowy tag i rozpoczął wykonywanie zdefiniowanego workflow.


## Weryfikacja działania workflow

Korzystając z poleceń gh workflow list oraz gh run view, mogłam podejrzeć wszystkie uruchomienia workflow oraz szczegóły danego przebiegu. Wszystkie kroki zostały wykonane poprawnie, co potwierdziło, że konfiguracja została przeprowadzona prawidłowo.

```

agataogrodnik@MacBook-Pro-Agata zadanie2 % gh run view --job=43121351557

✓ v1.0.7 Build & Push Multi-Arch Docker Image · 15326236155
Triggered via push about 3 hours ago

✓ build-and-push in 45s (ID 43121351557)
  ✓ Set up job
  ✓ Checkout source code
  ✓ Set up QEMU
  ✓ Set up Docker Buildx
  ✓ DockerHub login
  ✓ GitHub Container Registry login
  ✓ Extract metadata (tags, labels)
  ✓ Debug metadata (opcjonalne)
  ✓ Build & push image (multi-arch)
  ✓ Scan image for CVEs using Trivy
  ✓ Post Scan image for CVEs using Trivy
  ✓ Post Build & push image (multi-arch)
  ✓ Post GitHub Container Registry login
  ✓ Post DockerHub login
  ✓ Post Set up Docker Buildx
  ✓ Post Set up QEMU
  ✓ Post Checkout source code
  ✓ Complete job

```

## Weryfikacja utworzenia obrazu Dockera

Na zakończenie potwierdziłam, że nowy obraz Dockera został z powodzeniem opublikowany do GitHub Container Registry, a cache został zaktualizowany na DockerHub.

<img width="1219" alt="image" src="https://github.com/user-attachments/assets/6f14bde0-8057-49f5-8403-7b9286cc8fad" />

### Link do Dockerhub

  https://hub.docker.com/repository/docker/agatog/ci-cache/general

