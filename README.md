# 사내 IT Incident 관리 자동화 시스템

OutSystems ODC 기반으로 만든 AI 연동 IT 장애(Incident) 관리 시스템. 문의 접수부터 분류, 배정, 해결, 기간별 분석 보고서 생성까지 전 과정을 표준화·자동화한다.

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [주요 화면](#주요-화면)
- [담당 역할](#담당-역할)
- [시스템 아키텍처](#시스템-아키텍처)
- [데이터 모델](#데이터-모델)
- [AI 에이전트 설계](#ai-에이전트-설계)
- [프로젝트 특징](#프로젝트-특징)
- [산출물 문서](#산출물-문서)

**정상 흐름**
![에러 등록](screenshot/incident_create.gif)


## 프로젝트 개요

**팀 "길과 함께"** (김수길, 박병현, 장재혁, 정다혜) — OutSystems ODC + Gemini API 기반 팀 프로젝트.

기업 내 하드웨어·소프트웨어 장애를 하나의 ITSM 프로세스로 통합하고, 등록부터 분류·배정·처리·해결·종료까지의 업무 흐름을 표준화하는 것이 목표. 역할은 요청자 / 서비스데스크 / 전문 담당자 / 관리자 4가지로 구분되며 화면·메뉴 접근 권한이 역할별로 분리되어 있다.

**처리 흐름**

1. 요청자가 오류 사례를 자유 문장으로 등록
2. Incident Triage Agent가 Domain·Category·Symptom, 담당 지원팀, Impact/Urgency를 자동 추천
3. 서비스데스크가 추천 결과를 검토 후 승인/수정 (HITL) → 담당 지원팀·담당자 확정
4. 담당자가 처리 시작, 필요 시 Resolution Recommender로 유사 해결 사례 최대 3건 확인
5. 처리 완료 후 원인·조치·재발방지 내용 등록 → Resolved
6. 요청자가 최종 확인: 해결 완료 시 Closed, 미해결 시 Reopened로 전환되어 서비스데스크가 재검토
7. 지정 기간의 Incident들을 Incident Analysis PDF Agent가 종합 분석해 PDF 보고서 생성

```mermaid
flowchart TD
    A[요청자 등록] --> B[Triage Agent 분류]
    B --> C[서비스데스크 검토·배정 HITL]
    C --> D[담당자 처리]
    D --> E{요청자 최종 확인}
    E -->|해결 완료| F[Closed]
    E -->|미해결| G[Reopened] --> C
    D --> H[Incident Analysis PDF Agent]
```

**범위 밖**: 이메일·메신저·전화 자동 접수, 서버·네트워크 모니터링 연동, CMDB 등 복잡한 ITSM 기능, AI의 자동 해결/종료 — 모든 AI 결과는 추천이며 실제 반영은 권한을 가진 사용자가 검토·승인한다.

## 주요 화면

로그인 후 역할에 따라 다른 메인 화면과 메뉴로 연결된다.

![로그인 화면](screenshot/login.png)
*역할별 계정으로 로그인하면 각자의 메인 화면으로 이동한다.*

### 요청자

![요청자 메인 화면](screenshot/requester_main.png)
*접수·진행중·완료·반려 현황 카드와 공지사항·FAQ, 장애 신고 버튼을 제공하는 메인 화면.*

![장애 신고 화면](screenshot/requester_form.png)
*제목과 상세내용만 입력하면 되는 신고 폼. 카테고리를 직접 고를 필요 없이 AI가 자동으로 분류한다.*

![내 요청 내역](screenshot/requester_list.png)
*본인이 등록한 Incident 목록과 처리 상태를 조회하는 화면.*

![마이페이지](screenshot/requester_mypage.png)
*개인 정보를 확인하는 화면.*

### 서비스데스크

![서비스데스크 메인 화면](screenshot/servicedesk_main.png)
*신규·진행중·완료·이관요청·반려 현황과 Category별 장애 현황을 보여주는 메인 화면.*

![Incident Queue](screenshot/servicedesk_list.png)
*접수된 전체 Incident를 상태별로 조회하는 화면.*

![Incident 상세](screenshot/servicedesk_detail.png)
*신고 내용, 실제 조치 내용, AI 추천값과 최종 확정값 비교, 전체 처리 이력을 한 화면에서 확인한다.*

### 전문 담당자

![담당자 메인 화면](screenshot/specialist_main.png)
*접수·진행중·완료·재배정·이관요청 현황을 보여주는 메인 화면.*

![내 담당 요청 내역](screenshot/specialist_list.png)
*배정된 Incident 목록과 기간별 분석 보고서 생성 기능을 제공하는 화면.*

![담당 Incident 상세](screenshot/specialist_detail.png)
*신고 내용과 함께 Resolution Recommender의 추천 해결 방법을 표시하는 화면.*

### Incident Analysis PDF Agent

![보고서 생성 기간 선택](screenshot/pdf_period_select.png)
*담당자가 최근 1/3/6개월 또는 기간 직접 입력 중 선택해서 보고서 생성을 요청하는 화면.*

![분석 보고서 표지](screenshot/pdf_report_cover.png)
*생성된 PDF 보고서 표지 — 분석 대상 Incident 건수, 분석 기간, 생성일을 요약해서 보여준다.*

![분석 보고서 목차](screenshot/pdf_report_toc.png)
*통계 요약부터 AI 추천 활용 정보, 재발방지 권고, 최종 결론까지 10개 섹션으로 구성된 보고서 목차.*

### 관리자

![관리자 메인 화면](screenshot/admin_main.png)
*등록 임직원 수, 오늘 접수, 증상 코드, 오류 로그, 지원팀별 진행 현황을 종합해서 보여주는 화면.*

![임직원 관리](screenshot/admin_employee.png)
*사용자 등록·수정, 권한 및 부서 설정 화면.*

![인시던트 증상 관리](screenshot/admin_symptom.png)
*Symptom별 기본 지원팀, Impact/Urgency 매핑을 관리하는 화면.*

![로그 기록 관리](screenshot/admin_log.png)
*Error Log를 조회하는 화면.*

![AI 응답 JSON 확인](screenshot/admin_log_json.png)
*Incident Analysis PDF Agent가 생성한 응답 원본 JSON을 그대로 확인할 수 있는 화면.*

## 담당 역할

- Incident Analysis PDF Agent 설계 (프롬프트 구조 · 출력 스키마)
- 회원가입 및 권한(Role) 부여 기능
- 로그 관리 화면 — 에러 로그 · AI 응답(JSON) 조회 기능
- 실시간 알림 기능 구현

> 위 항목이 본인이 직접 구현한 부분이며, 앱 전체는 팀 공동 산출물이다.

## 시스템 아키텍처

```mermaid
graph TD
    A[2AA_ITSM_APP<br/>화면 UI] --> B[2AA_ITSM_CORE<br/>Entity · Server Action]
    B <--> C[2AA_ITSM_AGENT_TRIAGE]
    B <--> D[2AA_ITSM_AGENT_RECOMMANDER]
    B <--> E[2AA_ITSM_AGENT_REPORT_MAKER]
```

실제 Service Studio 구조에서도 화면 UI(APP)와 로직·데이터(CORE)가 모듈로 분리되어 있다.

![Assets 목록 — 실제 모듈명](screenshot/architecture_modules_list.png)
*Service Studio Assets에서 확인한 실제 모듈 5개 — 2AA_ITSM_APP, 2AA_ITSM_CORE, 2AA_ITSM_AGENT_TRIAGE, 2AA_ITSM_AGENT_RECOMMANDER, 2AA_ITSM_AGENT_REPORT_MAKER.*

![2AA_ITSM_APP — Interface 탭](screenshot/architecture_app_interface.png)
*APP 모듈은 화면(UI Flows)만 담당 — Login, Signup 등 화면 트리.*

![2AA_ITSM_CORE — Logic 탭](screenshot/architecture_core_logic.png)
*CORE 모듈의 Server Actions — Authentication, Triage, Recommender, PDF 등 로직을 담당.*

![2AA_ITSM_CORE — Data 탭](screenshot/architecture_core_data.png)
*CORE 모듈의 Entities — Incident, Employee, IncidentStatusHistory 등 데이터를 담당.*

## 데이터 모델

전체 19개 엔티티 중 Incident를 중심으로 한 핵심 구조만 발췌. 전체 모델은 `/docs` 데이터모델정의서 참고.

```mermaid
erDiagram
    DEPARTMENT ||--o{ EMPLOYEE : "소속"
    DEPARTMENT ||--o{ INCIDENT_SYMPTOM : "기본 담당부서"
    EMPLOYEE ||--o{ INCIDENT : "요청 (Requester)"
    EMPLOYEE ||--o{ INCIDENT : "배정 (Assignee)"
    EMPLOYEE ||--o{ INCIDENT : "전문담당 (Specialist)"
    INCIDENT_SYMPTOM ||--o{ INCIDENT : "장애유형"
    INCIDENT ||--o{ INCIDENT_STATUS_HISTORY : "상태변경 이력"
    INCIDENT ||--|| INCIDENT_TRIAGE_SUGGESTION : "AI 최초 분류 원본"
    INCIDENT ||--|| INCIDENT_RECOMMENDATION_RESULT : "AI 추천 원본"
    INCIDENT ||--o| INCIDENT_KNOWLEDGE : "해결 내용"

    DEPARTMENT {
        Identifier Id PK
        Text Code
        Text Name
        Boolean IsSupportTeam
    }
    EMPLOYEE {
        Identifier Id PK
        Integer EmployeeNo
        Text Name
        DepartmentIdentifier DepartmentId FK
        JobRankIdentifier JobRankId FK
        SupportLevelIdentifier SupportLevelId FK
    }
    INCIDENT {
        Identifier Id PK
        Text IncidentNo
        Text Title
        Text Content
        ImpactLevelIdentifier ImpactLevelId FK
        UrgencyLevelIdentifier UrgencyLevelId FK
        PriorityIdentifier PriorityId FK
        IncidentSymptomIdentifier IncidentSymptomId FK
        EmployeeIdentifier AssigneeId FK
        EmployeeIdentifier SpecialistId FK
        EmployeeIdentifier RequesterId FK
        DateTime RegDate
    }
    INCIDENT_SYMPTOM {
        Identifier Id PK
        Text Name
        ImpactLevelIdentifier DefaultImpactLevelId FK
        UrgencyLevelIdentifier DefaultUrgencyLevelId FK
        DepartmentIdentifier DefaultDepartmentId FK
    }
    INCIDENT_STATUS_HISTORY {
        Identifier Id PK
        IncidentStatusIdentifier IncidentStatusId FK
        EmployeeIdentifier RegId FK
        DateTime RegDate
        Text Memo
    }
    INCIDENT_TRIAGE_SUGGESTION {
        Identifier Id PK
        IncidentSymptomIdentifier IncidentSymptomId FK
        ImpactLevelIdentifier ImpactLevelId FK
        DepartmentIdentifier DepartmentId FK
        EmployeeIdentifier EmployeeId FK
        Text Reason
    }
    INCIDENT_RECOMMENDATION_RESULT {
        Identifier Id PK
        IncidentSymptomIdentifier IncidentSymptomId FK
        Text ResultStatus
        Text ResultJson
    }
    INCIDENT_KNOWLEDGE {
        Identifier Id PK
        Text ResolutionContent
        Boolean IsAIRecommendationApplied
    }
```

IncidentStatusHistory·IncidentTriageSuggestion처럼 변경 이력과 AI 최초 판단을 원본 그대로 보존하는 구조가, 앞서 말한 감사 추적성·피드백 기반 판단 보정의 실제 근거가 되는 부분이다.

## AI 에이전트 설계

- **Incident Triage Agent**: 사용자 원문과 분류체계를 기반으로 Domain·Category·Symptom, 담당 지원팀, Impact/Urgency를 추천. 서비스데스크가 분류를 수정하면 그 사유가 이력으로 남아 Triage Agent가 이후 판단 시 Context로 참고한다
- **Resolution Recommender Agent**: 과거 종료 Incident·Known Error 기반으로 해결 절차를 추천 사유·출처와 함께 최대 3건 제공
- **Incident Analysis PDF Agent**: 지정 기간 동안의 Incident를 종합해 통계·원인·해결·재발방지 권고가 담긴 비즈니스 문서 형식 PDF 보고서 생성
- 공통 원칙: 모든 AI 결과는 추천으로만 제공되며, 최종 반영은 권한을 가진 사용자가 검토·승인하는 HITL 구조

## 프로젝트 특징

- **HITL 원칙 전 구간 적용**: Triage 승인, 해결 최종 확인, 리포트 활용까지 AI는 추천만 하고 사람이 결정 — AI의 자동 해결·종료는 설계 단계에서부터 범위 밖으로 명시
- **피드백 기반 판단 보정**: 서비스데스크가 AI 추천을 수정하면 수정 결과와 사유가 이력으로 남고, Triage Agent가 이후 판단 시 관련 이력을 Context로 참고 — 운영 경험이 쌓일수록 반복되는 오분류를 줄이는 구조
- **불확실성을 인정하는 설계**: 판단 근거가 부족하면 억지로 결과를 만들지 않고 추천 불가를 반환하거나 서비스데스크 재검토로 넘김 — 근거 없는 확정을 막는 안전장치
- **감사 추적성 확보**: AI 입력·출력, 사람의 검토 결과, 데이터 변경 이력을 전부 감사 로그로 남겨 엔터프라이즈 감사 요구를 고려

## 산출물 문서

`/docs` 폴더 참고 (기획서, 요구사항정의서, 기능정의서, 데이터모델정의서, 화면정의서, 테스트시나리오 — 팀 공동 산출물)
