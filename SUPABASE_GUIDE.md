# Configuración de Supabase para NEXO Shop

Sigue estos pasos para conectar tu tienda con Supabase y asegurar tus datos.

## 1. Crear Tablas

Ejecuta el siguiente código en el **SQL Editor** de tu panel de Supabase:

```sql
-- Tabla de Productos
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  title TEXT NOT NULL,
  image TEXT NOT NULL,
  price NUMERIC NOT NULL,
  original_price NUMERIC,
  category TEXT DEFAULT 'Otros',
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tabla de Pedidos (Orders)
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  customer_name TEXT NOT NULL,
  customer_phone TEXT NOT NULL,
  customer_email TEXT,
  customer_city TEXT,
  items JSONB NOT NULL,
  total NUMERIC NOT NULL,
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## 2. Políticas de Seguridad (RLS) - ¡IMPORTANTE!

Por defecto, Supabase bloquea todo acceso. Debes configurar estas políticas para que la tienda funcione.

### Opción A: Prototipo Rápido (Menos Seguro)
*Usa esto solo para pruebas iniciales. Permite que cualquiera inserte pedidos pero no protege la privacidad de los clientes si alguien conoce tu URL de API.*

```sql
-- Permitir lectura pública de productos
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Permitir lectura pública de productos" ON products FOR SELECT USING (true);

-- Permitir insertar pedidos a cualquiera (necesario para el checkout)
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Permitir inserción pública de pedidos" ON orders FOR INSERT WITH CHECK (true);

-- ADVERTENCIA: Las siguientes líneas permiten que CUALQUIERA vea o edite los datos de tus clientes.
-- Solo usar para pruebas. En producción, usa la Opción B.
CREATE POLICY "Permitir lectura pública de pedidos" ON orders FOR SELECT USING (true);
CREATE POLICY "Permitir actualización pública de pedidos" ON orders FOR UPDATE USING (true);
```

### Opción B: Producción (Recomendado)
Para proteger los datos de tus clientes (Nombres, Teléfonos), debes usar **Supabase Auth**.
1. Crea un usuario en la sección **Authentication** de Supabase.
2. Cambia las políticas de `orders` para que solo usuarios autenticados puedan ver o actualizar:

```sql
-- Solo el administrador (tú) puede ver y editar pedidos
CREATE POLICY "Solo admin ve pedidos" ON orders FOR SELECT TO authenticated USING (true);
CREATE POLICY "Solo admin edita pedidos" ON orders FOR UPDATE TO authenticated USING (true);
```

*Nota: Si usas la Opción B, deberás implementar el login de Supabase en el panel de administración de `index.html`.*

## 3. Credenciales en index.html

Busca estas líneas al principio del script en `index.html` y reemplaza con tus valores:

```javascript
const SUPABASE_URL = 'https://tu-proyecto.supabase.co';
const SUPABASE_KEY = 'tu-anon-key';
```

## 4. Notas Técnicas

- **UUIDs:** El código JS usa comillas simples para manejar los IDs de Supabase: `addToCart('${p.id}')`. No las elimines o el carrito fallará.
- **Admin Password:** La contraseña `nexo2026admin` es puramente cosmética para el prototipo. Para seguridad real, utiliza la Opción B de las políticas RLS.
- **Campos de Pedido:** La aplicación espera `customer_name`, `customer_phone`, `customer_email`, `customer_city`, `total`, `items` (JSONB) y `status`.
