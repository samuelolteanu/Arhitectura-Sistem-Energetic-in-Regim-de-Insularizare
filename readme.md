
```text
================================================================================================
   ARHITECTURA SISTEM ENERGETIC CU FUNCȚIONARE ÎN REGIM DE INSULARIZARE
===============================================================================================
[ REȚEA NAȚIONALĂ 25A]                 [ PANOURI VEST: 5.2 kW ]         [ PANOURI SUD: 5.5 kW ]
+___|            +--->[ CONSUMATORI ]      |                                |
|                |                         |                                |
+__./.___->[ INVERTOR PRINCIPAL ] <--------+    [ INVERTOR SECUNDAR ]<------+
    ^  (Rețea Conectat   Deconectat)             Oprit         Pornit-->[ CONSUMATORI COMUTAȚI ]
    |            x            ^                    x             ^
  NC(23)         |    sau     |                    |     sau     |
    |            v            v                    v             v
    |           ===================================================
    |           | BUS COMUN 48V DC (transfer în orice direcție)   |<---+
    |           ===================================================    |
    |                                                                  |
    +>Modul Utility (UTI) NU permite amestec de curent AC.             |
      Invertorul face bypass Rețea catre consumatorii casei            v
      iar partea solara incarca bateria sau alimenteaza           [ BATERIE 15 kWh LiFePO4 ]
      invertorul secundar pe alt traseu intern, in paralel.     (BMS PACE, Victron SmartShunt)
================================================================================================
                       DIAGRAMA DE COMUTARE (TRANSFER AC)
================================================================================================
[ INVERTOR PRINCIPAL ] <------[Contactor(23)] <---[ REȚEA NAȚIONALĂ ]--------->[ PRIZĂ 21 ]
        |                            ^                  |
        v                            |                  v
        |                        [CTS (10)]  [Releu Protecție (19)]     [INVERTOR SECUNDAR] 
        |                                               |                        |
        |      [Prioritate B]←--------------------------+                        |
        |      [ TS: BYPASS MANUAL (24) ]                                        |
        +-----→[Prioritate A]        |                                           |
        +----------------------------+                                           |
        |                            |                                           |
        v                            v             [Prioritate A]←---------------+
    [ RCB 12 ]             [ Releu protectie 14 ]  [ ATS: Suplimentare Putere AC (13)]
        |                            |        +--->[Prioritate B]    | 
        |                            v        |                      v 
        |                       [ RCBO 20 ]---+            [Releu Protecție 11]
        |                            |                               |
        v                            v                               v
[FĂRĂ Releu Protecție AC]  [Restul consumatorilor]           [ MCB 4 și MCB 8]
        |                                                            |
        v                                                            v         
 [MCB 7 și MCB 10]--> - Iluminat                             - AC-uri Dormitoare și Baie
                      - Server HA, CTS                       - Hidrofor
                      - Centrală termică                     - Boiler Extern
================================================================================================         
                             LOGICĂ ȘI CONTROL
================================================================================================
 [ SERVER HA (Proxmox VE) ] +-----> - Calculează Prognoza Iradiere Solara(in kWh) ptr. ziua urmat.
     (cu UPS propriu)       |         și setează Prag SOC pentru Bypass Retea Dinamic. Nu descarcă
       ^         ^          |         bateria dacă mâine nu se va recupera 
       |         |          |         ce s-ar fi descărcat în mod normal.
       |         |          +-----> - UPS implicit pentru întreaga casă: 
       |         |                    - Lipsa severă de iluminare solară coincide 
       |         |                      des cu o pană de curent. Bateria nu e descarcata din 
       |         |                      raționamentul de mai sus.
       |         |                    - Se mentine un prag critic (legat de pragul dinamic) unde 
       |         |         WAN         se va "completa" cu incarcare din Retea.
       |         +----------x?----> - Conexiune nefuncțională la Internet 
       |                              NU duce la funcționare statică (autonomie prognoză 7 zile).
       x?-------------------+-----> - Funcționare statică, în cazul în care serverul
       |                    |         e nefuncțional (Praguri configurabile din webserverul local)
       |                    +-----> - CTS nealimentat/nefuncțional: invertorul degradeaza
       |                              la funcționare „din fabrică”, o data schimbat pe
       |                              mod SOLAR din ecran sau HA. Consum aferent rețea 30W 24h/7.
    [ CONTROLER TRANSFER și SINCRONIZARE (CTS) ]-----------------------------------+
       ^               |                                                           |
       |               v                                                           v
  Smartshunt  Contactor Rețea(23)    Buton Comutator Invertor Suplimentar <--- Releu izolat
================================================================================================
                                                                                 13.08.2026 v1.2
```
