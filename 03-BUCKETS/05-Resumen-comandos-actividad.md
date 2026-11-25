# Comandos de la actividad: ciclo de vida de un bucket en S3

Guía en orden para repetir la actividad. Reemplaza `cristian` y `practica-cristian-00` por tus nombres.

- ✅ = ya lo hice
- ⏳ = me falta
- 📸 = va en la evidencia

> Nunca copies la `SecretAccessKey` en archivos, capturas ni chats. Solo se pega una vez, en `aws configure --profile lector-s3`.

---

## 0. Antes de empezar ✅

```bash
# Quién soy y en qué región trabajo
aws sts get-caller-identity
aws configure get region

# Carpeta de trabajo
mkdir practica-s3
cd practica-s3
```

---

## Parte 1: Crear ✅

**Paso 1.** Crear el bucket

```bash
aws s3 mb s3://practica-cristian-00
```

**Paso 2.** Listar los buckets

```bash
aws s3 ls
```

**Paso 3.** Ver la región del bucket 📸

```bash
aws s3api get-bucket-location --bucket practica-cristian-00
```

Resultado: `"LocationConstraint": "us-east-2"`

---

## Parte 2: Subir ✅

**Paso 4.** Crear los tres archivos

```bash
touch notas.txt
touch tareas.txt
touch secret.txt
```

> La actividad pide una línea de contenido en cada uno. Con `touch` quedan vacíos; para escribirles algo usa `echo "texto" > notas.txt`.

**Paso 5.** Subir `notas.txt`

```bash
aws s3 cp notas.txt s3://practica-cristian-00/
```

**Paso 6.** Subir los otros dos con `sync`

```bash
aws s3 sync . s3://practica-cristian-00/
```

> Error que tuve: `aws s3 sync s3://practica-cristian-00/` falla con `ParamValidation`. `sync` necesita origen **y** destino.

**Paso 7.** Listar el bucket 📸

```bash
aws s3 ls s3://practica-cristian-00/

# Versión que pide la actividad: tamaños legibles y total
aws s3 ls s3://practica-cristian-00/ --recursive --human-readable --summarize
```

---

## Parte 3: Consultar y descargar ✅

**Paso 8.** Ver `notas.txt` sin descargarlo

```bash
aws s3 cp s3://practica-cristian-00/notas.txt -
```

**Paso 9.** Ver tamaño y fecha de `tareas.txt` sin descargarlo

```bash
aws s3 ls s3://practica-cristian-00/tareas.txt
```

**Paso 10.** Descargar `secret.txt` en `descargas`

```bash
mkdir descargas
aws s3 cp s3://practica-cristian-00/secret.txt descargas/
```

---

## Parte 4: Editar ✅

**Paso 11.** Renombrar `tareas.txt` a `pendientes.txt`

```bash
aws s3 mv s3://practica-cristian-00/tareas.txt s3://practica-cristian-00/pendientes.txt
```

**Paso 12.** Modificar `notas.txt` y volver a subirlo

```bash
echo "Nueva linea de texto" >> notas.txt
aws s3 sync . s3://practica-cristian-00/
```

> Ojo: `sync .` sube todo lo que cambió en la carpeta, y por eso también volvió a subir `tareas.txt`. Para subir solo un archivo: `aws s3 cp notas.txt s3://practica-cristian-00/`.

**Paso 13.** Ver el cambio desde el bucket 📸

```bash
aws s3 cp s3://practica-cristian-00/notas.txt -
```

> Error que tuve: sin el `-` final falla con `ParamValidation`, porque `cp` necesita un destino.

---

## Parte 5: Compartir sin hacerlo público ✅

**Pasos 14 a 16.** Enlace temporal de 60 segundos 📸

```bash
aws s3 presign s3://practica-cristian-00/notas.txt --expires-in 60
```

Abrir el enlace en el navegador antes del minuto y después de que expire.

---

## Parte 6: Menor privilegio ✅

**Paso 17.** Crear el usuario lector

```bash
aws iam create-user --user-name lector-s3-cristian
```

**Paso 18.** Crear la política `solo-lectura.json`

```bash
cat > solo-lectura.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::practica-cristian-00/*"
  }]
}
EOF
```

**Paso 19.** Adjuntar la política en línea

```bash
aws iam put-user-policy --user-name lector-s3-cristian --policy-name leer-objetos --policy-document file://solo-lectura.json
```

**Paso 20.** Crear la llave y configurar el perfil

```bash
aws iam create-access-key --user-name lector-s3-cristian
aws configure --profile lector-s3
```

**Paso 21.** Leer `notas.txt` con el perfil lector (permitido) 📸

```bash
aws s3 cp s3://practica-cristian-00/notas.txt - --profile lector-s3
```

**Paso 22.** Listar el bucket con el perfil lector (`AccessDenied`) 📸

```bash
aws s3 ls s3://practica-cristian-00/ --profile lector-s3
```

**Paso 23.** Borrar `secret.txt` con el perfil lector (`AccessDenied`) 📸

```bash
aws s3 rm s3://practica-cristian-00/secret.txt --profile lector-s3
```

---

## Parte 7: Eliminar ✅

**Paso 24.** Intentar borrar el bucket sin vaciarlo (`BucketNotEmpty`) 📸 ✅

```bash
aws s3 rb s3://practica-cristian-00
```

**Paso 25.** Vaciar el bucket ✅

```bash
aws s3 rm s3://practica-cristian-00/ --recursive
```

**Paso 26.** Borrar el bucket ✅

```bash
aws s3 rb s3://practica-cristian-00
```

**Paso 27.** Borrar la llave, la política y el usuario ✅

```bash
# Ver el ID de la llave
aws iam list-access-keys --user-name lector-s3-cristian

# Borrar la llave (reemplaza AKIA... por el ID)
aws iam delete-access-key --user-name lector-s3-cristian --access-key-id AKIA...

aws iam delete-user-policy --user-name lector-s3-cristian --policy-name leer-objetos

aws iam delete-user --user-name lector-s3-cristian
```

**Paso 28.** Borrar el perfil `lector-s3` ✅

Eliminar el bloque `[lector-s3]` de `~/.aws/credentials` y el bloque `[profile lector-s3]` de `~/.aws/config`.

**Paso 29.** Confirmar que no quedó nada 📸 ✅

```bash
aws s3 ls
aws iam list-users --query "Users[].UserName" --output table
aws configure list-profiles
```
