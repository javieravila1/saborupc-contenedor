# Contratos de comunicación — SaborUPC 2.0

## 1. Propósito

Este documento define los contratos de comunicación entre los micro frontends de SaborUPC 2.0.

La comunicación entre micro frontends se realiza mediante eventos del navegador (`CustomEvent`). Los micro frontends no importan código directamente de otros micro frontends ni comparten estado global.

---

## 2. Contratos de eventos

| Evento                | Publicador | Consumidor | `detail`                        | Versión |
| --------------------- | ---------- | ---------- | ------------------------------- | ------- |
| `carrito:agregar`     | Catálogo   | Carrito    | `{ id, nombre, precio }`        | 1       |
| `carrito:actualizado` | Carrito    | Contenedor | `{ cantidad, total }`           | 1       |
| `usuario:cambio`      | Perfil     | Contenedor | `{ nombre }`                    | 1       |
| `pedido:confirmado`   | Carrito    | Pedidos    | `{ version, id, items, total }` | 1       |
| `pedido:estado`       | Pedidos    | Contenedor | `{ version, id, estado }`       | 1       |

---

## 3. Evento `carrito:agregar`

### Publicador

Micro frontend Catálogo.

### Consumidor

Micro frontend Carrito.

### Estructura

```javascript
{
    id: Number,
    nombre: String,
    precio: Number
}
```

### Ejemplo

```javascript
window.dispatchEvent(
    new CustomEvent('carrito:agregar', {
        detail: {
            id: 1,
            nombre: 'Hamburguesa SaborUPC',
            precio: 15000
        }
    })
);
```

---

## 4. Evento `carrito:actualizado`

### Publicador

Micro frontend Carrito.

### Consumidor

Contenedor.

### Estructura

```javascript
{
    cantidad: Number,
    total: Number
}
```

### Ejemplo

```javascript
window.dispatchEvent(
    new CustomEvent('carrito:actualizado', {
        detail: {
            cantidad: 2,
            total: 30000
        }
    })
);
```

---

## 5. Evento `usuario:cambio`

### Publicador

Micro frontend Perfil.

### Consumidor

Contenedor.

### Estructura

```javascript
{
    nombre: String
}
```

### Ejemplo

```javascript
window.dispatchEvent(
    new CustomEvent('usuario:cambio', {
        detail: {
            nombre: 'Juan'
        }
    })
);
```

---

## 6. Evento `pedido:confirmado`

### Publicador

Micro frontend Carrito.

### Consumidor

Micro frontend Seguimiento de Pedidos.

### Propósito

Notificar que el usuario confirmó un pedido y proporcionar al micro frontend de Seguimiento la información necesaria para registrar y mostrar el pedido.

### Estructura

```javascript
{
    version: Number,
    id: String,
    items: Array,
    total: Number
}
```

### Ejemplo

```javascript
window.dispatchEvent(
    new CustomEvent('pedido:confirmado', {
        detail: {
            version: 1,
            id: 'PED-001',
            items: [
                {
                    id: 1,
                    nombre: 'Hamburguesa SaborUPC',
                    precio: 15000
                }
            ],
            total: 15000
        }
    })
);
```

### Compatibilidad hacia atrás

Los consumidores deben utilizar únicamente los campos definidos en el contrato que necesiten para funcionar.

Si en una versión posterior se agrega un campo nuevo al objeto `detail`, los consumidores existentes deben continuar funcionando.

Por ejemplo, si posteriormente se agrega:

```javascript
{
    version: 1,
    id: 'PED-001',
    items: [...],
    total: 15000,
    fecha: '2026-09-30T20:00:00'
}
```

un consumidor que solamente utilice `version`, `id`, `items` y `total` debe continuar funcionando sin modificaciones.

---

## 7. Evento `pedido:estado`

### Publicador

Micro frontend Seguimiento de Pedidos.

### Consumidor

Contenedor.

### Propósito

Notificar al contenedor cada vez que un pedido cambia de estado.

### Estados permitidos

```text
Recibido
En preparación
En camino
Entregado
```

### Estructura

```javascript
{
    version: Number,
    id: String,
    estado: String
}
```

### Ejemplo

```javascript
window.dispatchEvent(
    new CustomEvent('pedido:estado', {
        detail: {
            version: 1,
            id: 'PED-001',
            estado: 'En preparación'
        }
    })
);
```

---

## 8. Reglas de compatibilidad

1. Los nombres de los eventos forman parte del contrato y no deben cambiarse sin actualizar consumidores y documentación.
2. El campo `version` identifica la versión del contrato utilizado por el evento.
3. Los consumidores no deben depender de campos que no estén documentados.
4. Agregar campos al objeto `detail` no debe romper consumidores existentes.
5. Los micro frontends se comunican mediante eventos y no mediante imports directos entre sus repositorios.
6. No se utiliza un estado global compartido entre micro frontends.

---

## 9. Micro frontends

| Micro frontend | Ruta         | Tecnología                 | Puerto |
| -------------- | ------------ | -------------------------- | -----: |
| Contenedor     | `#/`         | JavaScript                 |   8080 |
| Catálogo       | `#/catalogo` | Vue 3                      |   8081 |
| Carrito        | `#/carrito`  | JavaScript                 |   8082 |
| Perfil         | `#/perfil`   | JavaScript / Web Component |   8083 |
| Pedidos        | `#/pedidos`  | JavaScript                 |   8084 |
