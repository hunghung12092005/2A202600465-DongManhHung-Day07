# Vinlex Benchmark Queries

Domain: Vinlex - chatbot tra cuu chinh sach VinUni cho sinh vien

## Benchmark Set

| # | Query | Gold Answer | Source Document | Metadata Filter Suggestion |
|---|-------|-------------|-----------------|----------------------------|
| 1 | Sinh vien co the phuc khao diem trong bao lau sau khi diem duoc cong bo tren Canvas? | Sinh vien co the nop don phuc khao trong vong 05 ngay lam viec sau khi diem duoc cong bo tren Canvas. | `data2.md` | `{"category": "undergraduate_academic_policy"}` |
| 2 | Dieu kien toi thieu ve CGPA de sinh vien dai hoc duoc cong nhan tot nghiep la gi? | Sinh vien phai co diem trung binh tich luy toan khoa dat toi thieu 2.00/4.00 de duoc xet cong nhan tot nghiep. | `data2.md` | `{"audience": "undergraduate_students"}` |
| 3 | Hoc vien thac si can dap ung nhung dieu kien nao de duoc cong nhan tot nghiep? | Hoc vien phai hoan thanh cac hoc phan cua chuong trinh va bao ve luan van/de an dat yeu cau, dat chuan ngoai ngu dau ra, hoan thanh cac trach nhiem theo quy dinh cua Truong, khong bi truy cuu trach nhiem hinh su va khong trong thoi gian bi ky luat dinh chi hoc tap. | `data3.md` | `{"category": "graduate_academic_policy"}` |
| 4 | Neu sinh vien cho rang diem cuoi cung bi nhap sai hoac cham thien vi, buoc dau tien trong quy trinh grade appeal la gi? | Sinh vien phai lien he truc tiep voi giang vien ngay sau khi diem duoc cong bo de trao doi ve co so cham diem; neu chua duoc giai quyet thi moi tiep tuc appeal bang van ban. | `data4.md` | `{"category": "grading_and_appeals"}` |
| 5 | Theo quy dinh cua VinUni, complainant trong vu sexual misconduct co quyen bao cong an khong, va Truong co the ho tro gi? | Co. Complainant co quyen bao cong an, va neu ho lua chon bao cong an thi VinUni co the ho tro viec thuc hien police report khi phu hop. | `data1.md` | `{"category": "student_safety_policy"}` |

## Notes

- Query 1, 2 tap trung vao quy che dao tao dai hoc.
- Query 3 dung de phan biet tai lieu thac si voi tai lieu dai hoc, rat hop cho metadata filtering.
- Query 4 la query procedural, de do kha nang retrieve quy trinh tung buoc.
- Query 5 la query policy/safety bang tieng Anh, huu ich de test retrieval da ngon ngu.
