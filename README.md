# Plataforma Educativa de Software Libre

Plataforma web interactiva para aprender los fundamentos del **software libre y de código abierto**. Incluye una prueba diagnóstica, nueve módulos de contenido con actividades interactivas, seguimiento del progreso y un panel de estadísticas para el administrador.

Es una aplicación de **una sola página (single-page app)** contenida en un único archivo `index.html`, sin proceso de compilación. Usa **Supabase** como backend para la autenticación y el almacenamiento de datos.

---

## Características

### Para estudiantes
- **Registro e inicio de sesión** con usuario y contraseña.
- **Prueba diagnóstica** inicial que mide saberes previos y asigna un nivel:

  | Porcentaje | Nivel |
  |---|---|
  | 90 % o más | Avanzado |
  | 70 % – 89 % | Intermedio |
  | 50 % – 69 % | Básico |
  | Menos de 50 % | Inicial |

  Incluye retroalimentación y recomendaciones según el resultado, y permite reintentar (se guarda el historial de intentos).
- **Módulos bloqueados hasta completar el diagnóstico**, con seguimiento del progreso por módulo.
- **Actividades interactivas**: tarjetas de conceptos, juegos de asociación, preguntas de verdadero/falso, listas de verificación, línea de tiempo, asistente de licencias, terminal simulada, comparador, antes/después, flujo de contribución y galerías de imágenes.
- **Proyectos prácticos** por módulo, incluido un proyecto final integrador.

### Para administradores
- **Panel de estadísticas** con indicadores (KPIs), distribución por niveles, gráficos, tabla de estudiantes con filtros y ordenamiento, y actividad reciente.
- **Gestión de usuarios**: crear usuarios con rol y eliminarlos.
- **Configuración de la cuenta**: cambio de nombre, usuario y contraseña.

---

## Contenido del curso

| Módulo | Tema |
|---|---|
| 0 | Prueba diagnóstica (saberes previos) |
| 1 | Fundamentos del software libre |
| 2 | Licencias de software libre y de código abierto |
| 3 | Sistemas operativos y software libre |
| 4 | Herramientas de software libre para el trabajo cotidiano |
| 5 | Software libre en educación y organizaciones |
| 6 | Comunidades y cultura del software libre |
| 7 | Seguridad, ética y responsabilidad |
| 8 | Proyecto práctico final |

---

## Tecnologías

- **HTML, CSS y JavaScript** (sin frameworks, un solo archivo)
- [**Supabase**](https://supabase.com) (`@supabase/supabase-js` v2.45.4): autenticación, base de datos PostgreSQL y Row Level Security
- [**Chart.js**](https://www.chartjs.org) v4.4.1 para los gráficos (opcional: si no carga, se dibujan gráficos de respaldo en CSS)
- **Google Fonts** (Sora e Inter)

Las librerías se cargan desde CDN, por lo que se necesita conexión a internet.

---

## Estructura del proyecto

```
.
├── index.html   # Toda la aplicación (HTML + CSS + JS)
└── README.md
```

---

## Configuración de Supabase

### 1. Credenciales
Al inicio del script en `index.html` se encuentran las constantes de conexión:

```js
const SUPABASE_URL = "https://TU-PROYECTO.supabase.co";
const SUPABASE_ANON_KEY = "TU_ANON_KEY";
```

Obtén ambos valores en **Project Settings → API** de tu proyecto de Supabase. Usa siempre la clave **`anon public`**.

> ⚠️ **Nunca** pongas la clave `service_role` en este archivo. La `anon key` es pública por diseño; la seguridad de los datos depende de las políticas **RLS** (Row Level Security).

### 2. Tablas requeridas

| Tabla | Uso |
|---|---|
| `perfiles` | Datos del usuario: nombre, usuario, rol, `protegido`, `creado_en` |
| `progreso_modulos` | Módulos completados por cada usuario (`user_id`, `modulo`) |
| `diagnosticos` | Resultados de la prueba: nota, porcentaje, nivel, respuestas, intento, etc. |
| `configuracion` | Pares clave/valor (por ejemplo `cartilla_nombre`) |

### 3. Funciones RPC requeridas

| Función | Propósito |
|---|---|
| `email_por_usuario(p_usuario)` | Devuelve el correo interno asociado a un nombre de usuario |
| `es_admin()` | Indica si el usuario autenticado es administrador |
| `asignar_rol(p_id, p_rol)` | Asigna un rol a un usuario recién creado |
| `eliminar_usuario(p_id)` | Elimina un usuario (solo administradores) |

### 4. Row Level Security
Activa RLS en **todas** las tablas con políticas que permitan:
- A cada estudiante, leer y escribir únicamente sus propios registros.
- A los administradores, ver y gestionar todos los registros.

### 5. Autenticación
La plataforma crea cuentas con un correo interno ficticio (`usuario@plataforma-sl.app`) para que el estudiante solo use su nombre de usuario. Por eso, en **Authentication → Providers → Email** debes **desactivar "Confirm email"**; de lo contrario nadie podrá iniciar sesión.

---

## Ejecución en local

No requiere instalación de dependencias. Basta con servir la carpeta con cualquier servidor estático:

```bash
# Con Python
python3 -m http.server 8000

# O con Node.js
npx serve .
```

Luego abre `http://localhost:8000` en el navegador.

---

## Despliegue

Al ser un sitio estático, puede publicarse en cualquier hosting estático. El archivo principal **debe llamarse `index.html`** y estar en la raíz.

### Vercel
1. Sube el proyecto a un repositorio de GitHub.
2. En [vercel.com](https://vercel.com) elige **Add New → Project** e importa el repositorio.
3. Configura **Framework Preset: Other** y deja vacíos *Build Command* y *Output Directory*.
4. Pulsa **Deploy**.

También funciona con **Netlify**, **GitHub Pages** y **Cloudflare Pages**.

### Después de desplegar
En Supabase, en **Authentication → URL Configuration**, agrega la URL de producción como *Site URL* y en *Redirect URLs*.

---

## Solución de problemas

| Problema | Posible causa |
|---|---|
| Error 404 al abrir el sitio | El archivo no se llama `index.html` o no está en la raíz |
| "Supabase sin configurar" | Faltan `SUPABASE_URL` o `SUPABASE_ANON_KEY` en el script |
| No se puede iniciar sesión | "Confirm email" está activado, o falta la función `email_por_usuario` |
| "Error de nube" al guardar | Políticas RLS incorrectas o tablas/funciones inexistentes (revisa la consola con F12) |
| Gráficos simplificados | Chart.js no cargó desde el CDN; se usan gráficos de respaldo |

---

## Limitaciones conocidas

- El plan gratuito de Supabase limita los correos y registros por hora, y pausa los proyectos tras una semana sin actividad.
- Requiere conexión a internet para cargar las librerías desde CDN.

---

## Licencia

Define aquí la licencia de tu proyecto (por ejemplo, [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html), [MIT](https://opensource.org/licenses/MIT) o [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)). Dado el tema de la plataforma, una licencia libre es lo más coherente.

## Autor

Tu nombre · tu correo o perfil de GitHub
