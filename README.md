# NOTAS Linux
Cuaderno de notas LINUX

- [LVM General](#lvm-general)
    - [Snapshot Recovery Process](#snapshot-recovery-process)
    - [Restaurar LVM Eliminado](#restaurar-lvm-eliminado)

## LVM General
### Snapshot Recovery Process
En este procedemiento se toma un snapshot a una de los logical volumen para proceder a actualizar el sistema operativo y si algo sale mal
se procede a revertir.
  
- Overview
  - Preparar crear snapshot
  - Opcional para revisar contenido
  - Revertir
  - Editar Menu Grub
  - Devolver disco

  - Preparar crear snapshot
	- ```vgs```
	- ```lvs```
	- ```vgextend vgroot /dev/sdb```
	- ```lvcreate -s -n snaproot -L400M vgroot/lvroot```
	- ```lvcreate -s -n snapusr -L100M vgroot/lvusr```
	- ```yum update -y```
	- ```mkdir /snaproot```
	- ```mkdir /snapusr```

  - Opcional para revisar contenido
	- ```mount /dev/vgroot/snaproot /snaproot/ -o nouuid,ro```
	- ```mount /dev/vgroot/snapusr /snapusr/ -o nouuid,ro```
	- ```umount /snaproot/```
	- ```lvremove /dev/vgroot/snaproot```

  - Revertir
    - ```lvconvert --merge vgroot snaproot```
	- ```lvconvert --merge vgroot snapusr```
	- ```reboot```

  - Editar Menu Grub
	- ```grep menuentry /boot/grub2/grub.cfg ```
	- ```vim /etc/default/grub   --> GRUB_DEFAULT=0```
	- ```grub2-mkconfig -o /boot/grub2/grub.cfg ```
	- ```reboot```

  - Devolver disco
	- ```vgreduce vgroot /dev/sdb```
	- ```vgs```
	- ```pvremove /dev/sdb```
	- ```pvs```


### Restaurar LVM Eliminado
Se realiza una recuperacion de un logical volume elimanado accidentalmente.
  - Simulacion de borrado de un logical volume
	- ```lvremove /dev/vgtest/lvtest```
	- ```lvs```
	- Verificar por fecha
```
		[root@localhost /]# ls -la /etc/lvm/archive/
		total 16
		drwx------. 2 root root  144 Apr  3 00:49 .
		drwxr-xr-x. 6 root root  100 Apr  3 00:13 ..
		-rw-------. 1 root root 1738 Apr  3 00:28 almalinux_00000-455080098.vg
		-rw-------. 1 root root  897 Apr  3 00:41 vgtest_00000-2097528995.vg
		-rw-------. 1 root root  910 Apr  3 00:42 vgtest_00001-1942101961.vg
		-rw-------. 1 root root 1338 Apr  3 00:49 vgtest_00002-2052112883.vg
```
	- Tener en cuenta el que dice **before**
	- ```vgcfgrestore --list vgtest```
```		
		File:         /etc/lvm/archive/vgtest_00002-2052112883.vg
		VG name:      vgtest
		Description:  Created *before* executing 'lvremove /dev/vgtest/lvtest'
		Backup Time:  Sat Apr  3 00:49:06 2021
```
	- ``` vgcfgrestore -f /etc/lvm/archive/vgtest_00002-2052112883.vg vgtest```
	- ``` lvscan```
	- ``` lvchange -ay /dev/vgtest/lvtest```
	- ``` lvscan```
	- ``` lvs```
	- ``` mount -a```