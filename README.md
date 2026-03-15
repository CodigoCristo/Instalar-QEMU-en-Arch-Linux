# 🖥️ Guía de configuración de QEMU/KVM — Arch Linux

Guía completa para configurar QEMU/KVM con aceleración 3D en Arch Linux, incluyendo los paquetes necesarios dentro de la máquina virtual para distintas distribuciones y Windows.

---

## 📦 1. Instalación en el Host (Arch Linux)

```bash
sudo pacman -S qemu-full qemu-emulators-full virt-manager virt-viewer \
               dnsmasq vde2 openbsd-netcat libvirt
```

### Habilitar y arrancar libvirt

```bash
sudo systemctl enable libvirtd
sudo systemctl start --now libvirtd
```

### Agregar tu usuario al grupo libvirt

```bash
sudo usermod -aG libvirt $(whoami)
newgrp libvirt
```

### Configurar la red virtual por defecto

```bash
sudo EDITOR=nano virsh net-edit default
sudo systemctl restart libvirtd
sudo virsh net-start default
sudo virsh net-autostart default
```

### Verificar instalación

```bash
qemu-system-x86_64 --version
sudo virsh net-list --all
virt-manager
```

---

### Puedes crear una imagen QEMU con tamaño dinámico (sparse/qcow2) que solo ocupa el espacio real usado

```bash
qemu-img create -f qcow2 mi-vm.qcow2 25G
qemu-img info mi-vm.qcow2
```

## ⚙️ 2. Configuración de la VM para Aceleración 3D

Antes de instalar los paquetes dentro de la VM, configura el hardware virtual correctamente en **virt-manager**:

1. En la configuración de la VM → **Video** → cambiar modelo a `virtio`
2. Marcar la casilla **"3D acceleration"**
3. En **Display** → usar `Spice` con `OpenGL` habilitado

> ⚠️ Si usas modelo `QXL` en lugar de `virtio`, instala `xf86-video-qxl` en lugar de los drivers virtio.

---

## 🐧 3. Paquetes dentro de la VM — Linux

### Paquetes necesarios

| Paquete | Función |
|---|---|
| `spice-vdagent` | Resolución dinámica, copiar/pegar entre host y VM |
| `qemu-guest-agent` | Integración con el host (snapshots, hora, etc.) |
| `virglrenderer` | Aceleración 3D via OpenGL (núcleo de la aceleración) |
| `libgl` | Biblioteca OpenGL |
| `libglvnd` | Dispatcher de OpenGL (permite múltiples implementaciones) |

---

### 🔵 Arch Linux / Manjaro

```bash
sudo pacman -S spice-vdagent qemu-guest-agent virglrenderer libgl libglvnd
```

```bash
sudo systemctl enable --now qemu-guest-agent.service
sudo systemctl enable --now spice-vdagentd.service
```

---

### 🟠 Ubuntu / Debian / Linux Mint / Pop!_OS

```bash
sudo apt update
sudo apt install spice-vdagent qemu-guest-agent virglrenderer-dev \
                 libgl1-mesa-dev libglvnd-dev libglvnd0
```

```bash
sudo systemctl enable --now qemu-guest-agent.service
```

> En Ubuntu 22.04+, `spice-vdagent` se inicia automáticamente en sesión gráfica.

---

### 🔴 Fedora / RHEL / CentOS Stream

```bash
sudo dnf install spice-vdagent qemu-guest-agent virglrenderer \
                 mesa-libGL mesa-libGLvnd
```

```bash
sudo systemctl enable --now qemu-guest-agent.service
```

---

### 🟢 openSUSE Tumbleweed / Leap

```bash
sudo zypper install spice-vdagent qemu-guest-agent virglrenderer \
                    libGL1 libglvnd
```

```bash
sudo systemctl enable --now qemu-guest-agent.service
```

---

### 🟡 Void Linux

```bash
sudo xbps-install -S spice-vdagent qemu-guest-agent virglrenderer \
                     libGL libglvnd
```

```bash
sudo ln -s /etc/sv/qemu-guest-agent /var/service/
```

---

### 🟣 NixOS

Agrega esto a tu `configuration.nix`:

```nix
services.qemuGuest.enable = true;
services.spice-vdagentd.enable = true;

environment.systemPackages = with pkgs; [
  virglrenderer
  mesa
  libglvnd
];
```

```bash
sudo nixos-rebuild switch
```

---

## 🪟 4. Paquetes dentro de la VM — Windows

Para Windows, los drivers e integración se instalan mediante dos paquetes oficiales:

### SPICE Guest Tools
Incluye: `spice-vdagent`, resolución dinámica, portapapeles compartido, redirección USB.

```
https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe
```

### Windows VirtIO Drivers
Incluye drivers para: disco (virtio-scsi), red (virtio-net), memoria balón, guest agent y más.

```
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/?C=M;O=D
```

> 💡 **Recomendación:** Descarga la ISO `virtio-win-*.iso` para tener todos los drivers disponibles durante la instalación de Windows, o el `.exe` para instalación post-instalación.

---

## ✅ 5. Verificación Final

Dentro de la VM Linux, verifica que la aceleración 3D esté activa:

```bash
# Verificar que virgl esté en uso
glxinfo | grep "OpenGL renderer"
# Debería mostrar algo como: "virgl" o "VIRGL"

# Verificar guest agent
systemctl status qemu-guest-agent.service

# Verificar spice-vdagent
systemctl status spice-vdagentd.service
```

---

## 📚 Referencias

- [Arch Wiki — QEMU](https://wiki.archlinux.org/title/QEMU)
- [Arch Wiki — KVM](https://wiki.archlinux.org/title/KVM)
- [Arch Wiki — Virt-Manager](https://wiki.archlinux.org/title/Virt-manager)
- [SPICE Project](https://www.spice-space.org/)
- [VirtIO Drivers (Fedora)](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/)
