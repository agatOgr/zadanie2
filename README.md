# Zadanie 2


W ramach zadania skonfigurowano workflow GitHub Actions, który automatycznie buduje i publikuje obraz Dockera do GHCR po wypchnięciu tagu do repozytorium. Proces obejmuje checkout kodu, konfigurację Buildx i QEMU dla wsparcia wielu architektur (amd64 i arm64), logowanie do rejestrów, budowanie obrazu na podstawie dwustopniowego Dockerfile oraz jego wysyłkę. Obraz jest dodatkowo skanowany pod kątem krytycznych i wysokich podatności za pomocą Trivy. Zastosowano również mechanizm cache, który publikuje dane do repozytorium na DockerHub, co przyspiesza kolejne buildy.

## Przebieg zadania

#### Logowanie do GitHub CLI, w celu uzyskania dostępu do operacji na GitHubie z terminala.

```
gh auth login
```


#### Zainicjowanie nowego lokalnego repozytorium Git z domyślną gałęzią main.
```
git init -b main
```


#### Utworzenie nowego repozytorium na GitHubie o nazwie zadanie2, oraz ustawienie je jako publiczne i dodanie zdalnego repozytorium origin.
```
agataogrodnik@MacBook-Pro-Agata ~ % gh repo create
? What would you like to do? Push an existing local repository to GitHub
? Path to local repository .
? Repository name zadanie2
? Description zadanie2
? Visibility Public
✓ Created repository agatOgr/zadanie2 on GitHub
  https://github.com/agatOgr/zadanie2
? Add a remote? Yes
? What should the new remote be called? origin
✓ Added remote https://github.com/agatOgr/zadanie2.git
```


```
agataogrodnik@MacBook-Pro-Agata ~ % gh repo list | grep zadanie2
agatOgr/zadanie2	zadanie2	public	2025-05-29T10:55:55Z
```


#### Dodanie wszystkich plików oraz katalogów projektu z zadania 1
```
git add .
```
```
git commit -m "Inicjalizacja repo zadanie2"
```



#### Wypchnięcie lokalną gałąź main do zdalnego repozytorium i ustawienie origin/main jako domyślny upstream.
```
 git push -u origin main
```


#### Wygenerowanie tokena dostępu w Dockerhub
<img width="1192" alt="image" src="https://github.com/user-attachments/assets/db3414de-14b1-460f-9d1d-77da84a737ad" />



<img width="817" alt="image" src="https://github.com/user-attachments/assets/0cc2b547-bc70-409c-956a-ec27fc19a515" />




```
git tag v1.0.2  
git push origin v1.0.2
```

```
agataogrodnik@MacBook-Pro-Agata zadanie2 % gh workflow list             
NAME                                  STATE   ID       
Build & Push Multi-Arch Docker Image  active  165023368
```

```
agataogrodnik@MacBook-Pro-Agata zadanie2 % gh run view                            
? Select a workflow run ✓ Build & Push Multi-Arch Docker Image, Build & Push Multi-Arch Docker Image [v1.0.7] 3h42m40s ago

✓ v1.0.7 Build & Push Multi-Arch Docker Image · 15326236155
Triggered via push about 3 hours ago

JOBS
✓ build-and-push in 45s (ID 43121351557)

For more information about the job, try: gh run view --job=43121351557
View this run on GitHub: https://github.com/agatOgr/zadanie2/actions/runs/15326236155
agataogrodnik@MacBook-Pro-Agata zadanie2 % 
```

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

<img width="1219" alt="image" src="https://github.com/user-attachments/assets/6f14bde0-8057-49f5-8403-7b9286cc8fad" />




```
name: Build & Push Multi-Arch Docker Image
# Nazwa workflow, widoczna w GitHub Actions

on:
  workflow_dispatch:
  # Pozwala na ręczne uruchomienie workflow z poziomu UI GitHub
  push:
    tags:
      - '*'  
      # Uruchamia workflow na push do dowolnego taga, np. v1.0.5, v2.0.0 itp.

permissions:
  contents: read
  # Uprawnienie do odczytu zawartości repozytorium
  packages: write
  # Uprawnienie do zapisu pakietów (np. obrazów w GitHub Container Registry)

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    # Workflow uruchamia się na maszynie z systemem Ubuntu w najnowszej wersji

    steps:
    - name: Checkout source code
      uses: actions/checkout@v4
      # Pobiera kod źródłowy repozytorium na maszynę roboczą

    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
      # Konfiguruje QEMU, potrzebne do emulacji różnych architektur (np. arm64) podczas budowania obrazu

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      # Konfiguruje Docker Buildx, narzędzie pozwalające budować obrazy multi-architekturowe

    - name: DockerHub login
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
      # Loguje się do DockerHub przy pomocy podanych w secrets danych (do cache i/lub wypychania)

    - name: GitHub Container Registry login
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GHCR_PAT }}
      # Loguje się do GitHub Container Registry (ghcr.io) z tokenem dostępu (PAT)

    - name: Extract metadata (tags, labels)
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: |
          ghcr.io/agatogr/zadanie2
        tags: |
          type=ref,event=tag
          type=sha
      # Generuje metadane (tagi i etykiety) na podstawie referencji git (np. tagu)
      # type=ref,event=tag - tag z git jako tag obrazu
      # type=sha - dodatkowy tag sha commita

    - name: Debug metadata (opcjonalne)
      run: |
        echo "Tags: ${{ steps.meta.outputs.tags }}"
        echo "Version: ${{ steps.meta.outputs.version }}"
      # Wyświetla na konsoli metadane wygenerowane przez poprzedni krok (do debugowania)

    - name: Build & push image (multi-arch)
      uses: docker/build-push-action@v5
      with:
        context: .
        # Kontekst builda - aktualny katalog repozytorium
        platforms: linux/amd64,linux/arm64
        # Buduje obrazy dla architektur amd64 i arm64
        push: true
        # Po zbudowaniu obraz zostanie wypchnięty do rejestru
        tags: ghcr.io/agatogr/zadanie2:${{ github.ref_name }}
        # Tag obrazu - nazwa repozytorium + tag git (np. v1.0.5)
        labels: ${{ steps.meta.outputs.labels }}
        # Dodaje etykiety (labels) do obrazu na podstawie metadanych
        cache-from: type=registry,ref=docker.io/agatog/ci-cache:latest
        cache-to: type=registry,ref=docker.io/agatog/ci-cache:latest,mode=max
        # Używa cache builda z repozytorium docker.io dla przyspieszenia kolejnych buildów

    - name: Scan image for CVEs using Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ghcr.io/agatogr/zadanie2:${{ github.ref_name }}
        format: table
        severity: CRITICAL,HIGH
        exit-code: 1
      # Skanuje zbudowany obraz pod kątem podatności (CVE) za pomocą Trivy
      # Wypisuje wynik w formacie tabeli
      # Ustawia poziom poważnych podatności do CRITICAL i HIGH
      # Zwraca błąd (exit code 1), jeśli wykryje podatności o tym poziomie lub wyższym

```
=======
# dodaj pustą linię
# dodaj pustą linię
# dodaj pustą linię
# dodaj pustą linię
>>>>>>> f7b93b9 (Build & Push Multi-Arch Docker Image)
# dodaj pustą linię
