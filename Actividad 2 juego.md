# JUEGO POO SDL2

# Diagrama UML

## Clases en el Diagrama UML
Entity

Atributos: 

x: int

y: int

width: int

height: int

texture: SDL_Texture*

Métodos:

Update()

Render()

Player (hereda de Entity)

Atributos:

score: int

health: int

fireRate: int (nueva mejora)

damage: int (nueva mejora)

Métodos:

Update()

Shoot()

Enemy (hereda de Entity)

Atributos:

health: int

Métodos:

Update()

Bullet (hereda de Entity)

Atributos:

damage: int

Métodos:

Update()

PowerUp (agregación de Entity)

Atributos:

isActive: int

activate(Player* player): void

Métodos:

PowerUp_Create(int x, int y, SDL_Texture* texture)

PowerUp_Activate(Player* player)

Game

Atributos:

Player* player

Enemy** enemies

Bullet** bullets

PowerUp* powerUp

int enemyCount

int bulletCount

Métodos:

Game_Update()

Game_Reset()

![diagrama_uml_detallado_juego](https://github.com/user-attachments/assets/b75a9397-5c92-4aab-a5e1-594348a29742)

# Informe de Diseño

## Estructuración de Clases:

### Entity: 
Esta clase base encapsula las propiedades y métodos comunes a todas las entidades del juego (jugador, enemigos, proyectiles, y power-ups). Contiene atributos como x, y, width, height y un puntero a una textura de SDL, lo que permite reutilizar el código y simplificar las clases derivadas.

### Player, Enemy y Bullet: 
Estas clases heredan de Entity, lo que permite que compartan funcionalidades comunes. Cada clase tiene sus propios atributos y métodos específicos. Por ejemplo, Player tiene atributos de score, health, fireRate, y damage, que son específicos para su comportamiento y habilidades.

### PowerUp: 
Se creó como una agregación de Entity, permitiendo que tenga su propia lógica de activación y efectos en el jugador. Esto proporciona una separación clara entre el efecto del power-up y la entidad del jugador.
## Herencia y Reutilización de Código:

La herencia se implementó para evitar la duplicación de código y facilitar el mantenimiento. Los métodos comunes, como Update() y Render(), se definen en la clase base Entity y se reutilizan en las clases derivadas. Esto mejora la modularidad del código, ya que permite añadir o modificar comportamientos específicos sin alterar la estructura básica.

## Encapsulación:

Los atributos de las clases se declararon como privados donde corresponde, y se proporcionaron métodos públicos (getters y setters) para acceder y modificar estos atributos. Esto protege los datos internos de las clases y permite un control sobre cómo se accede a ellos.

## Manejo del Bucle del Juego (Game Loop):

La clase Game se encarga del bucle principal del juego, controlando la inicialización, actualización y renderizado de las entidades. Esto mantiene el bucle del juego limpio y modular, facilitando la comprensión del flujo de ejecución del programa.

## Relaciones entre Clases:

La relación entre Game y las entidades (Player, Enemy, Bullet, PowerUp) se modeló como agregación, donde Game tiene referencias a instancias de estas entidades. Esto permite que el juego controle su ciclo de vida sin que las entidades tengan una dependencia fuerte entre sí.

## Modularización del Código:

El código se organizó en archivos separados por módulos o clases (game.c, player.c, enemy.c, bullet.c, powerup.c, y sus respectivos archivos de cabecera). Esta estructura facilita la navegación y el mantenimiento del código, permitiendo que los cambios en una parte del sistema no afecten a otras partes innecesariamente.

## Conclusiones

La implementación de POO en este proyecto ha permitido una organización clara del código y la posibilidad de expandir y mantener el juego en el futuro. Al encapsular los datos, utilizar herencia para reutilizar el código y modularizar la estructura del proyecto, se ha logrado un diseño eficiente. Esto  mejora la calidad del código. EL codigo no funciona, al añadir objetos y cambios y mejoras en el personaje, se me complicó el codigo y tengo un error que no supe solucionarlo

# main.c 
```c
#include "game.h"
#include <SDL.h>

int main(int argc, char* argv[]) {
    Game* game = Game_Create();
    Game_Init(game);

    Uint32 lastTime = SDL_GetTicks(), currentTime;
    while (game->isRunning) {
        currentTime = SDL_GetTicks();
        float deltaTime = (currentTime - lastTime) / 1000.0f;
        lastTime = currentTime;

        Game_HandleEvents(game);
        Game_Update(game, deltaTime);
        Game_Render(game);

        SDL_Delay(16);  // Aproximadamente 60 FPS
    }

    Game_Destroy(game);
    return 0;
}
```
# game.h
```c
#ifndef GAME_H
#define GAME_H

#include "player.h"
#include "enemy.h"
#include "bullet.h"
#include "powerup.h"

typedef struct Game {
    SDL_Window* window;
    SDL_Renderer* renderer;
    Player* player;
    Enemy* enemies[4];
    Bullet* bullets[100];
    PowerUp* powerUp;
    int bulletCount;
    int isRunning;
} Game;

Game* Game_Create();
void Game_Init(Game* game);
void Game_HandleEvents(Game* game);
void Game_Update(Game* game, float deltaTime);
void Game_Render(Game* game);
void Game_Destroy(Game* game);

#endif
```
# game.c
```c
#include "game.h"
#include <SDL.h>
#include <stdio.h>

// Función para cargar texturas
SDL_Texture* LoadTexture(SDL_Renderer* renderer, const char* filePath) {
    SDL_Surface* surface = SDL_LoadBMP(filePath);
    SDL_Texture* texture = SDL_CreateTextureFromSurface(renderer, surface);
    SDL_FreeSurface(surface);
    return texture;
}

// Función para detectar colisiones
int CheckCollision(Entity* a, Entity* b) {
    SDL_Rect rectA = { (int)a->x, (int)a->y, a->width, a->height };
    SDL_Rect rectB = { (int)b->x, (int)b->y, b->width, b->height };

    if (SDL_HasIntersection(&rectA, &rectB)) {
        return 1;  // Hay colisión
    }
    return 0;  // No hay colisión
}

Game* Game_Create() {
    Game* game = (Game*)malloc(sizeof(Game));
    game->window = NULL;
    game->renderer = NULL;
    game->player = NULL;
    game->powerUp = NULL;
    game->bulletCount = 0;
    game->isRunning = 1;
    for (int i = 0; i < 4; ++i) {
        game->enemies[i] = NULL;
    }
    for (int i = 0; i < 100; ++i) {
        game->bullets[i] = NULL;
    }
    return game;
}

void Game_Init(Game* game) {
    SDL_Init(SDL_INIT_VIDEO);

    game->window = SDL_CreateWindow("Juego SDL2", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, 800, 600, 0);
    game->renderer = SDL_CreateRenderer(game->window, -1, SDL_RENDERER_ACCELERATED);

    // Cargar texturas (debes implementar la función de carga de texturas)
    SDL_Texture* playerTexture = LoadTexture(game->renderer, "player.png");
    SDL_Texture* enemyTexture = LoadTexture(game->renderer, "enemy.png");
    SDL_Texture* bulletTexture = LoadTexture(game->renderer, "bullet.png");
    SDL_Texture* powerUpTexture = LoadTexture(game->renderer, "powerup.png");

    // Crear entidades
    game->player = Player_Create(400, 300, playerTexture);
    game->enemies[0] = Enemy_Create(0, 0, enemyTexture);
    game->enemies[1] = Enemy_Create(760, 0, enemyTexture);
    game->enemies[2] = Enemy_Create(0, 560, enemyTexture);
    game->enemies[3] = Enemy_Create(760, 560, enemyTexture);
    game->powerUp = PowerUp_Create(400, 200, powerUpTexture);
}

void Game_HandleEvents(Game* game) {
    SDL_Event event;
    while (SDL_PollEvent(&event)) {
        if (event.type == SDL_QUIT) {
            game->isRunning = 0;
        }
        if (event.type == SDL_MOUSEBUTTONDOWN && event.button.button == SDL_BUTTON_LEFT) {
            Player_Shoot(game->player, game->bullets, &game->bulletCount, LoadTexture(game->renderer, "bullet.png"));
        }
    }
}

void Game_Update(Game* game, float deltaTime) {
    // Actualizar jugador
    game->player->entity.update((Entity*)game->player, deltaTime);

    // Actualizar enemigos
    for (int i = 0; i < 4; ++i) {
        game->enemies[i]->entity.update((Entity*)game->enemies[i], deltaTime, (Entity*)game->player);
    }

    // Actualizar balas
    for (int i = 0; i < game->bulletCount; ++i) {
        game->bullets[i]->entity.update((Entity*)game->bullets[i], deltaTime);
    }

    // Actualizar power-up si está activo
    if (game->powerUp->isActive) {
        game->powerUp->entity.update((Entity*)game->powerUp, deltaTime);

        // Comprobar colisión con el jugador
        if (CheckCollision(&(game->player->entity), &(game->powerUp->entity))) {
            PowerUp_Activate(game->powerUp, game->player);
        }
    }
}

void Game_Render(Game* game) {
    SDL_RenderClear(game->renderer);

    // Renderizar jugador
    game->player->entity.render((Entity*)game->player, game->renderer);

    // Renderizar enemigos
    for (int i = 0; i < 4; ++i) {
        game->enemies[i]->entity.render((Entity*)game->enemies[i], game->renderer);
    }

    // Renderizar balas
    for (int i = 0; i < game->bulletCount; ++i) {
        game->bullets[i]->entity.render((Entity*)game->bullets[i], game->renderer);
    }

    // Renderizar power-up si está activo
    if (game->powerUp->isActive) {
        game->powerUp->entity.render((Entity*)game->powerUp, game->renderer);
    }

    SDL_RenderPresent(game->renderer);
}

void Game_Destroy(Game* game) {
    // Liberar memoria de entidades
    Entity_Destroy((Entity*)game->player);
    for (int i = 0; i < 4; ++i) {
        Entity_Destroy((Entity*)game->enemies[i]);
    }
    for (int i = 0; i < game->bulletCount; ++i) {
        Entity_Destroy((Entity*)game->bullets[i]);
    }
    Entity_Destroy((Entity*)game->powerUp);

    // Destruir SDL
    SDL_DestroyRenderer(game->renderer);
    SDL_DestroyWindow(game->window);
    SDL_Quit();

    free(game);
}
```
# player.h
```c
#ifndef PLAYER_H
#define PLAYER_H

#include "entity.h"
#include "bullet.h"

typedef struct Player {
    Entity entity;
    float fireRate;  // Velocidad de disparo
} Player;

Player* Player_Create(float x, float y, SDL_Texture* texture);
void Player_Update(Player* player, float deltaTime);
void Player_Render(Player* player, SDL_Renderer* renderer);
void Player_Shoot(Player* player, Bullet* bullets[], int* bulletCount, SDL_Texture* bulletTexture);
void Player_Upgrade(Player* player);

#endif
```
# player.c
```c
#include "player.h"
#include <SDL.h>
#include <math.h>

Player* Player_Create(float x, float y, SDL_Texture* texture) {
    Player* player = (Player*)malloc(sizeof(Player));
    player->entity = *Entity_Create(x, y, 50, 50, texture);
    player->fireRate = 1.0f;  // Velocidad normal
    player->entity.update = (void (*)(Entity*, float))Player_Update;
    player->entity.render = (void (*)(Entity*, SDL_Renderer*))Player_Render;
    return player;
}

void Player_Update(Player* player, float deltaTime) {
    const Uint8* state = SDL_GetKeyboardState(NULL);
    if (state[SDL_SCANCODE_W]) player->entity.y -= 200 * deltaTime;
    if (state[SDL_SCANCODE_S]) player->entity.y += 200 * deltaTime;
    if (state[SDL_SCANCODE_A]) player->entity.x -= 200 * deltaTime;
    if (state[SDL_SCANCODE_D]) player->entity.x += 200 * deltaTime;
}

void Player_Render(Player* player, SDL_Renderer* renderer) {
    SDL_Rect rect = { (int)player->entity.x, (int)player->entity.y, player->entity.width, player->entity.height };
    SDL_RenderCopy(renderer, player->entity.texture, NULL, &rect);
}

void Player_Shoot(Player* player, Bullet* bullets[], int* bulletCount, SDL_Texture* bulletTexture) {
    int mouseX, mouseY;
    SDL_GetMouseState(&mouseX, &mouseY);
    float dirX = mouseX - player->entity.x;
    float dirY = mouseY - player->entity.y;
    float magnitude = sqrt(dirX * dirX + dirY * dirY);
    dirX /= magnitude;
    dirY /= magnitude;

    if (*bulletCount < 100) {
        bullets[*bulletCount] = Bullet_Create(player->entity.x, player->entity.y, dirX, dirY, player->fireRate * 300, bulletTexture);
        (*bulletCount)++;
    }
}

void Player_Upgrade(Player* player) {
    player->fireRate = 2.0f;  // Aumentar la velocidad de disparo
}
```
# enemy.h
```c
#ifndef ENEMY_H
#define ENEMY_H

#include "entity.h"

typedef struct Enemy {
    Entity entity;
    int health;
} Enemy;

Enemy* Enemy_Create(float x, float y, SDL_Texture* texture);
void Enemy_Update(Enemy* enemy, float deltaTime, Entity* target);
void Enemy_Render(Enemy* enemy, SDL_Renderer* renderer);
void Enemy_Hit(Enemy* enemy);

#endif
```
# enemy.c
```c
#include "enemy.h"
#include <math.h>
#include <stdlib.h>

Enemy* Enemy_Create(float x, float y, SDL_Texture* texture) {
    Enemy* enemy = (Enemy*)malloc(sizeof(Enemy));
    enemy->entity = *Entity_Create(x, y, 40, 40, texture);
    enemy->health = 3;
    enemy->entity.update = (void (*)(Entity*, float))Enemy_Update;
    enemy->entity.render = (void (*)(Entity*, SDL_Renderer*))Enemy_Render;
    return enemy;
}

void Enemy_Update(Enemy* enemy, float deltaTime, Entity* target) {
    float dirX = target->x - enemy->entity.x;
    float dirY = target->y - enemy->entity.y;
    float magnitude = sqrt(dirX * dirX + dirY * dirY);
    dirX /= magnitude;
    dirY /= magnitude;

    enemy->entity.vx = dirX * 100;
    enemy->entity.vy = dirY * 100;
    enemy->entity.x += enemy->entity.vx * deltaTime;
    enemy->entity.y += enemy->entity.vy * deltaTime;
}

void Enemy_Render(Enemy* enemy, SDL_Renderer* renderer) {
    SDL_Rect rect = { (int)enemy->entity.x, (int)enemy->entity.y, enemy->entity.width, enemy->entity.height };
    SDL_RenderCopy(renderer, enemy->entity.texture, NULL, &rect);
}

void Enemy_Hit(Enemy* enemy) {
    enemy->health--;
}
```
# powerup.h
```c
#ifndef POWERUP_H
#define POWERUP_H

#include "entity.h"
#include "player.h"

typedef struct PowerUp {
    Entity entity;
    int isActive;
} PowerUp;

PowerUp* PowerUp_Create(float x, float y, SDL_Texture* texture);
void PowerUp_Update(PowerUp* powerup, float deltaTime);
void PowerUp_Render(PowerUp* powerup, SDL_Renderer* renderer);
void PowerUp_Activate(PowerUp* powerup, Player* player);

#endif
```
# powerup.c
```c
#include "powerup.h"
#include <stdlib.h>

PowerUp* PowerUp_Create(float x, float y, SDL_Texture* texture) {
    PowerUp* powerup = (PowerUp*)malloc(sizeof(PowerUp));
    powerup->entity = *Entity_Create(x, y, 30, 30, texture);
    powerup->isActive = 1;
    return powerup;
}

void PowerUp_Update(PowerUp* powerup, float deltaTime) {
    // Podrías agregar movimiento si es necesario
}

void PowerUp_Render(PowerUp* powerup, SDL_Renderer* renderer) {
    if (!powerup->isActive) return;

    SDL_Rect rect = { (int)powerup->entity.x, (int)powerup->entity.y, powerup->entity.width, powerup->entity.height };
    SDL_RenderCopy(renderer, powerup->entity.texture, NULL, &rect);
}

void PowerUp_Activate(PowerUp* powerup, Player* player) {
    powerup->isActive = 0;  // Desactiva el power-up tras la recolección
    Player_Upgrade(player);  // Aumenta la velocidad de disparo del jugador
}
```
# entity.h
```c
#ifndef ENTITY_H
#define ENTITY_H

#include <SDL.h>

typedef struct Entity {
    float x, y;
    float width, height;
    float vx, vy;  // Velocidad
    SDL_Texture* texture;
    void (*update)(struct Entity*, float deltaTime);
    void (*render)(struct Entity*, SDL_Renderer*);
} Entity;

Entity* Entity_Create(float x, float y, float width, float height, SDL_Texture* texture);
void Entity_Destroy(Entity* entity);

#endif
```
# entity.c
```c
#include "entity.h"
#include <stdlib.h>

Entity* Entity_Create(float x, float y, float width, float height, SDL_Texture* texture) {
    Entity* entity = (Entity*)malloc(sizeof(Entity));
    entity->x = x;
    entity->y = y;
    entity->width = width;
    entity->height = height;
    entity->vx = 0;
    entity->vy = 0;
    entity->texture = texture;
    entity->update = NULL;
    entity->render = NULL;
    return entity;
}

void Entity_Destroy(Entity* entity) {
    free(entity);
}
```
# bullet.c
```c
#include "bullet.h"
#include <stdlib.h>

Bullet* Bullet_Create(float x, float y, float dirX, float dirY, float speed, SDL_Texture* texture) {
    Bullet* bullet = (Bullet*)malloc(sizeof(Bullet));
    bullet->entity = *Entity_Create(x, y, 10, 10, texture);
    bullet->speed = speed;
    bullet->entity.vx = dirX * bullet->speed;
    bullet->entity.vy = dirY * bullet->speed;
    bullet->isActive = 1;
    bullet->entity.update = (void (*)(Entity*, float))Bullet_Update;
    bullet->entity.render = (void (*)(Entity*, SDL_Renderer*))Bullet_Render;
    return bullet;
}

void Bullet_Update(Bullet* bullet, float deltaTime) {
    if (!bullet->isActive) return;

    bullet->entity.x += bullet->entity.vx * deltaTime;
    bullet->entity.y += bullet->entity.vy * deltaTime;

    if (bullet->entity.x < 0 || bullet->entity.x > 800 || bullet->entity.y < 0 || bullet->entity.y > 600) {
        Bullet_Deactivate(bullet);
    }
}

void Bullet_Render(Bullet* bullet, SDL_Renderer* renderer) {
    if (!bullet->isActive) return;

    SDL_Rect rect = { (int)bullet->entity.x, (int)bullet->entity.y, bullet->entity.width, bullet->entity.height };
    SDL_RenderCopy(renderer, bullet->entity.texture, NULL, &rect);
}

void Bullet_Deactivate(Bullet* bullet) {
    bullet->isActive = 0;
}
```
# bullet.h
```c
#ifndef BULLET_H
#define BULLET_H

#include "entity.h"

typedef struct Bullet {
    Entity entity;
    float speed;
    int isActive;
} Bullet;

Bullet* Bullet_Create(float x, float y, float dirX, float dirY, float speed, SDL_Texture* texture);
void Bullet_Update(Bullet* bullet, float deltaTime);
void Bullet_Render(Bullet* bullet, SDL_Renderer* renderer);
void Bullet_Deactivate(Bullet* bullet);

#endif
```





