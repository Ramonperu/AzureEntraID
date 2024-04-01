# Creación de aplicación web en contenedores con Docker

Bienvenidos a la guía de EntraID, este resumen practico esta hecho gracias a la documentación oficial de Microsoft https://learn.microsoft.com/en-us/training/modules/manage-users-and-groups-in-aad/  Realizada para practicar para la renovación de la certificación **AZ-104** a **Marzo de 2024**.




<img align="center" src="/img/1ºimagenn.PNG"  />

## Conceptos básicos

El usuario también puede utilizar Microsoft Entra ID de forma independiente del Directorio Activo de Windows. Las empresas más pequeñas pueden emplear Microsoft Entra ID como su único servicio de directorio para controlar el acceso a sus aplicaciones y productos SaaS, como Microsoft 365, Salesforce y Dropbox.

### Directorios, suscripciones y usuarios 

Microsoft ofrece varias ofertas basadas en la nube en la actualidad, todas las cuales pueden utilizar Microsoft Entra ID para identificar a los usuarios y controlar el acceso:

- Microsoft Azure
- Microsoft 365
- Microsoft Intune
- Microsoft Dynamics 365

Cuando una empresa u organización se registra para utilizar una de estas ofertas, se le asigna un directorio predeterminado, una instancia de Microsoft Entra ID. Este directorio contiene los usuarios y grupos que tendrán acceso a cada uno de los servicios que la empresa ha adquirido. Puedes referirte a este directorio predeterminado como un tenant. Un tenant representa a la organización y el directorio predeterminado asignado a ella.

Una suscripción en Azure es tanto una entidad de facturación como un límite de seguridad. Los recursos como máquinas virtuales, sitios web y bases de datos están asociados con una sola suscripción. Cada suscripción también tiene un único propietario de cuenta responsable de los cargos incurridos por los recursos en esa suscripción. Si tu organización desea que una suscripción sea facturada a otra cuenta, puedes transferir la suscripción. *Un directorio de Microsoft Entra se puede asociar con varias suscripciones, pero una suscripción siempre está vinculada a un único directorio.*

Puedes agregar usuarios y grupos a múltiples suscripciones. Esto permite al usuario crear, controlar y acceder a recursos en la suscripción.

### Usuarios

- **Identidades en la nube**: Estos usuarios son exclusivos de Microsoft Entra ID. Ejemplos incluyen cuentas de administrador y usuarios gestionados internamente. Su origen es Microsoft Entra ID o Microsoft Entra ID externo si el usuario está definido en otra instancia de Microsoft Entra, pero necesita acceso a los recursos de suscripción controlados por este directorio. Cuando estas cuentas son eliminadas del directorio principal, también son eliminadas.
- **Identidades sincronizadas con el directorio**: Estos usuarios existen en un Directorio Activo local. A través de Microsoft Entra Connect, se sincronizan con Azure. Su fuente es el Directorio Activo de Windows Server.
- **Usuarios invitados**: Estos usuarios residen fuera de Azure. Ejemplos incluyen cuentas de otros proveedores de nube y cuentas de Microsoft, como una cuenta de Xbox LIVE. Su fuente es el Usuario Invitado. Este tipo de cuenta es útil cuando proveedores o contratistas externos necesitan acceso a tus recursos de Azure. Una vez que su colaboración ya no sea necesaria, puedes eliminar la cuenta y todos sus accesos.

### Agregar usuarios

Puede agregar identidades en la nube a Microsoft Entra ID de varias maneras:

- Sincronización de un Active Directory local de Windows Server.
- Usando el portal de Azure.
- Usando la línea de comando.
- Otras opciones.

### Sincronización de un Directorio Activo local de Windows Server

Microsoft Entra Connect es un servicio autónomo que posibilita la sincronización de un Directorio Activo convencional con tu instancia de Microsoft Entra. Esta es la manera en que la mayoría de los clientes empresariales añaden usuarios al directorio. La ventaja de este método es que los usuarios pueden emplear el inicio de sesión único (SSO) para acceder tanto a recursos locales como a los basados en la nube.

**Creación de usuarios via CLI**

Desde PowerShell

```
$PasswordProfile = @{ Password = "contraseñ1a!" }

New-MgUser -DisplayName "Abby Brown" -PasswordProfile $PasswordProfile -MailNickName "AbbyB" -UserPrincipalName "AbbyB@contoso.com" -AccountEnabled
```

Desde Azure Cli

```
az ad user create --display-name "Abby Brown" \
--password "<password>" \
--user-principal-name "AbbyB@contoso.com" \
--force-change-password-next-login true \
--mail-nickname "AbbyB"
```

También tenemos la opción de Importar un CSV con los datos de los uisuarios

**Grupos**

Microsoft Entra ID ofrece la capacidad de definir dos tipos distintos de grupos:

1. **Grupos de seguridad**: Estos son los más habituales y se utilizan para gestionar el acceso de miembros y dispositivos a recursos compartidos entre un conjunto de usuarios. Por ejemplo, puedes crear un grupo de seguridad para una política de seguridad específica. Al hacerlo de esta manera, puedes asignar un conjunto de permisos a todos los miembros simultáneamente, en lugar de tener que agregar permisos individualmente a cada miembro. Esta opción requiere un administrador de Microsoft Entra.
2. **Grupos de Microsoft 365**: Estos grupos ofrecen oportunidades de colaboración al otorgar a los miembros acceso a un buzón de correo compartido, calendario, archivos, sitio de SharePoint y más. Esta opción también te permite dar acceso al grupo a personas externas a tu organización. Está disponible tanto para usuarios como para administradores.

### Roles predefinidos para recursos de Azure (mediante PowerShell) 

Azure ofrece diversos roles predefinidos para abordar los escenarios de seguridad más comunes. Para comprender cómo funcionan estos roles, examinemos tres que se aplican a todos los tipos de recursos:

1. Propietario: Este rol tiene acceso completo a todos los recursos, incluido el derecho a delegar acceso a otros usuarios.
2. Colaborador: Puede crear y administrar cualquier tipo de recurso en Azure, pero no puede otorgar acceso a otros usuarios.
3. Lector: Tiene la capacidad de visualizar los recursos de Azure existentes.

### Definiciones de roles 

Cada rol es un conjunto de propiedades definidas en un archivo de notación de objetos JavaScript (JSON). Esta definición incluye un Nombre, ID y Descripción. Además, especifica los permisos permitidos (Actions), los permisos denegados (NotActions) y el alcance (por ejemplo, acceso de lectura) del rol.

Por ejemplo, para el rol de Propietario, esto implica todas las acciones, indicadas por un asterisco (*); ninguna acción denegada; y todos los alcances, indicados por una barra diagonal (/).

Puedes obtener esta información utilizando el cmdlet `Get-AzRoleDefinition` de PowerShell.

Estos son los datos que se completan al definir roles:

| nombre                | Descripción                                                  |
| :-------------------- | :----------------------------------------------------------- |
| `Id`                  | Identificador único para el rol, asignado por Azure          |
| `IsCustom`            | Verdadero si es un rol personalizado, Falso si es un rol integrado |
| `Description`         | Una descripción legible del rol.                             |
| `Actions []`          | Permisos permitidos; `*`indica todo                          |
| `NotActions []`       | Permisos denegados                                           |
| `DataActions []`      | Permisos permitidos específicos aplicados a los datos, por ejemplo`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` |
| `NotDataActions []`   | Permisos denegados específicos aplicados a los datos.        |
| `AssignableScopes []` | Ámbitos donde se aplica este rol; `/`indica global, pero puede llegar a un árbol jerárquico |

**Acciones y no acciones**

| Rol incorporado                                              | Comportamiento | NotAcciones                                                  |
| :----------------------------------------------------------- | :------------- | :----------------------------------------------------------- |
| Propietario (permitir todas las acciones)                    | `*`            | -                                                            |
| Colaborador (permitir todas las acciones excepto escribir o eliminar asignaciones de roles) | `*`            | `Microsoft.Authorization/*/Delete, Microsoft.Authorization/*/Write, Microsoft.Authorization/elevateAccess/Action` |
| Lector (permitir todas las acciones de lectura)              | `*/read`       | -                                                            |

Lista de acciones de datos y de nodata

Ejemplos mas aqui (https://learn.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations )

| Operación de datos                                           | Descripción                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete` | Eliminar datos de blobs                                      |
| `Microsoft.Compute/virtualMachines/login/action`             | Inicie sesión en una máquina virtual como usuario normal     |
| `Microsoft.EventHub/namespaces/messages/send/action`         | Enviar mensajes en un centro de eventos                      |
| `Microsoft.Storage/storageAccounts/fileServices/fileshares/files/read` | Devolver un archivo/carpeta o una lista de archivos/carpetas |
| `Microsoft.Storage/storageAccounts/queueServices/queues/messages/read` | Leer un mensaje de una cola                                  |

**Alcance o scope**

Ejemplos de alcance

| A                                                            | Alcance de uso                                               |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Restringir a una suscripción                                 | `"/subscriptions/{sub-id}"`                                  |
| Restringir a un grupo de recursos específico en una suscripción específica | `"/subscriptions/{sub-id}/resourceGroups/{rg-name}"`         |
| Restringir a un recurso específico                           | `"/subscriptions/{sub-id}/resourceGroups/{rg-name}/{resource-name}"` |
| Hacer que un rol esté disponible para su asignación en dos suscripciones | `"/subscriptions/{sub-id}", "/subscriptions/{sub-id}"`       |



### ENTRA CONNECT

Con Microsoft Entra Connect, puede proporcionar a sus usuarios una identidad común para aplicaciones Microsoft 365, Azure y SaaS integradas con Microsoft Entra ID en un entorno de identidad híbrido.

<img align="center" src="/img/2ºimagenn.PNG"  />

**¿Qué incluye Microsoft Entra Connect?**

Microsoft Entra Connect ofrece varios componentes que puedes instalar para establecer un sistema de identidad híbrido.

1. **Servicios de sincronización**: Este componente se encarga de crear usuarios, grupos y otros objetos, garantizando que la información de identidad de tus usuarios y grupos locales coincida con la de la nube.
2. **Monitoreo del estado**: Microsoft Entra Connect Health ofrece una sólida supervisión y una ubicación central en el portal de Azure para visualizar esta actividad.
3. **AD FS**: La federación es una parte opcional de Microsoft Entra Connect que puedes utilizar para configurar un entorno híbrido a través de una infraestructura AD FS local. Esto permite abordar implementaciones complejas, como SSO de unión a dominio, aplicación de la política de inicio de sesión de Active Directory y autenticación multifactor de terceros o tarjeta inteligente.
4. **Sincronización de hash de contraseña**: Esta característica es un método de inicio de sesión que sincroniza un hash de la contraseña del Directorio Activo local de un usuario con Microsoft Entra ID.
5. **Autenticación Pass-Through**: Esto permite a los usuarios iniciar sesión en aplicaciones locales y basadas en la nube utilizando las mismas contraseñas. Esto reduce los costos del servicio de asistencia técnica de TI, ya que es menos probable que los usuarios olviden cómo iniciar sesión. Esta función proporciona una alternativa a la sincronización de hash de contraseñas y permite a las organizaciones aplicar sus políticas de seguridad y complejidad de contraseñas.

**Beneficios**

La integración de sus directorios locales con Microsoft Entra ID mejora la productividad de sus usuarios al ofrecer una identidad unificada para acceder a los recursos locales y en la nube. Tanto los usuarios individuales como las organizaciones obtienen los siguientes beneficios:

1. Los usuarios pueden utilizar una sola identidad para acceder tanto a aplicaciones locales como a servicios en la nube, como Microsoft 365.
2. Una única herramienta proporciona una experiencia de implementación simple para la sincronización y el inicio de sesión.
3. La integración ofrece las capacidades más actualizadas para sus escenarios. Microsoft Entra Connect reemplaza versiones anteriores de herramientas de integración de identidades como DirSync y Azure AD Sync.
