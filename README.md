# 💪 PowerFit - Frontend E-Commerce

[![React](https://img.shields.io/badge/React-19.1.1-blue.svg)](https://reactjs.org/)
[![Vite](https://img.shields.io/badge/Vite-7.1.7-646CFF.svg)](https://vitejs.dev/)
[![Vercel](https://img.shields.io/badge/Deploy-Vercel-black.svg)](https://vercel.com/)

## 📋 Descripción

Frontend SPA del sistema PowerFit, construido con React + Vite y desplegado en Vercel. Ofrece catálogo de productos, autenticación de usuarios, carrito de compras y rutas protegidas.

## ✨ Características

- 🎨 **UI moderna** con React Bootstrap
- 🔐 **Autenticación** con Context API
- 🛒 **Carrito de compras** con persistencia local
- 🔍 **Filtrado de productos** por categoría
- 🖼️ **Imágenes seguras** desde AWS S3 (HTTPS)
- 🌐 **SPA fallback** para rutas dinámicas
- ⚡ **Proxies serverless** en Vercel para backend HTTP

## 🏗️ Arquitectura

```
PowerFit/
├── api/                    # Funciones serverless (proxies)
│   ├── usuario.js          # Proxy a microservicio usuario
│   └── producto.js         # Proxy a microservicio producto
├── src/
│   ├── components/         # Componentes reutilizables
│   │   └── layout/         # Header, Footer, ProductCard, etc.
│   ├── contexts/           # Context API (Auth, Cart)
│   ├── pages/              # Vistas principales (Inicio, Login, Productos)
│   ├── routes/             # Configuración de rutas
│   ├── services/           # Llamadas a API backend
│   ├── utils/              # Utilidades (formatCurrency, toSecureUrl)
│   └── styles/             # Estilos globales
├── public/                 # Assets estáticos
└── vercel.json             # Config de Vercel (rewrites, SPA fallback)
```

## 🛠️ Tecnologías

- **Framework**: React 19.1.1
- **Build Tool**: Vite 7.1.7
- **Routing**: React Router 7.1.1
- **UI**: React Bootstrap 2.10.8
- **HTTP Client**: Axios
- **Estado**: Context API (AuthContext, CartContext)
- **Deployment**: Vercel (Edge Functions + SPA)

## 🚀 Inicio Rápido

### Prerrequisitos

- 📦 Node.js 18+ (recomendado: 20 LTS)
- 📦 pnpm o npm
- 🔌 Backend corriendo (microservicios usuario y producto)

### Instalación

1. **Clona el repositorio**

   ```bash
   cd PowerFit
   ```

2. **Instala dependencias**

   ```bash
   pnpm install
   # o: npm install
   ```

3. **Configura variables de entorno**

   Crea `.env.local` en la raíz de `PowerFit/`:

   ```env
   # URLs de los microservicios backend (local)
   VITE_API_USUARIO_URL=http://localhost:8082
   VITE_API_PRODUCTO_URL=http://localhost:8081

   # En producción (Vercel), estas se configuran en el dashboard
   # y apuntan a las IPs de EC2 o DNS
   ```

### 🏃 Ejecución Manual (Paso a Paso)

#### Opción 1: Servidor de desarrollo

```bash
# Con pnpm
pnpm run dev

# Con npm
npm run dev
```

**Acceso**: `http://localhost:5173`

#### Opción 2: Build y preview local

```bash
# 1. Compilar para producción
pnpm run build

# 2. Preview del build
pnpm run preview
```

**Acceso**: `http://localhost:4173`

#### Opción 3: Con variables de entorno específicas

```powershell
# Windows PowerShell
$env:VITE_API_USUARIO_URL='http://192.168.1.100:8082'; pnpm run dev
```

```bash
# Linux/Mac
VITE_API_USUARIO_URL=http://192.168.1.100:8082 pnpm run dev
```

### ✅ Verificar que está corriendo

- **Home**: `http://localhost:5173/`
- **Login**: `http://localhost:5173/login`
- **Productos**: `http://localhost:5173/productos`
- **Swagger backend**: `http://localhost:8082/swagger-ui/index.html` (usuario)

## 📖 Funcionalidades Principales

### 🔐 Autenticación

- **Registro**: `POST /api/usuario/api/v1/usuarios`
- **Login**: `POST /api/usuario/api/v1/usuarios/login` (BCrypt validation)
- **Context**: `AuthContext` maneja estado de sesión
- **Rutas Protegidas**: `RutaProtegida.jsx` bloquea acceso sin auth

**Ejemplo de uso**:

```javascript
import { useAuth } from '@hooks/useAuth'

const { estaAutenticado, usuario, login, logout } = useAuth()
```

### 🛒 Catálogo de Productos

- **Listar**: `GET /api/producto/api/v1/productos`
- **Filtrado**: Por categoría local (frontend)
- **Destacados**: Componente `FeaturedProducts` (primeros N productos)
- **Imágenes**: URLs HTTPS normalizadas con `toSecureUrl()`

**Estructura de producto**:

```javascript
{
  id: 1,
  name: "Creatina",
  category: "Suplementos",
  price: 19990,
  description: "...",
  image: "https://app-react-powerfit.s3.us-east-1.amazonaws.com/images/..."
}
```

### 🛒 Carrito de Compras

- **Estado**: `CartContext` con localStorage
- **Funciones**: `addToCart`, `removeFromCart`, `clearCart`, `getTotal`
- **Persistencia**: Sobrevive recargas de página

### 🌐 Proxies Serverless (Vercel)

Los archivos en `api/usuario.js` y `api/producto.js` actúan como proxies HTTPS→HTTP:

**Flujo**:

1. Frontend llama `/api/usuario/api/v1/usuarios/login` (HTTPS)
2. Función serverless recibe request
3. Reenvía a `http://<EC2_IP>:8082/api/v1/usuarios/login`
4. Retorna respuesta al frontend

**Beneficios**:

- Evita mixed content (HTTPS→HTTP)
- Simplifica CORS
- Oculta IPs de backend

### 📋 SPA Fallback

`vercel.json` define:

```json
{
  "rewrites": [
    { "source": "/api/producto/:path*", "destination": "/api/producto" },
    { "source": "/api/usuario/:path*", "destination": "/api/usuario" },
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

**Resultado**: Cualquier ruta 404 redirige a `index.html` → React Router decide qué mostrar (o `NoEncontrado.jsx`).

## 🧪 Testing

```bash
# Ejecutar tests
pnpm test

# Con cobertura
pnpm test -- --coverage
```

**Tests incluidos**:

- `Cart.spec.jsx` - Carrito de compras
- `formatCurrency.spec.js` - Utilidad de formato
- `ProductCard.spec.jsx` - Tarjeta de producto
- `Inicio.spec.jsx` - Página de inicio

## 🏗️ Build para Producción

```bash
# Generar build optimizado
pnpm run build

# Preview del build
pnpm run preview
```

**Output**: Carpeta `dist/` con assets optimizados.

## 🚀 Deployment en Vercel

### Configuración

1. **Conecta repositorio** en Vercel dashboard
2. **Variables de entorno** (Settings → Environment Variables):

   ```
   VITE_API_USUARIO_URL=http://<EC2_IP>:8082
   VITE_API_PRODUCTO_URL=http://<EC2_IP>:8081
   ```

3. **Deploy automático**: Cada push a `master` → nuevo despliegue

### Comandos de CI

```bash
# Build command (automático en Vercel)
npm run build

# Output directory
dist
```

## 🔒 Seguridad

### Mecanismos Implementados

- **Autenticación con Context API**:
  - Estado de sesión en `AuthContext`
  - Persistencia en `localStorage` (opcional)
  - Logout limpia estado y redirige

- **Rutas Protegidas**:
  - `RutaProtegida.jsx` valida autenticación
  - Redirección automática a `/login` si no autenticado
  - Protección de `/perfil`, `/orders`, `/checkout`

- **Comunicación Segura**:
  - HTTPS obligatorio en Vercel
  - Proxies serverless evitan mixed content
  - No expone IPs de backend en cliente

- **Validación Frontend**:
  - Validación de formularios antes de envío
  - Sanitización de inputs
  - Manejo de errores HTTP (401, 404, 500)

- **Imágenes Seguras**:
  - `toSecureUrl()` normaliza HTTP→HTTPS
  - URLs de S3 con HTTPS obligatorio
  - Prevención de mixed content warnings

- **Variables de Entorno**:
  - `.env.local` nunca en Git
  - Variables `VITE_*` expuestas solo necesarias
  - URLs de backend configurables por ambiente

### Recomendaciones para Producción

- Implementar tokens JWT en lugar de sesión simple
- Añadir refresh tokens
- CSRF protection si se usan cookies
- Content Security Policy (CSP) headers
- Rate limiting en llamadas API

## 📊 Cobertura de Tests

### Estadísticas Actuales

```bash
pnpm test
```

**Resultados**:

- ✅ **6 suites de tests**
- ✅ **100% tests pasados**
- 📦 **Cobertura estimada**: ~70%

### Tests Incluidos

**Tests de Componentes**:

- `Cart.spec.jsx` - Funcionalidad del carrito
- `ProductCard.spec.jsx` - Tarjetas de producto
- `Header.spec.jsx` - Navegación y header
- `Footer.spec.jsx` - Footer componente
- `Inicio.spec.jsx` - Página de inicio

**Tests Unitarios**:

- `formatCurrency.spec.js` - Utilidad de formato de moneda
- Tests de servicios API (mocks)

### Ejecutar con Cobertura

```bash
# Generar reporte de cobertura
pnpm test -- --coverage

# Ver reporte en: coverage/index.html
```

### Comandos de Testing

```bash
# Watch mode
pnpm test -- --watch

# Single run
pnpm test -- --run

# Con UI
pnpm test -- --ui
```

## 🔧 Configuración Avanzada

### Vite Config

`vite.config.js` define alias y resolución de paths:

```javascript
export default defineConfig({
  resolve: {
    alias: {
      '@': '/src',
      '@components': '/src/components',
      '@hooks': '/src/hooks',
      '@services': '/src/services'
    }
  }
})
```

**Uso**:

```javascript
import { useAuth } from '@hooks/useAuth'
import ProductCard from '@components/layout/ProductCard'
```

### Variables de Entorno

- **Local**: `.env.local` (no se sube a Git)
- **Producción**: Vercel dashboard
- **Prefijo**: `VITE_` para exponer al frontend

## 🐛 Troubleshooting

### Problemas Comunes

1. **Imágenes no cargan**
   - Verificar que URLs en BD sean HTTPS
   - Comprobar acceso a S3 bucket
   - `toSecureUrl()` normaliza HTTP→HTTPS automáticamente

2. **Error 404 en rutas**
   - Verificar `vercel.json` tenga SPA fallback: `{ "source": "/(.*)", "destination": "/index.html" }`
   - En local con Vite, esto no aplica (funciona por defecto)

3. **API no responde**
   - Verificar microservicios estén corriendo
   - Comprobar variables `VITE_API_*_URL`
   - Revisar logs de funciones serverless en Vercel

4. **Build falla**
   - Limpiar cache: `rm -rf node_modules dist && pnpm install`
   - Verificar versión de Node (18+)

## 📝 Scripts Disponibles

```json
{
  "dev": "vite",                          // Servidor desarrollo
  "build": "vite build",                  // Build producción
  "preview": "vite preview",              // Preview build
  "test": "vitest",                       // Ejecutar tests
  "lint": "eslint . --ext js,jsx",        // Linter
  "format": "prettier --write \"src/**/*.{js,jsx}\"" // Formateo
}
```

## 🤝 Contribución

1. Fork el proyecto
2. Crea rama feature (`git checkout -b feature/AmazingFeature`)
3. Commit cambios (`git commit -m 'Add AmazingFeature'`)
4. Push a rama (`git push origin feature/AmazingFeature`)
5. Abre Pull Request

## 📋 Convenciones

- **Componentes**: PascalCase (`ProductCard.jsx`)
- **Hooks**: camelCase con prefijo `use` (`useAuth.js`)
- **Servicios**: camelCase (`usuarioService.js`)
- **Estilos**: CSS modules o global en `styles/`

---

<div align="center">
  <sub>Desarrollado con ❤️ usando React + Vite</sub>
  <br>
  <sub>Desplegado en Vercel | Backend en AWS EC2</sub>
</div>
