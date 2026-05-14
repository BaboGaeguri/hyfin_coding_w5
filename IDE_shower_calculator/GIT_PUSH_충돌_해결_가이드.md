# Git Push 거부(non-fast-forward) 문제 해결 가이드

## 1. 마주친 에러

```
$ git push
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/.../...git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
```

이어서 `git pull`을 시도하면 다음 에러가 추가로 발생할 수 있다.

```
hint: You have divergent branches and need to specify how to reconcile them.
fatal: Need to specify how to reconcile divergent branches.
```

---

## 2. 핵심 원인: 브랜치 갈라짐(divergent branches)

### 상황
- **나**: 로컬에서 새 커밋(예: `sangyun`)을 만들고 push 시도
- **팀원**: 그 사이에 원격(GitHub)에 다른 커밋들을 먼저 push 완료

### Git이 인식하는 상태

```
                       ┌─ b47c24d (내 커밋: sangyun)        ← local main
공통조상 065bbd2 ──────┤
                       └─ af0cc4a ── 5460c3b ── 9f50be7      ← origin/main (팀원 커밋들)
```

같은 공통 조상에서 출발했지만, 양쪽이 **각자 새 커밋을 만들면서 Y자로 분기**된 상태.

### "브랜치가 main 하나뿐인데 왜 갈라졌다고 하지?"

`git branch --all`로 보면 `main` 하나로 보이지만, 실제로는 **이름만 같은 2개의 포인터**가 존재한다.

| 포인터 | 가리키는 곳 |
|---|---|
| `main` | 내 로컬의 main이 가리키는 커밋 |
| `origin/main` | 마지막 fetch 시점에 본 GitHub의 main |

이 둘이 서로 다른 줄기로 분기되어 있을 때 Git은 "divergent"라고 부른다.
즉 **"브랜치 이름이 여러 개"라는 뜻이 아니라 "같은 main이 가리키는 커밋이 둘로 갈라졌다"는 뜻**.

---

## 3. 왜 이전엔 이런 일이 없었나?

이전 협업 경험에서는 보통 아래 패턴이었다.

```
팀원: 커밋 + push  →  내가 pull  →  내가 커밋 + push
```

이 경우 내가 pull 받기 전까지 **내 로컬엔 새 커밋이 없었기** 때문에 단순히 원격을 따라잡기만 하면 됐다 → **fast-forward** (분기 없음).

### 발생 조건 정리

| 내 로컬에 새 커밋? | 원격에 새 커밋? | 결과 |
|:---:|:---:|:---|
| ❌ | ✅ | `git pull` 정상 (fast-forward) |
| ✅ | ❌ | `git push` 정상 (fast-forward) |
| ✅ | ✅ | **갈라짐 → 봉합 작업 필요** |
| ❌ | ❌ | up to date |

> VS Code의 소스 컨트롤 GUI를 쓰든 터미널을 쓰든 **동일하게 발생**한다. GUI는 내부적으로 똑같은 `git` 명령을 실행하는 껍데기일 뿐이다.

---

## 4. 해결 방법

### 4-1. pull 방식 결정 (Merge vs Rebase)

`git pull`이 거부된 이유는 Git이 봉합 방식을 모르기 때문. 두 가지 중 하나를 선택해야 한다.

| 방식 | 명령어 | 결과 |
|---|---|---|
| **Merge** (기본, 안전) | `git pull --no-rebase` | 병합 커밋을 추가로 만들어 Y자를 봉합 |
| **Rebase** (히스토리 깔끔) | `git pull --rebase` | 내 커밋을 팀원 커밋 뒤로 재배치해 일직선화 |

> **입문자/협업 시 권장: merge.** Rebase는 "이미 push된 내 커밋"에 대해 쓰면 위험하다(상세는 6장).

### 4-2. Merge 방식 진행 순서

```bash
git pull --no-rebase
```

실행하면 자동으로 병합이 시도되고, **nano 에디터**가 떠서 병합 커밋 메시지 입력을 요청한다.

```
Merge branch 'main' of https://github.com/.../...
# Please enter a commit message to explain why this merge is necessary,
...
```

`Merge branch 'main' of ...` 라는 기본 메시지가 이미 채워져 있으므로 **그대로 저장**하면 된다.

#### nano 단축키
| 키 | 동작 |
|---|---|
| `Ctrl + O` | 저장 (Write Out). 이후 파일명 확인 프롬프트가 뜨면 **Enter** |
| `Ctrl + X` | 종료 |

> 하단 메뉴의 `^` 표기 = `Ctrl` 키, `M-` 표기 = `Alt` 키

### 4-3. 병합 완료 후 push

```bash
git push
```

이제 갈라짐이 봉합되어 로컬이 원격보다 한 발 앞선 상태(병합 커밋 1개 추가)가 되므로 fast-forward로 깔끔하게 올라간다.

---

## 5. VS Code에서 발생한 알림창

`pull`을 GUI로 시도했을 때 뜨는 ⓧ 아이콘 알림창은 **Git 명령 실패 알림**이다. 실패 이유는 알림창에 표시되지 않는다.

| 버튼 | 용도 |
|---|---|
| **Show Command Output** | 실제 git 에러 메시지 확인 (디버깅 시 가장 유용) |
| Open Git Log | VS Code Git 확장의 상세 로그 |
| Cancel | 알림만 닫음 (문제 미해결) |

GUI는 에러 메시지를 작게 가리기 때문에, 문제가 생기면 **터미널에서 직접 명령을 실행해 전체 출력**을 보는 편이 디버깅이 쉽다.

---

## 6. Merge vs Rebase 개념 정리

### Merge — 실제 일어난 일을 그대로 기록

```
        ┌─ 내 커밋 ─────────┐
공통조상┤                    ├─ 병합 커밋 (new)
        └─ 팀원 커밋 ────────┘
```

- Y자 자체가 "두 사람이 병렬로 작업했음"을 영구히 보존
- 커밋 해시 변경 없음 → 안전

### Rebase — 일직선으로 재구성

```
공통조상 ── 팀원 커밋 ── 내 커밋(재배치)
```

- 마치 팀원 작업 위에 내가 작업한 것처럼 히스토리를 재작성
- **내 커밋의 부모가 바뀌므로 커밋 해시가 새로 생성됨** → "역사를 다시 쓰는" 행위

### 비교

| 항목 | Merge | Rebase |
|---|---|---|
| 히스토리 모양 | Y자(분기 보존) | 일직선 |
| 실제 작업 흐름 | 그대로 기록 ✅ | 재구성 (실제와 다름) |
| 커밋 해시 | 유지 ✅ | 변경됨 |
| 이미 push된 커밋에 사용 | 안전 ✅ | **위험** (팀원 히스토리와 어긋남) |
| 충돌 해결 | 한 번에 | 커밋마다 |
| 가독성 | 가지 많으면 복잡 | 항상 깔끔 |

### 결론
- **둘 다 존재하는 이유**: "실제 역사 보존(merge) vs 깔끔한 히스토리(rebase)"라는 가치 충돌
- **입문자/협업 시**: 우선 **merge** 위주로
- **rebase는 "내 로컬에만 있는 커밋"에만 사용**

---

## 7. 앞으로 예방하기

1. **작업 시작 전**에 `git pull` 한 번
2. **push 직전**에도 `git pull` 한 번
3. 매번 묻기 싫다면 기본 전략을 설정:
   ```bash
   git config --global pull.rebase false   # 항상 merge 방식
   ```
4. `git push --force` / `--force-with-lease`는 **팀 커밋을 날려버릴 수 있으므로 절대 사용 금지**(특별한 합의가 있는 경우 제외)

---

## 8. 한 줄 요약

> **이번 push 실패의 원인은 "나와 팀원이 모두 push 전에 커밋을 만들어서 원격과 로컬이 Y자로 갈라진 것"이며,**
> **해결책은 `git pull --no-rebase` → 병합 커밋 저장(`Ctrl+O` → Enter → `Ctrl+X`) → `git push` 순서로 봉합하는 것이다.**
