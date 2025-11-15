# FR2025-1 Guia micro-ros
Guía rápida: instalar micro-ROS en ESP32‑S3 (Ubuntu + ROS 2 ya instalado)

Cargaremos varaibles de entorno: USER, ROS_DISTRO
```bash
export USER=$(whoami)
export ROS_DISTRO=humble  # Cambiar según la versión de ROS 2 instalada
```

## 1) Instalar Arduino IDE:
Descargar: https://www.arduino.cc/en/software/  
Mover a Carpeta Arduino, si no existe, crear:
```bash
cd
mkdir Arduino
cd Arduino
```
Copiar archivo descargado a esta carpeta usando el explorador de archivos o terminal.  
En mi caso es ´arduino-ide_2.3.6_Linux_64bit.AppImage´, ajustar según versión descargada.

Dar permisos de ejecución:
```bash
chmod +x arduino-ide_2.3.6_Linux_64bit.AppImage
```
Crear acceso directo:
```bash
nano arduino-ide.desktop
```

Pegar:
```
[Desktop Entry] 
Name=Arduino IDE
Comment=Arduino IDE
Exec=/home/$USER/Arduino/arduino-ide_2.3.6_Linux_64bit.AppImage
Icon=/home/$USER/Arduino/lib/arduino_icon.png
Terminal=false
Type=Application
Categories=Development;IDE;
```

Guardar y salir (Ctrl+O, Enter, Ctrl+X)  
Mover acceso directo a aplicaciones:
```bash
mv arduino-ide.desktop ~/.local/share/applications/
```

Abrir Arduino IDE desde aplicaciones.

## 2) Instalar ESP32 Board en IDE Arduino:
Ir a Archivo -> Preferencias  
En "Gestor de URLs adicionales de tarjetas" agregar:
```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```
Luego ir a Herramientas -> Placa -> Gestor de Tarjetas  
Buscar "esp32" e instalar "esp32 by Espressif Systems"  
Seleccionar Placa: Herramientas -> Placa -> ESP32S3 Dev Module

Ahora configurar el puerto serial: IMPORTANTE  
Conecta el ESP32-S3 a la computadora via USB  
Si es Maquina virtual, asegurarse que el USB este conectado a la VM.

```bash
sudo apt install -y usbutils

ls -la /dev | grep ttyACM # mi computadora me funciona ACM
```
Deberías tener una salida similar a:
```
crw-rw---- 1 root dialout 166, 0 Jun  12 10:00 ttyACM0
```
Si no hay ninguna respuesta, intenta con:
```bash
ls -la /dev | grep ttyUSB
```

Agregar tu usuario al grupo dialout para tener permisos de acceso al puerto serial:
```bash
sudo usermod -a -G dialout $USER
sudo chmod a+rw /dev/ttyACM0 # o /dev/ttyUSB0 según corresponda
```
Cerrar sesión y volver a entrar para que los cambios tengan efecto.

### Opcional: Testear con Blink
Si alguna vez ya has hecho un testing puedes saltarte esta parte, eso se hará para probar si todo salio bien, te lo puedes saltar solo es un test.  
Ir a Archivo -> Ejemplos -> 01.Basics -> Blink  
Seleccionar Puerto: Herramientas -> Puerto -> /dev/ttyUSB0 (o similar según tu sistema)  
Esto subirá el programa Blink al ESP32-S3, si todo sale bien, el LED en la placa debería parpadear.  
Si hay problemas, puede revisar si se debe a la comunicación con el puerto serial.

## 3) Instalar micro-ROS para Libreria de Arduino:
Descargar micro-ROS Arduino Library: https://github.com/micro-ROS/micro_ros_arduino/tree/humble  
Click en <>Code y luego Download ZIP  
Copiar ZIP a ~/Arduino  
Sketch -> Incluir Librería -> Añadir .ZIP Librería  
Seleccionar el ZIP descargado.


## 4) Como cargar un publicador simple:
Para probar que todo funciona, cargaremos un publicador simple de micro-ROS.  
Antes es necesario resolver un problema dentro de la librería:  
Ir a src de la librería:
```bash
cd ~/Arduino/libraries/micro_ros_arduino/src/
```
Buscar la carpeta esp32 y hacerle una copia, renombrarla a esp32s3:
```bash
cp -r esp32 esp32s3
```

Ahroa si podemos cargar el ejemplo:  
Ir a Archivo -> Ejemplos -> micro_ros_arduino -> basic_pub_sub -> simple_publisher  
Cargar el programa al ESP32-S3.


## 5) Como cargar micro-ROS Agent: (Para comunicar con ROS 2)
Para cargar el agente de micro-ROS, abrimos una terminal nueva y hacemos lo siguiente:
```bash
source /opt/ros/$ROS_DISTRO/setup.bash
cd ~
mkdir ws_test_microros
cd ws_test_microros
git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
colcon build
source install/setup.bash
source ~/ws_test_microros/install/local_setup.bash
ros2 run micro_ros_setup create_agent_ws.sh
ros2 run micro_ros_setup build_agent.sh
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0
```

Si todo salió bien, el agente debería estar corriendo y listo para recibir mensajes del ESP32-S3.
