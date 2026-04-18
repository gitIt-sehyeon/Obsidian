# Environment Variables 공격

## 1. Environment Variables란?

프로그램이 실행되기 **전에** 사용자가 설정할 수 있는 변수들입니다.  
프로그램 코드 안에는 보이지 않지만, **프로그램의 동작에 영향을 줄 수 있습니다.**

```bash
# 환경변수 확인
$ env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
HOME=/home/bob
USER=bob
...
```

Set-UID 프로그램은 root 권한으로 실행되지만, 환경변수는 **실행 전 사용자가 마음대로 설정**할 수 있기 때문에 공격 벡터가 됩니다.

---

## 2. PATH Environment Variable 공격

### PATH가 뭐냐?

터미널에서 명령어를 입력할 때, 전체 경로를 안 써도 되는 이유가 PATH 덕분입니다.

```bash
# 전체 경로 없이 그냥 ls 입력해도 동작하는 이유
$ ls

# 쉘이 PATH를 보고 ls 실행파일을 찾아줌
PATH=/usr/local/bin:/usr/bin:/bin  ← 이 순서대로 ls를 찾음
```

---

### system() 함수가 왜 위험한가?

`system("ls")` 같은 코드는 내부적으로 `/bin/sh`를 먼저 실행하고, 그 쉘이 PATH를 사용해서 `ls`를 찾습니다.

```
system("ls") 실행 흐름:

프로그램
  ↓
/bin/sh 실행  ← 쉘을 먼저 띄움
  ↓
PATH를 보고 "ls" 실행파일 위치 탐색
  ↓
/usr/bin/ls 실행
```

---

### 공격 시나리오

**취약한 Set-UID 프로그램 코드:**

```c
int main() {
    system("ls");   // 전체 경로 없이 ls 호출
    return 0;
}
```

**공격자가 하는 일:**

```bash
# 1. /tmp 디렉토리에 가짜 ls 파일 생성
$ echo "/bin/sh" > /tmp/ls     # 실행하면 쉘을 띄우는 가짜 ls
$ chmod 777 /tmp/ls

# 2. PATH를 조작해서 /tmp를 맨 앞에 추가
$ export PATH=/tmp:$PATH

# 3. Set-UID 프로그램 실행
$ ./vulnerable_setuid_program
```

**실행 흐름:**

```
system("ls") 호출
  ↓
/bin/sh가 PATH 순서대로 ls 탐색
  ↓
/tmp 가 PATH 맨 앞 → /tmp/ls 발견!
  ↓
/tmp/ls 실행 = /bin/sh 실행
  ↓
EUID = root 이므로 → root 쉘 획득!
```

---

### 핵심 문제

```
정상:    system("ls") → /usr/bin/ls 실행
공격 후: system("ls") → /tmp/ls(가짜) 실행 → root 쉘
```

프로그램이 **명령어 이름만 쓰고 전체 경로를 명시하지 않아서** 공격자가 PATH를 조작해 엉뚱한 파일을 실행시킬 수 있는 겁니다.

---

### 방어 방법

```c
// ❌ 위험 - ls를 PATH에서 찾음
system("ls");

// ✅ 안전 - 절대 경로 명시
system("/bin/ls");

// ✅ 더 안전 - execve() 사용 (쉘을 거치지 않음)
char *v[3];
v[0] = "/bin/ls";
v[1] = argv[1];
v[2] = NULL;
execve(v[0], v, 0);
```

---

## 3. system() vs execve() 비교

![[Pasted image 20260418234538.png]]
### system() 공격 예시 (슬라이드 catall.c)

```c
// catall.c - 취약한 코드
char *cat = "/bin/cat";
char *command = malloc(strlen(cat) + strlen(argv[1]) + 2);
sprintf(command, "%s %s", cat, argv[1]);
system(command);  // command = "/bin/cat 사용자입력"
```

공격자 입력: `"aa;/bin/sh"`

```
command = "/bin/cat aa;/bin/sh"
              ↓
쉘이 ; 기준으로 두 명령어로 분리해서 실행
  1. /bin/cat aa  → 파일 없음 (오류)
  2. /bin/sh      → root 쉘 실행!
```

### execve() 안전한 이유

```c
// safecatall.c - 안전한 코드
v[0] = "/bin/cat";
v[1] = argv[1];   // "aa;/bin/sh" 전체가 파일 이름으로 처리됨
v[2] = NULL;
execve(v[0], v, 0);
```

```
execve 실행:
  명령어: /bin/cat (고정, 개발자가 지정)
  데이터: "aa;/bin/sh" (그냥 파일 이름으로만 인식)
  결과: /bin/cat: aa;/bin/sh: No such file or directory
  → 공격 실패!
```

---

## 4. 다른 언어에서도 동일한 위험

C만의 문제가 아닙니다.

### PHP 예시

```php
<?php
$dir = $_GET['dir'];
system("/bin/ls $dir");   // 사용자 입력이 명령어에 섞임
?>
```

공격 URL:

```
http://localhost/list.php?dir=.;date
```

실제 실행되는 명령:

```bash
/bin/ls .;date    # ; 뒤의 date도 실행됨
```

---

## 5. 핵심 원칙 - Code/Data Isolation

환경변수 공격을 포함한 많은 공격들의 근본 원인은 **코드와 데이터를 섞는 것**입니다.

```
위반 사례:
  system("ls")          → PATH(환경변수)가 코드 실행에 영향
  system("/bin/cat " + user_input)  → 데이터가 코드가 됨
  printf(user_input)    → 데이터가 포맷 스트링(코드)이 됨

올바른 설계:
  명령어(코드)는 개발자가 하드코딩
  사용자 입력(데이터)은 철저히 데이터로만 취급
```

---

## 6. execlp(), execvp() 주의사항

`execve()`와 비슷하지만 **PATH를 사용하는 함수들**이 있어서 주의해야 합니다.

```c
// 위험 - PATH를 사용해서 실행파일 탐색
execlp("ls", "ls", NULL);   // PATH 조작에 취약
execvp("ls", args);          // PATH 조작에 취약

// 안전 - 절대 경로 명시
execve("/bin/ls", args, env);  // PATH 무관
```

> `execlp`, `execvp`, `execvpe`는 파일 이름에 `/`가 없으면 PATH를 사용해서 탐색합니다.  
> → PATH 환경변수 조작 공격에 그대로 노출됩니다.

---

## 7. 요약

```
환경변수 공격의 핵심:
  1. 사용자가 실행 전에 환경변수를 자유롭게 설정 가능
  2. Set-UID 프로그램이 환경변수(PATH)에 의존하는 명령어 실행
  3. 공격자가 PATH를 조작 → 가짜 프로그램 실행 → root 권한 획득

방어:
  - 명령어는 반드시 절대 경로 사용
  - system() 대신 execve() 사용
  - 코드와 데이터를 철저히 분리
```