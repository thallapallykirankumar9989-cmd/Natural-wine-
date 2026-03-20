<!DOCTYPE html>
<html lang="te">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mini Chess Clash</title>
    
    <link rel="stylesheet" href="https://unpkg.com/@chrisoakman/chessboardjs@1.0.0/dist/chessboard-1.0.0.min.css">
    
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            margin: 0; padding: 0;
            display: flex; justify-content: center;
        }
        .app-container {
            width: 100%; max-width: 480px;
            background-color: #ffffff; min-height: 100vh;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            display: flex; flex-direction: column;
        }
        .app-bar {
            background-color: #2962ff; /* Blue theme */
            color: white; padding: 16px 20px;
            text-align: center; font-size: 22px; font-weight: bold;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }
        .content {
            padding: 20px; flex-grow: 1;
            display: flex; flex-direction: column; align-items: center;
        }
        .player-profile {
            background-color: #e3f2fd; color: #0d47a1;
            padding: 10px 20px; border-radius: 20px;
            font-weight: bold; font-size: 16px;
            display: flex; align-items: center; gap: 8px;
            margin: 15px 0; width: fit-content;
        }
        /* చెస్ బోర్డ్ చుట్టూ షాడో మరియు డిజైన్ */
        #board-container {
            width: 100%; max-width: 400px;
            box-shadow: 0 10px 20px rgba(0,0,0,0.3);
            border-radius: 4px; border: 4px solid #4e342e;
            background-color: #4e342e;
        }
        #myBoard { width: 100%; }
        
        .status-box {
            margin-top: 20px; padding: 10px; width: 90%;
            text-align: center; font-size: 18px; font-weight: bold;
            border-radius: 8px; background-color: #fff3e0; color: #e65100;
            border: 1px solid #ffb74d;
        }
        .buttons-row {
            display: flex; justify-content: space-around;
            width: 100%; margin-top: 30px; gap: 15px;
        }
        .btn {
            flex: 1; padding: 12px; font-size: 16px; font-weight: bold;
            border: none; border-radius: 8px; cursor: pointer; color: white;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        .btn-undo { background-color: #f57c00; }
        .btn-reset { background-color: #d32f2f; }
        .btn:active { transform: scale(0.95); }
    </style>
</head>
<body>

<div class="app-container">
    <div class="app-bar">Mini Chess Clash ♟️</div>

    <div class="content">
        <div class="player-profile">👤 Player 2 (Black)</div>

        <div id="board-container">
            <div id="myBoard"></div>
        </div>

        <div class="player-profile">👤 Player 1 (White)</div>

        <div class="status-box" id="status">White to move</div>

        <div class="buttons-row">
            <button class="btn btn-undo" id="undoBtn">↩️ Undo Move</button>
            <button class="btn btn-reset" id="resetBtn">🔄 Reset Game</button>
        </div>
    </div>
</div>

<script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
<script src="https://unpkg.com/@chrisoakman/chessboardjs@1.0.0/dist/chessboard-1.0.0.min.js"></script>

<script>
    var board = null;
    var game = new Chess();
    var $status = $('#status');

    // కాయిన్ లాగుతున్నప్పుడు (Drag Start)
    function onDragStart (source, piece, position, orientation) {
        // గేమ్ అయిపోయినా, లేదా వేరేవాళ్ళ టర్న్ అయినా కాయిన్ కదలనివ్వదు
        if (game.game_over()) return false;
        if ((game.turn() === 'w' && piece.search(/^b/) !== -1) ||
            (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
            return false;
        }
    }

    // కాయిన్ వదిలేసినప్పుడు (Drop)
    function onDrop (source, target) {
        // మూవ్ రూల్స్ ప్రకారం కరెక్టో కాదో చెక్ చేస్తుంది
        var move = game.move({
            from: source,
            to: target,
            promotion: 'q' // చివరకి వెళ్తే ఆటోమేటిక్ గా రాణి (Queen) వస్తుంది
        });

        // తప్పు మూవ్ అయితే కాయిన్ వెనక్కి వెళ్లిపోతుంది (snapback)
        if (move === null) return 'snapback';

        updateStatus();
    }

    // బోర్డ్ ని అప్‌డేట్ చేయడానికి
    function onSnapEnd () {
        board.position(game.fen());
    }

    // గేమ్ స్టేటస్ (చెక్, చెక్‌మేట్, టర్న్) చూపించడానికి
    function updateStatus () {
        var statusHTML = '';
        var moveColor = 'White';
        if (game.turn() === 'b') { moveColor = 'Black'; }

        if (game.in_checkmate()) {
            statusHTML = 'Game over, ' + moveColor + ' is in checkmate! 🏆';
        } else if (game.in_draw()) {
            statusHTML = 'Game over, drawn position 🤝';
        } else {
            statusHTML = moveColor + ' to move';
            if (game.in_check()) {
                statusHTML += ', ' + moveColor + ' is in check! ⚠️';
            }
        }
        $status.html(statusHTML);
    }

    // చెస్ బోర్డ్ సెట్టింగ్స్
    var config = {
        draggable: true, // లాగడానికి పర్మిషన్
        position: 'start', // స్టార్టింగ్ పొజిషన్
        onDragStart: onDragStart,
        onDrop: onDrop,
        onSnapEnd: onSnapEnd,
        // కాయిన్స్ ఇమేజెస్ కోసం ఆన్‌లైన్ లింక్ (దీని వల్లే బొమ్మలు కనిపిస్తాయి)
        pieceTheme: 'https://chessboardjs.com/img/chesspieces/wikipedia/{piece}.png'
    };

    // బోర్డ్ ని స్టార్ట్ చేయడం
    board = Chessboard('myBoard', config);
    updateStatus();

    // Undo బటన్ నొక్కినప్పుడు
    $('#undoBtn').on('click', function() {
        game.undo(); // లాజిక్ వెనక్కి వెళ్తుంది
        board.position(game.fen()); // బోర్డ్ వెనక్కి మారుతుంది
        updateStatus();
    });

    // Reset బటన్ నొక్కినప్పుడు
    $('#resetBtn').on('click', function() {
        game.reset(); // లాజిక్ రీసెట్
        board.start(); // కాయిన్స్ మళ్లీ మొదటికి వస్తాయి
        updateStatus();
    });

    // స్క్రీన్ సైజు మారితే బోర్డ్ కూడా సెట్ అవ్వడానికి
    $(window).resize(board.resize);
</script>

</body>
</html>
