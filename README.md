# ROS Noetic Bebop TMR 2026 Autonomous Drone
<p align="center">
  <img src="Captura de pantalla 2026-06-24 163749.png" alt="Estudiantes UDLAP obtuvieron segundo lugar en el Torneo Mexicano de Robotica 2026" width="720">
</p>
<p align="center">
  <img src="videos/bebop_preview.gif" alt="Funcionamiento del sistema autonomo con Parrot Bebop" width="300">
</p>

<p align="center">
  <a href="videos/bebop.mp4">Ver video completo</a>
</p>

Paquete ROS Noetic para controlar un dron Parrot Bebop 2 en misiones autonomas inspiradas en la categoria Drones Autonomos del Torneo Mexicano de Robotica 2026. El proyecto integra teleoperacion de seguridad, control de movimiento, percepcion con OpenCV, ArUco y YOLO, supervisores de mision y un orquestador de competencia.

El paquete ROS se llama `bebop_tmr`.

## Caracteristicas

- Conexion con Parrot Bebop mediante `bebop_driver`.
- Publicacion de comandos en `/bebop/cmd_vel`, `/bebop/takeoff`, `/bebop/land`, `/bebop/reset` y `/bebop/camera_control`.
- Recepcion de camara en `/bebop/image_raw` y odometria en `/bebop/odom`.
- Modulo de teleoperacion por teclado para pruebas y seguridad.
- Supervisor de misiones con estados `IDLE`, `TAKEOFF`, `MISSION`, `TELEOP` y `LANDING`.
- Misiones autonomas para ventana naranja, pizarron con ArUco, helipad/plataforma, trayectorias punto a punto y misiones de competencia.
- Modelos YOLO incluidos en `perception/models/` para ventana/objetivo y helipad.
- Orquestador de competencia en `missions/full_missions/competition_orchestrator.py`.

## Estructura

```text
bebop_tmr/
|-- bebop_core/                 # Supervisores, teleop y administrador autonomo
|-- control/                    # Controladores de movimiento del Bebop
|-- launch/                     # Launch files ROS
|-- missions/                   # Misiones autonomas y orquestador TMR
|-- perception/                 # Detectores OpenCV, ArUco y YOLO
|-- perception/models/          # Pesos YOLO incluidos
|-- videos/                     # Video completo y GIF de demostracion
|-- CMakeLists.txt
`-- package.xml
```

## Requisitos

Ambiente recomendado:

- Ubuntu 20.04.
- ROS Noetic.
- Python 3.
- Parrot Bebop 2 conectado por WiFi.
- `bebop_autonomy` o el paquete que provea `bebop_driver` y `bebop_msgs`.

Instala dependencias base:

```bash
sudo apt update
sudo apt install -y \
  python3-catkin-tools python3-pip python3-rosdep \
  ros-noetic-cv-bridge \
  ros-noetic-image-transport \
  ros-noetic-geometry-msgs \
  ros-noetic-sensor-msgs \
  ros-noetic-std-msgs \
  ros-noetic-nav-msgs \
  ros-noetic-tf \
  ros-noetic-rqt-image-view
```

Instala dependencias Python:

```bash
pip3 install --user numpy ultralytics
```

Si tu instalacion de OpenCV no incluye `cv2.aruco`, instala la version contrib:

```bash
pip3 install --user opencv-contrib-python
```

Instala el driver del Bebop dentro del mismo workspace. Una opcion comun es:

```bash
cd ~/bebop_ws/src
git clone https://github.com/AutonomyLab/bebop_autonomy.git
```

Luego resuelve dependencias con `rosdep`.

## Clonar Y Compilar

```bash
mkdir -p ~/bebop_ws/src
cd ~/bebop_ws/src
git clone https://github.com/FernandoLH7/ros-noetic-bebop-tmr-2026-autonomous-drone.git bebop_tmr

cd ~/bebop_ws
source /opt/ros/noetic/setup.bash
rosdep install --from-paths src --ignore-src -r -y
catkin config --extend /opt/ros/noetic
catkin build
source devel/setup.bash
```

Para reconstruir desde cero:

```bash
cd ~/bebop_ws
catkin clean -y
rm -rf build devel logs .catkin_tools
source /opt/ros/noetic/setup.bash
catkin config --extend /opt/ros/noetic
catkin build
source devel/setup.bash
```

## Conexion Con El Bebop

1. Enciende el dron.
2. Conecta la laptop a la red WiFi del Bebop.
3. Verifica que el IP del dron sea accesible. Por defecto se usa `192.168.42.1`.
4. En una terminal:

```bash
cd ~/bebop_ws
source /opt/ros/noetic/setup.bash
source devel/setup.bash
roslaunch bebop_tmr bebop_node.launch ip:=192.168.42.1
```

En otra terminal valida los topicos:

```bash
source ~/bebop_ws/devel/setup.bash
rostopic list | grep bebop
rostopic echo /bebop/odom
rqt_image_view /bebop/image_raw
```

## Pruebas Paso A Paso

### 1. Verificar sintaxis Python

Desde la raiz del paquete:

```bash
cd ~/bebop_ws/src/bebop_tmr
python3 -m compileall .
```

### 2. Verificar compilacion ROS

```bash
cd ~/bebop_ws
source /opt/ros/noetic/setup.bash
catkin build bebop_tmr
source devel/setup.bash
```

### 3. Probar driver y camara

Terminal 1:

```bash
roslaunch bebop_tmr bebop_node.launch ip:=192.168.42.1
```

Terminal 2:

```bash
rqt_image_view /bebop/image_raw
```

### 4. Probar detector ArUco del pizarron

Con el driver activo:

```bash
rosrun bebop_tmr test_aruco_bebop_camera.py
```

El detector busca un marcador ArUco con ID `100` usando el diccionario `DICT_5X5_250`.

### 5. Probar teleoperacion de seguridad

Con el driver activo, ejecuta:

```bash
rosrun bebop_tmr teleop_node.py
```

Controles principales:

- `1`: despegue y modo teleop.
- `2`: aterrizaje.
- `w`, `a`, `s`, `d`: desplazamiento horizontal.
- `q`, `e`: giro.
- `+`, `-`: subir y bajar.
- `i`, `j`, `k`, `l`: control de camara.
- `x`: emergencia.
- `Ctrl+C`: aterrizar y salir.

### 6. Probar mision autonoma del pizarron

```bash
roslaunch bebop_tmr whiteboard_autonomous.launch mission_name:=mission_whiteboard_aruco ip:=192.168.42.1
```

Este launch inicia el driver y el `autonomous_mission_manager.py`, que dispara la mision despues del retraso configurado.

### 7. Ejecutar misiones individuales

Con el driver activo:

```bash
rosrun bebop_tmr mission_orange_window.py
rosrun bebop_tmr mission_whiteboard_aruco.py
rosrun bebop_tmr mission_4.py
```

Las misiones publican su resultado en:

```bash
rostopic echo /mission/status
```

Valores esperados:

- `done`: mision completada.
- `failed`: mision fallida o abortada.

### 8. Probar el orquestador de competencia

Listar misiones disponibles:

```bash
rosrun bebop_tmr competition_orchestrator.py --list-missions
```

Ejecutar secuencia recomendada de prueba:

```bash
rosrun bebop_tmr competition_orchestrator.py --combo 1,p2p,2,4 --skip-prep
```

Ejecutar solo misiones 1 y 2:

```bash
rosrun bebop_tmr competition_orchestrator.py --combo 1,2 --tunnel medium --skip-prep
```

Ejecutar aterrizaje en plataforma:

```bash
rosrun bebop_tmr competition_orchestrator.py --combo 4 --platform static --skip-prep
```

## Flujo Del Sistema

```text
Bebop Driver
  |-- /bebop/image_raw  -> perception/
  |-- /bebop/odom       -> missions/ y control/
  `-- comandos          <- control/

perception/
  |-- Detecta ventana naranja con YOLO
  |-- Detecta helipad con YOLO
  `-- Detecta ArUco ID 100 con OpenCV

missions/
  |-- Ejecutan logica autonoma por estados
  |-- Publican /mission/status
  `-- Envian Twist, takeoff, land y camera_control

bebop_core/
  `-- Supervisa transiciones, teleop, misiones y emergencia
```

## Notas De Seguridad

- Prueba primero sin helices o con el dron asegurado cuando solo necesites validar topicos.
- Manten un piloto de seguridad listo para aterrizar o cortar motores.
- Ejecuta las misiones en un espacio controlado, con red protectora y baterias en buen estado.
- Ajusta velocidades, tolerancias y tiempos antes de pruebas reales cerca de obstaculos.

## Estado Actual

- Mision 1, ventana naranja: implementada con YOLO.
- Mision 2, pizarron con ArUco: implementada con OpenCV ArUco.
- Mision 3, med-kit: stub, requiere hardware de audio y mecanismo de entrega.
- Mision 4, helipad/plataforma: implementada con detector YOLO y parametros ROS.
- `launch/start.launch` se conserva como archivo legacy; para pruebas reales usa `bebop_node.launch`, `whiteboard_autonomous.launch` o los comandos `rosrun` documentados arriba.
