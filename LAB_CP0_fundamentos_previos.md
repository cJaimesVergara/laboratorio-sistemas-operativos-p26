# LAB CP0 — Fundamentos previos: `fork()`, `wait()`, memoria compartida y `pthread`


**Objetivo general:** preparar al alumno para entender sincronización y comunicación entre procesos/hilos en Ubuntu.

**Entrega:** `lab_cp0_02XXXXX.tgz` con carpeta `evidencia/`

---

## Instrucciones generales de evidencias

Todos los archivos de evidencia deben guardarse en `~/lab_cp0/evidencia/` y **deben incluir el ID del alumno** en el nombre.

> **Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).


Al final:

```bash
tar -czf lab_cp0_02XXXXX.tar.gz evidencia
```

---

## Preparación

**Propósito:** crear estructura de trabajo y recordar herramientas básicas.  
**Se espera obtener:** carpeta `~/lab_cp0/evidencia/`.

```bash
mkdir -p ~/lab_cp0/evidencia                    # crea carpeta del laboratorio y evidencias
cd ~/lab_cp0                                    # entra al directorio del laboratorio
pwd                                             # confirma ruta actual
ls -la                                          # lista contenido actual
```

---

# Parte A — `fork()`: un proceso crea otro

## Idea clave
`fork()` crea un **proceso hijo** que es casi una copia del padre.  
Ambos siguen ejecutándose desde la línea siguiente al `fork()`.

## Diagrama conceptual

```text
Proceso padre
   |
   +---- fork() ----> Proceso hijo
```

## Código base: `fork_demo.c`

```bash
cat > fork_demo.c <<'EOF'                        # crea archivo fuente en C
#include <stdio.h>                                // printf
#include <unistd.h>                               // fork, getpid, getppid, sleep
#include <sys/types.h>                            // pid_t

int main(void) {                                  // función principal
    printf("Antes de fork: PID=%d PPID=%d\n",     // muestra identidad antes de crear hijo
           (int)getpid(), (int)getppid());

    pid_t pid = fork();                           // crea proceso hijo

    if (pid == 0) {                               // rama del hijo
        printf("HIJO : PID=%d PPID=%d\n",         // el hijo imprime su PID y PPID
               (int)getpid(), (int)getppid());
        sleep(30);                                 // mantiene vivo al hijo
    } else {                                      // rama del padre
        printf("PADRE: PID=%d HIJO=%d\n",         // el padre conoce el PID del hijo
               (int)getpid(), (int)pid);
        sleep(45);                                 // mantiene vivo al padre
    }

    return 0;                                     // termina el proceso
}
EOF
```

## Compilación y ejecución

```bash
gcc fork_demo.c -o fork_demo                      # compila programa
./fork_demo | tee evidencia/fork_output.txt  # ejecuta y guarda salida
```

## Observación del sistema

```bash
ps -o pid,ppid,stat,cmd | grep fork_demo | tee evidencia/fork_ps.txt   # observa padre e hijo
```

```bash
# El flag -p muestra los PIDs para comparar con los printfs
pstree -p | grep fork_demo | tee evidencia/fork_tree.txt
```

## ¡Ojo!
Identificas que **un programa puede convertirse en dos procesos**.

## Mini bug intencional para explorar
Cambia `sleep(30)`,`sleep(45)` por `sleep(0)` y repite.  
Pregunta: ¿por qué a veces ya no alcanzas a ver ambos procesos con `ps`?

---

# Parte B — `wait()`: el padre espera al hijo

## Idea clave
Sin `wait()`, el padre puede terminar antes que el hijo.  
Con `wait()`, el padre **sincroniza** su fin con el del hijo.

## Diagrama de tiempo

```text
Sin wait():
Padre: termina rápido
Hijo : sigue vivo un poco más

Con wait():
Padre: espera ------------- termina
Hijo : trabaja ---- termina
```

## Código base: `wait_demo.c`

```bash
cat > wait_demo.c <<'EOF'
#include <stdio.h>                                // printf
#include <unistd.h>                               // fork, sleep
#include <sys/wait.h>                             // wait
#include <sys/types.h>                            // pid_t

int main(void) {                                  // función principal
    pid_t pid = fork();                           // crea hijo

    if (pid == 0) {                               // rama hijo
        printf("Hijo trabajando...\n");           // mensaje del hijo
        sleep(30);                                 // simula trabajo
        printf("Hijo termina\n");                 // fin del hijo
    } else {                                      // rama padre
        printf("Padre espera al hijo\n");         // mensaje del padre
        wait(NULL);                               // espera a que termine el hijo
        printf("Padre continua despues de wait\n"); // el padre solo sigue después del hijo
    }

    return 0;                                     // termina
}
EOF
```

## Compilación y ejecución

```bash
gcc wait_demo.c -o wait_demo
./wait_demo | tee evidencia/wait_output.txt
```

```bash
# Observa el estado (STAT) de ambos procesos mientras el hijo aún "trabaja"
ps -o pid,ppid,stat,cmd -C wait_demo | tee evidencia/wait_states.txt

```

## Experimento guiado
Comenta la línea `wait(NULL);`, recompila y compara la salida.  
Guarda una breve conclusión en:

```bash
cat > evidencia/wait_reflexion.txt <<'EOF'
Sin wait():
Con wait():
Conclusión:
EOF
```

---

# Parte C — Memoria compartida básica

## Idea clave
Dos procesos distintos pueden leer/escribir una misma variable si usan memoria compartida.

## Diagrama conceptual

```text
Padre ----\
           > memoria compartida (entero x)
Hijo  ----/
```

## Código base: `shared_mem_demo.c`

```bash
cat > shared_mem_demo.c <<'EOF'
#include <stdio.h>                                // printf
#include <unistd.h>                               // fork, sleep
#include <sys/mman.h>                             // mmap
#include <sys/types.h>                            // pid_t

int main(void) {                                  // función principal
    int *x = mmap(NULL, sizeof(int),              // reserva memoria compartida
                  PROT_READ | PROT_WRITE,         // lectura y escritura
                  MAP_SHARED | MAP_ANONYMOUS,     // compartida y anónima
                  -1, 0);                         // sin archivo

    *x = 10;                                      // valor inicial

    pid_t pid = fork();                           // crea hijo

    if (pid == 0) {
        sleep(5); // Espera para que el alumno vea el valor inicial
        printf("Hijo lee x=%d, ahora lo cambiará...\n", *x);
        *x = 99;
        sleep(30); // Se mantiene vivo para observación
    } else {
        printf("Padre esperando cambio del hijo...\n");
        sleep(40); // Tiempo mayor que el del hijo
        printf("Padre lee cambio: x=%d\n", *x);
    }

    return 0;                                     // termina
}
EOF
```

## Compilación y ejecución

```bash
gcc shared_mem_demo.c -o shared_mem_demo
./shared_mem_demo | tee evidencia/shared_mem.txt
```

## ¡Ojo!
El padre puede ver un valor que el hijo escribió.  
Eso será la base de CP1 y otros labs.

## Mini bug intencional
Cambia `MAP_SHARED` por `MAP_PRIVATE`.  
¿Qué cambia? Guarda respuesta en:

```bash
cat > evidencia/shared_mem_bug.txt <<'EOF'
Con MAP_SHARED:
Con MAP_PRIVATE:
Conclusión:
EOF
```

---

# Parte D — `pthread`: hilos dentro del mismo proceso

## Idea clave
Los hilos comparten memoria de forma natural porque viven dentro del mismo proceso.

## Código base: `pthread_demo.c`

```bash
cat > pthread_demo.c <<'EOF'
#include <stdio.h>                                // printf
#include <pthread.h>                              // pthread_create, pthread_join

int contador = 0;                                 // variable global compartida por los hilos

void* tarea(void* arg) {                          // función que ejecuta cada hilo
    for (int i = 0; i < 100000; i++) {            // repite muchas veces
        contador++;                               // incrementa contador (sin protección)
    }
    printf("Hilo finalizó conteo, durmiendo para observación...\n");
    sleep(60);                                    // hilos vivos en la tabla de procesos
    return NULL;                                  // termina el hilo
}

int main(void) {                                  // función principal
    pthread_t t1, t2;                             // dos hilos

    pthread_create(&t1, NULL, tarea, NULL);       // crea hilo 1
    pthread_create(&t2, NULL, tarea, NULL);       // crea hilo 2

    pthread_join(t1, NULL);                       // espera hilo 1
    pthread_join(t2, NULL);                       // espera hilo 2

    printf("Contador final = %d\n", contador);    // muestra resultado final
    return 0;                                     // termina
}
EOF
```

## Compilación y ejecución

```bash
gcc pthread_demo.c -pthread -o pthread_demo
./pthread_demo | tee evidencia/pthread.txt
```

```bash
# Muestra el PID y el LWP (ID del hilo dentro del sistema)
ps -fL -C pthread_demo | tee evidencia/pthread_lwp.txt
```



## ¡Ojo!
A veces el contador no da exactamente `200000`.  
Eso anticipa una **condición de carrera**.

## Observación adicional

```bash
ps -L -p $$ | tee evidencia/ps_threads.txt   # muestra hilos del shell/proceso actual
```

---

# Cierre y entrega

```bash
cd ~/lab_cp0                                      # asegura estar en el lab
tar -czf lab_cp0_02XXXXX.tar.gz evidencia     # comprime evidencias con ID
ls -lh lab_cp0_02XXXXX.tar.gz                 # confirma archivo generado
```

---
