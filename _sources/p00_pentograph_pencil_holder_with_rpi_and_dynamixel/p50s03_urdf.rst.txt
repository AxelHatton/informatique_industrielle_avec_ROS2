###################################################
 Génération de la description URDF du pantographe
###################################################

Nous allons utiliser le modèle 3D au format step conçu et réalisé par M. Olivier PICCIN pour générer la description URDF du pantographe.

.. figure:: resources/img/pantograph_step_model.png
   :align: center

Il s'agit d'un assemblage comportant 5 pièces principales :

  #. BATI_ASM: le bâti de la strucuture du pantographe avec les 2 moteurs pas à pas. Dans l'URDF nous allons le nommer ``base``.
  #. LINK1_ASM: le bras gauche du pantographe. Dans l'URDF nous allons le nommer ``link1``.
  #. LINK2_ASM: l'avant bras gauche du pantographe. Dans l'URDF nous allons le nommer ``link2``.
  #. LINK3_ASM: l'avant bras droit du pantographe. Dans l'URDF nous allons le nommer ``link3``.
  #. LINK4_ASM: le bras droit du pantographe. Dans l'URDF nous allons le nommer ``link4``.

:download:`maquette-5-barres_asm.stp <resources/cad/maquette-5-barres_asm.stp>`

Du point de vue URDF, nous aurons aussi besoin de définir l'outil de travail: le crayon et donc de le référencer par rapport à son support qui se trouve être sur l'avant bras gauche du pentographe (``link2``).

Dans un fichier URDF, les modèles 3D sont utilisés à partir de fichiers collada (extension .dae).
Il va donc falloir convertir notre modèle 3D au format step en un ensemble de 5 modèles 3D au format collada.

==================================
Extraction des modèles 3D en step
==================================

Pour extraire les modèles 3D au format step, vous pouvez utiliser le logiciel freecad.
   
   #. :download:`base.stp <resources/cad/base.stp>`
   #. :download:`link1.stp <resources/cad/link1.stp>`
   #. :download:`link2.stp <resources/cad/link2.stp>`
   #. :download:`link3.stp <resources/cad/link3.stp>`
   #. :download:`link4.stp <resources/cad/link4.stp>`

=====================================
Conversion des modèles 3D en collada
=====================================

Pour convertir les modèles 3D en collada (dae), vous pouvez utiliser le logiciel freecad.
   
   #. :download:`base.dae <resources/cad/base.dae>`
   #. :download:`link1.dae <resources/cad/link1.dae>`
   #. :download:`link2.dae <resources/cad/link2.dae>`
   #. :download:`link3.dae <resources/cad/link3.dae>`
   #. :download:`link4.dae <resources/cad/link4.dae>`

===================
Création du package
===================

Tout d'abord créer le package du pantographe.

.. code-block:: bash
   
   ros2 pkg create panto_description --build-type ament_cmake

on devrait obtenir une arborescence de ce type

.. code-block:: bash
   
   panto_description/
   |__CMakeLists.txt
   |__package.xml
   |__src/

===============================
Création des dossiers standards
===============================

Créer les dossiers standards launch, meshes et urdf

.. code-block:: bash
   
   cd panto_description

.. code-block:: bash
   
   mkdir urdf meshes launch ros2_control config rviz gazebo

Vous devriez obtenir l'arborescence suivante

.. code-block:: bash
   
   panto_description/
   |__CMakeLists.txt
   |__package.xml
   |__src/
   |__launch/
   |__urdf/
   |__meshes/
   |__ros2_control/
   |__config/
   |__rviz/
   |__gazebo/

Copier les fichiers collada dans le répertoire panto_description/meshes/

Ouvrir le fichier CMakeLists.txt et remplacer par

.. code-block:: bash
   
   cmake_minimum_required(VERSION 3.8)
   project(panto_description)
   
   find_package(ament_cmake REQUIRED)
   
   install(
      DIRECTORY urdf meshes launch ros2_control config rviz gazebo
      DESTINATION share/${PROJECT_NAME}
   )
   ament_package()

=========================
Création du fichier URDF
=========================

Créer un fichier urdf dans le répertoire panto_description/urdf

.. code-block:: bash
   
   cd urdf

.. code-block:: bash
   
   touch panto.urdf

Dans un fichier URDF les modèles 3D sont référencés par des balises ``<mesh>``.

Copier le code urdf suivant dans le fichier panto.urdf

.. code-block:: bash
   
   <?xml version="1.0"?>
   <robot name="panto" xmlns:xacro="http://www.ros.org/wiki/xacro">

        <!-- Link pour fixer la base -->
        <link name="world"/>

        <link name="base_link">
            <visual>
                <geometry>
                    <mesh filename="package://panto_description/meshes/base.dae" scale="1 1 1"/>
                </geometry>
                <origin xyz="0 0 0" rpy="0 0 0" />
            </visual>
        </link>

        <joint name="base2world" type="fixed">
            <parent link="world"/>
            <child link="base_link"/>
        </joint>

        <link name="link1">
            <visual>
                <geometry>
                    <mesh filename="package://panto_description/meshes/link1.dae" scale="1 1 1"/>
                </geometry>
                <origin xyz="0.08 0 -0.07" rpy="0 0 0" />
            </visual>
        </link>

        <joint name="base_link_link1_joint" type="revolute">
            <parent link="base_link"/>
            <child link="link1"/>
            <origin xyz="-0.08 0 0.07" rpy="0 0 0"/>
            <axis xyz="0 1 0"/>
            <limit effort="10" lower="-1.57" upper="1.57" velocity="1.0"/>
        </joint>

        <link name="link2">
            <visual>
                <geometry>
                    <mesh filename="package://panto_description/meshes/link2.dae" scale="1 1 1"/>
                </geometry>
                <origin xyz="0.08 0 0.01" rpy="0 0 0" />
            </visual>
        </link>

        <joint name="link1_link2_joint" type="revolute">
            <parent link="link1"/>
            <child link="link2"/>
            <origin xyz="0 0 -0.08" rpy="0 0 0"/>
            <axis xyz="0 1 0"/>
            <limit effort="10" lower="-1.57" upper="1.57" velocity="1.0"/>
        </joint>

        <link name="link4">
            <visual>
                <geometry>
                    <mesh filename="package://panto_description/meshes/link4.dae" scale="1 1 1"/>
                </geometry>
                <origin xyz="-0.058 0 -0.07" rpy="0 0 0" />
            </visual>
        </link>

        <joint name="base_link_link4_joint" type="revolute">
            <parent link="base_link"/>
            <child link="link4"/>
            <origin xyz="0.058 0 0.070" rpy="0 0 0"/>
            <axis xyz="0 1 0"/>
            <limit effort="10" lower="-1.57" upper="1.57" velocity="1.0"/>
        </joint>

        <link name="link3">
            <visual>
                <geometry>
                    <mesh filename="package://panto_description/meshes/link3.dae" scale="1 1 1"/>
                </geometry>
                <origin xyz="-0.0902347 0 0.00212437" rpy="0 0 0" />
            </visual>
        </link>

        <joint name="link4_link3_joint" type="revolute">
            <parent link="link4"/>
            <child link="link3"/>
            <origin xyz="0.0322347 0 -0.07212437" rpy="0 0 0"/>
            <axis xyz="0 1 0"/>
            <limit effort="10" lower="-1.57" upper="1.57" velocity="1.0"/>
        </joint>

    </robot>
