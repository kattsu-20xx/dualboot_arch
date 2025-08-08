# Hướng Dẫn Dual Boot Arch Linux với Windows 11

## I. Chuẩn bị trên Windows 11

- Mở Disk Manager:
  - Nhấp chuột phải vào màn hình nền và mở **Terminal**.
  - Nhập lệnh:
    ```bash
    disk mgmt
    ```
    (để truy cập Disk Manager và thu nhỏ ổ đĩa cho Arch Linux).

---

## II. Trong môi trường Arch Linux Live (sau khi khởi động từ USB)

### Tăng kích thước console:
```bash
setfont ter-132n
```

### Kết nối Internet:

- **Nếu sử dụng Ethernet:**
  ```bash
  ping google.com
  ```

- **Nếu sử dụng Wi-Fi:**
  ```bash
  iwctl
  device list
  station wlan0 get-networks
  station wlan0 connect <Your_Wi-Fi_name>
  exit
  ping google.com
  ```

### Cập nhật keyring:
```bash
pacman -Syy
pacman -S archlinux-keyring
```

### Liệt kê ổ đĩa:
```bash
lsblk
```

### Phân vùng ổ đĩa:
```bash
cfdisk /dev/nvme0n1
```

Tạo:
- Phân vùng EFI (1GB)
- Phân vùng root (>=20GB)
- Phân vùng swap

---

## III. Định dạng và gắn kết các phân vùng

```bash
mkfs.fat -F 32 /dev/nvme0n1p5       # EFI
mkfs.ext4 /dev/nvme0n1p6            # Root
mkswap /dev/nvme0n1p7               # Swap
swapon /dev/nvme0n1p7
mount /dev/nvme0n1p6 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p5 /mnt/boot/efi
lsblk
```

---

## IV. Cài đặt hệ thống cơ sở Arch Linux

```bash
pacstrap -K /mnt base base-devel linux linux-firmware sudo nano git fastfetch networkmanager grub efibootmgr os-prober mtools dosfstools
genfstab -U /mnt >> /mnt/etc/fstab
```

---

## V. Vào hệ thống Arch Linux đã cài đặt (chroot)

```bash
arch-chroot /mnt
passwd
useradd -m -G wheel,audio,video,storage,network,power <your_name>
passwd <your_name>
EDITOR=nano visudo
# Bỏ ghi chú dòng:
# %wheel ALL=(ALL:ALL) ALL
```

---

## VI. Cấu hình hệ thống

### Thiết lập múi giờ:
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

### Ngôn ngữ hệ thống:
```bash
nano /etc/locale.gen        # Bỏ ghi chú dòng en_US.UTF-8 UTF-8
locale-gen
export LANG=en_US.UTF-8
```

### Đặt hostname:
```bash
echo arch > /etc/hostname
nano /etc/hosts
# Thêm:
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain arch
```

---

## VII. Cài đặt bộ quản lý khởi động (GRUB)

```bash
lsblk
mkdir -p /boot/efi/windows
mount /dev/nvme0n1p1 /boot/efi/windows
nano /etc/default/grub
# Bỏ ghi chú:
# GRUB_DISABLE_OS_PROBER=false

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## VIII. Các bước cuối cùng trong môi trường chroot

```bash
systemctl enable NetworkManager
exit
umount -R /mnt
shutdown now
```

---

## IX. Sau khi cài đặt (trong Arch Linux)

### Kết nối Wi-Fi:
```bash
sudo nmtui
# hoặc
sudo nmcli dev wifi connect <Wi-Fi_name> password "<password>"
ping google.com
```

### Cài HYPRLAND:
```bash
sudo pacman -S xorg xdg-desktop-portal xdg-desktop-portal-hyprland hyprland hyprpaper wofi thunar network-manager-applet pulseaudio pulseaudio-alsa
sudo systemctl enable gdm
reboot
```

---

## X. (Tùy chọn) Gỡ bỏ Arch Linux khỏi Dual Boot (từ Windows 11)

### Trong Command Prompt (Admin):

```bash
diskpart
list disk
select disk 0
list partition
select partition 5     # Phân vùng EFI của Arch
delete partition override
```

Sau đó dùng **Disk Management** để gộp phân vùng lại vào Windows.

---

**Chúc bạn cài đặt thành công Arch Linux song song Windows 11!**
