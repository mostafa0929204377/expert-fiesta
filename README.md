<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>دومنا مصطفى - أونلاين</title>
    <!-- مكتبة الربط المباشر بين اللاعبين P2P عبر الويب -->
    <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
    <style>
        :root {
            --bg-color: #0b3c22;
            --table-color: #11502e;
            --domino-bg: #fffbf0;
            --accent: #e9c46a;
            --primary: #e76f51;
        }
        * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, sans-serif; user-select: none; }
        body {
            margin: 0; padding: 10px; background: radial-gradient(circle, var(--table-color), var(--bg-color));
            color: #fff; min-height: 100vh; display: flex; flex-direction: column; align-items: center;
        }
        h1 { margin: 5px 0 15px; color: var(--accent); text-shadow: 2px 2px 4px #000; }
        
        /* شاشات الردهة */
        .lobby-container {
            display: flex; gap: 20px; flex-wrap: wrap; justify-content: center; width: 100%; max-width: 850px;
        }
        .card {
            background: rgba(0, 0, 0, 0.45); border: 2px solid #2a7a4c; border-radius: 12px;
            padding: 20px; flex: 1; min-width: 300px; text-align: center; box-shadow: 0 8px 16px rgba(0,0,0,0.4);
        }
        .card h2 { margin-top: 0; color: #fff; border-bottom: 1px solid #2a7a4c; padding-bottom: 8px; }
        input, select, button {
            width: 100%; padding: 12px; margin: 8px 0; border-radius: 8px; border: none; font-size: 15px;
        }
        input { background: #222; color: #fff; border: 1px solid #444; text-align: center; }
        button { background: var(--primary); color: #fff; font-weight: bold; cursor: pointer; transition: 0.2s; }
        button:hover { background: #f43f5e; }
        button:disabled { background: #666; cursor: not-allowed; }

        /* غرفة الانتظار */
        #waiting-room, #game-screen { display: none; width: 100%; max-width: 950px; text-align: center; }
        .code-display {
            background: #000; color: var(--accent); font-size: 24px; font-weight: bold;
            padding: 10px; border-radius: 8px; letter-spacing: 3px; margin: 10px 0;
        }

        /* طاولة اللعبة */
        #status-bar {
            background: rgba(0,0,0,0.6); padding: 10px; border-radius: 8px; font-size: 18px;
            font-weight: bold; color: var(--accent); margin-bottom: 10px;
        }
        #players-bar {
            display: flex; justify-content: space-around; margin-bottom: 10px; background: rgba(0,0,0,0.3);
            padding: 8px; border-radius: 8px;
        }
        .player-tag { padding: 4px 10px; border-radius: 5px; font-size: 14px; }
        .active-turn { background: var(--accent); color: #000; font-weight: bold; }

        #board-wrapper {
            background: rgba(0, 0, 0, 0.35); border: 2px dashed #3a8a5c; border-radius: 12px;
            height: 190px; overflow-x: auto; display: flex; align-items: center; padding: 15px; gap: 4px;
        }

        /* أحجار الدومينو */
        .domino {
            background: var(--domino-bg); color: #111; border: 2px solid #222; border-radius: 6px;
            box-shadow: 2px 4px 8px rgba(0,0,0,0.5); display: inline-flex; cursor: pointer;
        }
        .domino-hand {
            width: 44px; height: 86px; flex-direction: column; transition: transform 0.2s;
        }
        .domino-hand:hover { transform: translateY(-8px); }
        .domino-board { width: 80px; height: 42px; flex-direction: row; flex-shrink: 0; cursor: default; }
        .half { flex: 1; display: flex; align-items: center; justify-content: center; font-size: 20px; font-weight: 900; }
        .divider-h { height: 2px; background: #888; margin: 0 4px; }
        .divider-v { width: 2px; background: #888; margin: 4px 0; }
        
        .playable { border: 2px solid #ffcc00; box-shadow: 0 0 12px #ffcc00; transform: translateY(-6px); }

        /* منطقة القطع */
        #my-hand { display: flex; justify-content: center; gap: 8px; flex-wrap: wrap; margin-top: 15px; }
        .action-btns { display: flex; gap: 10px; justify-content: center; margin-top: 15px; }
        .action-btns button { width: auto; padding: 10px 25px; }
    </style>
</head>
<body>

    <h1>🎲 دومنا مصطفى 🎲</h1>

    <!-- 1. شاشة البداية (الردهة) -->
    <div id="lobby" class="lobby-container">
        <!-- خانة إنشاء فريق -->
        <div class="card">
            <h2>إنشاء فريق / غرفة</h2>
            <p style="font-size: 13px; color: #ccc;">اسمك يظهر للجميع: <b>المنشئ</b></p>
            <label style="display:block; text-align:right; margin-bottom:5px;">اختر نمط اللعب:</label>
            <select id="game-mode">
                <option value="duo">نمط الثنائي (1 ضد 1)</option>
                <option value="team">نمط الفريق (2 ضد 2)</option>
            </select>
            <button onclick="hostCreateRoom()">إنشاء الغرفة واستخراج الكود</button>
        </div>

        <!-- خانة انضمام لفريق -->
        <div class="card">
            <h2>انضمام إلى فريق</h2>
            <input type="text" id="join-name" placeholder="أدخل اسمك (تغيير الاسم)" maxlength="12">
            <input type="text" id="join-code" placeholder="أدخل كود الفريق">
            <button onclick="joinRoom()" style="background:#2a9d8f;">انضمام للعبة</button>
        </div>
    </div>

    <!-- 2. غرفة انتظار المنشئ واللاعبين -->
    <div id="waiting-room">
        <div class="card" style="margin: auto; max-width: 500px;">
            <h2>غرفة الانتظار</h2>
            <p>شارك هذا الكود مع أصدقائك للانضمام:</p>
            <div id="room-code-txt" class="code-display">------</div>
            <button onclick="copyCode()" style="background:#457b9d;">نسخ الكود 📋</button>
            <p id="waiting-status" style="margin-top: 15px; color: var(--accent);">بانتظار اكتمال اللاعبين...</p>
            <ul id="players-list" style="list-style: none; padding: 0;"></ul>
            <button id="start-game-btn" onclick="hostStartGame()" style="display:none; background:#2a9d8f;">بدء اللعبة الآن 🚀</button>
        </div>
    </div>

    <!-- 3. شاشة اللعبة الأساسية -->
    <div id="game-screen">
        <div id="status-bar">جاري التجهيز...</div>
        <div id="players-bar"></div>

        <!-- رقعة الطاولة -->
        <div id="board-wrapper">
            <div id="board" style="display:flex; align-items:center; gap:4px;"></div>
        </div>

        <!-- يد اللاعب -->
        <div id="my-hand"></div>

        <!-- أزرار الإجراءات -->
        <div class="action-btns">
            <button id="draw-btn" onclick="sendAction({type:'DRAW'})" disabled>سحب من البنك (<span id="boneyard-count">0</span>)</button>
            <button id="pass-btn" onclick="sendAction({type:'PASS'})" disabled>تمرير الدور</button>
        </div>
    </div>

    <script>
        let peer = null;
        let conn = null; // للمنضم
        let connections = []; // للمنشئ
        let isHost = false;
        let myId = "";
        let myName = "المنشئ";
        let roomCode = "";
        let selectedMode = "duo";

        // حالة اللعبة الكاملة (يديرها المنشئ فقط لضمان النزاهة)
        let state = {
            mode: "duo",
            players: [], // { id, name, team }
            deck: [],
            hands: {},
            board: [],
            currentTurn: 0,
            consecutivePasses: 0
        };

        // توليد كود عشوائي من 5 أحرف وأرقام
        function generateCode() {
            return Math.random().toString(36).substring(2, 7).toUpperCase();
        }

        // إنشاء طاولة جديدة بواسطة المنشئ
        function hostCreateRoom() {
            isHost = true;
            myName = "المنشئ";
            selectedMode = document.getElementById('game-mode').value;
            roomCode = generateCode();

            peer = new Peer("dm-mstf-" + roomCode);

            peer.on('open', (id) => {
                myId = id;
                state.mode = selectedMode;
                state.players.push({ id: myId, name: "المنشئ", team: 1 });

                document.getElementById('lobby').style.display = 'none';
                document.getElementById('waiting-room').style.display = 'block';
                document.getElementById('room-code-txt').innerText = roomCode;
                updateWaitingLobby();
            });

            peer.on('connection', (c) => {
                let maxPlayers = state.mode === 'duo' ? 2 : 4;
                if (connections.length >= maxPlayers - 1) {
                    c.send({ type: 'ERROR', msg: 'الغرفة ممتلئة!' });
                    c.close();
                    return;
                }

                c.on('data', (data) => {
                    if (data.type === 'JOIN') {
                        let team = (connections.length % 2 === 0) ? 2 : 1; // توزيع الفرق بالتناوب
                        connections.push(c);
                        state.players.push({ id: c.peer, name: data.name || "لاعب", team: team });
                        broadcastState({ type: 'LOBBY_UPDATE', players: state.players });
                        updateWaitingLobby();
                    } else {
                        handlePlayerAction(c.peer, data);
                    }
                });
            });

            peer.on('error', (err) => alert("حدث خطأ في إنشاء الغرفة: " + err));
        }

        // انضمام لاعب عبر الكود
        function joinRoom() {
            let code = document.getElementById('join-code').value.trim().toUpperCase();
            let name = document.getElementById('join-name').value.trim();

            if (!code) return alert("يرجى كتابة كود الفريق!");
            if (!name) return alert("يرجى اختيار اسم لك أولاً!");

            isHost = false;
            myName = name;
            peer = new Peer();

            peer.on('open', (id) => {
                myId = id;
                conn = peer.connect("dm-mstf-" + code);

                conn.on('open', () => {
                    conn.send({ type: 'JOIN', name: myName });
                    document.getElementById('lobby').style.display = 'none';
                    document.getElementById('waiting-room').style.display = 'block';
                    document.getElementById('room-code-txt').innerText = code;
                });

                conn.on('data', (data) => {
                    if (data.type === 'LOBBY_UPDATE') {
                        state.players = data.players;
                        updateWaitingLobby();
                    } else if (data.type === 'GAME_START') {
                        document.getElementById('waiting-room').style.display = 'none';
                        document.getElementById('game-screen').style.display = 'block';
                    } else if (data.type === 'SYNC') {
                        renderGame(data.payload);
                    } else if (data.type === 'ALERT') {
                        alert(data.msg);
                    }
                });
            });

            peer.on('error', () => alert("تعذر العثور على الغرفة، تأكد من الكود!"));
        }

        function copyCode() {
            navigator.clipboard.writeText(roomCode);
            alert("تم نسخ الكود بنجاح: " + roomCode);
        }

        function updateWaitingLobby() {
            let list = document.getElementById('players-list');
            list.innerHTML = "";
            state.players.forEach((p) => {
                let info = p.name;
                if (state.mode === 'team') info += ` (فريق ${p.team === 1 ? 'أ' : 'ب'})`;
                list.innerHTML += `<li style="padding:6px; font-weight:bold;">👤 ${info}</li>`;
            });

            let required = state.mode === 'duo' ? 2 : 4;
            if (isHost) {
                let btn = document.getElementById('start-game-btn');
                if (state.players.length === required) {
                    btn.style.display = 'block';
                    document.getElementById('waiting-status').innerText = "اكتمل العدد! يمكنك بدء اللعبة.";
                } else {
                    btn.style.display = 'none';
                    document.getElementById('waiting-status').innerText = `بانتظار اكتمال اللاعبين (${state.players.length}/${required})...`;
                }
            }
        }

        // --- محرك وقوانين الدومينو (عند المنشئ) ---

        function hostStartGame() {
            // توليد قطع الدومينو 28
            state.deck = [];
            for (let i = 0; i <= 6; i++) {
                for (let j = i; j <= 6; j++) {
                    state.deck.push({ a: i, b: j });
                }
            }
            state.deck.sort(() => Math.random() - 0.5);

            // توزيع 7 قطع لكل لاعب
            state.hands = {};
            state.players.forEach(p => {
                state.hands[p.id] = state.deck.splice(0, 7);
            });

            // إنزال أول قطعة على الطاولة
            state.board = [state.deck.pop()];
            state.currentTurn = 0;
            state.consecutivePasses = 0;

            broadcastState({ type: 'GAME_START' });
            document.getElementById('waiting-room').style.display = 'none';
            document.getElementById('game-screen').style.display = 'block';

            syncGameState();
        }

        function syncGameState() {
            state.players.forEach(p => {
                let payload = {
                    board: state.board,
                    boneyardCount: state.deck.length,
                    currentTurn: state.currentTurn,
                    players: state.players.map(pl => ({
                        name: pl.name,
                        team: pl.team,
                        count: state.hands[pl.id].length
                    })),
                    hand: state.hands[p.id],
                    turnPlayerName: state.players[state.currentTurn].name
                };

                if (p.id === myId) {
                    renderGame(payload);
                } else {
                    let c = connections.find(cn => cn.peer === p.id);
                    if (c) c.send({ type: 'SYNC', payload: payload });
                }
            });
        }

        function broadcastState(msg) {
            connections.forEach(c => c.send(msg));
        }

        function sendAction(action) {
            if (isHost) {
                handlePlayerAction(myId, action);
            } else {
                conn.send(action);
            }
        }

        function handlePlayerAction(playerId, action) {
            let activePlayer = state.players[state.currentTurn];
            if (activePlayer.id !== playerId) return;

            let hand = state.hands[playerId];
            let leftEnd = state.board[0].a;
            let rightEnd = state.board[state.board.length - 1].b;

            if (action.type === 'PLAY') {
                let tile = hand[action.index];
                let played = false;

                if (tile.b === leftEnd) {
                    state.board.unshift(hand.splice(action.index, 1)[0]);
                    played = true;
                } else if (tile.a === leftEnd) {
                    let t = hand.splice(action.index, 1)[0];
                    state.board.unshift({ a: t.b, b: t.a });
                    played = true;
                } else if (tile.a === rightEnd) {
                    state.board.push(hand.splice(action.index, 1)[0]);
                    played = true;
                } else if (tile.b === rightEnd) {
                    let t = hand.splice(action.index, 1)[0];
                    state.board.push({ a: t.b, b: t.a });
                    played = true;
                }

                if (played) {
                    state.consecutivePasses = 0;
                    if (hand.length === 0) {
                        endRound(activePlayer.name + " أنهى جميع أحجاره وفاز باللعبة! 🎉");
                        return;
                    }
                    nextTurn();
                }
            } else if (action.type === 'DRAW') {
                if (state.deck.length > 0) {
                    hand.push(state.deck.pop());
                    syncGameState();
                }
            } else if (action.type === 'PASS') {
                state.consecutivePasses++;
                if (state.consecutivePasses >= state.players.length) {
                    // قفلة اللعبة - احتساب النقاط
                    resolveLockGame();
                    return;
                }
                nextTurn();
            }
        }

        function nextTurn() {
            state.currentTurn = (state.currentTurn + 1) % state.players.length;
            syncGameState();
        }

        function resolveLockGame() {
            let scores = state.players.map(p => {
                let sum = state.hands[p.id].reduce((s, t) => s + t.a + t.b, 0);
                return { name: p.name, team: p.team, sum: sum };
            });

            scores.sort((a, b) => a.sum - b.sum);
            let winnerMsg = `قُفلت اللعبة!\nالفائز بأقل نقاط: ${scores[0].name} (${scores[0].sum} نقطة) 🏆`;
            endRound(winnerMsg);
        }

        function endRound(msg) {
            broadcastState({ type: 'ALERT', msg: msg });
            alert(msg);
            location.reload();
        }

        // --- عرض الواجهة لكل لاعب ---

        function renderGame(data) {
            document.getElementById('boneyard-count').innerText = data.boneyardCount;
            let isMyTurn = data.turnPlayerName === myName;

            document.getElementById('status-bar').innerText = isMyTurn ? "دورك الآن للعب!" : `الدور الحالي عند: ${data.turnPlayerName}`;

            // شريط اللاعبين
            let playersBar = document.getElementById('players-bar');
            playersBar.innerHTML = "";
            data.players.forEach(p => {
                let active = (p.name === data.turnPlayerName) ? 'active-turn' : '';
                playersBar.innerHTML += `<div class="player-tag ${active}">${p.name}: ${p.count} قطع</div>`;
            });

            // رسم الطاولة
            let boardEl = document.getElementById('board');
            boardEl.innerHTML = "";
            data.board.forEach(t => {
                boardEl.innerHTML += `
                    <div class="domino domino-board">
                        <div class="half">${t.a}</div>
                        <div class="divider-v"></div>
                        <div class="half">${t.b}</div>
                    </div>`;
            });
            document.getElementById('board-wrapper').scrollLeft = 9999;

            let leftEnd = data.board[0].a;
            let rightEnd = data.board[data.board.length - 1].b;

            // رسم يد اللاعب
            let handEl = document.getElementById('my-hand');
            handEl.innerHTML = "";
            let canPlayAny = false;

            data.hand.forEach((t, i) => {
                let valid = isMyTurn && (t.a === leftEnd || t.b === leftEnd || t.a === rightEnd || t.b === rightEnd);
                if (valid) canPlayAny = true;

                let d = document.createElement('div');
                d.className = `domino domino-hand ${valid ? 'playable' : ''}`;
                d.innerHTML = `<div class="half">${t.a}</div><div class="divider-h"></div><div class="half">${t.b}</div>`;
                if (valid) {
                    d.onclick = () => sendAction({ type: 'PLAY', index: i });
                }
                handEl.appendChild(d);
            });

            // تفعيل أزرار السحب والتمرير
            document.getElementById('draw-btn').disabled = !isMyTurn || data.boneyardCount === 0 || canPlayAny;
            document.getElementById('pass-btn').disabled = !isMyTurn || canPlayAny || data.boneyardCount > 0;
        }
    </script>
</body>
</html>
