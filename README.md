## 🎨 Integración del Graficado del Autómata (Asistido por Gemini IA)

Para la representación visual del Autómata Finito Determinista ($M = (Q, \Sigma, \delta, q_0, F)$), se integró el módulo de graficado con el apoyo de la inteligencia artificial **Gemini**, manteniendo toda la lógica matemática y algorítmica del simulador en Python puro y delegando únicamente el renderizado del diagrama de estados a **Graphviz**.

### 📦 Librerías y Herramientas Utilizadas

* **`graphviz` (Librería externa para Python):** Construye la estructura del grafo dirigido (`Digraph`), configurando orientación horizontal (`rankdir="LR"`), estados normales (`circle`), estados de aceptación (`doublecircle`), flecha de inicio mediante un nodo invisible (`shape="none"`) y agrupación de transiciones múltiples en una misma arista.
* **`subprocess` (Librería nativa de Python):** Ejecuta directamente el binario compilador `dot.exe` de Graphviz en Windows mediante ruta absoluta (`C:\Program Files\Graphviz\bin\dot.exe`), evitando fallos por variables de entorno (`PATH`) no reconocidas.
* **`os` (Librería nativa de Python):** Elimina el archivo fuente temporal (`automata.gv`) tras generar la imagen y despliega automáticamente el archivo `automata_grafica.png` con el visor predeterminado de Windows (`os.startfile`).

### ⚙️ Instalación de Dependencias (Windows)

Para ejecutar correctamente el módulo gráfico desde cero, abre una terminal (`CMD` o `PowerShell`) y ejecuta ambos comandos:

1. **Instalar el paquete `graphviz` en Python:**
   ```cmd
   pip install graphviz
   ```

2. **Instalar el motor de Graphviz en el sistema operativo (incluye `dot.exe`):**
   ```cmd
   winget install Graphviz.Graphviz
   ```

*(Nota: Las librerías `os` y `subprocess` pertenecen a la biblioteca estándar de Python y no requieren instalación adicional).*

### 💻 Bloque de Código Implementado

```python
import os
import subprocess
from graphviz import Digraph


def graficar_automata(automata):
    Q = automata["Q"]
    delta = automata["delta"]
    q0 = automata["q0"]
    F = automata["F"]

    # Crear grafo dirigido de izquierda a derecha (LR)
    dot = Digraph(name="Automata", format="png")
    dot.attr(rankdir="LR", dpi="150")
    dot.attr(
        "node",
        fontname="Arial",
        fontsize="12",
        style="filled",
        fillcolor="#EBF1F6",
        color="#1F3864",
        penwidth="1.5",
    )
    dot.attr(
        "edge",
        fontname="Arial",
        fontsize="11",
        color="#1F3864",
        penwidth="1.5",
    )

    # 1. Nodo invisible para la flecha inicial sin origen
    dot.node("inicio", label="", shape="none", width="0", height="0")

    # 2. Dibujar estados (doble círculo para finales, círculo simple para el resto)
    for estado in Q:
        forma = "doublecircle" if estado in F else "circle"
        dot.node(estado, label=estado, shape=forma)

    # Conectar flecha inicial hacia q0
    dot.edge("inicio", q0)

    # 3. Agrupar transiciones que comparten (origen, destino)
    aristas_agrupadas = {}
    for (origen, simbolo), destino in delta.items():
        aristas_agrupadas.setdefault((origen, destino), []).append(simbolo)

    # 4. Dibujar flechas (bucles y transiciones bidireccionales)
    for (origen, destino), simbolos in aristas_agrupadas.items():
        etiqueta = ", ".join(simbolos)
        dot.edge(origen, destino, label=f" {etiqueta} ")

    print("\nMostrando la gráfica del autómata...")
    ruta_dot = r"C:\Program Files\Graphviz\bin\dot.exe"

    try:
        # Guardar código fuente del grafo y renderizar llamando directo a dot.exe
        dot.save("automata.gv")
        subprocess.run(
            [ruta_dot, "-Tpng", "automata.gv", "-o", "automata_grafica.png"],
            check=True,
        )
        # Eliminar el archivo temporal .gv
        if os.path.exists("automata.gv"):
            os.remove("automata.gv")
        # Abrir la imagen generada en Windows
        os.startfile("automata_grafica.png")
    except Exception as e:
        print(f"Error al generar la gráfica: {e}")
```
