### dyld란?

**dyld (Dynamic Linker/Loader)** 는 macOS에서 프로그램 실행 시 필요한 **동적 라이브러리(.dylib)들을 메모리에 로드**해주는 시스템 컴포넌트입니다.

Java의 ClassLoader랑 비슷한 역할이에요. 프로그램이 `printf()` 같은 함수를 쓸 때, 그 함수가 담긴 라이브러리를 실행 전에 찾아서 연결해주는 역할입니다.

`DYLD_PRINT_TO_FILE`은 dyld가 이 과정에서 출력하는 **디버그 로그를 어느 파일에 쓸지** 지정하는 환경변수입니다.

---

### 취약점 흐름 단계별 설명

#### 1단계 - 환경변수 설정

bash

```bash
DYLD_PRINT_TO_FILE=/etc/sudoers
```

"dyld야, 앞으로 로그를 `/etc/sudoers`에 써줘" 라고 설정.

아직 아무 일도 안 일어남.

#### 2단계 - SetUID 프로그램 실행

bash

```bash
su bob
```

`su`는 **SetUID 비트**가 설정된 프로그램입니다. 즉, **실행 순간 root 권한으로 동작**합니다.

원래 SetUID 프로그램은 보안상 `DYLD_*` 환경변수를 **무시해야** 하는데, macOS 10.10의 dyld가 이걸 **무시하지 않았습니다.** 이게 첫 번째 버그.

그래서 `su`가 실행될 때 dyld는 **root 권한으로** `/etc/sudoers`를 열어버립니다.

#### 3단계 - Capability Leak

bash

```bash
echo "bob ALL=(ALL) NOPASSWD:ALL" >&3
```

`su`가 `/etc/sudoers`를 열고 fd(파일 디스크립터)를 **닫지 않았습니다.** 이게 두 번째 버그 (Capability Leak).

fd 3번이 **root 권한으로 열린 `/etc/sudoers`를 그대로 가리키고 있는 상태**라서, 일반 유저가 `>&3`으로 내용을 써버릴 수 있게 됩니다.

---

### 핵심 요약

```
일반 유저
  → 환경변수로 sudoers 지정
    → su (SetUID) 실행 → root 권한으로 sudoers fd 열림
      → su가 fd를 안 닫음 (Capability Leak)
        → 일반 유저가 열린 fd로 sudoers 직접 수정
          → 본인에게 sudo 권한 부여
```

`su bob`으로 계정을 바꾼 게 목적이 아니라, **SetUID 프로그램인 `su`를 트리거로 사용해서 root 권한의 fd를 얻는 것**이 핵심입니다.