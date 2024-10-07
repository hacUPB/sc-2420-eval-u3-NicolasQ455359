# 1. Abstracción
La abstracción es el principio que permite ocultar los detalles complejos de una implementación y exponer solo lo necesario. 

La estructura Personaje actúa como una clase base que define los atributos y métodos comunes (como nombre, vida, nivel y mostrar_estado). Esto permite que las subclases (Guerrero y Mago) utilicen estos atributos y métodos sin necesidad de implementar sus detalles.

# 2. Herencia
La herencia permite crear nuevas clases basadas en clases existentes, heredando sus atributos y métodos. 

La clase Guerrero y Mago heredan de la clase Personaje. Esto significa que Guerrero y Mago tienen acceso a todos los atributos y métodos de Personaje, como nombre, vida, nivel y mostrar_estado.
La herencia también permite que estas subclases añadan sus propios atributos y métodos. Por ejemplo, Guerrero tiene un atributo adicional fuerza, mientras que Mago tiene mana.

# 3. Encapsulamiento
El encapsulamiento es el principio que restringe el acceso a algunos de los componentes de un objeto, lo que ayuda a proteger el estado interno del objeto. 

# 4. Polimorfismo
El polimorfismo permite que una sola función se comporte de diferentes maneras según el objeto que la invoque. 

#CODIGO: 

## main.c
```c
#include "Personaje.h"
#include "Guerrero.h"
#include "Mago.h"
#include <stdio.h>
#include "Arma.h"

int main(void)
{
    Personaje* personajes[2];

    Guerrero* guerrero = Guerrero_crear("Arthas", 100, 10, 80);
    Mago* mago = Mago_crear("Jaina", 80, 12, 120);

    personajes[0] = (Personaje*)guerrero;
    personajes[1] = (Personaje*)mago;

    for (int i = 0; i < 2; ++i) {
        personajes[i]->mostrar_estado(personajes[i]);
    }

    Guerrero_destruir(guerrero);
    Mago_destruir(mago);

    return 0;

	arma* arco = Arma_crear("Arco", 30, 1);

	arco-> mostrar_info(arco);

	Arma_destruir(arco);
		
}
```
## Mago.c
```c
#include "mago.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void mostrar_estado_mago(const Personaje* this) {
    const Mago* mago = (const Mago*)this; // Hacemos un cast a Mago
    printf("Mago: %s | Vida: %d | Nivel: %d | Mana: %d\n", mago->base.nombre, mago->base.vida, mago->base.nivel, mago->mana);
}

Mago* Mago_crear(const char* nombre, int vida, int nivel, int mana) {
    Mago* nuevo_mago = (Mago*)malloc(sizeof(Mago));
    if (!nuevo_mago) return NULL;
    strcpy_s(nuevo_mago->base.nombre, 30, nombre);
    nuevo_mago->base.vida = vida;
    nuevo_mago->base.nivel = nivel;
    nuevo_mago->mana = mana;
    nuevo_mago->base.mostrar_estado = mostrar_estado_mago;
    return nuevo_mago;
}

void Mago_destruir(Mago* this) {
    if (this) {
        free(this);
    }
}
```
## Mago.h
```c
#ifndef MAGO_H
#define MAGO_H

#include "Personaje.h"

typedef struct Mago {
    Personaje base; // Herencia de Personaje
    int mana;
} Mago;

Mago* Mago_crear(const char* nombre, int vida, int nivel, int mana);
void Mago_destruir(Mago* this);

#endif // MAGO_H
#pragma once
```
## Personaje.c
```c
#include "Personaje.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void mostrar_estado_impl(const Personaje* this) {
    printf("Personaje: %s | Vida: %d | Nivel: %d\\n", this->nombre, this->vida, this->nivel);
}

Personaje* Personaje_crear(const char* nombre, int vida, int nivel) {
    Personaje* nuevo_personaje = (Personaje*)malloc(sizeof(Personaje));
    if (!nuevo_personaje) return NULL;
    //nuevo_personaje->nombre = strdup(nombre);
    strcpy_s(nuevo_personaje->nombre, 30, nombre);
    nuevo_personaje->vida = vida;
    nuevo_personaje->nivel = nivel;
    nuevo_personaje->mostrar_estado = mostrar_estado_impl;
    return nuevo_personaje;
}

char* get_name(Personaje* this) {
    return this->nombre;
}

void Personaje_destruir(Personaje* this) {
    if (this) {
        //free(this->nombre);
        free(this);
    }
}
```
## Personaje.h
```c
#ifndef PERSONAJE_H
#define PERSONAJE_H

typedef struct Personaje {
    char nombre[30];
    int vida;
    int nivel;
    void (*mostrar_estado)(const struct Personaje* this);
} Personaje;

Personaje* Personaje_crear(const char* nombre, int vida, int nivel);
void Personaje_destruir(Personaje* this);
char* get_name(Personaje* this);


#endif // PERSONAJE_H
```
## Arma.c
```c
#include "arma.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void mostrar_info_arma(arma* this) {
    if (this != NULL) {
        printf("Arma: %s\n", this->nombre);
        printf("Dano: %d\n", this->dano);
        printf("Alcance: %d\n", this->alcance);
    }
}

arma* Arma_crear(const char* nombre, int dano, int alcance) {
    arma* nueva_arma = (arma*)malloc(sizeof(arma));
    if (!nueva_arma) return NULL;
    strcpy_s(nueva_arma->nombre, 30, nombre);
    nueva_arma->dano = dano;
    nueva_arma->alcance = alcance;
    nueva_arma->mostrar_info = mostrar_info_arma;
    return nueva_arma;
}

void Arma_destruir(arma* this) {
    if (this) {
        //free(this->nombre);
        free(this);
    }
}
```
## Arma.h
```c
#ifndef ARMA_H
#define ARMA_H

typedef struct arma {
    char nombre[30];
    int dano;
    int alcance;
    void (*mostrar_info)(struct arma* this);
} arma;

arma* Arma_crear(const char* nombre, int dano, int alcance);

void Arma_destruir(arma* arma);


#endif // ARMA_H
```
## Guerrero.c
```c
#include "Guerrero.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void mostrar_estado_impl(const Personaje* this) {
    Guerrero* guerrero = (Guerrero*)this;
    printf("Guerrero: %s | Vida: %d | Nivel: %d | Fuerza: %d\\n", this->nombre, this->vida, this->nivel, guerrero->fuerza);
}

Guerrero* Guerrero_crear(const char* nombre, int vida, int nivel, int fuerza) {
    Guerrero* nuevo_guerrero = (Guerrero*)malloc(sizeof(Guerrero));
    if (!nuevo_guerrero) return NULL;
    //nuevo_guerrero->base.nombre = strdup(nombre);
    strcpy_s(nuevo_guerrero->base.nombre, 30, nombre);
    nuevo_guerrero->base.vida = vida;
    nuevo_guerrero->base.nivel = nivel;
    nuevo_guerrero->fuerza = fuerza;
    nuevo_guerrero->base.mostrar_estado = mostrar_estado_impl;
    return nuevo_guerrero;
}

void Guerrero_destruir(Guerrero* this) {
    if (this) {
        //free(this->base.nombre);
        free(this);
    }
}
```
## Guerrero.h
```c
#ifndef GUERRERO_H
#define GUERRERO_H

#include "Personaje.h"

typedef struct Guerrero {
    Personaje base;
    int fuerza;
} Guerrero;

Guerrero* Guerrero_crear(const char* nombre, int vida, int nivel, int fuerza);
void Guerrero_destruir(Guerrero* this);

#endif // GUERRERO_H
#pragma once
```



