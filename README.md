# Tienda Online 🛒
**SENA — Análisis y Desarrollo de Software | Ficha 3066446**

Aplicación web full stack para gestión de una tienda online de tecnología.  
Construida con **Express + Node.js** en el backend y **Vite + React** en el frontend,  
conectada a **MongoDB Atlas** como base de datos en la nube.

---

## Tecnologías utilizadas

| Capa | Tecnología | Para qué se usa |
|------|-----------|-----------------|
| Backend | Node.js + Express | Servidor REST y manejo de rutas |
| ODM | Mongoose | Modelado y conexión con MongoDB |
| Base de datos | MongoDB Atlas | Almacenamiento en la nube |
| Frontend | Vite + React | Interfaz de usuario dinámica |
| HTTP client | Axios | Peticiones del frontend al backend |
| Navegación | React Router | Rutas entre páginas en React |
| Entorno | dotenv | Variables de entorno seguras |
| Desarrollo | Nodemon | Reinicio automático del servidor |

---

## Estructura del proyecto
```
Tienda_Linea/
├── backend/
│   ├── config/
│   │   └── db.js              # Conexión a MongoDB Atlas
│   ├── data/
│   │   └── seed.js            # Script para poblar la BD con datos de prueba
│   ├── models/
│   │   ├── cliente.js         # Schema de clientes
│   │   ├── producto.js        # Schema de productos
│   │   ├── pedido.js          # Schema de pedidos (con JSON anidado)
│   │   └── proveedor.js       # Schema de proveedores
│   ├── routes/
│   │   ├── clientes.js        # CRUD de clientes
│   │   ├── productos.js       # CRUD de productos + validación de stock
│   │   ├── pedidos.js         # CRUD de pedidos + validación de stock
│   │   └── proveedores.js     # CRUD de proveedores
│   ├── .env                   # Variables de entorno (no se sube a GitHub)
│   ├── .gitignore
│   ├── index.js               # Servidor principal
│   └── package.json
└── frontend/
    ├── src/
    │   ├── api/
    │   │   └── axios.js       # Configuración base de Axios
    │   ├── components/
    │   │   ├── Navbar.jsx     # Barra de navegación
    │   │   └── Modal.jsx      # Modal personalizado (reemplaza alert/confirm)
    │   ├── pages/
    │   │   ├── Clientes.jsx   # CRUD de clientes
    │   │   ├── Productos.jsx  # CRUD de productos
    │   │   ├── Pedidos.jsx    # CRUD de pedidos con productos anidados
    │   │   ├── Proveedores.jsx# CRUD de proveedores
    │   │   └── Consultas.jsx  # 6 consultas ejecutables
    │   ├── App.jsx            # Rutas de la aplicación
    │   ├── App.css            # Estilos globales
    │   └── main.jsx           # Punto de entrada de React
    └── package.json
```

---

## Arquitectura
```
Frontend (Vite + React)          Backend (Express + Node.js)
http://localhost:5173    ←→     http://localhost:3000
                                        ↕
                              MongoDB Atlas (nube)
```

El frontend nunca se comunica directamente con la base de datos.  
Todas las operaciones pasan por el backend, que es el único punto de contacto con MongoDB Atlas.

---

## Backend

### `config/db.js`
Establece la conexión con MongoDB Atlas usando Mongoose.  
Si la conexión falla, el proceso se detiene con `process.exit(1)` para evitar que el servidor corra sin base de datos.

### `models/`
Define la estructura de cada documento usando **Schemas de Mongoose**.  
Mongoose valida los datos antes de guardarlos en la base de datos.

El modelo más especial es `pedido.js` — usa un **schema anidado** (`itemSchema`) para representar el array de productos dentro de cada pedido (JSON anidado).
```javascript
// Ejemplo: schema anidado en pedido.js
const itemSchema = new mongoose.Schema({
  nombre:   { type: String, required: true },
  cantidad: { type: Number, required: true },
  precio:   { type: Number, required: true }
}, { _id: false });

const pedidoSchema = new mongoose.Schema({
  cliente_id: { type: String, required: true },
  productos:  [itemSchema],   // array de itemSchema
  total:      { type: Number, required: true },
  estado:     { type: String, enum: ['pendiente', 'procesando', 'entregado'] }
});
```

### `routes/`
Cada archivo define los endpoints CRUD para su colección usando el **Router de Express**:

| Método | Ruta | Acción |
|--------|------|--------|
| GET | `/api/clientes` | Obtener todos |
| GET | `/api/clientes/:id` | Obtener uno por ID |
| POST | `/api/clientes` | Crear nuevo |
| PUT | `/api/clientes/:id` | Actualizar |
| DELETE | `/api/clientes/:id` | Eliminar |

El mismo patrón aplica para `/api/productos`, `/api/pedidos` y `/api/proveedores`.

#### Validación de stock en productos
Al actualizar un producto, el backend compara el stock nuevo con el actual.  
Si aumentó, incluye una propiedad `alerta` en la respuesta para notificar al frontend.

#### Validación de stock en pedidos
Al crear un pedido, el backend verifica que la cantidad de cada producto no supere el stock disponible.  
Si hay insuficiencia, rechaza el pedido con un error 400 y un array de alertas detalladas.

### `index.js`
Servidor principal. Inicializa Express, conecta a MongoDB, registra los middlewares y monta las rutas.  
El middleware **CORS** es esencial para permitir peticiones del frontend (puerto 5173) al backend (puerto 3000).

### `data/seed.js`
Script de un solo uso para poblar la base de datos con datos de prueba.  
Limpia todas las colecciones con `deleteMany()` e inserta documentos con `insertMany()`.
```bash
node data/seed.js
```

---

## Frontend

### `src/api/axios.js`
Instancia de Axios con la URL base del backend centralizada.  
Todos los componentes importan esta instancia en lugar de escribir la URL completa cada vez.

### `src/components/Navbar.jsx`
Barra de navegación con enlaces a cada sección usando `NavLink` de React Router.  
Aplica automáticamente una clase CSS al enlace de la página activa.

### `src/components/Modal.jsx`
Modal personalizado que reemplaza los `alert()` y `confirm()` nativos del navegador.  
Recibe cuatro props: `tipo`, `mensaje`, `onAceptar` y `onCancelar`.  
Muestra diferentes íconos y botones según el tipo: `alerta`, `confirmar` o `exito`.

### `src/pages/`
Cada página usa los hooks de React:
- `useState` — para manejar el estado del formulario, los datos y el modal
- `useEffect` — para cargar los datos cuando la página se monta

**`Pedidos.jsx`** es el más complejo porque permite agregar múltiples productos dinámicamente.  
El total se calcula automáticamente con `reduce()` multiplicando cantidad × precio.

**`Consultas.jsx`** ejecuta 6 consultas sobre los datos traídos del backend:

| # | Consulta |
|---|---------|
| 1 | Clientes mayores de 30 años |
| 2 | Productos con precio ≤ $100.000 |
| 3 | Clientes de Cali o Bogotá |
| 4 | Productos de categorías específicas |
| 5 | Condición AND: precio > $100.000 y stock > 10 |
| ★ | Reto: pedidos con Laptop Lenovo y total > $500.000 |

### `App.jsx`
Define las rutas de la aplicación con **React Router**.  
La ruta raíz `/` redirige automáticamente a `/clientes`.

### `App.css`
Estilos globales con **variables CSS** en `:root` para colores, bordes y radios consistentes en toda la app.

---

## Base de datos — MongoDB Atlas

MongoDB Atlas es el servicio en la nube de MongoDB.  
Los datos viven en servidores de AWS y se replican en 3 nodos automáticamente (cluster M0 gratuito).

### Colecciones

| Colección | Documentos de prueba | Descripción |
|-----------|---------------------|-------------|
| `clientes` | 4 | Compradores registrados |
| `productos` | 4 | Catálogo de artículos |
| `pedidos` | 2 | Órdenes con productos anidados |
| `proveedores` | 2 | Empresas suministradoras |

### Flujo de una operación — ejemplo: agregar un cliente
```
1. Usuario llena el formulario en Clientes.jsx y presiona Agregar
2. React ejecuta guardar() → api.post('/clientes', form)
3. Axios envía POST a http://localhost:3000/api/clientes
4. Express recibe la petición en routes/clientes.js
5. Mongoose ejecuta Cliente.create(req.body) y valida el schema
6. MongoDB Atlas guarda el documento en la colección clientes
7. El backend responde con { ok: true, data: clienteCreado }
8. React ejecuta cargar() y actualiza la tabla sin recargar la página
```

---

## Instalación y ejecución

### Requisitos
- Node.js v18 o superior
- Cuenta en MongoDB Atlas

### 1. Clonar el repositorio
```bash
git clone https://github.com/TU_USUARIO/tienda-linea.git
cd tienda-linea
```

### 2. Configurar el backend
```bash
cd backend
npm install
```

Crea el archivo `.env` en la carpeta `backend/`:
```
MONGODB_URI=mongodb+srv://usuario:password@cluster.mongodb.net/Tienda_Linea?retryWrites=true&w=majority&authSource=admin
PORT=3000
```

### 3. Poblar la base de datos
```bash
node data/seed.js
```

### 4. Iniciar el backend
```bash
npm run dev
```

### 5. Configurar el frontend
```bash
cd ../frontend
npm install
npm run dev
```

### 6. Abrir en el navegador
```
http://localhost:5173
```

---

## Endpoints de la API

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/clientes` | Listar todos los clientes |
| POST | `/api/clientes` | Crear cliente |
| PUT | `/api/clientes/:id` | Actualizar cliente |
| DELETE | `/api/clientes/:id` | Eliminar cliente |
| GET | `/api/productos` | Listar todos los productos |
| POST | `/api/productos` | Crear producto |
| PUT | `/api/productos/:id` | Actualizar producto + alerta de stock |
| DELETE | `/api/productos/:id` | Eliminar producto |
| GET | `/api/pedidos` | Listar todos los pedidos |
| POST | `/api/pedidos` | Crear pedido + validación de stock |
| PUT | `/api/pedidos/:id` | Actualizar pedido |
| DELETE | `/api/pedidos/:id` | Eliminar pedido |
| GET | `/api/proveedores` | Listar todos los proveedores |
| POST | `/api/proveedores` | Crear proveedor |
| PUT | `/api/proveedores/:id` | Actualizar proveedor |
| DELETE | `/api/proveedores/:id` | Eliminar proveedor |

---

*Proyecto desarrollado como actividad práctica del programa Análisis y Desarrollo de Software — SENA Regional Cauca*
