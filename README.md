# Analizador de Finanzas Personales para Estudiantes 📊🎒

Una aplicación de consola ligera, robusta y fácil de usar diseñada para ayudar a los estudiantes universitarios a registrar sus ingresos, gastos y visualizar un reporte resumido de su saldo actual.

Este proyecto fue desarrollado de manera autónoma como parte de mi ruta de aprendizaje en ingeniería de sistemas, aplicando bases de programación estructurada y persistencia de datos relacionales.

## 🚀 Características
* **Registro de Ingresos y Gastos:** Categorías personalizadas y adaptadas al entorno estudiantil (becas, comida, transporte, estudios, ocio, etc.).
* **Base de Datos Relacional:** Persistencia de datos utilizando **SQLite** de forma local.
* **Seguridad y Robustez:** Control de excepciones para evitar cierres inesperados y consultas parametrizadas para prevenir inyecciones SQL.
* **Interfaz Limpia en Consola:** Menú visual dinámico con reportes ordenados y formateados en S/ (Soles).

## 🛠️ Tecnologías Utilizadas
* **Lenguaje:** Python 3.x
* **Base de Datos:** SQLite (mediante la librería nativa `sqlite3`)
* **Librerías estándar:** `datetime`, `pathlib`

## 💻 Cómo Ejecutar el Proyecto

1. Asegúrate de tener instalado [Python 3](https://www.python.org/).
2. Descarga el archivo del repositorio.
3. Abre tu terminal o consola de comandos en la carpeta del archivo y ejecuta:
    ```bash
   python finanzas.py
