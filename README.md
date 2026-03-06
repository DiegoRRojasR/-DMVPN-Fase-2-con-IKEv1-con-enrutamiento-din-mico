# DMVPN Fase 2 con IKEv1 y EIGRP

## Video demostrativo

[![Ver video en YouTube](https://img.youtube.com/vi/1jGJAAF-umY/maxresdefault.jpg)](https://youtu.be/1jGJAAF-umY)

---

## I-) Descripción general
En esta práctica se implementó una VPN DMVPN (Dynamic Multipoint VPN) Fase 2 utilizando IKEv1 para el establecimiento de asociaciones de seguridad IPsec y EIGRP como protocolo de enrutamiento dinámico.

La arquitectura DMVPN permite conectar múltiples sucursales a través de un router central denominado HUB, mientras que los routers de las sucursales funcionan como SPOKES. La tecnología utiliza GRE multipunto combinado con NHRP para permitir el registro dinámico de los spokes en el hub y facilitar la comunicación entre redes remotas a través de un túnel protegido por IPsec.  
En esta topología, el router HUB actúa como punto central de registro NHRP y mantiene la base de datos de los spokes conectados. Los routers SPOKE1 y SPOKE2 se registran dinámicamente en el hub utilizando la red de túnel 10.200.9.0/24

---

## II-)Topología de red

La topología está compuesta por un router HUB, dos routers SPOKE, un router ISP que simula la red de Internet y una red LAN en cada sitio.

Cada router establece conectividad IP hacia el ISP mediante su interfaz WAN, mientras que las redes LAN locales se conectan a través de interfaces internas hacia switches y hosts de prueba.

El túnel DMVPN se establece utilizando una interfaz Tunnel100 en cada router, donde el HUB utiliza el modo GRE multipoint para permitir que múltiples spokes se conecten dinámicamente.

<img width="565" height="719" alt="Screenshot 2026-03-05 234442" src="https://github.com/user-attachments/assets/852c5077-58e6-401e-aaad-31e25b575aeb" />

---

## III-) Direccionamiento IP

El direccionamiento fue definido tomando como referencia la matrícula 20240918, utilizando subredes privadas para las redes LAN y el túnel DMVPN.

<img width="307" height="427" alt="Screenshot 2026-03-05 234501" src="https://github.com/user-attachments/assets/d8081179-60c0-48e2-8cbe-00819027641d" />

---

## IV-) Configuración del túnel DMVPN
El túnel DMVPN se configuró utilizando GRE multipoint junto con NHRP para permitir el registro dinámico de los spokes.

<img width="324" height="294" alt="Screenshot 2026-03-05 234752" src="https://github.com/user-attachments/assets/306038d5-1dec-404a-9f7c-2fb5706cbc0f" />

El HUB actúa como Next Hop Server (NHS), mientras que los spokes utilizan configuraciones de NHRP para mapear la dirección de túnel del hub con su dirección NBMA en la red WAN.

<img width="311" height="180" alt="Screenshot 2026-03-05 235113" src="https://github.com/user-attachments/assets/0be4de0a-1a4f-4187-a03f-de099d70c29e" />

Para asegurar el tráfico del túnel se implementó IPsec utilizando IKEv1, con cifrado AES-256, integridad SHA y autenticación mediante clave precompartida.

<img width="428" height="165" alt="Screenshot 2026-03-05 235156" src="https://github.com/user-attachments/assets/cdd113ca-4215-40d2-9669-a04eb28150d2" />

El tráfico protegido por IPsec corresponde al tráfico GRE generado por la interfaz de túnel, utilizando un IPsec profile aplicado directamente sobre la interfaz Tunnel100.

---

## V-) Enrutamiento dinámico 

Para el intercambio de rutas entre las redes remotas se configuró EIGRP en el sistema autónomo 100.

<img width="426" height="257" alt="Screenshot 2026-03-05 235235" src="https://github.com/user-attachments/assets/6879ed5e-703b-4d4c-a93b-b52eabf2c63c" />

Este protocolo permite anunciar las redes LAN del HUB y de los SPOKES a través del túnel DMVPN, facilitando la comunicación entre sucursales.

<img width="211" height="31" alt="Screenshot 2026-03-05 235304" src="https://github.com/user-attachments/assets/c42febfc-2746-4f45-ad7d-024a9a820836" />

En el router HUB se deshabilitó split horizon y next-hop-self en la interfaz del túnel para permitir que las rutas aprendidas de un spoke puedan ser redistribuidas hacia los demás spokes.

Gracias a esta configuración, cada sucursal aprende automáticamente las redes remotas a través del protocolo de enrutamiento dinámico.

---

## VI-) Verificación del funcionamiento

El correcto funcionamiento de la VPN se verificó mediante diversos comandos de diagnóstico.

El comando show dmvpn permitió confirmar que los routers SPOKE se registraron correctamente en el HUB mediante NHRP.

<img width="429" height="116" alt="Screenshot 2026-03-05 235457" src="https://github.com/user-attachments/assets/03f7ce01-9c8f-4678-aef2-f398e1963874" />

Posteriormente, show crypto isakmp sa mostró el estado QM_IDLE, indicando que la negociación IKEv1 se completó exitosamente.

<img width="435" height="86" alt="Screenshot 2026-03-05 235519" src="https://github.com/user-attachments/assets/141bdaa8-e0ca-4f5e-a515-fc4252511fd8" />

Las estadísticas del cifrado IPsec se verificaron con show crypto ipsec sa, donde los contadores de paquetes encapsulados y decapsulados confirmaron que el tráfico estaba siendo protegido correctamente.

<img width="501" height="109" alt="Screenshot 2026-03-05 235545" src="https://github.com/user-attachments/assets/ecfa3d8d-e023-43ab-b017-d1589a578620" />

El protocolo de enrutamiento dinámico se verificó con show ip eigrp neighbors, donde el HUB estableció vecindad con ambos spokes a través del túnel.

<img width="592" height="94" alt="Screenshot 2026-03-05 235643" src="https://github.com/user-attachments/assets/180020da-1801-400c-b081-c91d0a30dab7" />

Finalmente, la tabla de rutas mostró las redes LAN remotas aprendidas dinámicamente a través de EIGRP.

<img width="516" height="29" alt="Screenshot 2026-03-05 235708" src="https://github.com/user-attachments/assets/a4e01a62-90c3-43bd-91ca-dc150cb9ef3c" />

---

## VII-) Pruebas de conectividad

Para validar la conectividad entre sitios se realizaron pruebas de ping entre los hosts de cada red LAN.

<img width="418" height="102" alt="Screenshot 2026-03-05 235834" src="https://github.com/user-attachments/assets/0ef38592-db27-490b-9225-83934c361a80" />

Desde PC1 en SPOKE1 se realizó un ping hacia PC2 en SPOKE2, obteniendo respuesta exitosa a través del túnel DMVPN.

<img width="430" height="132" alt="Screenshot 2026-03-05 235907" src="https://github.com/user-attachments/assets/54e682c3-bfd2-426d-ad77-b219c7382eee" />

También se comprobó conectividad desde ambas sucursales hacia la red del HUB.  
Estas pruebas confirmaron que el túnel DMVPN, la protección IPsec y el enrutamiento dinámico estaban funcionando correctamente.

---

## VIII-) Conclusión

En esta práctica se implementó exitosamente una infraestructura DMVPN Fase 2 utilizando IKEv1 y EIGRP.

El router HUB permitió el registro dinámico de los spokes mediante NHRP, mientras que IPsec proporcionó confidencialidad e integridad al tráfico del túnel.

El uso de EIGRP permitió que las redes LAN de cada sucursal fueran aprendidas dinámicamente, habilitando comunicación completa entre los distintos sitios de la topología.
