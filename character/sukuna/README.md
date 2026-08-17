# README — Sukuna Character Engine

## 1. Mục đích

File `sukuna.html` chứa class `SukunaCharacter`, được thiết kế để có thể tách ra và import vào một game/project HTML5 Canvas khác.

**Không import trực tiếp `sukuna.html` như một JavaScript module.** File hiện tại gồm cả Character Engine và phần demo/testbed: Canvas, keyboard input, projectile system, UI và game loop.

Phần cần import là:

```js
class SukunaCharacter {
    // ...
}
```

---

## 2. API của `SukunaCharacter`

Class hiện tại cung cấp:

```text
constructor()
update()
takeDamage()
punch()
useDismantle()
useCleave()
useDash()
useHeal()
useFuga()
launchFugaArrow()
draw()
getState()
```

Đây là phần Character Engine cần tách ra khỏi file demo.

---

## 3. Tách thành ES Module

Tạo cấu trúc:

```text
project/
├── index.html
├── game.js
└── sukunaModule.js
```

Trong `sukunaModule.js`, copy toàn bộ class:

```js
class SukunaCharacter {
    // ...
}
```

Sau đó đổi thành:

```js
export default class SukunaCharacter {
    // ...
}
```

**Không copy** phần tạo Canvas, instance demo, keyboard listener hoặc `mainLoop()` sang module nếu project chính đã có các phần đó.

---

## 4. Import vào file khác

Trong `game.js`:

```js
import SukunaCharacter from './sukunaModule.js';
```

Trong `index.html`:

```html
<script type="module" src="./game.js"></script>
```

Ví dụ:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sukuna Game</title>
</head>

<body>

    <canvas id="gameCanvas" width="900" height="480"></canvas>

    <script type="module" src="./game.js"></script>
</body>
</html>
```

---

# 5. Khởi tạo Sukuna

Có thể tạo nhân vật:

```js
const sukuna = new SukunaCharacter({
    id: 1,
    x: 200,
    y: 338,
    facing: 1,
    outfit: 'black',
    hp: 100,
    ce: 100
});
```

Các config chính:

| Thuộc tính | Ý nghĩa |
|---|---|
| `id` | ID nhân vật |
| `x` | Vị trí X |
| `y` | Vị trí Y |
| `facing` | Hướng nhìn: `1` hoặc `-1` |
| `outfit` | Trang phục: `'black'` hoặc `'white'` |
| `hp` | HP ban đầu |
| `ce` | CE ban đầu |

Thông số mặc định trong class:

```text
width  = 48
height = 92
baseSpeed = 7.5
maxHp = 100
maxCe = 100
```

---

# 6. Canvas

Project chính cần Canvas để sử dụng `draw()`.

HTML:

```html
<canvas id="gameCanvas" width="900" height="480"></canvas>
```

JavaScript:

```js
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
```

Render:

```js
sukuna.draw(ctx);
```

---

# 7. Game Loop

Mỗi frame gọi:

```js
sukuna.update(opponent, projectiles);
```

sau đó:

```js
sukuna.draw(ctx);
```

Ví dụ:

```js
function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    sukuna.update(opponent, projectiles);
    sukuna.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

`update()` xử lý:

- cooldown
- hồi CE
- hitstun
- dash
- Fuga chant
- gravity
- movement
- giới hạn màn hình
- tự quay mặt về phía opponent

---

# 8. Projectile List

Tạo một mảng dùng chung:

```js
const projectiles = [];
```

Các skill cần mảng này:

```js
sukuna.useDismantle(opponent, projectiles);
```

và:

```js
sukuna.useFuga(opponent, projectiles);
```

`useDismantle()` tạo projectile:

```js
{
    type: 'dismantle',
    owner: this,
    x: ...,
    y: ...,
    vx: ...,
    radius: 32,
    duration: 40,
    damage: 18
}
```

Fuga sau khi niệm chú sẽ gọi `launchFugaArrow()` và tạo:

```js
{
    type: 'fugaArrow',
    owner: this,
    x: ...,
    y: ...,
    vx: ...,
    radius: 40,
    duration: 50,
    damage: 48
}
```

---

# 9. Project chính phải tự xử lý Projectile

Character Engine chỉ thêm projectile vào:

```js
projectiles
```

Project chính phải xử lý:

```text
Projectile
   ↓
Di chuyển
   ↓
Collision
   ↓
Damage
   ↓
Duration
   ↓
Xóa projectile
```

Ví dụ đơn giản:

```js
function updateProjectiles() {
    for (let i = projectiles.length - 1; i >= 0; i--) {
        const p = projectiles[i];

        p.x += p.vx;
        p.duration--;

        if (p.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}
```

---

# 10. Hai nhân vật

Ví dụ:

```js
const sukuna1 = new SukunaCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});

const sukuna2 = new SukunaCharacter({
    id: 2,
    x: 650,
    y: 338,
    outfit: 'white'
});

const projectiles = [];
```

Game loop:

```js
function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    sukuna1.update(sukuna2, projectiles);
    sukuna2.update(sukuna1, projectiles);

    updateProjectiles();

    sukuna1.draw(ctx);
    sukuna2.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

---

# 11. Combat API

## Punch

```js
sukuna.punch(opponent);
```

Đòn đánh:

```text
Cooldown: 12 frames
Damage:   11
Knockback: 8
```

---

## Dismantle

```js
sukuna.useDismantle(opponent, projectiles);
```

Thông số:

```text
CE cost: 18
Cooldown: 35 frames
Damage: 18
Projectile speed: 22
Duration: 40 frames
```

---

## Cleave

```js
sukuna.useCleave(opponent);
```

Thông số:

```text
CE cost: 30
Cooldown: 80 frames
Damage: 32
Knockback: 16
Hitstun: 22 frames
```

Đây là đòn đánh trực tiếp, không cần projectile list.

---

## Dash

```js
sukuna.useDash();
```

Thông số:

```text
CE cost: 10
Cooldown: 40 frames
Dash duration: 9 frames
Dash speed: 20
```

---

## Heal

```js
sukuna.useHeal();
```

Thông số:

```text
CE cost: 32
Cooldown: 150 frames
Heal: 30 HP
```

Skill không hoạt động nếu HP đã đầy hoặc nhân vật đang hitstun.

---

## Fuga — Fire Arrow

```js
sukuna.useFuga(opponent, projectiles);
```

Thông số:

```text
CE cost: 60
Cooldown: 220 frames
Chant duration: 35 frames
Damage: 48
Projectile speed: 26
Duration: 50 frames
```

Fuga không bắn ngay lập tức. Skill bắt đầu quá trình niệm chú:

```text
useFuga()
    ↓
fugaChantTimer = 35
    ↓
đứng yên trong lúc niệm
    ↓
launchFugaArrow()
    ↓
projectile được thêm vào projectiles
```

---

# 12. Blocking

Bật block:

```js
sukuna.isBlocking = true;
```

Tắt:

```js
sukuna.isBlocking = false;
```

Khi block:

```js
sukuna.takeDamage(100);
```

damage thực tế:

```text
100 × 0.1 = 10
```

Có thể bỏ qua block:

```js
sukuna.takeDamage(100, true);
```

Tham số:

```js
takeDamage(amount, bypassBlock)
```

---

# 13. Lấy trạng thái nhân vật

Dùng:

```js
const state = sukuna.getState();
```

Ví dụ:

```js
console.log(sukuna.getState());
```

State trả về:

```js
{
    id: 1,
    pos: {
        x: 200,
        y: 338
    },
    vel: {
        vx: "0.00",
        vy: "0.00"
    },
    hp: 100,
    ce: 100,
    facing: "Right",
    isDashing: false,
    cooldowns: {
        punch: 0,
        dismantle: 0,
        cleave: 0,
        dash: 0,
        heal: 0,
        fuga: 0
    }
}
```

Có thể dùng state để:

- tạo HUD
- đồng bộ UI
- debug
- lưu trạng thái
- gửi dữ liệu multiplayer

---

# 14. Input không thuộc module

Các phím trong `sukuna.html` chỉ là input của demo.

Character Engine **không bắt buộc** phải dùng các phím đó.

Project mới có thể tự gọi:

```js
sukuna.punch(opponent);
```

```js
sukuna.useDismantle(opponent, projectiles);
```

```js
sukuna.useCleave(opponent);
```

```js
sukuna.useDash();
```

```js
sukuna.useHeal();
```

```js
sukuna.useFuga(opponent, projectiles);
```

Ví dụ:

```js
if (keys['KeyJ']) {
    sukuna.punch(opponent);
}

if (keys['KeyK']) {
    sukuna.useDismantle(opponent, projectiles);
}

if (keys['KeyL']) {
    sukuna.useCleave(opponent);
}

if (keys['KeyU']) {
    sukuna.useDash();
}

if (keys['KeyI']) {
    sukuna.useHeal();
}

if (keys['KeyO']) {
    sukuna.useFuga(opponent, projectiles);
}
```

---

# 15. Dependency đặc biệt: `statusBanner`

Có một dependency quan trọng trong `useFuga()`:

```js
document.getElementById('statusBanner');
document.getElementById('statusText');
```

Fuga hiện tại sử dụng DOM để hiển thị thông báo niệm chú.

Nếu project mới không có:

```html
<div id="statusBanner">
    <span id="statusText"></span>
</div>
```

thì nên sửa phần UI này khi tách module.

### Khuyến nghị

Character Engine không nên phụ thuộc trực tiếp vào một ID HTML cụ thể.

Có thể thay bằng callback:

```js
sukuna.onSkill = (skill) => {
    console.log(skill);
};
```

Khi sử dụng Fuga:

```js
if (this.onSkill) {
    this.onSkill('fuga');
}
```

Project chính sẽ tự quyết định cách hiển thị:

```js
sukuna.onSkill = (skill) => {
    if (skill === 'fuga') {
        showFugaBanner();
    }
};
```

---

# 16. Font

`draw()` sử dụng:

```css
Orbitron
```

trong:

```js
ctx.font = 'bold 12px Orbitron, sans-serif';
```

và:

```js
ctx.font = 'bold 10px Orbitron, sans-serif';
```

Nếu project mới không load Orbitron thì trình duyệt sẽ dùng:

```text
sans-serif
```

làm fallback.

---

# 17. Ví dụ hoàn chỉnh

## `sukunaModule.js`

```js
export default class SukunaCharacter {
    // Dán toàn bộ class SukunaCharacter vào đây.
}
```

## `game.js`

```js
import SukunaCharacter from './sukunaModule.js';

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const projectiles = [];

const sukuna1 = new SukunaCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});

const sukuna2 = new SukunaCharacter({
    id: 2,
    x: 650,
    y: 338,
    outfit: 'white'
});

function updateProjectiles() {
    for (let i = projectiles.length - 1; i >= 0; i--) {
        const p = projectiles[i];

        p.x += p.vx;
        p.duration--;

        if (p.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    sukuna1.update(sukuna2, projectiles);
    sukuna2.update(sukuna1, projectiles);

    updateProjectiles();

    sukuna1.draw(ctx);
    sukuna2.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

## `index.html`

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sukuna Game</title>
</head>

<body>

    <canvas id="gameCanvas" width="900" height="480"></canvas>

    <script type="module" src="./game.js"></script>

</body>
</html>
```

---

# 18. Những phần KHÔNG cần copy

Nếu chỉ muốn import Character Engine, không cần copy:

```text
Tailwind CSS
Google Fonts
HTML UI
stageCanvas wrapper
statusBanner
statusText
Inspector
Debug UI
Copy Code button

keydown listener
keyup listener
handleInputs()
updateProjectiles()
drawProjectiles()
drawStage()
updateInspectorJSON()
mainLoop()
```

Chỉ lấy:

```text
SukunaCharacter
├── constructor()
├── update()
├── takeDamage()
├── punch()
├── useDismantle()
├── useCleave()
├── useDash()
├── useHeal()
├── useFuga()
├── launchFugaArrow()
├── draw()
└── getState()
```

---

# 19. Cấu trúc project khuyến nghị

```text
my-game/
│
├── index.html
│
├── js/
│   ├── game.js
│   ├── sukunaModule.js
│   ├── projectileSystem.js
│   └── input.js
│
└── assets/
```

Luồng:

```text
                  game.js
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Sukuna        Input       Projectile
     Module        System        System
        │            │            │
        └────────────┼────────────┘
                     ▼
                  Game Loop
                     │
                     ▼
                   Canvas
```

---

# 20. Tóm tắt

### Bước 1 — Tách class

Lấy:

```js
class SukunaCharacter {
    ...
}
```

đưa vào:

```text
sukunaModule.js
```

### Bước 2 — Export

Đổi thành:

```js
export default class SukunaCharacter {
    ...
}
```

### Bước 3 — Import

```js
import SukunaCharacter from './sukunaModule.js';
```

### Bước 4 — Khởi tạo

```js
const sukuna = new SukunaCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});
```

### Bước 5 — Update + Render

```js
sukuna.update(opponent, projectiles);
sukuna.draw(ctx);
```

### Bước 6 — Skill

```js
sukuna.punch(opponent);
sukuna.useDismantle(opponent, projectiles);
sukuna.useCleave(opponent);
sukuna.useDash();
sukuna.useHeal();
sukuna.useFuga(opponent, projectiles);
```

### Bước 7 — Project chính quản lý

```text
Input
Projectile
Collision
UI
Game Loop
```

**Quan trọng:** `sukuna.html` là demo hoàn chỉnh; phần cần tái sử dụng/import là `SukunaCharacter`. Khi tách module, đặc biệt cần xử lý dependency `statusBanner/statusText` của Fuga và hệ thống projectile bên ngoài class.
