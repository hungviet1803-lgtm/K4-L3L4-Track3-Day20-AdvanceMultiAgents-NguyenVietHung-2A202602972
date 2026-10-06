# Báo cáo Lab: Self evolving Agentic

> Sao chép tệp này thành `report/REPORT.md` (đã làm ở Phần 0) và điền dần qua các Phần của lab. Xóa các dòng hướng dẫn dạng trích dẫn (bắt đầu bằng `>`). Văn phong kỹ thuật, ngắn gọn, mọi nhận định đi kèm số liệu hoặc bằng chứng. Trong buổi học: điền mục 1 đến 7 (bản nháp). Sau buổi học: hoàn thiện mục 8 đến 10.

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| | | |

- Nhà cung cấp và mô hình (`LAB_MODEL`, không ghi khóa API), nhiệt độ (`LAB_TEMPERATURE`), `recursion_limit`: `openai:gpt-5.4-mini`, `LAB_TEMPERATURE=1.0`, `recursion_limit=60` (mặc định của runner).
- Phiên bản Deep Agents (`pip show deepagents`), hệ điều hành, chạy trực tiếp hay trong Docker: `deepagents 0.7.21`; máy chủ Windows 11, mọi lần chạy tác tử và test thực hiện trong Docker (image từ `Dockerfile` của lab, `python:3.12-slim`) vì shell của tác tử cần `/bin/sh`.
- Số lần chạy tác vụ đã dùng / ngân sách: xem Phụ lục (có ghi các lần chạy bị loại do lỗi hạ tầng).
- Commit của tag `freeze`:

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

> Dự đoán điều kiện nào đạt điểm cao nhất trên **tác vụ đánh giá** và vì sao. Nêu căn cứ từ phân loại lỗi (mục 4) và từ tài liệu tham khảo. Điền cả ba dòng; `verify_freeze.py` kiểm tra điều này.

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

> Ghi chú tính hợp lệ: lần chạy đầu tiên cho `code-learn` 6/10 với `tests_not_modified` thất bại dù vết không có lệnh ghi nào vào `tests/`. Nguyên nhân là `git core.autocrlf=true` trên Windows đã đổi `tasks/**` sang CRLF, làm sai hash SHA-256 mà `check.py` so sánh (hash tệp trên đĩa `efb5e765…` khác hash bản trong git `79e05f4c…`). Đây là lỗi hạ tầng: nhóm đã đặt `core.autocrlf=false`, khôi phục `tasks/` từ git (không sửa nội dung), loại bỏ cả 6 lần chạy đó và chạy lại. Bảng trên dùng các lần chạy sau khi sửa.

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

> Dán nội dung `report/table.md` và kết quả `python scripts/check_breakdown.py`. Nêu các lần chạy có `error` hoặc `skills_modified = true` (nếu có) và cách xử lý.

```text
(dán bảng ở đây)
```

## 8. Phân tích

> Trả lời từng câu bằng số liệu từ mục 7 và bằng chứng từ vết. Kết quả âm hoặc không có khác biệt vẫn hợp lệ nếu được phân tích tốt.

1. So với `baseline`, điều kiện nào cải thiện điểm tác vụ **học**? Điều kiện nào cải thiện điểm tác vụ **đánh giá**? Có điều kiện nào cải thiện tác vụ học nhưng không cải thiện tác vụ đánh giá? Nếu có, đó là dấu hiệu gì?
2. Tách điểm thành check kỹ thuật và check quy ước (`rule_`). Skill do curator sinh giúp nhóm check nào? Check quy ước **mới** của tác vụ đánh giá có được skill giúp không, và vì sao?
3. Dựa vào vết và `skills_read`, giải thích một check mà skill giúp đạt và một check mà skill không giúp (skill chưa được đọc, đọc nhưng không làm theo, skill thiếu hoặc sai).
4. Chi phí: so sánh số token trung bình giữa các điều kiện. Điều kiện nào có hiệu quả tốt nhất theo điểm trên mỗi token? Đa tác tử có đáng chi phí trong thí nghiệm này không?
5. Có dấu hiệu rò rỉ dữ liệu hoặc quá khớp nào trong skill sinh ra không? Nhóm đã phòng tránh như thế nào?
6. Nhiễu: so sánh điểm tác vụ học của cùng bộ skill ở Phần 3.4 (đã sao lưu) và sau đóng băng. Chênh lệch bao nhiêu? Nó cho biết điều gì về độ tin cậy của các chênh lệch trong bảng ở mục 7?

## 9. Hạn chế và tính hợp lệ

> Nêu ít nhất 3 hạn chế và ảnh hưởng của từng hạn chế đến kết luận (ví dụ: chỉ 3 tác vụ mỗi vai trò, mỗi cấu hình chạy một lần, nhiễu của mô hình, tác vụ do giảng viên thiết kế sẵn quy ước, chỉ một mô hình).

1.
2.
3.

## 10. Kết luận

> Tối đa 5 câu. Chỉ khẳng định điều số liệu hỗ trợ. Nêu một đề xuất cải tiến tiếp theo.

## Phụ lục

- Lệnh đã chạy (theo thứ tự):
- Thử thách mở rộng (nếu có): hướng chọn, kết quả, nhận xét.
- Ghi chú khác:
