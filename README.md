# Autobuilds APT Repository

This repository hosts an automated CI/CD pipeline that compiles CLI tools and utilities from source, packages them as native Debian (`.deb`) packages, and serves them as a custom APT repository hosted entirely on GitHub Pages.

## 🎯 What is this?

Building tools from source directly on your host machine often pollutes your environment with heavy build toolchains (like Rust, C++ compilers, and dev headers). 

This project solves that by:
1. Compiling tools inside isolated Docker multi-stage pipelines.
2. Packaging the compiled binaries into `.deb` archives using CPack.
3. Automatically publishing those packages to a static APT repository on GitHub Pages.

This gives you the best of both worlds: you get custom, compiled-from-source binaries, but with the clean installation, dependency management, and easy uninstallation provided natively by `apt install` and `apt remove`.

## 🖥️ Target Systems

This repository specifically targets and builds for the following operating systems:
* **Ubuntu 22.04 LTS (Jammy Jellyfish)**
* **Debian 12 (Bookworm)**

Note: Binaries are built against the latest point releases (e.g., Ubuntu 22.04.5 or Debian 12.7) and are fully compatible with any older minor or patch version within that same major release.

## 🚀 How to Use (Client Configuration)

To install packages from this repository on your target server or WSL instance, you must add our GPG signing key and APT source.

**1. Import the GPG Key**
```bash
sudo mkdir -p /etc/apt/keyrings
curl -sSL https://eshteharipour.github.io/autobuilds/autobuilds.asc | sudo tee /etc/apt/keyrings/autobuilds.asc >/dev/null
```

**2. Add the APT Source**
This command automatically resolves your OS codename (e.g., `jammy` or `bookworm`) to fetch the correct binaries:
```bash
echo "deb [signed-by=/etc/apt/keyrings/autobuilds.asc] https://eshteharipour.github.io/autobuilds/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/autobuilds.list
```

**3. Install Tools**
```bash
sudo apt update
sudo apt install et  # Replace 'et' with the desired package name
```

## 🛠️ How it Works (Under the Hood)

* **Build Phase:** GitHub Actions triggers a Docker build parameterized via matrix strategy for both `debian:12-slim` and `ubuntu:22.04` base images.
* **Packaging:** CMake/CPack generates the `.deb` files directly within the Docker build environment.
* **Hosting:** An aggregation workflow downloads the `.deb` artifacts, generates APT index metadata (`Packages.gz`, `Release`), signs the repository using GPG, and deploys it to the `gh-pages` branch.
