# Comandos de la actividad: ciclo de vida de un usuario IAM

Guía en orden para repetir la actividad. Reemplaza `cristian` por el nombre que uses.

- ✅ = ya lo hice
- ⏳ = me falta

> Nunca copies la `SecretAccessKey` en archivos, capturas ni chats. Solo se pega una vez, en `aws configure --profile lector`.

---

## 0. Antes de empezar ⏳

```bash
# Confirmar que estoy conectado como mi usuario administrador (admincap)
aws sts get-caller-identity

# Ver la región configurada
aws configure get region
```

---

## Parte 1: Crear ✅

**Paso 1.** Crear el usuario

```bash
aws iam create-user --user-name dev-cristian
```

**Paso 2.** Listar los usuarios y confirmar que aparece

```bash
aws iam list-users

# Versión corta, solo los nombres
aws iam list-users --query "Users[].UserName" --output table
```

**Paso 3.** Ver los datos del usuario y anotar el `Arn` 📸

```bash
aws iam get-user --user-name dev-cristian

# Solo el ARN
aws iam get-user --user-name dev-cristian --query "User.Arn" --output text
```

ARN obtenido: `arn:aws:iam::030453259980:user/dev-cristian`

---

## Parte 2: Editar ✅

**Paso 4.** Agregar la etiqueta de área

```bash
aws iam tag-user --user-name dev-cristian --tags Key=area,Value=desarrollo
```

**Paso 5.** Cambiar el nombre a `lector-cristian`

```bash
aws iam update-user --user-name dev-cristian --new-user-name lector-cristian
```

> Ojo: el renombrado se hace con `update-user`, no con `tag-user`. Si usas `tag-user --new-user-name` sale `ParamValidation` porque falta `--tags`.

**Paso 6.** Verificar nombre nuevo y etiqueta 📸

```bash
# El nombre viejo ya no existe (NoSuchEntity)
aws iam get-user --user-name dev-cristian

# El nombre nuevo conserva la etiqueta y tiene otro ARN
aws iam get-user --user-name lector-cristian
```

ARN nuevo: `arn:aws:iam::030453259980:user/lector-cristian` (el `UserId` no cambia).

---

## Parte 3: Dar permisos ✅

**Paso 7.** Adjuntar la política de solo lectura de EC2

```bash
aws iam attach-user-policy --user-name lector-cristian --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

**Paso 8.** Confirmar que quedó adjunta 📸

```bash
aws iam list-attached-user-policies --user-name lector-cristian
```

---

## Parte 4: Probar el menor privilegio

**Paso 9.** Crear la llave de acceso ✅

```bash
aws iam create-access-key --user-name lector-cristian
```

Guarda el `AccessKeyId` y la `SecretAccessKey` solo el tiempo necesario para el paso 10.

**Paso 10.** Configurar el perfil `lector` ✅

```bash
aws configure --profile lector
```

Valores: Access Key ID, Secret Access Key, región `us-east-2`, formato `json`.

**Paso 11.** Comprobar la identidad con ese perfil ⏳

```bash
aws sts get-caller-identity --profile lector
```

Debe mostrar `arn:aws:iam::030453259980:user/lector-cristian`.

**Paso 12.** Acción permitida ⏳ 📸

```bash
aws ec2 describe-instances --profile lector
```

Esperado: responde sin error (si no tienes instancias, sale `"Reservations": []`).

**Paso 13.** Acción no permitida ⏳ 📸

```bash
aws iam list-users --profile lector
```

Esperado: `AccessDenied ... is not authorized to perform: iam:ListUsers ...`

---

## Parte 5: Quitar permisos ⏳

**Paso 14.** Con el admin (sin `--profile`), quitar la política

```bash
aws iam detach-user-policy --user-name lector-cristian --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

# Confirmar que ya no tiene políticas
aws iam list-attached-user-policies --user-name lector-cristian
```

**Paso 15.** Repetir la acción del paso 12 📸

```bash
aws ec2 describe-instances --profile lector
```

Esperado: ahora sale `UnauthorizedOperation` / `AccessDenied`.
Si todavía funciona, espera unos segundos y repite: IAM tarda un poco en propagar los cambios.

---

## Parte 6: Eliminar ⏳

**Reto (opcional, antes del paso 16).** Intentar borrar el usuario con la llave todavía activa

```bash
aws iam delete-user --user-name lector-cristian
```

Esperado: `DeleteConflict ... Cannot delete entity, must delete access keys first.`

**Paso 16.** Listar las llaves y borrarlas

```bash
aws iam list-access-keys --user-name lector-cristian

# Solo los IDs
aws iam list-access-keys --user-name lector-cristian --query "AccessKeyMetadata[].AccessKeyId" --output text

aws iam delete-access-key --user-name lector-cristian --access-key-id <AccessKeyId>

# Confirmar que no quedan llaves
aws iam list-access-keys --user-name lector-cristian
```

**Paso 17.** Confirmar que no le quedan políticas

```bash
aws iam list-attached-user-policies --user-name lector-cristian
aws iam list-user-policies --user-name lector-cristian
```

Ambos deben devolver listas vacías.

**Paso 18.** Borrar el usuario

```bash
aws iam delete-user --user-name lector-cristian
```

**Paso 19.** Confirmar que ya no aparece 📸

```bash
aws iam list-users --query "Users[].UserName" --output table
```

**Paso 20.** Eliminar el perfil `lector` de la configuración local

```bash
# Revisar qué hay
cat ~/.aws/credentials
cat ~/.aws/config

# Editar y borrar el bloque [lector] en credentials y [profile lector] en config
nano ~/.aws/credentials
nano ~/.aws/config

# Confirmar que el perfil ya no existe
aws configure list-profiles
```

---

## Resumen rápido (todo seguido)

```bash
aws sts get-caller-identity
aws iam create-user --user-name dev-cristian
aws iam list-users
aws iam get-user --user-name dev-cristian
aws iam tag-user --user-name dev-cristian --tags Key=area,Value=desarrollo
aws iam update-user --user-name dev-cristian --new-user-name lector-cristian
aws iam get-user --user-name lector-cristian
aws iam attach-user-policy --user-name lector-cristian --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
aws iam list-attached-user-policies --user-name lector-cristian
aws iam create-access-key --user-name lector-cristian
aws configure --profile lector
aws sts get-caller-identity --profile lector
aws ec2 describe-instances --profile lector
aws iam list-users --profile lector
aws iam detach-user-policy --user-name lector-cristian --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
aws ec2 describe-instances --profile lector
aws iam list-access-keys --user-name lector-cristian
aws iam delete-access-key --user-name lector-cristian --access-key-id <AccessKeyId>
aws iam list-attached-user-policies --user-name lector-cristian
aws iam delete-user --user-name lector-cristian
aws iam list-users
aws configure list-profiles
```

📸 = paso que lleva captura en `evidencia.md` (3, 6, 8, 12, 13, 15, 19).
