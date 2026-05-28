# Unidad 2: Graficación 2D

Este módulo cubre los fundamentos teóricos y prácticos de la computación gráfica en dos dimensiones, abarcando desde las transformaciones geométricas lineales y afines mediante matrices, hasta el modelado de curvas complejas, geometría fractal y el renderizado de texto.

---

## 2.1. Transformación Bidimensional

Las transformaciones bidimensionales son operaciones matemáticas que alteran la posición, tamaño, orientación o forma de un objeto geométrico dentro de un plano bidimensional ($X, Y$). 

### 2.1.1. Traslación
La traslación desplaza un punto a una nueva posición en el plano añadiendo distancias de desplazamiento ($t_x, t_y$) a sus coordenadas originales ($x, y$).

* **Ecuaciones matemáticas:**
    $$x' = x + t_x$$
    $$y' = y + t_y$$

### 2.1.2. Escalamiento
El escalamiento altera el tamaño de un objeto multiplicando sus coordenadas por factores de escala ($s_x, s_y$). Si el factor es mayor a 1 el objeto se agranda; si está entre 0 y 1, se encoge. El escalamiento se realiza respecto al origen $(0,0)$ a menos que se aplique una transformación compuesta.

* **Ecuaciones matemáticas:**
    $$x' = x \cdot s_x$$
    $$y' = y \cdot s_y$$

### 2.1.3. Rotación
La rotación gira los puntos de un objeto alrededor del origen del sistema de coordenadas en un ángulo específico $\theta$. Por convención computacional, los ángulos positivos giran en sentido contrario a las manecillas del reloj (antihorario).

* **Ecuaciones matemáticas:**
    $$x' = x \cdot \cos(\theta) - y \cdot \sin(\theta)$$
    $$y' = x \cdot \sin(\theta) + y \cdot \cos(\theta)$$

### 2.1.4. Sesgado (Shearing / Cizallamiento)
El sesgado deforma la forma de un objeto a lo largo de un eje, haciendo que las líneas paralelas al eje permanezcan en su lugar mientras que las demás se desplazan proporcionalmente a su distancia de dicho eje.

* **Sesgado en X (factor $sh_x$):**
    $$x' = x + sh_x \cdot y$$
    $$y' = y$$
* **Sesgado en Y (factor $sh_y$):**
    $$x' = x$$
    $$y' = y + sh_y \cdot x$$

---

## 2.2. Representación Matricial de las Transformaciones Bidimensionales

Para encadenar múltiples operaciones (por ejemplo, trasladar, luego rotar y finalmente escalar) sin tener que calcular paso a paso intermedio, se utilizan las **Coordenadas Homogéneas**. Esto permite representar todas las transformaciones (incluida la traslación) como multiplicaciones de matrices de $3 \times 3$.

Un punto $(x, y)$ se expresa en forma homogénea como $(x, y, 1)$.

### Matrices de Transformación Estándar:

* **Traslación:**
    $$\begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix} = \begin{bmatrix} 1 & 0 & t_x \\ 0 & 1 & t_y \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ 1 \end{bmatrix}$$

* **Escalamiento:**
    $$\begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix} = \begin{bmatrix} s_x & 0 & 0 \\ 0 & s_y & 0 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ 1 \end{bmatrix}$$

* **Rotación:**
    $$\begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix} = \begin{bmatrix} \cos(\theta) & -\sin(\theta) & 0 \\ \sin(\theta) & \cos(\theta) & 0 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ 1 \end{bmatrix}$$

* **Sesgado (Generalizado):**
    $$\begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix} = \begin{bmatrix} 1 & sh_x & 0 \\ sh_y & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ 1 \end{bmatrix}$$

> **Nota de Transformación Compuesta:** El orden de la multiplicación importa. Para aplicar la transformación $A$, luego $B$ y luego $C$ a un punto $P$, la operación se calcula como $P' = C \cdot B \cdot A \cdot P$.

---

### ⌨️ Ejercicio Práctico: Control de Transformaciones con Teclas de Dirección

A continuación se presenta una implementación en Python utilizando `pygame` que demuestra el uso interactivo de traslación y escalamiento sobre un polígono 2D.

```python
# Archivo: transformaciones/control_teclas.py
import pygame
import sys
import numpy as np

# Inicializar Pygame
pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Control de Transformaciones 2D")
clock = pygame.time.Clock()

# Definición del objeto original (Triángulo centrado en el origen local)
# Formato de columnas: [x, y, 1]^T
polygon = np.array([
    [0, -50, 1],
    [-40, 40, 1],
    [40, 40, 1]
]).T

# Parámetros iniciales de transformación global
posX, posY = WIDTH // 2, HEIGHT // 2
scale = 1.0

while True:
    screen.fill((30, 30, 30))
    
    # Manejo de eventos / Teclado
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
            
    keys = pygame.key.get_pressed()
    # Traslación con Flechas de Dirección
    if keys[pygame.K_LEFT]:  posX -= 5
    if keys[pygame.K_RIGHT]: posX += 5
    if keys[pygame.K_UP]:    posY -= 5
    if keys[pygame.K_DOWN]:  posY += 5
    # Escalamiento con teclas W (Aumentar) y S (Disminuir)
    if keys[pygame.K_w]:     scale += 0.05
    if keys[pygame.K_s]:     scale = max(0.1, scale - 0.05)

    # Construcción de matrices homogéneas
    T = np.array([
        [1, 0, posX],
        [0, 1, posY],
        [0, 0, 1]
    ])
    
    S = np.array([
        [scale, 0,     0],
        [0,     scale, 0],
        [0,     0,     1]
    ])
    
    # Matriz compuesta: Primero escalar, luego trasladar al centro dinámico
    M = np.dot(T, S)
    
    # Aplicar transformación a todos los vértices
    transformed_poly = np.dot(M, polygon)
    
    # Extraer puntos para dibujar en Pygame (x, y)
    points = [(transformed_poly[0, i], transformed_poly[1, i]) for i in range(polygon.shape[1])]
    
    # Dibujar polígono y guía de controles
    pygame.draw.polygon(screen, (0, 255, 150), points, 0)
    
    # Render texto informativo simple
    font = pygame.font.SysFont("Arial", 18)
    info = font.render(f"Flechas: Mover | W/S: Escalar ({scale:.2f})", True, (255, 255, 255))
    screen.blit(info, (10, 10))
    
    pygame.display.flip()
    clock.tick(60)
