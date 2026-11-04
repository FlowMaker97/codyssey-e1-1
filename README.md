# Codyssey E1-1: 개발자 작업실 꾸미기

리눅스 터미널, 권한, Docker, Git/GitHub로 개발 환경을 직접 구축하고 검증한 기록입니다.
이 README만 보고 같은 결과를 재현할 수 있게 명령어와 결과를 순서대로 남겼습니다.

## 1. 실행 환경

| 항목 | 내용 |
|---|---|
| OS | Windows 11 + WSL2 Ubuntu |
| Shell | bash |
| Docker | Docker version 29.9.2, build a82e528 |
| Git | git version 2.53.0 |
| 작업 경로 | `~/codyssey/e1-1` |

## 2. 체크리스트

- [x] 터미널 기본 명령 (pwd, ls -la, cd, mkdir, cp, mv, rm, cat, touch)
- [x] 권한 실습 (파일 1개, 디렉토리 1개 변경 전후)
- [x] Docker 설치 및 점검 (`--version`, `info`)
- [x] hello-world 실행
- [x] ubuntu 컨테이너 실행, exec, logs, stats
- [x] Dockerfile로 커스텀 이미지 만들기
- [x] 포트 매핑 후 브라우저 접속
- [x] 바인드 마운트 변경 전후 확인
- [x] 볼륨 데이터 유지 확인
- [x] Git 설정 (user.name, user.email, 기본 브랜치)
- [x] VS Code GitHub 로그인 증거

## 3. 터미널 기본 조작

```bash
pwd
mkdir -p ~/codyssey/e1-1
cd ~/codyssey/e1-1
touch hello.txt
echo "hello codyssey" > hello.txt
cat hello.txt
cp hello.txt hello_copy.txt
mv hello_copy.txt hello_moved.txt
rm hello_moved.txt
ls -la
```

## 4. 권한 실습

권한은 r(읽기)=4, w(쓰기)=2, x(실행)=1을 더한 숫자로 표현합니다. 세 자리는 각각 소유자, 그룹, 기타 사용자입니다.

```bash
# 파일: 644(rw-r--r--) → 600(rw-------)
ls -l hello.txt
chmod 600 hello.txt
ls -l hello.txt

# 디렉토리: 755(rwxr-xr-x) → 700(rwx------)
mkdir perm_dir
ls -ld perm_dir
chmod 700 perm_dir
ls -ld perm_dir
```

변경 후에는 소유자만 파일과 디렉토리에 접근할 수 있습니다.

## 5. Docker 설치 및 점검

```bash
docker --version
docker info
docker run hello-world
docker images
```

## 6. 컨테이너 실행과 관리

```bash
docker run -it --name my-ubuntu ubuntu bash
# 컨테이너 안에서
ls
echo "inside container"
exit

docker ps -a              # exit 후 Exited 상태
docker start my-ubuntu
docker exec -it my-ubuntu bash
exit
docker ps                 # exec에서 나와도 Up 상태 유지
docker logs my-ubuntu
docker stats --no-stream
```

**attach와 exec의 차이**
- `run -it`/`attach`는 컨테이너의 메인 프로세스(bash)에 붙습니다. 여기서 `exit`하면 메인 프로세스가 끝나서 컨테이너도 멈춥니다.
- `exec`는 이미 실행 중인 컨테이너 안에 새 프로세스를 하나 더 띄웁니다. 그래서 `exit`해도 컨테이너는 계속 실행됩니다.

## 7. Dockerfile로 커스텀 웹 서버 만들기

```
site/index.html
Dockerfile
```

```dockerfile
FROM nginx:alpine
COPY site/ /usr/share/nginx/html/
```

| 줄 | 목적 |
|---|---|
| `FROM nginx:alpine` | 가벼운 nginx 웹서버 이미지를 기반으로 사용 |
| `COPY site/ /usr/share/nginx/html/` | 직접 만든 페이지를 nginx가 서비스하는 폴더에 복사 |

```bash
docker build -t my-web:1.0 .
docker run -d --name my-web -p 8080:80 my-web:1.0
```

## 8. 포트 매핑

`-p 8080:80`은 내 컴퓨터 8080번 포트로 들어온 요청을 컨테이너 안 80번 포트(nginx)로 연결한다는 뜻입니다.
브라우저에서 `http://localhost:8080`에 접속해 `Hello Codyssey E1-1`이 표시되는 것을 확인했습니다.

![포트 매핑 결과](screenshots/08_browser_localhost_8080.png)

## 9. 바인드 마운트

```bash
docker run -d --name my-web-bind -p 8081:80 \
  -v $(pwd)/site:/usr/share/nginx/html nginx:alpine
curl localhost:8081
# <h1>Hello Codyssey E1-1</h1>

echo '<h1>Bind mount changed!</h1>' > site/index.html
curl localhost:8081
# <h1>Bind mount changed!</h1>
```

호스트 폴더와 컨테이너 폴더를 연결했기 때문에, 이미지를 다시 빌드하지 않아도 호스트에서 바꾼 내용이 바로 반영됩니다.

## 10. 볼륨 데이터 유지

```bash
docker volume create mydata
docker run --name vol-test -v mydata:/data ubuntu bash -c "echo 'hello volume' > /data/test.txt"
docker rm vol-test
docker run --rm -v mydata:/data ubuntu cat /data/test.txt
# hello volume
docker volume ls
```

컨테이너를 삭제해도 볼륨에 저장된 데이터는 남아 있습니다.

## 11. Git 설정과 GitHub 연동

```bash
git config --global user.name "flowmaker97"
git config --global user.email "<마스킹>"
git config --global init.defaultBranch main
git config --list
# user.name=flowmaker97
# user.email=****@gmail.com
# init.defaultbranch=main

git init
git add .
git commit -m "E1-1 docker web, bind mount, volume"
git remote add origin https://github.com/FlowMaker97/codyssey-e1-1.git
git push -u origin main
```

푸시 때는 비밀번호 대신 GitHub Personal Access Token을 사용했습니다. 토큰은 공개하지 않습니다.

## 12. 트러블슈팅

### 사례 1. mv 대상 파일이 없음
- **문제**: `mv hello_copy.txt ...` 실행 시 `No such file or directory`
- **가설**: 옮길 파일이 아직 만들어지지 않았다.
- **확인**: `ls -la`로 보니 `hello_copy.txt`가 없었다. 앞의 `cp` 명령을 건너뛴 상태였다.
- **해결**: `cp hello.txt hello_copy.txt`를 먼저 실행한 뒤 `mv`를 다시 실행했다.

### 사례 2. sudo 비밀번호를 잊음
- **문제**: `sudo` 실행 시 인증 실패가 반복됐다.
- **가설**: WSL 사용자 비밀번호를 잘못 기억하고 있다.
- **확인**: 여러 번 시도해도 실패했다.
- **해결**: PowerShell에서 `wsl -u root`로 root 접속 후 `passwd flowmaker`로 비밀번호를 재설정했다.

### 사례 3. 명령어 두 줄이 한 줄로 합쳐짐
- **문제**: `executable file not found: bashdocker` 오류
- **가설**: 붙여넣은 두 명령어 사이 줄바꿈이 빠졌다.
- **확인**: 명령어가 `... ubuntu bashdocker run ...`처럼 이어져 있었다. 이어서 `exit`로 WSL 밖으로 나가 PowerShell에서 `docker`를 찾지 못하는 문제도 생겼다.
- **해결**: `wsl`로 다시 들어가 `cd ~/codyssey/e1-1`, `docker rm my-ubuntu` 후 한 줄씩 다시 실행했다.

## 13. 증거 자료

`screenshots/` 폴더에 실행 화면을 모았습니다. 이메일과 토큰 같은 민감 정보는 가렸습니다.

## 14. VS Code GitHub 연동
VS Code에서 GitHub 계정(FlowMaker97)으로 로그인 완료.

![VS Code GitHub 로그인](screenshots/13_vscode_github_login.png)
