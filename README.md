# Gestión del Ciclo de Vida de Identidades y Control de Acceso Dinámico (JML) en Microsoft Entra ID
## Objetivo
Implementar un modelo de **Control de Acceso Basado en Atributos (ABAC)** dentro de Microsoft Entra ID para automatizar el ciclo de vida de las identidades de un usuario: **Joiner, Mover y Leaver (JML)**.

utilizamos reglas de membresía dinámica en grupos de seguridad, eliminando la asignación manual de permisos y la acumulación de privilegios (**Privilege Creep**). Tambien aplicamos controles de seguridad estrictos para la revocación inmediata de sesiones y el bloqueo de cuentas durante la baja de colaboradores.

## Arquitectura Técnica y Alcance

- **Proveedor de Identidad:** Microsoft Entra ID
- **Modelo de Acceso:** Control de Acceso Basado en Atributos (ABAC)
- **Operaciones Principales:**
  - **Fase Joiner (Alta):** Aprovisionamiento automático de accesos basado en metadatos dentro perfil del usuario (`department`).
  - **Fase Mover (Transición):** Reasignación dinámica de grupos para garantizar la evolución de permisos sin acumulación de permisos o accesos.
  - **Fase Leaver (Baja):** Bloqueo de inicio de sesión, invalidación global de tokens de sesión y expulsión automatizada de los grupos.

---

## Prerrequisitos y Configuración de Infraestructura Antes de ejecutar las pruebas del ciclo de vida

Se configuró la estructura de grupos dinámicos basada en el modelo ABAC dentro de Microsoft Entra ID: 

- **Grupos de Seguridad Creados:**
  - `Depto. Ventas`
  - `Depto. IT`
<img width="1389" height="381" alt="Groups" src="https://github.com/user-attachments/assets/aa88a7e0-5562-4de1-a481-85b735cb341c" />

- **Tipo de Membresía:** Dynamic User   
- **Regla de Membresía Dinámica:**
  
- ```text
  (user.department -eq "Ventas")
<img width="1581" height="449" alt="Regla Ventas" src="https://github.com/user-attachments/assets/88261759-51f8-4c8f-8a1d-95084c2fc69c" />

- ```text
  (user.department -eq "IT")
<img width="1573" height="446" alt="Reglas IT" src="https://github.com/user-attachments/assets/69dc6cf0-d28f-4f90-9f9d-69429b08eaf6" />

---

## Implementación y Fases del Ciclo de Vida

### 1. Fase Joiner (Onboarding Automatizado)
- **Mecanismo:** Creacion del usuario y Evaluación de reglas de membresía dinámica basadas en el atributo del perfil del usuario (`user.department -eq "Ventas"`).
- **Comportamiento:** Al aprovisionar la cuenta y configurar `Department = Ventas`, Microsoft Entra ID agrega automáticamente la identidad al grupo `Depto. Ventas` sin intervención administrativa.
- **Objetivo de Seguridad:** Garantizar permisos base estandarizados mediante automatización sin intervención manual (*zero-touch*).

### Creación de la cuenta y Asignación de Atributos 

- Generamos el usuario con sus respectivas credenciales y datos. Luego, vamos al apartado `Properties`
<img width="647" height="525" alt="image" src="https://github.com/user-attachments/assets/dfb6a54a-4b23-4458-98b9-1a79a5037ca0" />

- Especificamos el atributo "Ventas" dentro del campo  `Department`
<img width="864" height="638" alt="Captura de pantalla 2026-10-15 175401" src="https://github.com/user-attachments/assets/c818d012-a98d-4928-9f56-183992cfefe0" />

### Verificación

- Al mirar el grupo "Dpto. Ventas" vemos que dentro de sus Miembros encontramos al user Marco Riera
<img width="1053" height="519" alt="Captura de pantalla 2026-10-15 175606" src="https://github.com/user-attachments/assets/4e5a82fc-5f95-43b6-b2da-6673369fb86c" />
<img width="1329" height="347" alt="Captura de pantalla 2026-10-15 175634" src="https://github.com/user-attachments/assets/a0c4adf3-2e6a-48e1-a4e8-59b62605f4f5" />

---

### 2. Fase Mover (Transición de Roles)
- **Mecanismo:** Modificación de atributos del perfil del usuario (Department = IT).
- **Comportamiento:** El motor de evaluación dinámica de Entra ID procesa el cambio de metadatos, removiendo al usuario de `Depto. Ventas` e incorporándolo a `Depto. IT`
- **Objetivo de Seguridad:** Eliminar el Privilege Creep (Acumulación innecesaria de derechos de acceso en usuarios).

### Actualización de atributos en el Usuario

- Dentro del apartado `Job Information` reemplazamos el atributo "Ventas" por "IT" dentro del campo `Department`
<img width="869" height="607" alt="Captura de pantalla 2026-10-15 175934" src="https://github.com/user-attachments/assets/26505f66-f67f-44a9-a9e7-2435a653a6e7" />
<img width="848" height="845" alt="Captura de pantalla 2026-10-15 180012" src="https://github.com/user-attachments/assets/62eba7e9-3a9e-436b-b178-550a983bd5fc" />

### Verificación

- Confirmamos que el grupo "Dpto. Ventas" ya no tiene miembros.
<img width="1032" height="523" alt="user 0" src="https://github.com/user-attachments/assets/7fc0bf58-c2bb-408f-b457-907487669a7c" />
<img width="661" height="345" alt="Captura de pantalla 2026-10-15 180250" src="https://github.com/user-attachments/assets/0814bec7-9c5e-41bf-9845-cf4e2956ba44" />

- El grupo "Dpto. IT" tiene como miembro al usuario "Marco Riera".
<img width="1040" height="527" alt="Captura de pantalla 2026-10-15 180327" src="https://github.com/user-attachments/assets/f7eee182-8472-43b9-b5bb-57ee6625ed22" />
<img width="1333" height="347" alt="Captura de pantalla 2026-10-15 180411" src="https://github.com/user-attachments/assets/3ac45669-737d-471c-85bc-345e2f3bf579" />

---

### 3. Fase Leaver (Desvinculación y Revocación de Accesos)
- **Mecanismo:**
  - Bloqueo de Inicio de Sesión (Block Sign-in)
  - Revocar Sesiones Activas (Revoke Sessions)
  - Eliminacion de Atributos
- **Comportamiento:** Prevención inmediata de nuevos intentos de autenticación y remoción automática de todos los grupos operativos.
- **Objetivo de Seguridad:** Eliminar cuentas huérfanas y prevenir accesos no autorizados posteriores a la culminación de la relación laboral.
 
### Proceso de inhabilitar Cuenta de usuario

- Para evitar que el usuario pueda loguearse, vamos al apartado `Settings` y desahibilitamos el campo `Account Enabled`
<img width="836" height="371" alt="Captura de pantalla 2026-10-15 182158" src="https://github.com/user-attachments/assets/ee57646d-b5d1-419c-b361-fe53e1e87d77" />
<img width="842" height="356" alt="Captura de pantalla 2026-10-15 182212" src="https://github.com/user-attachments/assets/b0dbd132-426b-49d7-af70-223e31cc7553" />

- Para cerrar todas las sesiones que el usuario pueda tener activas, vamos al apartado `Revoke Sessions` y confirmamos que queremos cerrar todas las sesiones de este usuario
<img width="1120" height="101" alt="Captura de pantalla 2026-10-15 182224" src="https://github.com/user-attachments/assets/53d0f894-ec11-4849-95b1-420639ad71df" />
<img width="616" height="105" alt="Captura de pantalla 2026-10-15 182511" src="https://github.com/user-attachments/assets/465b8a00-360e-4886-a572-4c87d74e1178" />

- Para eliminar automatica la cuenta comom miembro del grupo `Depto. IT` reemplazamos el atributo "IT" dentro del campo `Department` por el atributo "BAJA" 
<img width="865" height="853" alt="Captura de pantalla 2026-10-15 182756" src="https://github.com/user-attachments/assets/7551a400-2b6c-4c34-b249-948020bd7a0e" />
<img width="845" height="846" alt="Captura de pantalla 2026-10-15 182811" src="https://github.com/user-attachments/assets/9b05f692-908b-44c7-995b-a03533e3b367" />

### Verificación

- El grupo "Dpto. IT" ya no tiene miembros
<img width="1042" height="515" alt="Captura de pantalla 2026-10-15 182911" src="https://github.com/user-attachments/assets/ee4ad1f9-c997-409e-9f7a-75abe74336ab" />
<img width="788" height="333" alt="Captura de pantalla 2026-10-15 182929" src="https://github.com/user-attachments/assets/47e2fa9c-d7f9-4d38-8649-83a6edeafa57" />




