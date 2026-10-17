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

<img width="1389" height="381" alt="00-Groups" src="https://github.com/user-attachments/assets/11a5e1b6-8aac-4af3-aa0f-5dc9dcb25b04" />

- **Tipo de Membresía:** Dynamic User   
- **Regla de Membresía Dinámica:**
  
- ```text
  (user.department -eq "Ventas")
<img width="1581" height="449" alt="00-Regla Ventas" src="https://github.com/user-attachments/assets/f38931e1-c15b-45ab-bc58-c7bceaf08afd" />

- ```text
  (user.department -eq "IT")
<img width="1573" height="446" alt="00-Reglas IT" src="https://github.com/user-attachments/assets/2957c54a-221e-49d9-aa38-5d317b371c8d" />

---

## Implementación y Fases del Ciclo de Vida

### 1. Fase Joiner (Onboarding Automatizado)
- **Mecanismo:** Creacion del usuario y Evaluación de reglas de membresía dinámica basadas en el atributo del perfil del usuario.
- **Comportamiento:** Al aprovisionar la cuenta y configurar `Department = Ventas`, Microsoft Entra ID agrega automáticamente la identidad al grupo `Depto. Ventas` sin intervención administrativa.
- **Objetivo de Seguridad:** Garantizar permisos base estandarizados mediante automatización sin intervención manual (*zero-touch*).

### Creación de la cuenta y Asignación de Atributos 

- Generamos el usuario con sus respectivas credenciales y datos. Luego, vamos al apartado `Properties`
<img width="647" height="525" alt="01-creacion user" src="https://github.com/user-attachments/assets/11c5c13e-993b-4e16-93be-6a0cdb296756" />


- Especificamos el atributo "Ventas" dentro del campo  `Department`
<img width="864" height="638" alt="01-User Department Ventas" src="https://github.com/user-attachments/assets/e858cd96-a965-462e-9804-a67732f35822" />

### Verificación

- Al mirar el grupo "Dpto. Ventas" vemos que el user Marco Riera aparece en la lista de miembros 
<img width="1053" height="519" alt="01-Ventas Overview" src="https://github.com/user-attachments/assets/12dd6ecb-0be7-43b2-9c09-2f9092e53606" />
<img width="1329" height="347" alt="01-Ventas Members" src="https://github.com/user-attachments/assets/254d53d4-b8e2-496c-a207-adadcf48fd03" />

---

### 2. Fase Mover (Transición de Roles)
- **Mecanismo:** Modificación de atributos del perfil del usuario (Department = IT).
- **Comportamiento:** El motor de evaluación dinámica de Entra ID procesa el cambio de metadatos, removiendo al usuario de `Depto. Ventas` e incorporándolo a `Depto. IT`
- **Objetivo de Seguridad:** Eliminar el Privilege Creep (Acumulación innecesaria de derechos de acceso en usuarios).

### Actualización de atributos en el Usuario

- Dentro del apartado `Job Information` reemplazamos el atributo "Ventas" por "IT" dentro del campo `Department`
<img width="869" height="607" alt="02-Mod Depto Ventas" src="https://github.com/user-attachments/assets/f6707284-4f27-4b1f-90cb-8be826872939" />
<img width="848" height="845" alt="02-Mod Depto IT" src="https://github.com/user-attachments/assets/53954873-c485-41fb-9de0-5ba378010520" />

### Verificación

- Confirmamos que el grupo "Dpto. Ventas" ya no tiene miembros.
<img width="1032" height="523" alt="02-Ventas empty Overview" src="https://github.com/user-attachments/assets/7726eed5-e648-4b80-9320-d0bebc3e23c0" />
<img width="661" height="345" alt="02-Ventas Empty Members" src="https://github.com/user-attachments/assets/cf460b1f-fa74-42f8-9a97-365469a20e13" />

- El grupo "Dpto. IT" tiene como miembro al usuario "Marco Riera".
<img width="1040" height="527" alt="02-IT User Overview" src="https://github.com/user-attachments/assets/f1f3d1f2-cef9-48a4-8894-67f2dfbf6938" />
<img width="1333" height="347" alt="02-IT User Member" src="https://github.com/user-attachments/assets/b35ed6a4-0916-4e1e-84b0-f74751c1a970" />

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
<img width="836" height="371" alt="03-Enabled Acc ON" src="https://github.com/user-attachments/assets/72e2c1e5-3b0f-4aad-b830-57f372252da3" />
<img width="842" height="356" alt="03-Enabled Acc OFF" src="https://github.com/user-attachments/assets/8268a693-67f5-461f-b0ba-fc5fc4e0a514" />

- Para cerrar todas las sesiones que el usuario pueda tener activas, vamos al apartado `Revoke Sessions` y confirmamos que queremos cerrar todas las sesiones de este usuario
<img width="1120" height="101" alt="03-Revoke Sessions" src="https://github.com/user-attachments/assets/bcf06215-75b8-432d-bf83-24b5c38bcdbf" />
<img width="616" height="105" alt="03-Revoke Sessions OK" src="https://github.com/user-attachments/assets/dabbd299-c5ce-4df0-9ef2-2a187c7ed7c8" />

- Para eliminar automaticamente al user como miembro del grupo `Depto. IT` reemplazamos el atributo "IT" dentro del campo `Department` por el atributo "BAJA" 
<img width="865" height="853" alt="03-Baja Depto IT" src="https://github.com/user-attachments/assets/6f3bb911-aa08-4a77-ad89-4d37e76f8d5c" />
<img width="845" height="846" alt="03-Depto Baja" src="https://github.com/user-attachments/assets/0949784d-b9cc-4096-b94e-55fc5ccb67a1" />

### Verificación

- El grupo "Dpto. IT" ya no tiene miembros
<img width="1042" height="515" alt="03-IT empty Overview" src="https://github.com/user-attachments/assets/83966927-11ee-4123-8a6b-0c1346430c59" />
<img width="788" height="333" alt="03-IT Empty User" src="https://github.com/user-attachments/assets/02cfa101-9cf9-4023-8139-d7081448ad51" />


---

## Auditoría y Registros de Eventos (Audit Logs)

La plataforma Microsoft Entra ID registra de forma centralizada cada cambio de estado dentro del ciclo de vida JML. A continuación se detallan los eventos clave en los **Audit Logs**, diferenciando las acciones administrativas manuales de los procesos automatizados del sistema (**`Core Directory`**):

| Fase JML | Acción Ejecutada | Evento Registrado (`Logged Event`) | Actor / Initiated By |
| :--- | :--- | :--- | :--- |
| **1. Joiner** | Asignación de departamento | `Update user` | Administrador / Helpdesk |
| | Inclusión dinámica al grupo | `Add member to group` | `Core Directory` (Sistema) |
| **2. Mover** | Cambio a nuevo departamento | `Update user` | Administrador / Helpdesk |
| | Salida del grupo anterior | `Remove member from group` | `Core Directory` (Sistema) |
| | Entrada al nuevo grupo | `Add member to group` | `Core Directory` (Sistema) |
| **3. Leaver** | Bloqueo de inicio de sesión | `Update user` (`Account enabled: No`) | Administrador / Helpdesk |
| | Revocación de tokens activos | `Revoke user sessions` | Administrador / Helpdesk |
| | Expulsión del grupo dinámico | `Remove member from group` | `Core Directory` (Sistema) |

### Evidencias de Auditoría en la Consola
- **Historial de Actividad del Usuario:**
<img width="1098" height="808" alt="04-User AuditLogs" src="https://github.com/user-attachments/assets/4a7528a2-a715-4475-9672-c997bcb8a00e" />

- **Historial de Actividad del Grupo (Evaluación Dinámica):**
  - **Dpto. Ventas:**

  - **Dpto. IT:**

---

## Conclusiones e Impacto Operativo

La implementación del modelo **JML basado en ABAC** en Microsoft Entra ID resuelve desafíos clave en la gestión de identidades y accesos a nivel empresarial:

1. **Eficiencia:** Reduce el tiempo de tiempo de espera, eliminando tareas manuales repetitivas para el equipo de Helpdesk.
2. **Mitigación de Vulnerabilidades (Privilege Creep):** Garantiza que ningún usuario conserve accesos heredados de roles o departamentos anteriores, aplicando estrictamente el principio de **menor privilegio (Least Privilege)**.
3. **Cierre Inmediato de Brechas de Seguridad:** La combinación de `Block Sign-in` + `Revoke Sessions` en la fase Leaver asegura la invalidación de tokens activos, previniendo accesos no autorizados por medio de sesiones abiertas.
4. **Trazabilidad y Cumplimiento (Auditability):** Todo el lifecycle queda registrado y respaldado en los **Audit Logs** de Entra ID, facilitando auditorías de cumplimiento normativo (por ejemplo: ISO 27001).


