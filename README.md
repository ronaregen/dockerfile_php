# 🐘 PHP Base Image

Ini adalah repository berisi Dockerfile custom untuk PHP base image yang biasa saya pakai di berbagai project larave saya. 
saya akan menjaga base php version nya selalu versi latest stable official

Berikut adalah contoh file GitHub Actions workflow (`deploy.yml`) untuk Laravel yang:

1. Build Docker image dari kode terbaru
2. Push ke Docker Hub
3. SSH ke server production
4. Pull image terbaru
5. Jalankan `docker-compose up -d`

---

### 📁 **Struktur Proyek yang Diharapkan**

Sebelum itu, pastikan:

* Kamu punya `Dockerfile` dan `docker-compose.yml`
* Akun Docker Hub & repo sudah dibuat
* Sudah upload SSH Private Key ke GitHub Secrets

---

### 🔑 **GitHub Secrets yang Dibutuhkan**

Simpan ini di GitHub repo kamu → `Settings > Secrets and variables > Actions`:

| Secret Name          | Keterangan                               |
| -------------------- | ---------------------------------------- |
| `DOCKERHUB_USERNAME` | Username Docker Hub kamu                 |
| `DOCKERHUB_TOKEN`    | Token/Password Docker Hub                |
| `SSH_PRIVATE_KEY`    | Private Key untuk akses ke server        |
| `SSH_HOST`           | IP/domain server production kamu         |
| `SSH_USER`           | Username SSH server (misalnya: `ubuntu`) |

---

### ⚙️ `.github/workflows/deploy.yml`

```yaml
name: Deploy Laravel App via Docker

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v3

    - name: Log in to Docker Hub
      run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

    - name: Build Docker Image
      run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/laravel-app:latest .

    - name: Push Docker Image to Docker Hub
      run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/laravel-app:latest

    - name: Setup SSH Key
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

    - name: Deploy to Production via SSH
      run: |
        ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/laravel-app:latest
          docker-compose -f /path/to/your/docker-compose.yml up -d
        EOF
```

---

### 📝 Catatan Penting:

* Ganti `/path/to/your/docker-compose.yml` dengan path sesungguhnya di server kamu.
* Server harus sudah terinstall Docker & docker-compose.
* Jangan lupa `docker-compose.yml` di server sudah mengarah ke image dari Docker Hub.






