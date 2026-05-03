# xk-zuoye
let diskCount = 3;
let towers = [[], [], []];
let moveCount = 0;
let selectedDisk = null;
let autoSolving = false;
const DISK_COLORS = [
    '#e74c3c', '#e67e22', '#f39c12', '#2ecc71', 
    '#3498db', '#9b59b6', '#1abc9c', '#34495e'
];

function startGame() {
    diskCount = parseInt(document.getElementById('diskCount').value);
    resetGame();
    document.getElementById('app').style.display = 'none';
    document.getElementById('gameScreen').style.display = 'flex';
}

function goHome() {
    autoSolving = false;
    document.getElementById('gameScreen').style.display = 'none';
    document.getElementById('app').style.display = 'flex';
}

function resetGame() {
    autoSolving = false;
    towers = [[], [], []];
    moveCount = 0;
    
    for (let i = diskCount; i >= 1; i--) {
        towers[0].push(i);
    }
    
    updateMoveCount();
    updateMinMoves();
    renderDisks();
}

function updateMoveCount() {
    document.getElementById('moveCount').textContent = moveCount;
}

function updateMinMoves() {
    const minMoves = Math.pow(2, diskCount) - 1;
    document.getElementById('minMoves').textContent = minMoves;
    document.getElementById('finalMinMoves').textContent = minMoves;
}

function renderDisks() {
    const gameArea = document.querySelector('.game-area');
    gameArea.innerHTML = '';
    
    for (let t = 0; t < 3; t++) {
        const tower = document.createElement('div');
        tower.className = 'tower';
        tower.id = `tower${t}`;
        tower.addEventListener('click', () => handleTowerClick(t));
        
        const peg = document.createElement('div');
        peg.className = 'peg';
        tower.appendChild(peg);
        
        const indicator = document.createElement('span');
        indicator.className = 'drop-indicator';
        tower.appendChild(indicator);
        
        const diskHeight = 30;
        const baseWidth = 40;
        const widthIncrement = 25;
        
        towers[t].forEach((diskSize, index) => {
            const disk = document.createElement('div');
            disk.className = 'disk';
            disk.dataset.size = diskSize;
            disk.dataset.tower = t;
            disk.style.width = `${baseWidth + diskSize * widthIncrement}px`;
            disk.style.backgroundColor = DISK_COLORS[diskSize - 1];
            disk.style.bottom = `${10 + index * diskHeight}px`;
            disk.addEventListener('click', (e) => {
                e.stopPropagation();
                handleDiskClick(disk);
            });
            tower.appendChild(disk);
        });
        
        gameArea.appendChild(tower);
    }
}

function handleDiskClick(disk) {
    if (autoSolving) return;
    
    const towerIndex = parseInt(disk.dataset.tower);
    const disksOnTower = towers[towerIndex];
    
    if (disksOnTower[disksOnTower.length - 1] !== parseInt(disk.dataset.size)) {
        return;
    }
    
    if (selectedDisk) {
        selectedDisk.classList.remove('selected');
    }
    
    selectedDisk = disk;
    selectedDisk.classList.add('selected');
    
    updateDropIndicators();
}

function handleTowerClick(towerIndex) {
    if (autoSolving || !selectedDisk) return;
    
    const fromTower = parseInt(selectedDisk.dataset.tower);
    const diskSize = parseInt(selectedDisk.dataset.size);
    
    if (canMove(fromTower, towerIndex)) {
        moveDisk(fromTower, towerIndex);
    }
    
    selectedDisk.classList.remove('selected');
    selectedDisk = null;
    clearDropIndicators();
    
    checkWin();
}

function canMove(fromTower, toTower) {
    const fromDisks = towers[fromTower];
    const toDisks = towers[toTower];
    
    if (fromDisks.length === 0) return false;
    
    const movingDisk = fromDisks[fromDisks.length - 1];
    
    if (toDisks.length === 0) return true;
    
    const topDisk = toDisks[toDisks.length - 1];
    
    return movingDisk < topDisk;
}

function moveDisk(fromTower, toTower) {
    const disk = towers[fromTower].pop();
    towers[toTower].push(disk);
    moveCount++;
    updateMoveCount();
    renderDisks();
}

function updateDropIndicators() {
    clearDropIndicators();
    
    if (!selectedDisk) return;
    
    const fromTower = parseInt(selectedDisk.dataset.tower);
    
    for (let t = 0; t < 3; t++) {
        if (t === fromTower) continue;
        
        const tower = document.getElementById(`tower${t}`);
        
        if (canMove(fromTower, t)) {
            tower.classList.add('can-drop');
            tower.querySelector('.drop-indicator').textContent = '✓';
        } else {
            tower.classList.add('cannot-drop');
            tower.querySelector('.drop-indicator').textContent = '✗';
        }
    }
}

function clearDropIndicators() {
    const towers = document.querySelectorAll('.tower');
    towers.forEach(tower => {
        tower.classList.remove('can-drop', 'cannot-drop');
        tower.querySelector('.drop-indicator').textContent = '';
    });
}

function checkWin() {
    if (towers[2].length === diskCount) {
        document.getElementById('finalMoves').textContent = moveCount;
        document.getElementById('winModal').style.display = 'flex';
        createConfetti();
    }
}

function closeWinModal() {
    document.getElementById('winModal').style.display = 'none';
    goHome();
}

function showHelp() {
    document.getElementById('helpModal').style.display = 'flex';
}

function closeHelpModal() {
    document.getElementById('helpModal').style.display = 'none';
}

function createConfetti() {
    const colors = ['#e74c3c', '#e67e22', '#f39c12', '#2ecc71', '#3498db', '#9b59b6'];
    const container = document.getElementById('winModal');
    
    for (let i = 0; i < 50; i++) {
        const confetti = document.createElement('div');
        confetti.className = 'confetti';
        confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
        confetti.style.left = `${Math.random() * 100}%`;
        confetti.style.top = '-10px';
        confetti.style.animationDuration = `${3 + Math.random() * 2}s`;
        confetti.style.animationDelay = `${Math.random() * 0.5}s`;
        container.appendChild(confetti);
    }
    
    setTimeout(() => {
        const confettis = document.querySelectorAll('.confetti');
        confettis.forEach(c => c.remove());
    }, 5000);
}

function autoSolve() {
    if (autoSolving || towers[2].length === diskCount) return;
    
    autoSolving = true;
    selectedDisk = null;
    clearDropIndicators();
    
    const moves = [];
    generateHanoiMoves(diskCount, 0, 2, 1, moves);
    
    let moveIndex = 0;
    const interval = setInterval(() => {
        if (!autoSolving || moveIndex >= moves.length) {
            clearInterval(interval);
            autoSolving = false;
            return;
        }
        
        const [from, to] = moves[moveIndex];
        moveDisk(from, to);
        moveIndex++;
        
        if (moveIndex >= moves.length) {
            clearInterval(interval);
            autoSolving = false;
            checkWin();
        }
    }, 300);
}

function generateHanoiMoves(n, from, to, aux, moves) {
    if (n === 1) {
        moves.push([from, to]);
        return;
    }
    
    generateHanoiMoves(n - 1, from, aux, to, moves);
    moves.push([from, to]);
    generateHanoiMoves(n - 1, aux, to, from, moves);
}
* { 
    box-sizing: border-box; 
    margin: 0; 
    padding: 0; 
    user-select: none; 
} 

body { 
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
    background-color: #2c3e50; 
    color: #ecf0f1; 
    height: 100vh; 
    display: flex; 
    justify-content: center; 
    align-items: center; 
    overflow: hidden; 
} 

#app, #gameScreen { 
    width: 100%; 
    height: 100%; 
    display: flex; 
    justify-content: center; 
    align-items: center; 
} 

.btn { 
    padding: 10px 20px; 
    margin: 5px; 
    border: none; 
    border-radius: 5px; 
    cursor: pointer; 
    font-size: 1rem; 
    transition: background 0.2s; 
} 

.btn-primary { background-color: #3498db; color: white; } 
.btn-primary:hover { background-color: #2980b9; } 

.btn-danger { background-color: #e74c3c; color: white; } 
.btn-danger:hover { background-color: #c0392b; } 

.btn-success { background-color: #2ecc71; color: white; } 
.btn-success:hover { background-color: #27ae60; } 

.home-screen { 
    text-align: center; 
    background: rgba(0, 0, 0, 0.3); 
    padding: 40px; 
    border-radius: 10px; 
    box-shadow: 0 4px 15px rgba(0,0,0,0.5); 
} 

.home-screen h1 { margin-bottom: 20px; } 
.setting-item { margin: 15px 0; text-align: left; } 
.setting-item label { display: block; margin-bottom: 5px; } 
.setting-item select, .setting-item input { width: 100%; padding: 8px; border-radius: 4px; border: 1px solid #ccc; } 

.game-container { 
    position: relative; 
    width: 800px; 
    height: 500px; 
    background: #34495e; 
    border-radius: 10px; 
    padding: 20px; 
    display: flex; 
    flex-direction: column; 
    align-items: center; 
} 

.game-header { 
    width: 100%; 
    display: flex; 
    justify-content: space-between; 
    margin-bottom: 20px; 
    font-size: 1.2rem; 
    font-weight: bold; 
} 

.game-area { 
    flex-grow: 1; 
    width: 100%; 
    display: flex; 
    justify-content: space-around; 
    align-items: flex-end; 
    padding-bottom: 20px; 
    position: relative; 
} 

.tower { 
    width: 160px; 
    height: 100%; 
    display: flex; 
    flex-direction: column-reverse; 
    align-items: center; 
    justify-content: flex-start; 
    position: relative; 
} 

.tower::after { 
    content: ''; 
    width: 200px; 
    height: 10px; 
    background-color: #7f8c8d; 
    border-radius: 5px; 
    position: absolute; 
    bottom: 0; 
} 

.peg { 
    width: 10px; 
    height: 200px; 
    background-color: #95a5a6; 
    position: absolute; 
    bottom: 10px; 
    z-index: 1; 
} 

.disk { 
    height: 30px; 
    border-radius: 8px; 
    position: absolute; 
    cursor: pointer; 
    box-shadow: 0 4px 0 rgba(0,0,0,0.2); 
    transition: background-color 0.2s; 
    z-index: 10; 
} 

.disk:hover { 
    filter: brightness(1.1); 
} 

.disk.dragging { 
    opacity: 0.6; 
    cursor: grabbing; 
    z-index: 1000 !important; 
    transition: none; 
} 

.disk.selected {
    filter: brightness(1.2);
    box-shadow: 0 0 10px rgba(255,255,255,0.5);
}

.drop-indicator { 
    position: absolute; 
    top: -40px; 
    font-size: 30px; 
    font-weight: bold; 
    opacity: 0; 
    transition: opacity 0.2s; 
} 

.tower.can-drop .drop-indicator { color: #2ecc71; opacity: 1; } 
.tower.cannot-drop .drop-indicator { color: #e74c3c; opacity: 1; } 

.modal-overlay { 
    position: fixed; 
    top: 0; left: 0; right: 0; bottom: 0; 
    background: rgba(0,0,0,0.8); 
    display: flex; 
    justify-content: center; 
    align-items: center; 
    z-index: 2000; 
} 

.modal { 
    background: #fff; 
    color: #333; 
    padding: 40px; 
    border-radius: 10px; 
    text-align: center; 
    position: relative; 
    box-shadow: 0 0 20px rgba(255,255,255,0.2); 
    z-index: 2001; 
} 

.confetti { 
    position: absolute; 
    width: 10px; 
    height: 10px; 
    background-color: #f00; 
    animation: fall linear forwards; 
} 

@keyframes fall { 
    to { transform: translateY(100vh) rotate(720deg); } 
}

.game-controls {
    margin-top: 20px;
    display: flex;
    gap: 10px;
}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>汉诺塔游戏</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="app">
        <div class="home-screen">
            <h1>汉诺塔</h1>
            <div class="setting-item">
                <label for="diskCount">选择圆盘数量：</label>
                <select id="diskCount">
                    <option value="3">3个圆盘</option>
                    <option value="4">4个圆盘</option>
                    <option value="5">5个圆盘</option>
                    <option value="6">6个圆盘</option>
                    <option value="7">7个圆盘</option>
                </select>
            </div>
            <button class="btn btn-primary" onclick="startGame()">开始游戏</button>
            <button class="btn btn-success" onclick="showHelp()">游戏说明</button>
        </div>
    </div>

    <div id="gameScreen" style="display: none;">
        <div class="game-container">
            <div class="game-header">
                <div>步数: <span id="moveCount">0</span></div>
                <div>最小步数: <span id="minMoves">0</span></div>
                <button class="btn btn-danger" onclick="goHome()">返回</button>
            </div>
            <div class="game-area">
                <div class="tower" id="tower0">
                    <div class="peg"></div>
                    <span class="drop-indicator"></span>
                </div>
                <div class="tower" id="tower1">
                    <div class="peg"></div>
                    <span class="drop-indicator"></span>
                </div>
                <div class="tower" id="tower2">
                    <div class="peg"></div>
                    <span class="drop-indicator"></span>
                </div>
            </div>
            <div class="game-controls">
                <button class="btn btn-primary" onclick="autoSolve()">自动求解</button>
                <button class="btn btn-success" onclick="resetGame()">重置游戏</button>
            </div>
        </div>
    </div>

    <div id="winModal" class="modal-overlay" style="display: none;">
        <div class="modal">
            <h2>🎉 恭喜你赢了！</h2>
            <p>你用了 <span id="finalMoves">0</span> 步完成游戏</p >
            <p>最小步数是: <span id="finalMinMoves">0</span></p >
            <button class="btn btn-primary" onclick="closeWinModal()">确定</button>
        </div>
    </div>

    <div id="helpModal" class="modal-overlay" style="display: none;">
        <div class="modal">
            <h2>游戏说明</h2>
            <p style="text-align: left; margin: 15px 0;">
                汉诺塔是一个经典的益智游戏。规则如下：
            </p >
            <ul style="text-align: left; margin: 15px 0;">
                <li>将所有圆盘从左边柱子移到右边柱子</li>
                <li>每次只能移动一个圆盘</li>
                <li>大圆盘不能放在小圆盘上面</li>
            </ul>
            <p style="text-align: left; margin: 15px 0;">
                <strong>操作方式：</strong><br>
                点击选中最上方的圆盘，然后点击目标柱子进行移动。
            </p >
            <button class="btn btn-primary" onclick="closeHelpModal()">确定</button>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
