# wp8_edutainment

Meta repository for the WP8 Edutainment use case.

## Hardware requirements

- GPU with CUDA.
- Wifi, needed to communicate with the Robobo robot.
- Webcam.
- 3D camera (OAK-D PRO WIDE) with tripod.
- Robobo robot and smartphone with Robobo app.

## Software requirements

- Ubuntu 22.04.5 LTS (Jammy Jellyfish): <https://releases.ubuntu.com/jammy/>
- ROS 2 Humble: <https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html>
- e-MDB: <https://docs.pillar-robots.eu/en/latest/how_to_do/how_to_install.html>
- edutainmet_emdb: <https://github.com/pillar-robots/edutainment_emdb.git>
- VSCode: <https://code.visualstudio.com/docs/setup/linux>
- Miniconda: <https://www.anaconda.com/docs/getting-started/miniconda/install/linux-install>
  - Create a miniconda environment, activate it and install Python on it.

```bash
mkdir -p ~/progtutor
cd ~/progtutor
conda update -n base -c defaults conda
conda env list
conda create --name pillar-progtutor
conda activate pillar-progtutor
conda install -n pillar-progtutor python=3.12.4
```

**IMPORTANT:** the following steps asume that you are in the "~/progtutor" folder and have the environment active. Install the rest of the dependencies on it:

- Git LFS: <https://git-lfs.com/>
- DepthAI: <https://docs.luxonis.com/software-v3/depthai>
- DepthAI-viewer: <https://github.com/luxonis/depthai-viewer>
- combined_action_recognition: <https://github.com/pillar-robots/combined_action_recognition>

```bash
# Download and install combined_action_recognition
git clone https://github.com/pillar-robots/combined_action_recognition.git
cd combined_action_recognition/
git lfs pull
cd combined_action_recognition/combined_action_pillar_core
python3 setup.py sdist bdist_wheel
python3 -m pip install .
python3 -m pip install mediapipe==0.10.21 --no-deps --upgrade
depthai-viewer # Open depthai-viewer to let it install its dependancies.
```

- VLM_engagement_pillar: <https://github.com/pillar-robots/VLM_engagement_pillar>
- robobopy: <https://github.com/mintforpeople/robobo-programming/wiki/python-doc#setting-up-your-computer>
- ProgTutor VSCode extension: <https://progtutor.citic.udc.es/docs/progtutor-vscode-eng>
- RoboboSim: <https://progtutor.citic.udc.es/docs/pillar-linux>

```bash
mkdir -p ~/progtutor/robobosim-pillar
cd ~/progtutor/robobosim-pillar
## Download the latest RoboboSim and unzip it here
```

## RoboboSim configuration

- Copy your Python path:

```bash
conda activate pillar-progtutor
which python3 # Copy this path, for example: /home/ubuntu/miniconda3/envs/pillar-progtutor/bin/python3
```

- RoboboSim -> Options:
  - "Language": English (en).
  - "Simpler Physics Model (beta)": checked.
  - "Visualize Speech as Text": checked.
  - "Speech Voice Type": any [enUS] voice.
  - "Use Custom Python Path": checked.
  - "Custom Python Path": paste here your Python path.
  - "Test Python" click it to check your Python + robobopy library installation.
  - "Use webcam": checked.
  - "Use 3D camera": checked.
  - "Webcam script arguments": --display --device cuda --interval 10
  - "3D camera script arguments": --display --device cuda --disable-gaze --detect-interval 30
  - "Robobo IP": paste here your Robobo IP shown in the Robobo app.
  - "Test Connection": check your communication with the Robobo robot.

## Launch the experiment

- Launch the edutainment_emdb experiment in one terminal:

```bash
source /opt/ros/humble/setup.bash
cd ~/eMDB_ws/
colcon build --packages-select edutainment_emdb --symlink-install
source install/setup.bash
ros2 launch edutainment_emdb edutainment_launch.py |& tee ~/edutainment_output.txt
```

- Open RoboboSim in another terminal:

```bash
source /opt/ros/humble/setup.bash
source ~/eMDB_ws/install/setup.bash
cd ~/progtutor/robobosim-pillar/pillar-linux
./RoboboSim\ PILLAR\ Linux.x86_64 -logFile ~/robobosim-pillar.log
```

## Troubleshooting

- **Problems with library dependencies:**
  - Download the requirements_pip_pillar.txt file and install them.
    - **WARNING** this method is not fully tested and it doesn't replace the manual installation of the required software.

```bash
conda activate pillar-progtutor
python3 -m pip install --upgrade -r requirements_pip_pillar.txt
python3 -m pip install mediapipe==0.10.21 --no-deps --upgrade
```

- **e-MDB**: build fails with CMake Error.
  - Solution: make sure to source setup.bash before build.

```bash
source /opt/ros/humble/setup.bash
colcon build --symlink-install
```

- **VSCode extension:** fails with KeyboardInterrupt, as VSCode tries to activate the CONDA environment AFTER launching the script.
  - Solution:
    - VSCode Configuration:
      - Set "python.useEnvironmentsExtension": true.
      - set "python-envs.terminal.autoActivationType" to "shellStartup".

- **combined_action:**
  - Problem: _pickle.UnpicklingError: invalid load key, 'v'.
  - Solución: install and configure git lfs, and clone repos from GitHub Desktop or VSCode.

  - Problem: ImportError: cannot import name 'solutions' from 'mediapipe' (/home/ubuntu/miniconda3/envs/pillar-progtutor/lib/python3.12/site-packages/mediapipe/__init__.py)
  - Solution: python3 -m pip install mediapipe==0.10.21 --no-deps --upgrade

  - Problem: [depthai] [warning] Insufficient permissions to communicate with X_LINK_UNBOOTED device having name "3.4". Make sure udev rules are set
  - Solution:
    echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="03e7", MODE="0666"' | sudo tee /etc/udev/rules.d/80-movidius.rules
    sudo udevadm control --reload-rules && sudo udevadm trigger