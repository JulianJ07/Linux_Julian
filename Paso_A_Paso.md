# Guia segura para instalar CachyOS junto a Windows

## Objetivo

Instalar Linux en dual boot con Windows, usando CachyOS KDE Plasma como sistema base, dejando aproximadamente 300 GB para Linux.

PC objetivo:

- CPU: Intel Core i5-12600KF
- RAM: 32 GB
- GPU: NVIDIA RTX 4060
- Disco: SSD NVMe de 1 TB
- Windows ya instalado en `C:`

## Recomendacion principal

Usar **CachyOS Desktop Edition + KDE Plasma + drivers NVIDIA**.

No recomiendo instalar Omarchy/Hyprland desde el primer dia como sistema principal. Primero instala CachyOS KDE, verifica que Windows y Linux arrancan bien, y despues pruebas Hyprland/Omarchy si quieres ese flujo mas personalizado.

## Links oficiales

- CachyOS Desktop ISO: https://cachyos.org/download/
- Guia oficial de instalacion CachyOS: https://wiki.cachyos.org/installation/installation_on_root/
- Preparacion de USB CachyOS: https://wiki.cachyos.org/installation/installation_prepare/
- Verificacion SHA256 CachyOS: https://wiki.cachyos.org/cachyos_basic/download/
- Rufus: https://rufus.ie/en/
- Ventoy: https://ventoy.net/en/download.html
- Omarchy, opcional despues: https://omarchy.org/

## Punto importante: no tengo USB

Para tu caso, **si necesitas una USB**.

Windows solo te deja reducir `C:` unos 6677 MB porque el archivo que bloquea la reduccion es la MFT de NTFS (`$Mft`). Eso no se debe borrar ni forzar desde Windows. Para mover estructuras internas de NTFS con menos riesgo, hay que arrancar desde fuera de Windows usando un medio externo.

Recomendacion:

- Compra o pide prestada una USB de minimo 8 GB.
- Mejor si es de 16 GB o 32 GB.
- Todo lo que haya en esa USB se borrara al prepararla.

No recomiendo trucos de instalar desde el mismo disco interno porque aumentan el riesgo de romper el arranque o quedarte sin Windows ni Linux.

## Antes de tocar particiones

1. Haz copia de seguridad de tus archivos importantes.
2. Conecta el PC a corriente estable.
3. En Windows, abre PowerShell como administrador y ejecuta:

```powershell
powercfg /h off
manage-bde -status
```

4. En `manage-bde -status`, verifica que BitLocker este desactivado o que el metodo de cifrado diga `None`.
5. Reinicia Windows una vez.
6. Entra a la BIOS/UEFI y desactiva:

- Secure Boot
- CSM / Legacy Boot, si aparece

CachyOS recomienda instalar en modo UEFI, con Secure Boot y CSM desactivados.

## Crear la USB de instalacion

Opcion recomendada: Rufus.

1. Descarga CachyOS Desktop Edition desde: https://cachyos.org/download/
2. Descarga Rufus desde: https://rufus.ie/en/
3. Conecta la USB.
4. Abre Rufus.
5. En `Device`, selecciona tu USB.
6. En `Boot selection`, selecciona la ISO de CachyOS.
7. Usa esquema de particion `GPT` y destino `UEFI`.
8. Haz clic en `Start`.
9. Si Rufus pregunta modo ISO o DD, prueba primero modo ISO. Si falla el arranque, repite en modo DD.

Alternativa: Ventoy.

1. Descarga Ventoy: https://ventoy.net/en/download.html
2. Instala Ventoy en la USB.
3. Copia la ISO de CachyOS dentro de la USB.
4. Arranca desde la USB y elige la ISO.

## Arrancar desde la USB

1. Reinicia el PC.
2. Entra al menu de arranque de ASUS. Normalmente es `F8`, aunque tambien puede ser `Esc` o `F12`.
3. Elige la USB en modo `UEFI`.
4. En el menu de CachyOS, elige la opcion con NVIDIA/proprietary drivers si aparece.
5. Espera a entrar al escritorio live.

## Reducir Windows de forma segura

Dentro del live de CachyOS:

1. Abre `GParted` o `KDE Partition Manager`.
2. Identifica tu disco NVMe principal.
3. Veras algo parecido a:

```text
EFI 100 MB
C: NTFS ~918 GB
Recovery ~821 MB
D: NTFS pequeño
Espacio no asignado ~8 GB
```

4. Selecciona solo la particion grande de Windows (`C:` / NTFS).
5. Elige `Resize/Move`.
6. Reduce el tamaño para dejar aproximadamente `300 GiB` sin asignar.

Referencia:

```text
300 GiB = 307200 MiB
```

7. El espacio libre debe quedar como `unallocated` / `sin asignar`.
8. No formatees ese espacio desde Windows.
9. No toques:

- Particion EFI de 100 MB
- Particion Recovery
- La particion `C:`
- La particion `D:` salvo que decidas borrarla conscientemente

10. Aplica cambios y espera. Puede tardar bastante.

## Instalar CachyOS

1. Abre el instalador de CachyOS.
2. Elige idioma, zona horaria y teclado.
3. Cuando pregunte particionado, elige **Manual partitioning**.

La wiki oficial de CachyOS recomienda particionado manual porque `Install alongside` y `Replace partition` no son 100% fiables.

## Esquema de particiones recomendado

Usa el espacio `sin asignar` de 300 GiB.

Opcion simple y recomendada:

```text
/boot       4096 MiB   FAT32   flag boot
/           resto      Btrfs
swap        opcional
```

Notas:

- CachyOS recomienda `/boot` de al menos 4096 MiB para algunos gestores de arranque.
- Para `/`, usa Btrfs si quieres snapshots.
- Con 32 GB de RAM no necesitas swap grande para uso normal.
- Si quieres hibernar Linux, necesitarias swap igual o mayor que la RAM, pero no lo recomiendo al inicio.

Opcion aun mas simple:

```text
/boot       4096 MiB   FAT32   flag boot
/           todo lo demas Btrfs
```

## Que seleccionar en el instalador

- Desktop Environment: KDE Plasma
- Filesystem: Btrfs
- Drivers: NVIDIA / nvidia-open si aparece
- Bootloader: deja el recomendado por CachyOS
- Usuario: crea tu usuario normal
- Revisa el resumen antes de instalar

Antes de hacer clic en instalar, confirma:

- Windows `C:` no se va a formatear
- EFI de Windows no se va a borrar
- Solo se usara el espacio que dejaste sin asignar

## Primer reinicio

1. Termina la instalacion.
2. Apaga o reinicia.
3. Quita la USB cuando el sistema lo indique.
4. Entra primero a CachyOS.
5. Despues reinicia y prueba entrar a Windows.

Si Windows no aparece en el menu de arranque:

- No entres en panico.
- Usa el menu de arranque de ASUS para elegir `Windows Boot Manager`.
- Despues se puede reparar la deteccion desde CachyOS.

## Verificaciones en CachyOS

Abre terminal:

```bash
nvidia-smi
```

Si muestra tu RTX 4060, el driver NVIDIA esta funcionando.

Actualiza el sistema:

```bash
sudo pacman -Syu
```

Instala herramientas utiles:

```bash
sudo pacman -S git base-devel python python-pip nodejs npm
```

## Instalar Codex despues

Codex CLI esta disponible para Linux. Despues de tener CachyOS funcionando, puedes instalarlo con:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

Luego inicia sesion y prueba:

```bash
codex
```

Para tu idea de sistema con IA, empieza con permisos seguros:

```text
Fase 1: Codex solo trabaja en carpetas personales.
Fase 2: Codex organiza Downloads/Documents con confirmacion.
Fase 3: Codex ejecuta comandos del sistema solo con aprobacion.
Nunca: root automatico permanente.
```

## Despues, si quieres Omarchy

Cuando CachyOS KDE ya funcione bien:

1. Haz snapshot del sistema.
2. Prueba Hyprland desde CachyOS Settings o instalacion separada.
3. Investiga Omarchy: https://omarchy.org/
4. No mezcles demasiados escritorios el primer dia.

## Checklist final

- [ ] Backup hecho
- [ ] USB de 8 GB o mas conseguida
- [ ] CachyOS ISO descargada
- [ ] ISO verificada con SHA256
- [ ] Rufus o Ventoy preparado
- [ ] Secure Boot desactivado
- [ ] CSM/Legacy desactivado
- [ ] Fast Startup/Hibernation desactivado
- [ ] BitLocker desactivado
- [ ] `C:` reducido desde live USB
- [ ] 300 GiB dejados como espacio sin asignar
- [ ] Instalacion en particionado manual
- [ ] Windows no formateado
- [ ] CachyOS arranca
- [ ] Windows arranca
- [ ] `nvidia-smi` funciona
- [ ] Sistema actualizado

