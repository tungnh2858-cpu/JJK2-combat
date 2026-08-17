# README — GetoCharacter Import Guide

## 1. Mục đích

File `geto.html` chứa class `GetoCharacter`, là Character Engine cho Geto Suguru.

Class có:

- Di chuyển và vật lý cơ bản
- HP / CE
- Blocking
- Hitstun
- Tam Tiết Côn
- Triệu hồi Chú Linh bay
- Bẫy Lưới Nguyền Hồn
- Lướt nhanh
- Nuốt Chú Linh để hồi máu
- Tuyệt kỹ Uzumaki

Source gốc cũng cung cấp ví dụ sử dụng:

```js
const geto = new GetoCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'monk'
});

geto.update(opponent, projectiles);
geto.draw(canvasContext);
```

fileciteturn12file0L140-L147

---

# 2. Quan trọng: Không import trực tiếp `geto.html`

File `geto.html` hiện tại là một demo hoàn chỉnh, không phải ES Module thuần.

Nó chứa:

```text
HTML
CSS
GetoCharacter
Canvas
Keyboard Input
Projectile System
UI Inspector
Game Loop
```

Class `GetoCharacter` bắt đầu trong `<script>` và kết thúc trước phần khởi tạo Canvas/demo. fileciteturn12file0L152-L191 fileciteturn12file0L498-L510

Nếu muốn import vào project khác, hãy tách class ra thành file JavaScript riêng.

---

# 3. Cấu trúc project

Khuyến nghị:

```text
my-game/
│
├── index.html
│
├── js/
│   ├── game.js
│   └── getoModule.js
│
└── assets/
```

---

# 4. Tạo `getoModule.js`

Copy toàn bộ:

```js
class GetoCharacter {
    ...
}
```

từ `geto.html`.

Sau đó đổi dòng:

```js
class GetoCharacter {
```

thành:

```js
export default class GetoCharacter {
```

Cuối cùng file sẽ có dạng:

```js
export default class GetoCharacter {

    constructor(config = {}) {
        // ...
    }

    update(opponent, projectileList = []) {
        // ...
    }

    takeDamage(amount, bypassBlock = false) {
        // ...
    }

    // ...

    getState() {
        // ...
    }
}
```

**Không copy phần demo bên ngoài class** vào module.

---

# 5. Import vào `game.js`

```js
import GetoCharacter from './getoModule.js';
```

Sau đó tạo nhân vật:

```js
const geto = new GetoCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'monk'
});
```

Các giá trị `id`, `x`, `y`, `facing`, `outfit`, `hp`, `ce` được class hỗ trợ. Source đặt kích thước `48 × 92`, tốc độ cơ bản `7.2`, HP tối đa `100` và CE tối đa `100`. fileciteturn12file0L153-L176

---

# 6. `index.html`

Vì sử dụng:

```js
import
```

phải chạy script bằng:

```html
<script type="module" src="./js/game.js"></script>
```

Ví dụ:

```html
<!DOCTYPE html>
<html lang="vi">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Geto Game</title>
</head>

<body>

    <canvas
        id="gameCanvas"
        width="900"
        height="480">
    </canvas>

    <script
        type="module"
        src="./js/game.js">
    </script>

</body>

</html>
```

---

# 7. Canvas

`GetoCharacter.draw()` nhận một Canvas 2D context:

```js
geto.draw(ctx);
```

Tạo context:

```js
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
```

Sau đó:

```js
geto.draw(ctx);
```

---

# 8. Game Loop

Mỗi frame cần gọi:

```js
geto.update(opponent, projectiles);
```

và:

```js
geto.draw(ctx);
```

Ví dụ:

```js
function gameLoop() {

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    geto.update(
        opponent,
        projectiles
    );

    geto.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

`update()` xử lý cooldown, hồi CE, hitstun, dash, Uzumaki chant, gravity, vị trí và hướng nhìn. fileciteturn12file0L193-L244

---

# 9. Projectile List

Tạo một array dùng chung:

```js
const projectiles = [];
```

Truyền array này vào:

```js
geto.update(opponent, projectiles);
```

Các skill tạo projectile:

```js
geto.useSummonCurse(opponent, projectiles);
```

```js
geto.useCurseTrap(opponent, projectiles);
```

```js
geto.useUzumaki(opponent, projectiles);
```

Source tạo ba loại projectile:

```text
flyingCurse
curseNetTrap
uzumakiBeam
```

fileciteturn12file0L276-L352

---

# 10. Project chính phải xử lý projectile

`GetoCharacter` tạo projectile và đưa nó vào:

```js
projectiles
```

Nhưng hệ thống game chính phải xử lý:

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

Demo gốc thực hiện việc này trong `updateProjectiles()`. fileciteturn12file0L562-L603

Ví dụ tối thiểu:

```js
function updateProjectiles() {

    for (let i = projectiles.length - 1; i >= 0; i--) {

        const p = projectiles[i];

        p.duration--;

        if (p.type === 'flyingCurse') {
            p.x += p.vx;
        }

        if (p.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}
```

---

# 11. Collision

Ví dụ `flyingCurse`:

```js
if (
    p.x > opponent.x &&
    p.x < opponent.x + opponent.width &&
    p.y > opponent.y &&
    p.y < opponent.y + opponent.height
) {
    opponent.takeDamage(p.damage);
    opponent.vx = Math.sign(p.vx) * 12;
    p.duration = 0;
}
```

Đây chính là logic mà demo gốc sử dụng cho `flyingCurse`. fileciteturn12file0L569-L577

---

# 12. Hai nhân vật Geto

```js
const geto1 = new GetoCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'monk'
});

const geto2 = new GetoCharacter({
    id: 2,
    x: 650,
    y: 338,
    outfit: 'academy'
});

const projectiles = [];
```

Game loop:

```js
function gameLoop() {

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    geto1.update(geto2, projectiles);
    geto2.update(geto1, projectiles);

    updateProjectiles();

    geto1.draw(ctx);
    geto2.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

Demo gốc cũng tạo hai instance theo cấu trúc này. fileciteturn12file0L512-L518

---

# 13. API của Geto

## 13.1 Tam Tiết Côn

```js
geto.playfulCloudAttack(opponent);
```

Thông số trong source:

```text
Cooldown : 14 frames
Damage   : 15
Knockback: 11
Reach    : 85
```

Không cần projectile array. fileciteturn12file0L256-L274

---

## 13.2 Triệu Hồi Chú Linh

```js
geto.useSummonCurse(
    opponent,
    projectiles
);
```

Thông số:

```text
CE cost          : 16
Cooldown         : 32 frames
Damage           : 16
Projectile speed : 16
Duration         : 45 frames
Radius           : 28
```

fileciteturn12file0L276-L291

---

## 13.3 Lưới Nguyền Hồn

```js
geto.useCurseTrap(
    opponent,
    projectiles
);
```

Thông số:

```text
CE cost : 25
Cooldown: 70 frames
Damage  : 22
Duration: 55 frames
Size    : 60 × 60
```

fileciteturn12file0L293-L311

---

## 13.4 Lướt Nhanh

```js
geto.useDash();
```

Thông số:

```text
CE cost      : 10
Cooldown     : 38 frames
Dash duration: 9 frames
Dash speed   : 19
```

fileciteturn12file0L313-L319

---

## 13.5 Nuốt Chú Linh

```js
geto.useAbsorbCurse();
```

Thông số:

```text
CE cost : 30
Cooldown: 140 frames
Heal    : 28 HP
```

Skill không hoạt động khi HP đã đầy hoặc đang hitstun. fileciteturn12file0L321-L326

---

## 13.6 Uzumaki

```js
geto.useUzumaki(
    opponent,
    projectiles
);
```

Thông số:

```text
CE cost       : 60
Cooldown      : 230 frames
Chant duration: 38 frames
Damage        : 52
Beam width    : 600
Beam height   : 120
Duration      : 35 frames
```

Uzumaki có hai giai đoạn:

```text
useUzumaki()
      ↓
niệm chú 38 frames
      ↓
launchUzumakiBeam()
      ↓
tạo projectile
```

fileciteturn12file0L328-L352

---

# 14. Blocking

Bật block:

```js
geto.isBlocking = true;
```

Tắt:

```js
geto.isBlocking = false;
```

Khi block:

```js
geto.takeDamage(100);
```

Damage còn:

```text
100 × 0.1 = 10
```

Nếu:

```js
geto.takeDamage(100, true);
```

thì `bypassBlock = true`, bỏ qua block. fileciteturn12file0L246-L254

---

# 15. Input

Input **không phải một phần của `GetoCharacter`**.

Demo gốc chỉ dùng keyboard để điều khiển hai instance.

### Player 1

```text
A / D → di chuyển
W     → nhảy
S     → block

J → Tam Tiết Côn
K → Triệu Hồi Chú Linh
L → Lưới Nguyền Hồn
U → Dash
I → Nuốt Chú Linh
O → Uzumaki
```

### Player 2

```text
← / → → di chuyển
↑     → nhảy
↓     → block

1 → Tam Tiết Côn
2 → Triệu Hồi Chú Linh
3 → Lưới Nguyền Hồn
4 → Dash
5 → Nuốt Chú Linh
6 → Uzumaki
```

fileciteturn12file0L534-L560

Project mới có thể sử dụng input system hoàn toàn khác.

---

# 16. Dependency đặc biệt: `statusBanner`

Có một dependency cần chú ý.

`useUzumaki()` hiện tại truy cập trực tiếp:

```js
document.getElementById('statusBanner');
document.getElementById('statusText');
```

và hiển thị banner. fileciteturn12file0L328-L339

Nếu project mới không có:

```html
<div id="statusBanner">
    <span id="statusText"></span>
</div>
```

thì cần sửa phần UI này.

## Khuyến nghị

Character Engine nên dùng callback thay vì truy cập trực tiếp DOM.

Ví dụ:

```js
geto.onSkill = (skill) => {
    console.log(skill);
};
```

Trong `useUzumaki()`:

```js
if (this.onSkill) {
    this.onSkill('uzumaki');
}
```

Project chính:

```js
geto.onSkill = (skill) => {

    if (skill === 'uzumaki') {
        showUzumakiBanner();
    }

};
```

Như vậy module không phụ thuộc vào HTML demo.

---

# 17. Font

`draw()` sử dụng font:

```text
Orbitron
```

Ví dụ:

```js
ctx.font = 'bold 12px Orbitron, sans-serif';
```

và:

```js
ctx.font = 'bold 10px Orbitron, sans-serif';
```

Nếu project không load Orbitron, trình duyệt sẽ dùng:

```text
sans-serif
```

làm fallback. fileciteturn12file0L375-L389 fileciteturn12file0L484-L487

---

# 18. `getState()`

Lấy trạng thái:

```js
const state = geto.getState();
```

Ví dụ:

```js
console.log(
    geto.getState()
);
```

State gồm:

```js
{
    id,
    pos,
    vel,
    hp,
    ce,
    facing,
    isDashing,
    cooldowns
}
```

Source trả về các giá trị này tại `getState()`. fileciteturn12file0L498-L508

Có thể dùng cho:

```text
HUD
Debug
Inspector
Save
Multiplayer
```

---

# 19. Ví dụ hoàn chỉnh

## `getoModule.js`

```js
export default class GetoCharacter {

    // Dán toàn bộ class GetoCharacter
    // từ geto.html vào đây.

}
```

## `game.js`

```js
import GetoCharacter from './getoModule.js';

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const projectiles = [];

const geto = new GetoCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'monk'
});

const opponent = new GetoCharacter({
    id: 2,
    x: 650,
    y: 338,
    outfit: 'academy'
});

function updateProjectiles() {

    for (let i = projectiles.length - 1; i >= 0; i--) {

        const p = projectiles[i];

        p.duration--;

        const target =
            p.owner === geto
                ? opponent
                : geto;

        if (p.type === 'flyingCurse') {

            p.x += p.vx;

            if (
                p.x > target.x &&
                p.x < target.x + target.width &&
                p.y > target.y &&
                p.y < target.y + target.height
            ) {

                target.takeDamage(p.damage);

                target.vx =
                    Math.sign(p.vx) * 12;

                p.duration = 0;
            }
        }

        if (p.type === 'curseNetTrap') {

            if (
                !p.triggered &&
                target.x + target.width > p.x - 30 &&
                target.x < p.x + p.width + 30 &&
                target.y + target.height >= 400
            ) {

                p.triggered = true;

                target.takeDamage(p.damage);

                target.vx = 0;
                target.hitStun = 20;
            }
        }

        if (p.type === 'uzumakiBeam') {

            if (
                target.x + target.width > p.x &&
                target.x < p.x + p.width &&
                target.y + target.height > p.y &&
                target.y < p.y + p.height
            ) {

                target.takeDamage(
                    p.damage,
                    true
                );

                target.vx =
                    p.owner.facing * 18;
            }
        }

        if (p.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}

function gameLoop() {

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    geto.update(
        opponent,
        projectiles
    );

    opponent.update(
        geto,
        projectiles
    );

    updateProjectiles();

    geto.draw(ctx);
    opponent.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

---

# 20. Những phần KHÔNG cần copy

Nếu project chính đã có hệ thống riêng, **không cần copy**:

```text
Tailwind CSS
Google Fonts
HTML UI
stageCanvas
statusBanner
statusText
geto1Data
geto2Data

btnToggleDebug
btnCopyCode

keydown listener
keyup listener
handleInputs()
updateProjectiles()
drawProjectiles()
drawStage()
updateInspectorJSON()
mainLoop()
```

Các phần trên thuộc demo/testbed.

Phần cần tái sử dụng chính là:

```text
GetoCharacter
├── constructor()
├── update()
├── takeDamage()
├── playfulCloudAttack()
├── useSummonCurse()
├── useCurseTrap()
├── useDash()
├── useAbsorbCurse()
├── useUzumaki()
├── launchUzumakiBeam()
├── draw()
└── getState()
```

Source gốc xác định toàn bộ các method này trong class `GetoCharacter`. fileciteturn12file0L193-L510

---

# 21. Cấu trúc project khuyến nghị

```text
my-game/
│
├── index.html
│
├── js/
│   ├── game.js
│   ├── getoModule.js
│   ├── projectileSystem.js
│   └── input.js
│
└── assets/
```

Luồng hoạt động:

```text
                       game.js
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           Geto         Input       Projectile
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

# 22. Tóm tắt nhanh

### Bước 1 — Tách class

Từ:

```js
class GetoCharacter {
    ...
}
```

tạo:

```text
getoModule.js
```

### Bước 2 — Export

```js
export default class GetoCharacter {
    ...
}
```

### Bước 3 — Import

```js
import GetoCharacter from './getoModule.js';
```

### Bước 4 — Khởi tạo

```js
const geto = new GetoCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'monk'
});
```

### Bước 5 — Update

```js
geto.update(
    opponent,
    projectiles
);
```

### Bước 6 — Render

```js
geto.draw(ctx);
```

### Bước 7 — Sử dụng skill

```js
geto.playfulCloudAttack(opponent);

geto.useSummonCurse(
    opponent,
    projectiles
);

geto.useCurseTrap(
    opponent,
    projectiles
);

geto.useDash();

geto.useAbsorbCurse();

geto.useUzumaki(
    opponent,
    projectiles
);
```

### Bước 8 — Project chính quản lý

```text
Input
Projectile
Collision
UI
Game Loop
```

**Lưu ý:** File `geto.html` có sẵn nút `Copy ES Module Code`, và nút này lấy `GetoCharacter.toString()` để copy phần class. Tuy nhiên, để sử dụng chuẩn trong project, vẫn cần thêm `export default` và import bằng `<script type="module">`. fileciteturn12file0L707-L719
