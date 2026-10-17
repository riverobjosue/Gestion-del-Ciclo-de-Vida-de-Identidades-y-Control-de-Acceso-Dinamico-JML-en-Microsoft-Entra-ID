# Gestión del Ciclo de Vida de Identidades y Control de Acceso Dinámico (JML) en Microsoft Entra ID
## Objetivo
Implementar un modelo de **Control de Acceso Basado en Atributos (ABAC)** dentro de Microsoft Entra ID para automatizar el ciclo de vida de las identidades de un usuario: **Joiner, Mover y Leaver (JML)**.

Mediante el uso de reglas de membresía dinámica en grupos de seguridad, esta arquitectura elimina la asignación manual de permisos, mitiga la acumulación de privilegios (**Privilege Creep**) y aplica controles de seguridad estrictos para la revocación inmediata de sesiones y el bloqueo de cuentas durante la baja de colaboradores.

## Arquitectura Técnica y Alcance

- **Proveedor de Identidad:** Microsoft Entra ID
- **Modelo de Acceso:** Control de Acceso Basado en Atributos (ABAC)
- **Operaciones Principales:**
  - **Fase Joiner (Alta):** Aprovisionamiento automático de accesos basado en metadatos del perfil del usuario (`department`).
  - **Fase Mover (Transición):** Reasignación dinámica de grupos para garantizar la evolución de permisos sin acumulación de permisos o accesos.
  - **Fase Leaver (Baja):** Bloqueo de inicio de sesión, invalidación global de tokens de sesión y expulsión automatizada de los grupos.

---

## Prerrequisitos y Configuración de Infraestructura Antes de ejecutar las pruebas del ciclo de vida
se configuró la estructura de grupos dinámicos basada en el modelo ABAC dentro de Microsoft Entra ID: 
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
- **Mecanismo:** Evaluación de reglas de membresía dinámica basadas en el atributo del perfil del usuario (`user.department -eq "IT"`).
- **Comportamiento:** Al aprovisionar la cuenta y configurar `Department = IT`, Microsoft Entra ID asigna automáticamente la identidad al grupo `GRP-Dept-IT` sin intervención administrativa.
- **Objetivo de Seguridad:** Garantizar permisos base estandarizados mediante automatización sin intervención manual (*zero-touch*).

```text
Sintaxis de Regla: (user.department -eq "IT")

