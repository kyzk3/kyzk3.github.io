---
layout: single
title: Montaje del lab eJPT
excerpt: "Creación de un laboratorio inspirado en el eJPT, compuesto por dos subredes y seis máquinas virtuales (dos Windows y cuatro Linux), diseñado para cubrir todos los contenidos esenciales de la certificación."
date: 2025-11-26
classes: wide
header:
  teaser: /assets/images/INE/ejpt-certification.svg
  teaser_home_page: true
  icon: /assets/images/INE/INE_Logo.png 
categories:
  - Certificaciones
  - Diseño de Entorno
tags:
  - eJPTv2
  - Pentesting Básico
  - Pivoting
  - Laboratorio de Pentesting
  - Redes y Subredes
---

<p align="center">
<img src="/assets/images/eJPT/Arquitectura.png">
</p>

## Descripcion
En el examen de la eJPT se proporciona una máquina Kali Linux accesible desde una interfaz web segura. Esta máquina incluye todas las herramientas necesarias para las tareas de reconocimiento, enumeración y explotación, pero se mantiene totalmente aislada y sin acceso a Internet, obligando al candidato a trabajar únicamente con los recursos del entorno.

Esta simulación recrea un entorno similar al del examen, permitiendo practicar con las herramientas preinstaladas y reforzar las habilidades requeridas. El enfoque principal fue desarrollar y ejercitar técnicas de pivoting utilizando Metasploit dentro de un laboratorio compuesto por seis máquinas virtuales.

## Herramientas recomendadas para la eJPT

- nmap 
- Dirbuster 
- Nikto 
- WPScan 
- CrackMapExec 
- The Metasploit Framework
- Searchsploit 
- Hydra 

---

# Plataformas y Máquinas Utilizadas 

- `Linux`: **Vulnhub**
- `Windows`: **HackMyVM**
- `Software de Virtualización`: **VMware Workstation Pro**

- **Symfonos**: https://www.vulnhub.com/entry/symfonos-1,322/
- **Quoted**: https://hackmyvm.eu/machines/machine.php?vm=quoted
- **Ica 1**: https://www.vulnhub.com/entry/ica-1,748/
- **Venom 1**: https://www.vulnhub.com/entry/venom-1,701/
- **Runas**: https://hackmyvm.eu/machines/machine.php?vm=Runas
- **Durian 1**: https://www.vulnhub.com/entry/durian-1,553/

--- 

# Configuración del Entorno

## Descargar todas las máquinas virtuales necesarias.

![](/assets/images/eJPT/Montaje/All_Machines.png)

## Creacion de la segunda interfaz de red.

![](/assets/images/eJPT/Montaje/Edit_mode.png) 

![](/assets/images/eJPT/Montaje/Administrator_Settings.png)  

![](/assets/images/eJPT/Montaje/Add_Network.png) 

![](/assets/images/eJPT/Montaje/VMnet2.png)

Agrega la nueva red VMnet2 con la IP 10.10.0.0/24, luego haz clic en “Aplicar” y finalmente en “OK”.

## Configuración de Red de las Máquinas 

- Symfonos 1 

![](/assets/images/eJPT/Montaje/Symfonos_1_Settings.png)

![](/assets/images/eJPT/Montaje/Symfonos_1_Conf.png)

- Quoted - Ica 1 

Primera interfaz de red:

![](/assets/images/eJPT/Montaje/Quoted_red_1.png)

Luego en el apartado de **Add** agregamos otro adaptador de red.

Segunda interfaz de red:

![](/assets/images/eJPT/Montaje/Quoted_red_2.png) 

- Venom 1 – Runas – Durian 1 

![](/assets/images/eJPT/Montaje/Durian_1_Red.png)

