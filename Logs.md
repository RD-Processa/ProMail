# Logs

ProMail utiliza el módulo [NLog](https://github.com/RD-Processa/NLog) el cual permite visualizar y registrar las etapas de ejecución de los módulos registrados, con información útil para el seguimiento de incidencias que se puedan presentar.

## Visor de eventos

Use el comando **Get-Source** para visualizar los módulos registrados en NLog.

```powershell
PS X> Get-Source

ID Name          Descripción
-- ------        -----------
1  PluginManager Descripción del origen PluginManager
2  ProMail       Descripción del origen ProMail
```

Para visualizar los últimos 10 eventos registrados en NLog en el modulo ProMail puede utilizar el siguiente ejemplo:

```powershell
Get-Source -Name 'ProMail' | Get-Log -Last 10
```

## Revisión y seguimiento de eventos registrados

Para comprender las etapas de ejecucion de ProMail se debe las siguientes consideraciones:

1. Establecer inicialmente si el servicio de Windows denominado **Processa.Services.ProMail** se encuentra activo o detenido
2. Verificar en el visor de eventos la última fecha de registro teniendo presente que aunque algún proceso no este en ejecución, ProMail registra el monitoreo de las cuentas registradas cada hora, como se puede evidenciar en la siguiente imagen.

<h2 align="center"><img src="Setup/Monitoreo cuentas.png" /></h2>
4. Cuando ProMail encuentra un nuevo correo,se inicial el proceso de validación de la regla asociada a la cuenta (remitente, asunto y adjunto), si la validación no cumple con las condiciones establecidas se genera un correo de respuesta, señalando la existencia de un error en su procesamiento, como se puede evidenciar en la siguiente imagen.
<h2 align="center"><img src="Setup/Monitoreo Reglas.png" /></h2>
5. Cuando el correo cumple con las condiciones establecidas en la regla, se inicia el proceso de ejecución del plugin, en ese momento las funcionalidades de ProMail se encuentran a la espera de la respuesta que el plugin genere, la respuesta podría ser de ERROR o SUCCESS. 
<h2 align="center"><img src="Setup/Monitoreo Plugin.png" /></h2>

Luego se realiza la notificación de la respuesta por correo electrónico (se realiza 3 intentos, si no hay respuesta se remite correo a una cuenta de correo alterna), como se puede evidenciar en la siguiente imagen.
<h2 align="center"><img src="Setup/Correo Alterno.png" /></h2>

6. Por último, ProMail realiza el registro de la finalización del proceso y nuevamente inicia el proceso de monitoreo de las cuentas.
<h2 align="center"><img src="Setup/Monitoreo Notificacion.png" /></h2>

Se debe tener en cuenta que ProMail tiene la capacidad de ejecutar procesos en simultáneo los cuales se pueden evidenciar en el log, en  la propiedad **Proceso** de cada registro o traza se establece un número de proceso o hilo que lo identica de los demás procesos en ejecución. En el siguiente ejemplo se puede visualizar el registro de varios procesos, uno con valor 5716 el cual ha finalizado y la habilitación de un nuevo proceso con el valor 6788, sin embargo, se podría evidenciar las trazas de diferentes etapas de ejecución.

```powershell
Fecha             : 2018-11-28 09:27:34 a.m.
Nivel             : TRACE
PC                : PDDB001
Proceso           : powershell - 5716
Identidad usuario : PROCESSA\jleyton
Mensaje           : Finalizo el proceso del plugin
Excepción         :

Fecha             : 2018-11-28 10:21:32 a.m.
Nivel             : TRACE
PC                : PDDB001
Proceso           : powershell - 6788
Identidad usuario : PROCESSA\jleyton
Mensaje           : Monitoreando cuenta(s)...
```
En conclusión, para establecer el origen de la incidencia o error se debe tener presente la etapa de ejecucion del proceso.
