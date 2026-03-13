# LAB CP0 — Fundamentos previos: `fork()`, `wait()`, memoria compartida y `pthread`


**Objetivo general:** preparar al alumno para entender sincronización y comunicación entre procesos/hilos en Ubuntu.

**Entrega:** `lab_cp0_02XXXXX.tgz` con carpeta `evidencia/`

---

## Instrucciones generales de evidencias

Todos los archivos de evidencia deben guardarse en `~/lab_cp0/evidencia/` y **deben incluir el ID del alumno** en el nombre.

> **Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).


Al final:


```bash
tar -czf lab_cp0_02XXXXX.tar.gz evidencia                                        # lista contenido actual
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

# Explicación del código con `pthread`

## Fragmento de código

```c
pthread_t t1, t2;                             // dos hilos

pthread_create(&t1, NULL, tarea, NULL);       // crea hilo 1
pthread_create(&t2, NULL, tarea, NULL);       // crea hilo 2

pthread_join(t1, NULL);                       // espera hilo 1
pthread_join(t2, NULL);                       // espera hilo 2
```

---

## ¿Qué hace este código?

Este fragmento crea **dos hilos de ejecución** (`t1` y `t2`) dentro del mismo proceso.  
Ambos hilos ejecutan la misma función llamada `tarea`.

Después de crearlos, el programa principal **se detiene a esperar** a que ambos terminen antes de continuar o finalizar.

En otras palabras:

1. Se declaran dos variables para identificar dos hilos.
2. Se crea el primer hilo.
3. Se crea el segundo hilo.
4. El hilo principal espera a que termine el primero.
5. El hilo principal espera a que termine el segundo.

---

## Explicación línea por línea

### 1. Declaración de los hilos

```c
pthread_t t1, t2;
```

- `pthread_t` es el tipo de dato que utiliza la biblioteca POSIX Threads para representar un hilo.
- `t1` y `t2` son variables que almacenarán el identificador de cada hilo creado.
- Aquí **todavía no existen los hilos**; solo se están declarando las variables que los referenciarán.

---

### 2. Creación del primer hilo

```c
pthread_create(&t1, NULL, tarea, NULL);
```

Esta instrucción crea un nuevo hilo.

#### Parámetros de `pthread_create`

La función tiene esta forma general:

```c
pthread_create(&hilo, atributos, funcion, argumento);
```

En este caso:

- `&t1`  
  Guarda en `t1` el identificador del hilo recién creado.

- `NULL`  
  Indica que el hilo se crea con los **atributos por defecto**.

- `tarea`  
  Es la función que ejecutará el nuevo hilo.

- `NULL`  
  Es el argumento que se le pasa a la función `tarea`.  
  En este caso no se envía información adicional.

#### Resultado

Al ejecutarse esta línea:

- nace un nuevo hilo;
- el nuevo hilo comienza a ejecutar la función `tarea`;
- el hilo principal **no se detiene necesariamente aquí**: puede seguir ejecutando la siguiente instrucción mientras el nuevo hilo corre en paralelo.

---

### 3. Creación del segundo hilo

```c
pthread_create(&t2, NULL, tarea, NULL);
```

Hace exactamente lo mismo que la línea anterior, pero ahora crea un segundo hilo y guarda su identificador en `t2`.

#### Importante

Ahora existen **dos hilos concurrentes** ejecutando la misma función `tarea`:

- hilo `t1`
- hilo `t2`

Además, también sigue existiendo el **hilo principal** (`main`).

Eso significa que, en total, el proceso puede tener **tres flujos de ejecución activos**:

- el hilo principal,
- el hilo 1,
- el hilo 2.

---

## ¿Los hilos se ejecutan en orden?

No necesariamente.

Aunque el código crea primero `t1` y luego `t2`, **el sistema operativo decide cuándo corre cada hilo**.  
Por eso:

- `t1` podría avanzar más rápido,
- `t2` podría imprimir primero,
- ambos podrían intercalarse.

Si la función `tarea` imprime texto en pantalla, es normal que las salidas aparezcan mezcladas.

---

## 4. Esperar a que termine el primer hilo

```c
pthread_join(t1, NULL);
```

`pthread_join` hace que el hilo que llama a esta función (normalmente el hilo principal) **espere** hasta que el hilo indicado termine.

En este caso:

- el hilo principal se bloquea
- hasta que `t1` finalice su ejecución

El segundo parámetro es `NULL`, lo que significa que **no se recupera el valor de retorno** del hilo.

---

## 5. Esperar a que termine el segundo hilo

```c
pthread_join(t2, NULL);
```

Hace lo mismo, pero ahora espera a que termine `t2`.

Con esto se garantiza que el programa principal **no termine antes que los hilos**.

---

## Idea general del flujo

El comportamiento global es este:

```text
main inicia
   |
   |-- crea t1  ------> t1 ejecuta tarea
   |
   |-- crea t2  ------> t2 ejecuta tarea
   |
   |-- espera a t1
   |
   |-- espera a t2
   |
main continúa / termina
```

---

## ¿Para qué sirve `pthread_join`?

Sirve para sincronizar el final de los hilos.

Si no se usara `pthread_join`, puede ocurrir que:

- el `main` termine muy rápido,
- el proceso finalice,
- y los hilos no alcancen a completar su trabajo.

Por eso, `pthread_join` es una forma de decir:

> “No continúes hasta que este hilo haya terminado”.

---

## ¿Qué requisito debe cumplir `tarea`?

La función que ejecuta un hilo con `pthread_create` debe tener esta forma:

```c
void *tarea(void *arg);
```

Esto significa:

- devuelve un `void *`
- recibe un argumento de tipo `void *`

Ejemplo mínimo:

```c
void *tarea(void *arg) {
    printf("Hola desde un hilo\n");
    return NULL;
}
```

---

## ¿Qué conceptos pueden observar los alumnos con este código?

Este fragmento permite enseñar varios conceptos importantes:

### 1. Creación de hilos
Cómo un proceso puede tener varios flujos de ejecución al mismo tiempo.

### 2. Concurrencia
Los hilos pueden avanzar “a la vez” o intercalarse.

### 3. No determinismo
El orden exacto de ejecución puede cambiar en cada corrida.

### 4. Sincronización básica
`pthread_join` permite esperar a que un hilo termine.

---

## Lo que este código **sí hace**

- crea dos hilos;
- ambos ejecutan la misma función;
- el hilo principal espera a que ambos terminen.

---

## Lo que este código **no hace**

Este fragmento **no protege variables compartidas**.

Si la función `tarea` modifica una variable global compartida, puede haber condiciones de carrera si no se usan mecanismos adicionales, como:

- mutex,
- semáforos,
- variables de condición.

Es decir, `pthread_join` **espera el final**, pero **no evita conflictos durante la ejecución concurrente**.

---

## Explicación breve para poner en el laboratorio

Puedes usar este texto directamente:

> Este código crea dos hilos (`t1` y `t2`) usando la biblioteca `pthread`. Ambos hilos ejecutan la misma función llamada `tarea`. Después de crearlos, el hilo principal usa `pthread_join` para esperar a que ambos terminen antes de continuar. Esto permite observar concurrencia, ya que los dos hilos pueden ejecutarse de forma intercalada, y también muestra una forma básica de sincronización al final de su ejecución.

---

## Resumen final

En resumen, este código:

- declara dos identificadores de hilo;
- crea dos hilos que ejecutan la función `tarea`;
- deja que ambos trabajen concurrentemente;
- espera a que ambos finalicen antes de que el programa continúe o termine.

Es un ejemplo básico y muy útil para introducir a los alumnos en el uso de hilos POSIX en C.




---
