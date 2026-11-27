# Pi configuration for the Drone HAT

[`config.txt`](config.txt) enables every port on the [Raspberry Pi 4 Drone HAT Rev A](../../hardware/rpi4-drone-hat/rev-a-v1.0). Append it to `/boot/firmware/config.txt` and reboot.

| HAT port | Bus | GPIO | Linux device |
|---|---|---|---|
| J2 FC UART | UART0 (PL011) | 14 TX, 15 RX | `/dev/ttyAMA0` |
| J3 YDLiDAR X2 | UART3 RX + PWM0 | 5 RX, 18 M_CTR | `/dev/ttyAMA*`, `/sys/class/pwm/pwmchip0` |
| J8 UART5 | UART5 | 12 TX, 13 RX | `/dev/ttyAMA*` |
| J1 PCA9685, J6 | I2C1 | 2 SDA, 3 SCL | `/dev/i2c-1` |
| J7 | I2C4 | 6 SDA, 7 SCL | `/dev/i2c-*` (check `i2cdetect -l`) |
| J4 | SPI0 | 8 CE0, 9, 10, 11 | `/dev/spidev0.0` |
| J5 | SPI1 | 19, 20, 21, 24 CS | `/dev/spidev1.0` |
| J1 OE | GPIO22 | 22 | output, high = servos off |

Numbering of the extra UARTs (`ttyAMA1`, `ttyAMA2`...) depends on the order the overlays load. Check with `ls -l /dev/ttyAMA*` and `dmesg | grep tty`.

Quick checks after boot:

```bash
i2cdetect -y 1          # PCA9685 should answer at 0x40
ls /dev/spidev*         # spidev0.0 and spidev1.0
pinctrl get 5,6,7,12,13,18,22,24   # confirm pin functions (Raspberry Pi OS)
```
