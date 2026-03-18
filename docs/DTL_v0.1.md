# PMO Portal DTL v1.0

## 문서 목적
본 문서는 PRD를 구현 가능한 작업 단위로 분해한 DTL이다.  
각 항목은 아래 5개 축으로 정리한다.

- 목표
- 작업내용
- 산출물
- DoD
- 의도 전달용 코드 스니펫

---

# 1. 프로젝트 골격 구축

## 목표
Tauri + React + TypeScript 기반의 기본 실행 환경과 폴더 구조를 세팅하고, 이후 기능 개발이 흔들리지 않도록 공통 구조를 먼저 고정한다.

## 작업내용
- `pmo-doc-utility` 프로젝트 초기화
- Tauri + React + TypeScript + Tailwind 세팅
- 라우팅 구조 구성
- 공통 레이아웃 컴포넌트 구성
- 기능 중심 폴더 구조 생성
- 기본 상태 관리 구조 생성
- 리소스 디렉터리 생성
- SQLite 초기 연결 준비
- 개발/빌드/배포 기본 스크립트 정리

## 산출물
- 실행 가능한 초기 앱
- 기본 폴더 구조
- Tailwind 설정
- Tauri 설정 파일
- 공통 레이아웃 초안
- 기본 라우팅 파일
- DB 초기화 코드 초안

## DoD
- 앱이 로컬에서 실행된다
- 홈 화면 더미가 열린다
- Tauri command 호출이 가능하다
- Tailwind 스타일 적용이 확인된다
- 폴더 구조가 PRD 합의안과 일치한다
- 개발 서버와 데스크톱 실행 흐름이 문서화되어 있다

## 의도 전달용 코드 스니펫
```ts
// src/app/router/index.tsx
import { createBrowserRouter } from "react-router-dom";
import HomePage from "@/pages/Home";
import SettingsPage from "@/pages/Settings";
import HistoryPage from "@/pages/History";

export const router = createBrowserRouter([
  { path: "/", element: <HomePage /> },
  { path: "/settings", element: <SettingsPage /> },
  { path: "/history", element: <HistoryPage /> },
]);
```

```rust
// src-tauri/src/main.rs
fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![
            commands::ppt_to_pdf::convert_pptx_to_pdf,
            commands::pdf_merge::merge_pdfs,
            commands::pdf_compress::compress_pdf,
            commands::proposal_pack::build_proposal_package
        ])
        .run(tauri::generate_context!())
        .expect("failed to run app");
}
```

---

# 2. 공통 레이아웃 및 홈 화면 프레임

## 목표
상단 즐겨찾기, 중단 문서 도구, 하단 업무 매뉴얼의 3분할 홈 화면을 구현한다.  
이 화면은 항상 켜두는 사이드형 업무 런처의 기준 프레임이 된다.

## 작업내용
- 앱 쉘 레이아웃 구현
- 상단 헤더 구현
- 3분할 홈 화면 영역 구현
- 최대 폭 / 패널형 레이아웃 반영
- 스크롤 최소화 방향으로 높이 정책 설계
- 홈 카드 영역 배치
- 상세 기능 진입용 패널/모달 컨테이너 준비

## 산출물
- 홈 화면 레이아웃 컴포넌트
- AppShell
- Header
- FavoritesBar
- ToolGrid
- ManualSection

## DoD
- 홈 화면에서 상/중/하 3분할이 명확하다
- 기본 크기에서 주요 요소가 한 화면에 보인다
- 즐겨찾기 영역이 시각적으로 가장 먼저 인식된다
- 중단 도구 영역이 메인 액션 존으로 보인다
- 하단 매뉴얼 영역이 보조 정보 영역으로 보인다
- 상세 기능을 열 수 있는 공통 패널 구조가 준비되어 있다

## 의도 전달용 코드 스니펫
```tsx
// src/pages/Home/index.tsx
export default function HomePage() {
  return (
    <main className="h-screen max-w-[520px] min-w-[420px] mx-auto p-3">
      <div className="grid h-full grid-rows-[1.2fr_2.4fr_1.2fr] gap-3">
        <FavoritesBar />
        <ToolGrid />
        <ManualSection />
      </div>
    </main>
  );
}
```

---

# 3. 디자인 시스템 및 테마

## 목표
Notion 계열의 정돈된 느낌을 기반으로, 과한 효과 없이 차분하고 일관된 UI를 구성한다.  
MVP 범위로 라이트/다크 모드를 포함한다. :contentReference[oaicite:3]{index=3}

## 작업내용
- 컬러 토큰 정의
- 라이트/다크 공통 역할 정의
- 카드 / 버튼 / 입력창 / 패널 공통 스타일 정의
- 헤더, 섹션, 상태 메시지 컴포넌트 스타일 정의
- 테마 저장 및 복원 로직 구현
- 테마 토글 UI 구현

## 산출물
- theme token 파일
- 공통 UI 컴포넌트 스타일
- ThemeProvider
- ThemeToggle

## DoD
- 라이트/다크 모드 전환이 즉시 반영된다
- 앱 재실행 후 이전 테마가 유지된다
- 카드/버튼/텍스트 스타일이 일관적이다
- 과한 그림자/그라데이션/애니메이션이 없다

## 의도 전달용 코드 스니펫
```ts
// src/lib/theme/tokens.ts
export const themeTokens = {
  light: {
    bg: "bg-zinc-50",
    surface: "bg-white",
    text: "text-zinc-900",
    muted: "text-zinc-500",
    border: "border-zinc-200",
  },
  dark: {
    bg: "bg-zinc-950",
    surface: "bg-zinc-900",
    text: "text-zinc-100",
    muted: "text-zinc-400",
    border: "border-zinc-800",
  },
};
```

```tsx
// src/features/theme/components/ThemeToggle.tsx
export function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  return (
    <button onClick={toggleTheme} aria-label="테마 전환">
      {theme === "dark" ? "🌙" : "☀️"}
    </button>
  );
}
```

---

# 4. 즐겨찾기 기능

## 목표
앱의 시작점 역할을 하는 핵심 기능으로, 자주 사용하는 업무 사이트를 빠르게 여는 툴바형 즐겨찾기 영역을 구현한다.

## 작업내용
- 기본 즐겨찾기 데이터 JSON 정의
- 상단 툴바 UI 구현
- Main / Sub 그룹 구조 반영
- 기본 브라우저 열기 기능 연결
- 아이콘 리소스 매핑
- 링크 열기 실패 시 간단 오류 처리

## 산출물
- `resources/favorites/default-favorites.json`
- FavoritesBar 컴포넌트
- FavoriteItem 컴포넌트
- 외부 링크 열기 서비스

## DoD
- 상단 영역에서 즐겨찾기 링크를 클릭할 수 있다
- 클릭 시 기본 브라우저가 열린다
- Main 링크와 Sub 링크의 시각 구분이 있다
- JSON 수정만으로 링크 목록을 바꿀 수 있다
- 링크 실패 시 사용자에게 간단한 실패 메시지를 보여준다

## 의도 전달용 코드 스니펫
```json
[
  {
    "id": "groupware",
    "name": "그룹웨어",
    "url": "https://example.com",
    "category": "main",
    "order": 1,
    "enabled": true
  }
]
```

```tsx
// src/features/favorites/services/openFavorite.ts
import { open } from "@tauri-apps/plugin-shell";

export async function openFavorite(url: string) {
  await open(url);
}
```

---

# 5. PPTX → PDF 변환

## 목표
PowerPoint 설치 환경을 활용해 `.pptx` 파일을 고품질 PDF로 변환한다.  
앱의 핵심 문서 처리 기능이다. :contentReference[oaicite:4]{index=4}

## 작업내용
- 파일 선택 UI 구현
- 드래그 앤 드롭 지원
- 저장 경로 선택
- Rust command 작성
- PowerPoint 연동 서비스 작성
- 변환 상태 표시
- 성공/실패 메시지 처리
- 결과 폴더 열기 버튼 연결
- 폰트 관련 안내 문구 반영

## 산출물
- PptToPdf 패널/화면
- `src-tauri/src/commands/ppt_to_pdf.rs`
- `src-tauri/src/services/powerpoint/*`
- 변환 상태 UI
- 결과 열기 액션

## DoD
- `.pptx` 파일 1개를 선택해 PDF로 저장할 수 있다
- 저장 경로를 선택할 수 있다
- 변환 성공 시 파일이 실제 생성된다
- 변환 실패 시 "실패했습니다 + 이유 1줄" 형태로 표시된다
- 변환 완료 후 결과 폴더 열기가 동작한다
- 폰트 주의 문구가 노출된다

## 의도 전달용 코드 스니펫
```tsx
// src/features/ppt-to-pdf/components/PptToPdfPanel.tsx
async function handleConvert() {
  try {
    setStatus("loading");
    await invoke("convert_pptx_to_pdf", {
      inputPath,
      outputPath,
    });
    setStatus("success");
  } catch (e) {
    setStatus("error");
    setMessage("변환에 실패했습니다. PowerPoint 실행 상태를 확인해 주세요.");
  }
}
```

```rust
// src-tauri/src/commands/ppt_to_pdf.rs
#[tauri::command]
pub async fn convert_pptx_to_pdf(input_path: String, output_path: String) -> Result<(), String> {
    crate::services::powerpoint::convert_to_pdf(&input_path, &output_path)
        .map_err(|e| format!("PowerPoint 호출 실패: {}", e))
}
```

---

# 6. PDF 병합

## 목표
여러 PDF를 하나의 PDF로 병합하는 기능을 구현한다.

## 작업내용
- 파일 다중 선택 UI 구현
- 파일 리스트 표시
- 순서 변경 UI 구현
- 파일 삭제 액션 구현
- 병합 command 작성
- 저장 파일명 입력 UI 구현
- 병합 결과 저장 및 결과 알림

## 산출물
- PdfMerge 패널/화면
- `src-tauri/src/commands/pdf_merge.rs`
- PDF 병합 서비스
- 파일 리스트 UI

## DoD
- PDF 여러 개를 선택할 수 있다
- 리스트 순서를 바꿀 수 있다
- 특정 파일을 제거할 수 있다
- 병합 결과를 새 PDF로 저장할 수 있다
- 저장 실패 시 간단 오류 메시지가 나온다

## 의도 전달용 코드 스니펫
```tsx
// src/features/pdf-merge/components/PdfMergePanel.tsx
const [items, setItems] = useState<string[]>([]);

function moveItem(from: number, to: number) {
  const next = [...items];
  const [picked] = next.splice(from, 1);
  next.splice(to, 0, picked);
  setItems(next);
}
```

```rust
// src-tauri/src/commands/pdf_merge.rs
#[tauri::command]
pub async fn merge_pdfs(input_paths: Vec<String>, output_path: String) -> Result<(), String> {
    crate::services::pdf::merge::merge_files(input_paths, output_path)
        .map_err(|e| format!("PDF 병합 실패: {}", e))
}
```

---

# 7. PDF 용량 줄이기

## 목표
PDF를 제출 제한 용량에 맞게 줄일 수 있도록 한다.  
사용자 노출 옵션은 `최고품질 / 균형 / 소용량 / 목표 용량 입력`을 사용한다. :contentReference[oaicite:5]{index=5}

## 작업내용
- PDF 선택 UI 구현
- 옵션 선택 UI 구현
- 목표 용량 입력 UI 구현
- 압축 command 작성
- 원본 크기/결과 크기 표시
- 압축 결과 저장
- 실패 메시지 처리

## 산출물
- PdfCompress 패널/화면
- `src-tauri/src/commands/pdf_compress.rs`
- PDF 압축 서비스
- 크기 비교 UI

## DoD
- 원본 PDF를 선택할 수 있다
- 네 가지 옵션 중 하나를 선택할 수 있다
- 목표 용량을 입력할 수 있다
- 결과 파일이 생성된다
- 원본 파일 크기와 결과 파일 크기를 볼 수 있다
- 실패 시 이유 1줄 메시지가 표시된다

## 의도 전달용 코드 스니펫
```tsx
type CompressPreset = "high" | "balanced" | "small" | "target";

const [preset, setPreset] = useState<CompressPreset>("high");
const [targetMb, setTargetMb] = useState<number | "">("");
```

```rust
// src-tauri/src/commands/pdf_compress.rs
#[tauri::command]
pub async fn compress_pdf(
    input_path: String,
    output_path: String,
    preset: String,
    target_mb: Option<f32>,
) -> Result<(), String> {
    crate::services::pdf::compress::run(input_path, output_path, preset, target_mb)
        .map_err(|e| format!("PDF 용량 줄이기 실패: {}", e))
}
```

---

# 8. 제안서 패키징

## 목표
여러 PPTX/PDF 파일을 한 번에 등록해, 순서만 정한 뒤 PDF 변환 + 병합 + 선택적 압축까지 처리하는 단순화된 패키징 기능을 구현한다.

## 작업내용
- 입력 파일 리스트 UI 구현
- PPTX/PDF 혼합 등록 지원
- 파일 유형 표시
- 순서 변경 기능 구현
- 출력 파일명 자동 추천 로직 구현
- 패키징 command 작성
- 선택적 최종 용량 줄이기 옵션 연결
- 최종 결과 저장

## 산출물
- ProposalPackaging 화면
- `src-tauri/src/commands/proposal_pack.rs`
- 패키징 서비스
- 자동 파일명 추천 유틸

## DoD
- PPTX/PDF 혼합 파일을 등록할 수 있다
- 사용자가 순서를 변경할 수 있다
- PPTX는 PDF로 변환되고, PDF는 그대로 포함된다
- 최종 PDF가 하나로 생성된다
- 파일명은 자동 추천되지만 사용자가 수정 가능하다
- 사용자가 원하면 최종 압축을 적용할 수 있다

## 의도 전달용 코드 스니펫
```tsx
// src/features/proposal-packaging/services/suggestFileName.ts
export function suggestProposalFileName(projectName: string, date: string) {
  return `${projectName}_제안서_${date}_v01.pdf`;
}
```

```rust
// src-tauri/src/commands/proposal_pack.rs
#[tauri::command]
pub async fn build_proposal_package(
    items: Vec<String>,
    output_path: String,
    compress_preset: Option<String>,
) -> Result<(), String> {
    crate::services::pdf::proposal::build(items, output_path, compress_preset)
        .map_err(|e| format!("제안서 패키징 실패: {}", e))
}
```

---

# 9. 업무 매뉴얼

## 목표
하단 영역에서 업무 팁과 체크리스트를 7개 수준으로 제공한다.  
메인 기능을 방해하지 않으면서도 필요한 정보를 바로 확인할 수 있게 한다.

## 작업내용
- 매뉴얼 데이터 JSON 정의
- 하단 영역 UI 구현
- 카드형 기본안 구현
- 상세 모달 구현
- 콘텐츠 정렬/표시 로직 구현

## 산출물
- `resources/manual/default-manual-items.json`
- ManualSection 컴포넌트
- ManualCard 컴포넌트
- ManualDetailModal

## DoD
- 홈 하단에 7개 수준의 매뉴얼 아이템이 보인다
- 각 아이템의 제목과 요약을 볼 수 있다
- 더 자세한 내용은 모달로 볼 수 있다
- 매뉴얼이 메인 기능 영역을 침범하지 않는다
- JSON 수정만으로 콘텐츠를 바꿀 수 있다

## 의도 전달용 코드 스니펫
```json
[
  {
    "id": "check-01",
    "title": "제출 전 체크리스트",
    "summary": "파일명, 용량, 페이지 순서 확인",
    "detail": "최종 제출 전 파일명, 용량 제한, 병합 순서를 다시 확인합니다.",
    "category": "checklist",
    "order": 1,
    "enabled": true
  }
]
```

---

# 10. 설정 / 테마 / About

## 목표
사용자가 꼭 필요한 환경값만 간단하게 바꿀 수 있는 설정 화면을 제공한다.  
여기에 테마 전환, 기본 저장 경로, 결과 폴더 자동 열기, 버전 정보, About를 포함한다.

## 작업내용
- 설정 화면 구현
- JSON 기반 설정 저장/불러오기 구현
- 기본 저장 경로 선택 기능
- 결과 폴더 자동 열기 옵션 구현
- 테마 전환 UI 연결
- About 화면 구현
- 프로그램명/아이콘/개발자명/사인 이미지 표시

## 산출물
- Settings 페이지
- About 페이지 또는 모달
- 설정 저장 서비스
- 테마 토글 연동

## DoD
- 기본 저장 경로를 바꿀 수 있다
- 자동 열기 옵션을 바꿀 수 있다
- 테마가 저장된다
- About에서 버전/개발자/브랜딩 정보를 볼 수 있다

## 의도 전달용 코드 스니펫
```ts
// src/features/settings/services/settingsStorage.ts
export interface AppSettings {
  defaultOutputPath: string;
  openFolderAfterComplete: boolean;
  theme: "light" | "dark";
}
```

---

# 11. 작업 기록

## 목표
홈에는 노출하지 않지만, 보조 기능으로 최근 작업 이력을 조회하고 삭제할 수 있게 한다.

## 작업내용
- SQLite 스키마 정의
- 작업 기록 저장 로직 구현
- 작업 기록 조회 화면 구현
- 개별 삭제 / 전체 삭제 구현
- 성공/실패 여부 저장

## 산출물
- SQLite migration
- History 페이지
- history service
- 기록 삭제 액션

## DoD
- 작업이 끝날 때 기록이 저장된다
- 기록 목록에서 시간/기능/파일명/경로를 볼 수 있다
- 개별 삭제가 된다
- 전체 삭제가 된다
- 로컬 저장만 수행한다

## 의도 전달용 코드 스니펫
```sql
CREATE TABLE IF NOT EXISTS task_history (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  job_type TEXT NOT NULL,
  source_paths TEXT NOT NULL,
  output_path TEXT NOT NULL,
  output_file_name TEXT NOT NULL,
  status TEXT NOT NULL,
  created_at TEXT NOT NULL
);
```

---

# 12. 공통 오류 UX 및 피드백

## 목표
실패 시 사용자가 이해할 수 있는 수준으로만 메시지를 보여준다.  
복잡한 기술 설명 대신 “실패 + 이유 1줄” 원칙을 유지한다.

## 작업내용
- 공통 토스트/알림 컴포넌트 구현
- 성공/실패/진행 상태 배지 정의
- 기능별 오류 메시지 문구 정리
- 로딩 상태 스피너/문구 정의

## 산출물
- Toast 컴포넌트
- StatusMessage 컴포넌트
- 공통 에러 메시지 매핑 테이블

## DoD
- 모든 핵심 기능에서 동일한 오류 톤을 사용한다
- 성공/실패/진행 상태가 시각적으로 구분된다
- 오류 메시지가 길지 않다
- 기술 용어가 과도하게 노출되지 않는다

## 의도 전달용 코드 스니펫
```ts
// src/lib/constants/errorMessages.ts
export const errorMessages = {
  pptConvertFailed: "변환에 실패했습니다. 파일 상태를 확인해 주세요.",
  pdfMergeFailed: "병합에 실패했습니다. 선택한 PDF를 확인해 주세요.",
  pdfCompressFailed: "용량 줄이기에 실패했습니다. 파일을 다시 확인해 주세요.",
  packageFailed: "패키징에 실패했습니다. 등록한 파일을 다시 확인해 주세요.",
};
```

---

# 13. 업데이트 및 배포

## 목표
MSI 배포와 GitHub Releases 기반 업데이트 확인 흐름을 준비한다. :contentReference[oaicite:6]{index=6}

## 작업내용
- Tauri build 설정 정리
- MSI 빌드 확인
- updater 설정 반영
- GitHub Actions release workflow 초안 작성
- 버전 표기 연결
- 설정 화면 내 업데이트 확인 액션 추가

## 산출물
- MSI 빌드 결과물
- `tauri.conf.json` 배포 설정
- GitHub Actions workflow
- updater 설정 코드

## DoD
- 로컬에서 MSI를 생성할 수 있다
- 릴리즈 워크플로 초안이 있다
- 앱 내 버전이 표시된다
- 수동 업데이트 확인 UI가 존재한다

## 의도 전달용 코드 스니펫
```yaml
# .github/workflows/release.yml
name: release
on:
  push:
    tags:
      - "v*"
jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build app
        run: npm run tauri build
```

---

# 14. QA 및 통합 검증

## 목표
핵심 사용자 시나리오 기준으로 앱이 실제 업무 흐름에서 끊기지 않는지 검증한다.

## 작업내용
- 시나리오 기반 테스트 케이스 작성
- PPTX → PDF 테스트
- PDF 병합 테스트
- PDF 압축 테스트
- 제안서 패키징 테스트
- 즐겨찾기 링크 테스트
- 테마 전환 테스트
- 기록 저장/삭제 테스트

## 산출물
- QA 체크리스트
- 기능별 테스트 케이스
- 버그 리스트
- 수정 완료 보고

## DoD
- 핵심 기능 6개가 모두 동작한다
- 실패 메시지가 정상 노출된다
- 홈 화면 구조가 PRD 합의안과 일치한다
- 라이트/다크 모드가 정상 동작한다
- 앱이 기본 사용 시나리오에서 종료되지 않는다

## 의도 전달용 코드 스니펫
```md
- [ ] 즐겨찾기 클릭 시 기본 브라우저가 열린다
- [ ] PPTX 1개를 PDF로 변환할 수 있다
- [ ] PDF 3개를 순서대로 병합할 수 있다
- [ ] 목표 용량 옵션으로 압축할 수 있다
- [ ] 패키징 결과 파일명이 자동 추천된다
- [ ] 매뉴얼 7개가 홈 하단에 노출된다
```

---

# 15. 최종 통합 DoD

## 목표
MVP 릴리즈 가능 상태를 정의한다.

## DoD
- 홈 화면은 상단 즐겨찾기 / 중단 문서 도구 / 하단 매뉴얼 3분할 구조다
- 즐겨찾기는 핵심 기능으로 정상 동작한다
- PPTX → PDF, PDF 병합, PDF 용량 줄이기, 제안서 패키징이 정상 동작한다
- 업무 매뉴얼이 홈 하단에서 보조 정보로 동작한다
- 작업 기록은 홈에 노출되지 않지만 보조 기능으로 조회 가능하다
- 설정 / 테마 / About / 버전 표시가 동작한다
- 오류는 “실패했습니다 + 이유 1줄” 원칙으로 노출된다
- MSI 빌드가 가능하다
- 로컬 환경에서 핵심 기능이 오프라인으로 동작한다

---

# 16. 구현 우선순위

## 1차
- 프로젝트 골격
- 홈 화면 프레임
- 디자인 시스템
- 즐겨찾기

## 2차
- PPTX → PDF
- PDF 병합

## 3차
- PDF 용량 줄이기
- 제안서 패키징

## 4차
- 업무 매뉴얼
- 설정 / 테마 / About

## 5차
- 작업 기록
- 업데이트 / 배포
- QA 및 마감

---

# 17. 개발 메모

- 홈은 복잡한 도구 모음이 아니라 실행 런처다
- 즐겨찾기는 보조 기능이 아니라 메인 기능으로 취급한다
- 제안서 패키징은 고도화보다 단순 병합 흐름에 집중한다
- 작업 기록은 홈에서 드러내지 않는다
- “누르면 된다”는 느낌을 최우선으로 유지한다