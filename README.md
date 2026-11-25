# Wall-Climb 2D Platformer (KAPLAY)

A browser-based 2D platformer prototype built with **KAPLAY** (JavaScript) that teaches wall-sliding and wall-climbing through level layout, color contrast, and physics—with zero on-screen tutorial text.

## Controls
* **Move Left / Right:** `A` / `D` or `Left` / `Right` Arrow keys
* **Jump / Wall-Climb:** `Space`, `W`, or `Up` Arrow key

## How to Run
1. **Online (Recommended):** Open the shared KAPLAYGROUND / hosted web link in Google Chrome, click inside the game canvas to focus, and play immediately.
2. **Locally / In KAPLAYGROUND:**
   * Open [KAPLAYGROUND](https://play.kaplayjs.com/), create a new example, paste the contents of `main.js`, and press **Run** (`Ctrl + S`).
   * Alternatively, serve the project folder with any local static server (e.g., VS Code **Live Server** or `npx serve`) and open `index.html` in Chrome.

## Where to Edit the Level & Physics
All tunable settings live at the top of `main.js`:

* **Physics Constants (Lines 4–11):** Adjust movement, gravity, wall-slide friction, jump height, or enemy speed directly:
  ```javascript
  const SPEED = 220;
  const JUMP_FORCE = 520;
  const GRAVITY = 1200;
  const WALL_SLIDE_SPEED = 15;
  const WALL_JUMP_FORCE = 500;
  const ENEMY_SPEED = 60;


## CODE

     // ==========================================
     // 1. CONSTANTES DE FÍSICAS Y CONFIGURACIÓN
       // ==========================================
    const SPEED = 220;
    const JUMP_FORCE = 520;
    const GRAVITY = 1200;
    const WALL_SLIDE_SPEED = 15;   // Caída lenta para agarre firme en pared
    const WALL_JUMP_FORCE = 500;   // Impulso hacia arriba al trepar
    const WALL_COYOTE_TIME = 0.16; // Margen de tiempo para saltar tras soltar el muro
    const ENEMY_SPEED = 60;        // Velocidad de patrulla del enemigo
    const TILE_SIZE = 32;

     // ==========================================
     // 2. INICIALIZACIÓN
    // ==========================================
    kaplay({
    width: 640,
    height: 360,
    letterbox: true,
    background: [20, 24, 34],
    });

    setGravity(GRAVITY);

    // ==========================================
    // 3. MAPA DE TEXTO (DATA DESACOPLADA)
    // '@' = Inicio | '#' = Suelo/Muro | '^' = Pinchos
    // 'E' = Enemigo patrulla | '*' = Moneda | '>' = Meta
    // ==========================================
    const LEVEL_MAP = [
    "                                                                               ",
    "                                                                   *            ",
    "                                          #                  #######             ",
    "                     #                    #                                      ",
    "                     #            #       #                   #                          ",
    "                     #       #    #       #                 #            #           ",
    "                     #  #    #    #   *  E#               #                               ",
    "                     #  #    #    #  ######             #                                   ",
    "            #        #  #    #    #       #           #                       >        ",
    "     @      #        # *#    #    #       #   E    #            E*       ##############",
    "#########################    # *  #      ###########         ##############",
    "############  ###########^^^##### #^^^^^ ###########                     ##############",
    ];
 
    const LEVEL_CONFIG = {
    tileWidth: TILE_SIZE,
    tileHeight: TILE_SIZE,
    tiles: {
        "#": () => [
            rect(TILE_SIZE, TILE_SIZE),
            color(70, 80, 100),
            area(),
            body({ isStatic: true }),
            anchor("botleft"),
            "ground",
        ],
        "^": () => [
            polygon([vec2(0, 0), vec2(TILE_SIZE / 2, -TILE_SIZE * 0.8), vec2(TILE_SIZE, 0)]),
            color(240, 70, 70),
            area(),
            anchor("botleft"),
            "hazard",
        ],
        ">": () => [
            rect(TILE_SIZE * 0.6, TILE_SIZE * 1.5),
            color(80, 230, 120),
            area(),
            anchor("botleft"),
            "goal",
        ],
                },
       };

  
    scene("game", () => {
         addLevel(LEVEL_MAP, LEVEL_CONFIG);

    let playerSpawn = vec2(64, 200);
    const enemyPositions = [];
    const coinPositions = [];

    // Extraer jugador, enemigos y monedas desde el mapa de texto
    LEVEL_MAP.forEach((row, y) => {
        for (let x = 0; x < row.length; x++) {
            const char = row[x];
            const worldPos = vec2(x * TILE_SIZE, y * TILE_SIZE);

            if (char === "@") {
                playerSpawn = worldPos;
            } else if (char === "E") {
                enemyPositions.push(worldPos);
            } else if (char === "*") {
                coinPositions.push(worldPos);
            }
        }
    });

    // Crear monedas como entidades con forma redondeada y estado
    coinPositions.forEach((coinPos) => {
        add([
            rect(20, 20, { radius: 10 }), // Forma circular con hitbox cuadrada exacta
            color(255, 215, 0),
            pos(coinPos.x + TILE_SIZE / 2, coinPos.y - TILE_SIZE / 2),
            area(),
            anchor("center"),
            "coin",
            { collected: false },
        ]);
    });

    // Crear enemigos
    enemyPositions.forEach((spawnPos) => {
        add([
            rect(26, 26),
            color(235, 75, 45),
            pos(spawnPos),
            area(),
            body(),
            anchor("botleft"),
            "enemy",
            {
                dir: -1,
                startX: spawnPos.x,
                range: 48,
            },
        ]);
    });

    // Crear jugador
    const player = add([
        rect(24, 30, { radius: 4 }),
        color(70, 170, 255),
        pos(playerSpawn),
        area(),
        body(),
        anchor("botleft"),
        "player",
    ]);

    onUpdate("enemy", (e) => {
        e.move(e.dir * ENEMY_SPEED, 0);
        if (Math.abs(e.pos.x - e.startX) >= e.range) {
            e.dir *= -1;
        }
    });

    function isTouchingWall() {
        if (player.isGrounded()) return false;

        const checkLeft = get("ground").some((g) => {
            const pBox = { x: player.pos.x - 6, y: player.pos.y - 28, w: 6, h: 24 };
            return (
                pBox.x < g.pos.x + TILE_SIZE &&
                pBox.x + pBox.w > g.pos.x &&
                pBox.y < g.pos.y &&
                pBox.y + pBox.h > g.pos.y - TILE_SIZE
            );
        });

        const checkRight = get("ground").some((g) => {
            const pBox = { x: player.pos.x + 24, y: player.pos.y - 28, w: 6, h: 24 };
            return (
                pBox.x < g.pos.x + TILE_SIZE &&
                pBox.x + pBox.w > g.pos.x &&
                pBox.y < g.pos.y &&
                pBox.y + pBox.h > g.pos.y - TILE_SIZE
            );
        });

        return checkLeft || checkRight;
    }

    let wallCoyoteTimer = 0;

    onUpdate(() => {
        const onWall = isTouchingWall();

        if (onWall) {
            wallCoyoteTimer = WALL_COYOTE_TIME;
            if (player.vel.y > 0) {
                player.vel.y = WALL_SLIDE_SPEED;
            }
        } else {
            wallCoyoteTimer -= dt();
        }

        // Chequeo directo de contacto con cada moneda por proximidad
        get("coin").forEach((coin) => {
            if (!coin.collected) {
                const playerCenter = vec2(player.pos.x + 12, player.pos.y - 15);
                if (playerCenter.dist(coin.pos) < 26) {
                    coin.collected = true;
                    coin.color = rgb(110, 115, 125); // Cambia a gris apagado al tocarla
                }
            }
        });

        const currentCam = getCamPos();
        const targetCam = vec2(player.pos.x + 100, 200);
        setCamPos(currentCam.lerp(targetCam, 0.08));

        if (player.pos.y > 600) {
            go("game");
        }
    });

    onKeyDown("left", () => player.move(-SPEED, 0));
    onKeyDown("a", () => player.move(-SPEED, 0));
    onKeyDown("right", () => player.move(SPEED, 0));
    onKeyDown("d", () => player.move(SPEED, 0));

    const tryJump = () => {
        if (player.isGrounded()) {
            player.jump(JUMP_FORCE);
        } else if (wallCoyoteTimer > 0) {
            player.vel.y = -WALL_JUMP_FORCE;
            wallCoyoteTimer = 0;
        }
    };

    onKeyPress("space", tryJump);
    onKeyPress("up", tryJump);
    onKeyPress("w", tryJump);

    // Colisiones
    player.onCollide("hazard", () => go("game"));
    player.onCollide("enemy", () => go("game"));
    player.onCollide("goal", () => go("win"));
    });

    // ==========================================
    // 5. PANTALLA FINAL
    // ==========================================
    scene("win", () => {
    add([
        text("CLEAR", { size: 32 }),
        pos(width() / 2, height() / 2 - 20),
        anchor("center"),
    ]);

    add([
        text("PRESS SPACE", { size: 16 }),
        pos(width() / 2, height() / 2 + 30),
        anchor("center"),
    ]);

    onKeyPress("space", () => go("game"));
    });

    go("game");
