# superfile 전문가 — 키 체계를 이해하고 나만의 키맵을 설계·검증하기

> 📋 [명령 복사 시트](examples/demo-sheet.md) · ⚠️ [면책 고지](../../README.md#면책-고지)
>
> 이 폴더는 길리랩 강의 제작 저장소에서 자동으로 내보낸 자료입니다. 더 많은 글은 [길리랩 기술 블로그](https://blog.gillilab.com)에서 볼 수 있습니다.

> 기준 버전: superfile 1.6.0 (2026-10-02 확인) \
> 구성: 릴리스 바이너리 v1.6.0 darwin-arm64 · tmux 3.7c · Neovim · bat 0.26.1 · zoxide 0.10.0 · exiftool 없음 \
> 플랫폼: macOS Apple Silicon 실측 · 설정은 `XDG_CONFIG_HOME=~/.config` 로 옮긴 뒤 진행 · 실습 폴더 `~/work/spf-lab` \
> 실습 결과물: `~/.config/superfile` 에 모인 설정, Neovim 습관과 부딪히지 않는 혼합 키맵(`hotkeys.toml`), 키맵을 본 설정과 분리해 시험하는 `presets/` 폴더, `Q` 로 마지막 폴더에 내려 주는 셸 함수

터미널 파일 관리자 superfile(`spf`)을 **키 체계 단위로** 다룹니다. 설치와 설정 경로를 정리하고, 기본 키가 어떤 원칙으로 짜였는지 익힌 뒤, 단축키 파일(`hotkeys.toml`)의 구조와 문법을 실험으로 확인합니다. 그 위에서 Neovim 사용자용 혼합 키맵을 설계하고, 본 설정과 분리해 검증한 다음 승격합니다.

이 문서는 강의 전 예습 교재이자 강의 후 참조 문서입니다. 영상에서 위치만 가리킨 전체 표와 설계 배경이 여기에 있습니다. 명령과 예상 출력은 격리된 홈 디렉터리에서 v1.6.0 릴리스 바이너리로 실제 실행한 결과입니다. TUI 키 입력은 tmux `send-keys` 로 보내고 화면을 캡처해 확인했습니다(기록: `assets/verification.md`).

> **문서와 실제가 다른 곳**이 이 강의의 핵심 주제 중 하나입니다. 1.6.0 릴리스에 함께 들어 있는 단축키 목록 문서는 변수명이 실제 이름과 다르고(섹션 4), `--fix-hotkeys`·`--fix-config-file` 은 1.6.0에서 파일을 망가뜨립니다(섹션 4·6). 2026-10-02 현재 superfile.dev 의 단축키 목록은 main 브랜치 기준으로 고쳐져 있습니다.

---

## 빠른 시작 체크리스트

- [ ] `spf --version` 이 `superfile version v1.6.0` 인가 (섹션 1)
- [ ] `spf` 를 한 번 띄워 `config.toml`·`hotkeys.toml` 이 생겼는가 — `spf pl` 만으로는 테마만 생김 (섹션 1)
- [ ] `XDG_CONFIG_HOME` 을 설정했다면 `config.toml`·`hotkeys.toml`·**`theme` 폴더** 셋을 모두 옮겼는가 (섹션 1)
- [ ] zsh에서 `# 주석` 이 붙은 명령을 붙여넣는다면 `setopt interactivecomments` 가 켜져 있는가 (oh-my-zsh는 기본으로 켬)
- [ ] `?` 도움말로 지금 읽힌 키를 확인하는 습관이 있는가 (섹션 1·4)
- [ ] 단축키 이름은 superfile.dev 표가 아니라 **기본 `hotkeys.toml` 의 이름**을 기준으로 적었는가 (섹션 4)
- [ ] 1.6.0에서 `--fix-hotkeys`·`--fix-config-file` 을 쓰지 않고 `comm` 으로 빠진 이름을 찾았는가 (섹션 4·6)
- [ ] 새 키맵은 `presets/` 에 두고 `--hf` 로 시험한 뒤 승격했는가 (섹션 5)
- [ ] 겹친 키 검사에서 `ctrl+c`·`enter` 두 줄(입력 키가 덮는 정상 겹침)만 남는가 (섹션 5)
- [ ] `~/.zshrc` 의 `spf` 함수가 `spf pl --lastdir-file` 로 경로를 묻는가 (섹션 3)
- [ ] tmux prefix·`bind -n` 키와 superfile 키가 겹치지 않는가 (섹션 7)

---

## 1. 설치와 설정 경로 지도

### 실행 코드

```bash
brew install superfile        # Homebrew 설치 — 실행 명령은 spf
spf --version                 # superfile version v1.6.0
spf pl                        # 설정·단축키·로그·데이터 경로 출력
spf                           # 첫 실행 — 환영 창은 아무 키로 닫고 q 로 종료
```

설정 폴더를 `~/.config` 로 옮깁니다. 첫 줄 결과가 이미 `~/.config` 를 가리키면 2·3번째 줄은 건너뜁니다.

```bash
echo "XDG_CONFIG_HOME=[$XDG_CONFIG_HOME]"                                       # 비어 있으면 macOS 기본 경로를 씀
echo 'export XDG_CONFIG_HOME="$HOME/.config"' >> ~/.zshrc && source ~/.zshrc    # 설정 폴더를 ~/.config 로
mkdir -p ~/.config/superfile && mv ~/Library/Application\ Support/superfile/{config.toml,hotkeys.toml,theme} ~/.config/superfile/   # 설정 3종만 이동
spf pl                                                                          # 설정은 ~/.config, 로그·데이터는 그대로
```

실습 폴더를 만들고 그 안에서 실행합니다.

```bash
mkdir -p ~/work/spf-lab/{src,docs,logs,inbox} && cd ~/work/spf-lab                                    # 실습 폴더 4개
printf 'def main():\n    print("hello spf")\n\nif __name__ == "__main__":\n    main()\n' > src/app.py   # 미리보기용 코드
printf '# 회의 메모\n\n- superfile 키맵 정리\n- tmux 충돌 확인\n' > docs/notes.md                         # 미리보기용 문서
for i in 1 2 3; do printf '2026-10-02 INFO line %s\n' $i > logs/app-$i.log; done                    # 로그 3개
printf 'id,name\n1,alpha\n2,beta\n' > inbox/data.csv && touch inbox/report-2026.txt inbox/.draft     # .draft 는 숨김 파일
spf                                                                                                   # 실습 폴더에서 실행 — ? 로 도움말, q 로 닫기
```

### CLI 명령·플래그 전체

`spf --help`·`spf pl --help` 출력 기준입니다.

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

> `--hf` 는 도움말 표기입니다. `-hf` 로 적어도 같은 파일을 읽는 것을 `--debug-info` 로 확인했습니다.

### 경로 지도

| 항목 | macOS 기본 | `XDG_CONFIG_HOME=~/.config` 설정 후 (macOS) | 리눅스 기본 (공식 문서) |
|---|---|---|---|
| `config.toml` | `~/Library/Application Support/superfile/` | `~/.config/superfile/` | `~/.config/superfile/` |
| `hotkeys.toml` | `~/Library/Application Support/superfile/` | `~/.config/superfile/` | `~/.config/superfile/` |
| `theme/` (테마 22종) | `~/Library/Application Support/superfile/theme/` | `~/.config/superfile/theme/` | `~/.config/superfile/theme/` |
| `superfile.log` | `~/Library/Application Support/superfile/` | 그대로 (`XDG_STATE_HOME` 을 따름) | `~/.local/state/superfile/` |
| 데이터(`pinned.json`·토글 상태·`themeFileVersion`) | `~/Library/Application Support/superfile/` | 그대로 (`XDG_DATA_HOME` 을 따름) | `~/.local/share/superfile/` |
| `lastdir` (`Q` 종료 기록) | `~/Library/Application Support/superfile/lastdir` | 그대로 (`XDG_STATE_HOME` 을 따름) | `~/.local/state/superfile/lastdir` |

### 화면 구성 요소

| 영역 | 위치 | 하는 일 |
|---|---|---|
| 사이드바 | 왼쪽 | Home · Pinned(`P` 로 고정한 폴더) · Disks(마운트된 디스크). `s` 로 초점, `/` 로 사이드바 검색 |
| 파일 패널 | 가운데 | 현재 폴더 목록. 위에 경로와 검색줄 `(/) Type something`, 아래 테두리에 정렬 기준·모드(Browser/Select)·커서 위치 |
| 미리보기 | 오른쪽 | 커서가 가리키는 파일 내용 또는 폴더 목록. `f` 로 끄고 켬 |
| 프로세스 | 아래 왼쪽 | 복사·압축 같은 작업 진행 상황 (`Copied 2 files`, `Compressed 2 files`). `p` 로 초점 |
| 메타데이터 | 아래 가운데 | 이름·크기·수정 시각·권한·소유자. `m` 으로 초점 |
| 클립보드 | 아래 오른쪽 | `ctrl+c`·`ctrl+x` 로 담은 경로 목록 |
| 프롬프트 | 가운데 모달 | `:` 셸 모드 / `>` spf 모드 (섹션 3) |

아래쪽 세 영역(프로세스·메타데이터·클립보드)은 `F` 로 한꺼번에 숨기고 보입니다.

### 예상 출력

```
superfile version v1.6.0
```

`spf pl` (설정 폴더를 옮기기 전, 홈 경로는 `~` 로 줄임):

```
[Configuration file path]         ~/Library/Application Support/superfile/config.toml
[Hotkeys file path]               ~/Library/Application Support/superfile/hotkeys.toml
[Log file path]                   ~/Library/Application Support/superfile/superfile.log
[Configuration directory path]    ~/Library/Application Support/superfile
[Data directory path]             ~/Library/Application Support/superfile
```

`spf pl` (옮긴 뒤):

```
[Configuration file path]         ~/.config/superfile/config.toml
[Hotkeys file path]               ~/.config/superfile/hotkeys.toml
[Log file path]                   ~/Library/Application Support/superfile/superfile.log
[Configuration directory path]    ~/.config/superfile
[Data directory path]             ~/Library/Application Support/superfile
```

첫 실행 환영 창의 첫 안내(발췌):

```
 ** Very importantly ** If you are a Vim/Nvim user, go to:
 https://superfile.dev/configure/custom-hotkeys/ to change your hotkey settings!
```

`?` 도움말 창 오른쪽 아래에 `1/47` 이 보입니다. 47줄은 `spf` 실행 안내 한 줄과 단축키 46개입니다.

### 더 알아보기 — 왜 그런가

#### spf pl 만으로 생기는 파일과 첫 화면에서 생기는 파일

`spf pl` 은 경로만 출력하는 명령처럼 보이지만, superfile은 어떤 명령이든 시작할 때 기본 설정을 메모리에 올리고 테마 파일을 확인합니다. 그래서 빈 홈에서 `spf pl` 만 실행해도 `theme/` 폴더에 테마 22개와 데이터 폴더의 `themeFileVersion` 이 생깁니다. `config.toml`·`hotkeys.toml` 은 화면(TUI)을 처음 띄울 때 만들어집니다. 경로는 출력되는데 파일이 없다면 아직 한 번도 화면을 띄우지 않은 것입니다.

#### 테마 폴더를 빠뜨렸을 때의 조용한 실패

테마 파일을 다시 깔지 말지는 데이터 폴더의 `themeFileVersion` 으로 판단합니다. 이 값이 이미 `v1.6.0` 이면 테마를 다시 쓰지 않습니다. 설정 폴더만 옮기고 `theme/` 를 두고 오면, 새 설정 폴더에는 빈 `theme/` 만 만들어집니다. 그 상태에서 `theme = "nord"` 처럼 이름을 바꾸면 오류 창 없이 기본 테마(catppuccin-mocha)로 돌아가고, 로그에만 `Could not read user's theme file. Falling back to default theme` 가 남습니다(격리 환경에서 재현). 옮길 때 세 가지를 함께 옮기는 이유입니다.

#### 설정과 상태를 나누는 배치

`XDG_CONFIG_HOME` 만 바꾸면 설정 세 가지만 `~/.config` 로 가고, 로그·고정 폴더·토글 상태·lastdir 은 Library에 남습니다. 설정은 dotfiles 저장소로 여러 기기에서 같게 유지할 대상이고, 나머지는 기기마다 다른 상태라 저장소에 넣지 않는 것이 맞습니다. `XDG_STATE_HOME` 까지 옮기면 lastdir 위치도 바뀌는데, 공식 cd 연동 스크립트는 macOS에서 Library 경로를 고정해 두어 이 파일을 찾지 못합니다(섹션 3에서 재현).

#### XDG_CONFIG_HOME 이 다른 도구에 주는 영향

`XDG_CONFIG_HOME` 은 superfile 전용 변수가 아닙니다. 같은 규칙을 따르는 다른 도구도 이 값을 읽습니다. 이미 다른 값을 쓰고 있다면 덮어쓰지 말고, 첫 줄의 `echo` 결과를 보고 판단합니다. 오늘 강의는 비어 있는 상태에서 `~/.config` 로 정하는 경우만 다룹니다.

#### 환영 창의 Vim 사용자 안내

첫 실행 환영 창은 Vim/Nvim 사용자에게 단축키 설정 문서부터 보라고 안내합니다. 기본 키맵이 `j`·`k`·`h`·`l` 이동은 Vim과 같지만, `ctrl+d`(휴지통)·`ctrl+w`(붙여넣기)·`esc`(종료)처럼 Neovim 습관과 정면으로 부딪히는 키가 있기 때문입니다. 섹션 5의 혼합 키맵이 이 지점을 다룹니다.

---

## 2. 기본 키 체계

### 실행 키 순서

강의에서 누른 순서 그대로입니다. 섹션 1의 실습 폴더에서 `spf` 를 띄운 상태에서 시작합니다.

```text
# 이동과 패널 — 시작 커서는 docs
j j l h            # logs 로 내려가 들어갔다가 나옴 (나오면 커서는 logs)
n w                # 새 패널(홈 폴더에서 열림) → 닫기
N tab H w          # 지금 폴더로 패널 하나 더 → 다음 → 이전 → 닫기
f f                # 미리보기 끄기 → 켜기
s j s              # 사이드바 초점 → 아래로 → 파일 패널로
p p m m            # 프로세스 → 돌아옴, 메타데이터 → 돌아옴

# 선택 모드와 파일 조작 — inbox 에서
k l                          # inbox 로 들어감
ctrl+n archive/ <enter>      # 이름 끝 / → 폴더 생성
j j ctrl+r -q3 <enter>       # report-2026.txt → report-2026-q3.txt (커서가 확장자 앞에 놓임)
v K l ctrl+c                 # 선택 모드 → report 선택하며 위로 → data.csv 선택 → 복사 2개
v k l ctrl+v h               # 일반 모드 → archive 로 들어가 붙여넣기 → 나옴
ctrl+a j ctrl+e              # archive 압축 → archive.zip 으로 → 풀기(archive(1) 폴더)
ctrl+d <enter>               # archive(1) 를 휴지통으로
D <enter>                    # archive.zip 영구 삭제

# 검색·정렬·숨김 파일
/ rep <enter>      # 이름 필터 — 엔터 후에도 필터 유지
/ <esc>            # 필터 해제
o j <enter> R      # 정렬 메뉴 → 크기순 → 순서 뒤집기
. .                # 숨김 파일 보이기 → 숨기기 (상태가 다음 실행에도 이어짐)
F F q              # 아래쪽 패널 숨기기 → 보이기 → 종료
```

### 기본 키 전체

기본 `hotkeys.toml`(v1.6.0) 46개 항목 전부입니다. 계층은 파일의 세 구역입니다(섹션 4). 설명은 `?` 도움말 문구를 옮겼습니다.

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

### 예상 결과

`inbox` 의 마지막 상태(숨김 파일 포함):

```
inbox/
├── .draft
├── archive/
│   ├── data.csv
│   └── report-2026-q3.txt
├── data.csv
└── report-2026-q3.txt
```

`ctrl+d` 로 보낸 `archive(1)` 은 `~/.Trash/archive(1)` 에 있습니다. 확인 창 문구:

```
Are you sure you want to move this to trash can
This operation will move file or directory to trash can.
(enter) Confirm             (q) Cancel
```

```
Are you sure you want to completely delete
This operation cannot be undone and your data will be completely lost.
(enter) Confirm             (q) Cancel
```

정렬 메뉴는 `Name` · `Size` · `Date Modified` · `Type` · `Natural` 다섯 가지입니다(공식 튜토리얼은 세 가지만 적고 있습니다).

### 더 알아보기 — 왜 그런가

#### 파일을 바꾸는 키는 ctrl이라는 기본 원칙

공식 문서는 기본 키맵의 설계 개념을 두 줄로 설명합니다. 파일을 바꾸는 키는 모두 `ctrl+키` 를 쓰고("ctrl을 누르지 않는 한 파일은 안전하다"), 파일을 바꾸지 않는 키는 단어 첫 글자를 씁니다. 만들기(`ctrl+n`), 이름 바꾸기(`ctrl+r`), 복사·잘라내기·붙여넣기(`ctrl+c`·`ctrl+x`·`ctrl+v`), 압축·풀기(`ctrl+a`·`ctrl+e`), 휴지통(`ctrl+d`)이 이 약속을 따릅니다. 첫 글자 쪽은 `f`(file preview), `s`(sidebar), `p`(process), `m`(metadata), `o`(options), `n`(new), `w`(window 닫기) 같은 식입니다.

#### 영구 삭제 D 가 ctrl 원칙의 예외라는 점

가장 위험한 동작인 영구 삭제가 shift 조합인 대문자 `D` 입니다. 실수로 눌러도 "completely delete" 확인 창이 한 번 막아 주지만, 원칙만 놓고 보면 예외입니다. `delete` 키(휴지통)도 ctrl이 없습니다. macOS 노트북 키보드의 delete 키는 터미널에 backspace로 들어가므로(상위 폴더), 휴지통으로 가는 `delete` 는 fn+delete 쪽입니다. 키맵을 직접 짤 때 이 예외를 그대로 둘지 정해야 합니다(섹션 5).

#### n 과 N — 새 패널이 열리는 위치

`n` 은 새 패널을 항상 홈 폴더에서 엽니다. 지금 보고 있는 폴더를 옆에 하나 더 띄우려면 `N`(split) 입니다. 두 폴더를 나란히 놓고 복사·이동할 때는 `N` 으로 띄운 뒤 한쪽에서 폴더를 옮기는 것이 빠릅니다. spf 모드의 `split` 명령도 `N` 과 같습니다(섹션 3).

#### 선택 모드에서 L 은 선택이 아님

공식 튜토리얼은 선택 모드에서 `enter` 또는 `L`(shift+l)로 항목을 고른다고 적고 있습니다. 1.6.0에서 `L` 은 전역 키인 다음 패널 이동이라 선택되지 않습니다. 선택은 `confirm` 키인 `enter`·`right`·소문자 `l` 입니다(격리 환경에서 `L` → 미선택, `l` → 선택 확인). `J`·`K` 는 "위·아래 전부 선택"이 아니라 현재 항목을 고르고 한 칸 움직이는 동작입니다.

#### 휴지통의 실제 위치와 외장 디스크

macOS에서 `ctrl+d` 는 항목을 `~/.Trash` 로 옮깁니다(격리 홈의 `.Trash` 로 들어가는 것 확인). 공식 튜토리얼은 외장 디스크에서는 휴지통 대신 바로 지워진다고 안내합니다. 이 동작은 이번 검증에서 확인하지 않았습니다. 외장 디스크에서 지울 때는 `D` 와 같다고 생각하고 다루는 편이 안전합니다.

#### 압축·풀기와 이름 겹침 처리

`ctrl+a` 는 커서 항목(선택 모드면 선택 항목)을 `<이름>.zip` 으로 만들고, `ctrl+e` 는 zip을 같은 이름 폴더로 풉니다. 같은 이름이 있으면 덮어쓰지 않고 `archive(1)` 처럼 괄호 숫자를 붙입니다. 실습에서 `archive.zip` 을 풀면 `archive(1)/archive/…` 구조가 됩니다.

#### 다음 실행에도 이어지는 두 토글

`.`(숨김 파일)과 `F`(아래쪽 패널)는 데이터 폴더의 `toggleDotFile`·`toggleFooter` 파일에 `true`/`false` 로 저장됩니다. 녹화 전에 숨김 파일을 켜 둔 채 끝내면 다음 실행에도 켜진 채로 시작합니다. 정렬 기준은 저장되지 않으므로 기본 정렬은 `default_sort_type` 으로 정합니다(섹션 6).

#### 엔터로 파일을 열면 일어나는 일

폴더에서 엔터는 들어가기지만, 파일에서 엔터는 macOS `open` 명령으로 기본 앱을 엽니다(소스 기준 — 강의에서는 실행하지 않았습니다). 터미널 안에서 편집하려면 `e` 를 씁니다. 확장자별로 열 프로그램을 바꾸려면 `config.toml` 맨 끝의 `[open_with]` 표를 씁니다(섹션 6).

---

## 3. 프롬프트와 셸 연동

### 실행 코드

`~/work/spf-lab` 에서 `spf` 를 띄우고 프롬프트를 엽니다.

```text
:  mkdir -p backup && cp logs/*.log backup/ && ls backup    # 셸 모드 — 패널 폴더에서 /bin/sh 로 실행, 출력이 보임
   ls no-such-file                                          # 실패 — Error : status 1 + 오류 출력, 프롬프트는 열린 채
>                                                           # 입력란이 비어 있을 때 > → spf 모드
   split                                                    # 지금 폴더로 패널 하나 더
   cd docs                                                  # 현재 패널을 docs 로 (상대 경로 가능)
   cd ${HOME}/work/spf-lab/logs                             # 환경 변수 치환
   open $(dirname $(pwd))                                   # 명령 치환 — $(pwd) 는 패널 폴더에서 실행 → spf-lab 을 새 패널로
<esc> w w q                                                 # 프롬프트 닫기 → 패널 두 개 닫기 → 종료
```

`Q` 로 마지막 폴더에 내려 주는 셸 함수입니다.

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
spf → j l Q → pwd       # docs 로 들어가 Q → 셸이 docs 로 이동
spf → h q → pwd         # 상위로 올라가 q → 셸은 docs 그대로
```

zoxide 연동:

```bash
sed -i '' 's/^zoxide_support = false/zoxide_support = true          # z — zoxide 목록, 들어간 폴더를 zoxide에 기록 (zoxide 설치 필요)/' ~/.config/superfile/config.toml   # zoxide 연동 켜기
zoxide add ~/work/spf-lab/logs                                                               # 데모용으로 logs 를 기록
spf ~/work/spf-lab                                                                           # z → log → enter → q
spf logs                                                                                     # 없는 경로 → zoxide 가 찾은 logs 로 열림 → q
```

### spf 모드 명령 전체

| 명령 | 기능 | 예 |
|---|---|---|
| `split` | 지금 패널의 폴더로 새 패널 열기 (인자 없음) | `split` |
| `cd <경로>` | 현재 패널의 폴더 바꾸기 | `cd docs` · `cd ${HOME}/work` · `cd ~/work` |
| `open <경로>` | 지정한 경로로 새 패널 열기 | `open $(dirname $(pwd))` |

입력란이 비어 있을 때 `:` 를 치면 셸 모드, `>` 를 치면 spf 모드로 바뀝니다. 두 모드 모두 `enter` 로 실행, `esc`·`ctrl+c`(입력 취소 키)로 닫습니다.

### 셸 모드 동작 요약

| 항목 | 1.6.0 동작 (확인 방법) |
|---|---|
| 실행 셸 | `/bin/sh -c "<명령>"` (소스) — zsh 별칭·함수는 쓸 수 없음 |
| 실행 위치 | 초점이 있는 파일 패널의 폴더 (`ls backup` 결과로 확인) |
| 결과 | `Success`/`Error` + 종료 코드 + 출력 (실행 확인 — 공식 튜토리얼의 "출력을 받을 수 없다"는 옛 설명) |
| 시간 제한 | 5초 — `sleep 6; echo done` 은 `status -1 (No output)` (실행 확인) |

### 예상 출력

```
Success : Command exited with status 0, Output:
app-1.log
app-2.log
app-3.log
```

```
Error : Command exited with status 1, Output:
ls: no-such-file: No such file or directory
```

spf 모드 결과줄: `Success : Panel successfully split` · `Success : Panel directory changed` · `Success : New panel opened`

zoxide 창:

```
╭─┤  Zoxide Navigation ├──────────
│ log
├─────────────────────────────────
│    4.0 | ~/work/spf-lab/logs
```

`Q` 뒤의 `pwd`: `~/work/spf-lab/docs` (홈 경로는 줄임). lastdir 파일 내용은 `cd '<경로>'` 한 줄이고, 함수가 실행한 뒤 0바이트로 비워집니다.

### 더 알아보기 — 왜 그런가

#### 셸 모드가 대화형 셸이 아닌 이유

셸 모드는 입력한 문자열을 `/bin/sh -c` 로 넘기고 출력을 모아 결과 창에 보여 주는 구조입니다. 터미널을 넘겨주지 않으므로 `vim`·`less` 같은 대화형 명령은 맞지 않고, zsh 설정(별칭·함수·옵션)도 적용되지 않습니다. 5초 제한이 있어 오래 걸리는 작업은 끊깁니다. 짧은 파일 정리(`mkdir`·`cp`·`mv`·`git mv` 등)를 패널 폴더에서 바로 실행하는 용도로 씁니다. `shell_close_on_success = true` 로 두면 성공한 뒤 프롬프트가 자동으로 닫힙니다.

#### spf 모드의 치환은 패널 폴더에서 실행

spf 모드는 `${VAR}` 환경 변수, `$()` 명령 치환, `~` 홈 치환을 지원합니다. 명령 치환은 패널 폴더에서 실행되므로 `$(pwd)` 는 셸의 현재 폴더가 아니라 패널 경로입니다. `logs` 패널에서 `open $(dirname $(pwd))` 가 `spf-lab` 을 연 것으로 확인했습니다. `$(git rev-parse --show-toplevel)` 처럼 저장소 루트로 바로 가는 용도로 쓸 수 있습니다.

#### 프로그램이 셸의 폴더를 바꿀 수 없는 구조

자식 프로세스는 부모 셸의 현재 폴더를 바꿀 수 없습니다. superfile은 `Q`(또는 `cd_on_quit = true` 일 때의 `q`)로 종료하면서 lastdir 파일에 `cd '<마지막 폴더>'` 한 줄을 쓰고, 셸 함수가 그 파일을 `.` 으로 실행해 폴더를 옮깁니다. 폴더 이름의 작은따옴표는 `'\''` 로 바꿔 적어 깨지지 않게 합니다(소스). 그래서 셸 함수가 없으면 `Q` 는 `q` 와 똑같이 끝납니다.

#### 공식 cd_on_quit 함수가 macOS에서 놓치는 경우

공식 저장소의 `cd_on_quit/cd_on_quit.sh` 는 macOS에서 `SPF_LAST_DIR="$HOME/Library/Application Support/superfile/lastdir"` 로 경로를 고정합니다. superfile은 lastdir 을 `XDG_STATE_HOME` 기준으로 쓰므로, 이 변수를 설정한 macOS에서는 함수가 파일을 찾지 못하고 `Q` 로 나와도 폴더가 그대로입니다. `XDG_STATE_HOME=~/.local/state` 를 준 격리 환경에서 공식 함수는 cd 하지 않았고, `spf pl --lastdir-file` 로 경로를 묻는 위 함수는 cd 했습니다. 공식 함수는 다 쓴 파일을 `rm` 으로 지우지만, 위 함수는 `: >` 로 비웁니다. `-s`(크기가 0보다 큼) 검사로 같은 효과를 내면서 파일 삭제를 피합니다.

#### cd_on_quit 을 켜지 않는 판단

`cd_on_quit = true` 로 두면 소문자 `q` 로 나와도 마지막 폴더로 옮겨집니다(격리 환경 확인). 파일을 둘러보기만 하고 원래 자리로 돌아가고 싶을 때도 셸이 옮겨지므로, 옮길지 말지를 종료 키로 고르는 편(`Q` = 옮김, `q` = 그대로)이 실수가 적습니다.

#### zoxide가 기록되는 시점과 없는 경로 인자

`zoxide_support = true` 이고 `zoxide` 가 PATH에 있으면, superfile 안에서 폴더에 들어갈 때(`l`·엔터, `cd` 명령, 사이드바 선택)마다 그 폴더를 zoxide에 추가합니다. `z` 창은 입력한 글자로 목록을 거르고, 앞의 숫자는 zoxide 점수입니다. 실행 인자로 존재하지 않는 경로를 주면(`spf logs`) superfile이 zoxide에 질의해 가장 점수 높은 폴더를 엽니다. 연동이 꺼져 있으면 홈 폴더로 열립니다(소스). 녹화 중에는 실제 zoxide 목록에 다른 프로젝트 경로가 보일 수 있으니 미리 확인합니다.

---

## 4. hotkeys.toml 해부

### 실행 코드

```bash
spf pl | grep -i 'hotkeys file'                                                       # 단축키 파일 위치
grep -cE '^[a-z_]+ *=' ~/.config/superfile/hotkeys.toml                                # 항목 수 — 46
grep -nE 'Global hotkeys|Typing hotkeys|Mode-Specific Hotkeys' ~/.config/superfile/hotkeys.toml   # 세 구역 시작 줄
nvim ~/.config/superfile/hotkeys.toml                                                  # 구조 읽기 — :q 로 닫음
```

키 문법 실험 (복사본에서):

```bash
cd ~/.config/superfile && mkdir -p presets && cp hotkeys.toml presets/lab.toml                         # 실험용 복사본
sed -i '' "s/^toggle_footer = .*/toggle_footer = ['', 'F']/" presets/lab.toml && spf --hf presets/lab.toml   # 첫 칸 비우기 → 시작 거부
sed -i '' "s/^toggle_footer = .*/toggle_footer = ['shift+f', 'ctrl+i']/" presets/lab.toml && spf --hf presets/lab.toml   # shift+ 표기와 ctrl+i — F·ctrl+i 모두 무반응 → q
```

문서 변수명 함정 재현과 복구:

```bash
cp hotkeys.toml presets/typo.toml && sed -i '' "s/^focus_on_sidebar = .*/focus_on_side_bar = ['b', '']/" presets/typo.toml   # 문서 표기로 바꿔 재현
spf --hf presets/typo.toml                       # missing fields: [focus_on_sidebar] — b 무반응, s 는 기본값으로 동작 → q
spf --hf presets/typo.toml --fix-hotkeys         # 빠진 키를 채웠다는 메시지 → q
spf --hf presets/typo.toml                       # error decoding TOML file — 시작 거부
mv presets/typo.toml presets/typo.broken.toml && mv presets/typo.toml.bak-* presets/typo.toml   # 백업으로 되돌림 (망가진 파일은 보관)
comm -3 <(grep -oE '^[a-z_]+' hotkeys.toml | sort) <(grep -oE '^[a-z_]+' presets/typo.toml | sort)   # 왼쪽=빠진 이름, 오른쪽=모르는 이름
sed -i '' 's/^focus_on_side_bar = /focus_on_sidebar = /' presets/typo.toml && spf --hf presets/typo.toml   # 진짜 이름으로 → b 로 사이드바
```

### 3계층 구조

| 구역 (파일 주석) | 항목 수 | 규칙 (파일 주석 원문 요지) | 실제 처리 |
|---|---|---|---|
| Global hotkeys | 39 | 서로 겹치면 안 됨 (`These hotkeys should be unique.`) | 입력 창이 없을 때 모드와 무관하게 먼저 검사 |
| Typing hotkeys | 2 | 다른 모든 키를 덮어씀 (`can override all hotkeys`) | 이름 입력·검색·프롬프트·확인 창이 열려 있을 때 가장 먼저 검사 |
| Mode-Specific Hotkeys | 5 (일반 2 · 선택 3) | 다른 모드 키와는 겹쳐도 되지만 전역 키와는 안 됨 | 전역 키에 걸리지 않은 키만 모드별로 검사 |

각 구역 안에는 `#--` 로 시작하는 작은 제목(Basic Actions · Navigation · File Panel Controls · Focus Manipulation · File/Dir Creation/Renaming · Main File Operations · Archive Manipulation · Editor Actions · Other Actions, 모드 구역의 Normal Mode Actions · Selection Mode Actions)이 있습니다. 키를 바꿀 때는 이 묶음 단위로 읽으면 빠짐없이 훑을 수 있습니다. `?` 도움말의 묶음(General · Panel navigation · Panel movement · File operations)은 파일 구역과 다르게 기능별로 다시 나뉘어 있습니다.

### 키 문법 규칙

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

### 문서 표기와 실제 이름

1.6.0 태그의 `website/src/content/docs/list/hotkey-list.mdx`(릴리스 시점 superfile.dev 단축키 목록) 변수명 열과 실제 이름(`src/internal/common/config_type.go` 의 `HotkeysType` toml 태그 = 기본 `hotkeys.toml`)의 대응입니다. main 브랜치(2026-10-02 현재 superfile.dev)에서는 고쳐졌습니다.

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

1.6.0 문서 표에 아예 없던 항목은 `page_up`·`page_down`·`toggle_reverse_sort`·`copy_present_working_directory` 네 개이고, `confirm` 의 `l` 도 빠져 있었습니다.

### 예상 출력

```
11:#                                Global hotkeys                               #
74:#                                Typing hotkeys                               #
83:#                            Mode-Specific Hotkeys                            #
```

```
■ ERROR: Hotkey value for "ToggleFooter" is invalid : Hotkey list is empty; at least one key binding is required.
```

```
Error ┃ missing fields: [focus_on_sidebar]
To add missing fields to hotkeys file automatically run superfile with the --fix-hotkeys flag `spf --fix-hotkeys`
```

```
Error ┃ config file had issues. Its fixed successfully. Original backed up to : presets/typo.toml.bak-<숫자>
```

```
Error ┃ error decoding TOML file : toml: expected character =
```

`comm -3` (탭으로 들여쓴 줄 = 오른쪽 파일에만 있는 이름):

```
	focus_on_side_bar
focus_on_sidebar
```

### 더 알아보기 — 왜 그런가

#### 세 구역 규칙은 검사가 아니라 처리 순서

superfile은 키 겹침을 검사하지 않습니다. 겹치게 적어도 경고 없이 실행되고, 키를 처리하는 순서가 결과를 정합니다. 확인 창·이름 입력·검색·프롬프트처럼 입력 중인 창이 있으면 그 창의 키(입력 키 포함)가 먼저입니다. 그다음 종료 키(`quit`·`cd_quit`), 그다음 전역 키 목록, 마지막으로 현재 모드(일반·선택)의 키입니다. 전역 키 `toggle_footer` 에 `h` 를 준 실험에서 `h` 는 상위 폴더로 가지 않고 푸터만 토글했습니다. 같은 동작의 `backspace`·`left` 는 그대로 상위 폴더로 갔습니다. 겹침을 막는 책임이 키맵을 짜는 사람에게 있으므로 섹션 5의 겹침 검사를 씁니다.

#### 입력 키가 모든 키를 덮는다는 말의 뜻

입력 창이 열려 있을 때는 입력 키 두 개만 특별 취급하고 나머지는 글자로 들어갑니다. 기본값의 `ctrl+c` 는 전역에서는 복사지만, 새 파일 이름 창에서는 취소입니다. 반대로 입력 키에 글자를 주면 그 글자를 입력할 수 없게 됩니다. `cancel_typing = ['q', 'esc']` 로 바꾼 실험에서 새 파일 이름에 `q` 를 치는 순간 창이 닫혔습니다. 입력 키에는 글자를 주지 않습니다.

#### ctrl+m·ctrl+i·ctrl+[·ctrl+? 를 쓰지 않는 이유

전통적인 터미널은 ctrl과 글자를 누르면 그 글자 코드의 위 비트를 지운 제어 문자 하나를 보냅니다. `ctrl+m` 은 13(캐리지 리턴, 엔터), `ctrl+i` 는 9(탭), `ctrl+[` 는 27(esc)이고, `ctrl+?` 는 127(DEL)로 backspace와 같습니다. 프로그램 쪽에서는 이 둘을 구별할 방법이 없으므로 `ctrl+i` 로 묶은 키는 영영 오지 않고, 누르면 탭으로 처리됩니다. tmux 안에서 네 키를 보내 각각 엔터·탭·esc·backspace로 동작하는 것을 확인했습니다. 확장 키 프로토콜을 쓰는 터미널·tmux 설정에서는 구별될 여지가 있지만, 이번 검증 범위 밖이므로 쓰지 않는 것을 기본으로 합니다.

#### 모르는 이름이 조용히 버려지는 구조

단축키 파일은 46칸짜리 정해진 틀(`HotkeysType`)에 부어 넣는 방식으로 읽습니다. 틀에 없는 이름은 오류 없이 버려지고, 틀의 이름 중 파일에 없는 것만 `missing fields` 로 경고합니다. 빠진 칸은 내장 기본값으로 채워집니다. 그래서 `focus_on_side_bar = ['b', '']` 는 경고에 자기 이름이 나오지 않고 `focus_on_sidebar` 가 빠졌다는 경고로만 드러나며, `b` 는 무반응이고 `s` 는 기본값으로 계속 동작합니다. 오타를 찾을 때는 경고 문구보다 `comm -3` 으로 기본 파일과 이름 목록을 직접 비교하는 편이 정확합니다. 기본 파일이 없다면 1.6.0 태그의 원본(`https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/hotkeys.toml`)을 받아 비교합니다.

#### 1.6.0 의 --fix-hotkeys 가 파일을 망가뜨리는 원리

`--fix-hotkeys` 는 먼저 원본을 `<파일>.bak-<숫자>` 로 복사한 뒤, 메모리의 값을 TOML로 다시 써서 원본 파일 **앞에서부터 덮어씁니다**. 1.6.0은 파일 길이를 줄이지 않으므로, 새 내용이 원본보다 짧으면 원본의 뒷부분이 그대로 남습니다. 기본 파일은 주석이 많아 다시 쓴 내용이 항상 더 짧고, 남은 뒷부분(`ys can override all hotkeys.` 같은 잘린 줄과 중복 키)이 다음 실행에서 `error decoding TOML file` 을 냅니다. 같은 함수를 쓰는 `--fix-config-file` 도 똑같습니다(섹션 6). main 브랜치에는 덮어쓰기 전에 파일을 비우는 수정이 들어가 있지만 2026-10-02 현재 릴리스되지 않았습니다. 성공했더라도 모든 주석(직접 단 한글 주석 포함)과 모르는 이름이 사라집니다. 복구는 `.bak-` 파일을 `mv` 로 되돌리는 것입니다.

#### 문서와 실제 이름이 어긋난 배경

1.6.0 시점의 단축키 목록 문서는 예전 이름(`copy_single_item`·`file_panel_select_mode_item_copy` 처럼 모드별로 나뉘던 이름)과 단수형을 그대로 두고 있었습니다. 실제로는 복사·잘라내기·삭제가 모드와 무관한 한 이름(`copy_items` 등)으로 합쳐졌습니다. 문서는 이후 main에서 고쳐졌지만, 1.6.0 문서를 옮겨 적은 블로그·답변은 옛 이름을 그대로 담고 있습니다. 기준은 언제나 설치한 버전의 기본 `hotkeys.toml` 과 `?` 도움말입니다.

---

## 5. 나만의 키맵 설계

### 실행 코드

vim 프리셋 시험:

```bash
curl -fsSL https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/vimHotkeys.toml -o presets/vim.toml   # 1.6.0 vim 프리셋
diff <(grep -E '^[a-z_]+ *=' hotkeys.toml | sort) <(grep -E '^[a-z_]+ *=' presets/vim.toml | sed 's/ *#.*//' | sort) | grep '^>'   # 바뀐 키만
spf --hf presets/vim.toml ~/work/spf-lab                                                       # 본 설정은 그대로 두고 시험
```

```text
j <enter> h l - <esc> ctrl+s ctrl+s d <esc> ctrl+c
# docs 로 들어감 → h·l 무반응 → - 로 상위 → esc 로는 안 꺼짐 → ctrl+s 사이드바 왕복 → d 휴지통 창 → esc 취소 → ctrl+c 종료
```

혼합 키맵 작성:

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

겹침 검사·분리 시험·승격:

```bash
grep -E '^[a-z_]+ *=' presets/nvim-mix.toml | sed -E "s/ *#.*//; s/ *= *\[/ /; s/\]//; s/[',]/ /g" | awk '{for(i=2;i<=NF;i++) k[$i]=k[$i]" "$1} END{for(x in k) if (split(k[x],a," ")>1) print x " ->" k[x]}' | sort   # 겹친 키 목록
spf --hf presets/nvim-mix.toml --debug-info | grep -i 'hotkeys file'   # 이 실행이 읽을 단축키 파일
spf --hf presets/nvim-mix.toml ~/work/spf-lab                           # 혼합 키맵으로 시험 — ctrl+d ctrl+u esc ctrl+t esc q
cp config.toml presets/config-lab.toml && sed -i '' 's/^sidebar_width = .*/sidebar_width = 0/' presets/config-lab.toml   # 실험용 설정
spf -c presets/config-lab.toml --hf presets/nvim-mix.toml ~/work/spf-lab                                                   # 사이드바 없는 화면 → q
cp hotkeys.toml presets/default.toml && cp presets/nvim-mix.toml hotkeys.toml                                              # 기본값 보관 후 승격
```

### vim 프리셋과 기본값의 차이 전체

1.6.0 `vimHotkeys.toml` 과 기본 `hotkeys.toml` 의 46개 항목 중 값이 다른 23개입니다(위 `diff` 출력).

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

### 혼합 키맵 — 기본값과 다른 줄

강의 끝 상태의 `~/.config/superfile/hotkeys.toml` 이 기본값과 다른 줄 전부입니다(섹션 7의 `ctrl+g` 포함). 나머지 40줄은 기본값 그대로입니다.

```toml
# ~/.config/superfile/hotkeys.toml — 기본값에서 바꾼 줄만
quit = ['q', '']                       # esc 제거 — Neovim 습관으로 esc를 눌러도 종료되지 않음
page_down = ['ctrl+d', 'pgdown']        # Neovim 반 페이지 아래 습관 — 파일을 바꾸지 않는 동작에 배정
page_up = ['ctrl+u', 'pgup']            # ctrl+d와 짝 — 반 페이지 위
delete_items = ['ctrl+t', 'delete']     # ctrl+d를 비워 주고 휴지통(trash)의 t — ctrl 원칙 유지
paste_items = ['ctrl+v', '']           # ctrl+w 제거 — Neovim 창 이동 습관이 붙여넣기로 이어지지 않게
compress_file = ['ctrl+a', 'ctrl+g']        # ctrl+g — tmux prefix가 C-a인 환경용 둘째 키 (섹션 7)
```

### 예상 출력

겹친 키 검사 (혼합 키맵):

```
ctrl+c -> copy_items cancel_typing
enter -> confirm confirm_typing
```

같은 검사를 기본 키맵에 돌리면 `esc -> quit cancel_typing` 이 한 줄 더 나오고, vim 프리셋에서는 `enter -> confirm confirm_typing` 한 줄만 나옵니다. 세 줄 모두 입력 키가 덮는 정상 겹침입니다.

```
Hotkeys File        : presets/nvim-mix.toml
```

혼합 키맵 시험: `ctrl+d` → 휴지통 창 없이 목록 이동 · `esc` → 종료되지 않음 · `ctrl+t` → `move this to trash can` 확인 창.

### 더 알아보기 — 왜 그런가

#### vim 프리셋이 Neovim 사용자에게 위험한 지점

vim 프리셋은 이동을 Vim에 맞추는 대신 파일 조작을 한 글자(`y`·`x`·`p`·`d`·`a`·`r`)로 옮겼습니다. Neovim에서 한 글자 명령이 손에 붙은 사람일수록 목록을 보다가 무심코 `d` 나 `p` 를 누를 가능성이 큽니다. 휴지통은 확인 창이 막아 주지만, 붙여넣기와 잘라내기는 확인 없이 실행됩니다. 또 `ctrl+d` 가 메타데이터 초점, `h` 가 무반응, esc가 종료가 아닌 점까지 Neovim과 다릅니다. "vim 같다"는 인상과 실제 키가 엇갈리는 지점이 많아, 그대로 쓰기보다 비교 기준으로 두는 편이 낫습니다.

#### 혼합 키맵의 설계 기준

기본 키맵은 이미 `j`·`k`·`h`·`l` 로 움직이므로 이동은 거의 손댈 것이 없습니다. 고친 것은 Neovim 습관이 **파일을 바꾸거나 프로그램을 끄는 쪽으로 새는** 다섯 줄입니다. `ctrl+d`·`ctrl+u` 는 반 페이지 이동이라는 무해한 동작에 주고, 비워진 휴지통은 ctrl 원칙을 지키는 `ctrl+t` 로 옮겼습니다. `ctrl+t` 는 기본 키맵에서 비어 있고, tmux 기본 설정의 prefix 없는 키 목록(`list-keys -T root`)에도 없습니다. esc 연타로 꺼지지 않게 종료에서 esc를 빼고, 창 이동 습관인 `ctrl+w` 를 붙여넣기에서 뺐습니다. 영구 삭제 `D` 는 확인 창을 믿고 그대로 두었습니다.

#### 겹침 검사 한 줄이 하는 일

`grep` 으로 항목 줄만 뽑고, `sed` 로 주석·`=`·대괄호·따옴표·쉼표를 공백으로 바꿔 "이름 키1 키2" 꼴로 만든 뒤, `awk` 가 키마다 이름을 모아 두 개 이상인 키만 출력합니다. 빈 문자열 칸은 사라지므로 걸리지 않습니다. 결과에 입력 키(`confirm_typing`·`cancel_typing`)와의 겹침만 남으면 정상입니다. 전역 키끼리 겹친 줄이 나오면 처리 순서상 한쪽이 영영 동작하지 않으므로 고칩니다.

#### 옆 파일로 시험하고 승격하는 절차

본 파일을 직접 고치다 문법 실수를 하면 superfile이 시작을 거부합니다. `presets/` 에 실험 파일을 두고 `--hf`(단축키)·`-c`(설정)로 끼워 시험하면, 실패해도 옵션을 빼는 것으로 끝납니다. `--debug-info` 는 옵션을 반영한 실제 경로를 출력하므로 "지금 어느 파일을 읽는가"를 실행 전에 확인할 수 있습니다. 승격할 때는 기본값을 `presets/default.toml` 로 남기고 복사합니다. 되돌리기는 `cp presets/default.toml hotkeys.toml` 한 줄입니다. 단, `-c` 로 설정 파일을 바꿔도 테마는 본 설정 폴더의 `theme/` 에서 읽습니다(소스 — 테마 폴더 경로가 설정 폴더 기준으로 고정).

---

## 6. config.toml 실무 설정

### 실행 코드

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

```bash
sed -i '' \
  -e 's|^code_previewer = .*|code_previewer = "bat"          # 코드 미리보기를 bat으로 (bat 설치 필요, 비우면 내장 chroma)|' \
  -e 's|^enable_file_preview_border = .*|enable_file_preview_border = true   # 미리보기 패널에 테두리 — 파일 패널과 구분|' \
  config.toml
spf --debug-info | grep -A8 '^Dependencies'   # bat·zoxide·exiftool 확인
spf ~/work/spf-lab/src                        # 테두리 있는 bat 미리보기 → q
```

빠진 항목을 손으로 채우기 (1.6.0에서 `--fix-config-file` 대신):

```bash
grep -v '^shell_close_on_success' config.toml > presets/config-old.toml                    # 한 항목이 빠진 옛 설정 재현
comm -13 <(grep -oE '^[a-z0-9_]+' presets/config-old.toml | sort -u) <(grep -oE '^[a-z0-9_]+' presets/config-before.toml | sort -u)   # 빠진 이름
perl -0pi -e 's/^\[open_with\]/shell_close_on_success = false   # 셸 명령 성공 후 프롬프트 자동 닫기\n\n[open_with]/m' presets/config-old.toml   # [open_with] 위에 넣기
spf -c presets/config-old.toml ~/work/spf-lab                                              # 경고 없이 실행 → q
```

### config.toml 항목 전체

기본 `config.toml`(v1.6.0)의 40개 항목과 맨 끝 `[open_with]` 표입니다. "강의 값" 열은 강의 끝 상태입니다.

| 항목 | 기본값 | 설명 | 강의 값 |
|---|---|---|---|
| `editor` | `""` | `e` 로 파일을 열 편집기. 비우면 `$EDITOR`, 그것도 없으면 nano | `"nvim"` |
| `dir_editor` | `""` | `E` 로 현재 폴더를 열 편집기. 비우면 macOS는 `open`, 리눅스는 `vi` | `"nvim"` |
| `auto_check_update` | `true` | 종료할 때 새 버전 확인 (하루 한 번, GitHub API) | — |
| `cd_on_quit` | `false` | `q` 종료 때도 마지막 폴더로 cd (셸 함수 필요) | — |
| `default_open_file_preview` | `true` | 시작할 때 미리보기 패널 열기 | — |
| `show_image_preview` | `true` | 이미지 파일 미리보기 | — |
| `show_panel_footer_info` | `true` | 파일 패널 아래 테두리에 정렬·모드 표시 | — |
| `default_directory` | `"."` | 인자 없이 실행할 때의 시작 폴더 (`~`·상대 경로 가능) | — |
| `file_size_use_si` | `false` | `true` 면 1000 단위(kB), `false` 면 1024 단위(KiB) | — |
| `default_sort_type` | `0` | 0 이름 · 1 크기 · 2 수정 시각 · 3 종류 · 4 자연 정렬 | — |
| `sort_order_reversed` | `false` | 기본 정렬 순서 뒤집기 | — |
| `case_sensitive_sort` | `false` | 대소문자 구분 정렬 | — |
| `shell_close_on_success` | `false` | 셸 모드 명령이 성공하면 프롬프트 닫기 | — |
| `page_scroll_size` | `0` | `page_up`·`page_down` 이동 줄 수 (0 = 한 화면) | — |
| `debug` | `false` | 로그를 디버그 수준으로 | — |
| `ignore_missing_fields` | `false` | `missing fields` 경고 끄기 | — |
| `file_panel_extra_columns` | `0` | 파일 이름 외 추가 열 수 (0 = 끔) | — |
| `file_panel_name_percent` | `50` | 추가 열이 있을 때 이름 열 비율 (25–100) | — |
| `theme` | `"catppuccin-mocha"` | `theme/` 폴더의 파일 이름 (확장자 제외) | `"tokyonight"` |
| `code_previewer` | `""` | `""` 내장 chroma, `"bat"` 은 bat 사용 (`--plain` 으로 호출) | `"bat"` |
| `nerdfont` | `true` | Nerd Font 아이콘 사용 | — |
| `show_select_icons` | `true` | 선택 모드 체크박스 아이콘 (nerdfont 필요) | — |
| `transparent_background` | `false` | 배경 투명 (터미널 지원 필요) | — |
| `file_preview_width` | `0` | 미리보기 너비 = 전체의 1/n (2–10, 0 = 파일 패널과 같음) | — |
| `enable_file_preview_border` | `false` | 미리보기 패널 테두리 | `true` |
| `sidebar_width` | `20` | 사이드바 너비 (5–20, 0 = 숨김) | — (실험 파일에서 0) |
| `sidebar_sections` | `["home", "pinned", "disks"]` | 사이드바 묶음과 순서 | — |
| `border_top` | `'─'` | 테두리 문자 (한 칸 폭 문자만) | — |
| `border_bottom` | `'─'` | 〃 | — |
| `border_left` | `'│'` | 〃 | — |
| `border_right` | `'│'` | 〃 | — |
| `border_top_left` | `'╭'` | 〃 | — |
| `border_top_right` | `'╮'` | 〃 | — |
| `border_bottom_left` | `'╰'` | 〃 | — |
| `border_bottom_right` | `'╯'` | 〃 | — |
| `border_middle_left` | `'├'` | 〃 | — |
| `border_middle_right` | `'┤'` | 〃 | — |
| `metadata` | `false` | 자세한 메타데이터 (exiftool 필요) | — (exiftool 없음) |
| `enable_md5_checksum` | `false` | 메타데이터에 MD5 체크섬 추가 | — |
| `zoxide_support` | `false` | `z` 이동 창·zoxide 기록 (zoxide 필요) | `true` (섹션 3) |
| `[open_with]` | (빈 표) | 확장자별로 열 명령 (`png = "feh"` 꼴). **파일 맨 끝**에 있어야 함 | — |

### 예상 출력

```
> editor = "nvim"                 # e — 파일을 Neovim으로 (비우면 $EDITOR, 그것도 없으면 nano)
> dir_editor = "nvim"             # E — 현재 폴더를 Neovim으로 (비우면 macOS는 open)
> theme = "tokyonight"            # theme/ 폴더의 파일 이름 (확장자 제외)
```

```
Dependencies
ffmpeg              : ffmpeg version 9.0.1 Copyright (c) 2000-2026 the F...
pdftoppm            : pdftoppm version 26.09.0
exiftool            : Not Found
bat                 : bat 0.26.1
zoxide              : zoxide 0.10.0
open                : Found at /usr/bin/open
pbcopy              : Found at /usr/bin/pbcopy
```

(도구 버전은 환경마다 다릅니다.) `comm -13` 출력은 `shell_close_on_success` 한 줄입니다. `e` 는 Neovim으로 `app.py` 를 열고, `E` 는 폴더 목록(플러그인이 없으면 netrw)을 엽니다. `:q` 로 닫으면 superfile로 돌아옵니다.

### 더 알아보기 — 왜 그런가

#### 플러그인 문서 예시의 [plugins] 머리줄 함정

공식 "Enable Plugin" 문서의 diff 예시는 `[plugins]` 머리줄 아래에서 `metadata` 를 바꾸는 모양입니다. 1.6.0 `config.toml` 에는 그런 표가 없습니다. 그대로 `[plugins]` 를 넣으면 그 아래 항목들이 `plugins` 표 안으로 들어가, 최상위에서는 빠진 항목이 됩니다. 격리 환경에서 `metadata` 바로 위에 `[plugins]` 를 넣자 `missing fields: [metadata enable_md5_checksum zoxide_support]` 가 뜨고 세 값 모두 기본값(`false`)으로 동작했습니다. 플러그인은 해당 줄의 값만 `true` 로 바꿉니다.

#### [open_with] 가 맨 끝이어야 하는 이유

TOML에서 `[이름]` 머리줄 아래의 모든 키는 다음 머리줄이 나올 때까지 그 표에 속합니다. 기본 설정 파일 주석도 `MUST BE IN THE VERY END OF THE FILE` 이라고 적고 있습니다. 빠진 항목을 `echo … >> config.toml` 로 파일 끝에 붙이면 `[open_with]` 표 안으로 들어가고, 문자열만 받는 표라 `toml: cannot assign boolean to a false` 로 시작이 거부됩니다(격리 환경 재현). 새 항목은 `[open_with]` 위에 넣습니다.

#### --fix-config-file 이 지우는 것

1.6.0의 `--fix-config-file` 은 섹션 4의 `--fix-hotkeys` 와 같은 함수를 써서 같은 방식으로 파일을 망가뜨립니다. 격리 환경에서 한 항목을 뺀 설정에 실행하자 다음 실행이 `toml: key transparent_background is already defined` 로 거부되었습니다. 다시 쓴 앞부분에는 직접 단 한글 주석이 모두 사라지고 값도 작은따옴표 형식(`editor = 'nvim'`)으로 바뀌어 있었습니다. 빠진 이름은 `comm -13` 으로 찾아 손으로 넣는 편이 안전합니다. 비교 기준이 없다면 1.6.0 태그의 기본 설정(`https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/config.toml`)을 받습니다. 이름 패턴은 `enable_md5_checksum` 의 숫자 때문에 `[a-z0-9_]` 로 잡습니다.

#### 시작 폴더가 정해지는 순서

인자로 받은 경로가 있으면 그것을, 없으면 `default_directory` 를 씁니다. `~` 와 상대 경로는 현재 셸 폴더 기준으로 풉니다. 그 경로가 없으면 zoxide 연동이 켜져 있을 때 zoxide에 질의하고, 결과가 없거나 연동이 꺼져 있으면 홈 폴더로 엽니다(소스, `spf logs` 로 zoxide 경로 확인). 기본값 `.` 을 유지하면 "셸에서 폴더를 찾아 들어가 `spf` → `Q` 로 다른 폴더에 내려앉기" 흐름이 자연스럽게 이어집니다.

#### editor·dir_editor 값의 해석 순서

`editor`·`dir_editor` 값은 공백으로 나눠 앞을 명령, 뒤를 인자로 씁니다(소스). `editor` 가 비면 `EDITOR` 환경 변수, 그것도 없으면 nano입니다. 공식 튜토리얼은 `editor` 설정이 `EDITOR` 보다 우선한다고 적고 있고, 소스도 같은 순서입니다. AstroNvim처럼 폴더를 열면 파일 탐색기 플러그인이 뜨는 설정이라면 `E` 가 그 화면으로 열립니다.

#### bat 미리보기와 테마의 관계

`code_previewer = "bat"` 이면 superfile은 PATH에서 `bat`(없으면 `batcat`)을 찾아 `--plain --force-colorization --line-range :<줄수>` 로 호출하고, 결과에 superfile 테마의 배경색을 입힙니다. 코드 색은 bat 설정(`BAT_THEME` 등)을 따르므로 superfile 테마와 따로 움직입니다. bat이 없으면 미리보기가 기대대로 나오지 않으므로 `--debug-info` 의 `Dependencies` 를 먼저 봅니다.

#### 이미지 미리보기를 강의에서 다루지 않은 이유

`show_image_preview` 는 기본으로 켜져 있지만, 이미지가 실제로 보이는지는 터미널의 그래픽 지원과 tmux 설정에 달려 있습니다. 이번 검증은 tmux 화면 캡처(문자) 기반이라 이미지 표시를 확인할 수 없었습니다(미실행).

---

## 7. 터미널·tmux와의 키 충돌

### 실행 코드

지금 쓰는 tmux에서 (읽기만 함):

```bash
tmux show -gv prefix                                      # 지금 prefix — 기본은 C-b
tmux list-keys -T root | grep -vE 'Mouse|Wheel|Click'     # prefix 없이 가로채는 키(bind -n) — 비어 있으면 없음
tmux show -s extended-keys                                # 확장 키 전달 — 기본 off
```

tmux 밖의 새 터미널 탭에서 시험용 tmux를 따로 띄워 재현:

```bash
tmux -L spftest -f /dev/null new-session -s t \; set -g prefix C-a \; bind C-a send-prefix \; bind -n S-Left previous-window   # 시험용 tmux — prefix C-a, shift+left 가로채기
cd ~/work/spf-lab/inbox && spf                            # 시험용 tmux 안에서 실행 (커서는 archive) — ctrl+a, esc, ctrl+a ctrl+a, N, shift+left, H, q
sed -i '' "s/^compress_file = .*/compress_file = ['ctrl+a', 'ctrl+g']        # ctrl+g — tmux prefix가 C-a인 환경용 둘째 키/" ~/.config/superfile/hotkeys.toml   # 둘째 키 추가
spf                                                       # ctrl+g → archive(1).zip 생성 → q
exit                                                      # 시험용 tmux 종료
```

`-L spftest` 는 별도 소켓의 tmux 서버라 평소 쓰는 tmux 서버·설정과 섞이지 않습니다. `-f /dev/null` 로 설정 파일도 읽지 않습니다.

### 확인한 tmux 키 사례

| 누른 키 (시험용 tmux 안) | tmux 설정 | superfile에 닿았나 | 결과 |
|---|---|---|---|
| `ctrl+a` 한 번 | `prefix C-a` | 아니오 | tmux가 prefix로 먹음 — 압축 안 됨 |
| `ctrl+a` → `ctrl+a` | `prefix C-a` + `bind C-a send-prefix` | 예 (두 번째) | `archive.zip` 생성 |
| `shift+left` | `bind -n S-Left previous-window` | 아니오 | tmux 상태줄 `No previous window`, 패널 초점 그대로 |
| `shift+left` | `bind -n` 해제 후 | 예 | 이전 패널로 초점 이동 |
| `H` | (가로채기 없음) | 예 | 이전 패널로 초점 이동 |
| `ctrl+g` | (가로채기 없음) | 예 | `compress_file` 둘째 키로 `archive(1).zip` 생성 |

### 예상 출력

```
C-b
extended-keys off
```

(`list-keys -T root | grep -vE …` 는 tmux 기본 설정에서 아무것도 출력하지 않습니다.) 시험 뒤 `inbox` 에는 `archive.zip`·`archive(1).zip` 이 생깁니다.

### 더 알아보기 — 왜 그런가

#### prefix 와 bind -n — tmux가 먼저 가져가는 두 종류

tmux는 터미널에서 들어온 키를 먼저 자기 키 표와 대조합니다. prefix 키는 "다음 키는 tmux 명령"이라는 신호라 그 자체로 앱에 넘어가지 않습니다. `bind -n`(root 표)으로 묶은 키는 prefix 없이도 tmux가 가져갑니다. 둘 다 superfile에는 닿지 않으므로, 겹치는 키는 superfile 쪽에서 비켜 가는 것이 현실적입니다. prefix는 손에 깊이 배어 있어 바꾸기 어렵기 때문입니다.

#### ctrl+a 를 두 번 누르면 넘어가는 이유

`bind C-a send-prefix` 는 prefix 다음에 다시 `C-a` 를 누르면 그 키를 앱에 그대로 보내라는 설정입니다. prefix를 `C-a` 로 바꾸는 설정 예시는 보통 이 줄을 함께 둡니다. 이 저장소의 tmux 강의(0002, 시리즈 0001 3강)의 prefix 변경 예시도 같습니다. 그래서 `ctrl+a` 를 두 번 누르면 superfile의 압축이 됩니다. 다만 매번 두 번 누르는 것보다 `compress_file` 에 둘째 키를 주는 편이 실수가 적습니다. 첫 번째 `ctrl+a` 뒤 tmux가 명령 키를 기다리는 상태라, 실습에서는 esc로 그 대기를 끊은 뒤 두 번 눌렀습니다.

#### 글자 키 짝을 함께 두는 설계

기본 키맵은 `previous_file_panel = ['shift+left', 'H']`, `next_file_panel = ['tab', 'L']` 처럼 특수 키 조합과 글자 키를 짝으로 둡니다. 화살표·shift 조합은 터미널·tmux·OS 단축키에 가로채일 여지가 많지만, 대문자 글자는 거의 그대로 닿습니다. 직접 키맵을 짤 때도 가로채일 수 있는 키에는 글자 키 짝을 함께 두면 어느 환경에서도 한 가지는 동작합니다.

#### extended-keys 가 기본으로 꺼져 있다는 사실

tmux 3.7c의 `extended-keys` 기본값은 `off` 입니다. 이 상태에서 tmux는 `ctrl+i` 와 `tab` 처럼 전통 터미널이 구별하지 못하는 조합을 구별해 넘기지 않습니다. 섹션 4의 금지 키 실험이 tmux 안에서 탭·엔터·esc로 동작한 것과 맞습니다. `extended-keys` 를 켜고 터미널도 확장 키를 지원하면 일부 조합이 구별될 수 있지만, 이번 검증에서는 확인하지 않았습니다.

#### 터미널 앱에서 tmux까지는 확인 범위 밖

이번 검증은 tmux `send-keys` 로 키를 넣었기 때문에 "tmux → superfile" 구간만 확인한 것입니다. 그 앞의 "터미널 앱 → tmux" 구간은 터미널마다 다릅니다(Option 키를 Meta로 보낼지, OS·터미널 단축키가 먼저 가져가는지 등). 키가 안 먹으면 tmux 밖에서 superfile을 직접 띄워 보고, 거기서 되면 tmux 설정을, 안 되면 터미널 설정을 봅니다.

---

## 8. 트러블슈팅 & 클로징

### 트러블슈팅

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

### 로그와 진단 명령

```bash
spf pl                                     # 설정·단축키·로그·데이터 경로
spf pl --lastdir-file                      # Q 종료 기록 파일 경로
spf --debug-info                           # 버전·실제로 읽는 설정 경로·환경 변수·외부 도구
spf --hf presets/nvim-mix.toml --debug-info | grep -i 'hotkeys file'   # 옵션을 준 실행이 읽을 파일
grep -n 'ERROR' ~/Library/Application\ Support/superfile/superfile.log  # 로그의 오류 줄 — macOS 기본 위치 (XDG_STATE_HOME 을 쓰면 spf pl 의 Log 줄 경로)
```

`config.toml` 에서 `debug = true` 로 두면 로그가 디버그 수준으로 자세해집니다. 이슈를 올릴 때는 `spf --debug-info` 출력을 함께 붙입니다.

### 실습 정리 (지우지 않고 보관)

```bash
B="$HOME/rec-archive-0017-$(date +%Y%m%d-%H%M%S)"     # 보관 폴더
mkdir -p "$B"
mv ~/work/spf-lab "$B/"                               # 실습 폴더 보관
cp -R ~/.config/superfile/presets "$B/"               # 실험 키맵·설정 사본 보관 (원본은 그대로)
ls "$B"                                               # 옮긴 목록 확인
# 기본 키맵으로 되돌리려면: cp ~/.config/superfile/presets/default.toml ~/.config/superfile/hotkeys.toml
```

### 클로징

설정을 `~/.config` 로 모으고, 기본 키를 ctrl 원칙과 함께 익혔습니다. 프롬프트와 `Q` 로 셸과 오가는 길을 만들고, 단축키 파일의 세 구역과 문법을 실험으로 확인했습니다. 문서 표기와 실제 이름이 다른 함정, 1.6.0의 `--fix-` 옵션 문제를 재현한 뒤, 옆 파일에서 시험하고 검증해 승격하는 절차로 혼합 키맵을 만들었습니다. 확인 도구는 네 가지입니다 — `--hf`/`-c`, `--debug-info`, `?` 도움말, `comm` 비교.

---

## 유용한 링크

| 도구 | URL |
|---|---|
| superfile 저장소 | https://github.com/yorukot/superfile |
| superfile v1.6.0 릴리스 | https://github.com/yorukot/superfile/releases/tag/v1.6.0 |
| 공식 문서 (main 기준) | https://superfile.dev |
| 단축키 설정 문서 | https://superfile.dev/configure/custom-hotkeys/ |
| 단축키 목록 (main 기준) | https://superfile.dev/list/hotkey-list/ |
| 설정 파일 문서 | https://superfile.dev/configure/superfile-config/ |
| 설정 경로 문서 | https://superfile.dev/configure/config-file-path/ |
| 1.6.0 기본 hotkeys.toml | https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/hotkeys.toml |
| 1.6.0 vimHotkeys.toml | https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/vimHotkeys.toml |
| 1.6.0 기본 config.toml | https://raw.githubusercontent.com/yorukot/superfile/v1.6.0/src/superfile_config/config.toml |
| 공식 cd_on_quit 스크립트 | https://github.com/yorukot/superfile/tree/v1.6.0/cd_on_quit |
| zoxide | https://github.com/ajeetdsouza/zoxide |
| bat | https://github.com/sharkdp/bat |
| tmux 매뉴얼 (extended-keys·bind-key) | https://man.openbsd.org/tmux |
