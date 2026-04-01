# Claude Code Architecture & Integration Guide

---

## 1. 개요 (Executive Summary)

본 문서는 Claude Code 기반의 **3-컴포넌트 아키텍처**를 정의하며, 보안 연구(Reverse Engineering, Vulnerability Research), CTF 문제 해결, 자동화 스크립팅을 통합적으로 처리하기 위한 확장 가능한 실행 구조를 설명합니다.


### 핵심 목표

* 역할 분리(Separation of Concerns)를 통한 유지보수성 확보
* 자동화된 프로젝트 실행 및 상태 관리
* 도메인 특화 분석의 분리
* 확장 가능한 Agent 기반 구조 제공

---

## 2. 전체 아키텍처 구조

### 2.1 계층 구조

```text
User Input
    ↓
studyspace/CLAUDE.md (defines global rules)
    ↓
Claude Code / project-executor Skill (discovers & executes)
    ↓
Project Category Decision Tree
    ├─ reversing → security-researcher Agent
    ├─ soc (security context) → security-researcher Agent
    ├─ scripting → project-executor Skill (direct execution)
    ├─ research → Claude general capability or security-researcher
    └─ misc → project-executor or domain-specific agent
    ↓
Result Reporting & Status Update
    ↓
Output Files Generated & Status: pending → completed/failed
```

### 2.2 설계 원칙

* Hierarchical Control Flow
* Strict Responsibility Isolation
* Category-driven Execution Routing
* Non-overlapping Functional Scope

---

## 3. 컴포넌트 상세 분석

### 3.1 studyspace/CLAUDE.md (Root Directive Layer)

#### 역할

시스템 전체의 정책, 구조, 실행 규칙을 정의하는 **Global Configuration Layer**

#### 책임

* Directory hierarchy 정의
* Project-level CLAUDE.md format specification
* Execution methodology 정의 (Method A/B/C)
* Engine selection policy
  * **Haiku 4.5**: 문서 정리, 간단한 스크립팅, 번역/요약 (간단 작업)
  * **Sonnet 4.6**: 복잡한 추론, 중간 수준의 코드 작성, 데이터 분석
  * **Opus 4.6**: 심층 분석이 필요한 보안 연구 (역엔지니어링, 취약점 분석) - 사용자의 권한 허용 필요
* Domain-specific Engine Routing
  * category: reversing → **Sonnet 4.6** (복잡한 이진 분석 및 패턴 인식 필요)
  * category: soc → **Sonnet 4.6** (고도의 보안 컨텍스트 분석 필요)
  * category: scripting → **Haiku 4.5** (자동화 및 단순 로직)
  * category: research → **Haiku 4.6** (정보 수집 및 복합 분석)
* Output format 및 저장 규칙 정의
* Malware analysis safety guideline

#### 비책임

* Runtime execution
* Project discovery
* Binary analysis
* Status tracking

### 3.2 project-executor (Execution Orchestrator)

#### 역할

전체 시스템의 실행 흐름을 관리하는 **Orchestration Engine**

#### 핵심 기능

* Recursive filesystem traversal
* CLAUDE.md parsing
* Priority queue scheduling (high → medium → low)
* Code block execution
  * bash
  * python
  * powershell
* State machine 관리
  * pending → completed / failed
* Delegation routing (category 기반)
* Output artifact 생성
* Error handling 및 logging

#### 내부 동작 모델

```text
for project in discovered_projects:
    parse metadata
    route based on category
    execute instructions
    generate output
    update state
```

#### 비책임

* Domain-specific reasoning
* Reverse engineering
* Vulnerability analysis

### 3.3 security-researcher (Domain Expert Agent)

#### 역할

보안 분석을 담당하는 **Specialized Analytical Agent** (Opus 4.6 기반)

#### 책임

* Static / Dynamic Binary Analysis
* Vulnerability Research
* Exploit Development Insight
* Malware Behavioral Analysis
* CTF problem solving 및 writeup 생성
* IDA MCP integration 기반 자동 분석
* Reverse Engineering 자동화 파이프라인 관리

#### 엔진 선택 기준 (Engine Criteria)

**Opus 4.6 사용 조건 (사용자의 권한 허용 필요):**
* 바이너리 파일의 복잡한 패턴 분석 필요
* 역엔지니어링 결과의 고도의 추론 및 해석 필요
* 취약점 분석 및 exploit 가능성 판단 필요
* 멀티 레이어 보안 컨텍스트 분석 필요

#### 사용 도구 및 지원 방식

**1. IDA Pro (Reverse Engineering)**
* **경로**: `C:\Users\user\studyspace\settings\ida90\` (IDA Pro 9.0)
* **지원 방식**: IDA MCP Server를 통한 자동 분석
* **사용 시나리오**:
  * 바이너리 구조 분석
  * 함수 플로우 추출
  * 패턴 인식 및 자동 분류
  * 심볼 복원 및 분석
* **제약사항**: IDA MCP Server 설정 필수, 대형 바이너리는 분석 시간 고려 필요

**2. x64dbg / x32dbg (Dynamic Analysis)**
* **경로**: `C:\Users\user\studyspace\settings\x64dbg_x32dbg\`
* **지원 방식**: CLI 명령어 또는 수동 지침
* **사용 시나리오**:
  * Runtime behavior 추적
  * 메모리 상태 검사
  * 조건부 breakpoint 설정
  * 런타임 패치 및 테스트
* **제약사항**: Interactive debugging 필요시 사용자 참여 필요

**3. dnSpy (.NET Analysis)**
* **경로 (x64)**: `C:\Users\user\studyspace\settings\dnSpy-net-win64\`
* **경로 (x32)**: `C:\Users\user\studyspace\settings\dnSpy-net-win32\`
* **지원 방식**: CLI 분석 또는 수동 지침
* **사용 시나리오**:
  * .NET 어셈블리 역컴파일
  * 소스 코드 복원
  * 혼동 제거 (Deobfuscation)
  * 리소스 추출
* **제약사항**: .NET Framework/Core 대상에만 적용

**4. PE Analysis Tools**
* **ExeinfoPE**: `C:\Users\user\studyspace\settings\ExeinfoPE\`
  * PE 헤더 분석, 섹션 정보, Import/Export 테이블
* **PEview**: `C:\Users\user\studyspace\settings\PEview.exe`
  * 상세 PE 구조 분석
* **Scylla**: `C:\Users\user\studyspace\settings\Scylla_v0.9.8\`
  * IAT (Import Address Table) 복원, DLL injection 테스트
* **사용 시나리오**: 포킹(Packing) 감지, 섹션 분석, 바이너리 메타데이터 추출

**5. 부가 도구**
* **UPX**: `C:\Users\user\studyspace\settings\upx-5.1.1-win64\`
  * 포킹된 실행파일 언팩
* **지원 방식**: CLI 명령어

#### 데이터 흐름 (Tool Integration Flow)

```text
Binary Target
    ↓
ExeinfoPE / PEview (빠른 메타 분석)
    ↓
IDA MCP (구조적 분석)
    ↓
x64dbg / x32dbg (동적 검증)
    ↓
dnSpy (if .NET) / Scylla (IAT 복원)
    ↓
분석 결과 및 writeup 생성
```

#### 비책임

* Workflow orchestration
* Project lifecycle management
* General scripting
* Non-security 분석

---

## 4. 실행 흐름 (Execution Pipeline)

### 단계별 처리 과정

1. User Request Initiation
2. project-executor activation
3. Root CLAUDE.md parsing (engine selection policy 로드)
4. Target project discovery
5. Project CLAUDE.md parsing (category, priority 추출)
6. **Engine selection** based on category & context
7. Category-based routing decision
8. Handler execution (with engine-specific context)
9. Output generation
10. Status update
11. Reporting

### Engine Selection Logic (with Category Routing)

```text
Read category from project CLAUDE.md
    ↓
if category in [reversing, soc]:
    ├─ Engine: Opus 4.6 (deep analysis required)
    └─ Delegate → security-researcher with tools context
        └─ IDA MCP, x64dbg, dnSpy, PE tools 로드
    ↓
elif category == scripting:
    ├─ Engine: Haiku 4.5 (lightweight automation)
    └─ Execute directly via project-executor
    ↓
elif category == research:
    ├─ Engine: Sonnet 4.6 (complex reasoning)
    └─ Adaptive routing (general Claude or researcher)
    ↓
else (misc):
    ├─ Engine: Sonnet 4.6 (fallback reasoning)
    └─ Evaluate & route appropriately
```

### Domain-to-Engine Mapping Table

| Category | Engine | Reasoning | Handler |
|----------|--------|-----------|---------|
| reversing | Opus 4.6 | 복잡한 이진 분석, 패턴 인식 필요 | security-researcher |
| soc | Opus 4.6 | 고도의 보안 컨텍스트 분석 | security-researcher |
| scripting | Haiku 4.5 | 간단한 자동화, 반복 로직 | project-executor |
| research | Sonnet 4.6 | 정보 수집, 복합 추론 | general Claude |
| misc | Sonnet 4.6 | 문맥 판단 후 라우팅 | context-dependent |

---

## 5. Scope Boundary 정의

| 기능      | Root | Executor | Researcher | Engine Responsibility |
| ------- | ---- | -------- | ---------- | -------------------- |
| 구조 정의   | ✔    | ✘        | ✘          | N/A |
| Engine 선택 | ✔    | ✔        | ✘          | Root 정의, Executor 적용 |
| 실행      | ✘    | ✔        | ✘          | Executor (Haiku 4.5), Researcher (Opus 4.6) |
| 바이너리 분석 | ✘    | ✘        | ✔          | Opus 4.6 (high complexity) |
| 취약점 연구  | ✘    | ✘        | ✔          | Opus 4.6 (deep analysis) |
| 상태 관리   | ✘    | ✔        | ✘          | N/A |
| CTF/WriteUp | ✘    | ✘        | ✔          | Opus 4.6 (complex reasoning) |
| 도구 경로 정의 | ✔    | ✘        | ✘          | Root에서 도구 경로 관리 |
| 도구 사용   | ✘    | ✘        | ✔          | Researcher가 Root 정의 경로 사용 |

### 핵심 원칙

* 기능 중복 금지
* 책임 영역 명확화
* Layer 간 침범 금지
* **Engine Responsibility Clarity**: 각 태스크는 명확한 Engine 할당 필요

---

## 6. Integration Rules

### Rule 1: Hierarchical Governance

* 상위 레이어가 하위 레이어를 규제
* 하위 레이어는 상위 정책 override 불가
* Root에서 정의한 Engine Selection Policy는 모든 실행에서 준수

### Rule 2: Category-driven Delegation with Engine Assignment

* **reversing** → security-researcher (Opus 4.6)
  * 도구: IDA MCP, x64dbg, dnSpy, PE analysis tools
* **soc** → security-researcher (Opus 4.6)
  * 도구: IDA MCP, 바이너리 분석 toolchain
* **scripting** → project-executor 직접 실행 (Haiku 4.5)
  * 도구 제약 없음
* **research** → adaptive routing (Sonnet 4.6)
  * 대상에 따라 general Claude 또는 researcher 선택
* **misc** → context-aware routing (Sonnet 4.6)
  * 분석 후 적절한 핸들러 선택

### Rule 3: Engine Compliance

* Root CLAUDE.md에서 정의한 Engine selection policy 필수 준수
* reversing/soc 카테고리는 **반드시 Opus 4.6** 사용
* scripting 카테고리는 **반드시 Haiku 4.5** 사용
* 예외 상황은 Root CLAUDE.md 수정을 통해서만 변경 가능

### Rule 4: Error Handling Model

* executor: execution-level error 처리 (상태 관리)
* researcher: domain-specific error 처리 (분석 실패)
* root: policy-level error (engine selection 오류)

### Rule 5: Extensibility

* 신규 Agent는 명확한 domain scope 필요
* 신규 Agent는 명시적인 engine requirement 정의 필수
* 기존 컴포넌트와 역할 중복 금지
* 모든 신규 카테고리는 Root CLAUDE.md에 engine 매핑 추가 필수

---

## 7. Implementation Requirements

### 필수 구성 요소

* 모든 프로젝트에 CLAUDE.md 존재
* category / priority / status 필드 정의
* output directory 구조 존재
* toolchain path 정확성 확보
* IDA MCP server 구성
* Root CLAUDE.md의 engine selection policy 준수

### 실행 전 체크리스트

```text
[ ] CLAUDE.md format valid (metadata 포함)
[ ] category set (reversing/soc/scripting/research/misc)
[ ] status = pending
[ ] output directory exists
[ ] tools accessible (category별 도구 경로 확인)
[ ] Engine selection verified (category → engine mapping 확인)
    [ ] reversing: Opus 4.6 설정
    [ ] soc: Opus 4.6 설정
    [ ] scripting: Haiku 4.5 설정
    [ ] research: Sonnet 4.6 설정
[ ] IDA MCP server configured (reversing/soc 카테고리)
[ ] Tool paths match settings directory
```

### 카테고리별 도구 준비 확인

| Category | 필수 확인 항목 | 도구 경로 |
|----------|--------------|---------|
| reversing | IDA MCP, x64dbg 경로 | `settings/ida90/`, `settings/x64dbg_x32dbg/` |
| soc | Binary analysis toolchain | `settings/ida90/`, `settings/ExeinfoPE/` |
| scripting | N/A (일반 스크립팅) | N/A |
| research | 정보 수집 도구 | 일반 인터넷 접근 |
| misc | context-dependent | case-by-case |

---

## 8. 확장성 및 발전 방향

* Notion MCP integration
* Automated IDA analysis pipeline
* Multi-agent architecture
* Parallel execution engine
* Real-time monitoring dashboard
* ML-based binary pattern recognition

---

## 9. 결론 (Technical Insight)

본 아키텍처는 Claude Code를 단순한 AI 도구가 아닌 **엔진-기반 자동화된 보안 연구 플랫폼**으로 확장하기 위한 설계이다.

### 핵심 설계 원칙

```text
Policy Layer (Root CLAUDE.md)
    ├─ Engine Selection Policy 정의
    ├─ Category → Engine Mapping 정의
    └─ Tool Path & Safety Guideline 정의
        ↓
Execution Layer (project-executor)
    ├─ Category 기반 라우팅
    ├─ Engine 할당 및 적용
    └─ State Management
        ↓
Analysis Layer (security-researcher Agent)
    ├─ Opus 4.6 기반 심층 분석
    ├─ Multi-tool Integration (IDA MCP, x64dbg, dnSpy)
    └─ Domain-specific Expertise
```

### 구조적 강점

* **Engine-driven Architecture**: 각 작업은 명확한 엔진 할당
* 높은 확장성 (신규 카테고리 추가 시 engine 매핑만 정의)
* 명확한 책임 분리 (policy ≠ execution ≠ analysis)
* 자동화된 실행 파이프라인 with error handling
* 보안 분석에 특화된 Agent 구조 + tool integration
* 도구 경로 중앙화 (Root에서 정의, Agent가 사용)

### 실전 적용 가치

* **Reverse Engineering workflow 자동화**: Opus 4.6 + IDA MCP + tool chain
* **CTF 문제 batch 처리**: security-researcher agent 자동 dispatch
* **Fuzzing 및 취약점 연구 pipeline**: 계층화된 분석 프레임워크
* **보안 리포트 자동화**: category별 template 및 engine 기반 depth 조절

### Engine Selection이 핵심

본 아키텍처는 **"태스크 복잡도와 도메인을 기반으로 한 엔진 선택"** 을 중심으로 설계되었다:
- Simple tasks (scripting) → **Haiku 4.5** (빠른 실행, 저비용)
- Complex reasoning (research) → **Sonnet 4.6** (균형잡힌 성능)
- Deep analysis (reversing/soc) → **Opus 4.6** (최고 품질, 복잡 분석)

---

## Final Statement

이 아키텍처는 **"Engine-driven Claude Code 기반 Security Research Automation Framework"** 로 정의할 수 있으며,
향후 다양한 보안 분석 및 자동화 시나리오에 적용 가능한 기반 구조를 제공한다.

**핵심 메시지**: 올바른 엔진 선택은 작업 품질, 비용, 실행 속도의 최적화를 결정한다.
