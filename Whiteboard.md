# Pruebas De La Mision Whiteboard ArUco

Esta guia resume las pruebas especificas para `mission_whiteboard_aruco.py`.

## Seguridad

Mientras corre la mision en una terminal interactiva:

- `q`: aterrizaje inmediato.
- `e`: emergencia dura mediante `/bebop/reset`.

Usa `q` si el dron sigue estable y solo quieres abortar. Usa `e` si existe riesgo real de choque o perdida de control.

## Preparacion

Verifica antes de iniciar:

- El Bebop esta encendido y conectado por WiFi.
- El workspace esta compilado con `catkin build`.
- La terminal tiene cargado `source devel/setup.bash`.
- El ArUco ID `100` esta visible frente a la camara.
- Hay espacio suficiente para abortar y aterrizar.

## Compilar

```bash
cd ~/bebop_ws
source /opt/ros/noetic/setup.bash
catkin build bebop_tmr
source devel/setup.bash
```

## Prueba 1: Camara Y Detector ArUco

Terminal 1:

```bash
roslaunch bebop_tmr bebop_node.launch ip:=192.168.42.1
```

Terminal 2:

```bash
rostopic hz /bebop/image_raw
rqt_image_view /bebop/image_raw
```

Terminal 3:

```bash
rosrun bebop_tmr test_aruco_bebop_camera.py
```

Resultado esperado:

- La camara publica en `/bebop/image_raw`.
- Se abre una ventana de depuracion.
- El marcador ID `100` se detecta con `err_x`, `err_y` y `area`.

## Prueba 2: Mision Por Etapas

Despega manualmente:

```bash
rostopic pub --once /bebop/takeoff std_msgs/Empty "{}"
```

Ejecuta solo busqueda y alineacion:

```bash
rosrun bebop_tmr mission_whiteboard_aruco.py _test_mode:=search_align _show_debug:=true
```

Ejecuta aproximacion:

```bash
rosrun bebop_tmr mission_whiteboard_aruco.py _test_mode:=approach _show_debug:=true
```

Ejecuta mision completa:

```bash
rosrun bebop_tmr mission_whiteboard_aruco.py _test_mode:=full _show_debug:=true
```

Aterriza al terminar:

```bash
rostopic pub --once /bebop/land std_msgs/Empty "{}"
```

## Monitoreo

Comandos utiles:

```bash
rostopic echo /bebop/cmd_vel
rostopic echo /bebop/odom
rostopic echo /mission/status
```

## Resultado Esperado

La mision ajusta la camara, busca el ArUco, centra el dron, se aproxima de forma segura, realiza el trazo y aterriza.
