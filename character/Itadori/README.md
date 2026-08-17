# README — Itadori Yuji Character Engine

## 1. Mục đích

File `itadori.html` chứa class `ItadoriCharacter`, được thiết kế để tách ra và import vào một game/project HTML5 Canvas khác.

**Không import trực tiếp `itadori.html` như JavaScript module.** File này hiện chứa cả class nhân vật và phần demo/testbed: Canvas, keyboard input, projectile system, inspector và game loop. Class `ItadoriCharacter` nằm trong phần `<script>` và bắt đầu ở `class ItadoriCharacter`. 

---

## 2. Phần cần lấy để import

Bạn chỉ cần lấy:

```js
class ItadoriCharacter {
    // ...
}
```

Class hiện có các API:

```text
constructor()
update()
takeDamage()
punch()
useDivergentFist()
useDismantle()
useDash()
useHeal()
useSukunaCleaveCombo()
draw()
getState()
```

Phần demo phía dưới class không cần copy nếu project chính đã có game loop/input riêng.

---

## 3. Tạo file module

Tạo:

```text
project/
├── index.html
├── game.js
└── itadoriModule.js
```

Trong `itadoriModule.js`, đặt class vào và đổi dòng:

```js
class ItadoriCharacter {
```

thành:

```js
export default class ItadoriCharacter {
```

Như vậy class có thể được import bằng ES Module.

---

## 4. Import

Trong `game.js`:

```js
import ItadoriCharacter from './itadoriModule.js';
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
    <title>Itadori Game</title>
</head>
<body>

    <canvas id="gameCanvas" width="900" height="480"></canvas>

    <script type="module" src="./game.js"></script>
</body>
</html>
```

---

## 5. Khởi tạo Itadori

```js
const itadori = new ItadoriCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});
```

Constructor hỗ trợ các giá trị:

```js
{
    id: 1,
    x: 200,
    y: 338,
    facing: 1,
    outfit: 'black',
    hp: 100,
    ce: 100
}
```

Trong source, `width = 46`, `height = 92`, HP/CE tối đa đều là `100`. Hướng mặc định phụ thuộc `id`; outfit mặc định cũng phụ thuộc `id`. 

---

## 6. Tích hợp Canvas

Tạo Canvas:

```html
<canvas id="gameCanvas" width="900" height="480"></canvas>
```

Trong `game.js`:

```js
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
```

Sau đó render:

```js
itadori.draw(ctx);
```

---

## 7. Game Loop

Mỗi frame cần gọi:

```js
itadori.update(opponent, projectiles);
```

và:

```js
itadori.draw(ctx);
```

Ví dụ:

```js
function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    itadori.update(opponent, projectiles);
    itadori.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

`update()` xử lý cooldown, hồi CE, hitstun, dash, delayed impact, gravity, vị trí và hướng nhân vật. 

---

# 8. Projectile List

Tạo một mảng dùng chung:

```js
const projectiles = [];
```

Một số skill cần mảng này:

```js
itadori.useDismantle(opponent, projectiles);

itadori.useSukunaCleaveCombo(opponent, projectiles);
```

`useDismantle()` tạo projectile loại:

```js
type: 'dismantle'
```

và `useSukunaCleaveCombo()` tạo:

```js
type: 'sukunaCleave'
```

Các projectile này được thêm vào `projectileList`; class không tự chạy toàn bộ hệ thống projectile. 

---

# 9. Phải tự xử lý Projectile

Project chính cần xử lý:

```text
projectile movement
       ↓
collision detection
       ↓
damage
       ↓
duration
       ↓
remove projectile
```

Demo gốc xử lý việc này trong `updateProjectiles()`.

Ví dụ tối thiểu:

```js
function updateProjectiles() {
    for (let i = projectiles.length - 1; i >= 0; i--) {
        const p = projectiles[i];

        p.duration--;

        if (p.type === 'dismantle') {
            p.x += p.vx;
        }

        if (p.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}
```

---

# 10. Hai nhân vật

Nếu game có hai Itadori:

```js
const itadori1 = new ItadoriCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});

const itadori2 = new ItadoriCharacter({
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

    itadori1.update(itadori2, projectiles);
    itadori2.update(itadori1, projectiles);

    itadori1.draw(ctx);
    itadori2.draw(ctx);

    updateProjectiles();

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

Đây cũng là cách demo hiện tại truyền đối thủ và `activeProjectiles` vào hai instance. 

---

# 11. Điều khiển

**Input không nằm trong Character Engine.**

Demo hiện tại tự xử lý keyboard bên ngoài class. Ví dụ Itadori #1 dùng:

```text
A / D       Di chuyển
W           Nhảy
S           Block

J           Punch
K           Divergent Fist
L           Dismantle
U           Dash
I           Heal
O           Sukuna Cleave
```

Các phím này chỉ là input của demo, **không bắt buộc phải dùng trong project khác**. 

Project chính có thể tự gọi API:

```js
itadori.punch(opponent);
```

```js
itadori.useDivergentFist(opponent);
```

```js
itadori.useDismantle(opponent, projectiles);
```

```js
itadori.useDash();
```

```js
itadori.useHeal();
```

```js
itadori.useSukunaCleaveCombo(opponent, projectiles);
```

---

# 12. Blocking

Để bật block:

```js
itadori.isBlocking = true;
```

Tắt:

```js
itadori.isBlocking = false;
```

Khi block, `takeDamage()` giảm damage xuống còn 10%.

Ví dụ:

```js
itadori.isBlocking = true;

itadori.takeDamage(100);
```

Damage thực tế:

```text
100 × 0.1 = 10
```

Có thể bỏ qua block:

```js
itadori.takeDamage(100, true);
```

Trong trường hợp này `bypassBlock = true`.

---

# 13. API Combat

## Punch

```js
itadori.punch(opponent);
```

Không cần CE.

---

## Divergent Fist

```js
itadori.useDivergentFist(opponent);
```

Tiêu tốn:

```text
20 CE
```

Nếu đấm trúng, ngoài damage ban đầu còn tạo delayed impact.

---

## Dismantle

```js
itadori.useDismantle(opponent, projectiles);
```

Tiêu tốn:

```text
25 CE
```

Tạo projectile bay theo hướng `facing`.

---

## Dash

```js
itadori.useDash();
```

Tiêu tốn:

```text
12 CE
```

---

## Heal

```js
itadori.useHeal();
```

Tiêu tốn:

```text
30 CE
```

Hồi tối đa `28 HP`.

---

## Sukuna Cleave

```js
itadori.useSukunaCleaveCombo(opponent, projectiles);
```

Tiêu tốn:

```text
55 CE
```

Tạo projectile `sukunaCleave`.

---

# 14. Dependency đặc biệt: `statusBanner`

Có một điểm **quan trọng khi tách class**.

`useSukunaCleaveCombo()` hiện tại truy cập trực tiếp:

```js
document.getElementById('statusBanner');
document.getElementById('statusText');
```

và hiển thị banner khi sử dụng skill. 

Nếu project mới **không có**:

```html
<div id="statusBanner">
    <span id="statusText"></span>
</div>
```

thì không nên giữ nguyên phần DOM này.

### Cách đơn giản

Trong module, bỏ phần:

```js
const banner = document.getElementById('statusBanner');
const statusText = document.getElementById('statusText');

statusText.innerText = ...;
banner.classList.remove('hidden');

setTimeout(() => banner.classList.add('hidden'), 1800);
```

Sau đó để game chính tự xử lý UI.

### Cách tốt hơn

Có thể dùng callback:

```js
itadori.onSkill = (skill) => {
    console.log('Skill:', skill);
};
```

và trong class:

```js
if (this.onSkill) {
    this.onSkill('sukunaCleave');
}
```

Như vậy module không phụ thuộc vào HTML cụ thể.

---

# 15. `draw()` cũng có một dependency

`draw(ctx)` sử dụng:

```js
ctx
```

là Canvas 2D context.

Ngoài ra, source dùng font:

```js
Orbitron
```

ở phần:

```js
ctx.font = 'bold 10px Orbitron, sans-serif';
```

Nếu project mới không import font Orbitron, chữ vẫn có thể render bằng fallback `sans-serif`, nhưng giao diện sẽ khác demo.

---

# 16. Lấy trạng thái

Dùng:

```js
const state = itadori.getState();
```

Ví dụ:

```js
console.log(itadori.getState());
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

Phương thức `getState()` được định nghĩa ở cuối class. 

---

# 17. Ví dụ hoàn chỉnh

## `itadoriModule.js`

```js
export default class ItadoriCharacter {
    // Toàn bộ class ItadoriCharacter
}
```

## `game.js`

```js
import ItadoriCharacter from './itadoriModule.js';

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const projectiles = [];

const itadori = new ItadoriCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});

const enemy = new ItadoriCharacter({
    id: 2,
    x: 650,
    y: 338,
    outfit: 'white'
});

function updateProjectiles() {
    for (let i = projectiles.length - 1; i >= 0; i--) {
        const p = projectiles[i];

        p.duration--;

        if (p.type === 'dismantle') {
            p.x += p.vx;

            if (
                p.x > enemy.x &&
                p.x < enemy.x + enemy.width &&
                p.y > enemy.y &&
                p.y < enemy.y + enemy.height
            ) {
                enemy.takeDamage(p.damage);
                enemy.vx = Math.sign(p.vx) * 14;
                p.duration = 0;
            }
        }

        if (p.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    itadori.update(enemy, projectiles);
    enemy.update(itadori, projectiles);

    updateProjectiles();

    itadori.draw(ctx);
    enemy.draw(ctx);

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

    <title>Itadori Game</title>
</head>

<body>

    <canvas
        id="gameCanvas"
        width="900"
        height="480">
    </canvas>

    <script type="module" src="./game.js"></script>

</body>
</html>
```

---

# 18. Những phần KHÔNG cần import

Nếu project chính đã có hệ thống riêng, không cần copy:

```text
Tailwind CSS
Google Fonts
HTML UI
stageCanvas
statusBanner
statusText
Inspector
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

Các phần này thuộc demo/testbed chứ không phải API chính của `ItadoriCharacter`. Trong source, class kết thúc trước phần khởi tạo Canvas và hai instance demo. 

---

# 19. Cấu trúc khuyến nghị

```text
my-game/
│
├── index.html
│
├── js/
│   ├── game.js
│   ├── itadoriModule.js
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
     Itadori       Input       Projectile
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

# 20. Tóm tắt nhanh

### Bước 1

Tách:

```js
class ItadoriCharacter {
    ...
}
```

ra file:

```text
itadoriModule.js
```

### Bước 2

Đổi thành:

```js
export default class ItadoriCharacter {
    ...
}
```

### Bước 3

Import:

```js
import ItadoriCharacter from './itadoriModule.js';
```

### Bước 4

Khởi tạo:

```js
const itadori = new ItadoriCharacter({
    id: 1,
    x: 200,
    y: 338,
    outfit: 'black'
});
```

### Bước 5

Trong game loop:

```js
itadori.update(opponent, projectiles);
itadori.draw(ctx);
```

### Bước 6

Skill:

```js
itadori.punch(opponent);

itadori.useDivergentFist(opponent);

itadori.useDismantle(opponent, projectiles);

itadori.useDash();

itadori.useHeal();

itadori.useSukunaCleaveCombo(opponent, projectiles);
```

### Bước 7

Tự quản lý:

```text
Input
Projectile
Collision
Game Loop
UI
```

**Lưu ý cuối:** source hiện tại đã có sẵn nút **Copy ES Module Code**, nhưng nó chỉ copy phần `ItadoriCharacter.toString()`, không copy phần demo. 

