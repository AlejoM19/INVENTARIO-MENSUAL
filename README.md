# Plataforma de inventario mensual

Herramienta interna para el ciclo de inventario físico de los puntos de venta: reparto de formatos de conteo, consolidación del análisis de diferencias, revisión de justificaciones, cierre y archivo plano para SIESA.

Es una página estática. **Todos los archivos se procesan en el navegador**: ningún Excel se sube a un servidor.

---

## Contenido del repositorio

| Archivo | Para qué sirve |
|---|---|
| `index.html` | La plataforma completa. Un solo archivo, sin dependencias que compilar. |
| `sin-acceso.html` | Pantalla que ve quien no tiene una cuenta autorizada. |
| `staticwebapp.config.json` | Obliga a iniciar sesión antes de entregar la página. |
| `.github/workflows/azure-static-web-apps.yml` | Publica automáticamente con cada cambio en `main`. |

---

## Puesta en marcha

### 1. Crear el repositorio

Sube estos archivos a un repositorio **privado** de GitHub, en la rama `main`.

### 2. Crear la Static Web App

En el portal de Azure: **Crear recurso → Static Web App**.

- Plan: **Free**
- Origen: GitHub, apuntando a este repositorio y a la rama `main`
- Presets de compilación: **Custom**
- App location: `/` · Api location: en blanco · Output location: en blanco

Azure agrega solo el secreto `AZURE_STATIC_WEB_APPS_API_TOKEN` al repositorio.

### 3. Registrar la aplicación en Entra ID

En **Microsoft Entra ID → Registros de aplicaciones → Nuevo registro**:

- URI de redirección (Web): `https://<tu-sitio>.azurestaticapps.net/.auth/login/aad/callback`
- Copia el **Id. de directorio (inquilino)** y el **Id. de aplicación (cliente)**
- En **Certificados y secretos**, genera un secreto de cliente

### 4. Conectar las credenciales

En `staticwebapp.config.json`, reemplaza `PEGAR_AQUI_EL_TENANT_ID` por el Id. de inquilino.

En la Static Web App → **Configuración**, agrega:

| Nombre | Valor |
|---|---|
| `AAD_CLIENT_ID` | Id. de aplicación (cliente) |
| `AAD_CLIENT_SECRET` | El secreto generado |

### 5. Restringir el dominio

En `index.html`, busca la constante `DOMINIOS` y deja los dominios autorizados:

```js
const DOMINIOS=['templecolombia.com'];
```

Para limitar además a personas concretas, en Entra ID → *Aplicaciones empresariales → Propiedades*, activa **Asignación de usuarios requerida** y agrega solo a quienes deban entrar.

---

## Cómo se usa

| Pestaña | Qué hace |
|---|---|
| **01 · Tablero** | Avance del cierre por punto: cinco pasos, consecutivo SIESA y notas. |
| **02 · Formatos** | Carga el maestro de ítems y genera un formato de conteo por bodega. |
| **03 · Diferencias** | Junta los análisis que SIESA exporta punto por punto y devuelve el modelo de justificación con listas desplegables. |
| **04 · Correos** | Directorio de correos, coordinadores, horarios y las tres plantillas. |
| **05 · Gerencial** | Análisis del mes y el informe PDF para dirección. |
| **06 · Cierre** | Revisa las justificaciones devueltas y genera el archivo plano para SIESA. |
| **07 · Historial** | Archiva el mes, compara periodos y descarga el respaldo. |

---

## Dónde viven los datos

El avance, el directorio, las plantillas y el historial se guardan **en el navegador de cada persona**. No se comparten entre equipos y se pierden si se borran los datos de navegación.

Al cerrar cada mes, entra a **07 · Historial → Descargar respaldo** y guarda el `.json` en la carpeta sincronizada de OneDrive. Desde otro equipo se recupera con *Restaurar desde respaldo*.

> El `.gitignore` bloquea los respaldos y los Excel para que no terminen publicados por error. No los subas al repositorio: contienen correos y cifras del inventario.

---

## Mantenimiento

Todo el código está en `index.html`. Los puntos que se ajustan con más frecuencia:

| Qué | Dónde |
|---|---|
| Dominios autorizados | `const DOMINIOS` |
| Causas de la lista JUSTIFICACIÓN | `const JUST_DEF`, o desde la pestaña 03 |
| Coordinadores iniciales | `const COORDS_DEF` |
| Horarios de conteo | Pestaña 04, o `TPL_DEF.horPub` / `horBod` |
| Textos de los correos | Pestaña 04, o `TPL_DEF.t1b` / `t2b` / `t3b` |

Lo que se cambia desde las pestañas queda guardado en el navegador; lo que se cambia en el código aplica para todos.
