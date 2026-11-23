# Trabajo Práctico — Persistencia y Conexión Segura PHP–MySQL

## Descripción del proyecto

**AutoMarket** integra PHP con MySQL mediante el módulo CRUD de administración de vehículos. Desde el panel administrativo pueden agregarse, visualizarse, editarse y eliminarse publicaciones, con persistencia inmediata en la base de datos.

Este trabajo corresponde a la integración segura de la base diseñada en la Clase 6 y permite avanzar hacia el Hito 2.

## Objetivos cumplidos

- Conexión del backend PHP con el motor MySQL/MariaDB.
- Implementación de alta, consulta, modificación y eliminación de vehículos.
- Uso de sentencias preparadas PDO en las operaciones de administración.
- Protección de datos mostrados en HTML mediante `htmlspecialchars()`.
- Integración del CRUD con la interfaz y navegación existentes de AutoMarket.
- Validación de datos antes de guardar la información.

## Entidad manipulada

La entidad principal del CRUD es `vehiculos`. Sus principales campos son:

- `id`
- `usuario_id`
- `categoria_id`
- `titulo`
- `precio`
- `anio`
- `kilometros`
- `transmision`
- `combustible`
- `descripcion`
- `imagen`
- `created_at`
- `updated_at`

La entidad está vinculada con `usuarios` mediante `usuario_id` y con `categorias` mediante `categoria_id`.

## Conexión mediante PDO

Las operaciones de creación, edición y eliminación utilizan una instancia `PDO` configurada para MySQL y utf8mb4.

Ejemplo de configuración segura:

```php
$dsn = 'mysql:host=localhost;dbname=automarket;charset=utf8mb4';
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
];

try {
    $pdo = new PDO($dsn, 'root', '', $options);
} catch (PDOException $e) {
    die('Error crítico de conexión con la base de datos.');
}
```

Las opciones habilitan excepciones, desactivan la simulación de sentencias preparadas y establecen el modo de lectura asociativo.

## Flujo CRUD

### Create — Crear vehículo

La operación se realiza desde `admin/crear.php`.

- Verifica que el usuario tenga rol `admin`.
- Valida el título, precio, año, kilometraje y datos de la imagen.
- Usa una consulta preparada para insertar el vehículo.
- Guarda la foto en `uploads/`.
- Redirige al panel con el estado `created`.

Ejemplo:

```php
$sql = 'INSERT INTO vehiculos
        (usuario_id, titulo, precio, anio, kilometros,
         transmision, combustible, descripcion, imagen)
        VALUES
        (:usuario_id, :titulo, :precio, :anio, :kilometros,
         :transmision, :combustible, :descripcion, :imagen)';

$stmt = $pdo->prepare($sql);
$stmt->execute([
    ':usuario_id' => (int) $_SESSION['usuario_id'],
    ':titulo' => $titulo,
    ':precio' => $precio,
    // resto de parámetros sanitizados y validados
]);
```

### Read — Listar vehículos

El listado se presenta en `admin/index.php`.

- Muestra el resumen del inventario.
- Consulta vehículos recientes y totales relacionados.
- Permite acceder a las operaciones de edición.
- Se integran los datos dentro de la misma estructura visual del sitio.

### Update — Modificar vehículo

La operación se realiza desde `admin/editar.php`.

- Recupera el vehículo según su `id` mediante una consulta preparada.
- Presenta los datos existentes en el formulario.
- Ejecuta una sentencia `UPDATE` con parámetros nombrados `:parametro`.
- Redirige al panel con el estado `updated`.

Ejemplo:

```php
$stmt = $pdo->prepare('SELECT * FROM vehiculos WHERE id = :id');
$stmt->execute([':id' => $id]);

$sql = 'UPDATE vehiculos
        SET titulo = :titulo, precio = :precio, descripcion = :descripcion
        WHERE id = :id';
```

### Delete — Eliminar vehículo

La operación se realiza desde `admin/eliminar.php`.

- Recibe el identificador por GET.
- Convierte el identificador a entero.
- Ejecuta `DELETE` con una sentencia preparada.
- Solicita confirmación antes de la eliminación desde la interfaz.

Ejemplo:

```php
$stmt = $pdo->prepare('DELETE FROM vehiculos WHERE id = :id');
$stmt->execute([':id' => $id]);
```

## Seguridad aplicada

### Consultas preparadas

Las operaciones de alta, lectura individual, modificación y eliminación de vehículos utilizan marcadores nombrados o enlazan correctamente los valores. Los datos del usuario nunca se concatenan dentro de la consulta SQL.

### Protección contra XSS

Los datos que se representan en las vistas deben mostrarse con:

```php
htmlspecialchars($valor, ENT_QUOTES, 'UTF-8');
```

La aplicación aplica esta salida segura a títulos, nombres de usuario, mensajes, descripciones e identificadores mostrados en la interfaz.

### Validación del servidor

- Campos obligatorios comprobados antes de insertar.
- Precio, año y kilometraje validados como valores numéricos.
- Transmisión y combustible limitados a opciones válidas.
- Tipo y tamaño de imagen comprobados.
- Nombre de archivo generado aleatoriamente para evitar conflictos y rutas inseguras.
- Acceso administrativo protegido mediante `auth_guard.php`.

## Ruta de archivos del módulo

```text
admin/
├── index.php       # Listado, resumen y Read
├── crear.php       # Formulario y Create
├── editar.php      # Formulario y Update
└── eliminar.php    # Delete

config/
├── db.php          # Conexión utilizada por algunas vistas
└── env.php         # Variables de entorno

includes/
├── auth_guard.php  # Protección de administración
├── header.php      # Plantilla superior
└── footer.php      # Plantilla inferior

uploads/            # Imágenes de vehículos
css/                # Estilos visuales
```

## Requisitos previos

- XAMPP o Laragon.
- Apache activo.
- MySQL o MariaDB activo.
- Extensión `pdo_mysql` habilitada.
- PHP 8.x.
- Base de datos `automarket` importada desde `includes/schema.sql`.

## Ejecución local

1. Iniciar Apache y MySQL desde XAMPP.
2. Importar `includes/schema.sql` desde phpMyAdmin.
3. Configurar la cuenta administradora desde `registro.php` o phpMyAdmin.
4. Acceder al panel:

```text
http://localhost/Artaza/admin/index.php
```

5. Crear un vehículo desde **Agregar vehículo**.
6. Verificar el registro desde el panel y phpMyAdmin.
7. Abrir **Editar vehículo**, modificar un dato y confirmar el cambio.
8. Eliminar el registro de prueba y comprobar que desaparece del catálogo y de MySQL.

## Evidencias sugeridas

Agregar capturas dentro de `docs/screenshots/`:

- Conexión del panel administrativo con XAMPP activo.
- Formulario de alta de vehículo.
- Registro persistido en phpMyAdmin.
- Listado actualizado inmediatamente después del alta.
- Formulario de edición y resultado actualizado.
- Eliminación realizada con confirmación.
- Mensaje de error controlado ante datos inválidos.
- Código de una consulta preparada mostrando parámetros nombrados.

```text
![Panel CRUD](docs/screenshots/tp7-crud-panel.png)
![Alta persistida](docs/screenshots/tp7-alta.png)
![Consulta preparada](docs/screenshots/tp7-pdo.png)
```

## Prueba completa de persistencia

Para la demostración en vivo:

1. Abrir el catálogo público.
2. Iniciar sesión como administrador.
3. Crear un vehículo con datos visibles.
4. Verificar que se muestra en el panel y en el catálogo.
5. Editar su precio o descripción.
6. Comprobar el cambio sin reiniciar la tabla manualmente.
7. Eliminar el vehículo de prueba.
8. Confirmar que desaparece del sitio y de MySQL.

## Tecnologías utilizadas

- PHP 8.x
- PDO
- MySQL / MariaDB
- Apache
- XAMPP
- HTML5
- CSS3
- JavaScript
- Git y GitHub
