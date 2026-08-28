# Repo de ENSAYO · `prueba.yodesarrollo.mx`

Simulacro del cambio de dominio, **antes** de mover los 15 tableros de producción a
`tableros.yodesarrollo.mx`.

La página carga el `portero.js` **real** y le pega a los backends **reales**, así que
dice la verdad. Pero vive en un repo desechable con su propio custom domain, así que
si algo truena el radio de daño es **cero**: no toca ningún tablero, ninguna liga
que ya circula, ni ningún dato.

**Nada de esto está publicado todavía.** Son archivos locales.

## Qué hay aquí

| Archivo | Qué es |
|---|---|
| `index.html` | La pantalla de pruebas. Autocontenida: todo el CSS y el JS adentro (lo único externo son las fuentes de Google, igual que el resto del sistema) |
| `CNAME` | `prueba.yodesarrollo.mx` |
| `.nojekyll` | Para que GitHub Pages no procese nada y sirva el HTML tal cual |

## Cómo se despliega

1. **Crear el repo** `prueba-dominio` dentro de la organización **`yodesarrollomx`**.
   Público (Pages en cuenta gratis lo pide). Sin descripción, sin README.
2. **Subir los 3 archivos** de esta carpeta a la raíz del repo (`index.html`, `CNAME`,
   `.nojekyll`).
3. **Settings → Pages**: Source = `Deploy from a branch`, rama `main`, carpeta `/ (root)`.
4. **Custom domain**: escribir `prueba.yodesarrollo.mx` y guardar.
5. **DNS** (donde vive `yodesarrollo.mx`): un registro **CNAME** con nombre `prueba`
   apuntando a `yodesarrollomx.github.io`.
6. Esperar el certificado (de minutos a una hora) y palomear **Enforce HTTPS**.
7. Abrir `https://prueba.yodesarrollo.mx` y correr las pruebas.

> El custom domain del repo **sobreescribe** el de la organización. Por eso este
> ensayo no interfiere con nada de lo que ya sirve `yodesarrollomx`.

Si el repo se llamara distinto a `prueba-dominio`, hay que cambiarlo en dos lugares:
la constante `REPO_ENSAYO` dentro de `index.html`, y la cajita de texto de la prueba 6
(que se puede corregir a mano en el momento, sin tocar el archivo).

## Al terminar

Borrar el repo. No hay nada que conservar: ni datos, ni configuración, ni historial.

---

## Qué prueba, en una línea cada una

| # | Prueba | Qué contesta |
|---|---|---|
| 1 | Certificado y origen | ¿El subdominio sirve por https con certificado bueno? |
| 2 | El Portero real | ¿`portero.js` carga desde el dominio nuevo y arranca? |
| 3 | La sesión | ¿Viaja la sesión al origen nuevo? (no, y está bien) |
| 4 | Continuar con Google | ¿La consola de Google autoriza este origen? |
| 5 | Backends | ¿Contestan, hay CORS, y siguen cerrados sin credencial? |
| 6 | El fragmento | ¿El `#gas=…&clave=…&rol=…` sobrevive el 301? |

Detalles que conviene saber al leer los resultados:

- **La 2 no puede dar verde hasta que exista `tableros.yodesarrollo.mx`.** Es la única
  prueba que depende de que el dominio de producción ya esté puesto. Antes de eso
  saldrá en rojo con el mensaje "no pudo traer portero.js", y es correcto.
- **La 3 sale ámbar siempre** en una ventana limpia. Es el comportamiento esperado:
  la sesión del Portero se guarda por dominio y no viaja. Todos entran una vez más.
- **La 4 sale ámbar** hasta que alguien píchale el botón y elija su cuenta. La página
  no manda esa credencial a ningún lado ni inicia sesión en nada: solo comprueba que
  Google no rebote el origen.
- **La 6 requiere salir y volver**: navega a la liga vieja con un fragmento de adorno
  (`#gas=DEMO-NO-ES-SECRETO&clave=NO-ES-SECRETO&rol=editor` — **ahí no hay ninguna
  clave real**), GitHub redirige, y al aterrizar la página revisa si el `#` llegó
  entero.
- Ninguna prueba escribe: no guarda en Sheets, no toca el CRM, no manda bitácora.
  Cada una corre en su propio `try/catch`: si una revienta, las otras siguen.

---

## CRITERIO DE APROBACIÓN

Esto es lo que decide si se autoriza mover producción. Se corre **desde
`https://prueba.yodesarrollo.mx`**, no desde el disco ni desde `localhost`.

### Verde obligatorio (las cuatro)

| Prueba | Si no está en verde… |
|---|---|
| **1 · Certificado** | Sin candado no hay nada que probar. Se arregla el DNS / Enforce HTTPS y se repite. |
| **2 · El Portero carga** | Sin Portero no entra nadie a ningún tablero. Bloqueante absoluto. |
| **5 · Backends** | Ver abajo: hay dos formas distintas de fallar. |
| **6 · El fragmento** | Las ligas ya enviadas por correo y WhatsApp abrirían el tablero sin rol ni clave. Bloqueante. |

### Verde en la 4, o certeza equivalente

La prueba **4 (Google)** tiene que estar en verde **o** haber constancia de que
`tableros.yodesarrollo.mx` ya quedó dado de alta en *Orígenes de JavaScript
autorizados* del OAuth Client. **Si la 4 sale en rojo, no se mueve nada**: nadie
podría entrar con su cuenta.

Ojo con el matiz: la 4 en ámbar ("el botón montó, nadie completó el login") **no
alcanza**. Alguien tiene que picarle y elegir su cuenta.

### Ámbar aceptable

Solo en la **3** (sesión vacía). Es lo esperado, se avisa a todos antes del corte
y se acabó.

### Rojo que detiene todo

- **Cualquier rojo en la 5.** Dos casos, y los dos frenan:
  - *Un backend entrega datos sin credencial* → **hallazgo grave de seguridad**.
    Ese hueco ya existe hoy, con dominio nuevo o sin él, pero no se estrena dominio
    con eso abierto.
  - *Un backend no contesta o lo tapa CORS desde el origen nuevo* → ese tablero
    quedaría ciego al mudarse.
  - Excepción ya conocida y anotada: el backend de **textos del cuestionario** es
    público a propósito; sale en ámbar con su explicación, no en rojo.
- **Rojo en la 6.** Es el riesgo más fino de toda la mudanza y el más caro de
  descubrir después.

### La regla corta

> **Todo verde salvo la 3 → se autoriza el corte.**
> **Un solo rojo → no se mueve nada y se arregla primero.**
