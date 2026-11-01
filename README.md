# 코디세이 개발 입문 — Windows 11 개발 작업실 구축

> 필수 과제 제출용 프로젝트입니다. 실제 실행한 명령과 결과, 스크린샷을 추가한 뒤 제출하세요.
> 예시 명령은 Windows PowerShell 기준입니다.

## 1. 프로젝트 개요

Windows 11에서 PowerShell, Docker Desktop, Git, GitHub를 활용해 개발 환경을 구성합니다. 터미널 파일 조작과 권한 확인, Docker 이미지·컨테이너 운영, Dockerfile 기반 웹 서버 제작, 포트 매핑, 바인드 마운트, 볼륨 영속성을 실습합니다.

## 2. 실행 환경

| 항목 | 실제 환경 |
|---|---|
| OS | Windows 11 |
| Shell | PowerShell (실제 버전 기록) |
| Terminal | Windows Terminal 또는 PowerShell (실제 사용한 것 기록) |
| Docker Desktop | 실제 버전 기록 |
| Docker Engine | `docker --version` 결과 |
| Git | `git --version` 결과 |
| Editor | Visual Studio Code |

버전 확인:

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
$PSVersionTable.PSVersion
docker --version
docker version
git --version
```

## 3. 수행 체크리스트

- [ ] PowerShell 현재 위치·목록·이동·생성·복사·이름 변경·삭제
- [ ] 파일 내용 확인 및 빈 파일 생성
- [ ] 절대 경로와 상대 경로 설명
- [ ] 파일 및 디렉터리 권한 확인·변경 결과 기록
- [ ] Docker Desktop 실행 및 엔진 상태 확인
- [ ] 이미지 다운로드 및 목록 확인
- [ ] hello-world 실행
- [ ] Ubuntu 컨테이너 내부 명령 실행
- [ ] 컨테이너 실행·종료 및 exec/attach 차이 관찰
- [ ] Dockerfile 커스텀 이미지 빌드 및 실행
- [ ] 포트 매핑 후 웹 접속 확인
- [ ] 바인드 마운트 변경 반영 확인
- [ ] Docker 볼륨 영속성 확인
- [ ] Git 사용자 정보 및 main 브랜치 설정
- [ ] GitHub와 VSCode 연동
- [ ] 실제 문제 기반 트러블슈팅 2건 이상 작성

## 4. PowerShell 기본 명령 실습

프로젝트 폴더 안에서 PowerShell을 열고 실행합니다.

```powershell
# 현재 위치와 파일 목록
Get-Location
Get-ChildItem -Force

# 실습 폴더와 파일 만들기
New-Item -ItemType Directory -Force practice, practice\backup, practice\source
New-Item -ItemType File -Force practice\notes.txt
Set-Content practice\notes.txt "PowerShell terminal practice"
Get-Content practice\notes.txt

# 복사, 이름 변경, 이동
Copy-Item practice\notes.txt practice\backup\notes-copy.txt
Rename-Item practice\notes.txt notes-renamed.txt
Move-Item .\notes-renamed.txt practice\source\

# 파일·폴더 정보 확인
Get-Item practice\backup\notes-copy.txt
Get-ChildItem practice -Force

# 삭제 실습
Remove-Item practice\backup\notes-copy.txt
```

### 절대 경로와 상대 경로

- 절대 경로: 드라이브 문자부터 전체 위치를 표현합니다. 예: `C:\Users\사용자\project\site\index.html`
- 상대 경로: 현재 위치를 기준으로 표현합니다. 예: `.\site\index.html`

### Windows 파일 권한 실습

Windows의 권한은 Linux의 `chmod 644`, `chmod 755`와 동일한 방식으로 적용되지 않습니다. NTFS ACL을 `icacls`로 확인하고 변경합니다. 아래 명령은 현재 사용자에게 파일 읽기·쓰기 권한을 부여하는 예시입니다.

```powershell
# 테스트 파일 생성
New-Item -ItemType Directory -Force .\permission-practice
Set-Content .\permission-practice\sample.txt "permission test"

# 변경 전 권한 확인
icacls .\permission-practice\sample.txt
icacls .\permission-practice

# 현재 사용자에게 파일 읽기/쓰기 권한 부여
icacls .\permission-practice\sample.txt /grant "$($env:USERNAME):(R,W)"

# 변경 후 권한 확인
icacls .\permission-practice\sample.txt
```

실습한 PC의 계정과 권한 결과를 캡처합니다. 권한 변경 전후 출력은 실제 결과로 기록합니다.

## 5. Docker Desktop 실행 및 점검

1. 시작 메뉴에서 Docker Desktop을 실행합니다.
2. Docker Desktop 화면에서 엔진이 실행 상태가 될 때까지 기다립니다.
3. PowerShell을 새로 열고 다음 명령을 실행합니다.

```powershell
docker --version
docker version
docker info
docker images
docker ps
docker ps -a
```

`docker info`에서 서버 정보가 출력되면 Docker 엔진과 통신할 수 있는 상태입니다. 오류가 발생하면 오류 문구를 그대로 기록하고 해결 과정을 트러블슈팅에 작성합니다.

### 이미지와 컨테이너 운영

```powershell
docker pull hello-world
docker images
docker run --name hello-check hello-world
docker ps -a
docker logs hello-check
```

리소스 확인은 실행 중인 컨테이너가 있을 때 실행합니다.

```powershell
docker stats --no-stream
```

## 6. 컨테이너 실습

### Ubuntu 컨테이너

```powershell
docker run --name ubuntu-practice -it ubuntu bash
```

컨테이너 내부에서 실행:

```bash
ls
echo "Hello Docker"
exit
```

호스트 PowerShell로 돌아와 상태 확인:

```powershell
docker ps -a
```

### 종료 방식 정리

- `exit`: 컨테이너 내부 셸을 종료합니다. 해당 셸이 주 프로세스인 경우 컨테이너도 종료됩니다.
- `docker exec`: 실행 중인 컨테이너에 별도 명령/프로세스를 실행합니다.
- `docker attach`: 컨테이너의 주 프로세스 입출력에 연결합니다.

직접 관찰한 상태와 명령 결과를 기록합니다.

## 7. Dockerfile 기반 커스텀 웹 서버

이 프로젝트는 공식 `nginx:alpine` 이미지를 기반으로 직접 작성한 웹 페이지를 포함합니다.

### Dockerfile

```dockerfile
FROM nginx:alpine

LABEL org.opencontainers.image.title="codyssey-custom-nginx"
ENV APP_ENV=dev

COPY site/ /usr/share/nginx/html/
```

### 커스텀 설정 목적

| 설정 | 목적 |
|---|---|
| `FROM nginx:alpine` | NGINX 웹 서버 베이스 이미지 사용 |
| `LABEL` | 이미지 메타데이터 기록 |
| `ENV` | 컨테이너 환경 변수 설정 |
| `COPY` | 작성한 웹 페이지를 이미지 안으로 복사 |

### 빌드 및 실행

저장소 루트에서 PowerShell로 실행합니다.

```powershell
docker build -t codyssey-web:1.0 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:1.0
docker ps
```

브라우저에서 `http://localhost:8080`에 접속합니다.

```powershell
Invoke-WebRequest http://localhost:8080
docker logs codyssey-web
```

컨테이너 이름이 이미 사용 중이면 기존 컨테이너를 정리한 뒤 다시 실행합니다.

```powershell
docker rm -f codyssey-web
```

## 8. 포트 매핑 검증

`-p 8080:80`은 Windows 호스트의 8080번 포트를 컨테이너 내부 NGINX의 80번 포트에 연결합니다.

- 호스트 포트: `8080`
- 컨테이너 포트: `80`
- 접속 주소: `http://localhost:8080`

브라우저 주소창과 실제 페이지 내용이 함께 보이도록 캡처합니다.

## 9. 바인드 마운트 변경 반영

PowerShell에서 저장소 루트 디렉터리에 있는지 확인하고 실행합니다.

```powershell
docker run -d --name codyssey-bind -p 8081:80 --mount "type=bind,source=$((Resolve-Path .\site).Path),target=/usr/share/nginx/html" nginx:alpine
```

1. `site\index.html`을 열어 기존 내용을 확인합니다.
2. 문구를 수정하고 저장합니다.
3. `http://localhost:8081`을 새로고침합니다.
4. 변경 사항이 반영되는지 확인하고 전후 화면을 캡처합니다.

```powershell
Get-Content .\site\index.html
docker logs codyssey-bind
docker rm -f codyssey-bind
```

## 10. Docker 볼륨 영속성

아래 실습은 컨테이너 삭제 후에도 named volume의 데이터가 유지되는지 확인합니다.

```powershell
docker volume create codyssey-data

docker run -d --name volume-before --mount source=codyssey-data,target=/data ubuntu sleep infinity

docker exec volume-before bash -lc "echo 'persistent data' > /data/check.txt"
docker exec volume-before cat /data/check.txt

docker rm -f volume-before

docker run -d --name volume-after --mount source=codyssey-data,target=/data ubuntu sleep infinity

docker exec volume-after cat /data/check.txt
```

마지막 명령에서 `persistent data`가 출력되는지 확인합니다. 증거 캡처 후 컨테이너를 정리합니다.

```powershell
docker rm -f volume-after
docker volume ls
```

볼륨 자체를 삭제하면 데이터도 사라질 수 있으므로 검증 전에는 `docker volume rm`을 실행하지 않습니다.

## 11. Git 및 GitHub

Git 버전과 사용자 정보를 설정합니다.

```powershell
git --version
git config --global user.name "본인 이름"
git config --global user.email "본인 이메일"
git config --global init.defaultBranch main
git config --list
```

GitHub에서 빈 저장소를 만든 뒤 실제 저장소 주소로 바꾸어 실행합니다.

```powershell
git init
git add .
git status
git commit -m "Build Windows 11 developer workstation"
git branch -M main
git remote add origin https://github.com/본인계정/저장소이름.git
git push -u origin main
```

이미 `origin`이 등록되어 있다면 `git remote add origin` 대신 `git remote set-url origin 실제저장소주소`를 사용합니다.

VSCode에서 GitHub 로그인을 완료하고 저장소가 연결된 화면을 캡처합니다. 토큰, 비밀번호, 인증 코드가 화면에 보이지 않도록 합니다.

## 12. 트러블슈팅

실제로 발생한 문제만 아래 양식으로 2건 이상 작성합니다.

### 사례 1

- 문제:
- 원인 가설:
- 확인 명령 및 결과:
- 해결 또는 대안:
- 해결 후 재검증 결과:

### 사례 2

- 문제:
- 원인 가설:
- 확인 명령 및 결과:
- 해결 또는 대안:
- 해결 후 재검증 결과:

## 13. 증거 자료

실제 캡처를 `screenshots/`에 저장하고 아래 목록을 채웁니다.

| 파일명 예시 | 증거 내용 |
|---|---|
| `terminal-basics.png` | PowerShell 명령과 출력 |
| `permission-before-after.png` | ACL 권한 변경 전후 |
| `docker-version-info.png` | Docker 버전 및 엔진 정보 |
| `hello-world.png` | hello-world 성공 |
| `ubuntu-container.png` | Ubuntu 내부 명령 |
| `docker-build.png` | 이미지 빌드 결과 |
| `web-browser.png` | 주소창과 웹 페이지 |
| `bind-mount.png` | 호스트 변경 반영 |
| `volume-persistence.png` | 컨테이너 삭제 후 데이터 유지 |
| `git-github-vscode.png` | Git 설정 및 VSCode 연동 |

예시 이미지 삽입:

```markdown
![Docker 실행 확인](screenshots/docker-version-info.png)
```

## 14. 보안 주의사항

- 비밀번호, 토큰, 개인키, 인증 코드를 저장소에 올리지 않습니다.
- `.env`, 개인 인증 파일, 비밀키는 커밋하지 않습니다.
- `git config --list`에 이메일 등 공개를 원하지 않는 정보가 있으면 가립니다.
- 민감정보가 원격 저장소에 올라갔다면 파일에서 지우는 것만으로 충분하지 않을 수 있습니다. 해당 자격 증명을 폐기·재발급하고 Git 이력도 정리해야 합니다.

## 15. 최종 학습 정리

이번 과제를 통해 PowerShell을 이용한 파일 관리와 Windows 권한 확인, Docker 이미지와 컨테이너 운영, Dockerfile 작성, 포트 매핑, 바인드 마운트, 볼륨 영속성, Git 및 GitHub 연동을 실습합니다.

**GitHub Repository:** 실습 완료 후 주소 입력  
**진행 상태:** 실습 및 증거 수집 후 완료로 변경
