# studyspace — Claude Code 운영 지침

이 디렉토리는 **Obsidian + Claude Code** 통합 작업 공간입니다.
이 문서는 `studyspace/`의 전역 규칙과 정책을 정의합니다.

---

## 개요

`studyspace/`는 보안 연구, CTF 문제 풀이, 자동화 스크립팅 등을 통합적으로 관리하는 디렉토리입니다.
이 문서는 `studyspace/`의 전역 규칙과 정책을 정의합니다.

---

## 디렉토리 구조

```
studyspace/
├── Projects/                   # 프로젝트 단위 디렉토리
│   ├── [project_name]/         # 개별 프로젝트
│   │   ├── CLAUDE.md           # 작업 정의
│   │   ├── output/             # 실행 결과 저장
│   │   └── [작업 파일들]
├── .claude/                    # Claude 설정 파일
│   ├── skills/
│   │   └── project-executor/
│   └── agents/
│       └── security-researcher.md
├── CLAUDE.md                   # 이 파일 (전역 규칙)
```

---

## CLAUDE.md 형식

각 프로젝트 디렉토리의 `CLAUDE.md`는 다음 형식을 따릅니다.

### Frontmatter (필수)

```yaml
---
status: pending             # pending / in_progress / completed / failed
priority: medium            # high / medium / low
category: reversing         # reversing / soc / scripting / research / fuzzing / misc
created: YYYY-MM-DD
tags: []
---
```

### 구조

```markdown
# [작업 제목]

## Goal
달성하고자 하는 목표를 명확히 기술합니다.

## Context
작업에 필요한 배경 정보, 대상 파일 경로, 참고 사항 등을 작성합니다.

## Instructions
Claude Code / Agent가 수행해야 할 구체적인 지시 사항을 작성합니다.

- 단계 1: [지시 사항]
- 단계 2: [지시 사항]

## Output
결과물 저장 위치 및 형식을 지정합니다.
예: output/result.md, output/analysis.txt
```

### 카테고리 설명

| 카테고리          | 설명                 | 담당                    | 엔진                     |
| ------------- | ------------------ | --------------------- | ---------------------- |
| **reversing** | CTF 문제 풀이, 바이너리 분석 | security-researcher   | Sonnet 4.6 ~ Opus 4.6  |
| **soc**       | 보안 인시던트 분석, SOP 개발 | security-researcher   | Sonnet 4.6 ~ Opus 4.6  |
| **scripting** | 자동화 스크립트, 도구 개발    | project-executor (직접) | Haiku 4.5 ~ Sonnet 4.6 |
| **research**  | 정보 수집, 분석, 문서 작성   | Claude (일반)           | Haiku 4.5 ~ Sonnet 4.6 |
| **fuzzing**   | 퍼징 코드 작성, 취약점 연구   | security-researcher   | Sonnet 4.6 ~ Opus 4.6  |
| **misc**      | 기타 작업              | 상황에 따라                | Haiku 4.5 ~ Opus 4.6   |

---

## 엔진 사용 가이드

### 개요

각 작업은 복잡도에 따라 적절한 Claude 엔진이 할당됩니다.
프로젝트 정의 시 다음 기준을 참고하세요.

### 엔진별 특징

#### Haiku 4.5 (간단한 작업)

**사용 시점**: scripting, research 카테고리일 때 우선 선택 
- 도구 사용법 설명
- 간단한 스크립트 작성 및 실행
- 문서 정리, 번역, 요약
- 정보 수집

#### Sonnet 4.6 (중간 복잡도)

**사용 시점**: reversing, soc, fuzzing 카테고리일 때 우선 선택, scripting, research 카테고리일 때 사용자의 허용 후에 전환 가능
- 정보 수집 및 분석
- 분석 보고서 작성
- 여러 도구의 결과 종합
- 기본 코드 분석

#### Opus 4.6 (고복잡도 분석)

**사용 시점**: reversing, soc, fuzzing 카테고리일 때 사용자의 허용 후에 전환 가능
- 깊이 있는 바이너리 분석
- 복잡한 역분석
- 취약점 분석 및 평가
- CTF 문제 최종 해답 도출


### 자동 할당 규칙

```
reversing / soc / fuzzing     → Sonnet 4.6 (사용자 허용 시 Opus 4.6 전환)
scripting / research          → Haiku 4.5 (사용자 허용 시 Sonnet 4.6 전환)
misc                          → 상황에 따라 (Haiku 4.5 ~ Opus 4.6)
```

**당신이 해야 할 것**: CLAUDE.md에 category를 올바르게 설정하기만 하면, 엔진은 자동으로 할당됩니다.
