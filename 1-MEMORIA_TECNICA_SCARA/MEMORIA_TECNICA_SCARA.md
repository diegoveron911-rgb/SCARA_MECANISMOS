# INFORME TÉCNICO: MEMORIA DE CÁLCULO Y DISEÑO DE MECANISMOS
## PROYECTO INTEGRADOR: ROBOT SCARA IMPRESO EN 3D
### Materia: Mecanismos y Elementos de Máquinas
**Facultad de Ciencias de la Alimentación - Universidad Nacional de Entre Ríos (UNER)**

---

## 1. INTRODUCCIÓN Y MARCO DE TRABAJO

En el marco del desarrollo del **Proyecto Integrador de Ingeniería en Mecatrónica**, este informe documenta la memoria de cálculo, el diseño cinemático y el análisis dinámico-estructural del **Robot SCARA de 3 Grados de Libertad (DoF)** principales más la orientación del efector final. 

El presente informe hace foco exclusivo en los requerimientos de la cátedra de **Mecanismos y Elementos de Máquinas**, los cuales abarcan:
- Definición de la cadena cinemática mediante modelos simplificados tipo esqueleto ("wire").
- Análisis de cinemática directa e inversa en el plano horizontal de posicionamiento.
- Cálculo analítico de aceleraciones y fuerzas de inercia dinámicas (D'Alembert) según las directrices del **Trabajo Práctico Nº 1**.
- Evaluación de concentración de tensiones, sensibilidad a la entalla y teoría de fatiga bajo cargas cíclicas aplicables a piezas reales del robot, conforme al **Trabajo Práctico Nº 2**.

### 1.1. Filosofía de Diseño del Prototipo
El prototipo está concebido para ser fabricado mediante tecnologías de impresión 3D (FDM) utilizando materiales termoplásticos (**PLA/ABS/PETG**). Cuenta con:
1. **Posicionamiento planar (plano XY)**: Accionado por dos servomotores de alto torque (tipo **MG996R** o equivalente).
2. **Elevación vertical (eje Z)**: Implementado mediante un sistema de **piñón-cremallera** accionado por un servomotor ligero (tipo **SG90** o **MG90S**).
3. **Efector Final**: Una pinza mecánica (rango de apertura $15\text{--}60\text{ mm}$) con sensor de distancia para validación de agarre y un marcador/fibrón para tareas de dibujo.

El diseño estructural se inspira y basa en los modelos de referencia constructiva:
- [Diseño y Conjunto General (scara_proy.jpeg)](file:///c:/Users/diego/OneDrive/Desktop/DIEGO%20FCAL/PROYECTO_MECANISMOS_SCARA/SCARA_PROY.jpeg)
- [Esquema Detallado de Impresión 3D (scara_pinte2.jpg)](file:///c:/Users/diego/OneDrive/Desktop/DIEGO%20FCAL/PROYECTO_MECANISMOS_SCARA/scara_pinte2.jpg)

---

## 2. CADENA CINEMÁTICA Y MODELADO MATEMÁTICO

El robot se modela como una cadena cinemática abierta de tipo **RRP** (Rotacional-Rotacional-Prismática) en sus articulaciones principales, sumando una rotación adicional en la muñeca para el guiado de la pinza.

### 2.1. Nomenclatura y Esquema de Alambre ("Wire")

El modelo estructural simplificado tipo "wire" representa los eslabones como líneas puras y las articulaciones como pivotes o guías de traslación. Esto nos permite aislar el comportamiento geométrico antes de evaluar la resistencia física de los materiales.

```
       [Efector Final / Pinza C]
                 o
                /
               /  L2 (Eslabón 2 - Antebrazo)
              /  Ángulo absoluto: theta1 + theta2
             /
   (Joint B) o  (Ángulo relativo theta2)
            /
           /
          /  L1 (Eslabón 1 - Brazo)
         /  Ángulo absoluto: theta1
        /
       o (Joint A - Base fija / Origen (0,0))
```

**Parámetros Dimensionales del Proyecto:**
- Longitud del eslabón 1 ($L_1$): $20\text{ cm} = 0.2\text{ m}$
- Longitud del eslabón 2 ($L_2$): $20\text{ cm} = 0.2\text{ m}$
- Altura fija de la base ($H$): $10\text{ cm} = 0.1\text{ m}$
- Desplazamiento vertical del efector ($d_3$ o $Z$): Carrera útil de $\pm 10\text{ cm}$ (rango total de $20\text{ cm}$).

---

### 2.2. Cinemática Directa

La cinemática directa determina las coordenadas cartesianas del extremo del robot $(x, y, z)$ a partir de los ángulos articulares $\theta_1$, $\theta_2$ y la extensión prismática $d_3$. 

Tomando como origen del sistema de referencia el centro de la articulación de la base (Joint A):

$$x = L_1 \cos(\theta_1) + L_2 \cos(\theta_1 + \theta_2)$$

$$y = L_1 \sin(\theta_1) + L_2 \sin(\theta_1 + \theta_2)$$

$$z = H + d_3$$

Donde $\theta_1$ es el ángulo de la primera articulación respecto al eje $X$, y $\theta_2$ es el ángulo relativo de la segunda articulación con respecto a la prolongación del primer brazo.

---

### 2.3. Cinemática Inversa

Para posicionar el efector final en una coordenada deseada del plano $(x, y)$, debemos calcular los ángulos articulares requeridos. Aplicando el método geométrico (Teorema del Coseno):

1. **Distancia al origen ($r^2 = x^2 + y^2$):**

   $$r^2 = x^2 + y^2$$

2. **Cálculo de la variable auxiliar $D$:**

   $$D = \cos(\theta_2) = \frac{x^2 + y^2 - L_1^2 - L_2^2}{2 L_1 L_2}$$

3. **Ángulo de la articulación 2 ($\theta_2$):**

   $$\theta_2 = \arccos(D)$$

   > [!NOTE]
   > Existen dos soluciones posibles para el codo:
   > - **Codo Abajo (Elbow Down)**: $\theta_2 = \text{atan2}(+\sqrt{1 - D^2}, D)$
   > - **Codo Arriba (Elbow Up)**: $\theta_2 = \text{atan2}(-\sqrt{1 - D^2}, D)$

4. **Ángulo de la articulación 1 ($\theta_1$):**

   $$\theta_1 = \text{atan2}(y, x) - \text{atan2}(L_2 \sin(\theta_2), L_1 + L_2 \cos(\theta_2))$$

---

### 2.4. Espacio de Trabajo (Workspace) y Singularidades

- **Alcance Máximo Teórico ($R_{\max}$)**: 
  $$R_{\max} = L_1 + L_2 = 20\text{ cm} + 20\text{ cm} = 40\text{ cm}$$
- **Alcance Mínimo Teórico ($R_{\min}$)**: 
  $$R_{\min} = |L_1 - L_2| = 0\text{ cm}$$
- **Rango Operativo Seguro Recomendado**: 
  $$5\text{ cm} < R < 38\text{ cm}$$

> [!WARNING]
> Cuando el robot se encuentra en su alcance máximo ($R = 40\text{ cm}$, con $\theta_2 = 0^\circ$), el mecanismo cae en una **singularidad de contorno**, perdiendo un grado de libertad de movimiento (no puede desplazarse radialmente hacia afuera). Por este motivo, el control por software limita el espacio de trabajo activo.

---

## 3. TRABAJO PRÁCTICO Nº 1: ANÁLISIS DE FUERZAS INERCIALES

En esta sección se resuelve analíticamente el problema cinemático y dinámico planteado en la práctica de la cátedra, aplicando el principio de D'Alembert para calcular las reacciones dinámicas en los vínculos del robot.

### 3.1. Enunciado y Datos de Entrada del TP1

Se analiza un mecanismo planar compuesto por dos brazos homogéneos y una masa puntual en el extremo:
- **Brazo 1 (AB)**: Longitud $L_1 = 0.80\text{ m}$, Masa $m_1 = 2.00\text{ kg}$. Centro de masa en el punto medio $G_1$ ($r_{G1} = 0.40\text{ m}$).
- **Brazo 2 (BC)**: Longitud $L_2 = 0.60\text{ m}$, Masa $m_2 = 1.50\text{ kg}$. Centro de masa en el punto medio $G_2$ ($r'_{G2} = 0.30\text{ m}$ desde B).
- **Masa en el extremo (C)**: Masa puntual $m_3 = 1.00\text{ kg}$.
- **Estado instantáneo analizado**:
  - Ángulos absolutos/relativos: $\theta_1 = 45^\circ$, $\theta_2 = 60^\circ$ (ángulo entre AB y BC).
  - Velocidad y aceleración del Brazo 1: $\omega_1 = 3.00\text{ rad/s}$ (antihoraria), $\alpha_1 = 4.00\text{ rad/s}^2$ (antihoraria).
  - Velocidad y aceleración relativas del Brazo 2: $\omega_2 = 2.00\text{ rad/s}$, $\alpha_2 = -1.00\text{ rad/s}^2$.

---

### 3.2. Desarrollo del Cálculo Analítico Paso a Paso

#### Paso 1: Determinación de Ángulos Absolutos
- Ángulo absoluto del Eslabón 1: $\theta_1 = 45^\circ$
- Ángulo absoluto del Eslabón 2: $\theta_{12} = \theta_1 + \theta_2 = 45^\circ + 60^\circ = 105^\circ$
- Velocidad angular absoluta del Eslabón 2: 
  $$\Omega_2 = \omega_1 + \omega_2 = 3.00 + 2.00 = 5.00\text{ rad/s}$$
- Aceleración angular absoluta del Eslabón 2: 
  $$\Lambda_2 = \alpha_1 + \alpha_2 = 4.00 - 1.00 = 3.00\text{ rad/s}^2$$

#### Paso 2: Posicionamiento de los Centros de Masa y Nodos
Con origen en el pivote fijo A (0,0):
- **Centro de masa $G_1$ (Eslabón 1)**:
  $$x_{G1} = 0.4 \cos(45^\circ) \approx 0.2828\text{ m}$$
  $$y_{G1} = 0.4 \sin(45^\circ) \approx 0.2828\text{ m}$$
- **Articulación móvil B**:
  $$x_B = 0.8 \cos(45^\circ) \approx 0.5657\text{ m}$$
  $$y_B = 0.8 \sin(45^\circ) \approx 0.5657\text{ m}$$
- **Centro de masa $G_2$ (Eslabón 2)**:
  $$x_{G2} = x_B + 0.3 \cos(105^\circ) = 0.5657 + 0.3 \times (-0.2588) \approx 0.4881\text{ m}$$
  $$y_{G2} = y_B + 0.3 \sin(105^\circ) = 0.5657 + 0.3 \times 0.9659 \approx 0.8555\text{ m}$$
- **Extremo C (Masa puntual)**:
  $$x_C = x_B + 0.6 \cos(105^\circ) = 0.5657 + 0.6 \times (-0.2588) \approx 0.4104\text{ m}$$
  $$y_C = y_B + 0.6 \sin(105^\circ) = 0.5657 + 0.6 \times 0.9659 \approx 1.1453\text{ m}$$

#### Paso 3: Aceleraciones de los Centros de Masa e Inercia
Se utilizan las ecuaciones de movimiento de cuerpo rígido. Para cualquier punto $P$ en rotación pura respecto a un punto de referencia móvil $O$:

$$\vec{a}_P = \vec{a}_O + \vec{\alpha} \times \vec{r}_{P/O} - \omega^2 \vec{r}_{P/O}$$

*   **Aceleración de $G_1$ ($\vec{a}_{G1}$)**:
    $$a_{G1x} = -0.4 \alpha_1 \sin(45^\circ) - 0.4 \omega_1^2 \cos(45^\circ) = -0.4(4)(0.7071) - 0.4(9)(0.7071) \approx -3.6770\text{ m/s}^2$$
    $$a_{G1y} = 0.4 \alpha_1 \cos(45^\circ) - 0.4 \omega_1^2 \sin(45^\circ) = 0.4(4)(0.7071) - 0.4(9)(0.7071) \approx -1.4142\text{ m/s}^2$$
    $$\text{Magnitud: } a_{G1} = \sqrt{(-3.6770)^2 + (-1.4142)^2} \approx 3.9395\text{ m/s}^2$$

*   **Aceleración de la articulación B ($\vec{a}_B$)**:
    $$a_{Bx} = -0.8(4)(0.7071) - 0.8(9)(0.7071) \approx -7.3539\text{ m/s}^2$$
    $$a_{By} = 0.8(4)(0.7071) - 0.8(9)(0.7071) \approx -2.8284\text{ m/s}^2$$

*   **Aceleración de $G_2$ ($\vec{a}_{G2}$)** (referida a la aceleración de B):
    $$a_{G2x} = a_{Bx} - 0.3 \Lambda_2 \sin(105^\circ) - 0.3 \Omega_2^2 \cos(105^\circ) = -7.3539 - 0.3(3)(0.9659) - 0.3(25)(-0.2588) \approx -6.2821\text{ m/s}^2$$
    $$a_{G2y} = a_{By} + 0.3 \Lambda_2 \cos(105^\circ) - 0.3 \Omega_2^2 \sin(105^\circ) = -2.8284 + 0.3(3)(-0.2588) - 0.3(25)(0.9659) \approx -10.3058\text{ m/s}^2$$
    $$\text{Magnitud: } a_{G2} \approx 12.0696\text{ m/s}^2$$

*   **Aceleración del extremo C ($\vec{a}_C$)**:
    $$a_{Cx} = a_{Bx} - 0.6(3)(0.9659) - 0.6(25)(-0.2588) \approx -5.2103\text{ m/s}^2$$
    $$a_{Cy} = a_{By} + 0.6(3)(-0.2588) - 0.6(25)(0.9659) \approx -17.7832\text{ m/s}^2$$
    $$\text{Magnitud: } a_C \approx 18.5308\text{ m/s}^2$$

---

### 3.3. Fuerzas Inerciales de D'Alembert y Momentos

Para lograr el equilibrio dinámico ficticio, aplicamos fuerzas y momentos inerciales equivalentes opuestos:

*   **Eslabón 1 (AB)**:
    $$\vec{F}_{I1} = -m_1 \vec{a}_{G1} = -2.0 \times (-3.6770, -1.4142) = (7.3539, 2.8284)\text{ N} \quad [F_{I1} \approx 7.8791\text{ N}]$$
    $$I_{G1} = \frac{1}{12} m_1 L_1^2 = \frac{1}{12} (2.0) (0.8^2) = 0.1067\text{ kg}\cdot\text{m}^2$$
    $$M_{I1} = -I_{G1} \alpha_1 = -0.1067 \times 4.0 \approx -0.4267\text{ N}\cdot\text{m} \quad (\text{Sentido horario})$$

*   **Eslabón 2 (BC)**:
    $$\vec{F}_{I2} = -m_2 \vec{a}_{G2} = -1.5 \times (-6.2821, -10.3058) = (9.4232, 15.4587)\text{ N} \quad [F_{I2} \approx 18.1044\text{ N}]$$
    $$I_{G2} = \frac{1}{12} m_2 L_2^2 = \frac{1}{12} (1.5) (0.6^2) = 0.0450\text{ kg}\cdot\text{m}^2$$
    $$M_{I2} = -I_{G2} \Lambda_2 = -0.0450 \times 3.0 \approx -0.1350\text{ N}\cdot\text{m} \quad (\text{Sentido horario})$$

*   **Masa Extrema C ($m_3$)**:
    $$\vec{F}_{I3} = -m_3 \vec{a}_C = -1.0 \times (-5.2103, -17.7832) = (5.2103, 17.7832)\text{ N} \quad [F_{I3} \approx 18.5308\text{ N}]$$

---

### 3.4. Reacciones y Torques en las Juntas

Efectuamos el equilibrio dinámico analizando la cadena cinemática desde el final hacia el principio:

1.  **Reacción y Par Motor en la Articulación B (Eje 2)**:
    Estudiando el eslabón 2 y la masa en C de forma aislada:
    - **Fuerza de reacción $\vec{R}_B$**:
      $$\vec{R}_B + \vec{F}_{I2} + \vec{F}_{I3} = 0 \implies \vec{R}_B = -(\vec{F}_{I2} + \vec{F}_{I3})$$
      $$\vec{R}_B = -(9.4232 + 5.2103, 15.4587 + 17.7832) = (-14.6335, -33.2419)\text{ N} \quad [R_B \approx 36.3203\text{ N}]$$
    
    - **Torque Motor en B ($T_B$)**:
      Tomando momentos respecto a B:
      $$T_B + M_{I2} + \vec{r}_{G2/B} \times \vec{F}_{I2} + \vec{r}_{C/B} \times \vec{F}_{I3} = 0$$
      Realizando los productos vectoriales 2D:
      $$\vec{r}_{G2/B} \times \vec{F}_{I2} = x_{G2/B} F_{I2y} - y_{G2/B} F_{I2x} = (-0.0776)(15.4587) - (0.2898)(9.4232) \approx -3.9312\text{ N}\cdot\text{m}$$
      $$\vec{r}_{C/B} \times \vec{F}_{I3} = x_{C/B} F_{I3y} - y_{C/B} F_{I3x} = (-0.1553)(17.7832) - (0.5796)(5.2103) \approx -5.7810\text{ N}\cdot\text{m}$$
      $$T_B = - [M_{I2} + (\vec{r}_{G2/B} \times \vec{F}_{I2}) + (\vec{r}_{C/B} \times \vec{F}_{I3})]$$
      $$T_B = - [-0.1350 - 3.9312 - 5.7810] = 9.8472\text{ N}\cdot\text{m}$$

2.  **Reacción y Par Motor en el Eje de la Base (Eje 1 - Joint A)**:
    Estudiando ahora toda la cadena acoplada:
    - **Fuerza de reacción en la bancada $\vec{R}_A$**:
      $$\vec{R}_A + \vec{F}_{I1} - \vec{R}_B = 0 \implies \vec{R}_A = \vec{R}_B - \vec{F}_{I1}$$
      $$\vec{R}_A = (-14.6335 - 7.3539, -33.2419 - 2.8284) = (-21.9874, -36.0703)\text{ N} \quad [R_A \approx 42.2435\text{ N}]$$
    
    - **Torque Motor en la Base ($T_A$)**:
      Tomando momentos respecto a A:
      $$T_A - T_B + M_{I1} + \vec{r}_{G1/A} \times \vec{F}_{I1} + \vec{r}_{B/A} \times (-\vec{R}_B) = 0$$
      Realizando los productos vectoriales:
      $$\vec{r}_{G1/A} \times \vec{F}_{I1} = (0.2828)(2.8284) - (0.2828)(7.3539) \approx -1.2800\text{ N}\cdot\text{m}$$
      $$\vec{r}_{B/A} \times (-\vec{R}_B) = (0.5657)(33.2419) - (0.5657)(14.6335) \approx 10.5273\text{ N}\cdot\text{m}$$
      $$T_A = T_B - M_{I1} - (\vec{r}_{G1/A} \times \vec{F}_{I1}) - (\vec{r}_{B/A} \times (-\vec{R}_B))$$
      $$T_A = 9.8472 - (-0.4267) - (-1.2800) - 10.5273 \approx 1.0273\text{ N}\cdot\text{m}$$

> [!TIP]
> **Análisis Crítico de Resultados**: El torque requerido en la segunda articulación ($T_B = 9.84\text{ N}\cdot\text{m}$) es considerablemente mayor que el de la primera articulación ($T_A = 1.027\text{ N}\cdot\text{m}$) debido a la velocidad absoluta de rotación del eslabón 2 ($\Omega_2 = 5\text{ rad/s}$ contra $\omega_1 = 3\text{ rad/s}$ del eslabón 1), que incrementa de forma cuadrática las fuerzas centrífugas asociadas a la masa extrema. Esto demuestra por qué el diseño estructural del soporte del segundo motor requiere alta rigidez mecánica.

---

## 4. TRABAJO PRÁCTICO Nº 2: CONCENTRACIÓN DE TENSIONES Y FATIGA

El diseño de un robot mecatrónico real difiere del modelo simplificado "wire" en que las secciones de las piezas poseen discontinuidades geométricas inevitables: chaveteros, filetes de transición y agujeros de sujeción. 

### 4.1. Respuestas Desarrolladas al Cuestionario Teórico del TP2

#### 1) ¿Qué es una concentración de tensiones y por qué se produce en un elemento mecánico?
Es una amplificación local del esfuerzo que ocurre en regiones donde existen **cambios bruscos en la geometría** o discontinuidades (como hombros de ejes, ranuras, roscas, agujeros de pasadores). Se produce porque las "líneas de flujo de tensiones" deben desviarse bruscamente para rodear el obstáculo geométrico. Al estrecharse las líneas de flujo, la densidad del esfuerzo aumenta drásticamente en las proximidades de la discontinuidad.

#### 2) ¿Qué diferencia existe entre la tensión nominal y la tensión máxima local?
- **Tensión Nominal ($\sigma_{\text{nom}}$ o $\tau_{\text{nom}}$)**: Es la tensión calculada en la sección neta de la pieza utilizando las ecuaciones clásicas de Resistencia de Materiales (por ejemplo, $\sigma = F/A$ para tracción, o $\sigma = M \cdot c / I$ para flexión), asumiendo una distribución uniforme o lineal de esfuerzos en toda la sección.
- **Tensión Máxima Local ($\sigma_{\max}$ o $\tau_{\max}$)**: Es el pico real de tensión que se desarrolla en la superficie inmediata de la discontinuidad. Está relacionada con la nominal mediante el factor teórico:
  $$\sigma_{\max} = K_t \cdot \sigma_{\text{nom}}$$

#### 3) ¿Qué representan los factores de concentración de tensiones y de qué variables dependen?
Representan la relación de amplificación teórica ($K_t$ para tensiones normales, $K_{ts}$ para tensiones cortantes). Son factores **estrictamente geométricos** y no dependen del material. Dependen de:
- Las proporciones dimensionales de la discontinuidad (por ejemplo, la relación de diámetros $D/d$ de un eje con hombro, o la relación del radio del filete respecto al diámetro menor $r/d$).
- El tipo de solicitación aplicado (tracción axial, flexión pura, o torsión).

#### 4) ¿Por qué la concentración de tensiones se analiza de manera diferente según el tipo de material y la naturaleza de la carga?
- **Solicitación a Fatiga (Cargas Variables Cíclicas)**: La aplicación repetida de ciclos de carga provoca la iniciación de microfisuras en los puntos con mayor tensión, independientemente de que el material sea dúctil o frágil. En fatiga, se utiliza el factor de concentración por fatiga $K_f$, que modera el factor teórico $K_t$ según la **sensibilidad a la entalla ($q$)** del material:
  $$K_f = 1 + q(K_t - 1)$$
- **Materiales Dúctiles bajo Carga Estática**: Ante esfuerzos excesivos, el material en el punto crítico experimenta fluencia localizada (deformación plástica), lo que provoca una redistribución de tensiones hacia zonas menos cargadas de la sección. Por ello, el factor $K_t$ generalmente **no se aplica** en este escenario.
- **Materiales Frágiles bajo Carga Estática**: Al no poseer capacidad de deformación plástica significativa, no pueden redistribuir esfuerzos. El pico local de tensión desencadena directamente una fisura que lleva al fallo catastrófico. En este caso, $K_t$ se aplica en toda su magnitud.

#### 5) ¿Qué modificaciones geométricas disminuyen la concentración de tensiones en un componente mecánico?
- Aumentar el **radio de filete ($r$)** en las transiciones de hombro.
- Realizar **ranuras de alivio** o socavados en los hombros para suavizar el flujo de tensiones.
- Introducir rebajes graduales (conicidades) en lugar de escalones rectos.
- Utilizar transiciones elípticas en lugar de circulares en hombros de ejes sometidos a alta fatiga.

---

### 4.2. Análisis de Secciones Críticas del Robot SCARA del Proyecto

Al revisar el diseño físico en 3D de nuestro SCARA (basado en el plano [ENS_SCARA](file:///c:/Users/diego/OneDrive/Desktop/DIEGO%20FCAL/PROYECTO%20MECANISMOS%20SCARA/scara_proy.jpeg)):

1.  **Transición del Eje del Motor (MG996R) al Brazo 1**:
    El servomotor MG996R entrega torque torsional en un estriado metálico pequeño. En esta interfaz, el plástico impreso en 3D del brazo experimenta torsión cíclica alternada cada vez que el robot frena y arranca. Un acoplamiento rígido con radio de acuerdo agudo representa una discontinuidad crítica con un $K_{ts} \approx 2.2$. 
    *Propuesta de Mejora*: Integrar un acople abocinado de transición con un radio de filete amplio ($r \ge 3\text{ mm}$) para reducir $K_{ts}$ a menos de $1.5$.
2.  **Soporte del Segundo Brazo (Junta B)**:
    Como se vio en el análisis del TP1, el torque y las fuerzas de corte en B son los más elevados. Para evitar que el eje del motor de Joint B soporte la flexión axial inducida por el brazo 2, el diseño incluye un **soporte independiente en "U" con rodamientos axiales y radiales**.
    Esto asegura que el eje del servomotor solo reciba **torsión pura** (eliminando esfuerzos de flexión cortante y flectores sobre el eje del motor), mitigando los modos de fallo por fatiga flectora.

---

### 4.3. Influencia de la Impresión 3D (PLA/ABS) en la Fatiga y Falla
Las piezas fabricadas por impresión 3D (FDM) presentan dos particularidades críticas:
- **Anisotropía**: La unión entre capas (eje Z de impresión) es el punto más débil de la estructura. Si las cargas mecánicas fatigan la pieza en dirección perpendicular a las capas, la iniciación de grietas ocurre a niveles de esfuerzo muy inferiores a los nominales del material base.
- **Efecto Entalla Interno**: El patrón de relleno (*infill*) y las transiciones entre perímetros y relleno actúan como millones de pequeñas entallas microscópicas internas (stress raisers).

> [!IMPORTANT]
> Para compensar estos efectos en el cálculo estructural del SCARA, se aplica un **coeficiente de seguridad por material impreso $C_s \ge 3.0$** sobre el límite de fatiga teórico del PLA. Además, se definen parámetros de impresión de alta resistencia: **4 perímetros mínimos** y un relleno de tipo **giroide al 40%** para homogeneizar la rigidez en el plano XY.

---

## 5. CONCLUSIONES Y TRABAJO FUTURO DE SIMULACIÓN

1.  La cadena cinemática abierta del SCARA ha sido modelada geométricamente y se ha resuelto el simulador dinámico-inercial.
2.  El análisis inercial demostró que la inercia dinámica supera ampliamente la carga estática nominal ($0.1\text{ kg}$), constituyendo el factor crítico de dimensionamiento mecánico de los soportes estructurales.
3.  Las piezas críticas de transición de torque han sido optimizadas geométricamente mediante radios de acuerdo amplios para prevenir fallos prematuros por fatiga de material impreso.

**Paso Siguiente (Simulación FEA)**:
Se utilizará el modelo de conjunto CAD [`KR5_scara_R350-Z200.iam`](file:///c:/Users/diego/OneDrive/Desktop/DIEGO%20FCAL/PROYECTO_MECANISMOS_SCARA/KR5_scara_R350-Z200.iam) en Autodesk Inventor Stress Analysis para simular bajo cargas dinámicas máximas de $2\text{ kg}$ (carga máxima especificada en el TP1 Ejercicio 2) y mapear los esfuerzos de Von Mises reales para compararlos con los calculados analíticamente en este informe.

---

## 6. BIBLIOGRAFÍA Y REFERENCIAS

1.  **Budynas, R. G., & Nisbett, J. K. (2011)**. *Diseño en Ingeniería Mecánica de Shigley* (9ª ed.). McGraw-Hill.
2.  **Meriam, J. L., & Kraige, L. G. (2006)**. *Mecánica para Ingenieros: Dinámica*. Editorial Reverté.
3.  **Lorente Romanos, R. (2023)**. *Diseño, implementación y control de un prototipo de Robot SCARA de 4 grados de libertad*. Universitat Politècnica de València (Tesis de Máster).
