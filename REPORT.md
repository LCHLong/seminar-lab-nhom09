# Báo cáo lab Nhóm 09

**Họ tên:** Lê Chí Hoàng Long · **MSSV:** 23120294 · **JDK đã dùng** (kết quả `java -version`): OpenJDK 25.0.5.1 2026-11-17 LTS (Temurin-25.0.5.1+2-LTS)

## Bài 1: Giá trị biên

| Ngày trễ | Hạng | Số tiền bạn điền | Cách tính |
|---:|---|---:|---|
| 24 | NORMAL | 48000 | 24 ngày × 2.000đ = 48.000đ (chưa đạt trần 50.000đ) |
| 25 | NORMAL | 50000 | 25 ngày × 2.000đ = 50.000đ (chạm đúng trần tối đa 50.000đ) |
| 26 | NORMAL | 50000 | 26 ngày × 2.000đ = 52.000đ → áp dụng trần tối đa `Math.min(52000, 50000)` = 50.000đ |
| 2 | VIP | 0 | Thuộc thời gian ân hạn 2 ngày đầu (grace period), chưa tính phí |
| 3 | VIP | 1000 | Ngày thứ 3 (trễ 1 ngày sau ân hạn): (3 - 2) × 1.000đ = 1.000đ |
| 51 | VIP | 49000 | (51 - 2) ngày tính phí × 1.000đ = 49.000đ (chưa đạt trần 50.000đ) |
| 52 | VIP | 50000 | (52 - 2) ngày tính phí × 1.000đ = 50.000đ (chạm đúng trần tối đa 50.000đ) |
| 53 | VIP | 50000 | (53 - 2) ngày tính phí × 1.000đ = 51.000đ → áp dụng trần tối đa `Math.min(51000, 50000)` = 50.000đ |

**Vì sao chọn các ngày này mà không chọn ngày 10?**
Các ngày được chọn là các **giá trị biên (Boundary Values)** theo kỹ thuật Phân tích giá trị biên (Boundary Value Analysis - BVA):
- Với hạng `NORMAL`: Ngưỡng trần tối đa 50.000đ đạt được tại ngày 25 (25 × 2.000đ = 50.000đ). Bộ ba `{24, 25, 26}` đại diện cho: ngay dưới biên (chưa chạm trần), tại biên (vừa chạm trần), và ngay trên biên (vượt trần để kiểm tra logic `Math.min`).
- Với hạng `VIP`:
  + Biên thời gian ân hạn (grace period): ngày `{2, 3}` kiểm tra bước chuyển tiếp từ miễn phí (ngày 2 là ngày cuối ân hạn) sang bắt đầu tính phí (ngày 3).
  + Biên trần phí tối đa: số ngày trễ cần để chạm trần là `2 + (50.000 / 1.000) = 52`. Bộ ba `{51, 52, 53}` đại diện cho: ngay dưới biên (49.000đ), tại biên (50.000đ) và ngay trên biên (50.000đ).
- Trong khi đó, ngày 10 chỉ là giá trị danh định (nominal value) nằm giữa phân vùng tương đương, không kiểm tra được các điểm chuyển tiếp hành vi logic nơi lập trình viên dễ mắc lỗi off-by-one nhất.

## Bài 2a: Coverage

| | Branch coverage của `LoanService` |
|---|---|
| Trước khi viết test | 0.66 (66% - 4/6 nhánh) |
| Sau khi viết 2 test | 1.00 (100% - 6/6 nhánh) |

**Test `daDatHanMuc` của bạn dùng `countActiveByMemberId` trả về giá trị nào? Vì sao chọn giá trị đó?**
Test `daDatHanMuc` dùng giá trị **3L** (`when(loans.countActiveByMemberId(1)).thenReturn(3L);`).
Lý do chọn giá trị này: Theo đặc tả nghiệp vụ: *"Mỗi thành viên được mượn tối đa 3 cuốn: đang mượn 3 cuốn thì không được mượn thêm"*. Giá trị 3 chính là giá trị biên tối thiểu để kích hoạt điều kiện từ chối (`LIMIT_REACHED`). Sử dụng đúng giá trị biên 3L thay vì một giá trị lớn hơn (như 4L) giúp kiểm tra chính xác điều kiện biên và phát hiện được lỗi sai toán tử so sánh (`>` thay vì `>=`) trong mã nguồn.

## Bài 2b: Mutation testing

**Lần chạy PIT đầu tiên:** có bao nhiêu mutant sống sót (SURVIVED)? Nằm ở dòng nào, là loại mutant gì?
- Có **1 mutant sống sót (SURVIVED)** trên tổng số 9 mutants (tỷ lệ tiêu diệt 8/9, đạt 89%).
- Vị trí: Dòng 28 của `LoanService.java`: `if (loans.countActiveByMemberId(memberId) > 3)`.
- Loại mutant: `org.pitest.mutationtest.engine.gregor.mutators.ConditionalsBoundaryMutator` (thay đổi toán tử biên điều kiện từ `>` thành `>=`).

**Lỗi cài sẵn nằm ở đâu? Bạn phát hiện ra nó bằng cách nào?**
- Lỗi cài sẵn nằm tại **dòng 28 file `LoanService.java`**: sử dụng toán tử `> 3` thay vì `>= 3`.
- Cách phát hiện:
  + Báo cáo PIT lần đầu cho thấy mutant `ConditionalsBoundaryMutator` tại dòng 28 sống sót khi test dùng giá trị 4L (vì cả `4 > 3` và `4 >= 3` đều ra `true`).
  + Khi đối chiếu lại đặc tả: *"đang mượn 3 cuốn thì không được mượn thêm"*, ta thấy điều kiện `> 3` là sai logic vì khi `countActiveByMemberId == 3`, `3 > 3` trả về `false` và hệ thống vẫn cho phép mượn cuốn thứ 4.
  + Khi viết test `daDatHanMuc` dùng đúng giá trị biên 3L, test lập tức bị đỏ (AssertionError do không ném ngoại lệ như mong đợi).

**Bạn đã sửa test và code như thế nào?**

- **Đoạn test đã viết/sửa trong `LoanServiceTest.java`:**
```java
    // BÀI 2a: thành viên đang mượn đủ số cuốn tối đa thì bị từ chối với mã LIMIT_REACHED.
    @Test void daDatHanMuc() {
        availableBook();
        member(BigDecimal.ZERO);
        when(loans.countActiveByMemberId(1)).thenReturn(3L);
        assertCode(LIMIT_REACHED);
        verify(loans, never()).save(any());
        verify(books, never()).save(any());
    }

    // BÀI 2b: thành viên còn nợ phí thì bị từ chối với mã UNPAID_FINE.
    @Test void conPhiChuaTra() {
        availableBook();
        member(BigDecimal.valueOf(10_000));
        when(loans.countActiveByMemberId(1)).thenReturn(0L);
        assertCode(UNPAID_FINE);
        verify(loans, never()).save(any());
        verify(books, never()).save(any());
    }
```

- **Dòng code đã sửa trong `LoanService.java` (dòng 28):**
```java
        // Trước khi sửa:
        // if (loans.countActiveByMemberId(memberId) > 3) {
        
        // Sau khi sửa:
        if (loans.countActiveByMemberId(memberId) >= 3) {
            throw new LoanException(LIMIT_REACHED);
        }
```

**Kết quả PIT sau khi sửa** (Killed bao nhiêu / tổng bao nhiêu):
Killed **9 / 9** mutants (100% Mutation Coverage, toàn bộ 9 mutants đều bị tiêu diệt).

## Câu hỏi tổng kết
**Coverage 100% có nghĩa là code không còn lỗi không? Hãy giải thích dựa trên những gì bạn thấy trong lab.**
Không, Coverage 100% (cả Line coverage lẫn Branch coverage) **hoàn toàn không có nghĩa là code không còn lỗi**.
Bằng chứng qua quá trình thực hiện lab:
1. Ở Bài 2a, khi viết test `daDatHanMuc` với `countActiveByMemberId = 4`, JaCoCo đã báo cáo **Branch coverage đạt 100%** và build thành công gate coverage. Tuy nhiên, code vẫn chứa một lỗi logic nghiệp vụ nghiêm trọng là `> 3` thay vì `>= 3`, cho phép người đang mượn 3 cuốn vẫn mượn tiếp được cuốn thứ 4.
2. Code coverage chỉ đo lường xem các dòng lệnh/nhánh lệnh có được thực thi (execute) qua trong quá trình chạy test hay không, chứ không đo lường được tính đúng đắn của dữ liệu đầu vào (đặc biệt là giá trị biên) cũng như chất lượng của các câu lệnh assert.
3. Mutation testing đóng vai trò quan trọng trong việc đánh giá "sức mạnh" của bộ test suite (test strength) bằng cách giả lập các lỗi thường gặp của lập trình viên (đổi toán tử so sánh, phủ định điều kiện, trả về null, v.v.). Một bộ test thực sự tốt phải vừa đạt độ phủ cao vừa phải kiểm tra đúng giá trị biên để tiêu diệt được các mutant đột biến.
