---
title: 'Working With Your Neurodesk For Neuroimaging Data'
date: 2026-05-22
permalink: /posts/2026/05/2026-05-22-Working-With_Neurodesk-For-Neuroimaging/
author: Lookman Jamad, Mehdi Ketata, Sandrine Muller
tags:
  - Data Science
  - Neuroimaging
  - Neurodesk
  - Statistical Parametric Mapping (SPM)
  - Docker
  - Neurosciences workflows
---

As a data analyst or a student needing to perform complex neuroimaging tasks or workflows, you may be interested in accessing a wide array of specialized neuroimaging tools without the headache of manual installation. 

In this post, we will explore how **Neurodesk** provides a streamlined, containerized environment to run powerful neuroimaging tools seamlessly. We will walk through the setup process and show you how to get your environment ready for analysis.

---

# Neurodesk Installation Guide

Neurodesk is an open-source platform that provides ready-to-use neuroscience tools in a reproducible environment. It uses containers (via Docker or other engines) so you don’t need to manually install complex dependencies.

Learn more about it at: [https://neurodesk.org/](https://neurodesk.org/)

## Overview

Neurodesk simplifies neuroscience workflows by providing everything pre-configured and reproducible. It is primarily used for:

* **Neuroimaging analysis**: (MRI, fMRI, diffusion imaging, etc.)
* **Tool management**: Running software like FSL, SPM, and others without manual installation.
* **Reproducible research**: Ensuring the same environment across different machines.
* **Education**: Teaching and tutorials in neuroscience.
* **Flexibility**: Working both locally or remotely with the same setup.

## System Requirements & Prerequisites

Before you begin, ensure your system meets the following criteria:

* **Disk Space**: At least 5GB of free space.
* **System Privileges**: 
    * **With Privileged Access (Admin)**: Docker (recommended) or Podman.
    * **Without Privileged Access**: TinyRange Engine (included with Neurodesk App) and QEMU (macOS only).

> [!IMPORTANT]
> **Docker must be running** before starting Neurodesk.


## Installation Guide

Choose the section corresponding to your operating system.

### 🍎 macOS/Linux

#### 1. Checking system privileges
On macOS, Docker is the standard and easiest option. Ensure you have admin rights to install Docker Desktop.

#### 2. Docker Installation
1. Go to the official Docker website: [https://www.docker.com/](https://www.docker.com/)
2. Download **Docker Desktop for Mac**.
3. Install it like a normal application.
4. Launch Docker and ensure it is running (look for the Docker icon in the menu bar).

#### 3. Neurodesk Installation
1. Go to [https://neurodesk.org/](https://neurodesk.org/).
2. Click on **Install Locally**.
3. Follow the macOS installation prompts.

---

### 🪟 Windows

#### 1. Checking system privileges
Check whether your system has administrative rights. 
* **With Admin Access**: Use Docker (recommended) or Podman.
* **Without Admin Access**: Use TinyRange Engine or a remote Neurodesk instance.

#### 2. Docker Installation
1. Go to the official Docker website: [https://www.docker.com/](https://www.docker.com/)
2. Download **Docker Desktop for Windows**.
3. Install it like a normal application.
4. Launch Docker and ensure it is running (the icon should appear in the system tray).

**Troubleshooting Windows Docker:**
* If you encounter an error when running the `.exe` installer, please refer to this guide: [https://youtu.be/NfSHEVivHtI](https://youtu.be/NfSHEVivHtI)
* If Docker shows an error upon launching, refer to this guide: [https://youtu.be/Ap4ISsD7LJM](https://youtu.be/Ap4ISsD7LJM)

#### 3. Neurodesk Installation
1. Go to [https://neurodesk.org/](https://neurodesk.org/).
2. Click on **Install Locally**.
3. Install the application and launch it.



## Running Neurodesk

Once installed, follow these steps to start your environment:

1. Launch the **Neurodesk App**.
2. Click on **"Start a local Neurodesk server"**.

### What happens in the background?
Neurodesk starts a local server and uses Docker to pull and run the required containers. It then connects you to the environment through your web browser.

### Interacting with the interface
Once the **JupyterLab** interface opens, you have two ways to interact:
* **NeurodeskApp Icon**: Click the icon on the right sidebar to launch a dedicated Neurodesk interface where you can easily launch tools.
* **Module Loading**: Use the left sidebar to load containers as modules. You can interact with these modules via the command line interface.

> [!NOTE]
> The first launch may take some time as the necessary containers and data need to be downloaded.



## Useful Resources

* **Tutorials & Examples**:
    * [SPM Functional Imaging Tutorial](https://github.com/neurodesk/neurodeskedu/blob/main/books/examples/functional_imaging/first_and_second_level_spm.ipynb)
    * [Neurodesk Education Tutorials](https://neurodesk.org/edu/tutorials/functional_imaging/spm.html)
* **Video Guides**:
    * [Neurodesk System Overview (YouTube)](https://www.youtube.com/watch?v=8ToF6zHv-So)