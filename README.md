# ỨNG DỤNG RAG GROUNDING & NEGATIVE CONSTRAINT
*Phòng chống ảo giác AI (Hallucination) trong nghiệp vụ Thu mua & Kiểm soát hợp đồng MartX*

---

## PHẦN 1: PHÂN TÍCH NGUYÊN NHÂN ẢO GIÁC & SO SÁNH PHƯƠNG PHÁP

### 1. Nguyên nhân AI tự tin đưa ra con số sai ("Phạt 20% và đình chỉ 30 ngày")
Bản chất của các mô hình ngôn ngữ lớn (LLM) như ChatGPT là **hệ thống dự đoán từ tiếp theo dựa trên xác suất thống kê** từ dữ liệu công khai trên Internet mà nó đã được huấn luyện. Khi nhân viên Thu mua đặt câu hỏi mà không cung cấp tài liệu hợp đồng thực tế của MartX:
* **Thiếu vùng neo dữ liệu (No Grounding):** AI không có quyền truy cập vào kho lưu trữ hợp đồng nội bộ hay điều khoản riêng của MartX.
* **Cơ chế "Bài thi học thuộc" và Ảo giác (Hallucination):** Thay vì thừa nhận mình không biết thông tin nội bộ, mô hình tự động lấp đầy khoảng trống bằng cách tổng hợp các điều khoản phạt thương mại phổ biến trên Internet (như phạt 20%, đình chỉ giao hàng 30 ngày...) rồi ghép thành một câu trả lời trôi chảy. Do được tối ưu hóa để diễn đạt tự nhiên và thuyết phục, AI thể hiện giọng văn cực kỳ tự tin dù số liệu hoàn toàn bịa đặt.

### 2. Phân biệt: Hỏi LLM không kèm tài liệu vs. RAG Grounding

| Tiêu chí so sánh | Hỏi LLM không kèm tài liệu (Zero-context) | RAG Grounding (Dán kèm tài liệu nguồn) |
| :--- | :--- | :--- |
| **Nguồn kiến thức AI sử dụng** | Dữ liệu huấn luyện chung trên Internet (đại trà, không chứa quy định nội bộ doanh nghiệp). | Tài liệu gốc được cung cấp trực tiếp trong câu lệnh (hợp đồng, quy chế, bảng giá MartX). |
| **Cách thức xử lý câu hỏi** | Dự đoán xác suất chuỗi từ sao cho nghe hợp lý và tự nhiên nhất. | Trích xuất, đối chiếu và tổng hợp thông tin nằm trong phạm vi văn bản được cấp. |
| **Rủi ro sai lệch (Hallucination)** | Rất cao — dễ tự bịa số liệu, áp đặt quy định của công ty khác sang MartX. | Rất thấp — bị khóa chặt vào dữ liệu thực tế và buộc từ chối nếu ngoài phạm vi tài liệu. |
| **Tính ứng dụng nghiệp vụ** | Chỉ phù hợp hỏi kiến thức phổ thông, viết nháp, lên ý tưởng chung. | Bắt buộc áp dụng cho tra cứu hợp đồng, quy trình CSKH, pháp lý và số liệu nội bộ. |

---

## PHẦN 2: CẤU TRÚC RAG PROMPT HOÀN CHỈNH

Dưới đây là mẫu Prompt chuẩn **RAG Grounding** tích hợp khóa an toàn **Negative Constraint** để nhân viên Thu mua MartX sử dụng trực tiếp:

```text
[VAI TRÒ & NHIỆM VỤ]
Bạn là trợ lý pháp lý và kiểm soát hợp đồng nội bộ của MartX. Nhiệm vụ của bạn là trả lời câu hỏi nghiệp vụ của nhân viên thu mua chỉ dựa trên nội dung văn bản hợp đồng được cung cấp dưới đây.

[1. VĂN BẢN HỢP ĐỒNG CHUẨN (NGUỒN DỮ LIỆU DUY NHẤT)]
"""
Điều 8. Quy định về chất lượng hàng hóa nông sản và chế tài vi phạm:
8.1. Nhà cung cấp cam kết giao hàng tươi sống đạt tiêu chuẩn chất lượng MartX.
8.2. Trường hợp kiểm tra đầu vào phát hiện tỷ lệ rau củ bị dập nát, hư hỏng vượt quá 10% tổng khối lượng lô giao: MartX áp dụng mức phạt 5% giá trị lô hàng bị dập nát, đồng thời yêu cầu nhà cung cấp thực hiện đổi trả bù đủ số lượng đạt chuẩn trong vòng 24 giờ kể từ lúc lập biên bản.
"""

[2. CÂU HỎI NGHIỆP VỤ]
Theo quy định MartX, nhà cung cấp giao rau bị dập nát trên 10% thì bị phạt bao nhiêu?

[3. RÀNG BUỘC AN TOÀN (NEGATIVE CONSTRAINT)]
- Chỉ được phép trích xuất thông tin có thật trong mục [1. VĂN BẢN HỢP ĐỒNG CHUẨN].
- Tuyệt đối không sử dụng kiến thức bên ngoài, không suy diễn thêm hình thức xử phạt nào khác.
- Nếu thông tin không có trong văn bản, tuyệt đối trả lời 'Không có dữ liệu trong tài liệu', không tự suy đoán.
```

---

### Kết quả phản hồi kỳ vọng từ AI sau khi áp dụng RAG Prompt:
> *"Theo Điều 8.2 trong văn bản hợp đồng được cung cấp, nếu nhà cung cấp giao rau bị dập nát vượt quá 10%, mức xử phạt là **phạt 5% giá trị lô hàng bị dập nát** và yêu cầu nhà cung cấp **đổi trả bù đủ số lượng đạt chuẩn trong vòng 24 giờ** kể từ lúc lập biên bản."*
