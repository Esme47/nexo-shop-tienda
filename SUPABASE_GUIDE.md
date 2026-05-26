# Guía de Corrección: Supabase y Seguridad

Esta guía detalla las fallas encontradas en la configuración de Supabase para NEXO Shop y cómo corregirlas.

## 1. Diagnóstico de Fallas

### 🔐 Exposición de Credenciales
*   **Falla:** La `SUPABASE_KEY` (anon key) está expuesta en el código fuente del frontend.
*   **Valoración:** Esto es normal para aplicaciones de una sola página (SPA), pero **solo si el RLS (Row Level Security) está bien configurado**. Si el RLS no está activo o está mal configurado, cualquier usuario podría borrar o modificar todos tus productos.

### ⛔ Falla de Escritura (RLS)
*   **Falla:** Se detectó el error `new row violates row-level security policy for table "products"` al intentar insertar datos desde fuera del editor de Supabase.
*   **Valoración:** La tabla tiene RLS activado pero no tiene una política que permita inserciones (INSERT) para el rol `anon`.

---

## 2. Cómo Corregir (Pasos en Supabase)

Copia y pega los siguientes comandos en el **SQL Editor** de tu panel de Supabase:

### A. Habilitar Lectura Pública
Permite que cualquier visitante pueda ver los productos (necesario para que la tienda funcione).

```sql
-- Habilitar RLS
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Crear política de lectura para todos
CREATE POLICY "Permitir lectura pública"
ON products FOR SELECT
TO anon
USING (true);
```

### B. Gestión Segura de Datos (Escritura)
Para una tienda real, **NO** debes permitir que usuarios anónimos (`anon`) modifiquen tu base de datos.

**Recomendación de Seguridad:**
Las políticas de `INSERT`, `UPDATE` y `DELETE` deben estar restringidas a usuarios autenticados con rol de administrador.

```sql
-- Ejemplo: Solo usuarios autenticados pueden insertar productos
CREATE POLICY "Solo admin puede insertar"
ON products FOR INSERT
TO authenticated
WITH CHECK (true);
```

Si decides permitir inserciones anónimas para pruebas (¡no recomendado en producción!), usa:
```sql
-- Política temporal de prueba (BORRAR DESPUÉS)
CREATE POLICY "Permitir inserción temporal anon"
ON products FOR INSERT
TO anon
WITH CHECK (true);
```

---

## 3. Valoración General de la Tienda

*   **Diseño:** Interfaz limpia y amigable para móviles. Se ha mejorado el contraste y los efectos visuales.
*   **Funcionalidad:** Se añadió un sistema de carrito basado en `localStorage` para que los usuarios puedan guardar productos sin necesidad de login.
*   **Seguridad:** Se implementó un "Modo Debug" que oculta los errores técnicos a los clientes finales (solo visible añadiendo `?debug` a la URL).

---

## 4. Estructura de Tabla Sugerida
Asegúrate de que tu tabla `products` tenga estas columnas:
*   `id`: int8 (Primary Key)
*   `title`: text
*   `price`: numeric
*   `original_price`: numeric (opcional)
*   `image`: text (URL de la imagen)
*   `created_at`: timestamptz

---

## 5. Cómo Redesplegar en Vercel

Si el despliegue anterior fue eliminado o necesitas uno nuevo, sigue estos pasos:

1.  **Sube los cambios a GitHub**: Asegúrate de que esta versión con las mejoras esté en tu repositorio.
2.  **Conecta a Vercel**:
    - Ve a [vercel.com](https://vercel.com) e inicia sesión.
    - Haz clic en **"Add New"** > **"Project"**.
    - Importa tu repositorio `nexo-shop-tienda`.
3.  **Configuración**:
    - No necesitas cambiar los "Build Settings" ya que es un sitio estático.
    - Haz clic en **"Deploy"**.
4.  **Verificación**: Vercel te dará una nueva URL (ej: `nexo-shop-tienda.vercel.app`). ¡Esa será tu tienda actualizada!
