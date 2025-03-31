**SSH 키를 서비스별로 구분해서 설정하는 방법**

이 방식은 GitHub과 GitLab 모두를 동시에 사용하는 사람에게 아주 유용해 — 각각의 SSH 키를 등록해서, 충돌 없이 커밋과 푸시를 할 수 있어.

---

## 🛠️ 단계별 SSH 키 구분 설정법 (GitHub & GitLab)

---

### ✅ 1. GitHub용 SSH 키 생성

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519_github
```

- `-t ed25519`: 최신 암호화 방식
- `-C`: 주석 (보통 이메일)
- `-f`: 저장될 키 파일 이름을 지정

이 명령을 실행하면:

- `~/.ssh/id_ed25519_github` (비밀키)
- `~/.ssh/id_ed25519_github.pub` (공개키)

가 생성돼.

---

### ✅ 2. GitLab용 SSH 키 생성

```bash
ssh-keygen -t ed25519 -C "your-other-email@example.com" -f ~/.ssh/id_ed25519_gitlab
```

이렇게 하면:

- `~/.ssh/id_ed25519_gitlab`
- `~/.ssh/id_ed25519_gitlab.pub`

가 생겨.

---

### ✅ 3. SSH 공개키 등록

#### 👉 GitHub 등록

1. GitHub 접속 → **Settings** → **SSH and GPG keys**
2. "New SSH key" 클릭
3. 제목 입력 (예: "MacBook GitHub Key")
4. `~/.ssh/id_ed25519_github.pub` 파일 내용을 복사해서 붙여넣기

```bash
cat ~/.ssh/id_ed25519_github.pub
```

#### 👉 GitLab 등록

1. GitLab 접속 → **Preferences** → **SSH Keys**
2. 같은 방식으로 `~/.ssh/id_ed25519_gitlab.pub` 내용을 붙여넣기

---

### ✅ 4. `~/.ssh/config` 파일 수정

```bash
nano ~/.ssh/config
```

아래 내용을 추가:

```ssh
# GitHub용 설정
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes

# GitLab용 설정
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab
  IdentitiesOnly yes
```

**설명:**

- `Host`: Git에서 인식하는 이름 (여기선 github.com, gitlab.com)
- `HostName`: 실제 접속할 도메인
- `IdentityFile`: 사용할 SSH 개인키
- `IdentitiesOnly yes`: 지정한 키만 사용하라고 명시

---

### ✅ 5. 원격 저장소 설정 확인 및 변경

#### GitHub 프로젝트

```bash
git remote set-url origin git@github.com:username/repo.git
```

#### GitLab 프로젝트

```bash
git remote set-url origin git@gitlab.com:username/repo.git
```

이렇게 하면 `.ssh/config`에서 지정한 키를 각각 사용할 수 있어.

---

### ✅ 6. 연결 테스트

```bash
ssh -T git@github.com
ssh -T git@gitlab.com
```

성공 메시지가 나오면 설정 완료!

예:

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

---

필요하다면 `.ssh/config`에 더 많은 키를 추가할 수도 있어. 또는 `Host github-work`처럼 별칭을 지정해서 `git@github-work:username/repo.git` 형태로 원격 주소를 바꿔 쓰는 방법도 있어. 이것도 설명해줄 수 있어!
