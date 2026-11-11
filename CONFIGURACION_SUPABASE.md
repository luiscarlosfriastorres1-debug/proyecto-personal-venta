# Configuración de pedidos compartidos con Supabase

El proyecto ya está preparado para guardar pedidos en una tabla Supabase y sincronizar el panel en tiempo real. Antes de publicarlo, completa estos pasos.

## 1. Crear el proyecto
1. Entra en https://supabase.com/ y crea un proyecto.
2. En el panel del proyecto abre **SQL Editor** y ejecuta el SQL de la sección siguiente.
3. En **Project Settings > API** copia el Project URL y la clave pública (Publishable key o anon key).
4. En `index.html` y `admin.html`, reemplaza `PEGA_AQUI_TU_PROJECT_URL` y `PEGA_AQUI_TU_PUBLISHABLE_O_ANON_KEY` por esos valores. Usa la misma pareja en ambos archivos. Nunca coloques una `service_role` o `secret key` en el navegador.

## 2. SQL de la tabla y permisos
Ejecuta este SQL en SQL Editor:

```sql
create table if not exists public.pedidos (
  id uuid primary key,
  f jsonb not null,
  status text not null default 'Nuevo' check (status in ('Nuevo','Proceso','Entregado')),
  paid boolean not null default false,
  notes text not null default '',
  created bigint not null
);

alter table public.pedidos enable row level security;

-- Cualquier visitante puede crear un pedido, pero no leerlos.
create policy "public can create orders"
on public.pedidos for insert to anon, authenticated
with check (status = 'Nuevo' and paid = false and notes = '');

-- Solo usuarios autenticados (administradores) pueden consultar y administrar pedidos.
create policy "admins can read orders"
on public.pedidos for select to authenticated using (true);
create policy "admins can update orders"
on public.pedidos for update to authenticated using (true) with check (true);
create policy "admins can delete orders"
on public.pedidos for delete to authenticated using (true);

-- Habilita cambios en tiempo real para la tabla.
alter publication supabase_realtime add table public.pedidos;
```

Si la última instrucción indica que la tabla ya está en la publicación, puedes ignorar ese mensaje.

## 3. Crear el usuario administrador
1. En Supabase abre **Authentication > Users**.
2. Pulsa **Add user** y crea el correo y contraseña que usarás en `admin.html`.
3. Desactiva el registro público de usuarios en **Authentication > Settings > User Signups**. No compartas la cuenta de administrador.

## 4. Publicación
Sube `index.html` y `admin.html` a tu hosting. Ambos deben estar publicados bajo el mismo proyecto web. El pedido se registra en Supabase antes de abrir WhatsApp. Si falla el registro, no se abre WhatsApp para evitar que el pedido se pierda silenciosamente.

## Notas
- `admin.html` requiere iniciar sesión con el usuario creado en Supabase.
- Los pedidos nuevos aparecen en tiempo real en el panel cuando está abierto.
- La clave pública (anon/publishable) puede estar en el frontend; la seguridad depende de las políticas RLS. Nunca uses la clave `service_role`/`secret` en estos archivos.
- El panel actual no implementa recuperación de contraseña; puedes restablecerla desde Supabase Authentication.
