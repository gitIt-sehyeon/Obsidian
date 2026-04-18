# Capability Leaking 공격

## 1. Capability Leaking이란?

특권 프로그램이 실행 중에 **권한을 낮추는(downgrade) 과정에서,  
이미 획득한 특권 능력을 제대로 정리하지 않고 남겨두는 것**입니다.

```
권한은 낮아졌는데,
root 때 열어둔 파일 같은 것들이 그대로 살아있어서
공격자가 그걸 이용할 수 있음
```

---

## 2. File Descriptor(fd)가 뭐냐?

```c
fd = open("/etc/zzz", O_RDWR | O_APPEND);
```

`fd`는 **열린 파일을 가리키는 번호표**입니다.

```
open("/etc/zzz") 호출
  ↓
OS가 파일을 열고 번호표를 발급
  ↓
fd = 3   ← "3번 티켓으로 이 파일 접근 가능"
```

중요한 점은 **fd가 살아있는 한, 권한 체크 없이 그 파일에 계속 접근 가능**합니다.  
처음 열 때 root 권한으로 열었으면, 이후에 권한이 낮아져도 fd로는 계속 쓸 수 있어요.

---
## setuid() / getuid() 함수

### getuid() - 내가 누군지 알아오는 것

```c
getuid()   // Real UID를 반환
```

```
bob(UID=1000)이 프로그램 실행
  ↓
getuid() = 1000   ← "진짜 나는 bob"
```

---

### setuid() - EUID를 바꾸는 것

```c
setuid(1000)   // EUID를 1000으로 변경
```

```
Set-UID 프로그램 실행 직후:
  RUID = 1000 (bob)
  EUID = 0    (root)  ← Set-UID 비트 때문에 승격

setuid(getuid()) 호출:
  RUID = 1000 (bob)
  EUID = 1000 (bob)   ← root에서 일반 사용자로 내려옴
```

---
## 3. 슬라이드 코드 단계별 분석

### 전체 코드

```c
// [1] root 권한으로 /etc/zzz 열기
fd = open("/etc/zzz", O_RDWR | O_APPEND);
if (fd == -1) {
    printf("Cannot open /etc/zzz\n");
    exit(0);
}

// [2] fd 번호 출력
printf("fd is %d\n", fd);

// [3] 권한 다운그레이드 (EUID를 RUID로 낮춤)
setuid(getuid());

// [4] 쉘 실행
v[0] = "/bin/sh"; v[1] = 0;
execve(v[0], v, 0);
```

---
### 1단계 - root 권한으로 파일 오픈

```
Set-UID 프로그램 실행
  ↓
EUID = 0 (root)
  ↓
/etc/zzz 는 root만 쓸 수 있는 파일
  ↓
root 권한으로 /etc/zzz 열기 성공
  ↓
fd = 3  ← 파일 디스크립터 번호 발급
```

---

### 2단계 - 권한 다운그레이드 (근데 fd를 안 닫음!)

```c
setuid(getuid());   // EUID를 일반 사용자로 낮춤
```

```
EUID: 0 (root) → 1000 (일반 사용자)
RUID: 1000 (그대로)

근데 fd = 3 은 그대로 살아있음!!
  ↓
이미 열린 파일은 닫지 않는 한 계속 유효
```

---

### 3단계 - 쉘 실행

```c
v[0] = "/bin/sh"; v[1] = 0;
execve(v[0], v, 0);
```

```
권한은 낮아진 상태로 쉘을 실행
  ↓
쉘에서 EUID = 1000 (일반 사용자)
  ↓
근데 fd = 3 이 쉘에도 그대로 상속됨!
```

---

## 4. 실제 공격 (슬라이드 2번 참고)

```bash
# /etc/zzz 내용 확인
$ cat /etc/zzz
bbbbbbbbbbbbbbbb

# 일반 사용자로는 직접 못 씀
$ echo aaaaaaaaa > /etc/zzz
bash: /etc/zzz: Permission denied   ← 직접 쓰기 불가

# cap_leak 실행 → 쉘이 뜸 (fd = 3 이 살아있는 상태로)
$ cap_leak
fd is 3

# fd = 3 번으로 직접 쓰기 (권한 체크 없이!)
$ echo ccccccccccc >& 3   ← 누출된 fd 사용!

$ exit

# 결과 확인
$ cat /etc/zzz
bbbbbbbbbbbbbbbb
ccccccccccc                         ← 파일이 수정됨!
```

---

## 5. 왜 이게 가능한가?

```
파일 접근 권한 체크는 open() 할 때 딱 한 번만 함
  ↓
open() 성공 → fd 발급
  ↓
이후에는 fd만 있으면 권한 체크 없이 read/write 가능
  ↓
권한이 낮아져도 fd는 그대로 유효
  ↓
쉘에 fd가 상속되면 → 일반 사용자가 root 전용 파일에 쓸 수 있음
```

---

## 6. su 프로그램이 이 패턴을 쓰는 이유

슬라이드에서 `su` 프로그램을 예시로 든 이유가 있습니다.

```
su 실행 흐름:
  [1] EUID = root, RUID = user1 으로 시작
  [2] user2의 비밀번호 검증
  [3] 검증 성공 → EUID = user2, RUID = user2 로 다운그레이드
  [4] user2의 쉘 실행

문제:
  [3]에서 다운그레이드 전에 열어둔 파일이나 fd가 있으면
  user2의 쉘에 그대로 상속됨
```

---

## 7. 방어 방법

```c
// ❌ 취약한 코드
fd = open("/etc/zzz", O_RDWR | O_APPEND);  // root로 열기
setuid(getuid());                            // 권한 낮춤
execve("/bin/sh", v, 0);                    // fd가 쉘에 상속됨!

// ✅ 안전한 코드
fd = open("/etc/zzz", O_RDWR | O_APPEND);  // root로 열기
// ... 필요한 작업 수행 ...
close(fd);                                   // fd 먼저 닫기!
setuid(getuid());                            // 그 다음 권한 낮춤
execve("/bin/sh", v, 0);                    // 이제 fd 없음
```

**핵심: 권한을 낮추기 전에 특권으로 열어둔 모든 fd를 닫아야 합니다.**

---

## 8. 요약

```
Capability Leaking 핵심:

  발생 조건:
    특권(root)으로 파일을 열고 fd를 받음
    fd를 닫지 않고 권한을 낮춤
    낮아진 권한의 쉘에 fd가 상속됨

  공격 방법:
    echo data >& [fd번호]  로 root 전용 파일에 쓰기

  방어:
    권한 다운그레이드 전에 반드시 close(fd) 호출
```