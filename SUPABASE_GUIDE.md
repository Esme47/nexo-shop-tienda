# Configuración de Supabase - NEXO Shop

Sigue estos pasos para configurar tu base de datos en Supabase para que la tienda funcione correctamente.

## 1. Crear Tablas

Ejecuta el siguiente SQL en el **SQL Editor** de Supabase:

```sql
-- Tabla de Productos/Cursos
CREATE TABLE products (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    title TEXT NOT NULL,
    price DECIMAL NOT NULL,
    original_price DECIMAL,
    description TEXT,
    category TEXT,
    image TEXT,
    created_at TIMESTAMPTZ DEFAULT now()
);

-- Tabla de Pedidos/Inscripciones
CREATE TABLE orders (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    customer_name TEXT NOT NULL,
    customer_phone TEXT NOT NULL,
    customer_email TEXT,
    customer_city TEXT,
    items JSONB NOT NULL,
    total DECIMAL NOT NULL,
    status TEXT DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT now()
);
```

## 2. Configurar Políticas de Seguridad (RLS)

**¡IMPORTANTE!** Por defecto, las tablas son privadas. Debes habilitar RLS y añadir políticas.

### Para la tabla `products`:
Permitir que cualquiera pueda ver los productos.
```sql
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Permitir lectura pública de productos"
ON products FOR SELECT
TO anon
USING (true);
```

### Para la tabla `orders`:
**Seguridad Crítica:** En producción, la lectura de pedidos debe estar restringida a usuarios autenticados.

#### Opción A: Desarrollo (Menos Segura)
Permite insertar desde la web y leer (para el panel admin básico).
```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Permitir a cualquiera crear un pedido
CREATE POLICY "Permitir inserción pública de pedidos"
ON orders FOR INSERT
TO anon
WITH CHECK (true);

-- Permitir lectura pública (REQUERIDO para el Admin Panel actual, pero INSEGURO)
CREATE POLICY "Permitir lectura pública de pedidos"
ON orders FOR SELECT
TO anon
USING (true);

-- Permitir actualización de estado (Requerido para el Admin Panel)
CREATE POLICY "Permitir actualización pública de pedidos"
ON orders FOR UPDATE
TO anon
USING (true)
WITH CHECK (true);
```

#### Opción B: Producción (Recomendada)
Requiere Supabase Auth para ver los pedidos.
```sql
-- Solo permitir inserción a anónimos
CREATE POLICY "Permitir inserción pública de pedidos"
ON orders FOR INSERT
TO anon
WITH CHECK (true);

-- Solo permitir lectura a usuarios autenticados (Admin)
CREATE POLICY "Permitir lectura solo a admins"
ON orders FOR SELECT
TO authenticated
USING (true);

-- Solo permitir actualización a usuarios autenticados (Admin)
CREATE POLICY "Permitir actualización solo a admins"
ON orders FOR UPDATE
TO authenticated
USING (true)
WITH CHECK (true);
```

## 3. Advertencia de Seguridad

El panel de administración actual utiliza una contraseña estática en el código (`nexo2026admin`).
**Esto no es seguro para un entorno real con datos de clientes.**

**Recomendaciones para el futuro:**
1. Implementar **Supabase Auth** para el acceso administrativo.
2. Usar la **Opción B** de RLS para proteger los datos personales de los clientes.
3. No exponer información sensible en el código fuente del navegador.
