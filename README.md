# ♡ Personajes ocupados — registro del clan

Aplicación responsive en HTML/CSS/JavaScript con Supabase (PostgreSQL, Auth, RLS y Realtime). El proyecto está preparado, pero **no queda conectado ni publicado hasta que configures tu propio proyecto de Supabase y despliegues estos archivos**. No contiene credenciales reales.

## Archivos
- `index.html`: página pública, acceso de administración y formulario.
- `styles.css`: diseño negro/morado con corazones, adaptable a móvil.
- `app.js`: búsqueda, ordenamiento, estadísticas, CRUD, login, puntos y realtime.
- `config.example.js`: plantilla de configuración; cópiala como `config.js`.
- `supabase.sql`: tabla, políticas RLS, función atómica y datos iniciales.

## 1. Crear la base de datos
1. Entra en https://supabase.com/ y crea una cuenta/proyecto.
2. Abre **SQL Editor → New query**, pega el contenido completo de `supabase.sql` y ejecútalo.
3. En **Project Settings → API**, copia Project URL y la clave pública `anon`/`publishable`.
4. Copia `config.example.js` a un archivo llamado `config.js` en la misma carpeta. Sustituye los dos valores de ejemplo por los tuyos.
5. No compartas ni coloques nunca una clave `service_role`/secret en el navegador. La clave pública es visible por diseño; RLS es lo que limita las operaciones.

## 2. Crear el administrador
1. En Supabase abre **Authentication → Users → Add user** y crea el usuario con correo y contraseña. Si aparece la opción, confirma el correo.
2. En **SQL Editor** puedes asignar el rol de administrador al usuario creado con esta consulta, sustituyendo `UUID_DEL_USUARIO` por el ID de Authentication → Users:
   ```sql
   update auth.users
   set raw_app_meta_data = coalesce(raw_app_meta_data, '{}'::jsonb) || '{"role":"admin"}'::jsonb
   where id = 'UUID_DEL_USUARIO';
   ```
3. Cierra la sesión y vuelve a entrar en la web. Solo las cuentas cuyo `app_metadata.role` sea `admin` pasan la verificación.
4. En **Authentication → URL Configuration**, agrega la URL final de tu web a Site URL / Redirect URLs para que la recuperación de contraseña funcione.
5. Configura SMTP/correo en Supabase si quieres que los enlaces de recuperación lleguen de forma fiable. El servicio de correo de prueba tiene límites.

## 3. Publicar y compartir
Opción sencilla: Netlify (o GitHub Pages).
1. Asegúrate de que `index.html`, `styles.css`, `app.js` y `config.js` estén en la misma carpeta.
2. Sube esos archivos a Netlify mediante el despliegue manual de carpeta (o a un repositorio GitHub y conéctalo a Netlify).
3. La página es estática, por lo que no necesita servidor propio. Copia el enlace HTTPS que te dé Netlify y compártelo con el clan.
4. En Supabase añade ese dominio en **Authentication → URL Configuration**.
5. Revisa en **Database → Replication** que Realtime esté habilitado para la tabla `public.characters` si el proyecto no lo habilita automáticamente. La página vuelve a consultar la tabla al recibir eventos.

**Importante para Netlify:** `config.js` contiene solo URL y clave pública; está bien que el navegador las descargue. La seguridad debe permanecer en RLS y en la verificación del rol. Nunca incluyas secretos de servidor.

## 4. Probarlo en dos dispositivos
1. Abre el enlace en un teléfono sin iniciar sesión: debe mostrar personajes, búsqueda, ordenamiento y estadísticas; no debe mostrar botones para modificar puntos.
2. En otro dispositivo, inicia sesión como administrador.
3. Pulsa +1 o −1. Verifica que los puntos y el ranking cambien en ambos dispositivos.
4. Recarga la página y comprueba que los puntos persisten.
5. Prueba editar, crear y eliminar un personaje. El borrado pide confirmación.
6. Prueba −5 cuando el personaje tenga menos de 5 puntos: la función atómica evita valores negativos.
7. Crea un usuario de prueba sin `app_metadata.role=admin`; debe poder leer, pero no crear/editar/eliminar ni usar la función de puntos.

## 5. Sobre las puntuaciones iniciales
Se han aplicado directamente las coincidencias confirmadas: Jinshi (46), Mao Mao (48), Luka (2), Shisui (4), Denji (5), Nino Nakano (3), Hange Zoë (21), Mitsuri Kanroji (29), Mitsuba Sousuke de *Toilet-Bound Hanako-kun* (17), Arima Kana (9) y Suguru Geto (3). Los demás puntajes están preservados en registros editables `PENDIENTE · ...`, para no adjudicarlos por suposición. En particular, no se fusionaron `Ahko` con `Ah-Duo` ni `Al Hoshino` con `Ai Hoshino`. La forma `Geto Sugurk` se normalizó como `Suguru Geto`, interpretando que ese era el personaje al que te referías.

Coincidencias probables para que decidas:
Las coincidencias `Mitsuba` → `Mitsuba Sousuke`, `Mitsuri` → `Mitsuri Kanroji`, `zoe` → `Hange Zoë`, `kana` → `Arima Kana` y `geto` → `Suguru Geto` fueron confirmadas y ya están asignadas.
- `otogi` podría ser `Otogi Katsura`.
- `sebastián` y `kaneki` no aparecen en la lista inicial.
- `Noe` y `rimuru` tampoco aparecen en la lista inicial.
- `Al Hoshino` se conserva aparte de `Ai Hoshino`.

Al confirmar una coincidencia, edita el personaje oficial y actualiza sus puntos; después elimina el registro pendiente correspondiente. Así evitas contar dos veces esos puntos. No se han inventado edades, series ni miembros.

## Seguridad y límites
- Lectura pública deliberada; escritura protegida por RLS y rol de `app_metadata`.
- El cambio de puntos se realiza con una función SQL que bloquea la fila y evita actualizaciones concurrentes perdidas.
- Los puntos nunca bajan de cero.
- La página no muestra correos ni datos privados de administradores.
- Las operaciones de creación/edición y puntos necesitan conexión; si falla, la interfaz muestra un error.
- No se puede verificar el funcionamiento entre teléfonos hasta conectar el proyecto real y publicar el sitio.
