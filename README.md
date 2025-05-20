# Docker에서 Django 서비스 운영

## Docker를 사용하는 이유

- **개발, 테스트, 배포 환경을 일관되게 유지하면서, 효율적이고 확장 가능한 시스템을 운영**하기 위함
- 다음과 같은 장점

---

## ✅ 1. **환경 일관성 (It works on my machine 방지)**

- Docker 컨테이너는 실행 환경(OS, 라이브러리, 설정 등)을 이미지로 캡슐화함
- 개발자가 만든 환경 그대로 **테스트 서버, 운영 서버**에서도 동일하게 실행 가능

🎯 예시:

- 개발자 A가 만든 Django 앱이 다른 팀원 혹은 서버에서도 **그대로 실행됨**

---

## ✅ 2. **배포 자동화 및 DevOps 통합**

- CI/CD 파이프라인과 쉽게 통합 가능 (예: GitHub Actions, Jenkins, GitLab CI)
- `docker-compose`, `Dockerfile`, `.env` 등으로 설정을 코드화 (Infrastructure as Code)

🎯 예시:

- `git push`만 하면 자동으로 Docker 이미지 빌드 → 서버에 배포

---

## ✅ 3. **프로세스 격리 및 보안**

- 각 컨테이너는 **독립된 리눅스 공간**에서 실행됨 → 서로 간섭 없이 안전
- 프로덕션 환경에서 **서비스 단위 분리** 가능 (ex. Django, PostgreSQL, Redis 따로 격리)

---

## ✅ 4. **이식성 & 확장성**

- 어떤 OS든 Docker만 설치하면 동일한 이미지 실행 가능 (Windows, macOS, Linux)
- Kubernetes, AWS ECS, Azure Container Apps 등에서 **쉽게 확장**

---

## ✅ 5. **경량화된 VM과 같은 역할**

- Docker는 전체 OS를 포함하지 않고 커널 공유로 가벼움 (→ 실행 빠름, 자원 효율적)
- 기존 VM 대비 수배 빠른 기동 속도

---

## ✅ 6. **버전 관리 및 롤백 용이**

- Docker 이미지는 **버전별로 저장**되므로 언제든 과거 버전으로 롤백 가능

🎯 예시:

`pybo:1.0` → `pybo:2.0`으로 배포했더니 오류 발생 → 바로 1.0으로 복구 가능

---

## ✅ 7. **복잡한 의존성 해결**

- Python, Node.js, Java 등 다양한 앱의 라이브러리 충돌 없이 동시에 구동 가능
- 시스템 라이브러리 버전이 달라도 문제 없음

---

## ✨ 요약: 왜 Docker를 쓰는가?

| 목적 | 설명 |
| --- | --- |
| 환경 일관성 | 개발 ↔ 운영 환경 차이 제거 |
| 배포 자동화 | CI/CD와 자연스럽게 통합 |
| 경량 격리 | VM보다 가볍고 빠르며 안전 |
| 확장성 | Kubernetes 등과 연계 쉬움 |
| 관리 편의 | Dockerfile로 설정 버전 관리 |

---

---

### ✅ 장고를 도커에서 운영할 때의 장점

1. **환경 일관성 유지**
    - 개발자 로컬 환경, 테스트 서버, 운영 서버 간의 환경 차이 최소화
2. **의존성 격리**
    - Python, PostgreSQL, Redis 등의 의존성을 컨테이너로 분리
3. **배포 자동화 용이**
    - Docker Compose 및 CI/CD 파이프라인과 연동하여 자동 배포 가능
4. **확장성**
    - 여러 컨테이너 (ex. Django + Gunicorn + Nginx + DB)를 조합한 마이크로서비스 구조 가능

---

### 🔧 기본 구조 (Django + PostgreSQL 예시)

```
/project-root
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── myproject/     <- 장고 프로젝트 폴더

```

---

### 📁 `Dockerfile` 예시 (Django 애플리케이션용)

```
# Python 베이스 이미지
FROM python:3.11-slim

# 작업 디렉터리 설정
WORKDIR /app

# 의존성 복사 및 설치
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 프로젝트 복사
COPY . .

# 포트 오픈
EXPOSE 8000

# 명령어 실행 (예: 개발 서버)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

```

---

### 📁 `docker-compose.yml` 예시

```yaml
version: '3.9'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    depends_on:
      - db
    environment:
      - DEBUG=True
      - DJANGO_SETTINGS_MODULE=myproject.settings
      - DB_HOST=db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypass
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:

```

---

### ⚙️ 실행 명령어

```bash
# 빌드 및 실행
docker-compose up --build

# 초기 마이그레이션 및 관리자 생성
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser

```

---

### Docker Desktop Windows 설치

[](https://tse4.mm.bing.net/th?id=OIP.bNAlAmp8WYtklZ6d3cjZ7AHaE-&pid=Api)

- WSL2(Windows Subsystem for Linux 2)를 기반으로 하며, 최신 Windows 10/11 환경을 기준으로 작성.

---

## 🧰 사전 준비 사항

- **운영 체제**: Windows 10/11 64비트 (버전 22H2 이상)
- **필수 기능**:
    - **WSL2**: 리눅스 컨테이너 실행을 위한 필수 구성 요소
    - **가상화 지원**: BIOS/UEFI에서 가상화(Virtualization) 기능이 활성화되어 있어야 함.
- **하드웨어 요구 사항**:
    - 64비트 프로세서 (SLAT 지원)
    - 최소 4GB RAM

---

## 📥 1단계: Docker Desktop 설치 파일 다운로드

1. [Docker 공식 다운로드 페이지](https://www.docker.com/products/docker-desktop)로 이동.
2. "Download for Windows" 버튼을 클릭하여 설치 파일을 다운로드.

---

## ⚙️ 2단계: WSL2 활성화 및 설치

- Docker Desktop은 WSL2를 백엔드로 사용. WSL2를 설치하려면 다음 단계 사용:
1. **PowerShell을 관리자 권한으로 실행**.
2. 다음 명령어를 입력하여 WSL을 설치:
    
    ```powershell
    wsl --install
    
    ```
    
    - 이 명령은 WSL2와 기본 리눅스 배포판(Ubuntu)을 설치.
3. 설치가 완료되면 시스템을 재부팅.

> 참고: WSL2 설치에 대한 자세한 정보는 Microsoft 공식 문서를 참고.
> 

---

## 🖥️ 3단계: Docker Desktop 설치 및 설정

1. 다운로드한 `Docker Desktop Installer.exe` 파일을 실행.
2. 설치 마법사가 시작되면, "Use WSL 2 instead of Hyper-V" 옵션이 선택되어 있는지 확인.
3. 설치를 진행하고, 완료 후 시스템을 재부팅.

---

## 🚀 4단계: Docker Desktop 실행 및 테스트

1. **Docker Desktop 실행**:
    - 시작 메뉴에서 "Docker Desktop"을 검색하여 실행.
    - 시스템 트레이에 고래 아이콘이 나타나면 Docker가 실행 중.
2. **설치 확인**:
    - PowerShell 또는 명령 프롬프트를 열고 다음 명령어를 입력
        
        ```powershell
        docker version
        
        ```
        
        Docker의 클라이언트와 서버 버전 정보가 출력되면 설치가 정상적으로 완료된 것
        
3. **테스트 컨테이너 실행**:
    - 다음 명령어를 입력하여 테스트 컨테이너를 실행:
        
        ```powershell
        docker run hello-world
        
        ```
        
        "Hello from Docker!" 메시지가 출력되면 Docker가 정상적으로 작동하는 것.
        

---

## `pybo` Django 프로젝트를 **Docker에서 서비스하기**

---

## ✅ 목표

> pybo (질문/답변 게시판 Django 앱)을 Docker 환경에서 PostgreSQL과 함께 실행
> 

---

## 📁 1. 디렉토리 구조 준비

`djangobook-3-12/` 디렉토리 기준:

```
djangobook-3-12/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── manage.py
├── config/
├── pybo/
├── common/
├── static/
├── templates/
└── db.sqlite3  # ← 이후 삭제 예정 (PostgreSQL로 이동)

```

---

## ⚙️ 2. Dockerfile 생성

`djangobook-3-12/Dockerfile`

```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

```

---

## 📄 3. requirements.txt 생성

```
Django>=4.2
psycopg2-binary

```

> psycopg2-binary는 PostgreSQL 연결을 위한 Python 패키지입니다.
> 

---

## 🐘 4. PostgreSQL 포함 docker-compose.yml 생성

`djangobook-3-12/docker-compose.yml`

```yaml
version: '3.9'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    depends_on:
      - db
    environment:
      - DB_NAME=pybo_db
      - DB_USER=pybo_user
      - DB_PASSWORD=pybo_pass
      - DB_HOST=db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: pybo_db
      POSTGRES_USER: pybo_user
      POSTGRES_PASSWORD: pybo_pass
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:

```

---

## 🔧 5. settings.py 수정 (`config/settings.py`)

기존 `DATABASES` 설정을 다음으로 교체:

```python
import os

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'pybo_db'),
        'USER': os.environ.get('DB_USER', 'pybo_user'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'pybo_pass'),
        'HOST': os.environ.get('DB_HOST', 'db'),
        'PORT': '5432',
    }
}

```

`ALLOWED_HOSTS = ['*']` 설정도 꼭 확인하세요.

---

## 🗑️ 6. 기존 SQLite 파일 제거

```bash
rm db.sqlite3

```

---

## 🚀 7. Docker 실행

```bash
# 터미널에서 해당 폴더로 이동
cd djangobook-3-12

# 도커 빌드 및 실행
docker compose up --build

```

---

## 🛠️ 8. 마이그레이션 및 관리자 생성

```bash
# 마이그레이션
docker compose exec web python manage.py migrate

# 슈퍼유저 생성
docker compose exec web python manage.py createsuperuser

```

---

## ✅ 결과 확인

- 브라우저에서 `http://localhost:8000` 접속
- pybo 게시판 정상 실행
- `/admin`으로 관리자 로그인도 가능

---

## Docker 환경이 적용된 `pybo` Django 프로젝트

Docker 환경이 적용된 `pybo` Django 프로젝트 

---

## ✅ 포함된 내용

| 파일 | 설명 |
| --- | --- |
| `Dockerfile` | Django 앱 빌드를 위한 설정 |
| `docker-compose.yml` | Django + PostgreSQL 통합 실행 환경 |
| `requirements.txt` | Django 및 PostgreSQL 드라이버 등 의존성 |
| `config/settings.py` | PostgreSQL 연결을 위한 DB 설정 수정 필요 |
| `pybo/`, `common/`, `templates/`, `static/` | 기존 pybo 앱 전체 포함 |

---

## 🚀 사용 방법 (Docker 설치 후)

```bash
cd pybo_dockerized

# 1. Docker 빌드 및 실행
docker compose up --build

# 2. 마이그레이션
docker compose exec web python manage.py migrate

# 3. 관리자 계정 생성
docker compose exec web python manage.py createsuperuser

```

이후 브라우저에서 `http://localhost:8000` 접속 시 `pybo` 실행 확인 가능

---

## ❗ 주의 사항

- `config/settings.py`에 아래 설정이 포함되어 있어야 합니다:

```python
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'pybo_db'),
        'USER': os.environ.get('DB_USER', 'pybo_user'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'pybo_pass'),
        'HOST': os.environ.get('DB_HOST', 'db'),
        'PORT': '5432',
    }
}

```

- `db.sqlite3`는 더 이상 사용하지 않으며 PostgreSQL로 완전히 전환됩니다.

---

- `pybo` Django 프로젝트가 Docker 환경에서 **정상적으로 서비스되고 있음을 나타내는 로그**.

```jsx
web-1  | [15/May/2025 12:39:02] "GET /pybo HTTP/1.1" 301 0
web-1  | [15/May/2025 12:39:02] "GET /pybo/ HTTP/1.1" 200 7973
web-1  | [15/May/2025 12:39:02] "GET /static/style.css HTTP/1.1" 200 65
web-1  | [15/May/2025 12:39:02] "GET /static/jquery-3.4.1.min.js HTTP/1.1" 200 88145
web-1  | [15/May/2025 12:39:02] "GET /static/bootstrap.min.css HTTP/1.1" 200 159515
web-1  | [15/May/2025 12:39:02] "GET /static/bootstrap.min.js HTTP/1.1" 200 60010
web-1  | Not Found: /favicon.ico
web-1  | [15/May/2025 12:39:03] "GET /favicon.ico HTTP/1.1" 404 2663
```

✅ 완벽하게 작동 중

---

## 🔍 로그 해석

| 요청 | 상태 | 설명 |
| --- | --- | --- |
| `GET /pybo` → 301 | 🔁 리디렉션 | `/pybo` 요청이 자동으로 `/pybo/`로 리디렉션됨 |
| `GET /pybo/` → 200 | ✅ 성공 | pybo 질문 리스트 페이지 정상 렌더링 |
| `GET /static/*.css/.js` → 200 | ✅ 성공 | 정적 리소스 (CSS/JS) 모두 잘 불러옴 |
| `GET /favicon.ico` → 404 | ⚠️ 없음 | 브라우저 탭 아이콘 요청 → 파일 없어서 404 (무시 가능) |

---

## 결과 화면 1

- **docker compose up —build**

![image.png](image.png)

## 결과 화면 2

- [web] [db] 작동 로그 화면

![image.png](image%201.png)

## 결과 화면 3

- [web] [db] 작동 로그 화면

![image.png](image%202.png)

## 결과 화면 4

- [web] 작동 로그 화면

![image.png](image%203.png)

## 결과 화면 5

- 로컬호스트 운영 화면

![image.png](image%204.png)
