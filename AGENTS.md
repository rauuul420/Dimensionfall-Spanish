# Guía del agente de Dimensionfall

Este es el repositorio de Dimensionfall.
# Guía del agente de Dimensionfall

Este es el repositorio de Dimensionfall.

Este es el único `AGENTES.md` archivo en el repositorio. No busque archivos de instrucciones adicionales del agente.

---

# Descripción general del proyecto

Dimensionfall es un juego de rol de supervivencia modificable creado con Godot 4.

La mayor parte del contenido del juego se basa en datos y se almacena como JSON dentro del `Mods` directorio. El proyecto contiene un extenso editor de contenido dentro del juego para mantener los datos del juego.

A menos que se solicite explícitamente lo contrario, amplíe los sistemas existentes en lugar de introducir otros nuevos.

---

# Estructura del repositorio

## Lógica básica y escenas
- **Scripts/**: Lógica de juego escrita en GDScript. Incluye subdirectorios para componentes y lógica modular (por ejemplo, `Mob/`, `Runtimedata/`).
- **Escenas/**: Escenas de Godot, interfaces de editor y diseños de interfaz de usuario.
- **Pruebas/Unidad**: Las pruebas unitarias de GUT se encuentran aquí.

## Contenido y modificaciones
- **Mods/**: Contenido principal del juego, incluyendo mapas, *baldosas, muebles, objetos, criaturas, misiones, etc.* (Nota: La mayoría de los datos están en formato JSON).
- **Valores predeterminados/**: Configuraciones predeterminadas para el jugador, los bloques y las entidades.

## Activos y medios
- **Recursos/ / Texturas/ / Shaders/**: Recursos visuales, texturas y archivos GDShader personalizados.
- **Sonidos/ / Imágenes/ / Multimedia/**: Audio (efectos de sonido/música), capturas de pantalla y material de vista previa del editor.
- **Documentos/**: Documentación técnica y notas de diseño.

## Herramientas e infraestructura
- **Herramientas/**: Utilidades para desarrolladores y scripts de validación (por ejemplo, `map_validator.py`).
- **complementos/**: Complementos de terceros (por ejemplo, `gloot` para inventario, `gut` (para pruebas). No lo modifique a menos que se le solicite explícitamente.

---

# Estándares de codificación

- Utilice la sintaxis GDScript de Godot 4.x.
- Sigue el estilo de codificación existente y utiliza tabulaciones para la indentación.
- Añade comentarios solo cuando mejoren la facilidad de mantenimiento.
- Prefiero cambios pequeños y específicos.
- Evite introducir dependencias externas innecesarias.
- Reutilizar los sistemas existentes (especialmente en `Scripts/` y `Mods/`) siempre que sea posible.

---

# Normas de funcionamiento

- Trabaja desde tu espacio de trabajo actual.
- Conservar los cambios de usuario existentes; no sobrescribir archivos no relacionados.
- Inspeccione la implementación actual antes de proponer reemplazos o refactorizaciones.
- Explique los hallazgos inesperados o los conflictos arquitectónicos antes de realizar cambios importantes.
- Al cambiar los formatos de datos (JSON/Recurso) o la serialización, mantenga la compatibilidad con versiones anteriores.

---

# Validación

Validar los cambios cuando sea apropiado para la tarea. 
**Nota:** Al agregar nuevos archivos, asegúrese de que el proyecto los reconozca correctamente mediante `project.godot`.

Las herramientas de validación disponibles incluyen:
- Pruebas unitarias de GUT (`Tests/Unit`)
- Importación/validación sin interfaz gráfica de Godot
- Scripts de validación de proyectos en `Tools/`

Si se realiza la validación, informa lo siguiente:
- Errores y advertencias
- Pruebas fallidas
- Observaciones destacadas

No ocultes tus fracasos.

---

# Proceso de desarrollo

1. **Comprender**: Investiga la implementación existente y la documentación pertinente.
2. **Implementar**: Realizar el cambio más pequeño y razonable para lograr el objetivo.
3. **Validar**: Ejecutar las pruebas o los scripts de validación adecuados.
4. **Resumir**: Descripción clara de los cambios realizados.
5. **Informar**: Detalla cualquier problema pendiente o trabajo de seguimiento recomendado.
