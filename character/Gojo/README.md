# Gojo Satoru Character Engine

## Giới thiệu

`GojoCharacter` là class JavaScript chứa logic nhân vật Gojo Satoru để tích hợp vào game hoặc dự án HTML5 Canvas.

Module hỗ trợ:

- Di chuyển và vật lý cơ bản.
- HP và CE.
- Đấm thường.
- Thương Lam (Blue).
- Phát Đỏ (Red).
- Dịch Chuyển (Teleport).
- Phản Chuyển RCT (Heal).
- Hư Thức Tử (Hollow Purple).
- Vô Hạ Hạn (Infinity).
- Blocking giảm 90% sát thương.
- Render nhân vật bằng Canvas.
- Lấy trạng thái nhân vật bằng `getState()`.

> **Lưu ý:** File `gojo.html` hiện tại là một **demo/testbed hoàn chỉnh**, không phải một ES Module thuần. Trong file này, class `GojoCharacter` nằm trong `<script>` cùng với phần demo, điều khiển bàn phím và game loop. Class được định nghĩa tại phần `class GojoCharacter { ... }`. fileciteturn1file0L151-L156

---

## 1. Cách import vào project khác

Có 2 cách sử dụng.

### Cách 1 — Tách `GojoCharacter` thành file JavaScript riêng

Đây là cách **khuyến nghị** nếu bạn muốn dùng nhân vật trong project khác.

Tạo file:

```text
project/
├── index.html
├── game.js
└── gojoModule.js
```

Trong `gojoModule.js`, chỉ giữ phần:

```js
export default class GojoCharacter {
    // ...
}
```

Sau đó trong `game.js`:

```js
import GojoCharacter from './gojoModule.js';
```

Và trong `index.html`:

```html
<script type="module" src="./game.js"></script>
```

### Cách 2 — Chèn trực tiếp class vào HTML

Nếu project không sử dụng ES Module, bạn có thể chép toàn bộ class `GojoCharacter` vào một file JavaScript hoặc vào `<script>` của HTML.

Ví dụ:

```html
<script>
    class GojoCharacter {
        // toàn bộ class
    }
</script>

<script>
    const gojo = new GojoCharacter({
        id: 1,
        x: 200,
        y: 300
    });
</script>
```

---

## 2. Khởi tạo nhân vật

Sau khi import class:

```js
const gojo = new GojoCharacter({
    id: 1,
    x: 200,
    y: 300,
    facing: 1,
    hp: 100,
    ce: 100
});
```

Các thuộc tính cấu hình chính:

| Thuộc tính | Ý nghĩa                               |
| ------------ | --------------------------------------- |
| `id`       | ID của nhân vật                      |
| `x`        | Tọa độ X ban đầu                   |
| `y`        | Tọa độ Y ban đầu                   |
| `facing`   | Hướng nhìn:`1` phải, `-1` trái |
| `hp`       | HP ban đầu                            |
| `ce`       | CE ban đầu                            |

Trong source hiện tại, constructor thiết lập các thông số này và khởi tạo trạng thái vật lý/chiến đấu. fileciteturn1file0L156-L198

---

## 3. Tích hợp vào Game Loop

Class cần được cập nhật mỗi frame bằng:

```js
gojo.update(opponent, projectileList);
```

Sau đó render:

```js
gojo.draw(ctx);
```

Ví dụ game loop tối thiểu:

```js
function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    gojo.update(opponent, projectiles);
    gojo.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

`update()` xử lý cooldown, hồi CE, hitstun, vật lý và Infinity; `draw()` chịu trách nhiệm render nhân vật lên Canvas. fileciteturn1file0L202-L250

---

## 4. Sử dụng hệ thống chiến đấu

### Đấm

```js
gojo.punch(opponent);
```

### Thương Lam — Blue

Cần 20 CE:

```js
gojo.useBlue(opponent, projectiles);
```

### Phát Đỏ — Red

Cần 30 CE:

```js
gojo.useRed(opponent, projectiles);
```

### Dịch Chuyển — Teleport

Cần 15 CE:

```js
gojo.useTeleport(opponent);
```

### Phản Chuyển — Heal

Cần 35 CE:

```js
gojo.useHeal();
```

### Hư Thức Tử — Hollow Purple

Cần 60 CE:

```js
gojo.startPurpleChant(opponent, projectiles);
```

Các phương thức combat trên được định nghĩa trực tiếp trong `GojoCharacter`. fileciteturn1file0L296-L398

---

## 5. Projectile System

`useBlue()`, `useRed()` và `startPurpleChant()` sử dụng một mảng projectile bên ngoài class.

Tạo mảng:

```js
const projectiles = [];
```

Khi sử dụng skill:

```js
gojo.useBlue(opponent, projectiles);
gojo.useRed(opponent, projectiles);
gojo.startPurpleChant(opponent, projectiles);
```

Sau đó project của bạn phải tự xử lý projectile trong game loop.

Ví dụ:

```js
function updateProjectiles() {
    for (let i = projectiles.length - 1; i >= 0; i--) {
        const projectile = projectiles[i];

        projectile.x += projectile.vx || 0;
        projectile.duration--;

        if (projectile.duration <= 0) {
            projectiles.splice(i, 1);
        }
    }
}
```

**Quan trọng:** class chỉ tạo projectile và thêm chúng vào `projectileList`. Phần xử lý projectile hoàn chỉnh trong file demo nằm ngoài class, trong `updateProjectiles()`. fileciteturn1file0L632-L670

---

## 6. Render Canvas

Project cần có Canvas:

```html
<canvas id="gameCanvas" width="900" height="480"></canvas>
```

Lấy context:

```js
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
```

Sau đó:

```js
gojo.draw(ctx);
```

Phương thức `draw(ctx)` yêu cầu một `CanvasRenderingContext2D`. fileciteturn1file0L404-L405

---

## 7. Lấy trạng thái nhân vật

Có thể lấy trạng thái hiện tại bằng:

```js
const state = gojo.getState();

console.log(state);
```

Ví dụ dữ liệu trả về:

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
    isChanting: false,
    infinityActive: true,
    cooldowns: {
        punch: 0,
        blue: 0,
        red: 0,
        teleport: 0,
        heal: 0,
        purple: 0
    }
}
```

`getState()` được thiết kế để trả về snapshot có thể serialize thành JSON. fileciteturn1file0L537-L550

---

## 8. Ví dụ tích hợp hoàn chỉnh

### `gojoModule.js`

```js
export default class GojoCharacter {
    // Dán toàn bộ class GojoCharacter vào đây.
}
```

### `game.js`

```js
import GojoCharacter from './gojoModule.js';

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const projectiles = [];

const gojo1 = new GojoCharacter({
    id: 1,
    x: 200,
    y: 338
});

const gojo2 = new GojoCharacter({
    id: 2,
    x: 650,
    y: 338
});

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    gojo1.update(gojo2, projectiles);
    gojo2.update(gojo1, projectiles);

    gojo1.draw(ctx);
    gojo2.draw(ctx);

    requestAnimationFrame(gameLoop);
}

gameLoop();
```

### `index.html`

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gojo Game</title>
</head>

<body>

    <canvas id="gameCanvas" width="900" height="480"></canvas>

    <script type="module" src="./game.js"></script>

</body>
</html>
```

---

## 9. Nếu muốn dùng nguyên file `gojo.html`

**Không nên import trực tiếp `gojo.html` như một JavaScript module.**

File hiện tại chứa cả:

- giao diện demo,
- Canvas,
- keyboard controls,
- Inspector,
- projectile system,
- game loop,
- `GojoCharacter`.

Ví dụ, phần demo tự tạo hai nhân vật:

```js
const gojo1 = new GojoCharacter({ id: 1, x: 200, y: 338 });
const gojo2 = new GojoCharacter({ id: 2, x: 650, y: 338 });
```

và tự chạy `mainLoop()`. fileciteturn1file0L554-L562

Vì vậy, nếu project chính đã có game loop và hệ thống input riêng, hãy **tách `class GojoCharacter` ra thành `gojoModule.js`** thay vì nhúng toàn bộ `gojo.html`.

---

## 10. Những phần KHÔNG cần copy sang project chính

Nếu chỉ muốn lấy nhân vật làm module, không cần copy các phần demo sau:

```text
HTML UI
Tailwind CSS
Canvas #stageCanvas
Keyboard event listeners
handleInputEvents()
updateProjectiles()
drawProjectiles()
drawStageEnvironment()
updateInspectorJSON()
mainLoop()
Debug button
Copy ES Module button
```

Bạn chủ yếu cần:

```text
GojoCharacter
├── constructor()
├── update()
├── takeDamage()
├── punch()
├── useBlue()
├── useRed()
├── useTeleport()
├── useHeal()
├── startPurpleChant()
├── executePurpleSkill()
├── draw()
└── getState()
```

Các phương thức này nằm trong class `GojoCharacter` của source hiện tại. fileciteturn1file0L156-L550

---

## 11. Lưu ý quan trọng khi tách module

Có một dependency cần chú ý:

`startPurpleChant()` hiện tại trực tiếp truy cập DOM:

```js
document.getElementById('chantBanner');
document.getElementById('chantText');
```

Do đó, nếu bạn tách class sang một project hoàn toàn khác mà không có hai element này, phần Hollow Purple có thể gây lỗi.

Trong source hiện tại, hai element được lấy từ DOM và banner được hiển thị khi bắt đầu niệm chú. fileciteturn1file0L373-L385

Nếu project chính không sử dụng banner này, nên sửa `startPurpleChant()` để UI được xử lý bởi project chính thay vì để module phụ thuộc trực tiếp vào DOM.

---

## 12. Tóm tắt

Luồng tích hợp cơ bản:

```text
gojoModule.js
      │
      ▼
import GojoCharacter
      │
      ▼
new GojoCharacter(...)
      │
      ├── update()
      ├── skill methods
      ├── takeDamage()
      ├── getState()
      │
      ▼
game loop
      │
      └── draw(ctx)
```

**Khuyến nghị:** tách `GojoCharacter` thành `gojoModule.js` và dùng ES Module. Đây là cách phù hợp nhất khi bạn muốn import nhân vật vào một game/project khác mà không mang theo toàn bộ giao diện demo.
