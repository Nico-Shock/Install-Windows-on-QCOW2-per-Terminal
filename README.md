# Manuelle Windows 11 Installation mit QEMU

# KEIN BOCK DEN SCRIPT WEITER ZU MACHEN, WER WILL KANN DEN CODE WEITERFÜHREN ODER ANDEREN STUFF DAMIT MACHEN

# SCRIPT WURDE VON CHATGPT GESCHRIEBEN!!!

Dieses Dokument beschreibt die Schritte zur manuellen Installation von Windows 11 auf einem QEMU-virtuellen Computer mit UEFI-Unterstützung, TPM 2.0 und Secure Boot.

## Voraussetzungen

Stelle sicher, dass du die folgenden Pakete installiert hast:

- `qemu`
- `qemu-kvm`
- `wimlib`
- `gdisk`
- `dosfstools`
- `ntfs-3g`
- `cfdisk`

Du kannst die erforderlichen Pakete mit dem folgenden Befehl installieren:

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y qemu qemu-kvm wimlib dosfstools ntfs-3g

# Fedora
sudo dnf install -y qemu-kvm wimlib dosfstools ntfs-3g

# Arch Linux
sudo pacman -Syu --noconfirm qemu wimlib dosfstools ntfs-3g
```

## Schritt-für-Schritt-Anleitung

1. **ISO-Datei herunterladen**:
   Lade die Windows 11 ISO-Datei von der offiziellen Microsoft-Website herunter und speichere sie im Verzeichnis `/pfad/zu/deiner/datei/Windows 11/`. Stelle sicher, dass der Dateiname `Win11_24H2_German_x64.iso` ist.

2. **QCOW2-Datei erstellen**:
   Erstelle eine QCOW2-Datei für die virtuelle Festplatte:

   ```bash
   qemu-img create -f qcow2 windows.qcow2 70G
   ```

3. **QCOW2-Datei mit NBD verbinden**:
   Lade das NBD-Modul und verbinde die QCOW2-Datei:

   ```bash
   sudo modprobe nbd
   sudo qemu-nbd --connect=/dev/nbd0 /pfad/zu/deiner/datei/windows.qcow2
   ```

4. **Partitionen erstellen mit cfdisk**:
   Starte `cfdisk`, um die QCOW2-Datei zu partitionieren:

   ```bash
   sudo cfdisk /dev/nbd0
   ```

   - Erstelle eine **EFI-Partition**:
     - Wähle „New“ und gib 512 MB für die Größe an.
     - Setze den Typ auf „EFI System“.
   - Erstelle eine **Windows-Partition**:
     - Wähle „New“ und verwende den verbleibenden Speicherplatz.
     - Setze den Typ auf „Microsoft Basic Data“.
   - Speichere die Änderungen und beende `cfdisk`.

5. **Partitionen formatieren**:
   Formatiere die EFI-Partition und die Hauptpartition:

   ```bash
   sudo mkfs.fat -F32 /dev/nbd0p1
   sudo mkfs.ntfs -f /dev/nbd0p2
   ```

6. **Verzeichnisse erstellen**:
   Erstelle Verzeichnisse zum Mounten der ISO und der Partitionen:

   ```bash
   sudo mkdir -p /mnt/iso /mnt/efi /mnt/windows
   ```

7. **ISO-Datei mounten**:
   Mounte die Windows 11 ISO-Datei:

   ```bash
   sudo mount -o loop /pfad/zu/deiner/datei/Win11_24H2_German_x64.iso /mnt/iso
   ```

8. **Partitionen mounten**:
   Mounte die EFI- und Windows-Partition:

   ```bash
   sudo mount /dev/nbd0p1 /mnt/efi
   sudo mount /dev/nbd0p2 /mnt/windows
   ```

9. **Windows-Dateien extrahieren**:
   Übertrage die Windows-Dateien auf die Hauptpartition:

   ```bash
   sudo wimapply /mnt/iso/sources/install.wim 1 /mnt/windows
   ```

10. **Bootloader installieren**:
    Installiere den Windows-Bootloader in der EFI-Partition:

    ```bash
    sudo mkdir -p /mnt/efi/EFI/Microsoft/Boot
    sudo cp -r /mnt/windows/Windows/Boot/EFI/* /mnt/efi/EFI/Microsoft/Boot
    ```

11. **Unmounten**:
    Unmount die Partitionen und die ISO:

    ```bash
    sudo umount /mnt/iso /mnt/efi /mnt/windows
    sudo qemu-nbd --disconnect /dev/nbd0
    ```

12. **Virtuelle Maschine starten**:
    Starte die virtuelle Maschine mit den entsprechenden Optionen:

    ```bash
    sudo qemu-system-x86_64 \
      -enable-kvm \
      -m 4096 \
      -smp cores=2 \
      -vga virtio \
      -display sdl,gl=on \
      -drive file=/pfad/zu/deiner/datei/windows.qcow2,format=qcow2,if=virtio \
      -usbdevice tablet \
      -cdrom /pfad/zu/deiner/datei/Win11_24H2_German_x64.iso \
      -net nic,model=virtio \
      -net user
    ```
13. **Optional: QCOW2 in VMDK Konvertieren**:

```
qemu-img convert -f qcow2 -O vmdk windows.qcow2 windows.vmdk
```

## Wichtige Hinweise

- Stelle sicher, dass die Virtualisierung im BIOS aktiviert ist (Intel VT-x oder AMD-V).
- Bei Problemen mit dem Booten aus der virtuellen Maschine, überprüfe die Einstellungen in deinem Skript sowie die QEMU-Dokumentation für weitere Hinweise.
