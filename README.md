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
    ├─ scripting   →  project-executor (직접 실행)
    ├─ research    →  Claude 일반
    ├─ fuzzing     →  security-researcher Agent
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

```text
Policy Layer  (Root CLAUDE.md)
    ├─ Engine Selection Policy 정의
    ├─ Category → Engine Mapping 정의
    └─ Tool Path & Safety Guideline 정의
        ↓
Execution Layer  (project-executor Skill)
    ├─ Category 기반 라우팅
    ├─ Engine 할당 및 적용
    └─ State Management
        ↓
Analysis Layer  (security-researcher Agent)
    ├─ Opus 4.6 기반 심층 분석
    ├─ Multi-tool Integration
    └─ Domain-specific Expertise
```

---

## 3. 컴포넌트 상세 분석

### 3.1 studyspace/CLAUDE.md (Root Directive Layer)

#### 역할

시스템 전체의 정책, 구조, 실행 규칙을 정의하는 **Global Configuration Layer**

* Directory hierarchy 정의
* Project-level CLAUDE.md format specification
* Engine selection policy
* Domain-specific Engine Routing (category-based)
* Output format 및 저장 규칙 정의
* Malware analysis safety guideline

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
  * in-progress → completed / failed
* Delegation routing (category-based)
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

### 3.3 security-researcher (Domain Expert Agent)

#### 역할

보안 분석을 담당하는 **Specialized Analytical Agent**

#### 책임

* Static / Dynamic Binary Analysis
* Vulnerability Research
* Exploit Development Insight
* Malware Behavioral Analysis
* CTF problem solving 및 writeup 생성
* IDA MCP integration 기반 자동 분석
* Reverse Engineering 자동화 파이프라인 관리

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

---

## 5. Scope Boundary 정의

| 기능      | Root | Executor | Researcher | Engine Responsibility |
| ------- | ---- | -------- | ---------- | -------------------- |
| 구조 정의   | ✔    | ✘        | ✘          | N/A |
| Engine 선택 | ✔    | ✔        | ✘          | Root 정의, Executor 적용 |
| 실행      | ✘    | ✔        | ✘          | Executor, Researcher |
| 바이너리 분석 | ✘    | ✘        | ✔          | Sonnet 4.6 ~ Opus 4.6 (high complexity) |
| 취약점 연구  | ✘    | ✘        | ✔          | Sonnet 4.6 ~ Opus 4.6 (deep analysis) |
| 상태 관리   | ✘    | ✔        | ✘          | N/A |
| CTF/WriteUp | ✘    | ✘        | ✔          | Sonnet 4.6 ~ Opus 4.6 (complex reasoning) |
| 도구 경로 정의 | ✔    | ✘        | ✘          | Root에서 도구 경로 관리 |
| 도구 사용   | ✘    | ✘        | ✔          | Researcher가 Root 정의 경로 사용 |

### 핵심 원칙

* 기능 중복 금지
* 책임 영역 명확화
* Layer 간 침범 금지
* **Engine Responsibility Clarity**: 각 태스크는 명확한 Engine 할당 필요
