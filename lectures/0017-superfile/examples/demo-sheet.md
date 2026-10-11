# superfile 전문가 — 복사 시트

> 강의 흐름 순서대로 정리 | 멘트 없이 명령·키 순서·붙여넣기 블록·표만
> 기준 버전: superfile 1.6.0 (2026-10-02 확인)

§1부터 순서대로 실행하면 강의와 같은 상태(`~/.config/superfile` 설정, `~/work/spf-lab` 실습 폴더, 혼합 키맵)가 만들어집니다. 각 §의 `> 전제:` 는 그 블록을 실행하기 전의 상태입니다.
명령 뒤의 `# 주석` 을 함께 붙여넣으려면 zsh에서 `setopt interactivecomments` 가 켜져 있어야 합니다(oh-my-zsh는 기본으로 켭니다). 꺼져 있으면 주석이 명령 인자로 들어가 `mv`·`spf` 가 엉뚱한 오류를 냅니다.
`키:` 블록은 superfile 화면 안에서 누르는 키 순서입니다 — `ctrl+n` 은 Ctrl+N, 대문자는 Shift와 함께, `<enter>`·`<esc>` 는 해당 키, 그 밖의 글자는 그대로 입력합니다.

---

## § 1. 설치·버전·`spf pl`

> 전제: macOS + Homebrew, superfile 미설치

```bash
brew install superfile        # Homebrew 설치 — 실행 명령은 spf
spf --version                 # superfile version v1.6.0
spf pl                        # 설정·단축키·로그·데이터 경로 출력
spf                           # 첫 실행 — 환영 창은 아무 키로 닫고 q 로 종료
```

#### CLI 명령·플래그 전체

| 명령·플래그 | 줄임 | 기능 |
|---|---|---|
| `spf [경로]...` | — | 경로마다 파일 패널을 열고 실행. 경로가 없으면 `default_directory`, 없는 경로면 zoxide로 찾음(연동 켰을 때) |
| `spf path-list` | `spf pl` | 설정 파일·단축키 파일·로그 파일·설정 폴더·데이터 폴더 경로 출력 |
| `spf pl --lastdir-file` | `--ld` | `Q`(또는 `cd_on_quit`) 종료 때 `cd` 줄을 적는 lastdir 파일 경로 출력 |
| `spf help` | `spf h` | 명령 목록 |
| `--debug-info` | `--di` | 버전·OS·실제로 읽는 설정 파일 경로·환경 변수·외부 도구(bat·zoxide·exiftool 등) 출력 |
| `--fix-hotkeys` | `--fh` | 빠진 단축키를 채워 다시 씀 — **1.6.0에서는 파일을 망가뜨림** (섹션 4) |
| `--fix-config-file` | `--fch` | 빠진 설정 항목을 채워 다시 씀 — **1.6.0에서는 파일을 망가뜨림** (섹션 6) |
| `--print-last-dir` | `--pld` | 종료할 때 마지막 폴더를 표준 출력에 찍음 |
| `--config-file <파일>` | `-c` | 이번 실행만 다른 `config.toml` 사용 (파일이 없으면 오류로 종료) |
| `--hotkey-file <파일>` | `--hf` | 이번 실행만 다른 `hotkeys.toml` 사용 (파일이 없으면 오류로 종료) |
| `--chooser-file <파일>` | `--cf` | 파일을 열려고 하면 그 경로를 이 파일에 쓰고 종료 (다른 프로그램의 파일 선택기로 쓸 때) |
| `--version` / `--help` | `-v` / `-h` | 버전 / 도움말 |

---

## § 2. macOS 경로와 XDG_CONFIG_HOME

> 전제: §1 첫 실행 완료 — `~/Library/Application Support/superfile/` 에 `config.toml`·`hotkeys.toml`·`theme/` 가 있음
> 주의: 첫 줄 결과가 이미 `~/.config` 를 가리키면 2·3번째 줄은 건너뜁니다. `theme` 폴더를 빠뜨리면 테마가 조용히 기본값으로 돌아갑니다

```bash
echo "XDG_CONFIG_HOME=[$XDG_CONFIG_HOME]"                                       # 비어 있으면 macOS 기본 경로를 씀
echo 'export XDG_CONFIG_HOME="$HOME/.config"' >> ~/.zshrc && source ~/.zshrc    # 설정 폴더를 ~/.config 로
mkdir -p ~/.config/superfile && mv ~/Library/Application\ Support/superfile/{config.toml,hotkeys.toml,theme} ~/.config/superfile/   # 설정 3종만 이동
spf pl                                                                          # 설정은 ~/.config, 로그·데이터는 그대로
```

---

## § 3. 첫 화면 구조와 `?` 도움말

> 전제: §2 완료, 새 터미널 또는 같은 셸

```bash
mkdir -p ~/work/spf-lab/{src,docs,logs,inbox} && cd ~/work/spf-lab                                    # 실습 폴더 4개
printf 'def main():\n    print("hello spf")\n\nif __name__ == "__main__":\n    main()\n' > src/app.py   # 미리보기용 코드
printf '# 회의 메모\n\n- superfile 키맵 정리\n- tmux 충돌 확인\n' > docs/notes.md                         # 미리보기용 문서
for i in 1 2 3; do printf '2026-10-02 INFO line %s\n' $i > logs/app-$i.log; done                    # 로그 3개
printf 'id,name\n1,alpha\n2,beta\n' > inbox/data.csv && touch inbox/report-2026.txt inbox/.draft     # .draft 는 숨김 파일
spf                                                                                                   # 실습 폴더에서 실행
```

```text
키: ?  →  (목록 훑기)  →  q        # 도움말 열기 → 닫기 (오른쪽 아래 1/47)
```

---

## § 4. 이동과 패널

> 전제: §3 의 superfile 화면, 커서는 `docs`

```text
키: j j l h          # logs 로 들어갔다 나옴 — 나오면 커서는 logs
키: n w              # 새 패널(홈 폴더) → 닫기
키: N tab H w        # 지금 폴더로 패널 하나 더 → 다음 → 이전 → 닫기
키: f f              # 미리보기 끄기 → 켜기
키: s j s            # 사이드바 초점 → 아래로 → 파일 패널로
키: p p m m          # 프로세스 초점 → 돌아옴 → 메타데이터 초점 → 돌아옴
```

#### 기본 키 전체

| 키 | 기능 | 이름 (`hotkeys.toml`) | 계층 |
|---|---|---|---|
| `enter`, `right`, `l` | 선택 확정 — 폴더 들어가기·파일 열기 / 선택 모드에서는 항목 선택 | `confirm` | 전역 |
| `Q` | 종료하면서 셸을 마지막 폴더로 cd (셸 함수 필요) | `cd_quit` | 전역 |
| `q`, `esc` | 종료 · 모달 닫기 | `quit` | 전역 |
| `down`, `j` | 아래로 | `list_down` | 전역 |
| `up`, `k` | 위로 | `list_up` | 전역 |
| `pgdown` | 페이지 아래 | `page_down` | 전역 |
| `pgup` | 페이지 위 | `page_up` | 전역 |
| `w` | 초점 패널 닫기 | `close_file_panel` | 전역 |
| `n` | 새 파일 패널 (홈 폴더에서) | `create_new_file_panel` | 전역 |
| `tab`, `L` | 다음 파일 패널로 | `next_file_panel` | 전역 |
| `o` | 정렬 메뉴 | `open_sort_options_menu` | 전역 |
| `P` | 폴더를 사이드바에 고정/해제 | `pinned_directory` | 전역 |
| `shift+left`, `H` | 이전 파일 패널로 | `previous_file_panel` | 전역 |
| `N` | 지금 폴더로 패널 하나 더 (분할) | `split_file_panel` | 전역 |
| `f` | 미리보기 패널 끄기/켜기 | `toggle_file_preview_panel` | 전역 |
| `R` | 정렬 순서 뒤집기 | `toggle_reverse_sort` | 전역 |
| `m` | 메타데이터 패널로 초점 | `focus_on_metadata` | 전역 |
| `p` | 프로세스 패널로 초점 | `focus_on_process_bar` | 전역 |
| `s` | 사이드바로 초점 | `focus_on_sidebar` | 전역 |
| `ctrl+n` | 파일·폴더 만들기 (끝에 `/` 면 폴더, `a/b/c.txt` 처럼 한 번에 여러 단계 가능) | `file_panel_item_create` | 전역 |
| `ctrl+r` | 이름 바꾸기 | `file_panel_item_rename` | 전역 |
| `ctrl+c` | 선택 항목을 클립보드에 복사 | `copy_items` | 전역 |
| `ctrl+x` | 선택 항목을 클립보드에 잘라내기 | `cut_items` | 전역 |
| `ctrl+d`, `delete` | 선택 항목을 휴지통으로 | `delete_items` | 전역 |
| `ctrl+v`, `ctrl+w` | 클립보드 항목을 현재 패널에 붙여넣기 | `paste_items` | 전역 |
| `D` | 선택 항목 영구 삭제 | `permanently_delete_items` | 전역 |
| `ctrl+a` | zip으로 압축 | `compress_file` | 전역 |
| `ctrl+e` | 압축 풀기 | `extract_file` | 전역 |
| `E` | 현재 폴더를 편집기로 열기 | `open_current_directory_with_editor` | 전역 |
| `e` | 파일을 편집기로 열기 | `open_file_with_editor` | 전역 |
| `v` | 선택 모드 ↔ 일반(Browser) 모드 | `change_panel_mode` | 전역 |
| `ctrl+p` | 현재(또는 선택한) 파일·폴더 경로 복사 | `copy_path` | 전역 |
| `c` | 현재 작업 폴더 경로 복사 | `copy_present_working_directory` | 전역 |
| `:` | 셸 모드 프롬프트 | `open_command_line` | 전역 |
| `?` | 도움말 (지금 읽힌 키 목록) | `open_help_menu` | 전역 |
| `>` | spf 모드 프롬프트 | `open_spf_prompt` | 전역 |
| `z` | zoxide 이동 창 (연동 켰을 때) | `open_zoxide` | 전역 |
| `.` | 숨김 파일 보이기/숨기기 (상태 저장) | `toggle_dot_file` | 전역 |
| `F` | 아래쪽 패널 묶음 보이기/숨기기 (상태 저장) | `toggle_footer` | 전역 |
| `enter` | 입력 확정 | `confirm_typing` | 입력 |
| `ctrl+c`, `esc` | 입력 취소 | `cancel_typing` | 입력 |
| `h`, `left`, `backspace` | 상위 폴더로 | `parent_directory` | 모드(일반) |
| `/` | 검색줄 | `search_bar` | 모드(일반) |
| `shift+down`, `J` | 선택하며 아래로 | `file_panel_select_mode_items_select_down` | 모드(선택) |
| `shift+up`, `K` | 선택하며 위로 | `file_panel_select_mode_items_select_up` | 모드(선택) |
| `A` | 현재 폴더 전체 선택 | `file_panel_select_all_items` | 모드(선택) |

---

## § 5. 선택 모드와 파일 조작

> 전제: §4 끝 상태 — `~/work/spf-lab` 패널 하나, 커서는 `logs`
> 주의: `D` 는 되돌릴 수 없는 영구 삭제입니다

```text
키: k l                          # inbox 로 들어감
키: ctrl+n archive/ <enter>      # 이름 끝 / → 폴더 생성
키: j j ctrl+r -q3 <enter>       # report-2026.txt → report-2026-q3.txt (커서가 확장자 앞)
키: v K l ctrl+c                 # 선택 모드 → report 선택하며 위로 → data.csv 선택 → 2개 복사
키: v k l ctrl+v h               # 일반 모드 → archive 로 들어가 붙여넣기 → 나옴
키: ctrl+a j ctrl+e              # archive 압축 → archive.zip 으로 → 풀기(archive(1) 폴더)
키: ctrl+d <enter>               # archive(1) 를 휴지통(~/.Trash)으로
키: D <enter>                    # archive.zip 영구 삭제
```

---

## § 6. 검색·정렬·숨김 파일

> 전제: §5 끝 상태 — `inbox` 패널

```text
키: / rep <enter>      # 이름 필터 — 엔터 후에도 필터 유지
키: / <esc>            # 필터 해제
키: o j <enter> R      # 정렬 메뉴 → Size → 순서 뒤집기
키: . .                # 숨김 파일 보이기 → 숨기기 (상태가 다음 실행에도 이어짐)
키: F F q              # 아래쪽 패널 숨기기 → 보이기 → 종료
```

---

## § 7. `:` 셸 모드와 `>` spf 모드

> 전제: 셸이 `~/work/spf-lab` 에 있음

```bash
spf    # ~/work/spf-lab 에서 실행
```

```text
키: :                                                     # 셸 모드 (패널 폴더에서 /bin/sh 로 실행, 5초 제한)
    mkdir -p backup && cp logs/*.log backup/ && ls backup <enter>   # Success : status 0 + 출력
    ls no-such-file <enter>                                         # Error : status 1 + 오류 출력
키: >                                                     # 입력란이 비어 있을 때 → spf 모드
    split <enter>                                         # 지금 폴더로 패널 하나 더
    cd docs <enter>                                       # 현재 패널을 docs 로
    cd ${HOME}/work/spf-lab/logs <enter>                  # 환경 변수 치환
    open $(dirname $(pwd)) <enter>                        # 명령 치환 — $(pwd) 는 패널 폴더 → spf-lab 을 새 패널로
키: <esc> w w q                                           # 프롬프트 닫기 → 패널 두 개 닫기 → 종료
```

#### spf 모드 명령 전체

| 명령 | 기능 | 예 |
|---|---|---|
| `split` | 지금 패널의 폴더로 새 패널 열기 (인자 없음) | `split` |
| `cd <경로>` | 현재 패널의 폴더 바꾸기 | `cd docs` · `cd ${HOME}/work` · `cd ~/work` |
| `open <경로>` | 지정한 경로로 새 패널 열기 | `open $(dirname $(pwd))` |

---

## § 8. cd_on_quit · `Q` · zoxide

> 전제: §7 완료, 셸은 `~/work/spf-lab`. zoxide 부분은 zoxide 설치 필요

붙여넣기 — `~/.zshrc` 끝에 추가:

```bash
cat >> ~/.zshrc <<'EOF'

# ~/.zshrc — superfile: Q로 종료하면 마지막 폴더로 cd (경로는 spf에게 물어봄)
spf() {
  local lastdir="$(command spf pl --lastdir-file)"         # lastdir 파일 위치 — XDG 설정을 그대로 따라감
  command spf "$@"                                         # 실제 superfile 실행
  if [ -s "$lastdir" ]; then                               # Q(또는 cd_on_quit)로 종료해 내용이 있을 때만
    . "$lastdir" && : > "$lastdir"                         # cd 실행 후 파일을 비움 — 다음 q 종료 때 다시 cd 하지 않도록
  fi
}
EOF
source ~/.zshrc   # 함수 적용
```

```text
spf  →  키: j l Q  →  pwd       # docs 로 들어가 Q → 셸이 ~/work/spf-lab/docs 로 이동
spf  →  키: h q    →  pwd       # 상위로 올라가 q → 셸은 docs 그대로
```

```bash
sed -i '' 's/^zoxide_support = false/zoxide_support = true          # z — zoxide 목록, 들어간 폴더를 zoxide에 기록 (zoxide 설치 필요)/' ~/.config/superfile/config.toml   # zoxide 연동 켜기
zoxide add ~/work/spf-lab/logs                                                               # 데모용으로 logs 를 기록
spf ~/work/spf-lab                                                                           # z → log → enter → q
spf logs                                                                                     # 없는 경로 → zoxide 가 찾은 logs 로 열림 → q
```

---

## § 9. 파일 열기와 3계층 구조

> 전제: §8 완료 (셸 위치 상관없음)

```bash
spf pl | grep -i 'hotkeys file'                                                       # 단축키 파일 위치
grep -cE '^[a-z_]+ *=' ~/.config/superfile/hotkeys.toml                                # 항목 수 — 46
grep -nE 'Global hotkeys|Typing hotkeys|Mode-Specific Hotkeys' ~/.config/superfile/hotkeys.toml   # 세 구역 시작 줄
nvim ~/.config/superfile/hotkeys.toml                                                  # 구조 읽기 — :q 로 닫음
```

#### 3계층 구조

| 구역 (파일 주석) | 항목 수 | 규칙 (파일 주석 원문 요지) | 실제 처리 |
|---|---|---|---|
| Global hotkeys | 39 | 서로 겹치면 안 됨 (`These hotkeys should be unique.`) | 입력 창이 없을 때 모드와 무관하게 먼저 검사 |
| Typing hotkeys | 2 | 다른 모든 키를 덮어씀 (`can override all hotkeys`) | 이름 입력·검색·프롬프트·확인 창이 열려 있을 때 가장 먼저 검사 |
| Mode-Specific Hotkeys | 5 (일반 2 · 선택 3) | 다른 모드 키와는 겹쳐도 되지만 전역 키와는 안 됨 | 전역 키에 걸리지 않은 키만 모드별로 검사 |

---

## § 10. 키 문법 실험

> 전제: `~/.config/superfile/hotkeys.toml` 이 기본값 상태
> 주의: 실험은 `presets/lab.toml` 복사본에서만 합니다

```bash
cd ~/.config/superfile && mkdir -p presets && cp hotkeys.toml presets/lab.toml                         # 실험용 복사본
sed -i '' "s/^toggle_footer = .*/toggle_footer = ['', 'F']/" presets/lab.toml && spf --hf presets/lab.toml   # 첫 칸 비우기 → 시작 거부
```

```bash
sed -i '' "s/^toggle_footer = .*/toggle_footer = ['shift+f', 'ctrl+i']/" presets/lab.toml && spf --hf presets/lab.toml   # shift+ 표기와 ctrl+i
```

```text
키: F ctrl+i q       # 둘 다 푸터를 끄지 못함 (ctrl+i 는 탭 = 다음 패널) → 종료
```

#### 키 문법 규칙

| 규칙 | 예 | 확인 결과 (v1.6.0, tmux 안) |
|---|---|---|
| 값은 문자열 배열, 칸 수 제한 없음 | `confirm = ['enter', 'right', 'l']` | 세 칸 모두 동작 |
| 안 쓰는 칸은 빈 문자열 | `quit = ['q', '']` | 빈 칸은 도움말에 표시 안 됨 |
| **첫 칸은 비울 수 없음** | `toggle_footer = ['', 'F']` | `Hotkey value for "ToggleFooter" is invalid : Hotkey list is empty` 로 시작 거부 (내부 이름으로 표시) |
| 대문자는 글자 그대로 | `'F'`, `'D'`, `'Q'` | `'shift+f'` 로 적으면 `F` 를 눌러도 반응 없음 |
| 특수 키 이름 | `enter` `esc` `tab` `backspace` `delete` `up` `down` `left` `right` `pgup` `pgdown` | 기본·vim 프리셋에 쓰인 이름 |
| shift + 특수 키 | `shift+left` `shift+up` `shift+down` `shift+tab` | `shift+left` 는 tmux `bind -n` 이 가로채면 닿지 않음 (섹션 7) |
| ctrl + 글자 | `ctrl+n`, `ctrl+g` | 동작 |
| `ctrl+i` 금지 | 탭과 같은 신호 | `ctrl+i` 바인딩은 무반응, 누르면 다음 패널로 이동(= `tab`) |
| `ctrl+m` 금지 | 엔터와 같은 신호 | 선택 모드에서 `ctrl+m` 이 엔터처럼 항목 선택 |
| `ctrl+[` 금지 | esc와 같은 신호 | 기본 키맵에서 `ctrl+[` 로 superfile이 종료(= `esc`) |
| `ctrl+?` 금지 | backspace와 같은 신호 | `ctrl+?` 로 상위 폴더 이동(= `backspace`) |

---

## § 11. 문서 변수명 함정과 `--fix-hotkeys`

> 전제: 셸이 `~/.config/superfile`, `presets/` 폴더 있음
> 주의: 1.6.0의 `--fix-hotkeys` 는 파일을 망가뜨립니다 — 반드시 복사본(`presets/typo.toml`)에서만 재현합니다

```bash
cp hotkeys.toml presets/typo.toml && sed -i '' "s/^focus_on_sidebar = .*/focus_on_side_bar = ['b', '']/" presets/typo.toml   # 문서 표기로 바꿔 재현
spf --hf presets/typo.toml                       # missing fields: [focus_on_sidebar] — b 무반응, s 는 기본값으로 동작 → q
spf --hf presets/typo.toml --fix-hotkeys         # 빠진 키를 채웠다는 메시지 → q
spf --hf presets/typo.toml                       # error decoding TOML file — 시작 거부
```

```bash
mv presets/typo.toml presets/typo.broken.toml && mv presets/typo.toml.bak-* presets/typo.toml   # 백업으로 되돌림 (망가진 파일은 보관)
comm -3 <(grep -oE '^[a-z_]+' hotkeys.toml | sort) <(grep -oE '^[a-z_]+' presets/typo.toml | sort)   # 왼쪽=빠진 이름, 오른쪽=모르는 이름
sed -i '' 's/^focus_on_side_bar = /focus_on_sidebar = /' presets/typo.toml && spf --hf presets/typo.toml   # 진짜 이름으로 → b 로 사이드바
```

#### 문서 표기와 실제 이름

| 1.6.0 문서 표기 | 실제 이름 | 비고 |
|---|---|---|
| `confirm_typing` (선택 확정 행) | `confirm` | `confirm_typing` 은 따로 있는 입력 확정 키(`enter`) |
| `focus_on_side_bar` | `focus_on_sidebar` | 밑줄 위치 |
| `parent_folder` | `parent_directory` | folder ↔ directory |
| `pinned_folder` | `pinned_directory` | folder ↔ directory |
| `file_panel_select_all_item` | `file_panel_select_all_items` | 단수 ↔ 복수 |
| `file_panel_select_mode_item_select_up` | `file_panel_select_mode_items_select_up` | 단수 ↔ 복수 |
| `file_panel_select_mode_item_select_down` | `file_panel_select_mode_items_select_down` | 단수 ↔ 복수 |
| `copy_single_item` | `copy_items` | 일반·선택 모드가 한 이름 |
| `file_panel_select_mode_item_copy` | `copy_items` | 일반·선택 모드가 한 이름 |
| `file_panel_select_mode_item_cut` | `cut_items` | 잘라내기도 한 이름 |
| `paste_item` | `paste_items` | 단수 ↔ 복수 |
| `delete_item` | `delete_items` | 일반·선택 모드가 한 이름 |
| `file_panel_select_mode_item_delete` | `delete_items` / `permanently_delete_items` | 문서는 두 삭제 행에 같은 이름을 적음 |
| `current_directory_with_editor` | `open_current_directory_with_editor` | 앞의 `open_` 누락 |

---

## § 12. vim 프리셋 적용과 기본값과의 차이

> 전제: 셸이 `~/.config/superfile`, `hotkeys.toml` 은 아직 기본값
> 주의: vim 프리셋에서는 `d`·`p`·`x`·`y` 한 글자가 파일을 바꿉니다. 종료는 `ctrl+c`

```bash
curl -fsSL https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/vimHotkeys.toml -o presets/vim.toml   # 1.6.0 vim 프리셋
diff <(grep -E '^[a-z_]+ *=' hotkeys.toml | sort) <(grep -E '^[a-z_]+ *=' presets/vim.toml | sed 's/ *#.*//' | sort) | grep '^>'   # 바뀐 키만
spf --hf presets/vim.toml ~/work/spf-lab                                                       # 본 설정은 그대로 두고 시험
```

```text
키: j <enter> h l - <esc> ctrl+s ctrl+s d <esc> ctrl+c
# docs 로 들어감 → h·l 무반응 → - 로 상위 → esc 로는 안 꺼짐 → 사이드바 왕복 → d 휴지통 창 → esc 취소 → ctrl+c 종료
```

#### vim 프리셋과 기본값의 차이 전체

| 이름 | 기본 | vim 프리셋 | 주의점 |
|---|---|---|---|
| `cancel_typing` | `ctrl+c`, `esc` | `esc` | 입력 취소는 esc만 |
| `change_panel_mode` | `v` | `m` | 선택 모드가 `m` |
| `close_file_panel` | `w` | `q` | `q` 가 종료가 아니라 패널 닫기 |
| `confirm` | `enter`, `right`, `l` | `enter` | `l` 로 들어가지 않음 |
| `copy_items` | `ctrl+c` | `y` | 한 글자로 복사 |
| `copy_path` | `ctrl+p` | `Y` | — |
| `cut_items` | `ctrl+x` | `x` | 한 글자로 잘라내기 |
| `delete_items` | `ctrl+d`, `delete` | `d` | **한 글자로 휴지통** |
| `file_panel_item_create` | `ctrl+n` | `a` | 한 글자로 만들기 |
| `file_panel_item_rename` | `ctrl+r` | `r` | 한 글자로 이름 바꾸기 |
| `file_panel_select_mode_items_select_down` | `shift+down`, `J` | `J` | — |
| `file_panel_select_mode_items_select_up` | `shift+up`, `K` | `K` | — |
| `focus_on_metadata` | `m` | `ctrl+d` | Neovim의 반 페이지 키가 메타데이터 초점 |
| `focus_on_process_bar` | `p` | `ctrl+p` | — |
| `focus_on_sidebar` | `s` | `ctrl+s` | tmux 안에서 `ctrl+s` 동작 확인 |
| `list_down` | `down`, `j` | `j` | 화살표 아래 없음 |
| `list_up` | `up`, `k` | `k` | 화살표 위 없음 |
| `next_file_panel` | `tab`, `L` | `tab` | — |
| `parent_directory` | `h`, `left`, `backspace` | `-` | **`h` 로 나가지 않음** |
| `paste_items` | `ctrl+v`, `ctrl+w` | `p` | 한 글자로 붙여넣기 |
| `previous_file_panel` | `shift+left`, `H` | `shift+tab` | — |
| `quit` | `q`, `esc` | `ctrl+c` | **종료는 ctrl+c, esc로는 안 꺼짐**. 확인 창 취소도 `(ctrl+c) Cancel` |
| `toggle_footer` | `F` | `ctrl+f` | — |

---

## § 13. Neovim 사용자용 혼합 키맵 직접 작성

> 전제: 셸이 `~/.config/superfile`, `hotkeys.toml` 은 기본값

```bash
cp hotkeys.toml presets/nvim-mix.toml   # 기본값에서 출발
sed -i '' \
  -e "s/^quit = .*/quit = ['q', '']                       # esc 제거 — Neovim 습관으로 esc를 눌러도 종료되지 않음/" \
  -e "s/^page_down = .*/page_down = ['ctrl+d', 'pgdown']        # Neovim 반 페이지 아래 습관 — 파일을 바꾸지 않는 동작에 배정/" \
  -e "s/^page_up = .*/page_up = ['ctrl+u', 'pgup']            # ctrl+d와 짝 — 반 페이지 위/" \
  -e "s/^delete_items = .*/delete_items = ['ctrl+t', 'delete']     # ctrl+d를 비워 주고 휴지통(trash)의 t — ctrl 원칙 유지/" \
  -e "s/^paste_items = .*/paste_items = ['ctrl+v', '']           # ctrl+w 제거 — Neovim 창 이동 습관이 붙여넣기로 이어지지 않게/" \
  presets/nvim-mix.toml
nvim -d hotkeys.toml presets/nvim-mix.toml   # 바뀐 다섯 줄 확인 — :qa 로 닫음
```

---

## § 14. `--hotkey-file`·`-c` 로 분리해 시험하고 승격

> 전제: §13 의 `presets/nvim-mix.toml` 있음, 셸이 `~/.config/superfile`

```bash
grep -E '^[a-z_]+ *=' presets/nvim-mix.toml | sed -E "s/ *#.*//; s/ *= *\[/ /; s/\]//; s/[',]/ /g" | awk '{for(i=2;i<=NF;i++) k[$i]=k[$i]" "$1} END{for(x in k) if (split(k[x],a," ")>1) print x " ->" k[x]}' | sort   # 겹친 키 목록
spf --hf presets/nvim-mix.toml --debug-info | grep -i 'hotkeys file'   # 이 실행이 읽을 단축키 파일
spf --hf presets/nvim-mix.toml ~/work/spf-lab                           # 혼합 키맵으로 시험
```

```text
키: ctrl+d ctrl+u <esc> ctrl+t <esc> q     # 반 페이지 이동 → esc 로는 안 꺼짐 → ctrl+t 휴지통 창 → 취소 → 종료
```

```bash
cp config.toml presets/config-lab.toml && sed -i '' 's/^sidebar_width = .*/sidebar_width = 0/' presets/config-lab.toml   # 실험용 설정
spf -c presets/config-lab.toml --hf presets/nvim-mix.toml ~/work/spf-lab                                                   # 사이드바 없는 화면 → q
cp hotkeys.toml presets/default.toml && cp presets/nvim-mix.toml hotkeys.toml                                              # 기본값 보관 후 승격
```

되돌리기: `cp presets/default.toml hotkeys.toml`

---

## § 15. 편집기·테마·Nerd Font·시작 폴더

> 전제: 셸이 `~/.config/superfile`, Neovim 설치

```bash
cp config.toml presets/config-before.toml   # 바꾸기 전 설정 보관
sed -i '' \
  -e 's|^editor = .*|editor = "nvim"                 # e — 파일을 Neovim으로 (비우면 $EDITOR, 그것도 없으면 nano)|' \
  -e 's|^dir_editor = .*|dir_editor = "nvim"             # E — 현재 폴더를 Neovim으로 (비우면 macOS는 open)|' \
  -e 's|^theme = .*|theme = "tokyonight"            # theme/ 폴더의 파일 이름 (확장자 제외)|' \
  config.toml
diff presets/config-before.toml config.toml | grep '^>'   # 바뀐 세 줄
spf ~/work/spf-lab/src                                    # e → :q → E → :q → q
```

```text
키: e  →  :q  →  E  →  :q  →  q      # 파일을 Neovim으로 → 닫기 → 폴더를 Neovim으로 → 닫기 → 종료
```

---

## § 16. 미리보기·bat·zoxide·metadata · `--fix-config-file`

> 전제: §15 완료, bat 설치. 셸이 `~/.config/superfile`
> 주의: 1.6.0의 `--fix-config-file` 은 파일을 망가뜨립니다 — 빠진 항목은 손으로 `[open_with]` 위에 넣습니다

```bash
sed -i '' \
  -e 's|^code_previewer = .*|code_previewer = "bat"          # 코드 미리보기를 bat으로 (bat 설치 필요, 비우면 내장 chroma)|' \
  -e 's|^enable_file_preview_border = .*|enable_file_preview_border = true   # 미리보기 패널에 테두리 — 파일 패널과 구분|' \
  config.toml
spf --debug-info | grep -A8 '^Dependencies'   # bat·zoxide·exiftool 확인
spf ~/work/spf-lab/src                        # 테두리 있는 bat 미리보기 → q
```

```bash
grep -v '^shell_close_on_success' config.toml > presets/config-old.toml                    # 한 항목이 빠진 옛 설정 재현
comm -13 <(grep -oE '^[a-z0-9_]+' presets/config-old.toml | sort -u) <(grep -oE '^[a-z0-9_]+' presets/config-before.toml | sort -u)   # 빠진 이름
perl -0pi -e 's/^\[open_with\]/shell_close_on_success = false   # 셸 명령 성공 후 프롬프트 자동 닫기\n\n[open_with]/m' presets/config-old.toml   # [open_with] 위에 넣기
spf -c presets/config-old.toml ~/work/spf-lab                                              # 경고 없이 실행 → q
```

강의 끝 상태에서 `config.toml` 이 기본값과 다른 줄:

```toml
# ~/.config/superfile/config.toml — 기본값에서 바꾼 줄만
editor = "nvim"                 # e — 파일을 Neovim으로 (비우면 $EDITOR, 그것도 없으면 nano)
dir_editor = "nvim"             # E — 현재 폴더를 Neovim으로 (비우면 macOS는 open)
theme = "tokyonight"            # theme/ 폴더의 파일 이름 (확장자 제외)
code_previewer = "bat"          # 코드 미리보기를 bat으로 (bat 설치 필요, 비우면 내장 chroma)
enable_file_preview_border = true   # 미리보기 패널에 테두리 — 파일 패널과 구분
zoxide_support = true          # z — zoxide 목록, 들어간 폴더를 zoxide에 기록 (zoxide 설치 필요)
```

---

## § 17. tmux가 가로채는 키와 우회

> 전제: §16 완료, 혼합 키맵 승격 상태. 첫 블록은 평소 쓰는 tmux 안에서, 둘째 블록은 **tmux 밖의 새 터미널 탭**에서
> 주의: 시험용 tmux는 `-L spftest` 별도 서버라 평소 tmux 설정은 바뀌지 않습니다

```bash
tmux show -gv prefix                                      # 지금 prefix — 기본은 C-b
tmux list-keys -T root | grep -vE 'Mouse|Wheel|Click'     # prefix 없이 가로채는 키(bind -n) — 비어 있으면 없음
tmux show -s extended-keys                                # 확장 키 전달 — 기본 off
```

```bash
tmux -L spftest -f /dev/null new-session -s t \; set -g prefix C-a \; bind C-a send-prefix \; bind -n S-Left previous-window   # 시험용 tmux — prefix C-a, shift+left 가로채기
cd ~/work/spf-lab/inbox && spf                            # 시험용 tmux 안에서 실행 (커서는 archive)
```

```text
키: ctrl+a  <esc>  ctrl+a ctrl+a  N  shift+left  H  q
# 한 번은 tmux가 먹음 → 대기 해제 → 두 번째로 넘어가 archive.zip → 패널 추가 → shift+left 는 tmux가 가져감 → H 는 닿음 → 종료
```

```bash
sed -i '' "s/^compress_file = .*/compress_file = ['ctrl+a', 'ctrl+g']        # ctrl+g — tmux prefix가 C-a인 환경용 둘째 키/" ~/.config/superfile/hotkeys.toml   # 둘째 키 추가
spf                                                       # ctrl+g → archive(1).zip 생성 → q
exit                                                      # 시험용 tmux 종료
```

#### 확인한 tmux 키 사례

| 누른 키 (시험용 tmux 안) | tmux 설정 | superfile에 닿았나 | 결과 |
|---|---|---|---|
| `ctrl+a` 한 번 | `prefix C-a` | 아니오 | tmux가 prefix로 먹음 — 압축 안 됨 |
| `ctrl+a` → `ctrl+a` | `prefix C-a` + `bind C-a send-prefix` | 예 (두 번째) | `archive.zip` 생성 |
| `shift+left` | `bind -n S-Left previous-window` | 아니오 | tmux 상태줄 `No previous window`, 패널 초점 그대로 |
| `shift+left` | `bind -n` 해제 후 | 예 | 이전 패널로 초점 이동 |
| `H` | (가로채기 없음) | 예 | 이전 패널로 초점 이동 |
| `ctrl+g` | (가로채기 없음) | 예 | `compress_file` 둘째 키로 `archive(1).zip` 생성 |

---

## 트러블슈팅

| 문제 | 원인 | 해결 |
|---|---|---|
| 단축키를 바꿨는데 반응이 없음 | 이름 오타(모르는 이름은 조용히 버려짐), 다른 파일을 읽는 중 | `?` 도움말에 그 키가 보이는지 확인. `spf --debug-info` 의 `Hotkeys File` 경로 확인. `comm -3` 으로 기본 파일과 이름 비교 (섹션 4) |
| 시작할 때 `missing fields: [...]` | 이름 오타로 진짜 이름이 빠짐, 업그레이드로 새 항목 추가 | 빠진 이름을 기본 파일에서 찾아 손으로 추가. 1.6.0에서 `--fix-` 옵션은 쓰지 않음 |
| `error decoding TOML file : toml: expected character =` | 1.6.0 `--fix-hotkeys` 가 남긴 원본 찌꺼기 | 같은 폴더의 `<파일>.bak-<숫자>` 를 `mv` 로 되돌림 (섹션 4) |
| `toml: key … is already defined` | 1.6.0 `--fix-config-file` 이 남긴 중복 키, 또는 같은 키를 두 번 적음 | `.bak-` 파일로 되돌리거나 중복 줄 정리 |
| `Hotkey value for "…" is invalid : Hotkey list is empty` | 배열의 첫 칸이 빈 문자열 | 첫 칸에 키를 넣음. 오류의 이름은 내부 이름(`ToggleFooter` → `toggle_footer`) |
| `toml: cannot assign boolean to a false` 같은 `[open_with]` 관련 오류 | 새 항목을 파일 끝(`[open_with]` 표 안)에 붙임 | 그 줄을 `[open_with]` 위로 옮김 (섹션 6) |
| 플러그인을 켰는데 동작 안 함, `missing fields: [metadata …]` | 문서 예시대로 `[plugins]` 머리줄을 넣음 | `[plugins]` 줄 삭제, 값만 `true`. exiftool·zoxide 설치는 `--debug-info` 로 확인 |
| 테마를 바꿨는데 기본 테마 색 | 설정 폴더를 옮기며 `theme/` 를 빠뜨림 (빈 `theme/` 만 생김) | 옛 위치의 `theme/` 를 새 설정 폴더로 이동. 로그에 `Could not read user's theme file` 확인 |
| `spf pl` 경로에 `config.toml` 이 없음 | 아직 화면을 한 번도 안 띄움 | `spf` 를 한 번 실행 후 `q` |
| `Q` 로 나와도 셸 폴더 그대로 | 셸 함수 없음, 공식 함수가 Library 경로를 고정(`XDG_STATE_HOME` 설정 시) | `spf pl --lastdir-file` 로 경로를 묻는 함수 사용 (섹션 3) |
| `q` 로 나와도 폴더가 바뀜 | `cd_on_quit = true` | `false` 로 두고 옮길 때만 `Q` |
| 명령 뒤 `# 주석` 까지 붙여넣으니 이상한 오류(`unknown file attribute`, `is not a directory`) | zsh `interactivecomments` 꺼짐 — 주석이 인자로 들어감 | `setopt interactivecomments` (oh-my-zsh는 기본 켬) 또는 주석을 빼고 붙여넣기 |
| 셸 모드 명령이 `status -1 (No output)` | 5초 제한 초과 | 오래 걸리는 작업은 셸에서 직접 |
| 셸 모드에서 zsh 별칭이 안 됨 | `/bin/sh -c` 로 실행 | 전체 명령을 쓰거나 셸에서 실행 |
| `ctrl+d` 를 눌렀더니 휴지통 확인 창 | 기본 키맵의 `delete_items` | 혼합 키맵처럼 `page_down` 에 주고 휴지통은 다른 ctrl 키로 (섹션 5) |
| vim 프리셋에서 `q` 로 안 꺼짐, esc도 안 됨 | vim 프리셋은 `quit = ctrl+c`, `q` = 패널 닫기 | `ctrl+c` 로 종료. 확인 창 취소도 `ctrl+c`·esc |
| vim 프리셋에서 `h` 로 상위 폴더가 안 감 | `parent_directory = ['-', '']` | `-` 사용, 또는 혼합 키맵 |
| 선택 모드에서 `L` 로 선택이 안 됨 | `L` 은 전역 `next_file_panel` (튜토리얼 설명과 다름) | `l`·`enter`·`right` 로 선택 |
| `ctrl+i`·`ctrl+m`·`ctrl+[` 바인딩이 안 먹음 | 터미널에서 탭·엔터·esc와 같은 신호 | 다른 키 사용 (섹션 4) |
| `shift+f` 로 적은 키가 안 먹음 | 대문자는 글자 그대로 `F` 로 적어야 함 | `'F'` |
| tmux 안에서 `ctrl+a` 압축이 안 됨 | tmux prefix `C-a` | `compress_file` 에 둘째 키(`ctrl+g`) 추가 (섹션 7) |
| tmux 안에서 `shift+left` 가 안 먹음 | `bind -n S-Left …` | 글자 키 `H` 사용 또는 tmux 바인딩 해제 |
| 아이콘이 네모·물음표로 깨짐 | 터미널 글꼴이 Nerd Font가 아님 | Nerd Font 적용 또는 `nerdfont = false` |
| 같은 키를 두 동작에 줬는데 한쪽만 동작 | superfile은 겹침을 검사하지 않고 처리 순서로 결정 | 섹션 5의 겹침 검사로 찾아 정리 |
| 휴지통으로 보낸 파일 찾기 | macOS는 `~/.Trash` | Finder 휴지통 또는 `~/.Trash` 에서 `mv` 로 되돌림 |
| `-c`·`--hf` 실행이 바로 끝남 | 지정한 파일이 없음 (`Error: While reading … from argument`) | 경로 확인 (상대 경로는 현재 셸 폴더 기준) |

진단 명령:

```bash
spf pl                                     # 설정·단축키·로그·데이터 경로
spf pl --lastdir-file                      # Q 종료 기록 파일 경로
spf --debug-info                           # 버전·실제로 읽는 설정 경로·환경 변수·외부 도구
grep -n 'ERROR' ~/Library/Application\ Support/superfile/superfile.log  # 로그의 오류 줄 — macOS 기본 위치
```
