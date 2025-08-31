import tkinter as tk
from tkinter import ttk
from abc import ABC, abstractmethod

# --- Clases del sistema de inventario (sin cambios) ---
class Insumo(ABC):
    """
    Clase base abstracta para representar un insumo en el inventario.
    Define la estructura básica de cualquier insumo.
    """
    def __init__(self, sku: str, nombre: str, cantidad: int, min_stock: int):
        self.__sku = sku
        self.__nombre = nombre
        self.__cantidad = cantidad
        self.__min_stock = min_stock

    def get_sku(self):
        return self.__sku

    def get_nombre(self):
        return self.__nombre

    def get_cantidad(self):
        return self.__cantidad

    def get_min_stock(self):
        return self.__min_stock

    def set_cantidad(self, nueva_cantidad):
        if nueva_cantidad >= 0:
            self.__cantidad = nueva_cantidad
        else:
            raise ValueError("La cantidad no puede ser negativa.")

    def set_min_stock(self, nuevo_min_stock):
        if nuevo_min_stock >= 0:
            self.__min_stock = nuevo_min_stock
        else:
            raise ValueError("El stock mínimo no puede ser negativo.")

    @abstractmethod
    def mostrar_detalles(self):
        pass

class Material(Insumo):
    """
    Clase que hereda de Insumo y representa un material específico
    con una categoría.
    """
    def __init__(self, sku: str, nombre: str, cantidad: int, min_stock: int, categoria: str):
        super().__init__(sku, nombre, cantidad, min_stock)
        self.__categoria = categoria

    def get_categoria(self):
        return self.__categoria

    def mostrar_detalles(self):
        return (f"SKU: {self.get_sku()}\n"
                f"Nombre: {self.get_nombre()}\n"
                f"Cantidad: {self.get_cantidad()}\n"
                f"Stock Mínimo: {self.get_min_stock()}\n"
                f"Categoría: {self.get_categoria()}")

class SistemaInventario:
    """
    Clase que gestiona todo el inventario, incluyendo el registro,
    los movimientos y los reportes.
    """
    def __init__(self):
        self.__inventario = {}

    def registrar_insumo(self, insumo: Insumo):
        if insumo.get_sku() in self.__inventario:
            raise ValueError(f"El SKU '{insumo.get_sku()}' ya existe en el inventario.")
        self.__inventario[insumo.get_sku()] = insumo

    def registrar_movimiento(self, sku: str, cantidad: int):
        if sku not in self.__inventario:
            raise KeyError(f"El SKU '{sku}' no se encontró en el inventario.")
        
        insumo = self.__inventario[sku]
        nueva_cantidad = insumo.get_cantidad() + cantidad
        insumo.set_cantidad(nueva_cantidad)

    def obtener_inventario(self):
        return self.__inventario

# --- Clase principal de la interfaz gráfica ---
class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Sistema de Inventario Escolar")
        self.geometry("500x400")
        self.resizable(False, False)

        # Inicializar el sistema de inventario con datos de prueba
        self.sistema = SistemaInventario()
        self.lapiz = Material("LAPIZ-001", "Lápiz Grafito", 100, 20, "Material de Oficina")
        self.regla = Material("REGLA-002", "Regla de 30cm", 50, 10, "Material de Oficina")
        self.cuaderno = Material("CUADER-003", "Cuaderno Espiral", 75, 15, "Material de Oficina")
        
        
        try:
            self.sistema.registrar_insumo(self.lapiz)
            self.sistema.registrar_insumo(self.regla)
            self.sistema.registrar_insumo(self.cuaderno)
        except Exception as e:
            print(f"Error al inicializar el inventario: {e}")

        self.crear_widgets()
        self.actualizar_combobox()
        self.actualizar_vista_detalles()

    def crear_widgets(self):
        """Crea y posiciona todos los elementos de la GUI."""
        main_frame = ttk.Frame(self, padding="15")
        main_frame.pack(fill="both", expand=True)

        # Título
        title_label = ttk.Label(main_frame, text="Gestión de Insumos", font=("Arial", 16, "bold"))
        title_label.pack(pady=(0, 20))

        # Marco para la entrada de datos
        input_frame = ttk.LabelFrame(main_frame, text=" Registrar Movimiento ", padding="10")
        input_frame.pack(fill="x", pady=10)

        # Fila 1: Selección de insumo
        ttk.Label(input_frame, text="Seleccione Insumo:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.sku_var = tk.StringVar()
        self.insumo_combo = ttk.Combobox(input_frame, textvariable=self.sku_var, state="readonly")
        self.insumo_combo.grid(row=0, column=1, padx=5, pady=5, sticky="ew")

        # Fila 2: Cantidad
        ttk.Label(input_frame, text="Cantidad (+/-):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.cantidad_entry = ttk.Entry(input_frame)
        self.cantidad_entry.grid(row=1, column=1, padx=5, pady=5, sticky="ew")

        # Fila 3: Botón de actualización
        self.btn_actualizar = ttk.Button(input_frame, text="Actualizar Stock", command=self.actualizar_stock)
        self.btn_actualizar.grid(row=2, column=0, columnspan=2, pady=10)

        input_frame.columnconfigure(1, weight=1)

        # Marco para los detalles
        details_frame = ttk.LabelFrame(main_frame, text=" Detalles ", padding="10")
        details_frame.pack(fill="both", expand=True, pady=10)

        self.detalles_text = tk.Text(details_frame, height=10, state="disabled")
        self.detalles_text.pack(fill="both", expand=True)

        # Etiqueta de mensaje
        self.mensaje_label = ttk.Label(main_frame, text="", font=("Arial", 10, "italic"))
        self.mensaje_label.pack(pady=5)

    def actualizar_combobox(self):
        """Llena el combobox con los nombres de los insumos."""
        inventario = self.sistema.obtener_inventario()
        nombres_insumos = [f"{insumo.get_sku()} - {insumo.get_nombre()}" for insumo in inventario.values()]
        self.insumo_combo['values'] = nombres_insumos
        if nombres_insumos:
            self.insumo_combo.current(0)
            self.actualizar_vista_detalles()
            
    def actualizar_vista_detalles(self, event=None):
        """Actualiza el área de texto con los detalles del insumo seleccionado."""
        sku_seleccionado = self.insumo_combo.get().split(" ")[0]
        insumo = self.sistema.obtener_inventario().get(sku_seleccionado)
        
        self.detalles_text.config(state="normal")
        self.detalles_text.delete("1.0", tk.END)
        if insumo:
            self.detalles_text.insert(tk.END, insumo.mostrar_detalles())
        self.detalles_text.config(state="disabled")

    def actualizar_stock(self):
        """Maneja el evento del botón 'Actualizar Stock'."""
        sku_seleccionado = self.sku_var.get().split(" ")[0]
        try:
            cantidad = int(self.cantidad_entry.get())
            if not sku_seleccionado:
                self.mostrar_mensaje("Error: Debe seleccionar un insumo.", "red")
                return

            self.sistema.registrar_movimiento(sku_seleccionado, cantidad)
            self.mostrar_mensaje(f"Movimiento registrado. Nuevo stock de '{self.sistema.obtener_inventario()[sku_seleccionado].get_nombre()}': {self.sistema.obtener_inventario()[sku_seleccionado].get_cantidad()}", "green")
            self.actualizar_vista_detalles()
            self.cantidad_entry.delete(0, tk.END)

            # Revisa si hay alertas de stock bajo
            insumo_actual = self.sistema.obtener_inventario()[sku_seleccionado]
            if insumo_actual.get_cantidad() < insumo_actual.get_min_stock():
                self.mostrar_mensaje(f"ALERTA: El stock de {insumo_actual.get_nombre()} está bajo el mínimo.", "orange")
            else:
                self.mostrar_mensaje(f"Movimiento registrado con éxito.", "green")

        except ValueError as e:
            self.mostrar_mensaje(f"Error: {e}", "red")
        except KeyError as e:
            self.mostrar_mensaje(f"Error: {e}", "red")
        except Exception as e:
            self.mostrar_mensaje(f"Ocurrió un error inesperado: {e}", "red")

    def mostrar_mensaje(self, texto, color):
        """Actualiza el mensaje en la interfaz gráfica."""
        self.mensaje_label.config(text=texto, foreground=color)
        self.after(5000, lambda: self.mensaje_label.config(text="")) # Borra el mensaje después de 5 segundos

if __name__ == "__main__":
    app = App()
    app.mainloop()

