# Review Rev A – Raspberry Pi 4 Drone HAT V1.0

Cập nhật: 27/11/2026. Đối chiếu với netlist `pstxnet.dat` / `pstxprt.dat` (15:24), board `RaspberryPi4DroneHat_REV_A.brd` (Import Logic 15:42, 0 thay đổi) và gói Gerber xuất lúc 14:26.

Trạng thái: **chưa gia công**. Gerber chưa được đưa lên repo.

## 1. Logic và nối chân: đúng

- **Chống ngược cực:** Q1 AO4407A (P-MOSFET). S (chân 1–3) nối `+BAT`, G (chân 4) nối GND qua R10 10k, D (chân 5–8) nối `IN` (sau F1).
- **Buck XL4015E1:**
  - D1/D3: anode nối GND, cathode nối `SWITCH1/2`.
  - Cuộn L nối từ SW ra ngõ ra.
  - Tụ VC (C27/C31) nối giữa VC và VIN, đúng theo datasheet.
  - Phân áp FB R1/R2 và R4/R5 cho ra 5.04 V.
- **RT9193-33:** chân 1 VIN, 2 GND, 3 EN, 4 BP, 5 VOUT. Đúng.
- **GPIO** (không chân nào bị trùng):

  | Chức năng | GPIO |
  |---|---|
  | UART0 | 14/15 |
  | UART3 RX (LiDAR) | 5 |
  | UART5 | 12/13 |
  | I2C1 | 2/3 |
  | I2C4 (+ pull-up 4.7k) | 6/7 |
  | SPI0 | CE0 (8) |
  | SPI1 | CS = GPIO24 |
  | M_CTR | 18 |
  | OE (kéo lên 10k) | 22 |

- **Cơ khí:**
  - Lỗ bắt vít 58 × 49 mm, Ø2.75.
  - Header pin 1 ở (8.37, 51.48), pin lẻ nằm hàng trong, bước 2.54.
  - XT30 bước 5 mm.
  - Board 65 × 56.5 mm, bo góc R3.

## 2. Phải sửa trước khi gia công

1. **L1/L2 sai mã.** `CDRH127/LDNP-221MC` là cuộn 220 µH, chỉ chịu khoảng 1 A. Cần đổi sang 22 µH, cỡ 12×12 mm, Isat ≥ 6 A.
2. **Mã tụ gốm không khớp giá trị:**
   - `CL05B104KO5NNNC` (0402, 100 nF/16 V) đang được dùng cho cả tụ 1 µF và 22 nF.
   - `0603YC104KAZ2A` (0603, 100 nF/16 V) đang được dùng cho tụ 10 µF.
3. **Điện áp định mức tụ trên `+BAT`:** C25, C28, C32 chỉ chịu 16 V, trong khi pin 4S sạc đầy là 16.8 V. Cần loại ≥ 25 V (khuyên dùng 50 V, ví dụ `GRM155R61H105KE05`). Giữ nguyên footprint 0402.
4. **Silkscreen top (GTO):** viền board Pi 85 × 56 mm vẽ tràn ra ngoài board (tới x = 85 mm). Cần xóa trước khi xuất Gerber.
5. **Gerber:** file `.DSN` được sửa lúc 15:51, sau lần xuất netlist. Cần tạo netlist lại, chạy Import Logic, sửa silkscreen, rồi mới xuất lại Gerber và NC Drill (tắt *Repeat codes*).

## 3. Nên sửa

- **Điện áp 5 V cho Pi:** R1/R4 3.3k cho ra 5.04 V. Đổi sang 3.24k sẽ được 5.11 V, đỡ báo undervoltage khi Pi tải nặng.
- **F1 `0679L1000-01`:** mã này có vẻ là bản 10 A, không khớp nhãn 15A. Cần kiểm tra lại.
- **Mã điện trở:** tất cả điện trở dùng chung một MPN `3-2176337-1`. Cần đặt đúng mã cho từng giá trị.
- **J2:** chân 2 là TX của Pi, nên cáp nối sang cổng TELEM của PX4 phải đảo TX/RX. RTS/CTS không được nối.
- **LiDAR TX:** điện trở 1k nối tiếp đã bị bỏ. Cần đo mức TX của X2 (3.3 V hay 5 V) trước khi cắm vào GPIO5.
- **USB-C:** chưa có chống cấp ngược. Không cắm USB-C khi đang cấp nguồn từ pin.
- **Điện áp pin tối đa 4S:** do tụ 25 V, TVS SMBJ18A và giới hạn VGS của Q1.
- **Layout:**
  - Nút SW dài, chỉ rộng 0.8 mm.
  - Dây `+5_MCU` có đoạn thắt 0.7 mm.
  - 8 đảo GND.
  - Vài đoạn dây treo ngắn trên `+5_SERVO`, `+3.3`, `+BAT`.
  - Net `SHIELD` (U1.S1–S4) chưa nối.
- **Gia công:**
  - Pad tab TO-263 đang phủ kem hàn 100%, nên chia ô còn khoảng 50–70%.
  - 19 via Ø0.33 mm chưa phủ solder mask.
  - Đồng cách mép board 0.1 mm.

## 4. Quyết định đã chốt

- Không thêm zener G–S cho Q1, vì chỉ dùng tối đa 4S.
- Không làm khe luồn cáp camera.
- Giữ khoảng cách đồng–mép board ~0.1 mm.
- Không thêm 2 × 10 µF/50 V 1206 ở VIN mỗi buck, vì board thiếu chỗ. Thay vào đó:
  - Đổi C14/C20 sang tụ hybrid polymer cùng kích thước vỏ (ví dụ `EEH-ZA1E221P`, cần kiểm tra case).
  - Khi có board: chạy tải 4–5 A trong 10 phút và đo nhiệt C14/C20. Nếu nóng hơn môi trường quá khoảng 15–20 °C thì phải thêm tụ gốm.
- 67 lỗi DRC đã được waive (route keepout, neck). Đã xem lại từng lỗi.
