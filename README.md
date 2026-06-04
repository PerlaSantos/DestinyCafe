# DestinyCafe - Sistema de Gestión para Cafetería Artesanal


DestinyCafe es un sistema web de gestión integral diseñado para una cafetería artesanal ubicada en la colonia Reforma, Ciudad de México, que atiende principalmente a turistas y locales. La plataforma permite administrar de manera centralizada los procesos clave del negocio, incluyendo el control de clientes, productos, inventario de insumos, proveedores y ventas.

El proyecto surge de una necesidad real. A través de una entrevista con un amigo del equipo, quien labora en el establecimiento, se identificaron áreas de oportunidad en la gestión diaria, como el seguimiento a la experiencia del cliente, el control de stock con alertas de reabastecimiento, el registro de compras a proveedores y la generación de reportes de ventas. DestinyCafe resuelve estas problemáticas mediante un modelo de base de datos robusto y una interfaz web funcional.

## Especificaciones del Cliente

A continuación, se presentan los requerimientos funcionales identificados durante la entrevista con el cliente, los cuales guiaron todo el proceso de diseño e implementación.

*   **Gestión de clientes y su experiencia**: El sistema debe almacenar información sobre cómo se enteró el cliente del negocio (red social, sugerencia, vista del local), el motivo de su visita (desayuno, comida, solo bebida), la saturación del local durante su estancia y sus comentarios sobre la atención y los precios.
*   **Control de inventario y proveedores**: Debe permitir gestionar el stock de insumos por área (Barra, Cocina, Panadería), generar alertas automáticas cuando el stock caiga por debajo de un mínimo (con 3 a 4 días de anticipación), y mantener un registro centralizado de proveedores, facturas, compras y reseñas.
*   **Registro de ventas y reportes**: El sistema debe registrar cada venta con detalle de productos, precios, método de pago, propinas y descuentos. Así mismo, debe facilitar la generación de reportes semanales que muestren los productos más y menos vendidos.

## Modelado de Datos: Del Diagrama E-R al Modelo Relacional

Para sentar las bases del sistema, seguimos una metodología estructurada de modelado de bases de datos.

### Proceso de Diseño del Diagrama Entidad-Relación

El proceso inició con la identificación de las entidades principales a partir de la entrevista (Cliente, Producto, Venta, Empleado, Proveedor, Insumo, etc.). Posteriormente, se definieron sus atributos y las relaciones entre ellas, estableciendo las cardinalidades que reflejaran fielmente las reglas del negocio. Para enriquecer el modelo, aplicamos conceptos del Modelo Entidad-Relación Extendido (EER), como:

*   **Cardinalidades mínimas y máximas**: Para expresar restricciones como que un empleado debe tener al menos un horario asignado (min=1) o que una venta debe tener al menos un producto (min=1).
*   **Entidades débiles**: Se modelaron entidades como `DETALLE_VENTA`, cuya existencia depende completamente de una `VENTA`.
*   **Jerarquías de herencia**: Se exploró una especialización para `EMPLEADO` (Mesero, Bartender, Chef), aunque finalmente no se implementó en la base de datos física por decisión de alcance, priorizando la gestión de clientes, proveedores e inventario.

La siguiente imagen muestra el diagrama EER completo desarrollado en la notación de Peter Chen.

| Diagrama Entidad-Relación Extendido (EER) |
|:---:|
| <img src="https://github.com/PerlaSantos/DestinyCafe/blob/2a474471d76d9d47848d8ed9a2ecb3c6ec79de02/Documentaci%C3%B3n/Diagrama%20Entidad%20Relaci%C3%B3n.jpg?raw=true" alt="Diagrama Entidad-Relación Extendido de DestinyCafe" width="800"/> |

### Transformación al Modelo Relacional

Una vez validado el diagrama EER, procedimos a su transformación al modelo relacional, obteniendo un esquema de 15 tablas que normalizan la información y evitan redundancias. Se definieron claves primarias y foráneas, junto con restricciones de integridad (CHECK, NOT NULL, UNIQUE, DEFAULT) para garantizar la calidad de los datos. La siguiente imagen presenta el esquema relacional resultante.

| Modelo Relacional |
|:---:|
| <img src="https://github.com/PerlaSantos/DestinyCafe/blob/2a474471d76d9d47848d8ed9a2ecb3c6ec79de02/Documentaci%C3%B3n/Modelo%20Relacional.jpg?raw=true" alt="Modelo Relacional de DestinyCafe" width="800"/> |

### Decisión sobre la Gestión de Usuarios

Siguiendo las especificaciones del cliente, el sistema **no implementa un módulo de autenticación de usuarios** ni perfiles diferenciados. El requerimiento principal se centra en la gestión operativa del negocio (clientes, inventarios, ventas). Por lo tanto, toda la interacción con la base de datos se realiza de manera directa a través de la interfaz web, sin necesidad de registro o inicio de sesión por parte del personal. Esta decisión simplifica el desarrollo y se alinea con el alcance acordado con el cliente.

## Desarrollo de la Aplicación Web

### Diseño de la Interfaz

La interfaz web se diseñó con un enfoque en la usabilidad y la presentación clara de la información. Se priorizó una navegación sencilla que permitiera acceder rápidamente a las secciones principales: Control de Inventario, Registro de Ventas y Administración de Proveedores. El diseño es responsivo, adaptándose a dispositivos móviles y de escritorio.

### Tecnologías Utilizadas

*   **Frontend**: Se utilizó HTML5 para la estructura, CSS3 para los estilos y JavaScript vanilla para la lógica del lado del cliente y las interacciones con la API.
*   **Base de Datos en la Nube**: Se eligió **Supabase** como backend como servicio (BaaS) por las siguientes razones:
    *   Proporciona una base de datos PostgreSQL gestionada, lo que elimina la necesidad de administrar un servidor.
    *   Ofrece una API RESTful automática sobre las tablas, lo que simplifica enormemente las operaciones CRUD desde JavaScript.
    *   Incluye capacidades de almacenamiento y seguridad (Row Level Security) que podrían ser útiles en el futuro.
    *   Su capa gratuita es generosa y adecuada para proyectos académicos y prototipos funcionales.

### Conexión a la Base de Datos y su Implementación

La conexión entre la página web y Supabase se implementó de la siguiente manera:

1.  **Configuración del Cliente de Supabase**: En el archivo JavaScript principal, se inicializó el cliente de Supabase utilizando la URL del proyecto y la `anon key` pública (clave anónima) proporcionada por la plataforma.
2.  **Operaciones CRUD**: Para cada operación (consultar, insertar, actualizar o eliminar datos), se utilizó el cliente de Supabase. Por ejemplo, para obtener la lista de insumos se empleó `supabase.from('insumo').select('*')`, y para registrar una nueva venta se realizó una inserción en las tablas `venta` y `detalle_venta` dentro de una transacción implícita.
3.  **Manejo de la API REST**: La mayoría de las interacciones se realizaron mediante `fetch` a los endpoints de la API REST de Supabase, con las cabeceras adecuadas que incluían la clave API. Esto permitió un control más fino sobre las peticiones y las respuestas.

La aplicación web se despliega como un sitio estático en GitHub Pages, mientras que la base de datos permanece alojada y gestionada en la nube de Supabase, ofreciendo una solución completa y de bajo costo.

## Posibles Mejoras y Trabajo a Futuro

Si bien el sistema cumple con los requerimientos iniciales, se han identificado varias áreas de oportunidad para futuras iteraciones:

*   Implementar un módulo de autenticación y roles de usuario (administrador, mesero, gerente) con políticas de seguridad a nivel de fila (RLS) en Supabase.
*   Desarrollar un panel de reportes avanzados con gráficas interactivas que muestren tendencias de ventas, rotación de productos y análisis de la experiencia del cliente.
*   Añadir funcionalidad de escaneo de códigos de barras para agilizar el registro de productos en las ventas y el inventario.
*   Mejorar la interfaz de usuario con un framework moderno como React o Vue.js para ofrecer una experiencia más dinámica y reactiva.
*   Incorporar la jerarquía de herencia de empleados (Mesero, Bartender, Chef) en la base de datos física para una gestión más granular del personal.

## Integrantes del Equipo

*   Alarcon Herrera Julio Alexis
*   Cedillo Baeza Martha Clara
*   Santos Martínez Perla

## Enlaces del Proyecto

*   **Repositorio General de GitHub (Colaboración)**: https://github.com/gabrielhuav/DB-Coursework-2026-2
*   **Sitio Web (Demo en Vivo)**: https://perlasantos.github.io/DestinyCafe/


