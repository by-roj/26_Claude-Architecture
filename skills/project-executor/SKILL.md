# Project Executor Skill

미리 정의된 프로젝트 기술서의 내용을 파악해 자동으로 실행하는 Claude code skill입니다.


## 개요

`project-executor` skill은 파라미터로 입력 받은 프로젝트를 자동으로 실행합니다.
파라미터가 없을 경우, `studyspace/`의 모든 프로젝트를 자동으로 발견하고 실행합니다.
CLAUDE.md 파일의 category에 따라 적절한 핸들러에게 자동으로 위임합니다.

---

## 작동 방식

### 실행 프로세스

```

프로젝트 발견

  ↓

CLAUDE.md 파싱

  ↓

category 확인

  ↓

적절한 핸들러 선택

  ↓

실행 및 보고

  ↓

상태 업데이트

```

### 단계별 상세 프로세스

#### 1단계: 프로젝트 발견
  
- 파라미터로 입력 받은 프로젝트 폴더 탐색
- 파라미터가 없을 경우, `/studyspace` 내 모든 폴더를 재귀적으로 스캔
	- 각 폴더의 CLAUDE.md 파일 존재 확인
	- 파일이 없으면 해당 폴더는 스킵


#### 2단계: CLAUDE.md 파싱

```yaml
---

status: pending

priority: high

category: reversing

created: YYYY-MM-DD

---
```


다음 항목을 추출합니다:
- `status`: pending, in_progress, completed, failed
- `priority`: high, medium, low
- `category`: reversing, soc, scripting, research, fuzzing, misc
- `created`: 프로젝트 생성 날짜

**status가 'in_progress'가 아니면 스킵됩니다.**


#### 3단계: Category 기반 위임 결정

프로젝트의 category에 따라 다음과 같이 처리됩니다 (앞쪽에 기재된 엔진을 우선 선택):

- **reversing** → security-researcher agent (Sonnet 4.6 ~ Opus 4.6)
- **soc** → security-researcher agent (Sonnet 4.6 ~ Opus 4.6)
- **scripting** → 직접 실행 (Haiku 4.5 ~ Sonnet 4.6)
- **research** → Claude 일반 능력 (Haiku 4.5 ~ Sonnet 4.6)
- **fuzzing** → security-researcher agent
- **misc** → 상황에 따라 (Haiku 4.5 ~ Opus 4.6)


#### 4단계: Instructions 실행

Instructions 섹션을 추출하여 실행합니다.

  
#### 5단계: 결과 보고

각 단계별 결과, 생성된 파일, 에러 정보를 보고합니다.

  

#### 6단계: 상태 업데이트

성공 시 `status: completed`, 실패 시 `status: failed`로 업데이트합니다.

---

## 우선순위 처리

실행 순서:

1. `priority: high`
2. `priority: medium`
3. `priority: low`

같은 우선순위 내에서는 `created` 날짜순입니다.

---

## 에러 처리

  
실패 시:
1. 에러 원인 분석
2. 권장 해결책 제시
3. CLAUDE.md 하단에 에러 기록
4. status → `failed` 업데이트

재시도하려면 status를 `in-progress`으로 변경 후 다시 실행합니다.

---

## 주의사항

- 네트워크 접근이 필요한 경우 CLAUDE.md에 명시 필수
- 결과는 항상 `output/` 디렉토리에 저장됨