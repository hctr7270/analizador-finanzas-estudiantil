"""
Analizador de Finanzas Personales para Estudiantes
--------------------------------------------------
Aplicación de consola simple para registrar ingresos/gastos y ver reportes.
Usa SQLite como almacenamiento local.
"""

import sqlite3
from datetime import datetime
from pathlib import Path

# ──────────────────────────────────────────────────────────────
# CONSTANTES Y CONFIGURACIÓN
# ──────────────────────────────────────────────────────────────
DB_NAME = "finanzas.db"

# Categorías separadas por tipo
CATEGORIAS_INGRESO = ["Beca", "Trabajo", "Familia", "Ventas", "Inversiones", "Otros"]
CATEGORIAS_GASTO   = ["Comida", "Transporte", "Estudios", "Ocio", "Salud", "Suscripciones", "Otros"]

MENU = """
╔══════════════════════════════════════════╗
║   ANALIZADOR DE FINANZAS ESTUDIANTIL     ║
╠══════════════════════════════════════════╣
║ 1. Registrar Ingreso                     ║
║ 2. Registrar Gasto                       ║
║ 3. Ver todas las transacciones           ║
║ 4. Ver reporte resumido                  ║
║ 5. Salir                                 ║
║                                HectorZC  ║
╚══════════════════════════════════════════╝
"""


# ──────────────────────────────────────────────────────────────
# FUNCIONES DE BASE DE DATOS
# ──────────────────────────────────────────────────────────────
def crear_conexion() -> sqlite3.Connection:
    """Crea (si no existe) y devuelve la conexión a la BD."""
    conn = sqlite3.connect(DB_NAME)
    conn.execute("PRAGMA foreign_keys = ON;")
    return conn


def inicializar_bd(conn: sqlite3.Connection) -> None:
    """Crea la tabla 'transacciones' si no existe."""
    sql = """
    CREATE TABLE IF NOT EXISTS transacciones (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        fecha TEXT NOT NULL,
        tipo TEXT NOT NULL CHECK(tipo IN ('Ingreso', 'Gasto')),
        categoria TEXT NOT NULL,
        monto REAL NOT NULL CHECK(monto > 0),
        descripcion TEXT
    );
    """
    conn.execute(sql)
    conn.commit()


# ──────────────────────────────────────────────────────────────
# FUNCIONES DE LÓGICA DE NEGOCIO
# ──────────────────────────────────────────────────────────────
def pedir_fecha() -> str:
    """Solicita fecha al usuario en formato DD-MM-YYYY (por defecto hoy)."""
    while True:
        entrada = input("Fecha (DD-MM-YYYY) [hoy]: ").strip()
        if not entrada:
            return datetime.today().strftime("%Y-%m-%d")  # Guardamos en ISO internamente
        try:
            fecha_dt = datetime.strptime(entrada, "%d-%m-%Y")
            return fecha_dt.strftime("%Y-%m-%d")
        except ValueError:
            print("⚠️  Formato inválido. Usa DD-MM-YYYY (ej. 25-12-2024).")


def pedir_categoria(tipo: str) -> str:
    """Muestra categorías según el tipo (Ingreso/Gasto) y pide elegir una."""
    categorias = CATEGORIAS_INGRESO if tipo == "Ingreso" else CATEGORIAS_GASTO
    titulo = "CATEGORÍAS DE INGRESO" if tipo == "Ingreso" else "CATEGORÍAS DE GASTO"
    
    print(f"\n{titulo}:")
    for i, cat in enumerate(categorias, 1):
        print(f"  {i}. {cat}")
    while True:
        try:
            idx = int(input("Elige número de categoría: "))
            if 1 <= idx <= len(categorias):
                return categorias[idx - 1]
        except ValueError:
            pass
        print("⚠️  Opción no válida.")


def pedir_monto() -> float:
    """Pide monto numérico positivo."""
    while True:
        try:
            monto = float(input("Monto: ").replace(",", "."))
            if monto <= 0:
                raise ValueError
            return monto
        except ValueError:
            print("⚠️  Ingresa un número positivo (ej. 15.50).")


def registrar_transaccion(conn: sqlite3.Connection, tipo: str) -> None:
    """Flujo completo para registrar ingreso o gasto."""
    print(f"\n--- Registrar {tipo} ---")
    fecha = pedir_fecha()
    categoria = pedir_categoria(tipo)
    monto = pedir_monto()
    descripcion = input("Descripción (opcional): ").strip()

    sql = """INSERT INTO transacciones (fecha, tipo, categoria, monto, descripcion)
             VALUES (?, ?, ?, ?, ?);"""
    try:
        conn.execute(sql, (fecha, tipo, categoria, monto, descripcion))
        conn.commit()
        print(f"✅ {tipo} de S/{monto:.2f} en '{categoria}' registrado correctamente.")
    except sqlite3.Error as e:
        print(f"❌ Error al guardar: {e}")


def mostrar_transacciones(conn: sqlite3.Connection) -> None:
    """Lista todas las transacciones en tabla simple (mostrando DD-MM-YYYY)."""
    cursor = conn.execute("""
        SELECT id, fecha, tipo, categoria, monto, descripcion
        FROM transacciones
        ORDER BY fecha DESC, id DESC;
    """)
    filas = cursor.fetchall()

    if not filas:
        print("\n📭 No hay transacciones registradas.")
        return

    print(f"\n{'ID':<4} {'Fecha':<12} {'Tipo':<8} {'Categoría':<14} {'Monto':>10}  Descripción")
    print("─" * 80)
    for fid, fecha, tipo, cat, monto, desc in filas:
        try:
            fecha_show = datetime.strptime(fecha, "%Y-%m-%d").strftime("%d-%m-%Y")
        except ValueError:
            fecha_show = fecha
        desc_corta = (desc[:30] + "..") if desc and len(desc) > 30 else (desc or "")
        signo = "+" if tipo == "Ingreso" else "-"
        print(f"{fid:<4} {fecha_show:<12} {tipo:<8} {cat:<14} {signo}{monto:>9.2f}  {desc_corta}")


def mostrar_reporte(conn: sqlite3.Connection) -> None:
    """Muestra totales y saldo."""
    totales = conn.execute("""
        SELECT tipo, SUM(monto) as total
        FROM transacciones
        GROUP BY tipo;
    """).fetchall()

    ingresos = next((t for tp, t in totales if tp == "Ingreso"), 0)
    gastos   = next((t for tp, t in totales if tp == "Gasto"), 0)
    saldo = ingresos - gastos

    print("\n╔═════════════════════════════════╗")
    print("║      REPORTE RESUMIDO           ║")
    print("╠═════════════════════════════════╣")
    print(f"║ Ingresos totales: S/{ingresos:>10.2f}  ║")
    print(f"║ Gastos totales:   S/{gastos:>10.2f}  ║")
    print(f"╠═════════════════════════════════╣")
    print(f"║ SALDO ACTUAL:     S/{saldo:>10.2f}  ║")
    print("╚═════════════════════════════════╝")


# ──────────────────────────────────────────────────────────────
# BUCLE PRINCIPAL
# ──────────────────────────────────────────────────────────────
def main() -> None:
    conn = crear_conexion()
    inicializar_bd(conn)

    while True:
        print(MENU)
        opcion = input("Selecciona una opción (1-5): ").strip()

        if opcion == "1":
            registrar_transaccion(conn, "Ingreso")
        elif opcion == "2":
            registrar_transaccion(conn, "Gasto")
        elif opcion == "3":
            mostrar_transacciones(conn)
        elif opcion == "4":
            mostrar_reporte(conn)
        elif opcion == "5":
            print("\n👋 ¡Hasta pronto! Tu información queda guardada en 'finanzas.db'.")
            break
        else:
            print("⚠️  Opción no válida. Intenta de nuevo.")

        input("\nPresiona Enter para continuar...")

    conn.close()


# ──────────────────────────────────────────────────────────────
# PUNTO DE ENTRADA
# ──────────────────────────────────────────────────────────────
if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\n\n🛑 Programa interrumpido por el usuario.")
    except Exception as e:
        print(f"\n💥 Error inesperado: {e}")
