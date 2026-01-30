# Servidor SSH / SFTP con Autenticación Centralizada LDAP

Implementación de un servidor SSH/SFTP seguro con autenticación centralizada mediante OpenLDAP, aplicando principios de **mínimo privilegio**, **zero trust** y **segregación de roles**.

El proyecto está orientado a entornos Linux y documenta tanto la arquitectura como las decisiones de seguridad adoptadas.

---

## 🎯 Objetivo del Proyecto

Diseñar e implementar una infraestructura de acceso remoto segura que permita:

- Acceso administrativo completo vía SSH para personal autorizado.
- Acceso restringido vía SFTP para usuarios operativos.
- Autenticación centralizada de usuarios mediante LDAP.
- Aislamiento de usuarios SFTP mediante Chroot Jail.
- Gestión de permisos basada en grupos.
- Automatización de la sincronización de accesos.

---

## 🏗️ Arquitectura General

- **Servidor LDAP** como fuente única de verdad para usuarios y grupos.
- **Servidor SSH/SFTP** integrado con LDAP mediante NSS y NSLCD.
- Separación de roles:
  - Administradores: acceso SSH completo + sudo.
  - Usuarios SFTP: acceso exclusivo a transferencia de archivos, sin shell.

📌 *(El diagrama de arquitectura se encuentra en la documentación del proyecto)*

---

## 🔐 Seguridad y Hardening

### LDAP
- Deshabilitación de consultas anónimas (anonymous bind).
- Creación de cuenta de servicio con privilegios mínimos.
- Implementación de ACLs granulares:
  - Root DN: gestión total.
  - Service account: lectura y búsqueda.
  - Usuarios: modificación de su propia contraseña.
  - Otros: solo autenticación.

### SSH / SFTP
- Uso de `Match Group` para aplicar políticas específicas.
- `ForceCommand internal-sftp` para impedir acceso a shell.
- Chroot Jail por usuario.
- Deshabilitación de X11 y TCP forwarding.
- Prohibición de login directo como root.
- Acceso administrativo mediante sudo y usuarios nominales.

---

## 👥 Gestión de Usuarios

### Acceso SFTP
- Control basado en grupo LDAP `SFTPUsers`.
- Alta/baja de acceso mediante membresía al grupo.
- Sin gestión manual usuario por usuario en el servidor.

### Automatización
- Script `Actualizar.sh`:
  - Consulta LDAP.
  - Sincroniza usuarios autorizados.
  - Revoca accesos no permitidos.
- Ejecución periódica mediante `crontab`.

---

## 📁 Entorno Chroot SFTP

Estructura segura por usuario:

- Directorio raíz propiedad de `root`.
- `/upload`: escritura controlada (sticky bit).
- `/download`: solo lectura para el usuario.

Cumple con los requisitos de seguridad de OpenSSH para entornos chroot.

---

## 🧪 Pruebas Realizadas

- Conexión SFTP válida desde cliente (FileZilla).
- Verificación de aislamiento del entorno.
- Pruebas de carga y descarga de archivos.
- Negative testing: rechazo de usuarios no autorizados.
- Acceso SSH administrativo con privilegios sudo.

---

## 🛠️ Tecnologías Utilizadas

- Linux
- OpenSSH
- OpenLDAP
- NSS / NSLCD
- Bash scripting
- Crontab

---

## 📄 Documentación

La documentación técnica completa del proyecto se encuentra en la carpeta `Docs`, donde se detalla paso a paso la implementación, configuración y pruebas realizadas.

---

## 📌 Estado del Proyecto

✔ Implementación funcional  
✔ Seguridad validada  
✔ Automatización operativa  
🟡 Posibles mejoras futuras: Ansible, monitoreo, logging centralizado