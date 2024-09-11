# microfronts-angular-nx

Para crear un sistema Angular utilizando NX con Module Federation y tres microfrontends (Login, Listar Productos y Crear Productos), puedes seguir estos pasos generales. La idea es dividir la aplicación en microfrontends para cada funcionalidad y orquestarlos usando Module Federation.

## 1. Crear el Monorepo con NX y configurar Module Federation
Primero, debes instalar NX y configurar el monorepo.

### Instalar NX
 npx create-nx-workspace@latest my-workspace

### Selecciona las opciones adecuadas:

¿Qué tipo de aplicación quieres construir? → Angular
¿Aplicación o Biblioteca? → Application
Nombre de la aplicación → shell
¿Quieres usar el formato de archivo Nx Cloud? → No
Esto crea la estructura inicial.

## 2. Instalar el Plugin de Module Federation
Dentro de tu workspace, navega a la carpeta del shell y agrega Module Federation:

cd apps/shell
npm install @angular-architects/module-federation --save
nx g @nrwl/angular:setup-mf shell --type=host --port=4200

Esto convierte tu shell en la aplicación anfitriona. Luego, configuraremos tres microfrontends: login, listar-productos, y crear-productos.

## 3. Crear los Microfrontends
Usamos NX para generar las aplicaciones que serán nuestros microfrontends.

Login Microfrontend
nx g @nrwl/angular:app login --mfe --type=remote --port=4201
nx g @angular-architects/module-federation login --type=remote

Listar Productos Microfrontend
nx g @nrwl/angular:app listar-productos --mfe --type=remote --port=4202
nx g @angular-architects/module-federation listar-productos --type=remote

Crear Productos Microfrontend
nx g @nrwl/angular:app crear-productos --mfe --type=remote --port=4203
nx g @angular-architects/module-federation crear-productos --type=remote

## 4. Configurar el Module Federation
Abre webpack.config.js en la carpeta apps/shell y agrega los microfrontends como remotos:

const { ModuleFederationPlugin } = require("webpack").container;

module.exports = {
  output: {
    uniqueName: "shell",
    publicPath: "auto",
  },
  optimization: {
    runtimeChunk: false,
  },
  resolve: {
    alias: {},
  },
  plugins: [
    new ModuleFederationPlugin({
      remotes: {
        login: "login@http://localhost:4201/remoteEntry.js",
        listarProductos: "listarProductos@http://localhost:4202/remoteEntry.js",
        crearProductos: "crearProductos@http://localhost:4203/remoteEntry.js",
      },
      shared: ["@angular/core", "@angular/common", "@angular/router"],
    }),
  ],
};

## 5. Configurar el Routing en el Shell
const routes: Routes = [
  {
    path: 'login',
    loadChildren: () =>
      import('login/Module').then((m) => m.RemoteEntryModule),
  },
  {
    path: 'listar-productos',
    loadChildren: () =>
      import('listarProductos/Module').then((m) => m.RemoteEntryModule),
  },
  {
    path: 'crear-productos',
    loadChildren: () =>
      import('crearProductos/Module').then((m) => m.RemoteEntryModule),
  },
  { path: '', redirectTo: 'login', pathMatch: 'full' },
];

## 6. Crear las Funcionalidades de los Microfrontends

### Login
En la aplicación login, dentro del app.component.ts, crea un simple formulario de autenticación:

export class AppComponent {
  username: string = '';
  password: string = '';

  login() {
    console.log(`Logging in with ${this.username} and ${this.password}`);
  }
}

### Listar Productos
En la aplicación listar-productos, simula la lista de productos:
export class AppComponent {
  productos = [
    { id: 1, nombre: 'Producto 1', precio: 100 },
    { id: 2, nombre: 'Producto 2', precio: 200 },
  ];
}

### Crear Productos
En la aplicación crear-productos, crea un formulario para añadir productos:
export class AppComponent {
  nombre: string = '';
  precio: number = 0;

  crearProducto() {
    console.log(`Creando producto: ${this.nombre} - Precio: ${this.precio}`);
  }
}

## 7. Ejecutar las Aplicaciones
Finalmente, ejecuta cada microfrontend junto con el shell para ver la aplicación en acción:
nx serve shell
nx serve login
nx serve listar-productos
nx serve crear-productos



