# Linux Storage - Onder andere RAID, ext3, ext4. 

> Geschreven door: Faisal Anwari

---

## Schijven & partities bekijken
```bash
lsblk				# Simpel display
lsblk -o +FSTYPE                # Aanbevolen
sudo fdisk -l                   # Gedetailleerd display
sudo blkid                      # Handig om UUID te vinden
```

---

## Partities aanmaken
```bash
sudo cfdisk /dev/(partitie)
sudo fdisk /dev/(partitie)

# In fdisk:
# n → nieuwe partitie
# t → type wijzigen (fd = Linux RAID autodetect)
# w → opslaan en afsluiten
```

---

## Filesystems aanmaken
```bash
sudo mkfs.ext2 /dev/(partitie) 	# ext2 filesystem
sudo mkfs.ext3 /dev/(partitie) 	# ext3 filesystem
sudo mkfs.ext4 /dev/(partitie) 	# ext4 filesystem
sudo mkfs.xfs  /dev/(partitie) 	# xfs filesystem
```

---

## Mounten & unmounten
```bash
sudo mkdir -p /data/(map)			# Altijd kijken of folder bestaat waar je wilt mounten.
sudo mount -v /dev/(partitie) /data/(map)	# mounten
sudo umount /data/(map)
df -h                          		   	# Schijfruimte bekijken
mount                             		# Alle gemounte schijven tonen
sudo mount -a            		        # Alles uit /etc/fstab mounten
```

### Permanent mounten via /etc/fstab

```bash
sudo nano /etc/fstab

/dev/(partitie)  /data/(map)  (Filesystem: ext2,ext3,xfs)  defaults  0  2
```
---

## Filesystem controleren
```bash
sudo umount /data/(map)		# Altijd eerst unmounten
sudo fsck /dev/(partitie)	# Filesystem checker
```

---

## LVM
```bash
# Physical volume
sudo pvcreate /dev/(partitie of raid)	# Aanmaken van volume
sudo pvdisplay				# Volumes inzien

# Volume group
sudo vgcreate (vgnaam) /dev/(partitie)
sudo vgdisplay (vgnaam)

# Logical volumes
sudo lvcreate -L 2G -n (lvnaam) (vgnaam)	# 2G houdt opslag in. Kan ook met MB, KB etc.
sudo lvdisplay
sudo lvextend -L+1G /dev/(vgnaam)/(lvnaam)	# Vergroten van een volume
sudo resize2fs /dev/(vgnaam)/(lvnaam)    # Filesystem meegroeien
```

---

## RAID (mdadm)
```bash
# RAID array aanmaken
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 \
  /dev/(partitie1) /dev/(partitie2)

# Status monitoren 
watch cat /proc/mdstat
sudo mdadm --detail /dev/md0
sudo mdadm --examine /dev/(partitie1) /dev/(partitie2)

# mdadm.conf opslaan (persistentie na reboot)
sudo mkdir /etc/mdadm
sudo touch /etc/mdadm/mdadm.conf
echo "DEVICE /dev/(partitie1) /dev/(partitie2)" | sudo tee /etc/mdadm/mdadm.conf
mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf

# Disk laten falen / verwijderen / toevoegen
sudo mdadm /dev/md0 -f /dev/(partitie)    # Simuleer storing
sudo mdadm /dev/md0 -r /dev/(partitie)    # Verwijder schijf uit array
sudo mdadm /dev/md0 -a /dev/(partitie)    # Voeg schijf toe aan array
```

---

## Dump & Restore (backup)
```bash
# Backup maken
dump -0u -f ~/backup.0  /data/(map)    # Level 0 (volledig)
dump -1u -f ~/backup.1  /data/(map)    # Level 1 (wijzigingen)

# Restore
mkdir ~/restore && cd ~/restore
restore -f ~/backup.0 -x (bestandsnaam)

# Backup verwijderen
rm ~/backup.0
```

---

## Logs bekijken
```bash
sudo grep 'md[0-9]' /var/log/messages
sudo tail -n 50 /var/log/messages | grep 'md[0-9]'
```

---

## Packages installeren
```bash
sudo dnf install lvm2 dump mdadm
```
