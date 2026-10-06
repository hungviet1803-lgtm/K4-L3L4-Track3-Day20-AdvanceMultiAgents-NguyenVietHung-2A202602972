# Báo cáo Lab: Self evolving Agentic

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| Nguyen Viet Hung | 2A202602972 | Toàn bộ (làm cá nhân) |

- Nhà cung cấp và mô hình (`LAB_MODEL`, không ghi khóa API), nhiệt độ (`LAB_TEMPERATURE`), `recursion_limit`: `openai:gpt-5.4-mini`, `LAB_TEMPERATURE=1.0`, `recursion_limit=60` (mặc định của runner).
- Phiên bản Deep Agents (`pip show deepagents`), hệ điều hành, chạy trực tiếp hay trong Docker: `deepagents 0.7.21`; máy chủ Windows 11, mọi lần chạy tác tử và test thực hiện trong Docker (image từ `Dockerfile` của lab, `python:3.12-slim`) vì shell của tác tử cần `/bin/sh`.
- Số lần chạy tác vụ đã dùng / ngân sách: 21 lần chạy hợp lệ (18 chính thức + 3 ở Phần 3.4) và 2 lần gọi curator; không đặt ngân sách cố định. Các lần chạy bị loại do lỗi hạ tầng được ghi ở Phụ lục.
- Commit của tag `freeze`: `97d87f9` (commit `hypotheses`: `d22ebf5`).

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

- H1 (subagents so với baseline): `subagents` **không cao hơn** `baseline` trên tác vụ đánh giá (chênh lệch tổng không quá ±2 check, coi là nhiễu), nhưng tốn khoảng **+40-60% token**. Căn cứ: ở tác vụ học, check kỹ thuật đã đạt 18/18 ở cả hai điều kiện, nên không còn chỗ để cải thiện; 100% lỗi là nhóm E (quy ước không có trong đề), mà subagent không có thêm thông tin nào (lời giao việc ở `code-learn` chỉ chuyển tiếp nguyên câu "Follow Acme Python team conventions"); token tăng 36.988 → 54.959 (+49%). Bài viết của Anthropic về hệ đa tác tử ghi nhận chi phí token cao hơn nhiều lần, và lợi ích chỉ đến khi tác vụ cần song song hóa hoặc ngữ cảnh lớn, điều kiện không có ở các tác vụ nhỏ này.
- H2 (skills-auto so với baseline): `skills-auto` đạt **điểm cao nhất** trên tác vụ đánh giá, và toàn bộ phần tăng nằm ở **check quy ước (`rule_`) được tái sử dụng** từ tác vụ học; check kỹ thuật ngang nhau. Dự đoán: tăng rõ ở họ `code` và `logs` (skill nêu quy ước cụ thể; ở tác vụ học tăng 7→10 và 6→8), tăng ít hoặc không tăng ở họ `data` (skill thiếu tên khóa `meta`/`clean.csv`, và tác tử ưu tiên đề bài "number" hơn skill "integer cents", 5→5 ở tác vụ học). Quy ước **mới** của mỗi tác vụ đánh giá sẽ thất bại ở **cả ba** điều kiện, vì không skill nào có thông tin về nó. Căn cứ: SkillsBench ghi nhận skill do mô hình tự sinh trung bình không có lợi; ở đây nhóm dự đoán khác vì lỗi là thiếu tri thức tổ chức có phản hồi `detail` phát biểu rõ, đúng loại tri thức thủ tục mà skill truyền đạt được.
- H3 (tác vụ học so với tác vụ đánh giá): mức tăng của `skills-auto` so với `baseline` trên tác vụ đánh giá **nhỏ hơn** trên tác vụ học (học: 23/27 so với 18/27, tức +5 check). Lý do: (1) mỗi tác vụ đánh giá có thêm một quy ước mới mà skill không biết; (2) dữ liệu khác, và skill `log-triage-json-conventions` chép lại quy tắc lọc của đề học (bước 1-4), có nguy cơ quá khớp; (3) một phần mức tăng ở tác vụ học có thể là nhiễu của một lần chạy. Căn cứ: SkillEvolBench ghi nhận lợi ích trên tác vụ học thường không chuyển sang tác vụ mới.

## 3. Làm quen Deep Agents (Phần 0.3)

Nguồn: `python scripts/tour.py` (mô hình giả, không tốn token).

1. Tác tử mặc định có 9 công cụ: nhóm tệp `ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`; shell `execute`; subagent `task`. Công cụ cho phép chạy lệnh là `execute` ("Executes a shell command in an isolated sandbox and returns combined stdout/stderr with the exit code"), và chỉ hoạt động khi backend cài đặt `SandboxBackendProtocol`, nếu không sẽ trả về lỗi.
2. `task` khởi chạy một subagent tạm thời (ephemeral). Subagent `general-purpose` dùng cho câu hỏi phức tạp, tìm kiếm tệp/nội dung và tác vụ nhiều bước, và "has access to all tools as the main agent". Về ngữ cảnh: mỗi lần gọi mặc định là stateless, subagent **chỉ nhìn thấy prompt mà tác tử chính truyền vào** (không thấy lịch sử hội thoại, ý định người dùng hay vết trước đó) và trả về một báo cáo cuối duy nhất; vì vậy tác tử chính phải ghi đủ chi tiết trong prompt và nói rõ cần trả về gì.
3. System prompt mặc định là chuỗi rỗng `''`; hướng dẫn hành vi nằm trong mô tả công cụ:
   - Từ `task`: "Launch multiple agents concurrently when their tasks are independent, using a single message with multiple tool calls."
   - Từ `execute`: "You MUST avoid using search commands like find and grep. Instead use the grep, glob tools to search. Use read_file rather than cat/head/tail."

## 4. Đường cơ sở và phân loại lỗi (Phần 2.2)

Nguồn: `results/baseline/<task>/run.json` và `trace.md` (điều kiện `baseline`, 3 tác vụ học). Điểm: `code-learn` 7/10, `data-learn` 5/8, `logs-learn` 6/9.

| Tác vụ | Check thất bại | Nhóm lỗi (A-G) | Bằng chứng (trích ngắn từ `detail` hoặc vết) |
|---|---|---|---|
| code-learn | `rule_type_hints` | E | `detail`: "RULE: every public function ... has type annotations on all parameters and on the return value." Đề chỉ nói "checked ... against the Acme Python team conventions", không nêu quy ước. |
| code-learn | `rule_regression_tests` | E | `detail`: "RULE: add tests/test_regressions.py with one test function per bug you fixed (at least 3)". Vết: không có `write_file` nào vào `tests/`. |
| code-learn | `rule_changelog` | E | `detail`: "RULE: record each fix in CHANGELOG.md under the heading '## Unreleased' as a bullet '- fix(<function name>): ...'". Vết baseline không mở `CHANGELOG.md` (chỉ có tiêu đề `## Unreleased` trống, không nêu định dạng), nên không đọc cũng không suy ra được quy ước. |
| data-learn | `rule_money_in_cents` | E | `detail`: "RULE: money values in answer.json are integer cents (1606.67 USD is written 160667)". Đề lại ghi `north_q1_revenue` (number); tác tử ghi số thập phân USD. |
| data-learn | `rule_meta_block` | E | `detail`: "RULE: answer.json has an object `meta` = {source, rows_in, rows_used}". Đề chỉ nói "plus whatever the Acme reporting conventions require". |
| data-learn | `rule_clean_csv` | E | `detail`: "RULE: write workspace/clean.csv with the header order_id,timestamp_utc,region,amount_cents ...". Câu trả lời cuối: "Created `workspace/answer.json`", không tạo `clean.csv`. |
| logs-learn | `rule_service_names` | E | `detail`: "RULE: service names ... lower-case with '-' replaced by '_' (payment-service -> payment_service)". Ví dụ trong đề dùng `"payment-service"`, tác tử làm theo ví dụ. |
| logs-learn | `rule_sorted_errors` | E | `detail`: "RULE: `errors` is sorted by service, then by timestamp_utc, ascending." Tác tử giữ thứ tự xuất hiện trong log. |
| logs-learn | `rule_schema_header` | E | `detail`: "RULE: the top-level object has \"schema_version\": 2 and \"generated_by\": \"log-triage\"." |

**Nhóm chiếm đa số: E (9/9 check thất bại).** Nguyên nhân chung: cả 3 đề đều tham chiếu "Acme ... conventions" nhưng không nêu nội dung, và không tệp nào trong workspace mô tả quy ước (đã kiểm tra `README.md`, `CHANGELOG.md` của 3 tác vụ học). Tác tử không thể biết quy ước nếu không được cung cấp, nên đây là lỗi thiếu tri thức tổ chức, không phải lỗi năng lực.

**Bằng chứng phủ định cho nhóm A-D và F** (`python scripts/check_breakdown.py`: baseline learn **technical 18/18, house rules 0/9**):
- A (bỏ qua đặc tả): `data-learn` và `logs-learn` đọc `README.md` ngay sau `ls`; `code-learn` đọc toàn bộ mã nguồn và docstring trước khi sửa, và đạt cả `low_stock_follows_docstring`, `csv_quoting_follows_docstring`, là các lỗi chỉ thấy khi đối chiếu docstring.
- B (không kiểm chứng): `code-learn` chạy lại `pytest` sau khi sửa ("6 passed"); `logs-learn` đọc lại `errors.json` và in số entry để đối chiếu.
- C (vá triệu chứng): đạt `other_caller_fixed`, tức đã sửa hàm dùng chung `parse_price` chứ không vá từng nơi gọi.
- D (dữ liệu bẩn): đạt `duplicate_rows_removed`, `missing_amount_orders`, `timestamps_utc`, `repeat_counts`.
- F (báo cáo sai): câu trả lời cuối của cả 3 lần chạy chỉ nêu các tệp thật sự được ghi (`answer.json`, `errors.json`, 3 tệp trong `inventory/`).
- G (khác, không gây thất bại): ở `code-learn` (`python -m pytest /workspace/tests`) và `data-learn` (`path='/workspace/sales.csv'`), lệnh shell đầu tiên dùng đường dẫn tuyệt đối `/workspace/...` và thất bại, sau đó tác tử tự sửa thành `workspace/...`, tốn thêm 1-2 lần gọi công cụ. Nguyên nhân là công cụ tệp trả về đường dẫn ảo dạng `/workspace/...`, trái với `PATHS_NOTE`. `logs-learn` dùng đúng dạng tương đối ngay từ đầu.

**Skill có phòng ngừa được nhóm E không?** Có, và đây là trường hợp lý tưởng cho skill: phản hồi `detail` của tác vụ học phát biểu đúng quy ước, nên curator có thể chép quy ước thành checklist. Giới hạn: skill chỉ giúp được các quy ước đã thấy; một quy ước **mới** chỉ có ở tác vụ đánh giá thì skill không thể biết.

**Ghi chú tính hợp lệ:** lần chạy đầu tiên cho `code-learn` 6/10 với `tests_not_modified` thất bại dù vết không có lệnh ghi nào vào `tests/`. Nguyên nhân là `git core.autocrlf=true` trên Windows đã đổi `tasks/**` sang CRLF, làm sai hash SHA-256 mà `check.py` so sánh (hash tệp trên đĩa `efb5e765…` khác hash bản trong git `79e05f4c…`). Đây là lỗi hạ tầng: nhóm đã đặt `core.autocrlf=false`, khôi phục `tasks/` từ git (không sửa nội dung), loại bỏ cả 6 lần chạy đó và chạy lại. Bảng trên dùng các lần chạy sau khi sửa.

## 5. Điều kiện `subagents` (Phần 2.3)

- Các subagent đã định nghĩa (tên, vai trò, lý do thiết kế): theo vòng đời một tác vụ, mỗi vai trò có phạm vi tách biệt:
  - `explorer` (chỉ đọc): khảo sát README, CHANGELOG, docstring, test, mẫu dữ liệu; báo cáo quy ước và edge case kèm ví dụ. Lý do: giảm nhóm lỗi A và D bằng cách tách việc tìm hiểu đặc tả thành một bước riêng.
  - `implementer`: thực hiện thay đổi, sửa nguyên nhân gốc, tính bằng Python và chạy lại test. Lý do: nhóm lỗi C và B.
  - `reviewer` (chỉ đọc): kiểm tra độc lập theo từng quy tắc, PASS/FAIL kèm bằng chứng. Lý do: nhóm lỗi B và F.
  - `description` của mỗi subagent nêu rõ **khi nào** gọi (FIRST / once the rules are known / LAST) và nhắc tác tử chính gửi đủ quy tắc, vì subagent chỉ thấy lời giao việc.
- `subagent_calls` ở từng tác vụ và nhận xét (kể cả trường hợp bằng 0):

  | Tác vụ | `subagent_calls` | Subagent | Nhận xét từ vết |
  |---|---|---|---|
  | code-learn | 1 | `implementer` | Tác tử chính tự đọc README, CHANGELOG, mã nguồn và test (10 lần gọi công cụ) rồi mới giao toàn bộ việc sửa cho `implementer`; sau đó tự chạy lại `pytest` để kiểm tra báo cáo ("6 passed"). |
  | data-learn | 0 | (không) | Tác tử chính tự làm hết bằng 2 lệnh Python. Tác vụ ngắn (đọc README, 20 dòng CSV, một script), nên giao việc chỉ thêm chi phí; `SUBAGENTS_NOTE` chỉ khuyến khích, không bắt buộc. |
  | logs-learn | 1 | `explorer` | Giao việc khảo sát định dạng log; sau đó tác tử chính vẫn tự đọc lại toàn bộ `app.log` (156 dòng) trước khi viết parser, tức báo cáo của `explorer` bị làm lại một phần. |

  `reviewer` không được gọi lần nào; tác tử chính coi `pytest` hoặc kết quả script của mình là đủ để kiểm chứng.
- Thông tin thiếu hoặc thừa khi giao việc (nếu có giao việc): lời giao cho `implementer` ở `code-learn` khá đầy đủ (liệt kê các định dạng của `parse_price`, quy tắc làm tròn, quy tắc quote CSV, ràng buộc "do not modify anything under workspace/tests/"). Nhưng câu "Follow Acme Python team conventions" được chuyển tiếp **mà không có nội dung**, vì chính tác tử chính cũng không biết quy ước. Đa tác tử không tạo ra thông tin mới: 3 check `rule_` vẫn thất bại.
- Ảnh hưởng đến token và thời gian: kết quả giống hệt baseline (technical 18/18, house rules 0/9) nhưng token trung bình tăng từ 36.988 lên 54.959 (**+49%**). Chi tiết: `code-learn` 59.082 → 80.358 token (22,8 → 33,8 s); `logs-learn` 35.674 → 61.649 token (11,0 → 25,1 s); `data-learn` (không giao việc) 16.210 → 22.870 token, chênh lệch do system prompt dài hơn và do nhiễu. Ở tác vụ học, đa tác tử không đáng chi phí.

## 6. Self-evolving: skill do curator sinh (Phần 3)

- Số lần chạy curator, số skill bị xóa và lý do: **2 lần chạy, xóa 3 skill (toàn bộ lần 1)**.
  - Lần 1 sinh `package-fix-conventions`, `structured-answer-cleaning`, `log-triage-serialization` (đều hợp lệ về định dạng). Bị xóa vì **vô dụng với nhóm lỗi E (9/9 lỗi)**: các quy ước được viết dạng điều kiện ("When a rule says public APIs must be typed...", "normalize service names *if required*, sort entries *if required*") và mất toàn bộ chi tiết cụ thể (integer cents, khối `meta`, header `clean.csv`, định dạng bullet changelog). Ở tác vụ mới, tác tử không thấy rule nào, nên điều kiện "nếu rule yêu cầu" không bao giờ kích hoạt. Phần còn lại lặp lại các bước kỹ thuật mà tác tử đã làm đúng (18/18). Nguyên nhân nằm ở prompt curator của nhóm: câu "never mention ... numbers" khiến mô hình trừu tượng hóa cả quy ước. Nhóm sửa **prompt của curator** (không sửa skill): quy ước `RULE:` phải viết thành mệnh lệnh vô điều kiện, đủ chi tiết bắt buộc; cấm viết "if a rule requires"; không lặp lại check đã đạt.
  - Lần 2 sinh 3 skill trong bảng dưới. Nhóm **không dùng lượt chạy lại thứ 2** dù 2 skill còn thiếu sót: khi đã biết chính xác rule nào bị thiếu, việc chỉnh prompt tiếp thực chất là người lái nội dung skill, làm sai lệch thí nghiệm "tác tử tự tiến hóa". Các thiếu sót được giữ nguyên và phân tích.

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| `python-package-fix-conventions` | Tổng quát cho loại "sửa gói Python có test lỗi": không nêu tên gói, tên hàm hay con số; các tên có mặt (`## Unreleased`, `- fix(<function name>): ...`) là chính quy ước. | Đúng nhưng **thiếu**: không nêu tên tệp bắt buộc `tests/test_regressions.py` và mức "ít nhất 3" test. Không có hướng dẫn có hại. | 14 dòng, ngắn và mệnh lệnh, có self-check. `description` "Use when fixing a Python package so tests pass and review-bot conventions must also be satisfied" nêu đúng tình huống. Được đọc ở `code-learn` (`skills_read=1`, là hành động thứ 2 của tác tử). |
| `structured-data-answering-conventions` | Tổng quát cho loại "trả lời từ CSV/JSON bẩn"; không chứa con số hay tên cột của tác vụ học. | **Thiếu và một phần vẫn dạng điều kiện**: có "integer cents" (đúng), nhưng khối metadata chỉ mô tả mơ hồ ("metadata object with the source file name, input row count, and rows used count") mà không có tên khóa `meta`/`source`/`rows_in`/`rows_used`; bước 5 "If a cleaned CSV is required" không nêu tên tệp `clean.csv` và header. Đúng khiếm khuyết đã khiến lần 1 bị xóa. | 14 dòng. `description` đúng tình huống. Được đọc ở `data-learn` (`skills_read=2`: tác tử đọc cả skill log, không liên quan, do quy tắc "read every skill whose description could apply"). |
| `log-triage-json-conventions` | Tổng quát cho loại "log thành JSON"; nêu đủ 3 quy ước cụ thể (lower-case và `-`→`_`; sắp theo service rồi `timestamp_utc`; `schema_version: 2`, `generated_by: "log-triage"`). Bước 1-4 lặp lại các quy tắc của đề học (lọc ERROR/CRITICAL, repeat count), nên **có nguy cơ quá khớp** nếu một tác vụ log mới có quy tắc lọc khác. | Đúng; tốt nhất trong 3 skill. | 16 dòng, có self-check. `description` đúng tình huống. Được đọc ở `logs-learn` (`skills_read=1`) và bị đọc nhầm ở `data-learn`. |

**Kiểm tra trên tác vụ học (Phần 3.4, `results/skills-auto-dev/`)**, so với baseline:

| Tác vụ | baseline | skills-auto (3.4) | Skill được đọc | Làm theo skill? (từ `trace.md`) |
|---|---|---|---|---|
| code-learn | 7/10 | **10/10** | `python-package-fix-conventions` | Làm theo toàn bộ: thêm type hints vào mọi hàm public, tạo `tests/test_regressions.py` (tự chọn đúng tên dù skill không nêu), ghi 3+ bullet `- fix(parse_price): ...` dưới `## Unreleased`, rồi chạy lại pytest "10 passed". |
| data-learn | 5/8 | 5/8 | `structured-data-...`, `log-triage-...` | **Đọc nhưng không làm theo.** Skill ghi "Store monetary outputs in integer cents", nhưng tác tử vẫn ghi `north_q1_revenue: 3130.24` vì đề ghi "(number)": khi đề và skill khác nhau, tác tử làm theo đề. Không tạo `meta` và `clean.csv` vì skill không nêu tên khóa/tên tệp (dạng điều kiện). |
| logs-learn | 6/9 | 8/9 | `log-triage-json-conventions` | **Làm theo một phần**: có `schema_version`/`generated_by` và đã sắp xếp (đạt `rule_schema_header`, `rule_sorted_errors`), nhưng bỏ sót bước 5: output vẫn `"service": "auth-service"` (fail `rule_service_names`). Bước tự kiểm tra của tác tử chỉ `assert` hai khóa schema, không kiểm tra tên service. |

## 7. Kết quả so sánh (Phần 4.3, 4.4)

`report/table.md` (`python -m lab.compare`):

| Task | baseline | subagents | skills-auto |
|---|---|---|---|
| code-learn | 7/10 | 7/10 | 10/10 |
| data-learn | 5/8 | 5/8 | 5/8 |
| logs-learn | 6/9 | 6/9 | 9/9 |
| code-eval | 7/11 | 7/11 | 10/11 |
| data-eval | 5/9 | 0/9 | 5/9 |
| logs-eval | 6/10 | 1/10 | 6/10 |
| **Mean score - learning tasks** | 0.66 | 0.66 | 0.88 |
| **Mean score - evaluation tasks** | 0.60 | 0.25 | 0.69 |
| **Mean tokens per run** | 29,874 | 61,910 | 50,266 |
| **Runs that read a skill** | 0/6 | 0/6 | 6/6 |

`python scripts/check_breakdown.py`:

```text
condition     role    technical  house rules  mean tokens  read a skill
baseline      eval     18/18         0/12          22,760      0/3
baseline      learn    18/18         0/9           36,988      0/3
subagents     eval      8/18         0/12          68,862      0/3
subagents     learn    18/18         0/9           54,959      0/3
skills-auto   eval     18/18         3/12          54,928      3/3
skills-auto   learn    18/18         6/9           45,605      3/3
```

`python scripts/verify_freeze.py`: `checked 6 runs of skill conditions: OK`. Mọi `run.json` trong bảng có `error = null` và `skills_modified = false`.

Các lần chạy có `error` (đã chạy lại, chi tiết ở Phụ lục): `skills-auto` `data-learn`, `logs-learn`, `data-eval`, `logs-eval` bị `OpenAIRateLimitError 429` (giới hạn 200.000 token/phút), sau đó `data-eval` bị `OpenAIConnectionError` và 3 lần lỗi khóa API (401/403, 0 token). Mỗi tác vụ được chạy lại tuần tự với cùng mô hình và cùng skill đã đóng băng; bảng trên chỉ chứa lần chạy thành công.

**Sự cố cần lưu ý khi đọc bảng:** trong khoảng 10:45-10:53 UTC có **hai tiến trình runner chạy song song** (một lệnh gộp bị hủy trên giao diện nhưng container Docker vẫn chạy). Ba bản ghi hiện tại do tiến trình thứ hai ghi: `subagents/data-eval` (bắt đầu 10:46:29, 135,3 s, **0/9**), `subagents/logs-eval` (10:48:45, **1/10**) và `skills-auto/code-eval` (10:49:07, 10/11). Lần chạy cùng cấu hình của tiến trình chính in ra `subagents data-eval 4/9`, `subagents logs-eval 6/10`, `skills-auto code-eval 10/11` nhưng bị ghi đè. Các bản ghi còn lại đều là lần chạy hoàn chỉnh, sau `freeze`, cùng mô hình, không lỗi. Nhóm **giữ nguyên `results/`** (không chạy lại để chọn kết quả) và dùng các con số bị ghi đè làm bằng chứng về nhiễu (mục 8.6). Hai tiến trình song song cũng là nguyên nhân làm cạn giới hạn token/phút và gây các lỗi 429 ở trên.

## 8. Phân tích

1. **Tác vụ học và tác vụ đánh giá.** Trên tác vụ học, chỉ `skills-auto` cải thiện (0,88 so với 0,66; 24/27 so với 18/27 check), còn `subagents` bằng `baseline` (0,66). Trên tác vụ đánh giá, `skills-auto` vẫn cao nhất (0,69 so với 0,60; 21/30 so với 18/30), còn `subagents` **thấp hơn hẳn** (0,25). Mức tăng của `skills-auto` co lại từ +6 check (học) xuống +3 check (đánh giá), và toàn bộ +3 nằm ở **một họ** (`code-eval` 7→10). Ở họ `logs`, skill tăng 6→9 trên tác vụ học nhưng 6→6 trên tác vụ đánh giá. Đó là dấu hiệu **quá khớp**: skill log giúp tác vụ đã sinh ra nó nhưng không chuyển sang tác vụ mới cùng loại (cơ chế ở câu 5). Kết quả khớp H3 và kết quả của SkillEvolBench. H2 chỉ đúng một phần: `skills-auto` cao nhất, nhưng phần tăng chỉ ở họ `code`, không ở họ `logs` như dự đoán. H1 đúng (không cao hơn baseline), nhưng thực tế còn tệ hơn dự đoán (xem câu 4).
2. **Kỹ thuật và quy ước (`rule_`).** Check kỹ thuật đạt **18/18 ở cả 3 điều kiện** trên cả tác vụ học và đánh giá, trừ `subagents` eval (8/18). Như vậy skill **không** đóng góp vào phần kỹ thuật; toàn bộ phần tăng nằm ở check quy ước: học 0/9 → 6/9, đánh giá 0/12 → 3/12. Trên tác vụ đánh giá, 3 check đạt đều là quy ước **tái sử dụng** từ tác vụ học (`rule_type_hints`, `rule_regression_tests`, `rule_changelog` của `code-eval`). Mỗi tác vụ đánh giá có đúng một quy ước **mới** (`rule_version_bump`, `rule_sorted_keys_format`, `rule_source_line`), và quy ước mới đạt **0/3 ở cả 3 điều kiện**. Lý do: curator chỉ thấy phản hồi của tác vụ học, nên không skill nào chứa thông tin về quy ước mới. Skill chỉ truyền được tri thức đã quan sát, không suy ra được quy ước chưa từng có phản hồi.
3. **Một check skill giúp đạt, một check skill không giúp.**
   - Giúp: `code-eval` `rule_changelog` và `rule_regression_tests` (`skills_read=1`). Vết: tác tử đọc `python-package-fix-conventions` trước khi sửa, sau đó tạo `workspace/tests/test_regressions.py` và cập nhật `CHANGELOG.md` ("updated `workspace/CHANGELOG.md` with the required unreleased fix notes"). Baseline và subagents đều không làm hai việc này.
   - Không giúp, kiểu **đọc nhưng không làm theo**: `data-eval` `rule_money_in_cents`. `skills_read=1` (đọc `structured-data-answering-conventions`, có câu "Store monetary outputs in integer cents"), nhưng vết ghi `"march_revenue_utc": 52957.19`. Đề ghi "`march_revenue_utc` (number)", và khi đề và skill khác nhau, tác tử làm theo đề. Hiện tượng này lặp lại đúng như ở `data-learn` (mục 6).
   - Không giúp, kiểu **skill sai làm tác tử bỏ cả skill**: `logs-eval` `rule_schema_header`, `rule_sorted_errors`, `rule_service_names` (đạt 9/9 quy ước tương ứng ở `logs-learn`). Tác tử đọc `log-triage-json-conventions`, nhưng bước 1 của skill nói "keep only ERROR and CRITICAL levels" còn README eval nói "ERROR, SEVERE and FATAL". Tác tử làm theo README, và output không có `schema_version`, không sắp xếp, giữ nguyên `queue-worker`. Nhóm diễn giải rằng khi một phần skill mâu thuẫn rõ với đề, tác tử coi cả skill là không áp dụng được, kể cả các quy ước đúng.
   - Không giúp, kiểu **skill thiếu**: `rule_meta_block`, `rule_clean_csv` ở cả `data-learn` và `data-eval`. Skill không có tên khóa `meta`/`rows_in`/`rows_used` và viết "If a cleaned CSV is required", nên tác tử không tạo `clean.csv`.
4. **Chi phí.** Token trung bình mỗi lần chạy: `baseline` 29.874, `subagents` 61.910 (**+107%**), `skills-auto` 50.266 (+68%). Điểm trung bình trên 6 tác vụ: 0,630 / 0,455 / 0,782. Tính theo điểm trên 10.000 token: `baseline` **0,21**, `skills-auto` 0,16, `subagents` 0,07. `baseline` hiệu quả nhất trên mỗi token. `skills-auto` đổi +68% token lấy +0,15 điểm; chi phí tăng chủ yếu do đọc skill và làm thêm việc theo quy ước (`code-learn`: 22,8 s / 59.082 token → 27,0 s / 83.105 token khi thêm type hints, test và changelog). **Đa tác tử không đáng chi phí trong thí nghiệm này**: tốn gấp đôi token, không có lợi ở tác vụ học, và ở tác vụ đánh giá còn gây hại. Ở `subagents/data-eval` (0/9), tác tử chính giao toàn bộ phép tính cho `general-purpose` rồi chép nguyên các con số vào `answer.json` mà không tự tính lại, nên fail cả 5 check kỹ thuật (baseline tự tính thì đạt 5/5). Ở `subagents/logs-eval` (1/10), `implementer` ghi khóa `"entries"` thay vì `"errors"`; tác tử chính kiểm tra bằng `assert 'errors' in obj`, assert thất bại thì **sửa assert thành `'entries'`** thay vì sửa tệp, rồi chấp nhận báo cáo tự mâu thuẫn của subagent ("24 total" trong khi 8+23+21=52). Lời giao việc làm mất thông tin quan trọng của đề (ví dụ JSON mẫu), và "Check what a subagent returns" trong `SUBAGENTS_NOTE` không được thực hiện thật. Đúng nhận định của Anthropic: đa tác tử tốn nhiều token và chỉ có lợi với tác vụ cần song song hóa hoặc ngữ cảnh lớn.
5. **Rò rỉ và quá khớp.** *Rò rỉ*: không phát hiện. Cả 3 skill qua `validate_skill` (không chứa định danh tác vụ đánh giá); curator chỉ đọc bản ghi `role == "learn"` (có test kiểm tra prompt không chứa dữ liệu eval); trước tag `freeze`, nhóm không mở `check.py`, workspace hay `run.json` của tác vụ đánh giá (lịch sử git: commit `hypotheses` và `freeze` chỉ chứa kết quả tác vụ học). Các giá trị cụ thể trong skill (`schema_version: 2`, `## Unreleased`, integer cents) đều lấy từ `detail` của tác vụ học. *Quá khớp*: có. `log-triage-json-conventions` bước 1-4 chép lại quy tắc riêng của đề `logs-learn` (lọc ERROR/CRITICAL, traceback làm `exception`), làm skill mâu thuẫn với đề `logs-eval` và bị tác tử bỏ qua (câu 3). Biện pháp đã dùng: prompt curator yêu cầu không nêu tên tệp, hàm, cột hay con số của tác vụ, và không lặp lại các check đã đạt; skill lần 1 bị xóa (mục 6). Biện pháp còn thiếu: prompt chưa cấm chép **quy tắc nghiệp vụ của đề** vào skill. Đề xuất: chỉ cho curator thấy check thất bại (không thấy đề bài), hoặc yêu cầu tách rõ phần "quy ước tổ chức" và phần "quy trình chung".
6. **Nhiễu.**
   - Cùng bộ skill, tác vụ học, Phần 3.4 (`results/skills-auto-dev`) so với sau đóng băng (`results/skills-auto`): `code-learn` 10 → 10, `data-learn` 5 → 5, `logs-learn` 8 → 9; tổng 23/27 → 24/27, chênh **1 check**. Ở `logs-learn`, lần 3.4 bỏ sót `rule_service_names`, lần sau thì đạt, dù skill giống hệt.
   - Bằng chứng mạnh hơn từ sự cố chạy song song (mục 7): cùng cấu hình `subagents`, cùng tác vụ, hai lần chạy cho `data-eval` 4/9 và 0/9, `logs-eval` 6/10 và 1/10, tức chênh **4-5 check trên một tác vụ**.
   - Hệ quả: với một lần chạy mỗi cấu hình, chênh lệch 1-3 check (ví dụ +3 check của `skills-auto` trên tác vụ đánh giá) **nằm trong biên độ nhiễu** và không đủ để kết luận chắc chắn. Kết luận đáng tin hơn là các mẫu lặp lại nhất quán: check kỹ thuật 18/18 ở mọi điều kiện không có subagent; quy ước mới 0/3 ở mọi điều kiện; các quy ước của `code` được áp dụng ở cả học lẫn đánh giá. Điểm thấp của `subagents` trên tác vụ đánh giá nên đọc là "phương sai cao và có cơ chế thất bại rõ ràng", không phải là một mức điểm ổn định.

## 9. Hạn chế và tính hợp lệ

1. **Một lần chạy mỗi cấu hình, nhiễu lớn.** Hai lần chạy cùng cấu hình chênh tới 4-5 check trên một tác vụ (mục 8.6). Ảnh hưởng: mọi chênh lệch nhỏ trong bảng (đặc biệt +3 check của `skills-auto` trên tác vụ đánh giá, và mức thấp của `subagents`) chỉ là chỉ báo, không phải kết luận thống kê. Cần lặp lại ít nhất 3 lần mỗi cấu hình (hướng 6e) để có khoảng dao động.
2. **Chỉ 3 tác vụ mỗi vai trò, mỗi họ 1 cặp.** Kết quả "skill giúp" thực chất chỉ đến từ một họ (`code`); với `n = 3`, một tác vụ đổi kết quả là đổi kết luận. Ảnh hưởng: không thể khái quát "skill tự sinh có lợi" ra ngoài loại tác vụ sửa mã Python.
3. **Tác vụ do giảng viên thiết kế với quy ước ẩn.** 100% lỗi baseline là nhóm E, vì đề cố ý không nêu quy ước còn bot đánh giá thì phát biểu quy ước trong `detail`. Đây là môi trường thuận lợi cho skill (phản hồi rõ, có thể chép lại), khác với SkillsBench nơi skill tự sinh trung bình không có lợi. Ảnh hưởng: mức tăng của `skills-auto` có thể bị phóng đại so với tác vụ thực tế, nơi phản hồi mơ hồ hơn.
4. **Một mô hình duy nhất, một nhiệt độ.** Chỉ dùng `openai:gpt-5.4-mini`, `temperature = 1.0` (mô hình mạnh: check kỹ thuật gần như luôn đạt). Ảnh hưởng: với mô hình yếu hơn, nhóm lỗi A-D có thể xuất hiện, và kết luận về subagent hay skill có thể khác. Hành vi "làm theo đề thay vì skill" (integer cents) cũng có thể phụ thuộc mô hình.
5. **Can thiệp của con người vào curator.** Nhóm đã sửa prompt curator một lần sau khi đọc skill lần 1 (mục 6), nên skill cuối không hoàn toàn là sản phẩm tự tiến hóa. Ảnh hưởng: kết quả `skills-auto` phản ánh "curator + một vòng hiệu chỉnh prompt của người", không phải curator nguyên bản.
6. **Sự cố hạ tầng.** Kết quả phải chạy lại nhiều lần (429, lỗi kết nối, lỗi khóa API, CRLF trên Windows, hai tiến trình chạy song song ghi đè 3 bản ghi). Mọi bản ghi trong bảng là lần chạy hoàn chỉnh, sau freeze, không lỗi, nhưng thứ tự chạy và giới hạn tốc độ có thể đã ảnh hưởng đến hành vi của tác tử (ví dụ `subagents/data-eval` kéo dài 135 s). Ngoài ra `trace.md` chỉ có luồng chính, nên không quan sát được bên trong subagent.

## 10. Kết luận

Với `gpt-5.4-mini`, tác tử mặc định đã đạt toàn bộ check kỹ thuật (18/18 ở cả tác vụ học và đánh giá); mọi thất bại còn lại là quy ước tổ chức không có trong đề (nhóm E). Skill do curator sinh từ phản hồi của tác vụ học tăng điểm tác vụ học (0,66 → 0,88), nhưng trên tác vụ đánh giá chỉ tăng 0,60 → 0,69, toàn bộ từ các quy ước tái sử dụng của một họ (`code`), trong biên độ nhiễu của một lần chạy; quy ước mới không được giúp (0/3), và skill log quá khớp nên không chuyển sang được. Đa tác tử tốn gấp đôi token mà không cải thiện, và trên tác vụ đánh giá còn gây thất bại do chép kết quả subagent mà không kiểm chứng. Đề xuất tiếp theo: chạy lại mỗi cấu hình ít nhất 3 lần để đo nhiễu, và sửa curator chỉ trích quy ước từ `RULE:` (không đọc đề bài) để giảm quá khớp.

## Phụ lục

- Lệnh đã chạy (theo thứ tự). Mọi lệnh chạy trong container `docker run --rm --env-file .env -v <repo>:/lab -w /lab lab-deepagents <lệnh>`; `verify_freeze` chạy trong image tương tự có cài `git`:
  1. `pytest` (32 test offline đạt), `python scripts/tour.py`
  2. `python -m lab.runner --condition baseline --tasks learn`
  3. `python -m lab.runner --condition subagents --tasks learn`
  4. `python -m lab.curator` (lần 1, xóa); sửa prompt curator; `python -m lab.curator` (lần 2, giữ)
  5. `python -m lab.runner --condition skills-auto --tasks learn`, rồi `mv results/skills-auto results/skills-auto-dev`
  6. `git commit -m "hypotheses"` (d22ebf5); `git commit --allow-empty -m "freeze skills" && git tag freeze` (97d87f9)
  7. `python -m lab.runner --condition baseline --tasks eval`; `... --condition subagents --tasks eval`; `... --condition skills-auto --tasks all`; chạy lại riêng `skills-auto` cho `data-learn`, `logs-learn`, `logs-eval`, `data-eval`
  8. `python scripts/verify_freeze.py` (OK); `python -m lab.compare > report/table.md`; `python scripts/check_breakdown.py`
- Thử thách mở rộng (nếu có): không thực hiện.
- Ghi chú khác (sự cố hạ tầng, không tính là lỗi của tác tử):
  - Ban đầu dùng `google_genai:gemini-3.8-flash` (free tier): 1 lần chạy `baseline data-learn` hết quota (429, 20 request/ngày) và bị loại; nhóm chuyển sang `openai:gpt-5.4-mini` **trước mọi lần chạy hợp lệ**. Mọi kết quả trong báo cáo dùng cùng một mô hình.
  - 6 lần chạy tác vụ học đầu tiên bị loại do `core.autocrlf=true` (mục 4). Đã đặt `core.autocrlf=false` và khôi phục `tasks/` cùng `skills/auto/README.md` từ git (chỉ đổi ký tự xuống dòng, không sửa nội dung).
  - Giới hạn 200.000 token/phút của tài khoản gây 4 lỗi 429 ở `skills-auto` (do hai tiến trình chạy song song, mục 7); sau đó có 1 lỗi kết nối, và 3 lần lỗi khóa API (401 khóa bị thu hồi; 403 project mới không có quyền dùng mô hình) khi đổi khóa. Các lần lỗi khóa tốn 0 token. Khóa API đã được thay giữa chừng (cùng mô hình, cùng cấu hình); khóa không ảnh hưởng đến hành vi của mô hình.
  - `verify_freeze.py` phải chạy trên Linux: `hash_dir` băm đường dẫn tương đối, mà Windows dùng `\` nên hash khác.
  - Không có khóa API nào trong kho, vết hay báo cáo (`.env` nằm trong `.gitignore`; đã quét `results/`, `report/`, `src/`, `skills/`).
